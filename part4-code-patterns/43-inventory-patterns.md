# 43 - Inventory Code Patterns (MSEG / MKPF / MARD → NSDM)

> **Context:** In S/4HANA, the inventory management data model changed significantly. MKPF+MSEG are replaced by MATDOC. MARD (stock per storage location) is replaced by NSDM_V_MARD. Backward-compatible views exist so existing SELECT statements still execute — but they are deprecated and should be updated for performance and future compatibility.

---

## Pattern 1 — Reading Goods Movement Documents: MSEG → NSDM_E_MSEG

### Before (ECC)

```abap
DATA: wa_mseg TYPE mseg,
      it_mseg TYPE TABLE OF mseg.

" Read a single goods movement document line
SELECT SINGLE * FROM mseg INTO CORRESPONDING FIELDS OF wa_mseg
  WHERE mblnr = '1234567890'
    AND mjahr = '2023'
    AND zeile = '0001'.

" OR: read all lines for a document
SELECT * FROM mseg INTO TABLE it_mseg
  WHERE mblnr = '1234567890'
    AND mjahr = '2023'.
```

> **Important:** In S/4HANA, MSEG is internally a **CDS view** pointing to MATDOC. `SELECT FROM MSEG` still works and returns correct data — it is backward-compatible. However, it is a deprecated access path.

### After (S/4HANA) — Preferred: Use NSDM_E_MSEG

```abap
DATA: it_nsdm TYPE TABLE OF nsdm_e_mseg.

" NSDM_E_MSEG is a CDS view entity backed by MATDOC (the physical table)
SELECT * FROM nsdm_e_mseg INTO TABLE @DATA(it_mseg_new)
  WHERE mblnr = '1234567890'
    AND mjahr = '2023'
  ORDER BY PRIMARY KEY.
```

> **NSDM_E_MSEG** is a CDS view entity set that reads from MATDOC — the underlying physical transparent table. Do not try to create secondary indexes on NSDM_E_MSEG directly; index on MATDOC instead. `NSDM_V_MSEG` maps MATDOC columns back to classic MSEG field names for backward compatibility.

**Source:** https://community.sap.com/t5/enterprise-resource-planning-blog-posts-by-sap/sap-s-4hana-inventory-management-tables-new-simplified-data-model-nsdm/ba-p/13497469

---

## Pattern 2 — Stock per Storage Location: MARD → NSDM_V_MARD

### Before (ECC)

```abap
DATA: it_mard TYPE TABLE OF mard.

" Get stock for a specific material and plant
SELECT * FROM mard INTO TABLE it_mard
  WHERE matnr = '000000000000001234'
    AND werks = '1000'.
```

> **Problem:** MARD is replaced by NSDM in S/4HANA. Stock quantities are now calculated dynamically from MATDOC movements, not stored as static snapshots in a physical MARD table.
>
> **Important — calculated stock fields:** Fields like `LABST` (unrestricted stock), `EINME` (stock in transit), `INSME` (quality inspection stock) are computed on-the-fly. Consequences: (1) You cannot UPDATE stock directly via MARD — always use `BAPI_GOODSMVT_CREATE`. (2) If your code compares MARD stock values against other sources (e.g., interface files), recalibrate the comparison logic — values are now computed, not stored snapshots, and rounding or timing differences may appear.

### After (S/4HANA) — Option A: Compatibility view NSDM_V_MARD

```abap
DATA: it_mard TYPE TABLE OF mard.

" NSDM_V_MARD provides the same field names as MARD
SELECT * FROM nsdm_v_mard INTO TABLE it_mard
  WHERE matnr = '000000000000001234'
    AND werks = '1000'
  ORDER BY PRIMARY KEY.
```

### After (S/4HANA) — Option B: New entity NSDM_E_MARD (modern code)

```abap
" NSDM_E_MARD is the S/4HANA native entity
SELECT * FROM nsdm_e_mard INTO TABLE @DATA(it_mard_modern)
  WHERE matnr = '000000000000001234'
    AND werks = '1000'
  ORDER BY PRIMARY KEY.
```

**Source:** https://community.sap.com/t5/enterprise-resource-planning-blog-posts-by-sap/sap-s-4hana-inventory-management-tables-new-simplified-data-model-nsdm/ba-p/13497469

---

## Pattern 3 — Material Document Header: MKPF → NSDM

### Before (ECC)

```abap
DATA: it_mkpf TYPE TABLE OF mkpf.

" Read material document headers
SELECT * FROM mkpf INTO TABLE it_mkpf
  WHERE mblnr = '1234567890'
    AND mjahr = '2023'.
```

> **Note:** MKPF header data is merged into MATDOC in S/4HANA. There is no direct NSDM_E_MKPF — header fields are available on NSDM_E_MSEG (MATDOC has one row per item, with header fields repeated).

### After (S/4HANA) — Read header fields from NSDM_E_MSEG

```abap
" Header-level fields (mblnr, mjahr, budat, usnam etc.) are on NSDM_E_MSEG
SELECT DISTINCT mblnr mjahr budat usnam blart
  FROM nsdm_e_mseg
  INTO TABLE @DATA(lt_headers)
  WHERE mblnr = '1234567890'
    AND mjahr = '2023'
  ORDER BY PRIMARY KEY.
```

**Source:** https://community.sap.com/t5/enterprise-resource-planning-blog-posts-by-sap/sap-s-4hana-inventory-management-tables-new-simplified-data-model-nsdm/ba-p/13497469

---

## Pattern 4 — Writing Goods Movements: Direct INSERT → BAPI_GOODSMVT_CREATE

### Before (ECC) — NEVER do this

```abap
" Direct write to inventory tables — FORBIDDEN in S/4HANA
INSERT INTO mseg VALUES ls_mseg.
INSERT INTO mkpf VALUES ls_mkpf.
COMMIT WORK.
```

### After (S/4HANA) — Use BAPI_GOODSMVT_CREATE

```abap
DATA: ls_goodsmvt_header TYPE bapi2017_gm_head_01,
      lt_goodsmvt_item   TYPE TABLE OF bapi2017_gm_item_create,
      ls_goodsmvt_code   TYPE bapi2017_gm_code,
      lt_return          TYPE TABLE OF bapiret2,
      lv_matdoc          TYPE mblnr,
      lv_matdoc_year     TYPE mjahr.

ls_goodsmvt_code-gm_code = '01'.  " Goods receipt for purchase order
ls_goodsmvt_header-pstng_date = sy-datum.
ls_goodsmvt_header-doc_date   = sy-datum.

" Populate lt_goodsmvt_item with movement line items...

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

## NSDM View/Entity Reference

| Classic Table | S/4HANA Replacement | Notes |
|---|---|---|
| `MKPF` | Header fields in `NSDM_E_MSEG` | No separate header entity |
| `MSEG` | `NSDM_E_MSEG` (physical) / `NSDM_V_MSEG` (compatibility) | MSEG is a CDS view in S/4HANA |
| `MARD` | `NSDM_E_MARD` (physical) / `NSDM_V_MARD` (compatibility) | Stock calculated dynamically |
| `MARDH` | `NSDM_V_MARDH` | Historical stock |
| `MCHB` | `NSDM_V_MCHB` | Batch stock |
| `MSKU` | `NSDM_V_MSKU` | Special stock |

## Summary

| Pattern | Classification | Action |
|---|---|---|
| SELECT from MSEG (existing code) | Should-Fix | Replace with NSDM_E_MSEG or NSDM_V_MSEG |
| SELECT from MARD | Should-Fix | Replace with NSDM_V_MARD or NSDM_E_MARD |
| SELECT from MKPF | Should-Fix | Read header fields from NSDM_E_MSEG |
| INSERT/UPDATE into MSEG/MKPF | **Must-Fix** | Use BAPI_GOODSMVT_CREATE |
| New inventory reads | New code | Use NSDM_E_* tables directly |

## References

- SAP Community — S/4HANA Inventory Management: New Simplified Data Model (NSDM) (by SAP): https://community.sap.com/t5/enterprise-resource-planning-blog-posts-by-sap/sap-s-4hana-inventory-management-tables-new-simplified-data-model-nsdm/ba-p/13497469
- MATDOC reference in this KB: [32 — MATDOC Inventory Consolidation](../part3-database/32-matdoc-inventory.md)
