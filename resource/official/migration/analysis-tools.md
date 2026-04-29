# SAP S/4HANA 移行 判定エンジン・分析ツール

移行影響分析に使用するツールの概要・アクセス方法・KEYSTONE としての活用方針。  
調査日: 2026-04-29

---

## 全体像

SIMPL_OP2023（1,482ページ）を人間が通読して対処するものではない。  
以下のツールが「そのシステムに関係する項目」を自動的に絞り込む。人間はツールの出力を受けて対処する。

```
対象 ECC システム
  │
  ├─ SAP Readiness Check ─→ 影響 Simplification Item 一覧（システム全体）
  │
  └─ ATC + Simplification DB ─→ カスタムコード単位の違反箇所（行レベル）
```

---

## 1. SAP Readiness Check

### 概要

ECC または S/4HANA システム上で実行し、対象システムが S/4HANA 移行時に影響を受ける Simplification Item を自動検出するツール。

### アクセス・実行要件

| 項目 | 内容 |
|---|---|
| 実行場所 | 対象の SAP システム上（トランザクション） |
| 必要アカウント | S-user（SAP カスタマー／パートナー契約） |
| 参照先 | [SAP Help Portal — Readiness Check](https://help.sap.com/docs/SAP_READINESS_CHECK) |
| SAP Community | [community.sap.com/topics/readiness-check](https://pages.community.sap.com/topics/readiness-check) |

### 判定ロジックの公開状況

- **非公開**。SAP 内部のルールエンジン
- 出力結果（影響 Item 番号リスト・推奨アクション）は取得・活用可能
- ロジック自体への外部アクセス手段なし

### 出力内容

- システムに該当する Simplification Item の一覧
- 各 Item の対応ステータス（対処済み／未対処）
- カスタマイズ・アドオン・インターフェースへの影響サマリー

---

## 2. ATC（ABAP Test Cockpit）＋ Simplification Database

### 概要

ABAP システム組み込みのコード静的解析ツール。判定の根拠となるルールセットが **Simplification Database（SYCM）**。カスタムコードをコード行レベルでスキャンし、S/4HANA 非互換箇所を特定する。

### Simplification Database のアクセス方法

**ダウンロード**（S-user が必要）
- SAP Service Marketplace から ZIP ファイルとして入手
- 手順詳細: SAP Note [2241080](https://userapps.support.sap.com/sap/support/knowledge/en/2241080)

**インポート手順**
```
1. Central Check System にログオン（SAP_BASIS 7.52 以上が必要）
2. トランザクション SYCM を実行
3. Simplification Database → Import from ZIP File を選択
4. ダウンロードした ZIP ファイルを指定して確認
```
- 詳細手順: SAP Note [3693326](https://userapps.support.sap.com/sap/support/knowledge/en/3693326)
- 事前適用 Note（Checked System 側）: 2485231 / 2270689 / 2190065
- 事前適用 Note（Central Check System 側）: 2436688 / 2364916

### 判定ロジックの公開状況

- Simplification Database（ルール本体）は **S-user があればダウンロード可能**
- ATC エンジン自体は ABAP システムに組み込み済み（追加インストール不要）
- 実行には ABAP システム（Central Check System）が必要

### 出力内容

- カスタムコードオブジェクト単位・行番号レベルの違反リスト
- 違反している Simplification Item 番号と推奨対応
- Quick Fix（ADT 上で半自動修正）が使える項目あり

---

## 3. Cloudification Repository（GitHub 公開）

### 概要

ATC チェック「Usage of Released APIs (Cloudification Repository)」が参照する、SAP Cloud ERP の公開 API リスト。主に S/4HANA Cloud / BTP ABAP 環境でのカスタムコードが使用している API の公開状況を確認するためのもの。

### リポジトリ

- **URL:** [github.com/SAP/abap-atc-cr-cv-s4hc](https://github.com/SAP/abap-atc-cr-cv-s4hc)
- **ライセンス:** オープン、無料アクセス
- **形式:** JSON ファイル（API 名・リリース状況・後継 API 等）

### パートナーによる貢献

- 自社名前空間の API は Pull Request で追加可能
- プログラム `SYCM_API_CLASSIFICATION_MANAGR` で自社 API の JSON を生成し PR 投稿

### 適用範囲と注意点

- **対象:** S/4HANA Cloud Public Edition / BTP ABAP Environment のAPI適合チェック
- **対象外:** オンプレミス ECC → S/4HANA のフル Simplification Item チェック（それは上記 Simplification Database が担う）
- オンプレミス移行案件では補助的な位置づけ

---

## ツール比較まとめ

| ツール | 対象 | ルールの公開状況 | 実行に必要なもの |
|---|---|---|---|
| SAP Readiness Check | システム全体の影響分析 | 非公開 | S-user ＋ 対象 SAP システム |
| ATC ＋ Simplification DB | カスタムコードの行レベル分析 | DB は S-user でDL可 | ABAP システム（BASIS 7.52+）|
| Cloudification Repository | Cloud API 適合チェック | GitHub で完全公開 | ATC が動く ABAP システム |

---

## KEYSTONE としての活用方針

### 最も工数削減効果が高いアプローチ

1. **Central Check System を用意する**  
   SAP CAL で ECC IDES 環境（または S/4HANA 評価版）を起動し、Simplification Database を SYCM でインポートする

2. **クライアントのカスタムコードを抽出・スキャンする**  
   クライアントの開発環境に接続し、ATC を実行してカスタムコード違反を一括取得する

3. **Readiness Check の出力と突き合わせる**  
   システムレベルの影響 Item（Readiness Check）とコードレベルの違反（ATC）を組み合わせることで、対応優先度マップを作れる

### 参照 Issue

- [Issue #1](https://github.com/Yamanaka5963/SapMigrationKnowldgeBase/issues/1) — SAP CAL で ECC 環境を構築し公式ツールのサポート範囲を検証する作業

---

## 関連資料

- [SIMPL_OP2023/index.md](./SIMPL_OP2023/index.md) — Simplification List の構造と主要項目
- [index.md](./index.md) — OP2023 公式ガイド一覧
- [SAP Note 2241080](https://userapps.support.sap.com/sap/support/knowledge/en/2241080) — Simplification Database ダウンロード
- [SAP Note 3693326](https://userapps.support.sap.com/sap/support/knowledge/en/3693326) — Simplification Database インポート手順
- [github.com/SAP/abap-atc-cr-cv-s4hc](https://github.com/SAP/abap-atc-cr-cv-s4hc) — Cloudification Repository
