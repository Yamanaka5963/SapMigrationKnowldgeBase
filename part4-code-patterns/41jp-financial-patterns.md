# 41 - 財務コードパターン（BSEG / BKPF → ACDOCA）

> **背景：** S/4HANAでは、ユニバーサルジャーナル（ACDOCA）がFI、CO、AA、MLの転記を単一テーブルに統合します。BSEGは非クラスター化されて透明テーブルになるため、S/4HANAでもSELECTは動作します — ただし、新規コードではACDOCAから直接読み取ることを推奨します。BSEGへの直接INSERT/UPDATEは禁止されています。

---

## パターン1 — FOR ALL ENTRIESによるBSEG読み取り

### ビフォー（ECC）

```abap
DATA: it_bkpf TYPE TABLE OF bkpf,
      it_bseg TYPE TABLE OF bseg.

" ステップ1: 伝票ヘッダーを読み取り
SELECT * FROM bkpf INTO TABLE it_bkpf
  WHERE bukrs = '1000' AND gjahr = '2023'.

" ステップ2: 明細を読み取り — クラスターテーブルBSEGへのFOR ALL ENTRIES
" ECCではBSEGはクラスターテーブルのため、このSELECTはクラスター読み取りのオーバーヘッドが発生
SELECT bukrs belnr gjahr buzei wrbtr INTO TABLE it_bseg
  FROM bseg
  FOR ALL ENTRIES IN it_bkpf
  WHERE bukrs = it_bkpf-bukrs
    AND belnr = it_bkpf-belnr
    AND gjahr = it_bkpf-gjahr.
```

> **変更点：** S/4HANAでは、BSEGは**非クラスター化**されて透明テーブル/ビューになります。`SELECT FROM BSEG FOR ALL ENTRIES`はS/4HANAで正常に動作します。ブラウンフィールドの最小修正は`IS NOT INITIAL`と`ORDER BY`を追加するだけです。新規開発では、ACDOCAを直接読み取る（パターン2）とパフォーマンスが向上します。

### アフター（S/4HANA）— オプションA：最小ブラウンフィールド修正（BSEGの読み取りを維持）

```abap
DATA: it_bkpf TYPE TABLE OF bkpf,
      it_bseg TYPE TABLE OF bseg.

SELECT * FROM bkpf INTO TABLE @it_bkpf
  WHERE bukrs = '1000' AND gjahr = '2023'.

" 駆動テーブルが空でないことを確認
IF it_bkpf IS NOT INITIAL.
  SELECT bukrs belnr gjahr buzei wrbtr
    FROM bseg
    INTO TABLE @it_bseg
    FOR ALL ENTRIES IN @it_bkpf
    WHERE bukrs = @it_bkpf-bukrs
      AND belnr = @it_bkpf-belnr
      AND gjahr = @it_bkpf-gjahr
    ORDER BY PRIMARY KEY.
ENDIF.
```

> **これが動作する理由：** BSEGはS/4HANAではクラスターテーブルではなくなりました。クラスターテーブルへのFOR ALL ENTRIESに関する旧制限はもはや適用されません。`ORDER BY PRIMARY KEY`と`IS NOT INITIAL`ガードの追加だけが必要な変更です。

### アフター（S/4HANA）— オプションB：新規開発 — ACDOCAから直接読み取り

下のパターン2を参照してください。新規コードや大規模な書き直しの場合は、BSEGを完全にバイパスしてACDOCAから読み取ります。

**参照元：** https://community.sap.com/t5/enterprise-resource-planning-blogs-by-members/handling-of-select-statements-on-simplified-tables-during-s-4-hana/ba-p/13535678

---

## パターン2 — ACDOCAからFI明細を直接読み取り（新規コード）

S/4HANAに特化した新規コードでは、ACDOCAから直接読み取ります：

```abap
DATA: lt_acdoca TYPE TABLE OF acdoca.

SELECT rbukrs  " 会社コード
       belnr   " 伝票番号
       gjahr   " 会計年度
       buzei   " 明細番号
       ledger  " 元帳 — マルチ元帳システムで重複を避けるためにフィルタリング
       hsl     " 国内通貨金額（DMBTRに相当）
       wsl     " 取引/伝票通貨金額（WRBTRに相当）
       ksl     " グループ通貨金額
       prctr   " 利益センター
       kostl   " コストセンター
  FROM acdoca
  INTO TABLE @lt_acdoca
  WHERE rbukrs = '1000'
    AND gjahr  = '2023'
    AND blart  = 'RV'    " 請求伝票
    AND ledger = '0L'    " 主要元帳 — マルチ元帳の読み取りが意図的な場合のみこの行を省略
  ORDER BY PRIMARY KEY.
```

> **マルチ元帳の注意：** ACDOCAはすべての有効な元帳の転記を保管します。`LEDGER = '0L'`なしでクエリすると、すべての元帳（主要元帳+拡張元帳）の行が返され、合計が重複します。コードがマルチ元帳データを明示的に処理しない限り、必ず元帳フィルターを追加してください。

**主要フィールドマッピング：**

| ECC（BSEG） | S/4HANA（ACDOCA） | 備考 |
|---|---|---|
| `BUKRS` | `RBUKRS` | 会社コード |
| `DMBTR` | `HSL` | 国内（会社コード）通貨金額 |
| `WRBTR` | `WSL` | 取引/伝票通貨金額 |
| `KOSTL` | `KOSTL` | コストセンター（同名） |
| `PRCTR` | `PRCTR` | 利益センター（同名） |
| *（相当なし）* | `LEDGER` | 元帳ID — 新フィールド、主要元帳には`'0L'`でフィルタリング |

**参照元：** SAP Help — ACDOCAテーブル説明: https://help.sap.com/docs/SAP_S4HANA_ON-PREMISE/c6c3ffd90792427a9fee1a19df5b0925/17a2e2539c70424de10000000a174cb4.html

---

## パターン3 — FI伝票転記：テーブル直接挿入 → BAPI

### ビフォー（ECC）— 絶対禁止

```abap
" 会計テーブルへの直接挿入 — S/4HANAでは禁止
INSERT INTO bkpf VALUES ls_bkpf.
INSERT INTO bseg VALUES ls_bseg.
COMMIT WORK.
```

> **問題点：** S/4HANAでは、転記がユニバーサルジャーナル（ACDOCA）をバイパスします。データが不整合になります。ATCがこれをハードエラーとしてフラグします。

### アフター（S/4HANA）— BAPI_ACC_DOCUMENT_POSTを使用

```abap
DATA: ls_documentheader TYPE bapiache09,
      lt_accountgl      TYPE TABLE OF bapiacgl09,
      lt_currencyamount TYPE TABLE OF bapiaccr09,
      lt_return         TYPE TABLE OF bapiret2.

" 伝票ヘッダーの設定
ls_documentheader-bus_act    = 'RFBU'.
ls_documentheader-username   = sy-uname.
ls_documentheader-header_txt = 'テスト転記'.
ls_documentheader-comp_code  = '1000'.
ls_documentheader-doc_date   = sy-datum.
ls_documentheader-pstng_date = sy-datum.
ls_documentheader-doc_type   = 'SA'.

" lt_accountgl（GL明細）とlt_currencyamount（通貨金額）を設定

CALL FUNCTION 'BAPI_ACC_DOCUMENT_POST'
  EXPORTING
    documentheader = ls_documentheader
  TABLES
    accountgl      = lt_accountgl
    currencyamount = lt_currencyamount
    return         = lt_return.

" コミット前にリターンテーブルのエラーを確認
READ TABLE lt_return WITH KEY type = 'E' TRANSPORTING NO FIELDS.
IF sy-subrc <> 0.
  CALL FUNCTION 'BAPI_TRANSACTION_COMMIT'
    EXPORTING wait = 'X'.
ELSE.
  CALL FUNCTION 'BAPI_TRANSACTION_ROLLBACK'.
ENDIF.
```

**参照元：** https://community.sap.com/t5/application-development-and-automation-blog-posts/document-posting-using-bapi-acc-document-post-with-tax-and-updating-payment/ba-p/13577679

---

## まとめ

| パターン | 分類 | 対応 |
|---|---|---|
| FOR ALL ENTRIESによるBSEGのSELECT（既存ブラウンフィールドコード） | 推奨修正 | `IS NOT INITIAL`ガード + `ORDER BY PRIMARY KEY`を追加 — S/4HANAでそのまま動作 |
| FI明細の新規読み取り | 新規コード | ACDOCAを`LEDGER = '0L'`フィルターで直接使用 |
| BKPF/BSEGへのINSERT/UPDATE | **必須修正** | BAPI_ACC_DOCUMENT_POSTに置き換え |
| 既存BSEGフィールドのUPDATE | **必須修正** | 会計BAPIまたはFI変更関数を使用 |

## 参考資料

- SAP Community — S/4HANAで簡素化されたテーブルのSELECト処理（メンバー投稿）: https://community.sap.com/t5/enterprise-resource-planning-blogs-by-members/handling-of-select-statements-on-simplified-tables-during-s-4-hana/ba-p/13535678
- SAP Community — BAPI_ACC_DOCUMENT_POST with tax and payment（メンバー投稿）: https://community.sap.com/t5/application-development-and-automation-blog-posts/document-posting-using-bapi-acc-document-post-with-tax-and-updating-payment/ba-p/13577679
- SAP Help — ACDOCAユニバーサルジャーナル（公式）: https://help.sap.com/docs/SAP_S4HANA_ON-PREMISE/c6c3ffd90792427a9fee1a19df5b0925/17a2e2539c70424de10000000a174cb4.html
- このKBのACDOCAリファレンス: [31 — ACDOCAユニバーサルジャーナル](../part3-database/31jp-acdoca-universal-journal.md)
