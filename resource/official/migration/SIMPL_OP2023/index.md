# SIMPL_OP2023 — Simplification List for SAP S/4HANA 2023

**Version:** 1.35　**Released:** 2025-02-25　**Pages:** 1,482　**Size:** 9.7 MB  
**PDF:** [SIMPL_OP2023.pdf](./SIMPL_OP2023.pdf)  
**Source:** [SAP Help Portal](https://help.sap.com/doc/c34b5ef72430484cb4d8895d5edd12af/2023/en-US/SIMPL_OP2023.pdf)

---

## 本文書の目的

SAP ECC (ERP 6.0) から SAP S/4HANA 2023 に移行する際に影響を受ける**全Simplification Item（簡略化項目）**を網羅したカタログ。  
移行前の影響分析（Fit-Gap / Readiness Check）の正規参照先であり、カスタムコード対応優先度付けの基礎資料。

---

## 文書の構成

| セクション | 内容 |
|---|---|
| Introduction | 本文書の読み方、Impact Category の定義 |
| Overview Table | 全項目一覧（機能領域・Impact・SAP Note 番号） |
| 各機能領域の詳細 | 項目ごとの変更内容・影響オブジェクト・対応手順 |

### 各 Simplification Item の記載内容
- **Description:** 変更・廃止の内容
- **Impact Category:** 必須対応 (Mandatory) / 推奨対応 (Optional / Recommended)
- **Affected Objects:** テーブル・トランザクション・プログラム
- **Recommended Action:** 移行手順または代替手段
- **SAP Notes:** 関連ノート番号
- **Compatibility Scope:** 対象 ECC リリース

---

## 主要 Simplification Item（機能領域別）

### Finance (FI)
| 項目 | 概要 | Impact |
|---|---|---|
| Universal Journal (ACDOCA) | FI/CO 統合仕訳テーブル。BSEG/BKPF・COEP・COSS・COSP 等が ACDOCA に統合 | Mandatory |
| New Asset Accounting (FAA_DC) | 資産会計の新テーブル構造。旧 ANLA/ANLZ/ANLC が再編 | Mandatory |
| Bank Account Management | 銀行口座管理の新 Fiori アプリへ移行。旧 FI-BL が廃止 | Mandatory |
| Classic G/L の廃止 | Classic G/L は S/4HANA では使用不可。New G/L への移行完了が前提 | Mandatory |
| CO-PA (Account-Based) | 勘定ベース CO-PA が標準。原価要素ベース CO-PA は限定サポート | Mandatory |

### Controlling (CO)
| 項目 | 概要 | Impact |
|---|---|---|
| Cost Element → G/L Account 統合 | 原価要素 (Cost Element) が G/L 勘定科目に統合。SKA1/SKAT の構造変更 | Mandatory |
| Profit Center Accounting (EC-PCA) | 旧 EC-PCA テーブル（GLPCA 等）廃止。ACDOCA に統合 | Mandatory |
| Material Ledger 必須化 | S/4HANA では Material Ledger が必須。CKMLCR/CKMLHD 等のテーブル変更 | Mandatory |

### Materials Management (MM)
| 項目 | 概要 | Impact |
|---|---|---|
| Inventory Management (MATDOC) | MSEG/MKPF が廃止。MATDOC テーブルへ統合 | Mandatory |
| Purchasing の簡略化 | ME21N 等の一部機能変更。Ariba 連携向けの構造変更 | Optional |
| Batch Management | バッチ管理の UI が Fiori へ移行 | Optional |

### Sales & Distribution (SD)
| 項目 | 概要 | Impact |
|---|---|---|
| Pricing Conditions (PRCD_ELEMENTS) | KONV テーブル廃止。PRCD_ELEMENTS に移行。カスタムコード全面見直しが必要 | Mandatory |
| Output Management | NAST ベースの出力管理が廃止。BRF+ / Adobe Forms ベースへ移行 | Mandatory |
| Credit Management (FSCM-CR) | FI-AR ベースの与信管理廃止。FSCM Credit Management に統一 | Mandatory |

### Logistics (PP/PM/QM)
| 項目 | 概要 | Impact |
|---|---|---|
| Production Orders | PP テーブル構造の一部変更。Fiori アプリへの移行推奨 | Optional |
| Plant Maintenance (EAM) | PM オーダー・機能場所の構造一部変更 | Optional |
| Quality Management | QM 通知・検査ロットの一部 UI 変更 | Optional |

### Master Data
| 項目 | 概要 | Impact |
|---|---|---|
| Business Partner (CVI) | KNA1/LFA1 が廃止。BP（Business Partner）API に統一。CVI 移行ツール必須 | Mandatory |
| Material Master 簡略化 | 一部フィールド・トランザクションの廃止・変更 | Optional |

### Basis / Technical
| 項目 | 概要 | Impact |
|---|---|---|
| Pool/Cluster Tables の廃止 | BSEG（Pool）・RFBLG・VBDATA 等の Cluster テーブルが透過テーブルに変換 | Mandatory |
| BI Extractor の廃止 | 一部の旧 BW Extractor が廃止。CDS View への移行が推奨 | Mandatory |
| Process Observer の変更 | 旧プロセスオブザーバー機能の廃止・代替手段への移行 | Optional |
| RFC / ICF の変更 | 一部 RFC モジュールの廃止または仕様変更 | Optional |

---

## Impact Category の定義

| Category | 意味 |
|---|---|
| **Mandatory** | 対応しないとシステムが動作しない。コンバージョン前に必ず対処が必要 |
| **Recommended** | 対応しなくても動作するが、機能制限またはパフォーマンス劣化が発生 |
| **Optional** | 将来的な廃止に向けた任意対応。現時点では動作継続可能 |

---

## エンジニア向け利用ガイド

1. **アセスメント時：** SAP Readiness Check の結果と本文書を突き合わせ、影響項目を特定する
2. **カスタムコード対応時：** ATC（ABAP Test Cockpit）で検出された違反と本文書の推奨 Action を照合する
3. **重点確認領域：** ACDOCA・MATDOC・PRCD_ELEMENTS・CVI・Pool/Cluster は全移行案件で必須確認
4. **SAP Note の追跡：** 各項目に記載の SAP Note 番号で最新の修正・回避策を確認すること

---

## 関連資料

- [CONV_OP2023.pdf](https://help.sap.com/doc/2b87656c4eee4284a5eb8976c0fe88fc/2023/en-US/CONV_OP2023.pdf) — コンバージョン実施手順
- [../index.md](../index.md) — OP2023 公式ガイド一覧
