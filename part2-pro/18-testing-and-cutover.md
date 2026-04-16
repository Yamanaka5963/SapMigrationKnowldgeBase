# 18 - Testing Strategy & Cutover Plan

## Testing Strategy

### Testing Levels

| Level | Description | Who |
|---|---|---|
| **Unit Testing** | Individual config items, custom programs | Consultants / developers |
| **Integration Testing** | End-to-end process flows across modules | Consultants + key users |
| **UAT (User Acceptance Testing)** | Business validation of processes and data | Client business users |
| **Performance / Load Testing** | System response under expected load | Basis + technical team |
| **Regression Testing** | Verify existing functionality not broken post-conversion | Consultants + key users |
| **Cutover Rehearsal (Mock Cutover)** | Full dry run of cutover procedure in QAS | All tracks |

### UAT Acceptance Criteria (Go/No-Go)
Before go-live, confirm:
- [ ] Agreed percentage of UAT test cases passed (define threshold with client, e.g., 95%)
- [ ] No open critical (Priority 1) issues
- [ ] End-user training completed for defined user population
- [ ] Master and open transactional data validated and signed off
- [ ] User and authorization management confirmed operational
- [ ] All interfaces tested and confirmed operational
- [ ] Infrastructure and system integration channels ready

## Cutover Planning

### Key Principles
- Cutover plan must be ready **months before go-live** — not weeks
- Identify all tracks: Infrastructure, Deployment, Technical (Basis), Functional, Development, Authorizations, Data Migration
- Every task in the cutover plan must have: owner, duration, dependency, and contingency

### Cutover Plan Structure

| Track | Key Tasks |
|---|---|
| **Technical (Basis)** | System refresh, transport imports, kernel updates, post-conversion steps |
| **Data Migration** | Final data extracts from legacy, load to production (master data → open items → balances) |
| **Functional** | Configuration transports, post-import manual settings, smoke tests per module |
| **Authorizations** | Role assignments, user creation, authorization testing |
| **Interfaces** | Activate and test all inbound/outbound interfaces |
| **Cutover Sign-Off** | Go/no-go meeting with client project sponsor |

### Mock Cutover Runs
- Conduct **minimum 2 mock cutovers** in QAS — with the same rigor as production
- First mock: identify gaps in plan, timing issues, missing steps
- Second mock: validate fixes, finalize timing and ownership
- Document actual durations — use as basis for production downtime window

### Go-Live Day Sequence
1. Announce legacy system freeze (read-only)
2. Final data extracts from legacy
3. Transport imports to production
4. Data migration loads (master data → open items → balances)
5. Interface activation
6. Authorization assignments
7. Smoke tests per module
8. Go/no-go decision with client sponsor
9. Open system to users
10. Hypercare begins — see [19 - Hypercare & Handoff](19-hypercare-handoff.md)

### Rollback Plan
Always define a rollback trigger and procedure before go-live:
- What is the latest point at which rollback is still possible?
- Who has authority to make the rollback decision?
- What steps are required to restore the legacy system to operational state?

## References

- SAP Learning — Preparing for Cutover (official): https://learning.sap.com/courses/cloud-onboarding-for-sap-cloud-erp/preparing-for-cutover
- SAP Community — Cutover management: planning and orchestration (community): https://community.sap.com/t5/enterprise-architecture-blog-posts/cutover-management-planning-and-orchestration/ba-p/13693946
- SAP Community — Best practices for S/4HANA cutover planning (Q&A): https://community.sap.com/t5/enterprise-resource-planning-q-a/navigating-the-final-mile-best-practices-for-sap-s-4hana-cutover-planning/qaq-p/14166257
