# Phase 3 — Add to List Organizer | Test Cases

**Project:** HQ Data Hub × YLHQ Integration
**Phase:** 3 — Add to List Organizer Feature
**Prepared by:** Hassan (QA Engineer — Darkthorn Lab)
**Client:** Yellow Letter HQ (YLHQ)
**Last Updated:** 2026-07-26
**Total Test Cases:** 5

---

## Legend

| Symbol | Meaning |
|---|---|
| ✅ Pass | Executed and passed |
| ❌ Fail | Executed and failed |
| ⏳ Pending | Not yet executed |

---

## Feature Overview

A new **Add to List Organizer** button is added next to each CSV on the HQ DataHub CSV Search page (`/hq-datahub`). When clicked, a popup appears with the same options as manual list upload in List Organizer. On confirmation, the selected CSV list is imported into the user's List Organizer account.

---

## Module 1 — Add to List Organizer

---

### TC-01 — Add to List Organizer button visible on each CSV row

| Field | Detail |
|---|---|
| **Priority** | High |
| **Type** | Positive / UI |
| **Status** | ⏳ Pending |

**Steps:**
1. Navigate to `https://www.yellowletterhq.com/hq-datahub`
2. Observe action options next to each CSV

**Expected Result:**
- Every CSV row has an **Add to List Organizer** button
- Button visible and consistently positioned for all listed CSVs

---

### TC-02 — Popup displays correct options on button click

| Field | Detail |
|---|---|
| **Priority** | High |
| **Type** | Positive |
| **Status** | ⏳ Pending |

**Steps:**
1. Click **Add to List Organizer** on any CSV
2. Observe the popup

**Expected Result:**
- Popup appears with the same options as manual list upload in List Organizer
- All options correctly labeled and functional
- Popup does not auto-submit or auto-close

---

### TC-03 — Confirming popup imports list into List Organizer

| Field | Detail |
|---|---|
| **Priority** | Critical |
| **Type** | Positive / End-to-End |
| **Status** | ⏳ Pending |

**Steps:**
1. Click **Add to List Organizer** on a CSV
2. Complete popup options
3. Click Confirm
4. Open List Organizer

**Expected Result:**
- List appears in List Organizer
- List name, record count, and data are correct
- No duplicate list created on repeat action

---

### TC-04 — Property data displays correctly in List Organizer after import

| Field | Detail |
|---|---|
| **Priority** | Critical |
| **Type** | Negative / Bug Verification |
| **Status** | ❌ Fail — BUG-18 |
| **Linked Bug** | BUG-18 |

**Steps:**
1. Click **Add to List Organizer** on a CSV containing property data
2. Complete popup and confirm
3. Open List Organizer → open the imported list
4. Inspect property address and property data fields

**Actual Result:**
- Property data fields are empty in List Organizer
- `property_address` column header in the combined CSV is not mapping to the correct List Organizer field

**Expected Result:**
- All property data (property address, city, state, zip, etc.) appears correctly
- Column headers in the CSV match what List Organizer expects — consistent with standard YLHQ CSV format

---

### TC-05 — Cancelling popup does not import list

| Field | Detail |
|---|---|
| **Priority** | Medium |
| **Type** | Negative |
| **Status** | ⏳ Pending |

**Steps:**
1. Click **Add to List Organizer** on a CSV
2. When popup appears, click Cancel or close it
3. Open List Organizer

**Expected Result:**
- List is NOT added to List Organizer
- No partial or empty list created
- CSV Search page state unchanged — no error shown

---

## Summary Table

| TC | Test Case | Priority | Status |
|---|---|---|---|
| TC-01 | Add to List Organizer button visible on each CSV row | High | ⏳ Pending |
| TC-02 | Popup displays correct options on button click | High | ⏳ Pending |
| TC-03 | Confirming popup imports list into List Organizer | Critical | ⏳ Pending |
| TC-04 | Property data displays correctly after import | Critical | ❌ Fail — BUG-18 |
| TC-05 | Cancelling popup does not import list | Medium | ⏳ Pending |

---

> *Prepared by Hassan (QA Engineer — Darkthorn Lab) for client Yellow Letter HQ. Sensitive credentials excluded.*
