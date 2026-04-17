# 44 - ビジネスパートナーマスターデータパターン（KNA1 / LFA1 → BP）

> **背景：** S/4HANAでは、独立した得意先（KNA1）と仕入先（LFA1）のマスターレコードが統合されたビジネスパートナー（BP）モデルに置き換えられます。KNA1とLFA1はテーブルとして残りますが、CVI（顧客仕入先統合）によってBP（BUT000）と同期して管理される二次的なテーブルとなります。カスタムコードでKNA1またはLFA1に直接INSERT/UPDATEするとCVIの同期が壊れ、データの不整合が発生します。

---

## パターン1 — 得意先データの読み取り（動作するが非推奨パス）

### ビフォー（ECC）

```abap
DATA: ls_kna1 TYPE kna1.

" 得意先マスターデータを読み取り
SELECT SINGLE * FROM kna1 INTO ls_kna1
  WHERE kunnr = '0000001234'.
```

> **注意：** KNA1からのSELECTはS/4HANAでも動作します。KNA1はCVIによって同期されて維持されます。ただし、新規コードでは、正規のマスターデータレコードへのアクセスにCDSビューまたはBP APIを使用してください。

### アフター（S/4HANA）— 推奨：ビジネスパートナーCDSビューを使用

```abap
" 標準CDSビュー経由でBPマスターデータにアクセス
SELECT bp~partner      " BP番号
       bp~bu_sort1     " 検索キー1（名前）
       bp~name_org1    " 組織名
       bp~country      " 国
  FROM but000 AS bp
  INTO TABLE @DATA(lt_bp)
  WHERE bp~partner = '1234'
  ORDER BY PRIMARY KEY.

" 得意先固有データには、CVIリンクを結合
SELECT kna1~kunnr kna1~name1 kna1~land1 but000~bu_sort1
  FROM kna1
  INNER JOIN cvi_cust_link ON cvi_cust_link~customer_id = kna1~kunnr
  INNER JOIN but000        ON but000~partner             = cvi_cust_link~bp_idnr
  INTO TABLE @DATA(lt_customers)
  WHERE kna1~land1 = 'DE'
  ORDER BY PRIMARY KEY.
```

**参照元：** https://community.sap.com/t5/application-development-and-automation-blog-posts/create-business-partner-via-api-class-cl-md-bp-maintain/ba-p/13540478

---

## パターン2 — 得意先/仕入先の作成：直接INSERT → BP API

### ビフォー（ECC）— S/4HANAでは絶対禁止

```abap
" 禁止：CVIの同期をバイパス
DATA: ls_kna1 TYPE kna1,
      ls_lfa1 TYPE lfa1.

" 得意先を直接作成
INSERT INTO kna1 VALUES ls_kna1.
" 仕入先を直接作成
INSERT INTO lfa1 VALUES ls_lfa1.
COMMIT WORK.
```

> **問題点：** KNA1/LFA1はCVIによって維持されます。直接INSERTはCVIをバイパスします — 対応するBPレコードが作成されません。COUNT(KNA1) ≠ COUNT(CVI_CUST_LINK)チェックが失敗します。これは本番稼働前に解消すべき**ハードブロッカー**です。

### アフター（S/4HANA）— オプションA：CL_MD_BP_MAINTAIN（推奨クラス）

```abap
DATA: t_bp     TYPE TABLE OF cvis_ei_extern,
      t_return TYPE TABLE OF cvis_message.

" t_bpにBPデータ構造を設定...
" ロールを設定：FI得意先はFLCU01、FI仕入先はFLVN01

CALL METHOD cl_md_bp_maintain=>maintain
  EXPORTING
    i_data     = t_bp
    i_test_run = abap_false    " テスト実行にはabap_trueを設定
  IMPORTING
    e_return   = t_return.

" エラーのt_returnを確認
LOOP AT t_return INTO DATA(ls_return) WHERE type = 'E'.
  " エラー処理
ENDLOOP.

COMMIT WORK AND WAIT.
```

### アフター（S/4HANA）— オプションB：BAPI_BUPA_CREATE_FROM_DATA

```abap
DATA: ls_bapibus1006_head     TYPE bapibus1006_head,
      ls_bapibus1006_central  TYPE bapibus1006_central,
      ls_bapibus1006_address  TYPE bapibus1006_address,
      lv_businesspartner      TYPE bu_partner,
      lt_return               TYPE TABLE OF bapiret2.

ls_bapibus1006_head-partnercategory = '2'.  " 組織
ls_bapibus1006_head-partnergroup    = 'CUST01'.

ls_bapibus1006_central-searchterm1 = 'TESTCO'.

ls_bapibus1006_address-firstname  = 'テスト'.
ls_bapibus1006_address-lastname   = '株式会社'.
ls_bapibus1006_address-country    = 'JP'.

CALL FUNCTION 'BAPI_BUPA_CREATE_FROM_DATA'
  EXPORTING
    partnercategory = ls_bapibus1006_head-partnercategory
    partnergroup    = ls_bapibus1006_head-partnergroup
    centraldata     = ls_bapibus1006_central
    addressdata     = ls_bapibus1006_address
  IMPORTING
    businesspartner = lv_businesspartner
    return          = lv_return
  TABLES
    return          = lt_return.

READ TABLE lt_return WITH KEY type = 'E' TRANSPORTING NO FIELDS.
IF sy-subrc <> 0.
  CALL FUNCTION 'BAPI_TRANSACTION_COMMIT' EXPORTING wait = 'X'.
ENDIF.
```

**参照元：** https://community.sap.com/t5/application-development-and-automation-blog-posts/create-business-partner-via-api-class-cl-md-bp-maintain/ba-p/13540478

---

## パターン3 — 得意先/仕入先データの更新 → BP API

### ビフォー（ECC）

```abap
" 直接UPDATE — S/4HANAでは禁止
UPDATE kna1 SET name1 = '新しい会社名' WHERE kunnr = '0000001234'.
```

### アフター（S/4HANA）

```abap
" BAPI_BUPA_CENTRAL_CHANGEまたはMaintain APIを使用
CALL FUNCTION 'BAPI_BUPA_CENTRAL_CHANGE'
  EXPORTING
    businesspartner      = lv_bp_number
    centraldata          = ls_centraldata
    centraldata_x        = ls_centraldata_x   " 変更フラグ構造
  TABLES
    return               = lt_return.

CALL FUNCTION 'BAPI_TRANSACTION_COMMIT' EXPORTING wait = 'X'.
```

---

## パターン4 — CVIリンク整合性チェック

CVIの同期が完了していることを確認するためのクエリ：

```abap
" CVIの整合性を確認するためにHANA StudioまたはABAPのSELECTとして実行
" COUNT(KNA1) = COUNT(CVI_CUST_LINK)でなければならない

DATA: lv_kna1_count      TYPE i,
      lv_cvi_cust_count  TYPE i,
      lv_lfa1_count      TYPE i,
      lv_cvi_vend_count  TYPE i.

SELECT COUNT(*) FROM kna1 INTO lv_kna1_count.
SELECT COUNT(*) FROM cvi_cust_link INTO lv_cvi_cust_count.
SELECT COUNT(*) FROM lfa1 INTO lv_lfa1_count.
SELECT COUNT(*) FROM cvi_vend_link INTO lv_cvi_vend_count.

IF lv_kna1_count <> lv_cvi_cust_count.
  " 得意先のCVI同期が不完全 — 本番稼働前に調査
ENDIF.
IF lv_lfa1_count <> lv_cvi_vend_count.
  " 仕入先のCVI同期が不完全 — 本番稼働前に調査
ENDIF.
```

**参照元：** このKBのCVIリファレンス: [34 — CVIビジネスパートナー](../part3-database/34jp-cvi-business-partner.md)

---

## まとめ

| パターン | 分類 | 対応 |
|---|---|---|
| KNA1/LFA1からのSELECT（読み取り） | 推奨修正 | 新規コードにはBUT000またはCDSビューを使用 |
| KNA1/LFA1へのINSERT | **必須修正** | CL_MD_BP_MAINTAINまたはBAPI_BUPA_CREATE_FROM_DATAを使用 |
| KNA1/LFA1へのUPDATE | **必須修正** | BAPI_BUPA_CENTRAL_CHANGEを使用 |
| CVIリンク数の不一致 | **ブロッカー** | 本番稼働前に解消；CVI移行ログを確認 |

## 参考資料

- SAP Community — APIクラスCL_MD_BP_MAINTAINを使用したビジネスパートナーの作成（メンバー投稿）: https://community.sap.com/t5/application-development-and-automation-blog-posts/create-business-partner-via-api-class-cl-md-bp-maintain/ba-p/13540478
- SAP Community — 得意先/仕入先とビジネスパートナー間のCVIコンセプト（メンバー投稿）: https://community.sap.com/t5/technology-blog-posts-by-members/s-4hana-busines-partner-customer-vendor-integration-cvi-concept-between/ba-p/13529974
- このKBのCVIリファレンス: [34 — CVIビジネスパートナー](../part3-database/34jp-cvi-business-partner.md)
