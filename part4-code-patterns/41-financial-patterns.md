# 41 - Financial Code Patterns (BSEG / BKPF → ACDOCA)

> **Context:** In S/4HANA, the Universal Journal (ACDOCA) consolidates FI, CO, AA, and ML postings into a single table. BSEG remains as a compatibility view but direct SELECT from BSEG in custom code should be replaced. Direct INSERT/UPDATE into BSEG is forbidden.

---

## Pattern 1 — Reading BSEG via FOR ALL ENTRIES

### Before (ECC)

```abap
DATA: it_bkpf TYPE TABLE OF bkpf,
      it_bseg TYPE TABLE OF bseg.

" Step 1: Read document headers
SELECT * FROM bkpf INTO TABLE it_bkpf
  WHERE bukrs = '1000' AND gjahr = '2023'.

" Step 2: Read line items — FOR ALL ENTRIES on cluster table BSEG
SELECT bukrs belnr gjahr buzei wrbtr INTO TABLE it_bseg
  FROM bseg
  FOR ALL ENTRIES IN it_bkpf
  WHERE bukrs = it_bkpf-bukrs
    AND belnr = it_bkpf-belnr
    AND gjahr = it_bkpf-gjahr.
```

> **Problem:** BSEG is a cluster table in ECC. In S/4HANA it is declustered, but SAP recommends avoiding direct SELECT and using the provided function module instead.

### After (S/4HANA) — Use FAGL_GET_BSEG_FOR_ALL_ENTRIES

```abap
DATA: it_bkpf TYPE TABLE OF bkpf,
      et_bseg TYPE TABLE OF bseg.

SELECT * FROM bkpf INTO TABLE it_bkpf
  WHERE bukrs = '1000' AND gjahr = '2023'.

" Use the provided function module instead of direct SELECT from BSEG
CALL FUNCTION 'FAGL_GET_BSEG_FOR_ALL_ENTRIES'
  EXPORTING
    it_for_all_entries = it_bkpf[]
  IMPORTING
    et_bseg            = et_bseg
  EXCEPTIONS
    no_data_found      = 1
    OTHERS             = 2.
```

> **Why:** This FM is aware of the S/4HANA data model and reads from the correct underlying structures regardless of whether data is in ACDOCA or legacy tables.

**Source:** https://community.sap.com/t5/enterprise-resource-planning-blogs-by-members/handling-of-select-statements-on-simplified-tables-during-s-4-hana/ba-p/13535678

---

## Pattern 2 — Reading FI Line Items via ACDOCA (New Code)

For new code written specifically for S/4HANA, read directly from ACDOCA:

```abap
DATA: lt_acdoca TYPE TABLE OF acdoca.

SELECT rbukrs  " Company code
       belnr   " Document number
       gjahr   " Fiscal year
       buzei   " Line item
       hsl     " Amount in local currency (replaces DMBTR)
       ksl     " Amount in group currency
       prctr   " Profit center
       kostl   " Cost center
  FROM acdoca
  INTO TABLE @lt_acdoca
  WHERE rbukrs = '1000'
    AND gjahr  = '2023'
    AND blart  = 'RV'   " Billing document
  ORDER BY PRIMARY KEY.
```

**Key field mapping:**

| ECC (BSEG) | S/4HANA (ACDOCA) | Notes |
|---|---|---|
| `BUKRS` | `RBUKRS` | Company code |
| `DMBTR` | `HSL` | Amount in local currency |
| `WRBTR` | `WSL` | Amount in document currency |
| `KOSTL` | `KOSTL` | Cost center (same name) |
| `PRCTR` | `PRCTR` | Profit center (same name) |

**Source:** SAP Help — ACDOCA table description: https://help.sap.com/docs/SAP_S4HANA_ON-PREMISE/c6c3ffd90792427a9fee1a19df5b0925/17a2e2539c70424de10000000a174cb4.html

---

## Pattern 3 — Posting FI Documents: Direct Table Insert → BAPI

### Before (ECC) — NEVER do this

```abap
" Direct insert to accounting tables — FORBIDDEN in S/4HANA
INSERT INTO bkpf VALUES ls_bkpf.
INSERT INTO bseg VALUES ls_bseg.
COMMIT WORK.
```

> **Problem:** In S/4HANA, posting bypasses the Universal Journal (ACDOCA). Data will be inconsistent. ATC flags this as a hard error.

### After (S/4HANA) — Use BAPI_ACC_DOCUMENT_POST

```abap
DATA: ls_documentheader TYPE bapiache09,
      lt_accountgl      TYPE TABLE OF bapiacgl09,
      lt_currencyamount TYPE TABLE OF bapiaccr09,
      lt_return         TYPE TABLE OF bapiret2.

" Build document header
ls_documentheader-bus_act    = 'RFBU'.
ls_documentheader-username   = sy-uname.
ls_documentheader-header_txt = 'Test posting'.
ls_documentheader-comp_code  = '1000'.
ls_documentheader-doc_date   = sy-datum.
ls_documentheader-pstng_date = sy-datum.
ls_documentheader-doc_type   = 'SA'.

" Build GL line items (populate lt_accountgl)
" Build currency amounts (populate lt_currencyamount)

CALL FUNCTION 'BAPI_ACC_DOCUMENT_POST'
  EXPORTING
    documentheader = ls_documentheader
  TABLES
    accountgl      = lt_accountgl
    currencyamount = lt_currencyamount
    return         = lt_return.

" Check return table for errors before committing
READ TABLE lt_return WITH KEY type = 'E' TRANSPORTING NO FIELDS.
IF sy-subrc <> 0.
  CALL FUNCTION 'BAPI_TRANSACTION_COMMIT'
    EXPORTING wait = 'X'.
ELSE.
  CALL FUNCTION 'BAPI_TRANSACTION_ROLLBACK'.
ENDIF.
```

**Source:** https://community.sap.com/t5/application-development-and-automation-blog-posts/document-posting-using-bapi-acc-document-post-with-tax-and-updating-payment/ba-p/13577679

---

## Summary

| Pattern | Classification | Action |
|---|---|---|
| SELECT from BSEG via FOR ALL ENTRIES | Should-Fix | Replace with FAGL_GET_BSEG_FOR_ALL_ENTRIES |
| New reads on FI line items | New code | Use ACDOCA directly |
| INSERT/UPDATE into BKPF/BSEG | **Must-Fix** | Replace with BAPI_ACC_DOCUMENT_POST |
| UPDATE existing BSEG fields | **Must-Fix** | Use accounting BAPIs or FI change functions |

## References

- SAP Community — Handling SELECT on simplified tables during S/4HANA (member post): https://community.sap.com/t5/enterprise-resource-planning-blogs-by-members/handling-of-select-statements-on-simplified-tables-during-s-4-hana/ba-p/13535678
- SAP Community — BAPI_ACC_DOCUMENT_POST with tax and payment (member post): https://community.sap.com/t5/application-development-and-automation-blog-posts/document-posting-using-bapi-acc-document-post-with-tax-and-updating-payment/ba-p/13577679
- SAP Help — ACDOCA Universal Journal (official): https://help.sap.com/docs/SAP_S4HANA_ON-PREMISE/c6c3ffd90792427a9fee1a19df5b0925/17a2e2539c70424de10000000a174cb4.html
- ACDOCA reference in this KB: [31 — ACDOCA Universal Journal](../part3-database/31-acdoca-universal-journal.md)
