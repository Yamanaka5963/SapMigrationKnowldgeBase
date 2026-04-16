# 31 - ACDOCA: Universal Journal

## What Is It

ACDOCA is the central line item table in SAP S/4HANA Finance. It replaces the module-specific financial posting tables from ECC (FI, CO, Asset Accounting, Material Ledger) with a single unified table — the "Universal Journal."

Every financial posting writes a single document to ACDOCA instead of writing to separate FI, CO, and AA tables.

## ECC Tables Consolidated into ACDOCA

| ECC Table | Module | Description |
|---|---|---|
| `BKPF` | FI | Accounting Document Header |
| `BSEG` | FI | Accounting Document Segment (line items) |
| `COEP` | CO | CO-PA line items (actual + statistical, WRTTP 04/11) |
| `COSP` | CO | CO Object actual costs (total) |
| `COSS` | CO | CO Object statistical costs (total) |
| `ANEP` | FI-AA | Asset line items |
| `ANEA` | FI-AA | Asset line items (proportional values) |
| `ANLP` | FI-AA | Asset periodic values |
| `ANLC` | FI-AA | Asset value fields |
| `MLIT` | ML | Material Ledger document items |
| `MLPP` | ML | Material Ledger periodic processing |
| `MLCD` | ML | Material Ledger cost distribution |
| `CKMI1` | ML | Material Ledger index for material documents |
| `BSIM` | ML | Secondary index for material documents |
| `GLT0` | FI-GL | G/L Account totals (now derived from ACDOCA) |
| `FAGLFLXA` | FI-GL | New G/L actual line items |
| `FAGLFLEXT` | FI-GL | New G/L totals |

## Status of ECC Tables in S/4HANA

| Table | Status | Notes |
|---|---|---|
| `BKPF` | Still exists | Read access supported |
| `BSEG` | Still exists | Used only for open item management |
| `COEP` / `COSP` / `COSS` | Compatibility views | `V_COEP` reads from ACDOCA |
| `ANEP` / `ANLP` etc. | Compatibility views | Read from ACDOCA |
| `GLT0` | Derived | Calculated from ACDOCA on demand |
| `FAGLFLXA` / `FAGLFLEXT` | Replaced | ACDOCA is the new source |

> Direct table updates to any replaced ECC table are not supported in S/4HANA. All postings must go through standard SAP APIs.

## New ACDOCA-Related Tables

| Table | Description |
|---|---|
| `ACDOCA` | Universal Journal line items (central posting table) |
| `ACDOCT` | Universal Journal totals (aggregation layer) |
| `ACDOCR` | Universal Journal — recurring entries |
| `ACACTCONF` | Actual cost confirmation items |

## Why It Matters for Custom Code

Any Z-program or report that:
- `SELECT`s directly from `BSEG`, `COEP`, `ANEP`, or other replaced tables
- Posts directly to FI/CO tables via non-standard function modules
- Uses aggregate totals from `GLT0` or `FAGLFLEXT`

...must be refactored. Use:
- **CDS views** (`V_BSEG_ORI`, `V_COEP`) for read access during transition
- **SAP BAPI / posting APIs** for write access
- **ACDOCA** directly for new reporting queries (more efficient on HANA)

## Migration Impact

- **Material Ledger activation is mandatory** in S/4HANA — no exceptions
- Ledger determination must be configured before migration (SAP Note 2505200)
- Historical data from ECC is migrated to ACDOCA during system conversion
- Post-migration: validate totals in ACDOCA match legacy balances per ledger/company code

## References

- SAP Community — All you need to know about ACDOCA (member post): https://community.sap.com/t5/enterprise-resource-planning-blog-posts-by-members/all-you-need-to-know-about-universal-journal-acdoca-sap-s-4-hana-2020/ba-p/13545279
- SAP Community — Replacing COEP, COSS, COSP with ACDOCA (Q&A): https://community.sap.com/t5/enterprise-resource-planning-q-a/replacing-coep-coss-cosp-with-acdoca/qaq-p/602531
- SAP Community — BSEG and BKPF in S/4HANA (Q&A): https://community.sap.com/t5/enterprise-resource-planning-q-a/in-s4-the-main-table-for-financial-data-is-acdoca-so-how-bseg-and-bkpf-are/qaq-p/13902812
- Simplification List S/4HANA 2023 (official PDF): https://help.sap.com/doc/c34b5ef72430484cb4d8895d5edd12af/2023/en-US/SIMPL_OP2023.pdf
- SAP Note 2505200 — Ledger determination during migration (S-User required)
