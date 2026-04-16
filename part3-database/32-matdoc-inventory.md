# 32 - MATDOC: Inventory Management Consolidation

## What Is It

MATDOC (Material Document) is the new central table for all inventory movements in SAP S/4HANA. It replaces the ECC split between document header (`MKPF`) and line items (`MSEG`), and eliminates the redundant aggregate/snapshot tables that existed for reporting performance.

This change is part of the **New Simplified Data Model (NSDM)** for Inventory Management.

## ECC Tables Replaced

| ECC Table | Description | Status in S/4HANA |
|---|---|---|
| `MKPF` | Material Document Header | Replaced by CDS view `NSDM_E_MKPF` |
| `MSEG` | Material Document Segment (line items) | Replaced by CDS view `NSDM_E_MSEG` |
| `MARD` | Storage Location Stock | Still exists; stock figures calculated on-the-fly via CDS |
| `MCHB` | Batch Stock | Calculated on-the-fly |
| `MSKA` | Sales Order Stock | Calculated on-the-fly |
| `MSSL` | Special Stock with Vendor | Calculated on-the-fly |
| `MSSQ` | Project Stock | Calculated on-the-fly |
| Aggregate/history tables | Redundant stock snapshots | Eliminated |

## New MATDOC-Related Tables

| Table | Description |
|---|---|
| `MATDOC` | Central material document table (header + line items combined) |
| `MATDOC_EXTRACT` | Condensed extract for on-the-fly stock calculations (performance) |

## MATDOC Record Types

| Record Type | Description |
|---|---|
| `MDOC` | Standard header/item entries (replaces MKPF/MSEG) |
| `MDOC_CP` | Complementary postings (replaces XAUTO entries) |
| `101` | Stock in transit |
| `MIG_DELTA` | Quantity differences from archived documents post-migration |
| `ARC_DELTA` | Archived material documents no longer in MATDOC |

## How Stock Figures Are Calculated

In ECC, stock quantities were stored in snapshot tables (`MARD`, `MCHB`, etc.) and updated with each movement — write-heavy, redundant.

In S/4HANA, stock is **calculated on-the-fly** from `MATDOC` via CDS views. `MATDOC_EXTRACT` provides a condensed version for performance optimization.

```
MATDOC (all movements) → CDS aggregation → Current stock figure
```

`MARD` still exists in S/4HANA but its values are now derived, not directly written.

## MIG_DELTA — Post-Migration Reconciliation

After migration, a reconciliation step compares:
- Stock quantities in `MATDOC_EXTRACT`
- Stock quantities in old `MARD` / `LABST` fields

Differences are recorded as `MIG_DELTA` entries in MATDOC. Engineers must validate these are resolved — unexplained MIG_DELTA entries indicate a migration data quality issue.

**Migration programs:**
- `NSDM_MIG_MATERIAL_DOC` — migrates material documents to MATDOC
- `NSDM_MIG_FILL_EXTRACT` — fills MATDOC_EXTRACT
- `NSDM_MIG_CREATE_PERFY` — creates performance index

## Why It Matters for Custom Code

Any Z-program or report that:
- `SELECT`s directly from `MKPF` or `MSEG`
- Updates stock via non-standard function modules
- Reads stock figures directly from `MARD`, `MCHB`, etc.

...must be refactored. Use:
- CDS views `NSDM_E_MKPF` and `NSDM_E_MSEG` for material document reads
- Standard SAP BAPIs/APIs for inventory postings
- CDS views for stock reporting (do not read `MARD` directly)

## References

- SAP Community — NSDM for Inventory Management (by SAP): https://community.sap.com/t5/enterprise-resource-planning-blog-posts-by-sap/sap-s-4hana-inventory-management-tables-new-simplified-data-model-nsdm/ba-p/13497469
- SAP Community — MSEG or MATDOC, where is my data? (member post): https://community.sap.com/t5/enterprise-resource-planning-blog-posts-by-members/s-4-hana-mseg-or-matdoc-where-is-my-data-going/ba-p/13406549
- SAP Help — New Simplified Data Model (NSDM): https://help.sap.com/docs/SUPPORT_CONTENT/erpscm/3362167285.html
- Simplification List S/4HANA 2023 (official PDF): https://help.sap.com/doc/c34b5ef72430484cb4d8895d5edd12af/2023/en-US/SIMPL_OP2023.pdf
