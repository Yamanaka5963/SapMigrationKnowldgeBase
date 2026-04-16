# 21 - Common Issues & Workarounds

Known pitfalls encountered in SAP S/4HANA migration projects, with recommended actions.

---

## Assessment Phase

### Readiness Check shows high volume of custom code findings
**Risk:** Underestimated remediation effort blows up project timeline.
**Action:** Run ATC early — before project scoping. Use findings volume and severity to size the custom code workstream accurately. Decommission unused Z-objects first to reduce real scope.

### Add-on not S/4HANA certified
**Risk:** Blocker — SUM will reject the conversion if an incompatible add-on is installed.
**Action:** Contact the add-on vendor immediately. Options: upgrade to certified version, remove the add-on, or find a standard S/4HANA replacement. Do not start conversion until resolved.

### Non-Unicode source system
**Risk:** S/4HANA requires Unicode. Non-Unicode ECC cannot be converted directly.
**Action:** Plan a Unicode conversion of the ECC system as a prerequisite step before the S/4HANA conversion. Factor this into the project timeline.

---

## Custom Code Remediation

### SELECT without ORDER BY causing data inconsistencies
**Root cause:** HANA does not guarantee sort order without ORDER BY. On older DBs, primary key order was implicit.
**Action:** Add explicit ORDER BY clauses to all affected SELECT statements. ATC flags these — prioritize ones in financial or inventory posting logic.

### Pool/cluster table access in Z-code
**Root cause:** Pool and cluster tables are converted to transparent tables in S/4HANA. Direct SELECT on pool/cluster tables no longer works.
**Action:** Replace direct table access with the corresponding function modules or BAPIs provided by SAP, or adapt SQL to the new transparent table structure.

### SPAU backlog too large to complete during downtime
**Risk:** Downtime extends beyond the planned window.
**Action:** Pre-classify SPAU objects before go-live. Use the "Reset to Original" option for non-critical modifications that can be re-applied later. Prioritize objects in active business processes.

---

## Data Migration

### CVI (Customer-Vendor Integration) errors blocking go-live
**Root cause:** Customers/vendors in ECC have data quality issues (duplicate BP number ranges, missing fields required for BP).
**Action:** Run CVI pre-check report well before cutover. Cleanse customer/vendor master data early — this workstream is often underestimated. Assign a dedicated data steward.

### Migration Cockpit simulation passes but production load fails
**Root cause:** Data in production differs from QAS (e.g., different number ranges, org structure not fully configured in PRD).
**Action:** Always run at least one full mock migration load in a system that mirrors production configuration. Never rely solely on QAS simulation results.

### Object sequence errors during data load
**Root cause:** Dependent objects loaded before referenced objects (e.g., G/L account loaded before chart of accounts).
**Action:** Follow the correct load sequence: organizational data → master data → transactional data. Within master data, load referenced objects first.

---

## Conversion (Brownfield / Shell)

### SUM prerequisite check fails
**Common causes:** Missing SAP Notes, incompatible add-ons, inconsistent database statistics, insufficient disk space.
**Action:** Read the SUM log carefully — each check failure has a specific note or fix. Do not attempt to bypass prerequisite checks. Resolve each item before re-running.

### HANA sizing underestimated — disk space exceeded during conversion
**Risk:** Conversion fails mid-run requiring rollback.
**Action:** Use the SAP HANA Sizing Report and Readiness Check sizing output before provisioning hardware. Add at least 20% buffer above the calculated estimate for conversion temporary space.

### Downtime exceeds planned window
**Common causes:** Unexpected table migration duration (large tables not identified in uptime phase), SPDD/SPAU backlog larger than estimated, failed SUM steps requiring restart.
**Action:** Run the SUM impact analysis tool before conversion to identify large tables. Pre-migrate large tables in the uptime phase. Consider downtime-optimized conversion for tight windows.

---

## Go-Live & Hypercare

### Batch jobs failing after go-live
**Common causes:** Job variants reference old table/field names; authorization objects changed; job scheduling not migrated.
**Action:** Test all batch jobs in QAS post-conversion. Re-create job variants where needed. Include batch job testing in the UAT checklist.

### Interface errors after go-live
**Common causes:** Message types changed, partner profiles not configured in PRD, middleware (PI/PO, BTP Integration Suite) pointing to wrong system.
**Action:** Test all interfaces end-to-end in a full-landscape test before go-live. Maintain an interface inventory with owner, test status, and go-live activation step.

### Authorization errors reported by users
**Common causes:** New S/4HANA authorization objects not included in roles; Fiori tile authorization not assigned; authorization trace not run on new transactions.
**Action:** Run SU53 for each user-reported error. Use authorization trace (ST01/STAUTHTRACE) to identify missing objects. Plan a dedicated authorization workstream — S/4HANA introduces new auth objects not present in ECC.
