# 33 - Pool/Cluster Tables → Transparent Tables

## Background

ECC supports three types of database tables:
- **Transparent tables** — 1:1 mapping to a DB table (standard)
- **Pool tables** — multiple logical tables stored in one physical DB table (`ATAB`)
- **Cluster tables** — multiple logical tables stored in one physical DB table with binary-encoded data

SAP HANA supports **transparent tables only**. During system conversion (brownfield/shell), all pool and cluster tables are automatically converted to transparent tables by SUM/DMO. This process is called **declustering/depooling**.

## Major Pool and Cluster Tables in ECC

### Cluster Tables

| Table | Cluster Name | Description | S/4HANA Replacement |
|---|---|---|---|
| `KONV` | `KAPOL` | Pricing condition records | `PRCD_ELEMENTS` (transparent) |
| `BSEG` | `RFBLG` | FI document segment | Transparent (data migrated to ACDOCA) |
| `BSEC` | `RFBLG` | FI one-time account data | Transparent |
| `BSET` | `RFBLG` | FI tax data | Transparent |
| `BSIS` | `RFBLG` | FI secondary index (G/L) | Transparent |
| `STXH` | `STXD` | SAPscript text header | Transparent |
| `STXL` | `STXD` | SAPscript text body | Transparent |
| `PCL1`–`PCL4` | HR clusters | HR payroll/time data clusters | Transparent |

### Pool Tables

| Table | Pool Name | Description |
|---|---|---|
| `FUPARAREF` | `FUPARA` | Function module parameter references |
| `ADCP` | `ADA` | Address/person assignment |
| `CDPOS_STR` | `CDCLS` | Change document string values |
| Various SYST* | system pools | Miscellaneous system tables |

> To get the full list of pool/cluster tables in a specific client system, use SAP Note **2634739** which provides the SQL query to run against the ABAP Dictionary metadata tables (`DD02L` with `TABCLASS = 'POOL'` or `'CLUSTER'`).

## How Conversion Works During SUM

1. SUM identifies all pool/cluster tables in the system
2. New transparent table structures are created on SAP HANA
3. Data is migrated from the pooled/clustered physical storage to the new transparent tables
4. Large tables (e.g., `KONV`, `VBFA`) have their field changes executed **during the uptime phase** to minimize downtime impact
5. After conversion, the pool/cluster physical container tables no longer exist

**Limitation:** Tables with more than **749 fields** cannot be directly converted — they must be restructured before conversion can proceed.

## KONV → PRCD_ELEMENTS (Key Example)

`KONV` (Pricing Conditions) is one of the most impactful conversions for custom code.

| | ECC | S/4HANA |
|---|---|---|
| Table | `KONV` (cluster) | `PRCD_ELEMENTS` (transparent) |
| Access | Direct `SELECT FROM KONV` | API or CDS view |
| Direct `SELECT` | Works | Not supported |

**In S/4HANA, use:**
- API: `cl_prc_result_factory=>get_instance()->get_prc_result()`
- Fallback CDS view: `V_KONV_CDS` (read-only, for transition period)
- Do **not** write directly to `PRCD_ELEMENTS` — use pricing APIs

## Impact for Custom Code

Any code accessing pool or cluster tables directly must be remediated:

| Pattern | ECC | S/4HANA Action |
|---|---|---|
| `SELECT FROM KONV` | Works | Use `PRCD_ELEMENTS` or `V_KONV_CDS` |
| `SELECT FROM STXH/STXL` | Works | Use `READ_TEXT` / `SAVE_TEXT` function modules |
| `SELECT FROM PCL1/PCL2` | Works | Use HR cluster read APIs |
| `SELECT FROM BSEG` | Works | Use compatibility view or ACDOCA |
| Direct `INSERT/UPDATE` on pool/cluster | Works | Blocked — use standard APIs |

## References

- ABAP Keyword Doc — Transforming Pooled and Cluster Tables (official): https://help.sap.com/doc/abapdocu_752_index_htm/7.52/en-US/abenddic_database_tables_poclutr.htm
- ABAP Keyword Doc — Converting Database Tables (official): https://help.sap.com/doc/abapdocu_752_index_htm/7.52/en-US/abenddic_database_tables_conv.htm
- ABAP Keyword Doc — Pooled and Cluster Tables overview (official): https://help.sap.com/doc/abapdocu_750_index_htm/7.50/en-US/abenddic_database_tables_poolclu.htm
- SAP Community — Myth and truth about cluster/pool tables on HANA (member post): https://community.sap.com/t5/technology-blog-posts-by-members/myth-and-truth-about-cluster-pool-tables-on-hana/ba-p/13372144
- SAP Community — ECC tables to be replaced in S/4 brownfield (Q&A): https://community.sap.com/t5/enterprise-resource-planning-q-a/code-remediation-ecc-tables-to-be-replaced-in-s-4-brownfield-implementation/qaq-p/12648389
- SAP Note 2634739 — SQL to list all pool/cluster tables in a system (S-User required)
