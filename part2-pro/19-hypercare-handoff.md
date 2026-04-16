# 19 - Hypercare & Project Handoff

## Hypercare Overview

Hypercare is the intensive support period immediately after go-live. The project team remains on-site or on-call to resolve issues quickly and stabilize the new system before handing off to the client's operations team.

Typical hypercare duration: **2–4 weeks** (align with client at project kick-off — longer for complex or large-scope projects).

## Hypercare Setup

Define the following before go-live:

- [ ] Hypercare duration and end date agreed with client
- [ ] Support team roster and shift schedule (on-site vs. remote)
- [ ] Issue severity classification and SLA targets per level
- [ ] Escalation path (consultant → lead → project manager → SAP support)
- [ ] Issue tracking tool (Jira, ServiceNow, Solution Manager, etc.)
- [ ] Daily stand-up or war room schedule during first week

## Issue Severity Classification (Example)

| Severity | Definition | Response Target |
|---|---|---|
| **P1 — Critical** | System down or core business process fully blocked | Immediate (< 1 hour) |
| **P2 — High** | Major process impacted, workaround available | < 4 hours |
| **P3 — Medium** | Minor process issue or cosmetic defect | Next business day |
| **P4 — Low** | Enhancement request or non-urgent question | Backlog / next release |

## Day-1 Activities

- [ ] Monitor system performance and batch jobs
- [ ] Support users in first business transactions
- [ ] Confirm all interfaces are running (check IDocs, batch jobs, middleware)
- [ ] Review any failed postings or error logs from overnight jobs
- [ ] Quick smoke test of critical business processes (order-to-cash, procure-to-pay, etc.)

## Handoff Checklist

Before closing the project and handing off to client operations:

### Documentation
- [ ] System configuration documentation delivered (transport log, config decisions)
- [ ] Custom development documentation delivered (functional specs, technical specs)
- [ ] Operations runbook delivered (batch job schedule, monitoring guide, key transactions)
- [ ] Interface documentation delivered

### Knowledge Transfer
- [ ] Basis/IT team trained on system administration (transports, monitoring, patching)
- [ ] Key users trained and signed off
- [ ] Super users identified and trained for L1 support
- [ ] Client team can independently manage user and authorization changes

### Open Items
- [ ] All P1/P2 issues resolved or accepted by client with workaround
- [ ] Open P3/P4 backlog reviewed and handed to client
- [ ] SAP support contracts active and S-users assigned for client team
- [ ] SAP Early Watch Alert scheduled (system health monitoring)

### Formal Acceptance
- [ ] Project acceptance document signed by client project sponsor
- [ ] Lessons learned session completed
- [ ] Final project report delivered

## Post-Hypercare: Operations Transition

After hypercare, client's internal IT or AMS (Application Management Services) partner takes over. Ensure:
- Client has access to all transport routes in production
- Client knows how to raise SAP OSS messages
- Monitoring and alerting configured in SAP Solution Manager or SAP Cloud ALM
