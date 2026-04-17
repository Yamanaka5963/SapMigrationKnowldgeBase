# ABAP Language Versions and Variations

ABAP (Advanced Business Application Programming) is SAP's proprietary programming language, created in the 1980s. Unlike many programming languages, ABAP has **no independent version numbering** (no "ABAP 1.0", "ABAP 2.0" etc.). Instead, language features evolve with **SAP NetWeaver and S/4HANA release numbers**. Understanding which generation and variant of ABAP is in use is essential for migration planning, code review, and Clean Core compliance assessment.

---

## 1. Overview — How ABAP Versioning Works

ABAP language capabilities are tied to the **SAP NetWeaver (NW) Basis release** running on the system:

- **ECC 6.0** runs on NW 7.0x — supports up to early Modern ABAP syntax
- **S/4HANA on-premise** runs on NW 7.50+ — supports full Modern ABAP
- **BTP ABAP Environment (Steampunk)** and **S/4HANA Cloud Public Edition** — ABAP for Cloud Development only

There is no single "ABAP version number" you configure or upgrade independently. When SAP releases a new NetWeaver or S/4HANA version, new language keywords become available. Older syntax remains supported for backward compatibility (with some deprecated exceptions).

**Practical implication for migration projects:** Code written for ECC 6.0 (NW 7.0x era) uses a subset of what NW 7.50+ supports. All ECC syntax is valid in S/4HANA, but S/4HANA enables newer, cleaner alternatives.

---

## 2. Generational Evolution

### Generation 1 — Classic ABAP (up to early 2000s)

| Property | Detail |
|---|---|
| Era | SAP R/2 → R/3 |
| Programming style | Procedural, global variables, FORM routines |
| UI model | Dynpro (Screen Painter) |
| Status | Still present as legacy codebase; supported but discouraged for new development |

```abap
" Classic ABAP: FORM-based procedural structure
FORM get_material_data.
  SELECT * FROM mara INTO TABLE gt_mara
    WHERE mtart = 'FERT'.
ENDFORM.
```

---

### Generation 2 — ABAP Objects (1998~ / NW 6.x~)

| Property | Detail |
|---|---|
| Era | R/3 4.6 (1999) onward |
| Programming style | Object-oriented: CLASS / INTERFACE / METHOD |
| OOP concepts | Encapsulation, inheritance, polymorphism |
| Status | Standard for ECC-era custom development; still valid in S/4HANA |

```abap
" ABAP Objects: class-based structure
CLASS lcl_material DEFINITION.
  PUBLIC SECTION.
    METHODS: get_data IMPORTING iv_matnr TYPE matnr.
ENDCLASS.

CLASS lcl_material IMPLEMENTATION.
  METHOD get_data.
    SELECT SINGLE * FROM mara INTO @DATA(ls_mara)
      WHERE matnr = @iv_matnr.
  ENDMETHOD.
ENDCLASS.
```

> In practice, most ECC custom code mixes Classic ABAP (FORM routines, global includes) with ABAP Objects. Pure ABAP Objects is recommended for all new S/4HANA development.

---

### Generation 3 — Modern ABAP (NW 7.02~ / 2009~)

Language syntax expanded significantly across NW releases. Key milestones:

| NW Release | SAP Product Context | Key Language Additions |
|---|---|---|
| 7.02 | ECC EhP4+ | `NEW`, `CAST`, inline declarations `DATA(lv_x)` |
| 7.31 | ECC EhP6+ | `LOOP AT ... WHERE`, `REDUCE`, `FILTER` |
| 7.40 | S/4HANA 1511 (early) | `VALUE #(...)`, `CORRESPONDING #(...)`, String Templates `\|...\|` |
| 7.50 | S/4HANA 1610+ | Open SQL enhancements: JOIN, subqueries, Common Table Expressions |
| 7.52 | S/4HANA 1809+ | `FINAL` immutable declarations, further expression improvements |

```abap
" Modern ABAP (NW 7.40+): inline declarations, VALUE constructor, string templates
DATA(lt_result) = VALUE tt_mara( FOR ls_row IN lt_mara
                                  WHERE ( mtart = 'FERT' )
                                  ( ls_row ) ).

DATA(lv_msg) = |Material { ls_mara-matnr } type: { ls_mara-mtart }|.

" Modern Open SQL (NW 7.50+): host variables, inline INTO
SELECT matnr mtart matkl
  FROM mara
  INTO TABLE @DATA(lt_mara)
  WHERE mtart IN @so_mtart
  ORDER BY PRIMARY KEY.
```

> **Brownfield migration note:** S/4HANA on-premise uses NW 7.50+, so all Modern ABAP syntax is available. However, ECC-era code does not need to be rewritten to Modern ABAP to function — the upgrade is about Clean Core compliance and new development standards, not syntax enforcement.

---

### Generation 4 — ABAP Cloud / ABAP for Cloud Development (2020s~)

Designed for **SAP BTP ABAP Environment** (formerly "Steampunk") and **S/4HANA Cloud Public Edition**.

**Key restrictions (what is NOT allowed):**

| Restricted element | Reason |
|---|---|
| `CALL FUNCTION` (non-RFC) | No local FM calls — use class methods |
| `SUBMIT` | No batch job submission via classic API |
| Classic Dynpro / SAP GUI | UI must be Fiori/OData |
| Direct database table access (INSERT/UPDATE) | Must use Released APIs only |
| SE80 transaction | Development only via Eclipse ADT |

**Key enablers:**

- ABAP Objects (mandatory)
- CDS View Entities (data model)
- RAP + EML (CRUD behavior)
- Released API access (`@AbapCatalog.sqlViewAppendName`, `@AccessControl.authorizationCheck`)

```abap
" ABAP Cloud: only Released APIs, only ADT, only OOP
CLASS zcl_material_handler DEFINITION PUBLIC FINAL CREATE PUBLIC.
  PUBLIC SECTION.
    INTERFACES if_oo_adt_classrun.
ENDCLASS.

CLASS zcl_material_handler IMPLEMENTATION.
  METHOD if_oo_adt_classrun~main.
    " Released CDS view access only
    SELECT matnr mtart FROM i_material
      INTO TABLE @DATA(lt_materials)
      WHERE mtart = 'FERT'
      ORDER BY PRIMARY KEY.
    out->write( lt_materials ).
  ENDMETHOD.
ENDCLASS.
```

---

## 3. Language Variants (Official Subsets as of Q4 2022)

SAP defines official language subsets through DDIC domain `ABAPVRS`. These can be verified via program `DEMO_ABAP_VERSIONS` on a live system.

| # | Variant Name | Description | Status |
|---|---|---|---|
| 1 | **Standard ABAP (Unicode)** | Full language scope. Introduced in Basis 6.10. Stricter syntax rules than pre-Unicode. | Current — applies to all ECC and S/4HANA on-premise |
| 2 | **ABAP for Cloud Development** | Restricted language + restricted Released API access only. For BTP ABAP Environment and S/4HANA Cloud Public Edition. | Current — the future direction |
| 3 | **ABAP for Key Users** | Very restricted subset for key-user extensions via SAP Fiori apps (Custom Fields and Logic, Custom Business Objects). | Current — for business users, not developers |
| 4 | **Static ABAP with limited object use** | Restricted dynamic language elements. Transitional variant. | Obsolete |
| 5 | **Non-Unicode ABAP** | Pre-6.10 behavior. Not supported on current NetWeaver. | Obsolete |

> **For ECC → S/4HANA brownfield migrations:** existing custom code typically runs as **Standard ABAP (Unicode)** (variant 1) in both ECC and S/4HANA on-premise. ABAP for Cloud Development (variant 2) is a separate target relevant only if the roadmap includes moving to BTP or S/4HANA Cloud Public Edition.

---

## 4. RAP and EML — The Cloud-Native Development Paradigm

**RAP (RESTful Application Programming Model)** is the SAP-recommended framework for building Fiori applications in S/4HANA and BTP. It replaces classical Dynpro + BOR/BADIs for new development.

### RAP Stack Architecture

```
CDS View Entity          → Data model layer (SELECT, associations)
     ↓
Behavior Definition      → Declares CRUD capabilities, validations, determinations
     ↓
Behavior Implementation  → ABAP Class implementing the business logic
     ↓
Service Definition       → Exposes the entity as an OData service
     ↓
Service Binding          → Binds to OData v2 (Fiori Elements) or v4 endpoint
```

### EML — Entity Manipulation Language

EML is the DML syntax used to interact with RAP entities from ABAP:

```abap
" EML: Create a travel booking via RAP entity
MODIFY ENTITIES OF zi_travel_m
  ENTITY travel
    CREATE FIELDS ( agency_id customer_id begin_date end_date )
    WITH VALUE #( ( %cid        = 'CID_001'
                    agency_id   = '00000001'
                    customer_id = '000001'
                    begin_date  = '20240101'
                    end_date    = '20240110' ) )
  REPORTED DATA(lt_reported)
  FAILED   DATA(lt_failed)
  MAPPED   DATA(lt_mapped).

IF lt_failed IS NOT INITIAL.
  " Handle creation failure
ENDIF.

COMMIT ENTITIES.
```

> **Brownfield relevance:** RAP is not required for ECC→S/4HANA brownfield migration. Existing Dynpro/classical ABAP continues to work. RAP is the target architecture for **new development** after the migration is complete and for S/4HANA Cloud.

---

## 5. Clean Core Compliance Mapping

SAP's **Clean Core** principle means custom code should not directly modify SAP-managed database tables or call unreleased internal APIs. This is enforced by the **ABAP for Cloud Development** language variant.

| ABAP Generation / Style | Clean Core Status | Notes |
|---|---|---|
| Classic ABAP — direct table access (`INSERT INTO mara`), non-released FM calls | ❌ Not compliant | Must be refactored; blocked in ABAP for Cloud Development |
| ABAP Objects — custom logic using Released APIs and CDS views | △ Conditionally compliant | Compliant if only Released APIs are used; blocked if unreleased calls exist |
| Modern ABAP — Open SQL against standard SAP tables (SELECT only) | △ Partially compliant | Read access is tolerated; write access to SAP-managed tables is not |
| ABAP Cloud / RAP — Released APIs only, CDS views, EML | ✅ Fully compliant | This is the Clean Core target state |

**Practical guidance for migration projects:**

- **Brownfield S/4HANA on-premise (migration target):** Clean Core compliance is a **goal**, not a hard blocker. ATCcan flag non-compliant code with the ABAP for Cloud Development ruleset. The must-fix items for go-live are functional blockers (see Part 4 of this KB), not Clean Core violations.
- **S/4HANA Cloud Public Edition or BTP:** Clean Core compliance is **enforced** — non-released API calls are blocked at compile time by the ABAP for Cloud Development variant.

---

## 6. Reference Sources

Ranked by trustworthiness for ABAP language and migration information:

| Rank | Source | URL | Notes |
|---|---|---|---|
| 🥇 | SAP News (Official) | https://news.sap.com/2023/10/celebrating-abap-programming-language-turns-40/ | ABAP 40th anniversary article — SAP employees interviewed, authoritative history |
| 🥇 | SAP Community Blog (SAP-authored) | https://community.sap.com/t5/enterprise-resource-planning-blog-posts-by-sap/evolution-of-abap/ba-p/13522761 | ABAP evolution deep-dive by SAP staff (2022) — high technical accuracy |
| 🥈 | SAP PRESS | https://learning.sap-press.com/abap | SAP's official publishing partner — books reviewed by SAP before publication |
| 🥈 | Martin Maruskin's Blog | https://blog.maruskin.eu/2022/12/abap-language-versions.html | Detailed breakdown of official language variants (ABAPVRS domain); high technical accuracy for a community source |
| 🥉 | Wikipedia (EN) | https://en.wikipedia.org/wiki/ABAP | Good for historical overview; not reliable for technical implementation detail |

> **For migration project decisions:** rely on 🥇 sources. Community posts (SAP Community member blogs, Stack Overflow) are useful for patterns and workarounds but should be cross-checked against SAP Help documentation before applying to production systems.

---

## Quick Reference — ABAP Generation by System

| SAP System | NW Release | ABAP Generation Available |
|---|---|---|
| ECC 6.0 EhP0–EhP3 | 7.00–7.01 | Classic ABAP + ABAP Objects (Gen 1–2) |
| ECC 6.0 EhP4–EhP8 | 7.02–7.02 | Early Modern ABAP (Gen 3 partial) |
| S/4HANA 1511–1610 | 7.40–7.50 | Modern ABAP (Gen 3 full) |
| S/4HANA 1709+ | 7.52+ | Modern ABAP + ABAP Cloud-ready syntax |
| S/4HANA Cloud Public Edition | Cloud | ABAP for Cloud Development only (Gen 4) |
| BTP ABAP Environment | Cloud | ABAP for Cloud Development only (Gen 4) |
