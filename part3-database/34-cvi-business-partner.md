# 34 - Customer/Vendor → Business Partner (CVI)

## What Is It

In SAP S/4HANA, **Customers and Vendors are replaced by the unified Business Partner (BP)** object. The Customer-Vendor Integration (CVI) is the technical framework that synchronizes the legacy customer/vendor tables with the new BP master data.

This migration is **mandatory** in every S/4HANA project — Brownfield, Greenfield, and Shell Conversion alike.

## ECC Tables Affected

### Customer Master
| ECC Table | Description | Status in S/4HANA |
|---|---|---|
| `KNA1` | Customer Master (General Data) | Still exists — synced from BP |
| `KNB1` | Customer Master (Company Code) | Still exists — synced from BP |
| `KNBK` | Customer Bank Data | Still exists — synced from BP |
| `KNVV` | Customer Sales Data | Still exists — synced from BP |
| `KNVP` | Customer Partner Functions | Still exists |
| `KNVS` | Customer Shipping Data | Still exists |

### Vendor Master
| ECC Table | Description | Status in S/4HANA |
|---|---|---|
| `LFA1` | Vendor Master (General Data) | Still exists — synced from BP |
| `LFB1` | Vendor Master (Company Code) | Still exists — synced from BP |
| `LFBK` | Vendor Bank Data | Still exists — synced from BP |
| `LFM1` | Vendor Master (Purchasing Org) | Still exists |
| `LFM2` | Vendor Master (Purchasing Data) | Still exists |

> Legacy customer/vendor tables still exist in S/4HANA but are **secondary**. BP (`BUT000`) is the leading master data object. All changes must be made via BP — CVI synchronizes them back to `KNA1`/`LFA1`.

## New Business Partner Tables

| New Table | Description |
|---|---|
| `BUT000` | Business Partner Master (general data, central BP record) |
| `BUT001` | BP Addresses |
| `BUT100` | BP Roles assignment |
| `BUT020` | BP Addresses (assignment) |
| `CVI_CUST_LINK` | Sync link: BP GUID → KNA1 (Customer number) |
| `CVI_VEND_LINK` | Sync link: BP GUID → LFA1 (Vendor number) |

## Data Consistency Rule

After migration, the following must hold true:

```
COUNT(KNA1) = COUNT(CVI_CUST_LINK)
COUNT(LFA1) = COUNT(CVI_VEND_LINK)
```

Any mismatch indicates an incomplete CVI migration. This is a **hard blocker** — resolve before go-live.

## Migration Process

### Step 1 — Pre-Migration Check
- Run CVI consistency check report to identify customer/vendor records with data quality issues
- Clean up: missing required fields, duplicate number ranges, invalid addresses
- Check that `KNA1-LIFNR` (customer's vendor number) and `LFA1-KUNNR` (vendor's customer number) are correctly maintained where same entity is both customer and vendor

### Step 2 — BP Number Range Setup
- Define BP number ranges in the target system
- Decide: internal number assignment (SAP assigns BP number) or external (keep customer/vendor number as BP number)

### Step 3 — CVI Synchronization Run
- Run CVI migration program to create BP records for all existing customers and vendors
- Each customer → BP with role `FLCU01` (FI Customer)
- Each vendor → BP with role `FLVN01` (FI Vendor)

### Step 4 — Merge (Optional but Recommended)
Where the same real-world entity exists as both a customer AND a vendor:
- Use transaction **`CVI_LEDH`** (Legal Entity Data Harmonization) to identify and merge pairs
- Select leading entity for address, tax data, and bank data
- Use **MDS Load Cockpit** to sync merged pairs to single BP

### Step 5 — Validation
- Verify `COUNT(KNA1) = COUNT(CVI_CUST_LINK)`
- Verify `COUNT(LFA1) = COUNT(CVI_VEND_LINK)`
- Spot-check BP records for key customers/vendors — verify all data fields migrated correctly

## Impact for Custom Code

Any Z-code that:
- Creates customers via direct `INSERT INTO KNA1`
- Creates vendors via direct `INSERT INTO LFA1`
- Reads `KNA1`/`LFA1` without going through BP APIs

...must be reviewed. In S/4HANA:
- Use `BAPI_BUPA_CREATE_FROM_DATA` or CDS views for BP creation/reads
- Direct inserts into `KNA1`/`LFA1` are not supported — CVI synchronization will not trigger

## Key Transaction Reference

| Transaction | Purpose |
|---|---|
| `BP` | Business Partner master data maintenance (replaces XD01/XK01) |
| `CVI_LEDH` | Legal Entity Data Harmonization — merge customer+vendor into one BP |
| `VCVI_CUST_LINK` | View CVI customer link table |
| `VCVI_VEND_LINK` | View CVI vendor link table |

## References

- SAP Community — CVI concept explained (member post): https://community.sap.com/t5/technology-blog-posts-by-members/s-4hana-busines-partner-customer-vendor-integration-cvi-concept-between/ba-p/13529974
- SAP Community — FAQ: CVI for S/4HANA system conversion (by SAP): https://community.sap.com/t5/enterprise-resource-planning-blog-posts-by-sap/faq-cvi-customer-vendor-integration-for-system-conversion-to-sap-s-4hana/ba-p/13740757
- SAP Community — Merge customer and vendor using CVI_LEDH (member post): https://community.sap.com/t5/enterprise-resource-planning-blog-posts-by-members/sap-s-4hana-business-partner-conversion-merge-customer-and-vendor-using/ba-p/13529974
- Simplification List S/4HANA 2023 (official PDF): https://help.sap.com/doc/c34b5ef72430484cb4d8895d5edd12af/2023/en-US/SIMPL_OP2023.pdf
