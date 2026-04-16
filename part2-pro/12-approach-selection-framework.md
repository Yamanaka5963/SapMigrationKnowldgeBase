# 12 - Approach Selection Decision Framework

## Adoption Rates (Industry Reference)

| Approach | Adoption Rate |
|---|---|
| Brownfield (System Conversion) | ~44% |
| Shell Conversion (Selective Data Transition) | ~42% |
| Greenfield (New Implementation) | ~14% |

## Decision Matrix

| Factor | Brownfield | Shell Conversion | Greenfield |
|---|---|---|---|
| **Timeline** | Shortest | Medium | Longest |
| **Cost** | Lower | Medium | Higher |
| **Process redesign** | Minimal | Selective | Full |
| **Custom code retained** | Yes (requires remediation) | Partially | No (rewrite or drop) |
| **Historical data** | All preserved | Selectively migrated | Fresh start |
| **Change management** | Low | Medium | High |
| **Data quality risk** | Medium | Requires cleansing | Controlled |
| **Downtime** | Standard / optimizable | Near-zero possible | N/A (new system) |

## When to Choose Each

### Brownfield
- Client wants to preserve existing config, processes, and data
- Timeline is critical
- System is relatively clean and current
- Risk appetite is low

### Greenfield
- Client wants to redesign business processes from scratch
- Current ECC system is heavily outdated or custom-coded beyond maintainability
- Client is undergoing major restructuring (M&A, carve-out, etc.)
- Long-term standardization and TCO reduction is priority

### Shell Conversion (Selective Data Transition)
- Client wants to clean up data or remove obsolete company codes/history
- Phased go-live by company code is required
- Near-zero downtime is a hard requirement
- Hybrid: some processes redesigned, others retained

## Key Questions to Ask the Client

1. **How much of your current SAP configuration and processes are still valid?**
   → High validity → Brownfield or Shell; Low validity → Greenfield

2. **Do you need all historical transaction data in the new system?**
   → Yes → Brownfield; Selectively → Shell; No → Greenfield

3. **Is there a hard go-live deadline?**
   → Hard deadline → Brownfield (fastest); Flexible → any

4. **What is the volume and quality of custom ABAP code?**
   → Low / well-maintained → Brownfield; High / outdated → Greenfield or Shell

5. **Is a phased rollout (by company code) needed?**
   → Yes → Shell Conversion

## References

- SAP Selective Data Transition overview (official SAP): https://support.sap.com/en/offerings-programs/support-services/data-management-landscape-transformation/selective-data-transition-engagement.html
- Move to S/4HANA with Selective Data Transition (by SAP): https://community.sap.com/t5/enterprise-resource-planning-blog-posts-by-sap/move-to-sap-s-4hana-with-selective-data-transition/ba-p/13455833
- Greenfield new implementation options (SAP Learning, official): https://learning.sap.com/learning-journeys/discovering-sap-activate-implementation-tools-and-methodology/analyzing-options-for-greenfield-new-implementation-on-sap-s-4hana-cloud-sap-s-4hana-on-premise-and-sap-s-4hana-cloud-private-edition
- Illustrating Selective Data Transition (SAP Learning, official): https://learning.sap.com/learning-journeys/discovering-sap-activate-implementation-tools-and-methodology/illustrating-selective-data-transition
- SAP Community — choosing the right S/4HANA path (member post): https://community.sap.com/t5/financial-management-blog-posts-by-members/choosing-the-right-path-for-your-s-4hana-transformation-greenfield/ba-p/13877169
