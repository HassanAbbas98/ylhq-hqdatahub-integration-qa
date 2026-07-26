# Phase 1 — Webhook & CSV Management | Bug Reports

**Project:** HQ Data Hub × YLHQ Integration
**Phase:** 1 — RealFlow CSV Webhook & CSV Search Page
**Prepared by:** Hassan (QA Engineer — Darkthorn Lab)
**Client:** Yellow Letter HQ (YLHQ)
**Last Updated:** 2026-07-26
**Total Bugs:** 9

---

## Legend

| Status | Meaning |
|---|---|
| 🔴 Open | Reported — fix not yet applied |
| 🟡 In Progress | Fix in progress |
| ✅ Closed | Fixed and confirmed by QA |

| Severity | Meaning |
|---|---|
| Critical | Blocks core functionality entirely |
| High | Major feature broken or data integrity affected |
| Medium | Feature partially broken, workaround exists |
| Low | Minor / cosmetic |

---

## Bug Index

| ID | Title | Severity | Status |
|---|---|---|---|
| BUG-01 | Upload field mapping missing on first two webhook CSV files | Critical | ✅ Closed |
| BUG-02 | Quoted multiline fields break CSV row parsing | High | ✅ Closed |
| BUG-03 | Place Order rejects company-only rows with blank first/last name | High | ✅ Closed |
| BUG-04 | CSV downloaded from /hq-datahub contains UTF-8 BOM and 3 empty lines | Medium | 🔴 Open |
| BUG-05 | Select All checkbox inactive on initial page load | Low | 🔴 Open |
| BUG-06 | Upload tool displays internal temp ID instead of dashboard filename | Low | 🔴 Open |
| BUG-07 | Multi-CSV Place Order — wrong file count and duplicate file reference | Critical | 🔴 Open |
| BUG-08 | Multi-CSV Place Order — multiple mapping windows instead of single shared mapping | Medium | 🔴 Open |
| BUG-09 | Saved Agent UI accessible to Administrator role without explicit saved-agent role | High | ✅ Closed |

---

## BUG-01 — Upload Field Mapping Missing on First Two Webhook CSV Files

| Field | Detail |
|---|---|
| **Severity** | Critical |
| **Priority** | Critical |
| **Reported By** | PM (via Slack) |
| **Fixed By** | Dev (Ivana) |
| **Status** | ✅ Closed |
| **Area** | Place Order — Upload Tool Field Mapping |

### Description
The first two CSV files received via webhook could not proceed through Place Order. The upload tool mapping window was missing the expected YLHQ upload fields entirely.

### Steps to Reproduce
1. Trigger CSV export from HQ Data Hub — webhook fires
2. Click Place Order on one of the first two received files
3. Observe the upload tool mapping window

### Actual Result
YLHQ upload fields missing — field mapping impossible — order placement fully blocked

### Expected Result
All YLHQ upload fields present — user can map columns and place order

### Root Cause
1. CSV parser broke rows with quoted multiline fields incorrectly
2. Validation required `first_name` and `last_name` on every row — HQ DataHub company-only rows have these blank

### Fix Applied
- CSV parser updated for RFC 4180 compliance
- `first_name`/`last_name` no longer required — mailing address, city, state, zip are now the required fields
- Both files now pass Place Order with 25 records each ✅

---

## BUG-02 — Quoted Multiline Fields Break CSV Row Parsing

| Field | Detail |
|---|---|
| **Severity** | High |
| **Priority** | High |
| **Reported By** | Dev (during BUG-01 investigation) |
| **Fixed By** | Dev (Ivana) |
| **Status** | ✅ Closed |
| **Area** | CSV Parsing — Webhook Backend |

### Description
The CSV parser did not handle RFC 4180 quoted multiline fields. Line breaks inside quoted fields were treated as row delimiters, splitting one record into multiple broken rows.

### Steps to Reproduce
```
"John","Smith","123 Main St
Apt 4B","Dallas","TX","75201"
```
Send via webhook → Place Order → observe row count and mapping

### Actual Result
Row split into 2 — row count inflated — field mapping misaligned

### Expected Result
John Smith = 1 record — address field intact — row count accurate

### Fix Applied
CSV parsing updated to correctly handle RFC 4180 quoted multiline fields

---

## BUG-03 — Place Order Rejects Company-Only Rows with Blank First/Last Name

| Field | Detail |
|---|---|
| **Severity** | High |
| **Priority** | High |
| **Reported By** | Dev (during BUG-01 investigation) |
| **Fixed By** | Dev (Ivana) |
| **Status** | ✅ Closed |
| **Area** | Place Order — Validation |

### Description
Validation required `first_name` and `last_name` on every row. HQ DataHub exports include company-only records where these fields are legitimately blank.

### Fix Applied
`first_name`/`last_name` no longer enforced. Required fields updated to: `mailing_address`, `city`, `state`, `zip`

### Verification
Both previously failing files passed with 25 records each ✅

---

## BUG-04 — CSV Downloaded from /hq-datahub Contains UTF-8 BOM and 3 Empty Lines

| Field | Detail |
|---|---|
| **Severity** | Medium |
| **Priority** | High |
| **Reported By** | Hassan (QA Engineer) |
| **Status** | 🔴 Open |
| **Area** | CSV Download — YLHQ Frontend |

### Description
CSV files downloaded from `/hq-datahub` contain a UTF-8 BOM (`ï»¿`) prepended to the first header and 3 empty lines at the top. The same data downloaded directly from HQ DataHub is clean. Dev confirmed the issue is on the YLHQ side — raw stored file on disk already contains the BOM.

### Steps to Reproduce
1. Export leads from HQ DataHub — CSV appears on `/hq-datahub`
2. Click filename or Download action
3. Open downloaded file in text editor or Excel

### Actual Result
```
[empty line]
[empty line]
[empty line]
ï»¿FirstName    LastName    RecipientAddress...
```

### Expected Result
```
FirstName    LastName    RecipientAddress...
```

### Impact
File unusable directly — breaks any header-based column mapping — automated pipelines fail on BOM

### Recommended Fix
Strip UTF-8 BOM (`EF BB BF`) and trim leading empty lines before writing file to disk during CSV download from DownloadLink

---

## BUG-05 — Select All Checkbox Inactive on Initial Page Load

| Field | Detail |
|---|---|
| **Severity** | Low |
| **Priority** | Medium |
| **Reported By** | Hassan (QA Engineer) |
| **Status** | 🔴 Open |
| **Area** | CSV Search Page — `/hq-datahub` |

### Steps to Reproduce
1. Navigate to `/hq-datahub`
2. Without selecting any CSV, click the Select All checkbox

### Actual Result
Checkbox inactive — only activates after one CSV is manually selected first

### Expected Result
Select All active and clickable immediately on page load

---

## BUG-06 — Upload Tool Displays Internal Temp ID Instead of Dashboard Filename

| Field | Detail |
|---|---|
| **Severity** | Low |
| **Priority** | Low |
| **Reported By** | Hassan (QA Engineer) |
| **Status** | 🔴 Open |
| **Area** | Upload Tool — Post Place Order redirect |

### Description
Dashboard shows `20260602-000011-43712.csv`. Upload tool shows `id_1780934172280_qwe6z8.csv`. These are the same file — the upload tool assigns its own internal temp ID. Not a bug in the integration itself — a UX inconsistency. Raised for PM/dev review.

---

## BUG-07 — Multi-CSV Place Order — Wrong File Count and Duplicate File Reference

| Field | Detail |
|---|---|
| **Severity** | Critical |
| **Priority** | Critical |
| **Reported By** | Hassan (QA Engineer) |
| **Status** | 🔴 Open |
| **Area** | Place Order — Multi-CSV → Upload Tool |

### Steps to Reproduce
1. Select 2 CSVs: `20260602-000011-43712.csv` and `20260602-000011-39488.csv`
2. Click Place Order Selected
3. Complete shop page redirect
4. Observe upload tool

### Actual Result
```
Uploaded file: id_1780934172280_qwe6z8.csv ✓ ×  (×6 — all same file)
```
6 instances shown instead of 2 — second CSV never appeared

### Expected Result
2 distinct files — one per selected CSV

### Likely Root Cause
Loop error in multi-CSV pass logic — first file reference duplicated instead of passing each file once

---

## BUG-08 — Multi-CSV Place Order — Multiple Mapping Windows Instead of Single Shared Mapping

| Field | Detail |
|---|---|
| **Severity** | Medium |
| **Priority** | Medium |
| **Reported By** | Hassan (QA Engineer) |
| **Status** | 🔴 Open |
| **Area** | Place Order — Multi-CSV → Upload Tool Field Mapping |

### Description
Upload tool opens a separate mapping window per selected CSV. Since all HQ DataHub CSVs share the same column format, requiring repeated identical mapping creates unnecessary friction.

### Important Context
This is intended behavior for standard YLHQ upload tool orders. For the HQ DataHub flow specifically — where all CSVs share the same format — a single mapping applied to all selected files is the desired behavior. A dedicated code change is required.

---

## BUG-09 — Saved Agent UI Accessible to Administrator Role Without Explicit saved-agent Role

| Field | Detail |
|---|---|
| **Severity** | High |
| **Priority** | High |
| **Reported By** | Hassan (QA Engineer) |
| **Fixed By** | Dev (Ivana) |
| **Status** | ✅ Closed |
| **Area** | Agent Details — Role-Based Access Control |

### Description
Saved Agent UI was accessible to users with the WordPress `administrator` role even without the `ylhq-saved-agent` role explicitly assigned. WordPress administrator role implicitly satisfies most capability checks — the gate was not strict enough.

### Fix Applied
Gate tightened to require explicit `ylhq-saved-agent` role only.

### Verification

| User | Role | Access After Fix |
|---|---|---|
| hassan.abbas | Administrator (no saved-agent role) | ❌ Correctly blocked |
| Dev YLHQ | `ylhq-saved-agent` assigned | ✅ Correctly granted |

---

> *Prepared by Hassan (QA Engineer — Darkthorn Lab) for client Yellow Letter HQ. Sensitive credentials excluded.*
