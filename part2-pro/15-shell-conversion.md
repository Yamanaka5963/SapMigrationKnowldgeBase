# 15 - Shell Conversion Guide (Selective Data Transition)

## Overview

Shell Conversion (also called Selective Data Transition / SDT) is a hybrid approach. It creates a copy of the existing ECC system (the "shell") — preserving all configuration and repository objects but excluding master and transactional data — then converts the shell to S/4HANA, and selectively migrates only the data that is needed.

It combines the speed of Brownfield (reuse existing config) with the data selectivity of Greenfield (choose what to bring forward).

**Industry adoption: ~42%** — the most popular approach alongside Brownfield.

## Two Variants

| Variant | Description |
|---|---|
| **Shell Conversion** | Retains all legacy customizing and interfaces; selectively migrates data |
| **Mix & Match** | Redesigns selected business processes while retaining others; partial config reuse |

## When to Use

- Client wants to remove obsolete company codes, historical data, or inactive org units
- Phased go-live by company code is required
- Near-zero downtime is a hard requirement
- Data quality is poor and migration is an opportunity to cleanse
- Some processes need redesign, but not all

## High-Level Process

### Step 1 — Shell Copy Creation
- Create a 1:1 copy of the ECC system
- Strip all master data and transactional data from the copy
- Retain: customizing, configuration, repository objects (ABAP, dictionary)
- Tool: SAP Business Transformation Center (BTC) or partner tools (SNP, Syniti, etc.)

### Step 2 — Shell System Conversion
- Convert the shell (config-only copy) to S/4HANA using SUM/DMO
- Follow standard brownfield conversion steps — see [13 - Brownfield Execution](13-brownfield-execution.md)
- SPDD/SPAU adjustments still apply
- Custom code remediation still required — see [16 - Custom Code Remediation](16-custom-code-remediation.md)

### Step 3 — Data Scope Definition
Define which data to migrate using SAP Business Transformation Center (BTC):
- Which company codes
- Which fiscal years / time ranges
- Which master data objects
- Which open transactional data

### Step 4 — Data Migration
- Migrate selected data from legacy ECC to the converted S/4HANA shell
- Database-level migration for high-volume data (faster than file-based)
- Data cleansing and harmonization must be done before or during this step

### Step 5 — Cutover
- Supports multiple cutover strategies:
  - **Big bang** — all company codes at once
  - **Company-code staged** — phased go-live per company code
  - **Near-zero downtime** — parallel operation with sync until cutover

> Data cleansing is mandatory — migrating data does not make it fit for S/4HANA. Plan data cleansing and post-go-live governance as part of the project scope.

## Key Tools

| Tool | Role |
|---|---|
| **SAP Business Transformation Center (BTC)** | SAP's native tool for data readiness analysis and SDT orchestration |
| **SNP Transformation Backbone** | Common partner tool for high-speed selective migration |
| **Syniti Data Migration** | Partner tool for data migration and quality management |

## References

- SAP Support — Selective Data Transition Engagement (official): https://support.sap.com/en/offerings-programs/support-services/data-management-landscape-transformation/selective-data-transition-engagement.html
- SAP Community — Move to S/4HANA with SDT (by SAP): https://community.sap.com/t5/enterprise-resource-planning-blog-posts-by-sap/move-to-sap-s-4hana-with-selective-data-transition/ba-p/13455833
- SAP Community — SDT using SAP Business Transformation Center (by SAP): https://community.sap.com/t5/technology-blog-posts-by-sap/selective-data-transition-using-sap-business-transformation-center-btc/ba-p/14165170
- SAP Learning — Illustrating Selective Data Transition (official): https://learning.sap.com/learning-journeys/discovering-sap-activate-implementation-tools-and-methodology/illustrating-selective-data-transition
- SAP Community — Is SDT the best fit? (Q&A): https://community.sap.com/t5/enterprise-resource-planning-q-a/is-selective-data-transition-the-best-fit-for-your-sap-s-4hana-migration/qaq-p/14126275
