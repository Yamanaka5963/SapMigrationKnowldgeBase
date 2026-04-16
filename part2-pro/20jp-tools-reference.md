# 20 - ツールリファレンスカード

SAP S/4HANA 移行プロジェクトで使用するすべてのツールのクイックリファレンスです。

## アセスメント & 計画ツール

| ツール | 機能 | アクセス先 |
|---|---|---|
| **SAP Readiness Check** | ECCシステムのS/4HANA適合性を分析 — カスタムコード、アドオン、簡素化項目、サイジング | SAP Note 2399707でインストール；結果は [help.sap.com](https://help.sap.com/docs/SAP_READINESS_CHECK) |
| **Maintenance Planner** | SUMのstack.XMLを生成；必要なソフトウェアパッケージとパッチを計算 | [Maintenance Planner](https://apps.support.sap.com/sap/support/mp)（S-User必要） |
| **SAP Transformation Navigator** | ランドスケープ別のアップグレードパスガイダンス（2024年Q1以降は読み取り専用） | [support.sap.com/stn](https://support.sap.com/stn)（S-User必要） |
| **Simplification Item Catalog** | S/4HANAで変更/削除されたECC機能の完全リスト | Readiness Checkに含まれる；PDFはhelp.sap.comで提供 |

## 変換 & アップグレードツール

| ツール | 機能 | アクセス先 |
|---|---|---|
| **SUM**（Software Update Manager） | ブラウンフィールドのインプレース変換の中核ツール；アップグレードスタックを管理 | [SAP Support](https://support.sap.com/en/tools/software-logistics-tools/software-update-manager.html) |
| **DMO**（Database Migration Option） | SUMのアドオン；システム変換と同時にDBをHANAへ移行 | SUMに含まれる；[SAP Support](https://support.sap.com/en/tools/software-logistics-tools/software-update-manager/database-migration-option-dmo.html)でドキュメント参照 |
| **RISE Transition Workbench** | RISE with SAP（プライベートクラウド）移行のガイド付きツール；5つの技術的移行アプローチ | 移行サーバーにプリインストール；[SAP Support](https://support.sap.com/en/tools/software-logistics-tools/rise-with-sap-system-transition-workbench.html) |

## カスタムコードツール

| ツール | 機能 | アクセス先 |
|---|---|---|
| **ABAP Test Cockpit（ATC）** | カスタムABAPのS/4HANA適合性問題をチェック | SAPシステム内のトランザクション ATC |
| **Custom Code Migration App** | カスタムコード分析と改修進捗を管理するFioriアプリ | S/4HANA Fioriランチパッド |
| **ABAP Development Tools（ADT）** | EclipseプラグインによるATCクイックフィックス（問題の約40〜60%を自動修正） | Eclipse Marketplace |
| **SAP Code Inspector（SCI）** | FUNCTIONAL_DBチェックバリアントによる補完的な静的分析 | SAPシステム内のトランザクション SCI |
| **SAP Joule for Developers** | AIを活用したカスタムコード移行（新ツール、工数を削減） | SAP BTP / S/4HANA Cloud |
| **SPDD** | 変換後のデータディクショナリ修正再調整 | トランザクション SPDD |
| **SPAU / SPAU_ENH** | 変換後のリポジトリ/エンハンスメント修正再調整 | トランザクション SPAU |

## データ移行ツール

| ツール | 機能 | アクセス先 |
|---|---|---|
| **Migration Cockpit** | 定義済みオブジェクトを使用してマスターデータとトランザクションデータを移行；ファイル/ステージング/ダイレクト転送 | S/4HANAに組み込み；[トピックページ](https://pages.community.sap.com/topics/s4hana-migration-cockpit) |
| **SAP Business Transformation Center（BTC）** | シェルコンバージョン（SDT）向けのデータ準備分析とオーケストレーション | SAP管理；SDTエンゲージメントの一部 |
| **SAP Data Services** | Migration Cockpitのステージングテーブル投入向けETLツール | SAP BTP / オンプレミス |

## プロジェクト & 監視ツール

| ツール | 機能 | アクセス先 |
|---|---|---|
| **SAP Cloud ALM** | S/4HANAのプロジェクト管理・テスト管理・監視（新規プロジェクトで推奨） | [SAP Cloud ALM](https://www.sap.com/products/technology-platform/cloud-alm.html) |
| **SAP Solution Manager** | レガシーALMプラットフォーム；旧プロジェクトで引き続き使用（トランスポート管理、監視） | オンプレミスのSAP SolManシステム |
| **SAP Early Watch Alert** | SAPが提供する定期的なシステム健全性チェックレポート | Solution ManagerまたはCloud ALM経由で設定 |

## 主要トランザクション（クイックリファレンス）

| トランザクション | 目的 |
|---|---|
| **SPDD** | データディクショナリ修正調整 |
| **SPAU / SPAU_ENH** | リポジトリ/エンハンスメント修正調整 |
| **ATC** | ABAP Test Cockpit — カスタムコードチェック |
| **SCI** | SAP Code Inspector |
| **SUSG** | 使用状況ログ — 未使用カスタムオブジェクトの特定 |
| **SM37** | ジョブ監視 — バッチジョブ |
| **SM21** | システムログ |
| **SNOTE** | SAPノートの適用 |
| **STMS** | トランスポート管理システム |
