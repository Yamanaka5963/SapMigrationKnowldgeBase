# 43 - 在庫コードパターン（MSEG / MKPF / MARD → NSDM）

> **背景：** S/4HANAでは、在庫管理のデータモデルが大幅に変更されました。MKPF+MSEGはMATDOCに置き換えられます。MARD（保管場所別在庫）はNSDM_V_MARDに置き換えられます。後方互換ビューが存在するため、既存のSELECT文は引き続き実行されますが、非推奨とされており、パフォーマンスと将来の互換性のために更新する必要があります。

---

## パターン1 — 入出庫伝票読み取り：MSEG → NSDM_E_MSEG

### ビフォー（ECC）

```abap
DATA: wa_mseg TYPE mseg,
      it_mseg TYPE TABLE OF mseg.

" 入出庫伝票の単一明細を読み取り
SELECT SINGLE * FROM mseg INTO CORRESPONDING FIELDS OF wa_mseg
  WHERE mblnr = '1234567890'
    AND mjahr = '2023'
    AND zeile = '0001'.

" または：伝票のすべての明細を読み取り
SELECT * FROM mseg INTO TABLE it_mseg
  WHERE mblnr = '1234567890'
    AND mjahr = '2023'.
```

> **重要：** S/4HANAでは、MSEGは内部的にMATDOCを指す**CDSビュー**です。`SELECT FROM MSEG`は引き続き動作し、正しいデータを返します — 後方互換性があります。ただし、これは非推奨のアクセスパスです。

### アフター（S/4HANA）— 推奨：NSDM_E_MSEGを使用

```abap
DATA: it_nsdm TYPE TABLE OF nsdm_e_mseg.

" NSDM_E_MSEGはMATDOC（物理テーブル）を基盤とするCDSビューエンティティ
SELECT * FROM nsdm_e_mseg INTO TABLE @DATA(it_mseg_new)
  WHERE mblnr = '1234567890'
    AND mjahr = '2023'
  ORDER BY PRIMARY KEY.
```

> **NSDM_E_MSEG**はMATDOCのデータにアクセスするCDSビューエンティティセットです。MATDOCが基礎となる物理的な透明テーブルです。NSDM_E_MSEGに直接セカンダリインデックスを作成しようとしないでください — インデックスはMATDOCに作成します。`NSDM_V_MSEG`はMATDOCの列をクラシックMSEGフィールド名にマッピングして後方互換性を提供します。

**参照元：** https://community.sap.com/t5/enterprise-resource-planning-blog-posts-by-sap/sap-s-4hana-inventory-management-tables-new-simplified-data-model-nsdm/ba-p/13497469

---

## パターン2 — 保管場所別在庫：MARD → NSDM_V_MARD

### ビフォー（ECC）

```abap
DATA: it_mard TYPE TABLE OF mard.

" 特定の品目と工場の在庫を取得
SELECT * FROM mard INTO TABLE it_mard
  WHERE matnr = '000000000000001234'
    AND werks = '1000'.
```

> **問題点：** MARDはS/4HANAでNSDMに置き換えられます。在庫数量は静的なスナップショットとしてMARDの物理テーブルに格納されるのではなく、MATDOCの移動から動的に計算されるようになります。
>
> **重要 — 計算された在庫フィールド：** `LABST`（制限なし在庫）、`EINME`（輸送中在庫）、`INSME`（品質検査在庫）などのフィールドはオンザフライで計算されます。その影響：（1）MARD経由で直接在庫をUPDATEできません — 必ず`BAPI_GOODSMVT_CREATE`を使用してください。（2）コードがMARD在庫値を他のソース（例：インターフェースファイル）と比較している場合、比較ロジックを再調整してください — 値は保存済みスナップショットではなく計算値のため、端数処理やタイミングの差異が生じる場合があります。

### アフター（S/4HANA）— オプションA：互換ビューNSDM_V_MARDを使用

```abap
DATA: it_mard TYPE TABLE OF mard.

" NSDM_V_MARDはMARDと同じフィールド名を提供
SELECT * FROM nsdm_v_mard INTO TABLE it_mard
  WHERE matnr = '000000000000001234'
    AND werks = '1000'
  ORDER BY PRIMARY KEY.
```

### アフター（S/4HANA）— オプションB：新エンティティNSDM_E_MARDを使用（最新コード）

```abap
" NSDM_E_MARDはS/4HANAネイティブのエンティティ
SELECT * FROM nsdm_e_mard INTO TABLE @DATA(it_mard_modern)
  WHERE matnr = '000000000000001234'
    AND werks = '1000'
  ORDER BY PRIMARY KEY.
```

**参照元：** https://community.sap.com/t5/enterprise-resource-planning-blog-posts-by-sap/sap-s-4hana-inventory-management-tables-new-simplified-data-model-nsdm/ba-p/13497469

---

## パターン3 — 材料伝票ヘッダー：MKPF → NSDM

### ビフォー（ECC）

```abap
DATA: it_mkpf TYPE TABLE OF mkpf.

" 材料伝票ヘッダーを読み取り
SELECT * FROM mkpf INTO TABLE it_mkpf
  WHERE mblnr = '1234567890'
    AND mjahr = '2023'.
```

> **注意：** MKPFのヘッダーデータはS/4HANAのMATDOCに統合されます。NSDM_E_MKPFは独立して存在しません — ヘッダーフィールドはNSDM_E_MSEG上で利用可能です（MATDOCは1行/明細で、ヘッダーフィールドが繰り返されます）。

### アフター（S/4HANA）— NSDM_E_MSEGからヘッダーフィールドを読み取り

```abap
" ヘッダーレベルフィールド（mblnr、mjahr、budat、usnamなど）はNSDM_E_MSEG上にある
SELECT DISTINCT mblnr mjahr budat usnam blart
  FROM nsdm_e_mseg
  INTO TABLE @DATA(lt_headers)
  WHERE mblnr = '1234567890'
    AND mjahr = '2023'
  ORDER BY PRIMARY KEY.
```

**参照元：** https://community.sap.com/t5/enterprise-resource-planning-blog-posts-by-sap/sap-s-4hana-inventory-management-tables-new-simplified-data-model-nsdm/ba-p/13497469

---

## パターン4 — 入出庫の書き込み：直接INSERT → BAPI_GOODSMVT_CREATE

### ビフォー（ECC）— 絶対禁止

```abap
" 在庫テーブルへの直接書き込み — S/4HANAでは禁止
INSERT INTO mseg VALUES ls_mseg.
INSERT INTO mkpf VALUES ls_mkpf.
COMMIT WORK.
```

### アフター（S/4HANA）— BAPI_GOODSMVT_CREATEを使用

```abap
DATA: ls_goodsmvt_header TYPE bapi2017_gm_head_01,
      lt_goodsmvt_item   TYPE TABLE OF bapi2017_gm_item_create,
      ls_goodsmvt_code   TYPE bapi2017_gm_code,
      lt_return          TYPE TABLE OF bapiret2,
      lv_matdoc          TYPE mblnr,
      lv_matdoc_year     TYPE mjahr.

ls_goodsmvt_code-gm_code = '01'.  " 発注書に対する入荷
ls_goodsmvt_header-pstng_date = sy-datum.
ls_goodsmvt_header-doc_date   = sy-datum.

" lt_goodsmvt_itemに移動明細を設定...

CALL FUNCTION 'BAPI_GOODSMVT_CREATE'
  EXPORTING
    goodsmvt_header = ls_goodsmvt_header
    goodsmvt_code   = ls_goodsmvt_code
  IMPORTING
    materialdocument     = lv_matdoc
    matdocumentyear      = lv_matdoc_year
  TABLES
    goodsmvt_item   = lt_goodsmvt_item
    return          = lt_return.

READ TABLE lt_return WITH KEY type = 'E' TRANSPORTING NO FIELDS.
IF sy-subrc <> 0.
  CALL FUNCTION 'BAPI_TRANSACTION_COMMIT' EXPORTING wait = 'X'.
ELSE.
  CALL FUNCTION 'BAPI_TRANSACTION_ROLLBACK'.
ENDIF.
```

---

## NSDMビュー/エンティティ参照

| 旧テーブル | S/4HANA代替 | 備考 |
|---|---|---|
| `MKPF` | `NSDM_E_MSEG`のヘッダーフィールド | 独立したヘッダーエンティティなし |
| `MSEG` | `NSDM_E_MSEG`（物理） / `NSDM_V_MSEG`（互換） | MSEGはS/4HANAではCDSビュー |
| `MARD` | `NSDM_E_MARD`（物理） / `NSDM_V_MARD`（互換） | 在庫は動的に計算 |
| `MARDH` | `NSDM_V_MARDH` | 過去在庫 |
| `MCHB` | `NSDM_V_MCHB` | バッチ別在庫 |
| `MSKU` | `NSDM_V_MSKU` | 特殊在庫 |

## まとめ

| パターン | 分類 | 対応 |
|---|---|---|
| MSEGからのSELECT（既存コード） | 推奨修正 | NSDM_E_MSEGまたはNSDM_V_MSEGに置き換え |
| MARDからのSELECT | 推奨修正 | NSDM_V_MARDまたはNSDM_E_MARDに置き換え |
| MKPFからのSELECT | 推奨修正 | NSDM_E_MSEGからヘッダーフィールドを読み取り |
| MSEG/MKPFへのINSERT/UPDATE | **必須修正** | BAPI_GOODSMVT_CREATEを使用 |
| 新規在庫読み取り | 新規コード | NSDM_E_*テーブルを直接使用 |

## 参考資料

- SAP Community — S/4HANA在庫管理：新簡素化データモデル（NSDM）（SAP公式）: https://community.sap.com/t5/enterprise-resource-planning-blog-posts-by-sap/sap-s-4hana-inventory-management-tables-new-simplified-data-model-nsdm/ba-p/13497469
- このKBのMATDOCリファレンス: [32 — MATDOC在庫管理統合](../part3-database/32jp-matdoc-inventory.md)
