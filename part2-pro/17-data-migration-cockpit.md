# 17 - Data Migration (Migration Cockpit)

## Overview

The SAP S/4HANA Migration Cockpit is SAP's standard tool for migrating business data (master data and transactional data) into S/4HANA. It uses predefined migration objects with built-in validation, mapping, and simulation. No additional licensing cost — included with S/4HANA.

Used primarily in Greenfield and Shell Conversion projects. In Brownfield, the existing data is carried over by SUM/DMO, but the Cockpit may still be used for initial data loads (e.g., cutover open items).

## Migration Approaches

| Approach | Description | Best For |
|---|---|---|
| **File-Based (Local Templates)** | Download XML/CSV template, populate manually, upload | Small to medium volumes; simple setup |
| **Staging Tables** | Populate intermediate HANA DB tables via ETL or SAP Data Services | High volume; complex data transformations |
| **Direct Transfer (RFC)** | RFC connection between source SAP system and target S/4HANA; auto-converts data model | SAP-to-SAP migrations; fastest for compatible systems |

## Workflow (File-Based)

1. **Select migration object** — e.g., Customer Master, Open Purchase Orders, G/L Account Balances
2. **Download template** — XML or CSV with predefined field structure
3. **Populate template** — map and fill from legacy data extract
4. **Upload template** — import into Migration Cockpit
5. **Simulate** — validation run without posting; review error log
6. **Fix errors** — correct data in source template, re-upload
7. **Migrate** — execute final data load to production

## Key Migration Objects (Examples)

| Category | Objects |
|---|---|
| **Financial** | G/L Accounts, Cost Centers, Profit Centers, Open Items (AR/AP), Asset Master |
| **Procurement** | Vendor Master, Purchase Info Records, Open Purchase Orders |
| **Sales** | Customer Master, Sales Orders (open), Pricing Conditions |
| **Logistics** | Material Master, Storage Locations, Batch Master, Inventory Balances |
| **HR** | Organizational Units, Positions (if S/4HANA HCM in scope) |

## Business Partner (CVI) — Mandatory

In S/4HANA, Customers and Vendors must be converted to **Business Partners (BP)**. This is mandatory and must be addressed in every migration — even Brownfield.

- Run CVI (Customer-Vendor Integration) synchronization
- Ensure no duplicate BP records
- Address any CVI errors found in the Readiness Check before go-live

## Best Practices

- Run simulation multiple times until error count reaches zero before final migration
- Stage data loads: master data first, then open transactional data
- Sequence matters: create referenced objects before dependent objects (e.g., Cost Centers before G/L Accounts that reference them)
- Keep the legacy system in read-only mode once final data extract is taken for cutover load

## References

- SAP Help Portal — Migration Cockpit (official): https://help.sap.com/docs/SAP_S4HANA_CLOUD/d5699934e7004d048c4801b552f3b013/121b34742a904d10bca907bbf2fd5617.html
- SAP Community topic page — Migration Cockpit: https://pages.community.sap.com/topics/s4hana-migration-cockpit
- SAP Learning — Introducing the Migration Cockpit (official): https://learning.sap.com/courses/implementing-sap-s-4hana-cloud-public-edition/introducing-the-sap-s-4hana-migration-cockpit
- SAP Community — Step-by-step Migration Cockpit guide (member post): https://community.sap.com/t5/enterprise-resource-planning-blog-posts-by-members/step-by-step-guide-to-ecc-to-s-4hana-migration-with-migration-cockpit/ba-p/13983763
- SAP Community — Comprehensive data migration guide (member post): https://community.sap.com/t5/technology-blog-posts-by-members/a-comprehensive-guide-to-data-migration-with-the-sap-s-4hana-migration/ba-p/14224541
