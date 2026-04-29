# SAP S/4HANA 移行ツール全景

移行プロジェクトで使用できるツールの一覧。SAP公式・公開リソース・サードパーティを網羅。  
調査日: 2026-04-29

---

## SAP 公式ツールチェーン（フロー順）

移行プロジェクトにおける標準的なツール使用順序：

```
[1年前〜]  ABAP Call Monitor (SCMON)    ← 未使用コード特定
[アセスメント]  SAP Readiness Check          ← システム影響分析
[アセスメント]  Custom Code Migration App    ← コード影響の可視化・スコーピング
[アセスメント]  ATC + Simplification DB      ← コード行レベル違反検出
[技術準備]   Maintenance Planner           ← stack.xml 生成
[実行]       SUM + DMO                     ← システムコンバージョン本体
[後処理]     ADT (Quick Fix)               ← コード修正（半自動）
```

---

### 1. ABAP Call Monitor（SCMON / SUSG）

**重要度：高。移行の1年以上前から開始が必要。**

| 項目 | 内容 |
|---|---|
| トランザクション | `/SDF/SCMON`（収集）・`SUSG`（分析） |
| 目的 | 本番稼働中のカスタムコードの使用実績を収集し、未使用・デッドコードを特定する |
| アクセス | SAP システム標準機能（追加費用なし） |
| 推奨期間 | **本番環境で最低1年間**の収集が必要。収集期間が短いほど精度が低下 |

**活用：** Custom Code Migration App が SCMON データを読み込み、「未使用コード」を自動判定する。コンバージョン時にこれらを自動削除することで、対応工数を大幅削減できる。

---

### 2. SAP Readiness Check

→ [analysis-tools.md](./analysis-tools.md) 参照

---

### 3. Custom Code Migration Fiori App

| 項目 | 内容 |
|---|---|
| 形態 | SAP Fiori アプリ（S/4HANA または BTP 上で動作） |
| 目的 | ATC の検査結果を可視化・フィルタリング。移行スコープの管理 |
| アクセス | S-user + S/4HANA または BTP ABAP Environment |
| ガイド | [Custom Code Migration Guide (2023.latest)](https://help.sap.com/doc/9dcbc5e47ba54a5cbb509afaa49dd5a1/2023.latest/en-US/CustomCodeMigration_EndToEnd.pdf) |
| ハンズオン | [github.com/SAP-samples/abap-platform-ccm-workshops](https://github.com/SAP-samples/abap-platform-ccm-workshops)（無料） |

**ATC との違い：** ATC は検査エンジン。Custom Code Migration App は ATC 結果の**フロントエンド**。スコープ管理・進捗追跡・未使用コード特定・自動削除の UI を提供する。

---

### 4. ATC（ABAP Test Cockpit）＋ Simplification Database

→ [analysis-tools.md](./analysis-tools.md) 参照

---

### 5. Maintenance Planner

| 項目 | 内容 |
|---|---|
| 形態 | Web ツール（SAP For Me / SAP Support Portal） |
| 目的 | アドオン・コンポーネントの互換性確認、SUM 用の **stack.xml** を生成 |
| アクセス | S-user が必要 |
| URL | [apps.support.sap.com/sap/support/mp](https://apps.support.sap.com/sap/support/mp) |

**stack.xml：** SUM が読み込む設定ファイル。これがないと SUM が起動できない。Maintenance Planner のステップはコンバージョン前の必須作業。

---

### 6. SUM（Software Update Manager）＋ DMO

| 項目 | 内容 |
|---|---|
| 形態 | オンプレミスで動作するインストーラ型ツール |
| 目的 | ECC → S/4HANA のシステムコンバージョン本体。DB マイグレーション（HANA への移行）を同時実行 |
| アクセス | SAP Service Marketplace からダウンロード（S-user 必要） |
| 参照 | [support.sap.com — DMO](https://support.sap.com/en/tools/software-logistics-tools/software-update-manager/database-migration-option-dmo.html) |

**DMO の役割：** SUM に統合されたオプション。Unicode 変換・S/4HANA アップグレード・DB の HANA 移行を**1回の処理で実行**できる。DB が既に HANA の場合は DMO 不要。

---

### 7. ADT（ABAP Development Tools for Eclipse）

| 項目 | 内容 |
|---|---|
| 形態 | Eclipse プラグイン（無料） |
| 目的 | ATC 結果の確認とコード修正。**Quick Fix** 機能で一部の違反を半自動修正可能 |
| アクセス | 無料。[tools.hana.ondemand.com](https://tools.hana.ondemand.com) からインストール |

**Quick Fix：** ABAP コード内の S/4HANA 非互換箇所に対して、Eclipse 上でワンクリック修正候補を提示する機能。単純な置換（BSEG → ACDOCA の読み替え等）は自動化できる。

---

### 8. SAP Joule for Developers / ABAP AI（2025年〜）

| 項目 | 内容 |
|---|---|
| 形態 | ADT（Eclipse）上の AI アシスタント機能 |
| 目的 | カスタムコードの S/4HANA 対応を AI が支援・自動生成 |
| 対象 | SAP S/4HANA Cloud Private Edition 2025 以降が必要 |
| 参照 | [SAP Community — Joule for Developers](https://community.sap.com/t5/technology-blog-posts-by-sap/custom-code-migration-to-sap-s-4hana-powered-by-sap-joule-for-developers/ba-p/14329094) |

**現状の制約：** S/4HANA Cloud PE 2025 以降が対象。オンプレミス ECC からのコンバージョン案件では現時点では直接使えない。将来の活用を視野に入れる程度。

---

## GitHub 公開リソース（無料）

| リポジトリ | 内容 | URL |
|---|---|---|
| abap-platform-ccm-workshops | Custom Code Migration のハンズオン教材（SAP公式） | [GitHub](https://github.com/SAP-samples/abap-platform-ccm-workshops) |
| abap-atc-cr-cv-s4hc | Cloudification Repository（API リリース状況 JSON） | [GitHub](https://github.com/SAP/abap-atc-cr-cv-s4hc) |

---

## サードパーティツール

### Panaya

| 項目 | 内容 |
|---|---|
| 形態 | SaaS（商用） |
| 特徴 | AI 駆動のコード分析・自動修正。SAP 指摘の95%以上を自動修正と主張 |
| 強み | ATC と比較してコンサルタント工数を削減できる可能性。クラウド上でコードをスキャン |
| 注意 | コードを外部にアップロードするため、クライアントの情報セキュリティポリシー確認が必要 |
| URL | [panaya.com/sap/sap-s4hana](https://www.panaya.com/sap/sap-s4hana/) |

### SNP BLUEFIELD

| 項目 | 内容 |
|---|---|
| 形態 | 商用サービス＋ツール |
| 特徴 | Greenfield／Brownfield の中間アプローチ（Selective Data Transition）を自動化 |
| 強み | 移行データ・プロセスの選択的移行が可能。Shell Conversion に近い概念 |
| 実績 | 12,500件以上のプロジェクト実績（SNP 公表値） |
| URL | [snpgroup.com — BLUEFIELD](https://www.snpgroup.com/en/resources/blog/greenfield-brownfield-bluefield/) |

### smartShift

| 項目 | 内容 |
|---|---|
| 形態 | 商用（自動コード変換） |
| 特徴 | カスタムコードの自動変換に特化 |
| URL | [smartshift.com](https://smartshift.com/solution/sap-cloud-migration/) |

---

## KEYSTONE での優先度評価

| ツール | 優先度 | 理由 |
|---|---|---|
| ABAP Call Monitor | **今すぐ着手** | 本番収集に1年必要。クライアントに早期起動を勧めるだけでよい |
| ATC + Simplification DB | **高** | 最も正確なコード影響分析。Central Check System 構築が前提 |
| Custom Code Migration App | **高** | ATC 結果の可視化・未使用コード管理。工数削減に直結 |
| Maintenance Planner + SUM/DMO | **必須** | コンバージョン実行に不可欠。スキップ不可 |
| ADT Quick Fix | **中** | 単純な修正の自動化。対応済み件数の底上げに有効 |
| Panaya | **参考** | 商用。競合他社の手法として把握しておく価値あり |
| SNP BLUEFIELD | **参考** | Selective Data Transition の自動化選択肢 |
| SAP Joule / ABAP AI | **将来** | Cloud PE 2025 以降。現案件では直接利用不可 |

---

## 関連資料

- [analysis-tools.md](./analysis-tools.md) — ATC / Readiness Check / Cloudification Repo の詳細
- [SIMPL_OP2023/index.md](./SIMPL_OP2023/index.md) — Simplification List の構造と判定エンジンの使い方
- [index.md](./index.md) — OP2023 公式ガイド一覧
