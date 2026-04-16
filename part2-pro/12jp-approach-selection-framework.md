# 12 - アプローチ選定フレームワーク

## 採用率（業界参考値）

| アプローチ | 採用率 |
|---|---|
| ブラウンフィールド（システムコンバージョン） | 約44% |
| シェルコンバージョン（セレクティブデータトランジション） | 約42% |
| グリーンフィールド（新規インプリメンテーション） | 約14% |

## 判断マトリクス

| 判断軸 | ブラウンフィールド | シェルコンバージョン | グリーンフィールド |
|---|---|---|---|
| **期間** | 最短 | 中程度 | 最長 |
| **コスト** | 低い | 中程度 | 高い |
| **プロセス再設計** | 最小限 | 部分的 | 全面的 |
| **カスタムコード継承** | あり（改修必要） | 一部 | なし（再作成または廃棄） |
| **履歴データ** | 全件保持 | 選択的に移行 | ゼロスタート |
| **チェンジマネジメント** | 低い | 中程度 | 高い |
| **データ品質リスク** | 中程度 | クレンジング必要 | コントロール可能 |
| **ダウンタイム** | 標準 / 最適化可能 | ニアゼロ可能 | 該当なし（新規システム） |

## アプローチ別の適用条件

### ブラウンフィールド
- 既存の設定・プロセス・データを保持したい
- スケジュールが最優先
- システムが比較的クリーンで最新の状態
- リスク許容度が低い

### グリーンフィールド
- 業務プロセスをゼロから再設計したい
- 現ECCが過度にカスタマイズされており保守が限界
- 大規模な組織再編（M&A、カーブアウト等）が進行中
- 長期的な標準化とTCO削減が優先目標

### シェルコンバージョン
- データのクレンジングや不要な会社コード・履歴の整理が必要
- 会社コード単位での段階的な本番稼働が必要
- ニアゼロダウンタイムが必須要件
- 一部プロセスは再設計、他は継承というハイブリッドアプローチ

## クライアントへの確認事項

1. **現在のSAP設定・プロセスはどの程度まだ有効ですか？**
   → 高い → ブラウンフィールドまたはシェル；低い → グリーンフィールド

2. **新システムにすべての履歴トランザクションデータが必要ですか？**
   → 必要 → ブラウンフィールド；選択的 → シェル；不要 → グリーンフィールド

3. **本番稼働のハードデッドラインはありますか？**
   → あり → ブラウンフィールド（最速）；柔軟 → いずれでも可

4. **カスタムABAPコードの量と品質は？**
   → 少ない / 整備済み → ブラウンフィールド；多い / 老朽化 → グリーンフィールドまたはシェル

5. **会社コード単位の段階的展開が必要ですか？**
   → 必要 → シェルコンバージョン

## 参考資料

- SAP セレクティブデータトランジション概要（SAP公式）: https://support.sap.com/en/offerings-programs/support-services/data-management-landscape-transformation/selective-data-transition-engagement.html
- S/4HANAへのSDTによる移行（SAP公式ブログ）: https://community.sap.com/t5/enterprise-resource-planning-blog-posts-by-sap/move-to-sap-s-4hana-with-selective-data-transition/ba-p/13455833
- グリーンフィールド新規導入オプション（SAP Learning、公式）: https://learning.sap.com/learning-journeys/discovering-sap-activate-implementation-tools-and-methodology/analyzing-options-for-greenfield-new-implementation-on-sap-s-4hana-cloud-sap-s-4hana-on-premise-and-sap-s-4hana-cloud-private-edition
- セレクティブデータトランジションの解説（SAP Learning、公式）: https://learning.sap.com/learning-journeys/discovering-sap-activate-implementation-tools-and-methodology/illustrating-selective-data-transition
