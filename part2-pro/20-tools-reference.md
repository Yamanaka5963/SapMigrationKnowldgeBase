# 20 - Tools Reference Card

Quick reference for all tools used across SAP S/4HANA migration projects.

## Assessment & Planning Tools

| Tool | What It Does | Where to Access |
|---|---|---|
| **SAP Readiness Check** | Analyzes ECC system for S/4HANA compatibility — custom code, add-ons, simplification items, sizing | Install via SAP Note 2399707; results at [help.sap.com](https://help.sap.com/docs/SAP_READINESS_CHECK) |
| **Maintenance Planner** | Generates stack.XML for SUM; calculates required software packages and patches | [Maintenance Planner](https://apps.support.sap.com/sap/support/mp) (S-User required) |
| **SAP Transformation Navigator** | Landscape-specific upgrade path guidance (read-only since Q1 2024) | [support.sap.com/stn](https://support.sap.com/stn) (S-User required) |
| **Simplification Item Catalog** | Full list of ECC features changed/removed in S/4HANA | Included in Readiness Check; PDFs at help.sap.com |

## Conversion & Upgrade Tools

| Tool | What It Does | Where to Access |
|---|---|---|
| **SUM** (Software Update Manager) | Core tool for brownfield in-place conversion; manages upgrade stack | [SAP Support](https://support.sap.com/en/tools/software-logistics-tools/software-update-manager.html) |
| **DMO** (Database Migration Option) | SUM add-on; migrates DB to HANA in same run as system conversion | Included in SUM; docs at [SAP Support](https://support.sap.com/en/tools/software-logistics-tools/software-update-manager/database-migration-option-dmo.html) |
| **RISE Transition Workbench** | Guided migration tool for RISE with SAP (Private Cloud) transitions; 5 technical approaches | Pre-installed on migration server; [SAP Support](https://support.sap.com/en/tools/software-logistics-tools/rise-with-sap-system-transition-workbench.html) |

## Custom Code Tools

| Tool | What It Does | Where to Access |
|---|---|---|
| **ABAP Test Cockpit (ATC)** | Checks custom ABAP for S/4HANA compatibility issues | Transaction ATC in SAP system |
| **Custom Code Migration App** | Fiori app to manage custom code analysis and remediation tracking | S/4HANA Fiori launchpad |
| **ABAP Development Tools (ADT)** | Eclipse plugin; provides quick-fix suggestions for ATC findings (~40-60% auto-fixable) | Eclipse marketplace |
| **SAP Code Inspector (SCI)** | Supplementary static analysis; use FUNCTIONAL_DB check variant | Transaction SCI in SAP system |
| **SAP Joule for Developers** | AI-assisted custom code migration (newer tool, reduces effort) | SAP BTP / S/4HANA Cloud |
| **SPDD** | Re-adjust Data Dictionary modifications post-conversion | Transaction SPDD |
| **SPAU / SPAU_ENH** | Re-adjust repository/enhancement modifications post-conversion | Transaction SPAU |

## Data Migration Tools

| Tool | What It Does | Where to Access |
|---|---|---|
| **Migration Cockpit** | Migrate master and transactional data using predefined objects; file/staging/direct transfer | Built into S/4HANA; [topic page](https://pages.community.sap.com/topics/s4hana-migration-cockpit) |
| **SAP Business Transformation Center (BTC)** | Data readiness analysis and orchestration for Shell Conversion (SDT) | SAP-managed; part of SDT engagement |
| **SAP Data Services** | ETL tool for populating staging tables for Migration Cockpit | SAP BTP / on-premise |

## Project & Monitoring Tools

| Tool | What It Does | Where to Access |
|---|---|---|
| **SAP Cloud ALM** | Project management, test management, and monitoring for S/4HANA (preferred for new projects) | [SAP Cloud ALM](https://www.sap.com/products/technology-platform/cloud-alm.html) |
| **SAP Solution Manager** | Legacy ALM platform; still used for older projects (transport management, monitoring) | On-premise SAP SolMan system |
| **SAP Early Watch Alert** | Periodic system health check report delivered by SAP | Triggered via Solution Manager or Cloud ALM |

## Key Transactions (Quick Reference)

| Transaction | Purpose |
|---|---|
| **SPDD** | Data Dictionary modification adjustment |
| **SPAU / SPAU_ENH** | Repository/enhancement modification adjustment |
| **ATC** | ABAP Test Cockpit — custom code checks |
| **SCI** | SAP Code Inspector |
| **SUSG** | Usage and procedure logging — identify unused custom objects |
| **SM37** | Job monitoring — batch jobs |
| **SM21** | System log |
| **SNOTE** | SAP Note implementation |
| **STMS** | Transport Management System |
