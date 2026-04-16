# 16 - Custom Code Remediation

## Overview

Custom ABAP code (Z-programs, Z-tables, enhancements, modifications) must be analyzed and adapted for S/4HANA compatibility before and after system conversion. This is one of the most effort-intensive workstreams in any brownfield or shell conversion project.

## Tools

| Tool | Purpose |
|---|---|
| **ABAP Test Cockpit (ATC)** | Primary tool — runs S/4HANA-specific checks on custom code, produces findings list |
| **Custom Code Migration App** (Fiori) | Manages custom code analysis workflow, tracks remediation progress |
| **ABAP Development Tools (ADT) in Eclipse** | Provides quick-fix suggestions — covers ~40-60% of typical issues automatically |
| **SAP Code Inspector (SCI)** | Supplementary analysis with FUNCTIONAL_DB check variant |
| **SAP Joule for Developers** | AI-assisted custom code migration (newer, reduces project timelines) |

## Remediation Process

### Step 1 — Pre-Conversion Analysis
- Run ATC with S/4HANA check variant on DEV system
- Review findings: categorize by severity and volume
- Identify unused custom objects → decommission before remediation (saves effort)
- Estimate remediation effort per finding category

### Step 2 — Decommission Unused Code
- Remove or retire Z-programs/enhancements that are no longer in use
- Use usage statistics (transaction SUSG or similar) to identify unused objects
- Less code = less remediation = less regression testing

### Step 3 — Remediation (Pre- and Post-Conversion)
- Apply quick fixes in ADT in Eclipse where available (~40-60% of issues)
- Manually fix remaining findings — reference the relevant SAP Note per simplification item
- Key issues to address:
  - **Native SQL from predecessor DBs** — must be eliminated
  - **SELECT without ORDER BY** — unexpected behavior on HANA (no guaranteed sort order)
  - **Pool/cluster table access** — converted to transparent tables in S/4HANA
  - **Removed/changed standard SAP objects** — review simplification item notes
  - **Implicit ORDER BY PRIMARY KEY** — behavior changed for transparent tables

### Step 4 — SPDD / SPAU (During Conversion Downtime)
- **SPDD** — re-adjust Data Dictionary modifications (runs early in SUM downtime phase)
- **SPAU** — re-adjust repository object modifications (runs at end of SUM procedure)
- **SPAU_ENH** — for enhancement-based modifications
- Prepare adjustment transport requests before downtime begins

### Step 5 — Post-Conversion Verification
- Run ATC again on the converted system
- Fix any remaining findings
- Perform functional testing on custom objects

## Common Simplification Item Categories

S/4HANA has **430+ simplification items**. High-impact categories to prioritize:
- FI-CO integration changes (profit center, cost element merged into G/L)
- Material Ledger — mandatory activation
- Business Partner (CVI) — customers and vendors must become BPs
- Inventory management tables (MATDOC replaces MSEG + MKPF)
- HR structure changes (if HCM in scope)

Each simplification item references a dedicated SAP Note with adaptation instructions.

## References

- Custom Code Migration Guide for S/4HANA 2025 (official PDF): https://help.sap.com/doc/9dcbc5e47ba54a5cbb509afaa49dd5a1/2025.001/en-US/CustomCodeMigration_EndtoEnd.pdf
- SAP Community — Get started with ABAP custom code migration (by SAP): https://community.sap.com/t5/technology-blog-posts-by-sap/get-started-with-the-abap-custom-code-migration-process/ba-p/13531886
- SAP Community — Custom code adaptation process (by SAP): https://community.sap.com/t5/enterprise-resource-planning-blog-posts-by-sap/sap-s-4hana-system-conversion-custom-code-adaptation-process/ba-p/13337309
- SAP Community — Custom code impact analysis using ATC (member post): https://community.sap.com/t5/enterprise-resource-planning-blog-posts-by-members/s-4hana-custom-code-impact-analysis-using-atc/ba-p/13566164
- SAP Community — Custom code migration with SAP Joule (by SAP, newer): https://community.sap.com/t5/technology-blog-posts-by-sap/custom-code-migration-to-sap-s-4hana-powered-by-sap-joule-for-developers/ba-p/14329094
- Simplification Item Catalog overview (by SAP): https://community.sap.com/t5/enterprise-resource-planning-blog-posts-by-sap/simplification-item-catalog-simplification-item-check-and-sap-readiness/ba-p/13440989
