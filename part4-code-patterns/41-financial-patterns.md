# 41 - Financial Code Patterns (BSEG / BKPF → ACDOCA)

> **Context:** In S/4HANA, the Universal Journal (ACDOCA) consolidates FI, CO, AA, and ML postings into a single table. BSEG is declustered to a transparent table, so SELECT from BSEG works in S/4HANA — but reading from ACDOCA directly is preferred for new code. Direct INSERT/UPDATE into BSEG is forbidden.

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
" In ECC, BSEG is a cluster table and this SELECT carries cluster-read overhead
SELECT bukrs belnr gjahr buzei wrbtr INTO TABLE it_bseg
  FROM bseg
  FOR ALL ENTRIES IN it_bkpf
  WHERE bukrs = it_bkpf-bukrs
    AND belnr = it_bkpf-belnr
    AND gjahr = it_bkpf-gjahr.
```

> **What changed:** In S/4HANA, BSEG is **declustered** — it is now a transparent table/view. `SELECT FROM BSEG FOR ALL ENTRIES` works correctly in S/4HANA. The minimal brownfield fix is to add `IS NOT INITIAL` and `ORDER BY`. For new development, reading ACDOCA directly (Pattern 2) gives better performance.

### After (S/4HANA) — Option A: Minimal brownfield fix (keep BSEG reads)

```abap
DATA: it_bkpf TYPE TABLE OF bkpf,
      it_bseg TYPE TABLE OF bseg.

SELECT * FROM bkpf INTO TABLE @it_bkpf
  WHERE bukrs = '1000' AND gjahr = '2023'.

" Guard against empty driving table
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

> **Why this works now:** BSEG is no longer a cluster table in S/4HANA. The old restriction against FOR ALL ENTRIES on cluster tables no longer applies. Adding `ORDER BY PRIMARY KEY` and the `IS NOT INITIAL` guard are the only required changes.

### After (S/4HANA) — Option B: New development — read ACDOCA directly

See Pattern 2 below. For new code or significant rewrites, bypass BSEG entirely and read from ACDOCA.

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
       ledger  " Ledger — filter to avoid duplicates in multi-ledger systems
       hsl     " Amount in local currency (= DMBTR equivalent)
       wsl     " Amount in transaction/document currency (= WRBTR equivalent)
       ksl     " Amount in group currency
       prctr   " Profit center
       kostl   " Cost center
  FROM acdoca
  INTO TABLE @lt_acdoca
  WHERE rbukrs = '1000'
    AND gjahr  = '2023'
    AND blart  = 'RV'    " Billing document
    AND ledger = '0L'    " Leading ledger — omit this line only if multi-ledger reads are intentional
  ORDER BY PRIMARY KEY.
```

> **Multi-ledger note:** ACDOCA stores postings for all active ledgers. Without `LEDGER = '0L'` the query returns rows for every ledger (leading + extension), causing duplicate amounts in totals. Always add a ledger filter unless your code explicitly handles multi-ledger data.

**Key field mapping:**

| ECC (BSEG) | S/4HANA (ACDOCA) | Notes |
|---|---|---|
| `BUKRS` | `RBUKRS` | Company code |
| `DMBTR` | `HSL` | Amount in local (company code) currency |
| `WRBTR` | `WSL` | Amount in transaction/document currency |
| `KOSTL` | `KOSTL` | Cost center (same name) |
| `PRCTR` | `PRCTR` | Profit center (same name) |
| *(no equivalent)* | `LEDGER` | Ledger ID — new field, filter to `'0L'` for leading ledger |

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
| SELECT FROM BSEG FOR ALL ENTRIES (existing brownfield code) | Should-Fix | Add `IS NOT INITIAL` guard + `ORDER BY PRIMARY KEY` — works as-is in S/4HANA |
| New reads on FI line items | New code | Use ACDOCA with `LEDGER = '0L'` filter |
| INSERT/UPDATE into BKPF/BSEG | **Must-Fix** | Replace with BAPI_ACC_DOCUMENT_POST |
| UPDATE existing BSEG fields | **Must-Fix** | Use accounting BAPIs or FI change functions |

## References

- SAP Community — Handling SELECT on simplified tables during S/4HANA (member post): https://community.sap.com/t5/enterprise-resource-planning-blogs-by-members/handling-of-select-statements-on-simplified-tables-during-s-4-hana/ba-p/13535678
- SAP Community — BAPI_ACC_DOCUMENT_POST with tax and payment (member post): https://community.sap.com/t5/application-development-and-automation-blog-posts/document-posting-using-bapi-acc-document-post-with-tax-and-updating-payment/ba-p/13577679
- SAP Help — ACDOCA Universal Journal (official): https://help.sap.com/docs/SAP_S4HANA_ON-PREMISE/c6c3ffd90792427a9fee1a19df5b0925/17a2e2539c70424de10000000a174cb4.html
- ACDOCA reference in this KB: [31 — ACDOCA Universal Journal](../part3-database/31-acdoca-universal-journal.md)
