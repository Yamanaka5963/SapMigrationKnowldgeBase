# 45 - Pool / Cluster Table Patterns (STXH / STXL / PCL1 / PCL2 / VBUK / VBUP)

> **Context:** During DMO/SUM conversion, SAP automatically converts pool and cluster tables to transparent tables. Custom code using these tables via standard ABAP syntax continues to work. However, code using **native SQL** (EXEC SQL) to access pool/cluster tables breaks and must be rewritten.

---

## Pattern 1 — Long Text Access: STXH / STXL → READ_TEXT Function Module

### Before (ECC) — Wrong approach (native SQL)

```abap
" NEVER access STXH/STXL via native SQL — breaks in S/4HANA
EXEC SQL.
  SELECT * FROM stxh WHERE tdname = :lv_name
ENDEXEC.
```

> **Problem:** STXH/STXL are cluster tables in ECC. After conversion to transparent tables, the internal format changes. More importantly, using native SQL to access them was always wrong — use the standard FM.

### Before (ECC) — Correct approach (works in both ECC and S/4HANA)

```abap
DATA: lt_lines TYPE TABLE OF tline.

" Standard function module — works in both ECC and S/4HANA
CALL FUNCTION 'READ_TEXT'
  EXPORTING
    id       = 'ABCD'       " Text ID
    language = 'E'          " Language
    name     = '12345678'   " Text name (e.g. document number)
    object   = 'VBBK'       " Text object (e.g. sales order header)
  TABLES
    lines    = lt_lines
  EXCEPTIONS
    not_found     = 4
    OTHERS        = 8.

IF sy-subrc = 4.
  " Text not found — handle gracefully
ELSEIF sy-subrc <> 0.
  " Unexpected error
ENDIF.
```

### After (S/4HANA) — New Approach: I_TextObjectPlainLongText CDS View

For new development in S/4HANA, SAP provides CDS views for long text access:

> **Release note:** `I_TextObjectPlainLongText` is available from **S/4HANA 1610** onwards. For systems on 1511, use the `READ_TEXT` function module (shown above) — it works in all releases.

```abap
" Read long text via CDS view (S/4HANA preferred approach, 1610+)
SELECT * FROM i_textobjectplainlongtext
  INTO TABLE @DATA(lt_texts)
  WHERE language = 'E'
    AND textobject = 'VBBK'
    AND textname   = '12345678'
    AND textid     = 'ABCD'
  ORDER BY PRIMARY KEY.
```

**Source:** https://community.sap.com/t5/technology-blog-posts-by-members/accessing-long-text-of-sap-master-data-objects-or-fields-in-s4-hana-through/ba-p/13574939

---

## Pattern 2 — HR Payroll Clusters: PCL1 / PCL2

### Before (ECC)

```abap
" Access HR payroll result cluster via IMPORT/EXPORT statements
" This uses the cluster DB interface — NOT native SQL
DATA: st  TYPE pay99_result,
      ert TYPE hrpay99_ert.

IMPORT st ert FROM DATABASE pcl1(b1) ID b1_key.
```

> **Important:** PCL1/PCL2 IMPORT/EXPORT statements use SAP's cluster DB abstraction layer, NOT native SQL. After DMO conversion, the underlying storage is transparent tables but the IMPORT/EXPORT interface remains unchanged. This code **still works** in S/4HANA.

### After (S/4HANA) — Preferred: Use Standard HR Function Modules

```abap
" Preferred: use standard HR functions for payroll data access
CALL FUNCTION 'PYXX_READ_PAYROLL_RESULT'
  EXPORTING
    clusterid                    = 'RD'
    employeenumber               = lv_pernr
    sequencenumber               = '01'
    payroll_area                 = lv_area
  CHANGING
    payroll_result               = ls_result
  EXCEPTIONS
    illegal_isocode_or_clusterid = 1
    error_generating_import      = 2
    OTHERS                       = 3.
```

> **When to migrate:** Only when code uses native SQL (EXEC SQL) to access PCL tables. IMPORT FROM DATABASE pcl1/pcl2 syntax is safe.

**Source:** SAP Help — PCL1/PCL2 cluster access: https://help.sap.com/docs/SAP_S4HANA_ON-PREMISE/c6c3ffd90792427a9fee1a19df5b0925/17a2e2539c70424de10000000a174cb4.html

---

## Pattern 3 — Sales Order Status: VBUK / VBUP → VBAK / LIKP / VBRK

### Before (ECC)

```abap
DATA: ls_vbuk TYPE vbuk,
      ls_vbup TYPE vbup.

" Read sales order header status
SELECT SINGLE * FROM vbuk INTO ls_vbuk
  WHERE vbeln = lv_vbeln.

" Read item status
SELECT SINGLE * FROM vbup INTO ls_vbup
  WHERE vbeln = lv_vbeln
    AND posnr = lv_posnr.
```

> **Background:** In S/4HANA, VBUK and VBUP status fields are incorporated directly into the main document header tables (VBAK for sales orders, LIKP for deliveries, VBRK for billing). The separate status tables still exist as compatibility structures but SAP recommends accessing status fields from the main tables.

### After (S/4HANA) — Read status from header table directly

```abap
DATA: ls_vbak TYPE vbak.

" Status fields are now directly on VBAK
SELECT SINGLE vbeln erdat ernam gbstk lfstk fkstk  " Header + status fields
  FROM vbak
  INTO CORRESPONDING FIELDS OF @ls_vbak
  WHERE vbeln = @lv_vbeln.

" GBSTK = overall processing status
" LFSTK = delivery status
" FKSTK = billing status
```

### After (S/4HANA) — Use VBFA for document flow (still valid)

```abap
" VBFA (document flow table) is unchanged in S/4HANA
SELECT vbelv posnv vbeln posnn vbtyp_n
  FROM vbfa
  INTO TABLE @DATA(lt_vbfa)
  WHERE vbelv IN @so_vbeln
  ORDER BY PRIMARY KEY.
```

**Source:** https://www.kodyaz.com/sap-abap/s4hana-search-for-database-operations-db-operation-select-found.aspx

---

## Summary

| Table | Classification | S/4HANA Status | Action |
|---|---|---|---|
| STXH/STXL (via EXEC SQL) | **Must-Fix** | Transparent, different format | Use READ_TEXT FM or CDS view |
| STXH/STXL (via READ_TEXT FM) | No change needed | Works unchanged | Keep as-is |
| PCL1/PCL2 (via IMPORT FROM DATABASE) | No change needed | Works unchanged | Keep as-is; prefer PYXX_READ_PAYROLL_RESULT for new code |
| PCL1/PCL2 (via EXEC SQL) | **Must-Fix** | Not supported | Rewrite using standard FM |
| VBUK/VBUP (read) | Should-Fix | Still exists as compatibility | Read status from VBAK/LIKP/VBRK |
| VBFA | No change needed | Unchanged in S/4HANA | Keep as-is |

## References

- SAP Community — Accessing long text via CDS in S/4HANA (member post): https://community.sap.com/t5/technology-blog-posts-by-members/accessing-long-text-of-sap-master-data-objects-or-fields-in-s4-hana-through/ba-p/13574939
- SAP Help — PCL cluster tables in S/4HANA: https://help.sap.com/docs/SAP_S4HANA_ON-PREMISE/c6c3ffd90792427a9fee1a19df5b0925/17a2e2539c70424de10000000a174cb4.html
- Pool/Cluster reference in this KB: [33 — Pool/Cluster Conversion](../part3-database/33-pool-cluster-conversion.md)
