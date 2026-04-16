# 02 - Migration Approaches Overview

There are three standard approaches for migrating from SAP ECC to S/4HANA:

| Approach | Also Called | Description | Best For |
|---|---|---|---|
| **Brownfield** | System Conversion | Convert existing ECC in-place. Config, customizations & data preserved. | Clients wanting to minimize rework and go-live risk |
| **Greenfield** | New Implementation | Fresh S/4HANA install. Clean slate, redesign processes. | Clients with outdated processes or undergoing major reorg |
| **Shell Conversion** | Selective Data Transition | Move org structure/config only, selectively migrate data | Clients needing data selectivity or partial cutover |

> For the decision framework on choosing between approaches, see [Part 2 — 12: Approach Selection Framework](../part2-pro/12-approach-selection-framework.md).

## References

- SAP Community — all 3 approaches compared (member post): https://community.sap.com/t5/enterprise-resource-planning-blog-posts-by-members/sap-s-4hana-migration-strategies-greenfield-brownfield-hybrid-comparison/ba-p/13451694
- SAP Community — Brownfield vs Greenfield strategic guide (member post): https://community.sap.com/t5/enterprise-resource-planning-blog-posts-by-members/choosing-between-brownfield-and-greenfield-a-strategic-guide-for-sap-s/ba-p/13914233
- SAP Learning — conversion approaches overview (official): https://learning.sap.com/courses/sap-s-4hana-conversion-and-sap-system-upgrade
- SAP S/4HANA Conversion Guide 2025 (official PDF, found via `help.sap.com/docs/SAP_S4HANA_ON-PREMISE`): https://help.sap.com/doc/8cec006e962a46049506bc10ed64f557/2025/en-US/CONV_PE2025.pdf
