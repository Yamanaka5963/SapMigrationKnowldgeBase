# 11 - Assessment & Readiness Check

## Overview

The SAP Readiness Check is the primary automated tool for assessing an ECC system's readiness for S/4HANA. Run it early — findings drive approach selection, scoping, and project planning.

## What It Checks

| Category | Details |
|---|---|
| **Custom Code Analysis** | Volume of custom ABAP, compatibility issues with S/4HANA |
| **Simplification Items** | ECC features/tables removed or changed in S/4HANA (430+ items) |
| **Add-On Compatibility** | Whether installed add-ons are S/4HANA-certified |
| **Business Functions** | Active Business Functions in scope |
| **Business Process Discovery** | Which standard SAP processes are actively used |
| **S/4HANA Sizing** | Hardware sizing for target system |
| **Data Volume Management** | Data archiving recommendations |
| **Fiori App Recommendations** | Recommended Fiori apps for active processes |
| **Planned Downtime Analysis** | Estimated downtime for conversion |
| **Customer Vendor Integration (CVI)** | BP migration readiness (customers/vendors → Business Partners) |

## How to Run

1. Install SAP Note **2399707** on the source ECC system
2. Execute the Readiness Check from within the source system or via SAP Cloud ALM
3. Upload results to the Readiness Check portal for the full results dashboard
4. Download the Results Document (PDF) for offline review and sharing with client

## Reading the Results

- **Red items** — blockers that must be resolved before conversion can proceed
- **Yellow items** — items requiring review and decision
- **Green items** — no action required

Focus first on:
1. Add-on compatibility (hard blockers if not S/4HANA certified)
2. Simplification Items with mandatory adaptation
3. Custom code volume and critical findings

## Simplification Items

Each simplification item maps to a dedicated SAP Note. Key categories:
- Removed or changed application functionality
- Table structure changes (pool/cluster → transparent)
- FI-CO integration changes
- Material Ledger mandatory activation
- Business Partner (CVI) mandatory migration

Simplification Lists (PDF):
- S/4HANA 2023: https://help.sap.com/doc/c34b5ef72430484cb4d8895d5edd12af/2023/en-US/SIMPL_OP2023.pdf
- S/4HANA 2022: https://help.sap.com/doc/59bf7d0f62d24af78f87c560da8f18ce/2022/en-US/SIMPL_OP2022.pdf

## References

- SAP Readiness Check help portal (official): https://help.sap.com/docs/SAP_READINESS_CHECK
- SAP Readiness Check functions (official): https://help.sap.com/docs/SAP_READINESS_CHECK/a281af437b3e4ef4a187c7f35a9093e9/69463edf26e44ba8a99e16c0c0e27100.html
- SAP Readiness Check topic page (SAP Community): https://pages.community.sap.com/topics/readiness-check
- Understanding Readiness Check and Simplification Check (community post): https://community.sap.com/t5/technology-blog-posts-by-members/understanding-sap-readiness-check-and-simplification-check-for-sap-s-4hana/ba-p/13770134
- Simplification Item Catalog overview (by SAP): https://community.sap.com/t5/enterprise-resource-planning-blog-posts-by-sap/simplification-item-catalog-simplification-item-check-and-sap-readiness/ba-p/13440989
- SAP Note 2399707 — Readiness Check installation (requires S-User)
