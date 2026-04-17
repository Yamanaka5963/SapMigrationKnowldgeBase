# 40 - ABAP Code Migration Patterns — Overview

> This part is intended to support ABAP code review and remediation during S/4HANA migration projects. All patterns show real before/after code examples with reference links to the source material.

## Purpose

When migrating from SAP ECC to S/4HANA, custom ABAP code must be reviewed because:
- Several core database tables have been replaced or restructured (BSEG, KONV, MSEG, MARD, KNA1, LFA1)
- Pool and cluster tables are converted to transparent tables — direct SQL access patterns may break
- SAP HANA requires deterministic sort order (SELECT without ORDER BY is unsafe)
- Some table structures now live in CDS views, not physical tables

The **ATC (ABAP Test Cockpit)** identifies most issues automatically. The patterns in this section explain *why* each change is needed and *how* to fix it.

## Must-Fix vs. Should-Fix

| Category | Description | Risk if Left |
|---|---|---|
| **Must-Fix** | Direct INSERT/UPDATE/DELETE into simplified tables (ACDOCA, PRCD_ELEMENTS, MATDOC) | Runtime errors, data corruption |
| **Must-Fix** | SELECT from pool/cluster tables via native SQL | Syntax errors post-conversion |
| **Must-Fix** | Missing ORDER BY on BINARY SEARCH result sets | Non-deterministic results, wrong reads |
| **Should-Fix** | SELECT from BSEG/MSEG/MKPF/MARD (backward-compatible views exist) | Deprecated path, potential performance risk |
| **Should-Fix** | Direct INSERT INTO KNA1/LFA1 instead of BP API | CVI sync not triggered, inconsistency |
| **Should-Fix** | Old Open SQL syntax without host variables | Accepted by compiler but not future-proof |

> **Brownfield note on Should-Fix:** Backward-compatible views (NSDM_V_MARD, V_KONV, MSEG compatibility view in S/4HANA) are **production-safe** — they return correct data. "Should-Fix" means: migrate when you touch this code in a sprint, or use the new path for new development. It does **not** mean these must be fixed before go-live. Prioritize Must-Fix items for cutover.

## Section Map

| File | Patterns Covered |
|---|---|
| [41 — Financial Patterns](41-financial-patterns.md) | BSEG/BKPF reads, FI document posting via BAPI |
| [42 — Pricing Patterns](42-pricing-patterns.md) | KONV → PRCD_ELEMENTS / V_KONV / API |
| [43 — Inventory Patterns](43-inventory-patterns.md) | MSEG/MKPF/MARD → NSDM views |
| [44 — Business Partner Patterns](44-bp-master-data-patterns.md) | KNA1/LFA1 → CL_MD_BP_MAINTAIN / BAPI_BUPA |
| [45 — Pool/Cluster Patterns](45-pool-cluster-patterns.md) | STXH/STXL, PCL1/PCL2, VBUK/VBUP |
| [46 — General ABAP Patterns](46-general-abap-patterns.md) | ORDER BY, Open SQL host variables, FOR ALL ENTRIES |

## How to Use During a Migration Project

1. Run **SAP Readiness Check** on the ECC source system to get a custom code volume report
2. Run **ATC with S/4HANA ruleset** in Eclipse/ADT — export findings as a list
3. For bulk simple fixes (ORDER BY, syntax), use ATC **Quick Fixes** to auto-apply
4. For table replacement patterns (BSEG, KONV, MARD, KNA1), use these files to guide manual remediation
5. Validate all changes in the sandbox/dev S/4HANA system before brownfield go-live

## References

- SAP Community — Custom code adaptation after S/4HANA system conversion (by SAP): https://community.sap.com/t5/enterprise-resource-planning-blog-posts-by-sap/semi-automated-custom-code-adaptation-after-sap-s-4hana-system-conversion/ba-p/13359048
- SAP Help — ATC documentation: https://help.sap.com/docs/SAP_NETWEAVER_AS_ABAP_FOR_SOH_740/c1dea27e8c984234bdb31ea86cd6b4d4/4ec5711c6e391014adc9fffe4e204223.html
- Custom Code Remediation guide in this KB: [16 — Custom Code Remediation](../part2-pro/16-custom-code-remediation.md)
