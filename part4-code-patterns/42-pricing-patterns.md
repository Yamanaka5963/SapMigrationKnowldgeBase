# 42 - Pricing Code Patterns (KONV → PRCD_ELEMENTS)

> **Context:** In S/4HANA, pricing conditions previously stored in the cluster table KONV are now stored in the transparent table PRCD_ELEMENTS. SAP provides backward-compatible view V_KONV for simple reads and a new API class for write/complex access.

---

## Pattern 1 — Simple Pricing Condition Read: KONV → V_KONV

### Before (ECC)

```abap
DATA: it_konv TYPE TABLE OF konv.

" KONV is a cluster table in ECC — accessed via SELECT
SELECT * FROM konv INTO TABLE it_konv
  WHERE knumv = lv_knumv
    AND kschl = lv_kschl.
```

> **Problem:** KONV is a cluster table. In S/4HANA it is replaced by PRCD_ELEMENTS. After DMO conversion, SELECT from KONV is redirected via compatibility view, but this is a deprecated path.

### After (S/4HANA) — Option A: Use compatibility view V_KONV (simple reads)

```abap
DATA: it_konv TYPE TABLE OF konv.

" V_KONV maps to PRCD_ELEMENTS — same field names, transparent access
SELECT * FROM v_konv INTO TABLE it_konv
  WHERE knumv = lv_knumv
    AND kschl = lv_kschl
  ORDER BY PRIMARY KEY.
```

> **When to use:** Simple read-only access where field names are the same as KONV. Quick fix with minimal code change.

**Source:** https://community.sap.com/t5/technology-blog-posts-by-members/sap-s-4hana-advantage-prcd-elements-simplified-data-fetch/ba-p/13568684

---

## Pattern 2 — Direct Read from PRCD_ELEMENTS (Preferred New Code)

```abap
DATA: lt_prcd TYPE TABLE OF prcd_elements.

" Read directly from the new transparent table
SELECT knumv  " Pricing document number
       kposn  " Condition item number
       kschl  " Condition type
       kwert  " Condition value
       waers  " Currency
  FROM prcd_elements
  INTO TABLE @lt_prcd
  WHERE knumv = @lv_knumv
    AND kschl = @lv_kschl
  ORDER BY PRIMARY KEY.
```

> **Key difference from KONV:** PRCD_ELEMENTS is a transparent table — standard SQL performance applies, and indexes can be created.

**Source:** https://community.sap.com/t5/technology-blog-posts-by-members/sap-s-4hana-advantage-prcd-elements-simplified-data-fetch/ba-p/13568684

---

## Pattern 3 — Complex Access via Pricing API Class

For write operations, complex access, or integration with pricing engine logic:

### Before (ECC) — direct INSERT/UPDATE

```abap
" NEVER do this in S/4HANA
INSERT INTO konv VALUES ls_konv.
MODIFY konv FROM TABLE lt_konv.
```

### After (S/4HANA) — Use cl_prc_result_factory

```abap
DATA: hkomv TYPE TABLE OF komv.

" Read pricing conditions using the new API
cl_prc_result_factory=>get_instance(
)->get_prc_result(
)->get_price_element_db(
  EXPORTING
    it_selection_attribute     = VALUE #(
      ( fieldname = 'KNUMV' value = komk-knumv ) )
  IMPORTING
    et_prc_element_classic_format = hkomv ).

" Process hkomv — same structure as classic KOMV internal table
LOOP AT hkomv INTO DATA(ls_komv).
  " Access ls_komv-kschl, ls_komv-kwert, etc.
ENDLOOP.
```

> **When to use:** When you need pricing engine integration, write access to conditions, or when the pricing API's business logic (redetermination, scale evaluation) must be invoked.

**Source:** https://community.sap.com/t5/technology-blog-posts-by-members/sap-s-4hana-advantage-prcd-elements-simplified-data-fetch/ba-p/13568684

---

## Pattern 4 — Accessing Tax Condition (MWST) via CDS

A common pattern in SD reporting: reading the MWST tax condition from pricing:

### Before (ECC)

```abap
SELECT kawrt FROM konv INTO CORRESPONDING FIELDS OF konv
  WHERE knumv = t_listk-knumv
    AND kposn = t_listp-posnr
    AND kschl = 'MWST'.
```

### After (S/4HANA)

```abap
" Use V_KONV view (compatible) or PRCD_ELEMENTS directly
SELECT kawrt FROM v_konv INTO CORRESPONDING FIELDS OF @konv
  WHERE knumv = @t_listk-knumv
    AND kposn = @t_listp-posnr
    AND kschl = 'MWST'
  ORDER BY PRIMARY KEY.
```

**Source:** https://www.kodyaz.com/sap-abap/s4hana-search-for-database-operations-db-operation-select-found.aspx

---

## Summary

| Pattern | Classification | Action |
|---|---|---|
| SELECT from KONV (simple read) | Should-Fix | Replace with V_KONV or PRCD_ELEMENTS |
| INSERT/UPDATE into KONV | **Must-Fix** | Use cl_prc_result_factory API |
| New pricing reads | New code | Use PRCD_ELEMENTS directly |
| Tax condition (MWST) reads | Should-Fix | Replace with V_KONV |

## References

- SAP Community — PRCD_ELEMENTS simplified data fetch (member post): https://community.sap.com/t5/technology-blog-posts-by-members/sap-s-4hana-advantage-prcd-elements-simplified-data-fetch/ba-p/13568684
- SAP Community — KONV to PRCD_ELEMENTS migration details (member post): https://community.sap.com/t5/technology-blog-posts-by-members/konv-to-prcd-elements/ba-p/13568684
- Pool/Cluster table reference in this KB: [33 — Pool/Cluster Conversion](../part3-database/33-pool-cluster-conversion.md)
