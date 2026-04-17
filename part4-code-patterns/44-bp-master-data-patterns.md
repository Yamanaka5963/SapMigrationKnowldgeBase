# 44 - Business Partner Master Data Patterns (KNA1 / LFA1 → BP)

> **Context:** S/4HANA replaces the separate Customer (KNA1) and Vendor (LFA1) master records with a unified Business Partner (BP) model. KNA1 and LFA1 still exist as tables but are now secondary — they are maintained by CVI (Customer-Vendor Integration) in sync with BP (BUT000). Direct INSERT/UPDATE into KNA1 or LFA1 in custom code breaks CVI sync and causes data inconsistencies.

---

## Pattern 1 — Reading Customer Data (still works, but deprecated path)

### Before (ECC)

```abap
DATA: ls_kna1 TYPE kna1.

" Read customer master data
SELECT SINGLE * FROM kna1 INTO ls_kna1
  WHERE kunnr = '0000001234'.
```

> **Note:** SELECT from KNA1 still works in S/4HANA. KNA1 is maintained in sync by CVI. However, for new code, use CDS views or the BP API to access the canonical master data record.

### After (S/4HANA) — Preferred: Use Business Partner CDS view

```abap
" Access BP master data via standard CDS views
SELECT bp~partner      " BP number
       bp~bu_sort1     " Sort field 1 (name)
       bp~name_org1    " Organization name
       bp~country      " Country
  FROM but000 AS bp
  INTO TABLE @DATA(lt_bp)
  WHERE bp~partner = '1234'
  ORDER BY PRIMARY KEY.

" For customer-specific data, join with CVI link
SELECT kna1~kunnr kna1~name1 kna1~land1 but000~bu_sort1
  FROM kna1
  INNER JOIN cvi_cust_link ON cvi_cust_link~customer_id = kna1~kunnr
  INNER JOIN but000        ON but000~partner             = cvi_cust_link~bp_idnr
  INTO TABLE @DATA(lt_customers)
  WHERE kna1~land1 = 'DE'
  ORDER BY PRIMARY KEY.
```

**Source:** https://community.sap.com/t5/application-development-and-automation-blog-posts/create-business-partner-via-api-class-cl-md-bp-maintain/ba-p/13540478

---

## Pattern 2 — Creating a Customer / Vendor: Direct INSERT → BP API

### Before (ECC) — NEVER do this in S/4HANA

```abap
" FORBIDDEN: bypasses CVI synchronization
DATA: ls_kna1 TYPE kna1,
      ls_lfa1 TYPE lfa1.

" Creating customer directly
INSERT INTO kna1 VALUES ls_kna1.
" Creating vendor directly
INSERT INTO lfa1 VALUES ls_lfa1.
COMMIT WORK.
```

> **Problem:** KNA1/LFA1 are now maintained by CVI. Direct INSERT bypasses CVI — the corresponding BP record is never created. COUNT(KNA1) ≠ COUNT(CVI_CUST_LINK) check will fail. This is a **hard blocker** for go-live.

### After (S/4HANA) — Option A: CL_MD_BP_MAINTAIN (recommended class)

```abap
DATA: t_bp     TYPE TABLE OF cvis_ei_extern,
      t_return TYPE TABLE OF cvis_message.

" Populate t_bp with BP data structure...
" Set role: FLCU01 for FI Customer, FLVN01 for FI Vendor

CALL METHOD cl_md_bp_maintain=>maintain
  EXPORTING
    i_data     = t_bp
    i_test_run = abap_false    " Set to abap_true for dry run first
  IMPORTING
    e_return   = t_return.

" Check t_return for errors
LOOP AT t_return INTO DATA(ls_return) WHERE type = 'E'.
  " Handle error
ENDLOOP.

COMMIT WORK AND WAIT.
```

### After (S/4HANA) — Option B: BAPI_BUPA_CREATE_FROM_DATA

```abap
DATA: ls_bapibus1006_head     TYPE bapibus1006_head,
      ls_bapibus1006_central  TYPE bapibus1006_central,
      ls_bapibus1006_address  TYPE bapibus1006_address,
      lv_businesspartner      TYPE bu_partner,
      lt_return               TYPE TABLE OF bapiret2.

ls_bapibus1006_head-partnercategory = '2'.  " Organization
ls_bapibus1006_head-partnergroup    = 'CUST01'.

ls_bapibus1006_central-searchterm1 = 'TESTCO'.

ls_bapibus1006_address-firstname  = 'Test'.
ls_bapibus1006_address-lastname   = 'Company'.
ls_bapibus1006_address-country    = 'DE'.

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

**Source:** https://community.sap.com/t5/application-development-and-automation-blog-posts/create-business-partner-via-api-class-cl-md-bp-maintain/ba-p/13540478

---

## Pattern 3 — Updating Customer/Vendor Data → BP API

### Before (ECC)

```abap
" Direct UPDATE — FORBIDDEN in S/4HANA
UPDATE kna1 SET name1 = 'New Name' WHERE kunnr = '0000001234'.
```

### After (S/4HANA)

```abap
" Use BAPI_BUPA_CENTRAL_CHANGE or the Maintain API
CALL FUNCTION 'BAPI_BUPA_CENTRAL_CHANGE'
  EXPORTING
    businesspartner      = lv_bp_number
    centraldata          = ls_centraldata
    centraldata_x        = ls_centraldata_x   " Change flag structure
  TABLES
    return               = lt_return.

CALL FUNCTION 'BAPI_TRANSACTION_COMMIT' EXPORTING wait = 'X'.
```

---

## Pattern 4 — Checking CVI Link Consistency

Use this query to verify CVI synchronization is complete:

```abap
" Run this in HANA Studio or as an ABAP SELECT to verify CVI integrity
" COUNT(KNA1) must equal COUNT(CVI_CUST_LINK)

DATA: lv_kna1_count      TYPE i,
      lv_cvi_cust_count  TYPE i,
      lv_lfa1_count      TYPE i,
      lv_cvi_vend_count  TYPE i.

SELECT COUNT(*) FROM kna1 INTO lv_kna1_count.
SELECT COUNT(*) FROM cvi_cust_link INTO lv_cvi_cust_count.
SELECT COUNT(*) FROM lfa1 INTO lv_lfa1_count.
SELECT COUNT(*) FROM cvi_vend_link INTO lv_cvi_vend_count.

IF lv_kna1_count <> lv_cvi_cust_count.
  " CVI sync incomplete for customers — investigate before go-live
ENDIF.
IF lv_lfa1_count <> lv_cvi_vend_count.
  " CVI sync incomplete for vendors — investigate before go-live
ENDIF.
```

**Source:** CVI reference in this KB: [34 — CVI Business Partner](../part3-database/34-cvi-business-partner.md)

---

## Summary

| Pattern | Classification | Action |
|---|---|---|
| SELECT from KNA1/LFA1 (read) | Should-Fix | Use BUT000 or CDS view for new code |
| INSERT into KNA1/LFA1 | **Must-Fix** | Use CL_MD_BP_MAINTAIN or BAPI_BUPA_CREATE_FROM_DATA |
| UPDATE into KNA1/LFA1 | **Must-Fix** | Use BAPI_BUPA_CENTRAL_CHANGE |
| CVI link count mismatch | **Blocker** | Resolve before go-live; check CVI migration logs |

## References

- SAP Community — Create Business Partner via API class CL_MD_BP_MAINTAIN (member post): https://community.sap.com/t5/application-development-and-automation-blog-posts/create-business-partner-via-api-class-cl-md-bp-maintain/ba-p/13540478
- SAP Community — CVI concept between Customer/Vendor and Business Partner (member post): https://community.sap.com/t5/technology-blog-posts-by-members/s-4hana-busines-partner-customer-vendor-integration-cvi-concept-between/ba-p/13529974
- CVI reference in this KB: [34 — CVI Business Partner](../part3-database/34-cvi-business-partner.md)
