# SIMPL_OP2023 — Simplification List for SAP S/4HANA 2023

**Version:** 1.35　**Released:** 2025-02-25　**Pages:** 1,482　**Size:** 9.7 MB  
**PDF:** [SIMPL_OP2023.pdf](./SIMPL_OP2023.pdf)　**抽出テキスト:** [SIMPL_OP2023.txt](./SIMPL_OP2023.txt) (3 MB / 71,180行)  
**Source:** [SAP Help Portal](https://help.sap.com/doc/c34b5ef72430484cb4d8895d5edd12af/2023/en-US/SIMPL_OP2023.pdf)

---

## 本文書の目的

正式タイトル：*Simplification List for SAP S/4HANA 2023 initial shipment, Feature Pack Stack 1-3  
and SAP S/4HANA Cloud Private Edition 2023 initial shipment, Feature Pack Stack 1-3*

SAP ECC (ERP 6.0) から SAP S/4HANA 2023 に移行する際に影響を受ける**全 Simplification Item** を網羅したカタログ。  
移行前の影響分析（Fit-Gap / Readiness Check）の正規参照先であり、カスタムコード対応優先度付けの基礎資料。

---

## 文書構成（全 63 章）

| # | 章タイトル | 概要 |
|---|---|---|
| 1 | Introduction | 文書の読み方・Impact Category 定義・コンバージョン順序 |
| 2 | Cross Topics | ABAP全般・Basis・Pool Tables廃止・BI Extractor・Output Management など横断技術項目 |
| 3 | Master Data | Business Partner (CVI)・Material Master・Batch Management |
| 4 | Commercial Project Management | CPM機能変更 |
| 5 | Commodity Management | Commodity Pricing Engine・Market Data・Risk Analytics |
| 6 | Corporate Close | EC-CS・Group Reporting |
| 7 | Customer Service | CS機能変更・CIC |
| 8 | Financials - General | データモデル変更 (ACDOCA)・置換トランザクション・Travel Management |
| 9 | Financials - AP/AR | FSCM Biller Direct 廃止 |
| 10 | Financials - Asset Accounting | 資産会計テーブル構造変更 (FAA_DC) |
| 11 | Financials - Cash Management | 現金管理の新データモデル |
| 12 | Financials - Controlling | CO統合・Material Ledger 必須化 |
| 13 | Cost Management and Profitability Analysis | CO-PA (Account-Based 標準化) |
| 14 | Financials - Financial Operations | 金融操作機能 |
| 15 | Financials - General Ledger | New G/L必須・Special Purpose Ledger廃止 |
| 16 | Financials - International Trade | 外国貿易機能変更 |
| 17 | Financials - Treasury and Risk Management | TRM機能変更 |
| 18 | Financials - Miscellaneous | FI雑則 |
| 19 | Human Resources | HCM/SuccessFactors統合・Payroll・ESS/MSS・Time Management |
| 20 | Logistics - General | ロジスティクス横断 |
| 21 | Logistics - AB (Agency Business) | 代理業務 |
| 22 | Logistics - ATP | 有効在庫確認 |
| 23 | Logistics - DSD | Direct Store Delivery |
| 24 | Logistics - EHS | 環境・健康・安全管理 |
| 25 | Logistics - GT (Global Trade) | グローバル貿易 |
| 26 | Logistics - Management of Change | 変更管理 |
| 27 | Logistics - MM-IM | 在庫管理 (MATDOC統合) |
| 28 | Logistics - PLM | 製品ライフサイクル管理 |
| 29 | Logistics - PM (Plant Maintenance) | プラント保全 |
| 30 | Logistics - PP | 生産計画 |
| 31 | Logistics - PP/DS | 詳細スケジューリング |
| 32 | Logistics - PS (Project System) | プロジェクトシステム |
| 33 | Logistics - PSS | |
| 34 | Logistics - QM | 品質管理 |
| 35 | Logistics - TRA | |
| 36 | Logistics - WM | 倉庫管理 |
| 37 | Portfolio and Project Management (PPM) | PPM |
| 38 | Procurement | 購買 |
| 39 | Sales & Distribution | SD (KONV→PRCD_ELEMENTS・Credit Management) |
| 40 | Service Operations & Processes | サービス業務 |
| 41 | Transportation Management | 輸送管理 |
| 42 | Globalization Financials | 財務ローカライゼーション |
| 43 | Globalization Logistics | ロジスティクスローカライゼーション |
| 44 | Industry - Cross | 業種横断 |
| 45 | Industry Banking | 銀行業 |
| 46 | Industry Beverage | 飲料業 |
| 47 | Defense & Security | 防衛・安全保障 |
| 48 | Industry DIMP - Aerospace & Defense | 航空宇宙・防衛 |
| 49 | Industry DIMP - Automotive | 自動車 |
| 50 | Industry DIMP - EC&O | エンジニアリング・建設 |
| 51 | Industry DIMP - High Tech | ハイテク |
| 52 | Industry DIMP - Mill | 製紙・鉄鋼 |
| 53 | Healthcare (IS-H) | 医療（S/4HANAでは非対応） |
| 54 | Industry Insurance | 保険業 |
| 55 | Industry Media | メディア業 |
| 56 | Industry Oil | 石油業 |
| 57 | Industry Oil - OGSD | 石油 OGSD |
| 58 | Industry Oil - OTAS | 石油 OTAS |
| 59 | Industry Public Sector | 公共部門 |
| 60 | Industry Retail & Fashion | 小売・ファッション |
| 61 | Industry Telecommunications | 通信業 |
| 62 | Industry Utilities | ユーティリティ業（IS-U CVI統合等） |
| 63 | SAP Ariba | Ariba統合 |

---

## 移行必須の主要 Simplification Item

### Finance / Controlling

| 項目 | 変更内容 | Impact |
|---|---|---|
| **8.2 DATA MODEL CHANGES IN FIN** | BSEG/BKPF・COEP・COSS・COSP 等が ACDOCA（Universal Journal）に統合 | Mandatory |
| **10.1 Asset Accounting テーブル変更** | 旧資産会計テーブル廃止。新 FAA_DC データ構造へ移行 | Mandatory |
| **15.x New G/L 必須** | Classic G/L 廃止。New G/L への移行完了が前提 | Mandatory |
| **15.x Special Purpose Ledger** | SPL廃止。Extension Ledger に移行 | Mandatory |
| **12.x Material Ledger 必須化** | S/4HANAでは Material Ledger が必須。ML未使用システムは要対応 | Mandatory |
| **13.x CO-PA Account-Based** | 勘定ベース CO-PA が標準。原価要素ベース CO-PA はサポート限定 | Mandatory |

### Master Data

| 項目 | 変更内容 | Impact |
|---|---|---|
| **3.19 Business Partner Approach** | KNA1/LFA1 は廃止。Business Partner (BP) API に統一。CVI移行ツール必須 | Mandatory |
| **3.22 Material Number Field Length** | 品目番号の桁数拡張（最大40桁）に伴うカスタムコード影響 | Mandatory |

### Logistics

| 項目 | 変更内容 | Impact |
|---|---|---|
| **27.x MM-IM (MATDOC)** | MSEG/MKPF 廃止。MATDOC テーブルへ統合 | Mandatory |
| **39.x SD - Pricing Conditions** | KONV テーブルが PRCD_ELEMENTS に置換。データは SUM ACT_UPG フェーズで自動移行されるが、KONV へのアペンド構造は手動対応必要。カスタムコードは PRCD_ELEMENTS を直接参照するよう修正が必要 | Mandatory |
| **39.x SD - Credit Management** | FI-AR ベースの与信管理廃止。FSCM Credit Management に統一 | Mandatory |

### Technical / Basis

| 項目 | 変更内容 | Impact |
|---|---|---|
| **2.17 End of Support for Pool Tables** | BSEG (Pool)・RFBLG・VBDATA 等の Cluster/Pool テーブルが透過テーブルに変換 | Mandatory |
| **2.27 BI Extractors** | 一部の旧 BW Extractor 廃止。CDS View への移行推奨 | Mandatory |
| **2.43 Output Management** | NAST ベースの出力管理廃止。BRF+ / Adobe Forms ベースへ移行 | Mandatory |
| **2.14 Dual Stack not supported** | AS Java との Dual Stack は S/4HANA 非対応 | Mandatory |
| **2.13 AS Java not available** | AS Java 廃止 | Mandatory |

---

## KONV → PRCD_ELEMENTS の補足（SD エンジニア向け）

- **自動移行：** SUM の `ACT_UPG` フェーズで KONV データが PRCD_ELEMENTS に自動移行される
- **KONV の扱い：** KONV テーブル自体は残存するが、データ永続化の役割は PRCD_ELEMENTS が担う
- **アペンド構造：** KONV にアペンド構造を追加している場合、SUM がプロンプトを出す → 手動で PRCD_ELEMENTS 側に移行作業が必要
- **カスタムコード：** KONV を直接 SELECT しているコードは PRCD_ELEMENTS 参照に書き換えること。変換ファンクション（PRCD_ELEMENT↔KONV）は SAP が提供している
- **フィールド変更：** `STUFE`（Access Number）・`KOLNR` 等のフィールドが PRCD_ELEMENTS で変更されている

---

## この文書の正しい使い方

**SIMPL_OP2023 は通読する文書ではない。ツールが参照する辞書である。**

人間が1,482ページを読んで対処するものではなく、以下のツールが「あなたのシステムに関係する項目」を自動的に絞り込んでくれる。エンジニアはツールの出力結果に含まれる Item 番号をこの文書で引くだけでよい。

```
SAP Readiness Check 実行
  → "Item 8.2, 27.3, 39.1 があなたのシステムに該当します" と出力
  → その番号だけを SIMPL_OP2023 で参照（辞書引き）

ATC（ABAP Test Cockpit）実行
  → "このコードは Simplification Item XXX に違反" と指摘
  → 詳細と対応手順を SIMPL_OP2023 で確認
```

全体で把握しておくべきは**章構成（どの領域が何章にあるか）**だけ。上記の目次表がその役割を果たす。

---

## 判定エンジンへのアクセス（ATC / Readiness Check）

ツールの判定ロジックがどこまで公開・アクセス可能かの調査結果。

### SAP Readiness Check

- **形態：** SAP システム内で動作するオンプレミスツール（トランザクション）
- **アクセス：** S-user アカウント（SAP カスタマー／パートナー契約）があれば使用可能
- **判定ロジック：** 非公開。SAP 内部のルールエンジン
- **結論：** ロジック自体には外部からアクセス不可。出力結果（影響 Item リスト）を使う

### ATC（ABAP Test Cockpit）＋ Simplification Database

- **形態：** ABAP システム組み込みのコード検査ツール。判定の根拠データが **Simplification Database（SYCM）**
- **Simplification Database のアクセス方法：**
  - SAP Service Marketplace からダウンロード可能（SAP Note [2241080](https://userapps.support.sap.com/sap/support/knowledge/en/2241080)）
  - Central Check System（SAP_BASIS 7.52 以上）に `SYCM` トランザクションで ZIP インポート
  - SAP Note [3693326](https://userapps.support.sap.com/sap/support/knowledge/en/3693326) に手順詳細あり
- **結論：** データベース（ルール）自体は S-user があればダウンロード・参照可能。ただし実行には ABAP システムが必要

### Cloudification Repository（GitHub 公開）

- **形態：** ATC チェック「Usage of Released APIs」が参照する API リリース情報の JSON ファイル群
- **リポジトリ：** [github.com/SAP/abap-atc-cr-cv-s4hc](https://github.com/SAP/abap-atc-cr-cv-s4hc)
- **対象：** 主に S/4HANA Cloud／BTP ABAP 環境向け。オンプレミスの Simplification Item とは別物
- **パートナー貢献：** 自社名前空間の API は PR で追加可能
- **結論：** 完全公開。ただしカスタムコードの「どの API が使用禁止か」の確認用であり、S/4HANA 移行の Simplification Item 全般をカバーするものではない

### まとめ

| ツール | ルールの公開状況 | 実行に必要なもの |
|---|---|---|
| SAP Readiness Check | 非公開 | S-user + 対象 SAP システム |
| ATC + Simplification DB | DB は S-user でダウンロード可 | ABAP システム（BASIS 7.52+）|
| Cloudification Repository | GitHub で完全公開 | ATC が動く ABAP システム |

**KEYSTONE として活用できる現実的な方法：**  
クライアントから ABAP システムへのアクセス（開発環境）が得られれば、Simplification Database を導入した Central Check System を立て、ATC でカスタムコードを一括スキャンできる。これが現時点で最も工数削減効果が高い。SAP CAL で ECC IDES 環境を立てる Issue (#1) もこの観点で有効。

---

## テキスト抽出について

PDF から [SIMPL_OP2023.txt](./SIMPL_OP2023.txt) を生成した手順。

### 使用ツール

`pdftotext`（Xpdf / Poppler）— Git for Windows に同梱されている場合がある。

```bash
# パス確認
where pdftotext
# → C:\Program Files\Git\mingw64\bin\pdftotext.exe

# 抽出実行（レイアウト保持）
pdftotext -layout SIMPL_OP2023.pdf SIMPL_OP2023.txt
```

### テキストの活用例

```bash
# 目次のトップレベル章を一覧表示
grep -n "^[0-9]\{1,2\}\. " SIMPL_OP2023.txt

# 特定キーワードを検索（例: KONV）
grep -n "KONV" SIMPL_OP2023.txt

# 特定行範囲を確認（例: 48850〜48970行目）
sed -n '48850,48970p' SIMPL_OP2023.txt
```

### 備考

- `pdftotext` が未インストールの場合は [Xpdf tools](https://www.xpdfreader.com/download.html) または Poppler をインストールすること
- `-layout` オプションでページレイアウトを保持。テーブル形式のデータは `-table` オプションも有効
- 本 txt は PDF v1.35（2025-02-25）から生成。PDF が更新された場合は再抽出すること

---

## 関連資料

- [CONV_OP2023.pdf](https://help.sap.com/doc/2b87656c4eee4284a5eb8976c0fe88fc/2023/en-US/CONV_OP2023.pdf) — コンバージョン実施手順
- [../index.md](../index.md) — OP2023 公式ガイド一覧
