# 14 - Greenfield Execution Guide (New Implementation)

## Overview

Greenfield is a fresh S/4HANA implementation — no system conversion. The existing ECC system continues running in parallel until cutover. Business processes are redesigned using SAP best practices (fit-to-standard). Historical data is selectively migrated via the Migration Cockpit.

Greenfield is the least common approach (~14% adoption) but delivers the cleanest outcome when current ECC is too customized or outdated to carry forward.

## Key Characteristics

- No existing configuration, custom code, or history carried over automatically
- Processes redesigned using SAP standard (S/4HANA best practice content)
- Data migration scoped to required master data and open/relevant transactional data only
- Higher upfront effort; lower long-term TCO

## Execution Phases (SAP Activate)

### Phase 1 — Discover & Prepare
- [ ] Confirm Greenfield decision (see [12 - Approach Selection](12-approach-selection-framework.md))
- [ ] Set up project governance, team structure, RACI
- [ ] Stand up DEV/QAS/PRD S/4HANA landscape
- [ ] Activate SAP Best Practice content in the system
- [ ] Set up SAP Cloud ALM or Solution Manager for project tracking
- [ ] Define data migration scope (which master data, which open items)

### Phase 2 — Explore (Fit-to-Standard Workshops)
- [ ] Run fit-to-standard workshops per module (FI, CO, MM, SD, PP, etc.)
- [ ] Demonstrate SAP standard processes to business stakeholders
- [ ] Document gaps — classify as: configuration, enhancement, or process change
- [ ] Define organizational structure (company codes, plants, sales orgs, etc.)
- [ ] Confirm data migration objects and legacy data mapping

> Minimize customization. Every custom development adds upgrade risk and maintenance burden. Push back on gaps that can be solved by process change.

### Phase 3 — Realize (Build & Test)
- [ ] Configure baseline system (organizational structure, master data settings)
- [ ] Develop approved gaps (custom reports, forms, interfaces)
- [ ] Build and test interfaces to external systems
- [ ] Prepare data migration templates (Migration Cockpit) — see [17 - Data Migration Cockpit](17-data-migration-cockpit.md)
- [ ] Execute unit testing → integration testing → UAT
- [ ] Iterate in sprints per SAP Activate agile approach

### Phase 4 — Deploy (Cutover)
- [ ] Execute mock cutover runs (minimum 2)
- [ ] Load master data to production
- [ ] Load open items / initial balances
- [ ] Execute go/no-go checklist
- [ ] Go-live
- [ ] See [18 - Testing & Cutover](18-testing-and-cutover.md) for full cutover details

### Phase 5 — Run (Hypercare)
- See [19 - Hypercare & Handoff](19-hypercare-handoff.md)

## Timeline Reference

| Project Size | Typical Duration |
|---|---|
| Small (1-2 company codes, limited scope) | 6–12 months |
| Medium (multi-country, multi-module) | 12–18 months |
| Large / complex | 18–36 months |

> These are rough benchmarks. Always validate with client based on scope, team size, and decision-making speed.

## References

- SAP Learning — Greenfield new implementation options (official): https://learning.sap.com/learning-journeys/discovering-sap-activate-implementation-tools-and-methodology/analyzing-options-for-greenfield-new-implementation-on-sap-s-4hana-cloud-sap-s-4hana-on-premise-and-sap-s-4hana-cloud-private-edition
- SAP Community — Greenfield with Activate methodology (member post): https://community.sap.com/t5/enterprise-resource-planning-blog-posts-by-members/s4hana-green-field-implementation-activate-methodology-deliverables-3/ba-p/14010838
- SAP Activate phases overview (official): https://learning.sap.com/courses/discovering-sap-activate-implementation-tools-and-methodology/analyzing-each-phase-of-sap-activate
