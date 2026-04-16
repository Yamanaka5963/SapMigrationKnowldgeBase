# 16 - カスタムコード改修

## 概要

カスタムABAPコード（Zプログラム、Zテーブル、エンハンスメント、モディフィケーション）は、システム変換の前後にS/4HANA互換性のための分析と改修が必要です。これはブラウンフィールドおよびシェルコンバージョンプロジェクトにおいて最も工数のかかるワークストリームの一つです。

## 使用ツール

| ツール | 目的 |
|---|---|
| **ABAP Test Cockpit（ATC）** | 主要ツール — カスタムコードにS/4HANA固有チェックを実行し、検出結果一覧を生成 |
| **Custom Code Migration App**（Fiori） | カスタムコード分析ワークフローの管理と改修進捗のトラッキング |
| **ABAP Development Tools（ADT）in Eclipse** | クイックフィックス候補を提示 — 典型的な問題の約40〜60%を自動修正可能 |
| **SAP Code Inspector（SCI）** | FUNCTIONAL_DBチェックバリアントによる補完的な静的分析 |
| **SAP Joule for Developers** | AIを活用したカスタムコード移行（新ツール、プロジェクト期間を短縮） |

## 改修プロセス

### ステップ1 — 変換前の分析
- DEVシステムでS/4HANAチェックバリアントを使用してATCを実行
- 検出結果のレビュー：重要度と量でカテゴリ分類
- 未使用のカスタムオブジェクトを特定 → 改修前に廃止（工数削減）
- カテゴリ別の改修工数を見積もり

### ステップ2 — 未使用コードの廃止
- 使用されていないZプログラム/エンハンスメントを削除または廃止
- 使用統計（トランザクションSUSG等）で未使用オブジェクトを特定
- コードが少ない = 改修が少ない = リグレッションテストが少ない

### ステップ3 — 改修（変換前後）
- ADT in Eclipseでクイックフィックスを適用（問題の約40〜60%をカバー）
- 残存する検出結果を手動修正 — 各簡素化項目の関連SAPノートを参照
- 主な対応が必要な問題：
  - **旧DBからのNative SQL** — 削除必須
  - **ORDER BY なしの SELECT** — HANA上では並び順が保証されない
  - **プール/クラスターテーブルへのアクセス** — S/4HANAでは透過テーブルに変換済み
  - **削除/変更されたSAP標準オブジェクト** — 各簡素化項目ノートを確認
  - **暗黙的なORDER BY PRIMARY KEY** — 透過テーブルでは動作が変更されている

### ステップ4 — SPDD / SPAU（変換ダウンタイム中）
- **SPDD** — データディクショナリ修正の再調整（SUMダウンタイムの初期フェーズで実施）
- **SPAU** — リポジトリオブジェクト修正の再調整（SUMプロセスの終盤で実施）
- **SPAU_ENH** — エンハンスメントベースの修正に使用
- ダウンタイム開始前に調整トランスポート要求を準備しておくこと

### ステップ5 — 変換後の検証
- 変換済みシステムで再度ATCを実行
- 残存する検出結果を修正
- カスタムオブジェクトの機能テストを実施

## 主な簡素化項目カテゴリ

S/4HANAには**430以上の簡素化項目**があります。優先的に対応すべき重要カテゴリ：
- FI-CO統合変更（利益センター、原価要素がG/Lに統合）
- マテリアルレジャー — 強制有効化
- ビジネスパートナー（CVI） — 顧客・仕入先のBP化が必須
- 在庫管理テーブル（MATDOCがMSEG + MKPFを代替）
- HR構造変更（HCMがスコープに含まれる場合）

## 参考資料

- カスタムコード移行ガイド S/4HANA 2025（公式PDF）: https://help.sap.com/doc/9dcbc5e47ba54a5cbb509afaa49dd5a1/2025.001/en-US/CustomCodeMigration_EndtoEnd.pdf
- SAP Community — ABAPカスタムコード移行の始め方（SAP公式）: https://community.sap.com/t5/technology-blog-posts-by-sap/get-started-with-the-abap-custom-code-migration-process/ba-p/13531886
- SAP Community — カスタムコード適合プロセス（SAP公式）: https://community.sap.com/t5/enterprise-resource-planning-blog-posts-by-sap/sap-s-4hana-system-conversion-custom-code-adaptation-process/ba-p/13337309
- SAP Community — ATCを使用したカスタムコード影響分析（メンバー投稿）: https://community.sap.com/t5/enterprise-resource-planning-blog-posts-by-members/s-4hana-custom-code-impact-analysis-using-atc/ba-p/13566164
- SAP Community — SAP JouleによるS/4HANA向けコード移行（SAP公式、最新）: https://community.sap.com/t5/technology-blog-posts-by-sap/custom-code-migration-to-sap-s-4hana-powered-by-sap-joule-for-developers/ba-p/14329094
