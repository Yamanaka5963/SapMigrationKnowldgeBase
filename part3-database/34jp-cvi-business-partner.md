# 34 - 得意先/仕入先 → ビジネスパートナー（CVI）

## 概要

SAP S/4HANAでは、**得意先と仕入先が統合されたビジネスパートナー（BP）オブジェクトに置き換えられます**。CVI（Customer-Vendor Integration：顧客仕入先統合）は、レガシーの得意先/仕入先テーブルと新しいBPマスターデータを同期させる技術的フレームワークです。

この移行は、ブラウンフィールド・グリーンフィールド・シェルコンバージョンを問わず、**すべてのS/4HANAプロジェクトで必須**です。

## 影響を受けるECCテーブル

### 得意先マスター
| ECCテーブル | 説明 | S/4HANAでの状態 |
|---|---|---|
| `KNA1` | 得意先マスター（一般データ） | 引き続き存在 — BPから同期 |
| `KNB1` | 得意先マスター（会社コード） | 引き続き存在 — BPから同期 |
| `KNBK` | 得意先銀行データ | 引き続き存在 — BPから同期 |
| `KNVV` | 得意先販売データ | 引き続き存在 — BPから同期 |
| `KNVP` | 得意先パートナー機能 | 引き続き存在 |
| `KNVS` | 得意先出荷データ | 引き続き存在 |

### 仕入先マスター
| ECCテーブル | 説明 | S/4HANAでの状態 |
|---|---|---|
| `LFA1` | 仕入先マスター（一般データ） | 引き続き存在 — BPから同期 |
| `LFB1` | 仕入先マスター（会社コード） | 引き続き存在 — BPから同期 |
| `LFBK` | 仕入先銀行データ | 引き続き存在 — BPから同期 |
| `LFM1` | 仕入先マスター（購買組織） | 引き続き存在 |
| `LFM2` | 仕入先マスター（購買データ） | 引き続き存在 |

> レガシーの得意先/仕入先テーブルはS/4HANAでも引き続き存在しますが、**二次的なもの**です。BP（`BUT000`）がリーディングマスターデータオブジェクトです。変更はすべてBP経由で行い、CVIが`KNA1`/`LFA1`に同期します。

## 新しいビジネスパートナーテーブル

| 新テーブル | 説明 |
|---|---|
| `BUT000` | ビジネスパートナーマスター（一般データ、中心的なBPレコード） |
| `BUT001` | BPアドレス |
| `BUT100` | BPロール割り当て |
| `BUT020` | BPアドレス（割り当て） |
| `CVI_CUST_LINK` | 同期リンク：BP GUID → KNA1（得意先番号） |
| `CVI_VEND_LINK` | 同期リンク：BP GUID → LFA1（仕入先番号） |

## データ整合性ルール

移行後、以下の条件が成立している必要があります：

```
COUNT(KNA1) = COUNT(CVI_CUST_LINK)
COUNT(LFA1) = COUNT(CVI_VEND_LINK)
```

不一致がある場合は、CVI移行が不完全であることを示します。これは**ハードブロッカー**です — 本番稼働前に解消すること。

## 移行プロセス

### ステップ1 — 移行前チェック
- CVI整合性チェックレポートを実行して、データ品質問題のある得意先/仕入先レコードを特定
- クレンジング：必須フィールドの欠損、ナンバーレンジの重複、無効な住所
- 同一企業が得意先でもあり仕入先でもある場合、`KNA1-LIFNR`（得意先の仕入先番号）と`LFA1-KUNNR`（仕入先の得意先番号）が正しく設定されているか確認

### ステップ2 — BPナンバーレンジの設定
- ターゲットシステムでBPナンバーレンジを定義
- 決定事項：内部ナンバー付与（SAPがBP番号を割り当て）または外部ナンバー付与（得意先/仕入先番号をBP番号として使用）

### ステップ3 — CVI同期実行
- CVI移行プログラムを実行して、既存の全得意先/仕入先のBPレコードを作成
- 各得意先 → ロール `FLCU01`（FI得意先）のBP
- 各仕入先 → ロール `FLVN01`（FI仕入先）のBP

### ステップ4 — 統合（任意だが推奨）
同一の実企業が得意先と仕入先の両方として存在する場合：
- トランザクション **`CVI_LEDH`**（法人データ統合）を使用してペアを特定・統合
- 住所、税データ、銀行データのリーディングエンティティを選択
- **MDSロードコックピット**を使用して統合済みペアを単一BPに同期

### ステップ5 — 検証
- `COUNT(KNA1) = COUNT(CVI_CUST_LINK)`を検証
- `COUNT(LFA1) = COUNT(CVI_VEND_LINK)`を検証
- 主要な得意先/仕入先のBPレコードをスポットチェック — 全データフィールドが正しく移行されていることを確認

## カスタムコードへの影響

以下のようなZコードは要確認です：
- `INSERT INTO KNA1`で直接得意先を作成
- `INSERT INTO LFA1`で直接仕入先を作成
- BP APIを経由せずに`KNA1`/`LFA1`を読み取り

S/4HANAでは：
- BP作成/読み取りには`BAPI_BUPA_CREATE_FROM_DATA`またはCDSビューを使用
- `KNA1`/`LFA1`への直接INSERT不可 — CVIの同期がトリガーされない

## 主要トランザクションリファレンス

| トランザクション | 目的 |
|---|---|
| `BP` | ビジネスパートナーマスターデータ管理（XD01/XK01を代替） |
| `CVI_LEDH` | 法人データ統合 — 得意先+仕入先を1つのBPに統合 |
| `VCVI_CUST_LINK` | CVI得意先リンクテーブルの参照 |
| `VCVI_VEND_LINK` | CVI仕入先リンクテーブルの参照 |

## 参考資料

- SAP Community — CVIコンセプト解説（メンバー投稿）: https://community.sap.com/t5/technology-blog-posts-by-members/s-4hana-busines-partner-customer-vendor-integration-cvi-concept-between/ba-p/13529974
- SAP Community — S/4HANAシステム変換向けCVI FAQ（SAP公式）: https://community.sap.com/t5/enterprise-resource-planning-blog-posts-by-sap/faq-cvi-customer-vendor-integration-for-system-conversion-to-sap-s-4hana/ba-p/13740757
- SAP Community — CVI_LEDHによる得意先と仕入先の統合（メンバー投稿）: https://community.sap.com/t5/enterprise-resource-planning-blog-posts-by-members/sap-s-4hana-business-partner-conversion-merge-customer-and-vendor-using/ba-p/13529974
- 簡素化一覧リスト S/4HANA 2023（公式PDF）: https://help.sap.com/doc/c34b5ef72430484cb4d8895d5edd12af/2023/en-US/SIMPL_OP2023.pdf
