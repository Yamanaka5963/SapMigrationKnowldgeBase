# 32 - MATDOC：在庫管理の統合

## 概要

MATDOC（Material Document）はSAP S/4HANAにおけるすべての在庫移動の新しい中心テーブルです。ECCのドキュメントヘッダー（`MKPF`）と明細（`MSEG`）の分割構造を置き換え、レポートパフォーマンスのために存在していた冗長な集計/スナップショットテーブルを廃止します。

この変更は在庫管理における**New Simplified Data Model（NSDM）**の一部です。

## 置き換えられたECCテーブル

| ECCテーブル | 説明 | S/4HANAでの状態 |
|---|---|---|
| `MKPF` | 材料ドキュメントヘッダー | CDSビュー `NSDM_E_MKPF` に置き換え |
| `MSEG` | 材料ドキュメント明細 | CDSビュー `NSDM_E_MSEG` に置き換え |
| `MARD` | 保管場所別在庫 | 引き続き存在；在庫数量はCDS経由でオンザフライ計算 |
| `MCHB` | バッチ別在庫 | オンザフライで計算 |
| `MSKA` | 受注別在庫 | オンザフライで計算 |
| `MSSL` | 仕入先特殊在庫 | オンザフライで計算 |
| `MSSQ` | プロジェクト在庫 | オンザフライで計算 |
| 集計/履歴テーブル | 冗長な在庫スナップショット | 廃止 |

## 新しいMATDOC関連テーブル

| テーブル | 説明 |
|---|---|
| `MATDOC` | 中心的な材料ドキュメントテーブル（ヘッダー+明細を統合） |
| `MATDOC_EXTRACT` | オンザフライ在庫計算用の圧縮エクストラクト（パフォーマンス最適化） |

## MATDOCのレコードタイプ

| レコードタイプ | 説明 |
|---|---|
| `MDOC` | 標準ヘッダー/明細エントリ（MKPF/MSEGを代替） |
| `MDOC_CP` | 補完転記（XAUTOエントリを代替） |
| `101` | 輸送中在庫 |
| `MIG_DELTA` | 移行後のアーカイブドキュメントとの数量差分 |
| `ARC_DELTA` | MATDOCに含まれないアーカイブ済み材料ドキュメント |

## 在庫数量の計算方法

ECCでは、在庫数量はスナップショットテーブル（`MARD`、`MCHB`等）に保存され、各移動のたびに更新されていました — 書き込み負荷が高く、冗長な構造でした。

S/4HANAでは、在庫は`MATDOC`からCDSビュー経由で**オンザフライで計算**されます。`MATDOC_EXTRACT`はパフォーマンス最適化のための圧縮バージョンを提供します。

```
MATDOC（全移動） → CDS集計 → 現在の在庫数量
```

`MARD`はS/4HANAでも引き続き存在しますが、その値は導出されるものであり、直接書き込みはされません。

## MIG_DELTA — 移行後の照合

移行後、以下を比較する照合ステップが実施されます：
- `MATDOC_EXTRACT`の在庫数量
- 旧`MARD` / `LABST`フィールドの在庫数量

差分は`MIG_DELTA`エントリとしてMATDOCに記録されます。エンジニアはこれらが解消されていることを検証する必要があります — 説明のつかないMIG_DELTAエントリは移行データ品質の問題を示しています。

**移行プログラム：**
- `NSDM_MIG_MATERIAL_DOC` — 材料ドキュメントをMATDOCへ移行
- `NSDM_MIG_FILL_EXTRACT` — MATDOC_EXTRACTを生成
- `NSDM_MIG_CREATE_PERFY` — パフォーマンスインデックスを作成

## カスタムコードへの影響

以下のようなZプログラム/レポートは改修が必要です：
- `MKPF`や`MSEG`を直接SELECT
- 非標準のファンクションモジュールで在庫転記
- `MARD`、`MCHB`等から在庫数量を直接読み取り

対応方法：
- 材料ドキュメントの読み取りにはCDSビュー `NSDM_E_MKPF` と `NSDM_E_MSEG` を使用
- 在庫転記には標準のSAP BAPI/APIを使用
- 在庫レポートにはCDSビューを使用（`MARD`を直接読み取らないこと）

## 参考資料

- SAP Community — 在庫管理向けNSDM（SAP公式）: https://community.sap.com/t5/enterprise-resource-planning-blog-posts-by-sap/sap-s-4hana-inventory-management-tables-new-simplified-data-model-nsdm/ba-p/13497469
- SAP Community — MSEGとMATDOC、データはどこへ？（メンバー投稿）: https://community.sap.com/t5/enterprise-resource-planning-blog-posts-by-members/s-4-hana-mseg-or-matdoc-where-is-my-data-going/ba-p/13406549
- SAP Help — 新簡素化データモデル（NSDM）: https://help.sap.com/docs/SUPPORT_CONTENT/erpscm/3362167285.html
- 簡素化一覧リスト S/4HANA 2023（公式PDF）: https://help.sap.com/doc/c34b5ef72430484cb4d8895d5edd12af/2023/en-US/SIMPL_OP2023.pdf
