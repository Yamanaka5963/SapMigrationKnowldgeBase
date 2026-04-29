# Official SAP S/4HANA 2023 On-Premise — Migration Guides

Official documents published by SAP for S/4HANA 2023 On-Premise (OP2023) migrations.  
Source: [SAP Help Portal — S/4HANA On-Premise 2023](https://help.sap.com/docs/SAP_S4HANA_ON-PREMISE)  
**※ 全ドキュメント英語のみ（日本語版なし）**

---

## Document List

| Document ID | Title | Version | Released | Size | PDF |
|---|---|---|---|---|---|
| SIMPL_OP2023 | Simplification List for SAP S/4HANA | 1.35 | 2025-02-25 | 9.7 MB | [PDF](https://help.sap.com/doc/c34b5ef72430484cb4d8895d5edd12af/2023/en-US/SIMPL_OP2023.pdf) |
| UPGR_OP2023 | Upgrade Guide | 6.0 | 2026-01-28 | 655 KB | [PDF](https://help.sap.com/doc/760ce610a2af4174a329d2d8315378e2/2023/en-US/UPGR_OP2023.pdf) |
| CONV_OP2023 | Conversion Guide | 6.0 | 2026-01-28 | 662 KB | [PDF](https://help.sap.com/doc/2b87656c4eee4284a5eb8976c0fe88fc/2023/en-US/CONV_OP2023.pdf) |
| INST_OP2023 | Installation Guide | 6.0 | 2026-01-28 | 1.8 MB | [PDF](https://help.sap.com/doc/6b11678926d3409bbfea8897cb34d10f/2023/en-US/INST_OP2023.pdf) |

---

## Document Details

### SIMPL_OP2023 — Simplification List
- **Version:** 1.35
- **Released:** 2025-02-25
- **Description:** SAP S/4HANA 2023 の全簡略化項目（Simplification Item）の一覧。ECC から S/4HANA 移行時に影響を受ける機能変更・廃止項目を網羅。アセスメント・カスタムコード対応の基礎資料。
- **Primary use:** 影響分析 (Fit-Gap)、カスタムコード対応優先度付け
- **PDF:** [SIMPL_OP2023.pdf](https://help.sap.com/doc/c34b5ef72430484cb4d8895d5edd12af/2023/en-US/SIMPL_OP2023.pdf)

---

### UPGR_OP2023 — Upgrade Guide
- **Version:** 6.0
- **Released:** 2026-01-28
- **Description:** 既存 S/4HANA On-Premise 環境を S/4HANA 2023 へアップグレードする手順書。Software Update Manager (SUM) を使用したアップグレードパスを詳述。
- **Primary use:** S/4HANA 既存環境のバージョンアップ（ECC からの移行ではなく S/4HANA 間の更新）
- **PDF:** [UPGR_OP2023.pdf](https://help.sap.com/doc/760ce610a2af4174a329d2d8315378e2/2023/en-US/UPGR_OP2023.pdf)

---

### CONV_OP2023 — Conversion Guide
- **Version:** 6.0
- **Released:** 2026-01-28
- **Description:** SAP ECC (ERP 6.0) から S/4HANA 2023 On-Premise へのシステムコンバージョン（Brownfield）手順書。DMO (Database Migration Option) を含む SUM ベースの変換プロセスを詳述。
- **Primary use:** Brownfield 移行の主要参照ドキュメント。技術的手順の正規ソース。
- **PDF:** [CONV_OP2023.pdf](https://help.sap.com/doc/2b87656c4eee4284a5eb8976c0fe88fc/2023/en-US/CONV_OP2023.pdf)

---

### INST_OP2023 — Installation Guide
- **Version:** 6.0
- **Released:** 2026-01-28
- **Description:** SAP S/4HANA 2023 On-Premise の新規インストール手順書。SAP SWPM (Software Provisioning Manager) を使用したシステム構築手順を詳述。
- **Primary use:** Greenfield 移行における新規システム構築、またはシェルコンバージョンのターゲットシステム構築
- **PDF:** [INST_OP2023.pdf](https://help.sap.com/doc/6b11678926d3409bbfea8897cb34d10f/2023/en-US/INST_OP2023.pdf)

---

## 関連ドキュメント

| ファイル | 内容 |
|---|---|
| [SIMPL_OP2023/index.md](./SIMPL_OP2023/index.md) | Simplification List の構造・主要項目・辞書としての使い方 |
| [analysis-tools.md](./analysis-tools.md) | ATC / Readiness Check / Cloudification Repo の詳細とアクセス方法 |
| [migration-tools-landscape.md](./migration-tools-landscape.md) | 移行ツール全景（SAP公式チェーン・GitHub公開リソース・サードパーティ） |

---

## Notes

- **OP2023** = SAP S/4HANA 2023 On-Premise（クラウド版は別系列のドキュメントID）
- 各ガイドの最新版は [SAP Help Portal — S/4HANA On-Premise](https://help.sap.com/docs/SAP_S4HANA_ON-PREMISE) から確認すること
- 上記 PDF 直リンクはバージョン固定URL（ハッシュ含む）。SAP がドキュメントを更新した場合はリンク先の版数を確認すること
- CONV / UPGR は SUM のバージョンと対応関係があるため、実施時に SUM バージョンとの互換性を確認すること
