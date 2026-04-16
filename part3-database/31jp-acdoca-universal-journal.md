# 31 - ACDOCA：ユニバーサルジャーナル

## 概要

ACDOCAはSAP S/4HANA Financeにおける中心的な明細テーブルです。ECCのモジュール別財務転記テーブル（FI、CO、固定資産、マテリアルレジャー）を、単一の統合テーブル「ユニバーサルジャーナル」に置き換えます。

すべての財務転記は、FI・CO・AA個別のテーブルに書き込む代わりに、単一のACDOCAドキュメントとして記録されます。

## ACDOCAに統合されたECCテーブル

| ECCテーブル | モジュール | 説明 |
|---|---|---|
| `BKPF` | FI | 会計ドキュメントヘッダー |
| `BSEG` | FI | 会計ドキュメント明細（FI明細） |
| `COEP` | CO | CO-PA明細（実績+統計、WRTTP 04/11） |
| `COSP` | CO | COオブジェクト実績費用（合計） |
| `COSS` | CO | COオブジェクト統計費用（合計） |
| `ANEP` | FI-AA | 固定資産明細 |
| `ANEA` | FI-AA | 固定資産明細（比例値） |
| `ANLP` | FI-AA | 固定資産期間別値 |
| `ANLC` | FI-AA | 固定資産価値フィールド |
| `MLIT` | ML | マテリアルレジャードキュメント明細 |
| `MLPP` | ML | マテリアルレジャー期末処理 |
| `MLCD` | ML | マテリアルレジャー原価配分 |
| `CKMI1` | ML | 材料ドキュメント向けMLインデックス |
| `BSIM` | ML | 材料ドキュメントのセカンダリインデックス |
| `GLT0` | FI-GL | G/L勘定合計（ACDOCAから導出） |
| `FAGLFLXA` | FI-GL | 新G/L実績明細 |
| `FAGLFLEXT` | FI-GL | 新G/L合計 |

## S/4HANAにおけるECCテーブルの状態

| テーブル | 状態 | 備考 |
|---|---|---|
| `BKPF` | 引き続き存在 | 読み取りアクセスはサポート |
| `BSEG` | 引き続き存在 | 未決明細管理のみに使用 |
| `COEP` / `COSP` / `COSS` | 互換性ビュー | `V_COEP` がACDOCAから読み取り |
| `ANEP` / `ANLP` 等 | 互換性ビュー | ACDOCAから読み取り |
| `GLT0` | 導出 | 必要に応じてACDOCAから計算 |
| `FAGLFLXA` / `FAGLFLEXT` | 置き換え済み | ACDOCAが新しいデータソース |

> 置き換えられたECCテーブルへの直接更新はS/4HANAではサポートされません。すべての転記は標準のSAP API経由で行う必要があります。

## 新しいACDOCA関連テーブル

| テーブル | 説明 |
|---|---|
| `ACDOCA` | ユニバーサルジャーナル明細（中心的な転記テーブル） |
| `ACDOCT` | ユニバーサルジャーナル合計（集計レイヤー） |
| `ACDOCR` | ユニバーサルジャーナル — 定期転記 |
| `ACACTCONF` | 実績原価確認明細 |

## カスタムコードへの影響

以下のようなZプログラム/レポートは改修が必要です：
- `BSEG`、`COEP`、`ANEP`等の置き換えられたテーブルを直接SELECT
- 非標準のファンクションモジュールでFI/COテーブルに直接転記
- `GLT0`や`FAGLFLEXT`の集計合計を使用

対応方法：
- 読み取りには**CDS互換性ビュー**（`V_BSEG_ORI`、`V_COEP`等）を使用
- 書き込みには**SAP BAPI / 転記API**を使用
- 新しいレポートクエリには**ACDOCA**を直接使用（HANAで効率的）

## 移行への影響

- **マテリアルレジャーの有効化はS/4HANAで必須** — 例外なし
- 移行前に元帳決定の設定が必要（SAP Note 2505200）
- ECCの履歴データは変換中にACDOCAへ移行される
- 移行後：元帳/会社コード別のACDOCA合計がレガシー残高と一致することを検証

## 参考資料

- SAP Community — ACDOCAについて知っておくべきこと（メンバー投稿）: https://community.sap.com/t5/enterprise-resource-planning-blog-posts-by-members/all-you-need-to-know-about-universal-journal-acdoca-sap-s-4-hana-2020/ba-p/13545279
- SAP Community — COEP/COSS/COSPのACDOCAへの置き換え（Q&A）: https://community.sap.com/t5/enterprise-resource-planning-q-a/replacing-coep-coss-cosp-with-acdoca/qaq-p/602531
- SAP Community — S/4HANAでのBSEGとBKPFの扱い（Q&A）: https://community.sap.com/t5/enterprise-resource-planning-q-a/in-s4-the-main-table-for-financial-data-is-acdoca-so-how-bseg-and-bkpf-are/qaq-p/13902812
- 簡素化一覧リスト S/4HANA 2023（公式PDF）: https://help.sap.com/doc/c34b5ef72430484cb4d8895d5edd12af/2023/en-US/SIMPL_OP2023.pdf
- SAP Note 2505200 — 移行中の元帳決定（S-User必要）
