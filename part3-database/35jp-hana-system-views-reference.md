# 35 - SAP HANA システムビューリファレンス

## 概要

SAP HANAは、メタデータ・監視・管理情報を**システムビュー**として公開しています。`SYS`スキーマにある読み取り専用の仮想テーブルで、S/4HANAランドスケープを管理するBasisエンジニアやDBAにとって必須の参照先です。

システムビューはSQL経由でアクセスします（HANA Studio、HANA Cockpit、またはHANAに接続したDBeaver経由）。常に`SYS`スキーマに存在します。

```sql
SELECT * FROM SYS.M_TABLES WHERE SCHEMA_NAME = 'SAPHANADB';
```

## 主要なシステムビューカテゴリ

### スキーマ & テーブル情報

| ビュー | 説明 |
|---|---|
| `SYS.SCHEMAS` | HANAデータベース内のすべてのスキーマ |
| `SYS.TABLES` | すべてのテーブル（種別、スキーマ、レコード数メタデータ） |
| `SYS.TABLE_COLUMNS` | すべてのテーブルの列定義 |
| `SYS.VIEWS` | すべてのビュー |
| `SYS.INDEXES` | すべてのインデックス |
| `SYS.M_TABLES` | ランタイムテーブル情報：メモリサイズ、レコード数、テーブル種別 |
| `SYS.M_TABLE_PERSISTENCE_STATISTICS` | テーブル永続化統計（ディスク vs メモリ） |

### メモリ & パフォーマンス監視

| ビュー | 説明 |
|---|---|
| `SYS.M_MEMORY_OVERVIEW` | HANAメモリ使用量の概要サマリー |
| `SYS.M_HOST_MEMORY` | ホスト別メモリ使用量 |
| `SYS.M_CS_TABLES` | カラムストアテーブルの詳細 — メモリ消費量、圧縮率 |
| `SYS.M_RS_TABLES` | ローストアテーブルの詳細 |
| `SYS.M_TABLE_COLUMNS_STATISTICS` | 列レベルの統計（重複排除値数、圧縮率） |
| `SYS.M_EXPENSIVE_STATEMENTS` | 最近実行されたコストの高いSQLステートメント |
| `SYS.M_SQL_PLAN_CACHE` | SQLプランキャッシュ — 実行済みステートメントとパフォーマンス |

### セッション & 接続

| ビュー | 説明 |
|---|---|
| `SYS.M_CONNECTIONS` | アクティブなデータベース接続 |
| `SYS.M_SESSIONS` | アクティブなセッション |
| `SYS.M_BLOCKED_TRANSACTIONS` | ブロックされたトランザクション — デッドロック調査に使用 |
| `SYS.M_LOCK_WAITS_STATISTICS` | ロック待機履歴 |

### サービス & ランドスケープ

| ビュー | 説明 |
|---|---|
| `SYS.M_SERVICES` | 稼働中のHANAサービス（indexserver、nameserver等） |
| `SYS.M_HOST_INFORMATION` | ホストレベルのシステム情報（CPU、メモリ、OS） |
| `SYS.M_SYSTEM_OVERVIEW` | システム全体の健全性ダッシュボード |
| `SYS.M_LANDSCAPE_HOST_CONFIGURATION` | スケールアウト構成のトポロジー |
| `SYS.M_DATABASE` | データベースレベル情報（システムID、バージョン、起動時刻） |

### バックアップ & リカバリー

| ビュー | 説明 |
|---|---|
| `SYS.M_BACKUP_CATALOG` | バックアップ履歴とカタログ |
| `SYS.M_BACKUP_CATALOG_FILES` | バックアップファイルの物理的な保存先 |
| `SYS.M_LOG_SEGMENTS` | REDOログセグメントの状態 |

### ユーザー & セキュリティ

| ビュー | 説明 |
|---|---|
| `SYS.USERS` | すべてのデータベースユーザー |
| `SYS.ROLES` | すべてのロール |
| `SYS.GRANTED_ROLES` | ユーザー別のロール割り当て |
| `SYS.GRANTED_PRIVILEGES` | ユーザー別の権限割り当て |
| `SYS.M_PASSWORD_POLICY` | 現在のパスワードポリシー設定 |
| `SYS.AUDIT_LOG` | 監査証跡（監査ポリシーが有効な場合） |

## S/4HANAにおけるSAPアプリケーションスキーマ

S/4HANAのアプリケーションデータ（`ACDOCA`、`MATDOC`等のABAPワークロードテーブル）は、**SAPアプリケーションスキーマ**に格納されています。通常、SAP システムID（SID）にちなんで命名されます（例：`SAPHANADB`、`SAP<SID>`）。

```sql
-- 全スキーマの一覧
SELECT SCHEMA_NAME FROM SYS.SCHEMAS;

-- アプリケーションスキーマ内のS/4HANAテーブルを検索
SELECT TABLE_NAME, RECORD_COUNT, TABLE_SIZE
FROM SYS.M_TABLES
WHERE SCHEMA_NAME = 'SAP<SID>'
ORDER BY TABLE_SIZE DESC;
```

## よく使う診断クエリ

### メモリサイズ上位20テーブル
```sql
SELECT TOP 20 TABLE_NAME, SCHEMA_NAME,
       ROUND(TABLE_SIZE / 1024 / 1024, 2) AS SIZE_MB,
       RECORD_COUNT
FROM SYS.M_TABLES
WHERE SCHEMA_NAME = 'SAP<SID>'
ORDER BY TABLE_SIZE DESC;
```

### 現在実行中のコストの高いステートメント
```sql
SELECT TOP 20 STATEMENT_STRING, DURATION_MICROSEC,
       START_TIME, USER_NAME
FROM SYS.M_EXPENSIVE_STATEMENTS
ORDER BY DURATION_MICROSEC DESC;
```

### アクティブなブロッキングロック
```sql
SELECT * FROM SYS.M_BLOCKED_TRANSACTIONS;
```

## 参考資料

- SAP HANA SQL・システムビューリファレンス（公式PDF、HANA 2.0 SPS03）: https://help.sap.com/doc/9b40bf74f8644b898fb07dabdd2a36ad/2.0.03/en-US/SAP_HANA_SQL_and_System_Views_Reference_en.pdf
- SAP Help — システムビューリファレンス、HANAプラットフォーム（公式）: https://help.sap.com/docs/SAP_HANA_PLATFORM/4fe29514fd584807ac9f2a04f6754767/20cbb10c75191014b47ba845bfe499fe.html
- SAP Help — システムビューリファレンス、HANA Cloud（公式）: https://help.sap.com/docs/hana-cloud-database/sap-hana-cloud-sap-hana-database-sql-reference-guide/system-views-reference
