# 14 - グリーンフィールド実行ガイド（新規インプリメンテーション）

## 概要

グリーンフィールドは、S/4HANAの新規インプリメンテーションです。システム変換は行いません。既存のECCシステムはカットオーバーまで並行稼働し続けます。業務プロセスはSAPベストプラクティス（フィット・トゥ・スタンダード）に基づいて再設計されます。履歴データはMigration Cockpitを使用して選択的に移行されます。

グリーンフィールドは採用率が最も低いアプローチ（約14%）ですが、現ECCが過度にカスタマイズされているか老朽化している場合に最もクリーンな結果をもたらします。

## 主な特徴

- 既存の設定・カスタムコード・履歴データは自動的には引き継がれない
- SAP標準（S/4HANAベストプラクティスコンテンツ）を使用してプロセスを再設計
- データ移行は必要なマスターデータと未処理/関連トランザクションデータのみにスコープを絞る
- 初期工数は多いが長期的なTCOは低い

## 実行フェーズ（SAP Activate）

### フェーズ1 — Discover & Prepare（発見・準備）
- [ ] グリーンフィールド採用を確定 → [12 - アプローチ選定](12jp-approach-selection-framework.md)を参照
- [ ] プロジェクトガバナンス、チーム体制、RACI設定
- [ ] S/4HANA DEV/QAS/PRDランドスケープの構築
- [ ] SAP Best Practiceコンテンツのシステムへの有効化
- [ ] SAP Cloud ALMまたはSolution Managerのプロジェクト管理設定
- [ ] データ移行スコープの定義（対象マスターデータ、未処理明細）

### フェーズ2 — Explore（フィット・トゥ・スタンダードワークショップ）
- [ ] モジュール別フィット・トゥ・スタンダードワークショップの実施（FI、CO、MM、SD、PP等）
- [ ] ビジネスステークホルダーへのSAP標準プロセスのデモ
- [ ] ギャップの文書化 — 設定・エンハンスメント・プロセス変更のいずれかに分類
- [ ] 組織構造の確定（会社コード、プラント、販売組織等）
- [ ] データ移行オブジェクトとレガシーデータマッピングの確認

> カスタマイズを最小限に抑えること。カスタム開発はすべてアップグレードリスクと保守負荷を増加させます。プロセス変更で解決できるギャップへの追加開発はできる限り避けるよう働きかけてください。

### フェーズ3 — Realize（構築・テスト）
- [ ] ベースラインシステムの設定（組織構造、マスターデータ設定）
- [ ] 承認されたギャップの開発（カスタムレポート、帳票、インターフェース）
- [ ] 外部システムとのインターフェースの構築とテスト
- [ ] データ移行テンプレートの準備（Migration Cockpit）→ [17 - データ移行](17jp-data-migration-cockpit.md)を参照
- [ ] 単体テスト → 統合テスト → UATの実施
- [ ] SAP Activateアジャイルアプローチによるスプリントでの反復

### フェーズ4 — Deploy（カットオーバー）
- [ ] モックカットオーバーの実施（最低2回）
- [ ] 本番システムへのマスターデータロード
- [ ] 未処理明細 / 期首残高のロード
- [ ] Go/No-Goチェックリストの実行
- [ ] 本番稼働
- [ ] カットオーバー詳細は[18 - テスト・カットオーバー](18jp-testing-and-cutover.md)を参照

### フェーズ5 — Run（ハイパーケア）
- [19 - ハイパーケア・引き渡し](19jp-hypercare-handoff.md)を参照

## 期間の目安

| プロジェクト規模 | 標準期間 |
|---|---|
| 小規模（会社コード1〜2、限定スコープ） | 6〜12ヶ月 |
| 中規模（多国籍、マルチモジュール） | 12〜18ヶ月 |
| 大規模 / 複雑 | 18〜36ヶ月 |

> これらはあくまで目安です。スコープ、チーム規模、意思決定スピードに基づいてクライアントと検証してください。

## 参考資料

- SAP Learning — グリーンフィールド新規導入オプション（公式）: https://learning.sap.com/learning-journeys/discovering-sap-activate-implementation-tools-and-methodology/analyzing-options-for-greenfield-new-implementation-on-sap-s-4hana-cloud-sap-s-4hana-on-premise-and-sap-s-4hana-cloud-private-edition
- SAP Community — Activate方法論によるグリーンフィールド（メンバー投稿）: https://community.sap.com/t5/enterprise-resource-planning-blog-posts-by-members/s4hana-green-field-implementation-activate-methodology-deliverables-3/ba-p/14010838
- SAP Activateフェーズ概要（公式）: https://learning.sap.com/courses/discovering-sap-activate-implementation-tools-and-methodology/analyzing-each-phase-of-sap-activate
