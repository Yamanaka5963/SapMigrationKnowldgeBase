# 15 - シェルコンバージョンガイド（セレクティブデータトランジション）

## 概要

シェルコンバージョン（セレクティブデータトランジション / SDTとも呼ばれる）はハイブリッドアプローチです。既存ECCシステムのコピー（「シェル」）を作成し、マスターデータとトランザクションデータを除いた設定・リポジトリオブジェクトを保持した状態でS/4HANAへ変換し、必要なデータのみを選択的に移行します。

ブラウンフィールドのスピード（既存設定の再利用）とグリーンフィールドのデータ選択性（移行データを選択）を組み合わせたアプローチです。

**業界採用率：約42%** — ブラウンフィールドと並び最も多いアプローチです。

## 2つのバリアント

| バリアント | 説明 |
|---|---|
| **シェルコンバージョン** | レガシーカスタマイズとインターフェースをすべて継承し、データのみ選択的に移行 |
| **ミックス&マッチ** | 一部の業務プロセスを再設計しながら、他のプロセスと設定は継承 |

## 適用条件

- 不要な会社コード、履歴データ、非アクティブな組織単位を削除したい
- 会社コード単位での段階的な本番稼働が必要
- ニアゼロダウンタイムが必須要件
- データ品質が低く、移行をデータクレンジングの機会として活用したい
- 一部プロセスは再設計したいが、すべてではない

## プロセスの概要

### ステップ1 — シェルコピーの作成
- ECCシステムの1対1コピーを作成
- マスターデータとトランザクションデータをコピーから除去
- 保持するもの：カスタマイズ、設定、リポジトリオブジェクト（ABAP、ディクショナリ）
- 使用ツール：SAP Business Transformation Center（BTC）またはパートナーツール（SNP、Syniti等）

### ステップ2 — シェルシステムの変換
- 設定のみのシェルコピーをSUM/DMOでS/4HANAへ変換
- 標準のブラウンフィールド変換ステップに従う → [13 - ブラウンフィールド実行](13jp-brownfield-execution.md)を参照
- SPDD/SPAUの調整も同様に必要
- カスタムコード改修も引き続き必要 → [16 - カスタムコード改修](16jp-custom-code-remediation.md)を参照

### ステップ3 — データスコープの定義
SAP Business Transformation Center（BTC）を使用して移行データを定義：
- どの会社コード
- どの会計年度 / 期間
- どのマスターデータオブジェクト
- どの未処理トランザクションデータ

### ステップ4 — データ移行
- レガシーECCから変換済みS/4HANAシェルへ選択したデータを移行
- 大容量データはデータベースレベルの移行（ファイルベースより高速）
- データクレンジングと統合はこのステップの前または最中に実施

### ステップ5 — カットオーバー
複数のカットオーバー戦略をサポート：
- **ビッグバン** — 全会社コードを一括移行
- **会社コード段階的** — 会社コード単位で順次本番稼働
- **ニアゼロダウンタイム** — 同期が完了するまで並行稼働

> データクレンジングは必須です。データを移行しても、S/4HANAに適したデータになるとは限りません。プロジェクトスコープに必ずデータクレンジングとガバナンス計画を含めてください。

## 主要ツール

| ツール | 役割 |
|---|---|
| **SAP Business Transformation Center（BTC）** | SAPネイティブのデータ準備分析とSDTオーケストレーションツール |
| **SNP Transformation Backbone** | 高速セレクティブ移行向けの一般的なパートナーツール |
| **Syniti Data Migration** | データ移行と品質管理向けのパートナーツール |

## 参考資料

- SAP Support — セレクティブデータトランジションエンゲージメント（公式）: https://support.sap.com/en/offerings-programs/support-services/data-management-landscape-transformation/selective-data-transition-engagement.html
- SAP Community — SDTによるS/4HANA移行（SAP公式）: https://community.sap.com/t5/enterprise-resource-planning-blog-posts-by-sap/move-to-sap-s-4hana-with-selective-data-transition/ba-p/13455833
- SAP Community — SAP BTCを使用したSDT（SAP公式）: https://community.sap.com/t5/technology-blog-posts-by-sap/selective-data-transition-using-sap-business-transformation-center-btc/ba-p/14165170
- SAP Learning — セレクティブデータトランジションの解説（公式）: https://learning.sap.com/learning-journeys/discovering-sap-activate-implementation-tools-and-methodology/illustrating-selective-data-transition
