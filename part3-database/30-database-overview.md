# 30 - Database Overview: ECC vs S/4HANA

## The Core Difference

SAP ECC is database-agnostic — it runs on Oracle, Microsoft SQL Server, IBM DB2, SAP ASE (Sybase), or SAP MaxDB. The application layer is the same regardless of which DB sits underneath.

SAP S/4HANA requires **SAP HANA exclusively** — an in-memory, columnar database. This is not just a DB swap; S/4HANA's simplified data model is designed specifically for HANA's architecture.

| | SAP ECC | SAP S/4HANA |
|---|---|---|
| Supported DBs | Oracle, MSSQL, DB2, SAP ASE, MaxDB, HANA | SAP HANA only |
| Table types | Transparent, Pool, Cluster | Transparent only |
| Data model | Redundant aggregate/index tables for performance | Simplified — HANA handles aggregation in-memory |
| Stock figures | Stored in multiple snapshot tables | Calculated on-the-fly from MATDOC |
| Financial postings | Distributed across FI/CO/AA/ML tables | Single table: ACDOCA |
| Business partners | Separate customer (KNA1) and vendor (LFA1) | Unified Business Partner (BUT000) |

## Why the Data Model Changed

In ECC, performance was achieved by pre-aggregating data into summary tables (totals, indexes, snapshots). HANA's columnar, in-memory engine makes those redundant tables unnecessary — and in fact a liability (double-write overhead, consistency risks).

S/4HANA's simplified data model:
- Eliminates redundant aggregate/index tables
- Consolidates module-specific line item tables into single structures
- Replaces pool/cluster tables (not supported on HANA) with transparent tables
- Unifies business master data

## Impact for Migration Engineers

Every migration project — regardless of Brownfield, Greenfield, or Shell approach — must address:

1. **ACDOCA** — all financial postings consolidated (see [31](31-acdoca-universal-journal.md))
2. **MATDOC** — all inventory movements consolidated (see [32](32-matdoc-inventory.md))
3. **Pool/Cluster → Transparent** — table structure conversion during SUM (see [33](33-pool-cluster-conversion.md))
4. **CVI / Business Partner** — mandatory customer/vendor unification (see [34](34-cvi-business-partner.md))
5. **HANA System Views** — querying the HANA layer directly (see [35](35-hana-system-views-reference.md))

Custom code that reads or writes to any of the replaced ECC tables must be remediated. Compatibility views and APIs are provided by SAP as a transition path.

## Official References

- SAP S/4HANA Conversion Guide 2025 (official PDF, found via `help.sap.com/docs/SAP_S4HANA_ON-PREMISE`): https://help.sap.com/doc/8cec006e962a46049506bc10ed64f557/2025/en-US/CONV_PE2025.pdf
- Simplification List S/4HANA 2023 (official PDF): https://help.sap.com/doc/c34b5ef72430484cb4d8895d5edd12af/2023/en-US/SIMPL_OP2023.pdf
- Simplification List S/4HANA 2022 (official PDF): https://help.sap.com/doc/59bf7d0f62d24af78f87c560da8f18ce/2022/en-US/SIMPL_OP2022.pdf
