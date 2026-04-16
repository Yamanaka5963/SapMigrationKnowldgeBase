# 03 - Key Tools & Terminology

| Term / Tool | Description | Reference |
|---|---|---|
| **SUM** (Software Update Manager) | Core tool for brownfield in-place conversion. Manages upgrade stack execution. | [SAP Community (by SAP)](https://community.sap.com/t5/technology-blog-posts-by-sap/database-migration-option-dmo-of-sum-introduction/ba-p/13262160) |
| **DMO** (Database Migration Option) | SUM add-on to migrate DB to SAP HANA in the same run as system conversion. | Same as above |
| **Migration Cockpit** | SAP tool for migrating business data (master & transactional) using predefined migration objects. | [SAP Help Portal](https://help.sap.com/docs/SAP_S4HANA_CLOUD/d5699934e7004d048c4801b552f3b013/121b34742a904d10bca907bbf2fd5617.html) · [SAP Community topic](https://pages.community.sap.com/topics/s4hana-migration-cockpit) |
| **Readiness Check** | Automated tool that analyzes client's ECC for S/4HANA compatibility — add-ons, custom code, simplification items. | [SAP Help Portal](https://help.sap.com/docs/cloud-alm/applicationhelp/sap-readiness-check) |
| **RISE with SAP** | SAP's managed cloud migration offering. Bundles S/4HANA Private Cloud + tools + methodology. | [SAP official](https://www.sap.com/products/erp/rise.html) |
| **RISE Transition Workbench** | Single-entry tool for RISE migrations. 5 technical migration approaches, guided step-by-step. Pre-installed on migration server. | [SAP Support](https://support.sap.com/en/tools/software-logistics-tools/rise-with-sap-system-transition-workbench.html) |
| **SPDD / SPAU** | Re-adjust dictionary objects (SPDD) and repository objects (SPAU) after system conversion. Required post-upgrade step. | Standard SAP transaction |
| **SAP Activate** | SAP's current project methodology (agile, replaced ASAP). 6 phases. | See [04 - SAP Activate Methodology](04-sap-activate-methodology.md) |
| **Simplification Items** | ECC features/tables changed or removed in S/4HANA. Must be reviewed during assessment. | Included in Readiness Check output |
