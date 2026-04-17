# 41 - 財務コードパターン（BSEG / BKPF → ACDOCA）

> **背景：** S/4HANAでは、ユニバーサルジャーナル（ACDOCA）がFI、CO、AA、MLの転記を単一テーブルに統合します。BSEGは互換ビューとして残りますが、カスタムコードでBSEGを直接SELECTするのは置き換えを推奨します。BSEGへの直接INSERT/UPDATEは禁止されています。

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
SELECT bukrs belnr gjahr buzei wrbtr INTO TABLE it_bseg
  FROM bseg
  FOR ALL ENTRIES IN it_bkpf
  WHERE bukrs = it_bkpf-bukrs
    AND belnr = it_bkpf-belnr
    AND gjahr = it_bkpf-gjahr.
```

> **問題点：** BSEGはECCではクラスターテーブルです。S/4HANAでは非クラスター化されますが、SAPは直接SELECTを避け、提供されているファンクションモジュールを使用することを推奨しています。

### アフター（S/4HANA）— FAGL_GET_BSEG_FOR_ALL_ENTRIESを使用

```abap
DATA: it_bkpf TYPE TABLE OF bkpf,
      et_bseg TYPE TABLE OF bseg.

SELECT * FROM bkpf INTO TABLE it_bkpf
  WHERE bukrs = '1000' AND gjahr = '2023'.

" BSEGを直接SELECTせず、提供されているファンクションモジュールを使用
CALL FUNCTION 'FAGL_GET_BSEG_FOR_ALL_ENTRIES'
  EXPORTING
    it_for_all_entries = it_bkpf[]
  IMPORTING
    et_bseg            = et_bseg
  EXCEPTIONS
    no_data_found      = 1
    OTHERS             = 2.
```

> **理由：** このFMはS/4HANAデータモデルを認識しており、データがACDOCAまたはレガシーテーブルのどちらにあっても正しい基礎構造から読み取ります。

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
       hsl     " 国内通貨金額（DMBTRに代替）
       ksl     " グループ通貨金額
       prctr   " 利益センター
       kostl   " コストセンター
  FROM acdoca
  INTO TABLE @lt_acdoca
  WHERE rbukrs = '1000'
    AND gjahr  = '2023'
    AND blart  = 'RV'   " 請求伝票
  ORDER BY PRIMARY KEY.
```

**主要フィールドマッピング：**

| ECC（BSEG） | S/4HANA（ACDOCA） | 備考 |
|---|---|---|
| `BUKRS` | `RBUKRS` | 会社コード |
| `DMBTR` | `HSL` | 国内通貨金額 |
| `WRBTR` | `WSL` | 伝票通貨金額 |
| `KOSTL` | `KOSTL` | コストセンター（同名） |
| `PRCTR` | `PRCTR` | 利益センター（同名） |

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
| FOR ALL ENTRIESによるBSEGのSELECT | 推奨修正 | FAGL_GET_BSEG_FOR_ALL_ENTRIESに置き換え |
| FI明細の新規読み取り | 新規コード | ACDOCAを直接使用 |
| BKPF/BSEGへのINSERT/UPDATE | **必須修正** | BAPI_ACC_DOCUMENT_POSTに置き換え |
| 既存BSEGフィールドのUPDATE | **必須修正** | 会計BAPIまたはFI変更関数を使用 |

## 参考資料

- SAP Community — S/4HANAで簡素化されたテーブルのSELECト処理（メンバー投稿）: https://community.sap.com/t5/enterprise-resource-planning-blogs-by-members/handling-of-select-statements-on-simplified-tables-during-s-4-hana/ba-p/13535678
- SAP Community — BAPI_ACC_DOCUMENT_POST with tax and payment（メンバー投稿）: https://community.sap.com/t5/application-development-and-automation-blog-posts/document-posting-using-bapi-acc-document-post-with-tax-and-updating-payment/ba-p/13577679
- SAP Help — ACDOCAユニバーサルジャーナル（公式）: https://help.sap.com/docs/SAP_S4HANA_ON-PREMISE/c6c3ffd90792427a9fee1a19df5b0925/17a2e2539c70424de10000000a174cb4.html
- このKBのACDOCAリファレンス: [31 — ACDOCAユニバーサルジャーナル](../part3-database/31jp-acdoca-universal-journal.md)
