# 46 - 一般ABAPパターン（ORDER BY / Open SQL / ホスト変数）

> **背景：** SAP HANAはORDER BYを指定しない限り行の順序を保証しません。つまり、ORDER BYなしでデータベースレベルの順序付けに依存していたコード（OracleやMSSQLで一般的）は、HANAで誤った結果を生成する可能性があります。また、ホスト変数（@）を使用した最新のOpen SQL構文は、S/4HANAの新規コードで推奨されます。

---

## パターン1 — BINARY SEARCHの前にORDER BYが欠落

ブラウンフィールド移行で最も一般的なATCの検出項目です。

### ビフォー（ECC — HANAで問題あり）

```abap
DATA: it_mkpf TYPE TABLE OF mkpf.

" ORDER BYなしのSELECT — HANAでは結果の順序が未定義
SELECT * FROM mkpf INTO TABLE it_mkpf
  WHERE mblnr IN so_mblnr.

" BINARY SEARCHはソート済みテーブルが必要 — 順序が異なる場合にHANAで失敗
READ TABLE it_mkpf WITH KEY mblnr = lv_mblnr BINARY SEARCH.
```

> **問題点：** Oracle/DB2/MSSQLでは、テーブルがORDER BYなしで主キー順に行を返すことがよくありました。SAP HANAはそのような保証を行いません。ソートされていないテーブルでのBINARY SEARCHは誤った結果を返します（実行時エラーなし、誤ったデータ）。

### アフター（S/4HANA）— オプションA：ORDER BY PRIMARY KEYを追加

```abap
DATA: it_mkpf TYPE TABLE OF mkpf.

" ORDER BY PRIMARY KEYを追加してソート順を保証
SELECT * FROM mkpf INTO TABLE it_mkpf
  WHERE mblnr IN so_mblnr
  ORDER BY PRIMARY KEY.

READ TABLE it_mkpf WITH KEY mblnr = lv_mblnr BINARY SEARCH.
```

### アフター（S/4HANA）— オプションB：SORTED TABLEを使用

```abap
" ソートテーブル型を使用 — ORDER BY不要、挿入時に常にソート済み
DATA: it_mkpf TYPE SORTED TABLE OF mkpf
                WITH UNIQUE KEY mblnr mjahr zeile.

SELECT * FROM mkpf INTO TABLE it_mkpf
  WHERE mblnr IN so_mblnr.

" ソートテーブルのREAD TABLEは自動的にバイナリサーチを使用
READ TABLE it_mkpf WITH KEY mblnr = lv_mblnr.
```

**参照元：** https://community.sap.com/t5/enterprise-resource-planning-blog-posts-by-sap/semi-automated-custom-code-adaptation-after-sap-s-4hana-system-conversion/ba-p/13359048

---

## パターン2 — Open SQL：旧構文 → ホスト変数付きの新構文

ATCはS/4HANAコンテキストで旧Open SQL構文をフラグします。実行時エラーではありませんが、インライン宣言には新構文が必要で、新規コードのSAP標準です。

### ビフォー（ECC — 旧構文）

```abap
DATA: lv_matnr TYPE matnr VALUE '000000000000001234',
      lv_werks TYPE werks_d VALUE '1000',
      ls_mara  TYPE mara.

SELECT SINGLE * FROM mara INTO ls_mara
  WHERE matnr = lv_matnr.

SELECT * FROM marc INTO TABLE lt_marc
  WHERE matnr = lv_matnr
    AND werks = lv_werks.
```

### アフター（S/4HANA — @ホスト変数付きの新構文）

```abap
DATA: lv_matnr TYPE matnr VALUE '000000000000001234',
      lv_werks TYPE werks_d VALUE '1000'.

" ホスト変数には@プレフィックスを使用（インライン宣言に必須）
SELECT SINGLE * FROM mara
  INTO @DATA(ls_mara)
  WHERE matnr = @lv_matnr.

" @DATA(...)インライン宣言を使用したインラインテーブル宣言
SELECT * FROM marc
  INTO TABLE @DATA(lt_marc)
  WHERE matnr = @lv_matnr
    AND werks = @lv_werks
  ORDER BY PRIMARY KEY.
```

> **@プレフィックスが必要な理由：** `INTO @DATA(...)`インライン宣言を使用する場合に必須です。また、Open SQLパーサーでのホスト変数の使用を明確にします。

**参照元：** https://community.sap.com/t5/enterprise-resource-planning-blog-posts-by-sap/semi-automated-custom-code-adaptation-after-sap-s-4hana-system-conversion/ba-p/13359048

---

## パターン3 — FOR ALL ENTRIES：空テーブルチェック

HANAで全テーブルスキャンを引き起こすクラシックなABAPのバグ — ECCから変わらないが重要な検出項目：

### ビフォー（ECCとS/4HANA両方で問題あり）

```abap
DATA: lt_bkpf TYPE TABLE OF bkpf,
      lt_bseg TYPE TABLE OF bseg.

" lt_bkpfが空の場合、FOR ALL ENTRIESはすべての行を返す — 全テーブルスキャン
SELECT * FROM bseg INTO TABLE lt_bseg
  FOR ALL ENTRIES IN lt_bkpf
  WHERE bukrs = lt_bkpf-bukrs
    AND belnr = lt_bkpf-belnr.
```

### アフター（正しいパターン）

```abap
DATA: lt_bkpf TYPE TABLE OF bkpf,
      lt_bseg TYPE TABLE OF bseg.

" 駆動テーブルが空でないことを常に確認
IF lt_bkpf IS NOT INITIAL.
  SELECT * FROM bseg INTO TABLE lt_bseg
    FOR ALL ENTRIES IN lt_bkpf
    WHERE bukrs = lt_bkpf-bukrs
      AND belnr = lt_bkpf-belnr
    ORDER BY PRIMARY KEY.
ENDIF.
```

**参照元：** https://community.sap.com/t5/enterprise-resource-planning-blog-posts-by-sap/semi-automated-custom-code-adaptation-after-sap-s-4hana-system-conversion/ba-p/13359048

---

## パターン4 — ネイティブSQL（EXEC SQL）によるSAPテーブルアクセス

### ビフォー（ECC）

```abap
" ECCでパフォーマンスのためにネイティブSQLを使用することがあった
DATA: lv_count TYPE i.

EXEC SQL.
  SELECT COUNT(*) INTO :lv_count FROM sapsr3.mara WHERE mandt = :sy-mandt
ENDEXEC.
```

> **問題点：** ネイティブSQLはABAPのOpen SQLレイヤーをバイパスします。HANAでは、テーブルストレージ（カラムストアvs行ストア、内部フォーマット）が従来のデータベースと異なります。SAPのスキーマ名が異なる場合があります。ネイティブSQL経由でアクセスされたプール/クラスターテーブルは変換後に完全に失敗します。

### アフター（S/4HANA）

```abap
" ABAP Open SQLを使用 — オプティマイザーがHANA固有の実行を処理
SELECT COUNT(*) FROM mara INTO @lv_count.
```

> **例外：** AMDP（ABAPマネージドデータベースプロシージャ）とCDSテーブル関数は、パフォーマンスが重要なシナリオで本当に必要な場合にHANAネイティブSQLを書くためのサポートされた方法です。

---

## パターン5 — ワイドテーブルでのSELECT *を避ける

HANAのカラムストアは列選択的な読み取りに最適化されています。ワイドテーブルで全列（*）を選択することは、行ストアデータベースと比較してHANAでは特にコストがかかります。

### ビフォー（ECC）

```abap
" 200フィールドのテーブルでSELECT *はすべての列をカラムストアから読み取る
SELECT * FROM vbak INTO TABLE lt_vbak
  WHERE erdat = sy-datum.
```

### アフター（S/4HANA）

```abap
" 必要な列のみを選択 — HANAでのパフォーマンスが劇的に向上
SELECT vbeln erdat auart kunnr netwr waerk
  FROM vbak
  INTO TABLE @DATA(lt_vbak_slim)
  WHERE erdat = @sy-datum
  ORDER BY PRIMARY KEY.
```

---

## ATCクイックフィックスの対象範囲

以下のパターンはEclipse/ADTのATCクイックフィックスで自動修正可能です：

| 問題 | ATCクイックフィックス |
|---|---|
| ORDER BYの欠落 | あり — `ORDER BY PRIMARY KEY`を追加 |
| @ホスト変数の欠落 | あり — @プレフィックスを追加 |
| フィールドリストなしのSELECT（SELECT *） | 部分的 — フラグのみ、手動修正 |
| 空のFOR ALL ENTRIESテーブルチェック | なし — 手動修正が必要 |
| EXEC SQLブロック | なし — 手動書き直しが必要 |

**参照元：** https://community.sap.com/t5/enterprise-resource-planning-blog-posts-by-sap/semi-automated-custom-code-adaptation-after-sap-s-4hana-system-conversion/ba-p/13359048

---

## まとめ

| パターン | 分類 | 対応 |
|---|---|---|
| ORDER BYなしのSELECT + BINARY SEARCH | **必須修正** | ORDER BY PRIMARY KEYを追加またはSORTED TABLEを使用 |
| 旧Open SQL構文（@なし） | 推奨修正 | @プレフィックスを追加；@DATA(...)インラインを使用 |
| 空テーブルへのFOR ALL ENTRIES | **必須修正** | IS NOT INITIALチェックを追加 |
| ネイティブSQL（EXEC SQL） | **必須修正** | Open SQLに書き直し；HANAネイティブにはAMDPを使用 |
| ワイドテーブルでのSELECT * | 推奨修正 | 必要な列のみ選択 |

## 参考資料

- SAP Community — S/4HANAシステム変換後のカスタムコード適応（SAP公式）: https://community.sap.com/t5/enterprise-resource-planning-blog-posts-by-sap/semi-automated-custom-code-adaptation-after-sap-s-4hana-system-conversion/ba-p/13359048
- SAP Community — ABAPとHANAのコーディングガイドライン（SAP公式）: https://community.sap.com/t5/application-development-and-automation-blog-posts/abap-and-hana-coding-guidelines/ba-p/13268891
- このKBのカスタムコード改修ガイド: [16 — カスタムコード改修](../part2-pro/16jp-custom-code-remediation.md)
