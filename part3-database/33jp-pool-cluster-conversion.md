# 33 - プール/クラスターテーブル → 透過テーブル

## 背景

ECCは3種類のデータベーステーブルをサポートしています：
- **透過テーブル** — DBテーブルと1対1でマッピング（標準）
- **プールテーブル** — 複数の論理テーブルを1つの物理DBテーブル（`ATAB`）に格納
- **クラスターテーブル** — 複数の論理テーブルをバイナリエンコードされた1つの物理DBテーブルに格納

SAP HANAは**透過テーブルのみ**をサポートします。システム変換（ブラウンフィールド/シェル）中、SUM/DMOによってすべてのプール/クラスターテーブルが自動的に透過テーブルに変換されます。このプロセスは**デクラスタリング/デプーリング**と呼ばれます。

## ECCの主なプール/クラスターテーブル

### クラスターテーブル

| テーブル | クラスター名 | 説明 | S/4HANAでの置き換え |
|---|---|---|---|
| `KONV` | `KAPOL` | 価格条件レコード | `PRCD_ELEMENTS`（透過テーブル） |
| `BSEG` | `RFBLG` | FIドキュメント明細 | 透過テーブル（データはACDOCAへ移行） |
| `BSEC` | `RFBLG` | FI一時勘定データ | 透過テーブル |
| `BSET` | `RFBLG` | FI税データ | 透過テーブル |
| `BSIS` | `RFBLG` | FIセカンダリインデックス（G/L） | 透過テーブル |
| `STXH` | `STXD` | SAPscriptテキストヘッダー | 透過テーブル |
| `STXL` | `STXD` | SAPscriptテキスト本文 | 透過テーブル |
| `PCL1`〜`PCL4` | HRクラスター | HRペイロール/タイムデータクラスター | 透過テーブル |

### プールテーブル

| テーブル | プール名 | 説明 |
|---|---|---|
| `FUPARAREF` | `FUPARA` | ファンクションモジュールパラメータ参照 |
| `ADCP` | `ADA` | 住所/人物割り当て |
| `CDPOS_STR` | `CDCLS` | 変更ドキュメント文字列値 |
| 各種SYST* | システムプール | その他のシステムテーブル |

> 特定のクライアントシステムのプール/クラスターテーブルの完全なリストを取得するには、SAP Note **2634739** を使用してください。ABAPディクショナリのメタデータテーブル（`DD02L`で`TABCLASS = 'POOL'`または`'CLUSTER'`を検索）に対して実行するSQLクエリが記載されています。

## SUM実行中の変換プロセス

1. SUMがシステム内のすべてのプール/クラスターテーブルを特定
2. SAP HANA上に新しい透過テーブル構造を作成
3. プール/クラスターの物理ストレージから新しい透過テーブルへデータを移行
4. 大容量テーブル（例：`KONV`、`VBFA`）のフィールド変更は**アップタイムフェーズ中**に実行してダウンタイムへの影響を最小化
5. 変換後、プール/クラスターの物理コンテナテーブルは存在しなくなる

**制限事項：** **749フィールドを超える**テーブルは直接変換できません — 変換前に構造変更が必要です。

## KONV → PRCD_ELEMENTS（代表的な変換例）

`KONV`（価格条件）はカスタムコードへの影響が最も大きい変換の一つです。

| | ECC | S/4HANA |
|---|---|---|
| テーブル | `KONV`（クラスター） | `PRCD_ELEMENTS`（透過テーブル） |
| アクセス | `SELECT FROM KONV`で直接アクセス | APIまたはCDSビュー経由 |
| 直接SELECT | 動作する | サポートされない |

**S/4HANAでは以下を使用：**
- API: `cl_prc_result_factory=>get_instance()->get_prc_result()`
- フォールバックCDSビュー: `V_KONV_CDS`（読み取り専用、移行期間中）
- `PRCD_ELEMENTS`への直接書き込みはしないこと — 価格設定APIを使用する

## カスタムコードへの影響

プール/クラスターテーブルへの直接アクセスを行うコードはすべて改修が必要です：

| パターン | ECC | S/4HANAでの対応 |
|---|---|---|
| `SELECT FROM KONV` | 動作する | `PRCD_ELEMENTS`または`V_KONV_CDS`を使用 |
| `SELECT FROM STXH/STXL` | 動作する | `READ_TEXT` / `SAVE_TEXT` FMを使用 |
| `SELECT FROM PCL1/PCL2` | 動作する | HRクラスター読み取りAPIを使用 |
| `SELECT FROM BSEG` | 動作する | 互換性ビューまたはACDOCAを使用 |
| プール/クラスターへの直接INSERT/UPDATE | 動作する | ブロック — 標準APIを使用すること |

## 参考資料

- ABAP Keyword Doc — プール/クラスターテーブルの変換（公式）: https://help.sap.com/doc/abapdocu_752_index_htm/7.52/en-US/abenddic_database_tables_poclutr.htm
- ABAP Keyword Doc — データベーステーブルの変換（公式）: https://help.sap.com/doc/abapdocu_752_index_htm/7.52/en-US/abenddic_database_tables_conv.htm
- ABAP Keyword Doc — プール/クラスターテーブル概要（公式）: https://help.sap.com/doc/abapdocu_750_index_htm/7.50/en-US/abenddic_database_tables_poolclu.htm
- SAP Community — HANA上のクラスター/プールテーブルの神話と真実（メンバー投稿）: https://community.sap.com/t5/technology-blog-posts-by-members/myth-and-truth-about-cluster-pool-tables-on-hana/ba-p/13372144
- SAP Note 2634739 — システム内の全プール/クラスターテーブルを一覧するSQL（S-User必要）
