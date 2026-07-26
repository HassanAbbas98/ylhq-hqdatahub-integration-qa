# Phase 3 — Add to List Organizer | Bug Reports

**Project:** HQ Data Hub × YLHQ Integration
**Phase:** 3 — Add to List Organizer Feature
**Prepared by:** Hassan (QA Engineer — Darkthorn Lab)
**Client:** Yellow Letter HQ (YLHQ)
**Last Updated:** 2026-07-26
**Total Bugs:** 1

---

## Legend

| Status | Meaning |
|---|---|
| 🔴 Open | Reported — fix not yet applied |
| 🟡 In Progress | Fix in progress |
| ✅ Closed | Fixed and confirmed by QA |

---

## Bug Index

| ID | Title | Severity | Status |
|---|---|---|---|
| BUG-18 | Property data not showing in List Organizer — column header mapping mismatch | High | 🔴 Open |

---

## BUG-18 — Property Data Not Showing in List Organizer — Column Header Mapping Mismatch

| Field | Detail |
|---|---|
| **Severity** | High |
| **Priority** | High |
| **Reported By** | Hassan (QA Engineer) |
| **Status** | 🔴 Open |
| **Area** | Add to List Organizer — CSV Column Mapping |
| **Environment** | `https://www.yellowletterhq.com/hq-datahub` |

### Description

When a CSV from the HQ DataHub CSV Search page is sent to List Organizer via the Add to List Organizer feature, the property data fields do not appear in the imported list. The issue is a column header mismatch — the combined CSV uses `property_address` as the header, but List Organizer expects the standard YLHQ CSV column header format used across the platform.

### Steps to Reproduce

1. Navigate to `https://www.yellowletterhq.com/hq-datahub`
2. Click **Add to List Organizer** on any CSV containing property data
3. Complete the popup options and confirm
4. Open List Organizer → open the imported list
5. Inspect property address and related property data fields

### Actual Result

- List appears in List Organizer
- Property data fields (property address, city, state, zip, etc.) are empty / missing
- No error or warning shown to the user — data silently lost
- `property_address` column header in the combined CSV is not recognized by List Organizer

### Expected Result

- All property data appears correctly in List Organizer after import
- Column headers in the CSV sent to List Organizer match what List Organizer expects
- Behavior consistent with standard YLHQ CSV uploads to List Organizer

### Root Cause

The combined/merged CSV generated from HQ DataHub exports uses `property_address` as the column header for the property address field. List Organizer expects a different column header name matching the standard YLHQ CSV format. The mapping layer between HQ DataHub CSV headers and List Organizer fields does not account for this difference.

### Impact

- Property data is silently lost on every list imported via Add to List Organizer
- Core functionality of the feature is broken — users importing lists for property-based campaigns receive incomplete data
- No error or warning is shown — users may not notice the data loss until they attempt to use the list

### Recommended Fix

In the Add to List Organizer mapping layer, map `property_address` from the HQ DataHub CSV to the corresponding standard YLHQ column header that List Organizer expects — consistent with the existing YLHQ CSV format used across the platform.

---

> *Prepared by Hassan (QA Engineer — Darkthorn Lab) for client Yellow Letter HQ. Sensitive credentials excluded.*
