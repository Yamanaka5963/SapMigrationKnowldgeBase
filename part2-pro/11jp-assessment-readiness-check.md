# 11 - アセスメント & レディネスチェック

## 概要

SAP Readiness Check は、ECCシステムのS/4HANA移行準備状況を評価する主要な自動化ツールです。早期に実施することが重要で、その結果がアプローチ選定・スコーピング・プロジェクト計画の根拠となります。

## チェック内容

| カテゴリ | 詳細 |
|---|---|
| **カスタムコード分析** | カスタムABAP量、S/4HANAとの互換性問題 |
| **簡素化項目** | S/4HANAで削除または変更されたECC機能・テーブル（430項目以上） |
| **アドオン互換性** | インストール済みアドオンのS/4HANA認定状況 |
| **ビジネス機能** | スコープ内のアクティブなビジネス機能 |
| **ビジネスプロセス探索** | 実際に使用中の標準SAPプロセス |
| **S/4HANAサイジング** | ターゲットシステムのハードウェアサイジング |
| **データ量管理** | データアーカイブの推奨事項 |
| **Fioriアプリ推奨** | アクティブなプロセスに対する推奨Fioriアプリ |
| **計画ダウンタイム分析** | 変換に必要な推定ダウンタイム |
| **CVI（顧客仕入先統合）** | BPへの移行準備状況（顧客/仕入先 → ビジネスパートナー） |

## 実施手順

1. ソースECCシステムにSAP Note **2399707** を適用
2. ソースシステム内またはSAP Cloud ALM経由でレディネスチェックを実行
3. 結果をレディネスチェックポータルにアップロードして完全なダッシュボードを確認
4. 結果ドキュメント（PDF）をダウンロードしてオフラインレビューとクライアント共有に使用

## 結果の読み方

- **赤項目** — 変換前に必ず解決すべきブロッカー
- **黄項目** — レビューと判断が必要な項目
- **緑項目** — 対応不要

優先的に確認すべき項目：
1. アドオン互換性（S/4HANA未認定の場合はハードブロッカー）
2. 対応必須の簡素化項目
3. カスタムコードの量と重大度の高い検出結果

## 簡素化項目について

各簡素化項目には専用のSAP Noteが対応しています。主なカテゴリ：
- 削除・変更されたアプリケーション機能
- テーブル構造変更（プール/クラスター → 透過テーブル）
- FI-CO統合変更
- マテリアルレジャーの強制有効化
- ビジネスパートナー（CVI）移行の強制対応

簡素化一覧リスト（公式PDF）：
- S/4HANA 2023版: https://help.sap.com/doc/c34b5ef72430484cb4d8895d5edd12af/2023/en-US/SIMPL_OP2023.pdf
- S/4HANA 2022版: https://help.sap.com/doc/59bf7d0f62d24af78f87c560da8f18ce/2022/en-US/SIMPL_OP2022.pdf

## 参考資料

- SAP Readiness Check ヘルプポータル（公式）: https://help.sap.com/docs/SAP_READINESS_CHECK
- SAP Readiness Check 機能詳細（公式）: https://help.sap.com/docs/SAP_READINESS_CHECK/a281af437b3e4ef4a187c7f35a9093e9/69463edf26e44ba8a99e16c0c0e27100.html
- SAP Community — Readiness Checkトピックページ: https://pages.community.sap.com/topics/readiness-check
- Readiness Check と Simplification Check の解説（コミュニティ投稿）: https://community.sap.com/t5/technology-blog-posts-by-members/understanding-sap-readiness-check-and-simplification-check-for-sap-s-4hana/ba-p/13770134
- 簡素化項目カタログ概要（SAP公式）: https://community.sap.com/t5/enterprise-resource-planning-blog-posts-by-sap/simplification-item-catalog-simplification-item-check-and-sap-readiness/ba-p/13440989
- SAP Note 2399707 — レディネスチェックインストール（S-User必要）
