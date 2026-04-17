# 46 - General ABAP Patterns (ORDER BY / Open SQL / Host Variables)

> **Context:** SAP HANA does not guarantee row order unless ORDER BY is specified. This means code that relied on database-level ordering without ORDER BY (common on Oracle and MSSQL) may produce incorrect results on HANA. Additionally, modern Open SQL syntax with host variables (@) is preferred for new S/4HANA code.

---

## Pattern 1 — Missing ORDER BY Before BINARY SEARCH

This is the most common ATC finding in brownfield migrations.

### Before (ECC — problematic on HANA)

```abap
DATA: it_mkpf TYPE TABLE OF mkpf.

" SELECT without ORDER BY — result order is undefined on HANA
SELECT * FROM mkpf INTO TABLE it_mkpf
  WHERE mblnr IN so_mblnr.

" BINARY SEARCH requires sorted table — will fail on HANA if order differs
READ TABLE it_mkpf WITH KEY mblnr = lv_mblnr BINARY SEARCH.
```

> **Problem:** On Oracle/DB2/MSSQL, tables often happened to return rows in primary key order without ORDER BY. SAP HANA makes no such guarantee. BINARY SEARCH on an unsorted table returns wrong results silently (no runtime error, wrong data).

### After (S/4HANA) — Option A: Add ORDER BY PRIMARY KEY

```abap
DATA: it_mkpf TYPE TABLE OF mkpf.

" Add ORDER BY PRIMARY KEY to guarantee sort order
SELECT * FROM mkpf INTO TABLE it_mkpf
  WHERE mblnr IN so_mblnr
  ORDER BY PRIMARY KEY.

READ TABLE it_mkpf WITH KEY mblnr = lv_mblnr BINARY SEARCH.
```

### After (S/4HANA) — Option B: Use SORTED TABLE

```abap
" Use a sorted table type — no ORDER BY needed, always sorted on insert
DATA: it_mkpf TYPE SORTED TABLE OF mkpf
                WITH UNIQUE KEY mblnr mjahr zeile.

SELECT * FROM mkpf INTO TABLE it_mkpf
  WHERE mblnr IN so_mblnr.

" READ TABLE on a sorted table automatically uses binary search
READ TABLE it_mkpf WITH KEY mblnr = lv_mblnr.
```

**Source:** https://community.sap.com/t5/enterprise-resource-planning-blog-posts-by-sap/semi-automated-custom-code-adaptation-after-sap-s-4hana-system-conversion/ba-p/13359048

---

## Pattern 2 — Open SQL: Old Syntax → New Syntax with Host Variables

ATC flags old Open SQL syntax in S/4HANA context. While not a runtime error, new syntax is required for inline declarations and is the SAP standard for new code.

### Before (ECC — old syntax)

```abap
DATA: lv_matnr TYPE matnr VALUE '000000000000001234',
      lv_werks TYPE werks_d VALUE '1000',
      ls_mara  TYPE mara.

SELECT SINGLE * FROM mara INTO ls_mara
  WHERE matnr = lv_matnr.

SELECT * FROM marc INTO TABLE lt_marc
  WHERE matnr = lv_matnr
    AND werks = lv_werks.
```

### After (S/4HANA — new syntax with @ host variables)

```abap
DATA: lv_matnr TYPE matnr VALUE '000000000000001234',
      lv_werks TYPE werks_d VALUE '1000'.

" Use @ prefix for host variables (required for inline declarations)
SELECT SINGLE * FROM mara
  INTO @DATA(ls_mara)
  WHERE matnr = @lv_matnr.

" Inline table declaration with @DATA(...)
SELECT * FROM marc
  INTO TABLE @DATA(lt_marc)
  WHERE matnr = @lv_matnr
    AND werks = @lv_werks
  ORDER BY PRIMARY KEY.
```

> **Why the @ prefix:** Mandatory when using `INTO @DATA(...)` inline declarations. Also makes host variable usage unambiguous in the Open SQL parser.

**Source:** https://community.sap.com/t5/enterprise-resource-planning-blog-posts-by-sap/semi-automated-custom-code-adaptation-after-sap-s-4hana-system-conversion/ba-p/13359048

---

## Pattern 3 — FOR ALL ENTRIES: Empty Table Check

A classic ABAP bug that causes full table scans on HANA — unchanged from ECC but important to catch:

### Before (problematic in both ECC and S/4HANA)

```abap
DATA: lt_bkpf TYPE TABLE OF bkpf,
      lt_bseg TYPE TABLE OF bseg.

" If lt_bkpf is empty, FOR ALL ENTRIES returns ALL rows — full table scan
SELECT * FROM bseg INTO TABLE lt_bseg
  FOR ALL ENTRIES IN lt_bkpf
  WHERE bukrs = lt_bkpf-bukrs
    AND belnr = lt_bkpf-belnr.
```

### After (correct pattern)

```abap
DATA: lt_bkpf TYPE TABLE OF bkpf,
      lt_bseg TYPE TABLE OF bseg.

" Always check that the driving table is not empty
IF lt_bkpf IS NOT INITIAL.
  SELECT * FROM bseg INTO TABLE lt_bseg
    FOR ALL ENTRIES IN lt_bkpf
    WHERE bukrs = lt_bkpf-bukrs
      AND belnr = lt_bkpf-belnr
    ORDER BY PRIMARY KEY.
ENDIF.
```

**Source:** https://community.sap.com/t5/enterprise-resource-planning-blog-posts-by-sap/semi-automated-custom-code-adaptation-after-sap-s-4hana-system-conversion/ba-p/13359048

---

## Pattern 4 — Native SQL (EXEC SQL) Access to SAP Tables

### Before (ECC)

```abap
" Native SQL was sometimes used for performance in ECC
DATA: lv_count TYPE i.

EXEC SQL.
  SELECT COUNT(*) INTO :lv_count FROM sapsr3.mara WHERE mandt = :sy-mandt
  " Note: 'sapsr3' is the typical ECC schema prefix.
  " In S/4HANA the schema name is 'SAP<SID>' — this explicit prefix breaks on S/4HANA.
ENDEXEC.
```

> **Problem:** Native SQL bypasses the ABAP Open SQL layer. On HANA, table storage (column store vs row store, internal format) differs from traditional databases. SAP schema names may differ. Pool/cluster tables accessed via native SQL fail completely post-conversion.

### After (S/4HANA)

```abap
" Use ABAP Open SQL — the optimizer handles HANA-specific execution
SELECT COUNT(*) FROM mara INTO @lv_count.
```

> **Exception:** AMDP (ABAP Managed Database Procedures) and CDS table functions are the supported way to write HANA-native SQL when truly needed for performance-critical scenarios.

---

## Pattern 5 — Avoiding SELECT * on Wide Tables

HANA's columnar store is optimized for column-selective reads. Selecting all columns (*) on wide tables is especially costly on HANA compared to row-store databases.

### Before (ECC)

```abap
" SELECT * on a 200-field table reads all columns from column store
SELECT * FROM vbak INTO TABLE lt_vbak
  WHERE erdat = sy-datum.
```

### After (S/4HANA)

```abap
" Select only needed columns — dramatic performance improvement on HANA
SELECT vbeln erdat auart kunnr netwr waerk
  FROM vbak
  INTO TABLE @DATA(lt_vbak_slim)
  WHERE erdat = @sy-datum
  ORDER BY PRIMARY KEY.
```

---

## ATC Quick Fix Coverage

The following patterns are automatically fixable by ATC Quick Fixes in Eclipse/ADT:

| Issue | ATC Quick Fix Available |
|---|---|
| Missing ORDER BY | Yes — adds `ORDER BY PRIMARY KEY` |
| Missing @ host variables | Yes — adds @ prefix |
| SELECT without field list (SELECT *) | Partial — flags, manual fix |
| Empty FOR ALL ENTRIES table check | No — manual fix required |
| EXEC SQL blocks | No — manual rewrite required |

**Source:** https://community.sap.com/t5/enterprise-resource-planning-blog-posts-by-sap/semi-automated-custom-code-adaptation-after-sap-s-4hana-system-conversion/ba-p/13359048

---

## Summary

| Pattern | Classification | Action |
|---|---|---|
| SELECT without ORDER BY + BINARY SEARCH | **Must-Fix** | Add ORDER BY PRIMARY KEY or use SORTED TABLE |
| Old Open SQL syntax (no @) | Should-Fix | Add @ prefix; use @DATA(...) inline |
| FOR ALL ENTRIES on empty table | **Must-Fix** | Add IS NOT INITIAL check |
| Native SQL (EXEC SQL) | **Must-Fix** | Rewrite as Open SQL; use AMDP for HANA-native |
| SELECT * on wide tables | Should-Fix | Select only required columns |

## References

- SAP Community — Semi-automated custom code adaptation after S/4HANA conversion (by SAP): https://community.sap.com/t5/enterprise-resource-planning-blog-posts-by-sap/semi-automated-custom-code-adaptation-after-sap-s-4hana-system-conversion/ba-p/13359048
- SAP Community — ABAP and HANA coding guidelines (by SAP): https://community.sap.com/t5/application-development-and-automation-blog-posts/abap-and-hana-coding-guidelines/ba-p/13268891
- Custom Code Remediation guide in this KB: [16 — Custom Code Remediation](../part2-pro/16-custom-code-remediation.md)
