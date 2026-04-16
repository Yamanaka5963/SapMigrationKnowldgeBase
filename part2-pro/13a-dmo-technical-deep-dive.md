# 13a - DMO Technical Deep Dive

> This is a supplement to [13 - Brownfield Execution Guide](13-brownfield-execution.md). It covers the internal technical mechanics of DMO (Database Migration Option) for engineers who need to understand what is happening under the hood.

## What DMO Is

DMO combines three operations into a single SUM run:
1. **Software upgrade** — ECC to target S/4HANA release
2. **Database migration** — source DB (Oracle, MSSQL, DB2, etc.) to SAP HANA
3. **Data model conversion** — restructuring data to S/4HANA simplified data model

The core engine is **R3load** — SAP's proprietary data migration utility.

## R3load — The Migration Engine

R3load runs as **parallel pairs** (one export process + one import process):

- Export R3load reads from the **source database**
- Import R3load writes to the **target SAP HANA database**
- Data is transferred via **memory pipes** — direct host main-memory transfer with no dump files and no compression
- Work is divided into **buckets** (work packages of tables); each R3load pair processes one bucket, terminates, then SUM starts the next pair

```
Source DB → [R3load export] ——pipe——> [R3load import] → Target HANA DB
```

> Pipe mode is significantly faster than traditional file-based migration (export to disk → compress → transfer → decompress → import). No intermediate files are created.

**Technical detail:** In the MIGRATE_UT command files, the export R3load starts with a socket option and listens on a port. The import R3load connects to that port to receive the piped data stream directly.

## Shadow Repository & Shadow Instance

Before any downtime begins, SUM creates two components on the target system:

| Component | Description |
|---|---|
| **Shadow Instance** | An additional ABAP instance created by SUM on the Primary Application Server, running with the **target release kernel** (not the source kernel) |
| **Shadow Repository** | A full database repository built on the **target HANA DB** during uptime — ABAP dictionary and repository objects in the new release |

The shadow repository is migrated using R3load pipe mode while users are still working on the production system. Since **SUM 2.0 SP08**, the shadow repository is created directly on the target HANA DB (previously it was staged on the source DB first).

**Purpose of the shadow system:**
- Prepare and test the system conversion on the target release without touching production
- Create new object types required for the target S/4HANA release
- Enable uptime data migration in parallel with live production operations

## UPTIME Phase — Detail

The system is **live** — users continue working normally.

### What happens:

1. SUM creates **database triggers** on selected large application tables
2. R3load pairs perform the initial migration of those tables to the target HANA DB via pipes
3. Triggers record every INSERT, UPDATE, and DELETE made by users while the initial migration runs
4. The **CRR (Change Recording & Replication)** framework runs a dedicated replicator on the shadow instance — it reads trigger logs and continuously replays delta changes to the target DB
5. The **Writer** (a temporary TMP instance using the target kernel) receives the deltas and writes them to the target HANA DB

```
User activity on production
        ↓
DB triggers capture changes
        ↓
CRR replicator reads trigger log
        ↓
Writer (TMP, target kernel) → Target HANA DB
```

**Key phase name:** `EU_CLONE_MIG_OPTDMO_INI_RUN` (downtime-optimized variant)

**Important constraints during uptime phase:**
- Avoid large batch jobs or mass data changes — trigger logs grow rapidly and slow replication
- Avoid archiving on tables being migrated — delta replication cannot handle archived data
- SLT (SAP Landscape Transformation) replication: existing SLT triggers may be replaced by DMO triggers; an initial reload in SLT may be required after conversion

## DOWNTIME Phase — Detail

The system is **offline**. Users cannot access the system.

| Step | SUM Phase | What Happens |
|---|---|---|
| 1 | — | SAP system shuts down; no new changes possible |
| 2 | `MIG2NDDB_SWITCH` | **DB connection switches from source DB to target HANA** |
| 3 | — | Target kernel directories activated (kernel switch) |
| 4 | — | Final R3load migration of remaining tables not migrated during uptime |
| 5 | — | Delta replay — all changes captured by triggers during uptime are replayed to target HANA |
| 6 | `PARCONV_UPG` | Application table structures converted to new S/4HANA release format |
| 7 | `XPRAS_AIMMRG` | Post-import programs and After Import Methods (AIM) execute — transforms data to S/4HANA data model |
| 8 | — | System starts on target HANA DB with target kernel |

> The source database is left **completely unchanged** throughout the entire process. It serves as a fallback — if something fails, SUM RESET restores the connection to the source DB.

## DMO Variants

| Variant | Description | Use Case |
|---|---|---|
| **Standard DMO** | In-place migration; App Server stays, DB migrates to HANA on same or nearby host | Standard brownfield on-premise to on-premise |
| **DMO with System Move** | Migrates to a different server or data center during the conversion run | Infrastructure refresh during migration |
| **DMOVE2S4** | Cloud-specific: migrates ECC to RISE with SAP S/4HANA Private Cloud in a single run | Moving to RISE Private Cloud |
| **Homogeneous DMO** | HANA-to-HANA migration (source already on HANA) | System already on HANA, upgrading to S/4HANA |
| **Downtime-Optimized DMO (doDMO)** | Pre-migrates large tables and converts data during uptime via CRR; minimizes downtime window | Large databases where downtime is a hard constraint |

## DMOVE2S4 — Cloud Migration Variant

DMOVE2S4 is the approach for migrating ECC on-premise to **RISE with SAP S/4HANA Private Cloud** in a single SUM run.

**Key differences from standard DMO:**

| | Standard DMO | DMOVE2S4 |
|---|---|---|
| Target | On-premise HANA | S/4HANA Private Cloud (hyperscaler) |
| R3load mode | File mode or pipe mode | **Pipe mode required** |
| Network latency requirement | None | **< 20ms** between source and target |
| Network bandwidth requirement | None | **> 400 Mbit/s** |
| HANA-to-HANA support | Via Homogeneous DMO | Not supported — use Homogeneous DMO |

**How it differs technically:**
- SUM operates on the application installed in the **target cloud environment** while initially still connected to the source on-premise DB
- Data is streamed via R3load pipe mode directly to the cloud target — no intermediate files
- SUM performs a mandatory latency check at startup; if latency > 20ms, warnings are logged in CHECKS.log and DBSTAT.LOG

**References:**
- SAP Community — DMOVE2S4: your one-step approach into the cloud (by SAP): https://community.sap.com/t5/technology-blog-posts-by-sap/dmove2s4-your-one-step-approach-into-the-cloud/ba-p/13872751
- SAP Help — DMO Move to S/4HANA on hyperscaler (official): https://help.sap.com/docs/SLTOOLSET/7d57e56e12104cc68bce7646cd9f4cbf/c17f45ffa4004ca4837913a745b3a081.html

## Downtime-Optimized DMO (doDMO)

Moves large-table migration **and** data conversion into the uptime phase. The downtime window is reduced to: final delta replay + kernel switch + consistency checks only.

### Real-world results (documented case, ~30TB database):

| Approach | Downtime |
|---|---|
| Standard DMOVE2S4 | 94.5 hours |
| Downtime-optimized DMO | **26.5 hours** |
| **Reduction** | **72%** |

### Requirements:
- SUM 2.0 SP17 or higher (General Availability since May 2023)
- At least one team member must have completed training **ADM329** + passed knowledge assessment **ADM329k**
- SUM Toolbox implemented on source system (SAP Note 3092738)
- ZDIMPANA.ZIP file from impact analysis must be available

**References:**
- SAP Support — Downtime-optimized Conversion Approach (official): https://support.sap.com/en/tools/software-logistics-tools/software-update-manager/downtime-optimized-conversion-approach.html
- SAP Learning — Understanding downtime-optimized conversion (official): https://learning.sap.com/courses/minimizing-downtime-of-system-conversion-to-sap-s-4hana-with-downtime-optimized-conversion-of-sum/understanding-the-concept-of-downtime-optimized-conversion
- SAP Community — Strategies for doDMO for large databases (member post): https://community.sap.com/t5/technology-blog-posts-by-members/strategies-for-downtime-optimized-dmo-dodmo-doc-dmove2s4-for-large/ba-p/14366512

## Key Concepts Summary

| Concept | Technical Role |
|---|---|
| **R3load** | SAP data migration utility; runs as parallel export+import pairs |
| **Memory Pipes** | Direct main-memory channel between R3load pair; no files, no compression |
| **Buckets** | Work packages of tables; one pair per bucket, sequential |
| **Shadow Instance** | Additional ABAP instance using target kernel; manages shadow repo and delta writes |
| **Shadow Repository** | Full ABAP repository on target HANA DB, built during uptime |
| **CRR** | Change Recording & Replication; captures user changes via triggers during uptime migration |
| **Writer (TMP)** | Temporary instance that writes CRR delta changes to target HANA using target kernel |
| **MIG2NDDB_SWITCH** | The SUM phase where DB connection flips from source to target HANA |
| **DMOVE2S4** | DMO variant for cloud migration to RISE Private Cloud; pipe mode required |
| **doDMO** | Downtime-optimized DMO; pre-migrates + converts large tables during uptime |

## References

- SAP Support — DMO official page: https://support.sap.com/en/tools/software-logistics-tools/software-update-manager/database-migration-option-dmo.html
- SAP Community — DMO introduction (by SAP): https://community.sap.com/t5/technology-blog-posts-by-sap/database-migration-option-dmo-of-sum-introduction/ba-p/13262160
- SAP Community — DMO pipe vs file mode for R3load (member post): https://community.sap.com/t5/technology-blogs-by-members/phases-behind-dmo-r3load-parallel-export-import-during-uptime-and-downtime/ba-p/13113084
- SAP Community — Shadow repository on target DB for S/4HANA 1909 (member post): https://community.sap.com/t5/technology-blog-posts-by-members/dmo-shadow-repository-on-target-database-for-conversion-to-sap-s4hana-1909/ba-p/13372144
- SAP Community — DMO downtime optimization by migrating tables during uptime (member post): https://community.sap.com/t5/technology-blog-posts-by-members/dmo-downtime-optimization-by-migrating-app-tables-during-uptime/ba-p/13113084
- SAP Community — CRR Control Center for Record & Replay monitoring (member post): https://community.sap.com/t5/technology-blog-posts-by-members/introducing-the-new-crr-control-center-to-monitor-record-replay-in-sum-2-0/ba-p/13372144
