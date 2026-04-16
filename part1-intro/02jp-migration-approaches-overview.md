# 02 - 移行アプローチ概要

SAP ECC から S/4HANA への移行には、3つの標準アプローチがあります。

| アプローチ | 別称 | 概要 | 適したケース |
|---|---|---|---|
| **ブラウンフィールド** | システムコンバージョン | 既存ECCをそのまま変換。設定・カスタマイズ・データを保持。 | 手戻りを最小化したいクライアント |
| **グリーンフィールド** | 新規インプリメンテーション | S/4HANAをゼロから構築。業務プロセスを再設計。 | 旧来のプロセスを刷新したいクライアント、または大規模な組織再編がある場合 |
| **シェルコンバージョン** | セレクティブデータトランジション | 組織構造・設定のみ移行し、データは選択的に移行。 | データの選択的移行や部分的なカットオーバーが必要なクライアント |

> アプローチ選定の詳細な判断基準については、[パート2 — 12: アプローチ選定フレームワーク](../part2-pro/12-approach-selection-framework.md)（英語）を参照してください。

## 参考資料

- SAP Community — 3アプローチ比較（メンバー投稿）: https://community.sap.com/t5/enterprise-resource-planning-blog-posts-by-members/sap-s-4hana-migration-strategies-greenfield-brownfield-hybrid-comparison/ba-p/13451694
- SAP Community — ブラウンフィールド vs グリーンフィールド戦略ガイド（メンバー投稿）: https://community.sap.com/t5/enterprise-resource-planning-blog-posts-by-members/choosing-between-brownfield-and-greenfield-a-strategic-guide-for-sap-s/ba-p/13914233
- SAP Learning — 変換アプローチ概要（公式）: https://learning.sap.com/courses/sap-s-4hana-conversion-and-sap-system-upgrade
- SAP S/4HANA 変換ガイド 2025（公式PDF、`help.sap.com/docs/SAP_S4HANA_ON-PREMISE` 経由）: https://help.sap.com/doc/8cec006e962a46049506bc10ed64f557/2025/en-US/CONV_PE2025.pdf
