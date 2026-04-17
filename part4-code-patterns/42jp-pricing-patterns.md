# 42 - 価格設定コードパターン（KONV → PRCD_ELEMENTS）

> **背景：** S/4HANAでは、クラスターテーブルKONVに格納されていた価格条件が透明テーブルPRCD_ELEMENTSに移行されました。SAPは単純な読み取りには後方互換ビューV_KONVを、書き込み/複雑なアクセスには新しいAPIクラスを提供しています。

---

## パターン1 — 単純な価格条件読み取り：KONV → V_KONV

### ビフォー（ECC）

```abap
DATA: it_konv TYPE TABLE OF konv.

" KONVはECCではクラスターテーブル — SELECTでアクセス
SELECT * FROM konv INTO TABLE it_konv
  WHERE knumv = lv_knumv
    AND kschl = lv_kschl.
```

> **問題点：** KONVはクラスターテーブルです。S/4HANAではPRCD_ELEMENTSに置き換えられます。DMO変換後、KONVからのSELECTは互換ビューを経由しますが、これは非推奨のパスです。

### アフター（S/4HANA）— オプションA：互換ビューV_KONVを使用（単純な読み取り）

```abap
DATA: it_konv TYPE TABLE OF konv.

" V_KONVはPRCD_ELEMENTSにマッピング — フィールド名は同じ
SELECT * FROM v_konv INTO TABLE it_konv
  WHERE knumv = lv_knumv
    AND kschl = lv_kschl
  ORDER BY PRIMARY KEY.
```

> **使用場面：** フィールド名がKONVと同じである単純な読み取り専用アクセス。コード変更が最小限の素早い修正。

**参照元：** https://community.sap.com/t5/technology-blog-posts-by-members/sap-s-4hana-advantage-prcd-elements-simplified-data-fetch/ba-p/13568684

---

## パターン2 — PRCD_ELEMENTSから直接読み取り（推奨新規コード）

```abap
DATA: lt_prcd TYPE TABLE OF prcd_elements.

" 新しい透明テーブルから直接読み取り
SELECT knumv  " 価格設定伝票番号
       kposn  " 条件項目番号
       kschl  " 条件タイプ
       kwert  " 条件値
       waers  " 通貨
  FROM prcd_elements
  INTO TABLE @lt_prcd
  WHERE knumv = @lv_knumv
    AND kschl = @lv_kschl
  ORDER BY PRIMARY KEY.
```

> **KONVとの主な違い：** PRCD_ELEMENTSは透明テーブルです — 標準的なSQLパフォーマンスが適用され、インデックスを作成できます。

**参照元：** https://community.sap.com/t5/technology-blog-posts-by-members/sap-s-4hana-advantage-prcd-elements-simplified-data-fetch/ba-p/13568684

---

## パターン3 — 価格設定APIクラスによる複雑なアクセス

書き込み操作、複雑なアクセス、または価格設定エンジンロジックとの統合の場合：

### ビフォー（ECC）— 直接INSERT/UPDATE

```abap
" S/4HANAでは絶対禁止
INSERT INTO konv VALUES ls_konv.
MODIFY konv FROM TABLE lt_konv.
```

### アフター（S/4HANA）— cl_prc_result_factoryを使用

```abap
DATA: hkomv TYPE TABLE OF komv.

" 新しいAPIを使用して価格条件を読み取り
cl_prc_result_factory=>get_instance(
)->get_prc_result(
)->get_price_element_db(
  EXPORTING
    it_selection_attribute     = VALUE #(
      ( fieldname = 'KNUMV' value = komk-knumv ) )
  IMPORTING
    et_prc_element_classic_format = hkomv ).

" hkomvの処理 — クラシックKOMV内部テーブルと同じ構造
LOOP AT hkomv INTO DATA(ls_komv).
  " ls_komv-kschl、ls_komv-kwertなどにアクセス
ENDLOOP.
```

> **使用場面：** 価格設定エンジンの統合、条件への書き込みアクセス、または価格設定APIのビジネスロジック（再決定、スケール評価）が必要な場合。

**参照元：** https://community.sap.com/t5/technology-blog-posts-by-members/sap-s-4hana-advantage-prcd-elements-simplified-data-fetch/ba-p/13568684

---

## パターン4 — CDSによる税金条件（MWST）アクセス

SDレポートでよく使われるパターン：価格設定からMWST税金条件を読み取る。

### ビフォー（ECC）

```abap
SELECT kawrt FROM konv INTO CORRESPONDING FIELDS OF konv
  WHERE knumv = t_listk-knumv
    AND kposn = t_listp-posnr
    AND kschl = 'MWST'.
```

### アフター（S/4HANA）

```abap
" V_KONVビュー（互換）またはPRCD_ELEMENTSを直接使用
SELECT kawrt FROM v_konv INTO CORRESPONDING FIELDS OF @konv
  WHERE knumv = @t_listk-knumv
    AND kposn = @t_listp-posnr
    AND kschl = 'MWST'
  ORDER BY PRIMARY KEY.
```

**参照元：** https://www.kodyaz.com/sap-abap/s4hana-search-for-database-operations-db-operation-select-found.aspx

---

## まとめ

| パターン | 分類 | 対応 |
|---|---|---|
| KONVからのSELECT（単純読み取り） | 推奨修正 | V_KONVまたはPRCD_ELEMENTSに置き換え |
| KONVへのINSERT/UPDATE | **必須修正** | cl_prc_result_factory APIを使用 |
| 新規価格設定読み取り | 新規コード | PRCD_ELEMENTSを直接使用 |
| 税金条件（MWST）読み取り | 推奨修正 | V_KONVに置き換え |

## 参考資料

- SAP Community — PRCD_ELEMENTSによるデータフェッチの簡素化（メンバー投稿）: https://community.sap.com/t5/technology-blog-posts-by-members/sap-s-4hana-advantage-prcd-elements-simplified-data-fetch/ba-p/13568684
- このKBのプール/クラスターテーブルリファレンス: [33 — プール/クラスター変換](../part3-database/33jp-pool-cluster-conversion.md)
