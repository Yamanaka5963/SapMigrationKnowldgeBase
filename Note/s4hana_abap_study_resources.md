# S/4HANA ABAP Study Resources

Curated, trustworthy links for studying S/4HANA business-logic / core / DB access via ABAP, plus hyperscaler (Azure / AWS) hands-on paths.

Context: based on the SAP stack doc at
<https://github.com/Yamanaka5963/SapMigrationKnowldgeBase/blob/main/Note/sap_tech_stack.md>

Layer mapping used below:

- UI Layer: SAPGUI, Fiori/UI5
- Application Layer: BTP → CAP (Node.js/Java) **or** RAP (ABAP on BTP)
- API: OData
- **ERP Core: S/4HANA on ABAP ← focus area**
- **Data Layer: HANA DB ← focus area**

Modern direct-access stack in 2026: **CDS (Virtual Data Model) + RAP + Clean Core / ABAP Cloud**, developed in **ADT (Eclipse)**.

---

## 1. ABAP language itself (authoritative reference)

- [ABAP Keyword Documentation — latest](https://help.sap.com/doc/abapdocu_latest_index_htm/latest/en-US/index.htm) — official full language reference.
- [ABAP Keyword Documentation — Cloud edition](https://help.sap.com/doc/abapdocu_cp_index_htm/CLOUD/en-US/index.htm) — restricted ABAP subset used in S/4HANA Cloud / Steampunk.
- [ABAP Language Versions & Released APIs](https://help.sap.com/doc/abapdocu_latest_index_htm/latest/en-US/abenabap_versions_and_apis.htm) — which APIs are safe under clean-core.
- [SAP-samples/abap-cheat-sheets (GitHub)](https://github.com/SAP-samples/abap-cheat-sheets) — official executable examples.

## 2. DB layer access — CDS & the Virtual Data Model (VDM)

How modern ABAP reaches HANA DB (code pushdown; no raw-table SELECTs).

- [Virtual Data Model and CDS Views — S/4HANA On-Prem](https://help.sap.com/docs/SAP_S4HANA_ON-PREMISE/ee6ff9b281d8448f96b4fe6c89f2bdc8/8573b810511948c8a99c0672abc159aa.html)
- [VDM Layers and View Types](https://help.sap.com/docs/SAP_S4HANA_ON-PREMISE/ee6ff9b281d8448f96b4fe6c89f2bdc8/0a875bc7a005465aad92c08becc11776.html)
- [Virtual Data Model — S/4HANA Cloud](https://help.sap.com/docs/SAP_S4HANA_CLOUD/c0c54048d35849128be8e872df5bea6d/8573b810511948c8a99c0672abc159aa.html)
- [ABAP Core Data Services — Best Practice Guide (SAP PDF)](https://www.sap.com/documents/2019/01/0e6d5904-367d-0010-87a3-c30de2ffd8ff.html)
- [SAP Learning: Building Data Models with ABAP Dictionary & CDS](https://learning.sap.com/courses/building-data-models-with-the-abap-dictionary-and-abap-core-data-services/exploring-abap-core-data-services_f6b58d1f-ca51-461b-8355-aef95aa3864e)

## 3. Business logic layer — RAP (ABAP RESTful Application Programming Model)

RAP is where behavior/business logic lives on top of CDS, exposed as OData.

- [Defining the Data Model in CDS Views — ABAP Cloud / RAP](https://help.sap.com/docs/abap-cloud/abap-rap/defining-data-model-in-cds-views) — official RAP docs.
- [SAP Community RAP topic page](https://pages.community.sap.com/topics/abap/rap) — hub for blogs, Q&A, learning map.
- [Coursera — RAP and Extensions (SAP)](https://www.coursera.org/learn/restful-application-programming-model-and-extensions) — SAP-authored course.
- [SAP PRESS — Learn ABAP RAP (2nd ed., 2025)](https://www.sap-press.com/abap-restful-application-programming-model_6161/) — most current book.

## 4. Clean Core / ABAP Cloud (allowed ways to touch the core in 2026)

Essential framing: direct DB writes / classic modifications are Level D (avoid).

- [Clean Core Extensibility Guide — Aug 2025 Update (SAP Community)](https://community.sap.com/t5/technology-blog-posts-by-sap/abap-extensibility-guide-clean-core-for-sap-s-4hana-cloud-august-2025/ba-p/14175399)
- [Clean Core Extensibility — Private Cloud with ABAP Cloud (SAP PDF)](https://www.sap.com/assetdetail/2024/10/fe82c9b4-db7e-0010-bca6-c68f7e60039b.html)
- [SAP Learning — Practicing Clean Core Extensibility](https://learning.sap.com/courses/practicing-clean-core-extensibility-for-sap-s-4hana-cloud)

### Clean Core A–D levels (memory aid)

- **A** — Released SAP APIs via ABAP Cloud or BTP side-by-side. Upgrade-safe. Recommended.
- **B** — Classic but supported APIs. Stable, needs governance.
- **C** — Internal SAP objects (internal FMs/classes). Upgrade risk.
- **D** — Core mods, implicit enhancements, direct writes to SAP tables. Avoid.

## 5. Tooling — ADT (Eclipse)

- [SAP Development Tools download site](https://tools.hana.ondemand.com/) — current ADT update URL.
- [Tutorial: Install Eclipse + ADT](https://developers.sap.com/tutorials/abap-install-adt..html)
- [ADT Installation Guide (PDF)](https://help.sap.com/doc/2e9cf4a457d84c7a81f33d8c3fdd9694/Cloud/en-US/inst_guide_abap_development_tools.pdf)

## 6. Free official courses (SAP Learning — the former openSAP)

- [SAP Learning — courses hub](https://learning.sap.com/courses)
- [SAP S/4HANA Development track](https://learning.sap.com/products/business-technology-platform/development/s4hana-development)
- [SAP Expert Lectures (former openSAP archive)](https://learning.sap.com/expert-lectures-former-opensap)
- [Learning Journey: Acquiring Core ABAP Skills](https://learning.sap.com/learning-journeys/acquiring-core-abap-skills)
- [Coursera specialization: S/4HANA — From ABAP to Cloud-Ready Apps](https://www.coursera.org/specializations/sap-s4hana-from-abap-to-cloud-ready-applications)

---

## 7. Hyperscaler hands-on — Azure / AWS

Two different things get conflated — split them:

### A. Hyperscaler hosts the SAP system, you develop ABAP on it

Azure/AWS provide **infrastructure only**. ABAP dev still happens in ADT (Eclipse) against the system. These tutorials teach you how to **stand up** S/4HANA, not how to code ABAP.

#### Azure

Note: "AOSS" is not an official product name. It usually refers to **ACSS = Azure Center for SAP Solutions** (sometimes marketed as "Azure on SAP").

- [Microsoft Learn — Azure Center for SAP Solutions (overview)](https://learn.microsoft.com/en-us/azure/sap/center-sap-solutions/overview)
- [Microsoft Learn training module — Explore ACSS](https://learn.microsoft.com/en-us/training/modules/explore-azure-center-sap-solutions/)
- [AZ-120 Lab — ACSS Deployment (hands-on)](https://microsoftlearning.github.io/AZ-120-Planning-and-Administering-Microsoft-Azure-for-SAP-Workloads/Instructions/AZ-120_Lab05-ACSS_Deployment.html)
- [AZ-120 GitHub labs repo](https://github.com/MicrosoftLearning/AZ-120-Planning-and-Administering-Microsoft-Azure-for-SAP-Workloads)
- [Azure/Azure-Center-for-SAP-solutions (GitHub)](https://github.com/Azure/Azure-Center-for-SAP-solutions)
- [Microsoft Certified: Azure for SAP Workloads Specialty (AZ-120)](https://learn.microsoft.com/en-us/certifications/azure-for-sap-workloads-specialty/)

#### AWS

- [AWS Launch Wizard for SAP — docs](https://docs.aws.amazon.com/launchwizard/latest/userguide/launch-wizard-sap.html)
- [AWS Launch Wizard for SAP tutorials](https://docs.aws.amazon.com/launchwizard/latest/userguide/launch-wizard-sap-tutorials.html)
- [AWS blog — Automated S/4HANA Fully-Activated Appliance deployment](https://aws.amazon.com/blogs/awsforsap/automated-sap-s4hana-fully-activated-appliance-deployment/)
- [AWS blog — Automate SAP deployments with Launch Wizard APIs](https://aws.amazon.com/blogs/awsforsap/automate-sap-deployments-with-aws-launch-wizard-for-sap-apis/)

### B. Fastest path to an ABAP system you can actually code in

Skip hyperscaler-admin labs and jump straight into ABAP.

1. **SAP Cloud Appliance Library (CAL)** — one-click deploy of S/4HANA / ABAP Platform trial into your own AWS / Azure / GCP account. SAP license free for trial window; you only pay IaaS.
   - [SAP CAL portal](https://cal.sap.com/)
   - [SAP CAL community hub](https://pages.community.sap.com/topics/cloud-appliance-library)
   - [S/4HANA Fully-Activated Appliance blog (SAP)](https://community.sap.com/t5/enterprise-resource-planning-blog-posts-by-sap/sap-s-4hana-fully-activated-appliance-create-your-sap-s-4hana-system-in-a/ba-p/13365960)
   - [ABAP Platform 2023 Developer Edition on CAL](https://community.sap.com/t5/technology-blog-posts-by-sap/abap-platform-2023-developer-edition-on-cloud-appliance-library-cal-now/ba-p/14185861)
2. **SAP Developer Trials hub**
   - <https://developers.sap.com/trials-downloads.html>
3. **BTP ABAP Environment (Steampunk) free tier** — cloud-only, no IaaS to manage; pure ABAP Cloud / RAP.
   - [Mission: Get Started with SAP S/4HANA Cloud, ABAP environment](https://discovery-center.cloud.sap/missiondetail/4036/4243/)

### C. Integration-focused hands-on (Azure ↔ SAP ABAP)

- [SAP Community — Kick-start SAP ABAP Platform integration with Microsoft](https://community.sap.com/t5/enterprise-resource-planning-blog-posts-by-members/kick-start-your-sap-abap-platform-integration-journey-with-microsoft/ba-p/13551984) — reproducible with BTP free tier + Azure trial + M365 sandbox.

---

## 8. Recommended study order

1. Install **ADT** (Eclipse) → §5.
2. **ABAP language basics** via keyword docs + cheat sheets → §1.
3. **CDS & VDM** (the DB bridge) → §2.
4. **RAP** (business logic on top of CDS) → §3.
5. **Clean Core** rules as a filter over everything you write → §4.
6. Stand up a real system via **CAL on AWS** (easiest) or **BTP ABAP Environment** (simplest) → §7.B.

## 9. Recommendation for hands-on

- Pure ABAP study → **§7.B.1 CAL on AWS** (best docs, flexible EC2 sizing, S/4HANA Fully-Activated Appliance is well-maintained).
- Also want Basis/infra skills → add **§7.A** Azure ACSS or AWS Launch Wizard labs.
- Want simplest cloud-only ABAP → **§7.B.3 BTP ABAP Environment** (accept the ABAP Cloud language subset).
