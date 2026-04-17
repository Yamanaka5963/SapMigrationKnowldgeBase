# 40 - ABAPコード移行パターン — 概要

> このパートは、S/4HANAへの移行プロジェクトにおけるABAPコードのレビューと改修をサポートすることを目的としています。すべてのパターンは実際のビフォー/アフターのコード例と、参照元URLを含んでいます。

## 目的

SAP ECCからS/4HANAに移行する際、以下の理由からカスタムABAPコードをレビューする必要があります：
- いくつかのコアデータベーステーブルが置き換えまたは再構成されています（BSEG、KONV、MSEG、MARD、KNA1、LFA1）
- プールテーブルとクラスターテーブルは透明テーブルに変換され、直接SQLアクセスのパターンが動作しなくなる場合があります
- SAP HANAはORDER BYを指定しない限りソート順を保証しません（ORDERなしのSELECTは安全ではありません）
- 一部のテーブル構造は物理テーブルではなくCDSビューに移行されました

**ATC（ABAPテストコックピット）**がほとんどの問題を自動的に識別します。このセクションのパターンは、各変更が*なぜ*必要で、*どのように*修正するかを説明します。

## 必須修正 vs. 推奨修正

| カテゴリ | 説明 | 放置した場合のリスク |
|---|---|---|
| **必須修正** | 簡素化テーブル（ACDOCA、PRCD_ELEMENTS、MATDOC）への直接INSERT/UPDATE/DELETE | 実行時エラー、データ破損 |
| **必須修正** | ネイティブSQL経由でのプール/クラスターテーブルへのSELECT | 変換後の構文エラー |
| **必須修正** | BINARY SEARCH結果セットへのORDER BY欠落 | 非決定論的な結果、誤った読み取り |
| **推奨修正** | BSEG/MSEG/MKPF/MARDからのSELECT（後方互換ビューが存在） | 非推奨のパス、潜在的なパフォーマンスリスク |
| **推奨修正** | BP APIの代わりにKNA1/LFA1に直接INSERT | CVIの同期がトリガーされない、不整合 |
| **推奨修正** | ホスト変数なしの旧Open SQL構文 | コンパイラは受け入れるが将来性がない |

> **ブラウンフィールドの注意（推奨修正について）：** 後方互換ビュー（NSDM_V_MARD、V_KONV、S/4HANAのMSEG互換ビューなど）は**本番環境で安全**です — 正しいデータを返します。「推奨修正」とは「このコードに触れるスプリントで移行する」または「新規開発には新しいパスを使用する」という意味です。**本番稼働前に必ず修正しなければならない**という意味ではありません。カットオーバーでは必須修正項目を優先してください。

## セクションマップ

| ファイル | カバーするパターン |
|---|---|
| [41 — 財務パターン](41jp-financial-patterns.md) | BSEG/BKPF読み取り、BAPI経由のFI伝票転記 |
| [42 — 価格設定パターン](42jp-pricing-patterns.md) | KONV → PRCD_ELEMENTS / V_KONV / API |
| [43 — 在庫パターン](43jp-inventory-patterns.md) | MSEG/MKPF/MARD → NSDMビュー |
| [44 — ビジネスパートナーパターン](44jp-bp-master-data-patterns.md) | KNA1/LFA1 → CL_MD_BP_MAINTAIN / BAPI_BUPA |
| [45 — プール/クラスターパターン](45jp-pool-cluster-patterns.md) | STXH/STXL、PCL1/PCL2、VBUK/VBUP |
| [46 — 一般ABAPパターン](46jp-general-abap-patterns.md) | ORDER BY、Open SQLホスト変数、FOR ALL ENTRIES |

## 移行プロジェクトでの使い方

1. ECCソースシステムで**SAP Readiness Check**を実行してカスタムコード量レポートを取得
2. Eclipse/ADTで**S/4HANAルールセットを使用したATC**を実行 — 調査結果をリストとしてエクスポート
3. 単純な一括修正（ORDER BY、構文）には、ATCの**クイックフィックス**を使用して自動適用
4. テーブル置換パターン（BSEG、KONV、MARD、KNA1）には、これらのファイルを手動改修のガイドとして使用
5. ブラウンフィールドの本番稼働前に、サンドボックス/開発用S/4HANAシステムですべての変更を検証

## 参考資料

- SAP Community — S/4HANAシステム変換後のカスタムコード適応（SAP公式）: https://community.sap.com/t5/enterprise-resource-planning-blog-posts-by-sap/semi-automated-custom-code-adaptation-after-sap-s-4hana-system-conversion/ba-p/13359048
- SAP Help — ATCドキュメント: https://help.sap.com/docs/SAP_NETWEAVER_AS_ABAP_FOR_SOH_740/c1dea27e8c984234bdb31ea86cd6b4d4/4ec5711c6e391014adc9fffe4e204223.html
- このKBのカスタムコード改修ガイド: [16 — カスタムコード改修](../part2-pro/16jp-custom-code-remediation.md)
