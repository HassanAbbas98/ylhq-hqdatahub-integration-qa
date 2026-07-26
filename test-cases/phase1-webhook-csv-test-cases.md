# Phase 1 — Webhook & CSV Management | Test Cases

**Project:** HQ Data Hub × YLHQ Integration
**Phase:** 1 — RealFlow CSV Webhook & CSV Search Page
**Prepared by:** Hassan (QA Engineer — Darkthorn Lab)
**Client:** Yellow Letter HQ (YLHQ)
**Last Updated:** 2026-07-26
**Total Test Cases:** 35

---

## Legend

| Symbol | Meaning |
|---|---|
| ✅ Pass | Executed and passed |
| ❌ Fail | Executed and failed |
| ⏳ Pending | Not yet executed |

| Priority | Meaning |
|---|---|
| Critical | Must pass before release |
| High | Core feature coverage |
| Medium | Important but not blocking |
| Low | Edge case or cosmetic |

---

## Module Index

| Module | TCs |
|---|---|
| [Module 1 — Webhook Core](#module-1--webhook-core) | TC-01 to TC-07 |
| [Module 2 — Error Handling & Validation](#module-2--error-handling--validation) | TC-08 to TC-12 |
| [Module 3 — CSV Search Page](#module-3--csv-search-page) | TC-13 to TC-16 |
| [Module 4 — Place Order (Single CSV)](#module-4--place-order-single-csv) | TC-17 to TC-21 |
| [Module 5 — Place Order (Multiple CSVs)](#module-5--place-order-multiple-csvs) | TC-22 to TC-25 |
| [Module 6 — Agent Details CRUD](#module-6--agent-details-crud) | TC-26 to TC-30 |
| [Module 7 — Multi-CSV Bugs & Edge Cases](#module-7--multi-csv-bugs--edge-cases) | TC-31 to TC-35 |

---

## Module 1 — Webhook Core

---

### TC-01 — Successful webhook with ExternalAccountId

| Field | Detail |
|---|---|
| **Priority** | Critical |
| **Type** | Positive |
| **Status** | ✅ Pass |

**Steps:**
1. POST to `https://apiorders.yellowletterhq.com/api/realflow/csv-webhook`
2. Header: `x-api-key: YOUR_KEY`
3. Body:
```json
{
  "ExternalAccountId": "user_001",
  "AccountId": 000011,
  "DownloadLink": "https://real-csv-url.com/file.csv"
}
```

**Expected:** HTTP 201 — `ylhqId: "user_001"`, `hqdatahubId: "000011"`, `csvPath: "uploads/csvs/user_001/..."`, `status: "downloaded"`

---

### TC-02 — File stored under ExternalAccountId folder

| Field | Detail |
|---|---|
| **Priority** | Critical |
| **Type** | Positive |
| **Status** | ✅ Pass |

**Steps:** After TC-01, check `csvPath` in the 201 response.

**Expected:** `csvPath` = `uploads/csvs/user_001/filename.csv`

---

### TC-03 — Filename follows correct format

| Field | Detail |
|---|---|
| **Priority** | High |
| **Type** | Positive |
| **Status** | ✅ Pass |

**Steps:** After TC-01, check `originalFileName` in 201 response and on CSV Search page.

**Expected:** `YYYYMMDD-{accountId}-{5charId}.csv` — e.g. `20260601-000011-05e5e.csv`

---

### TC-04 — Webhook fallback — ExternalAccountId absent

| Field | Detail |
|---|---|
| **Priority** | High |
| **Type** | Fallback |
| **Status** | ⏳ Pending |

**Steps:**
```json
{
  "AccountId": 000011,
  "DownloadLink": "https://real-csv-url.com/file.csv"
}
```

**Expected:** HTTP 201 — `csvPath` = `uploads/csvs/000011/...` — CSV still appears for user user_001 via hqdatahubId lookup

---

### TC-05 — Backward compatibility — legacy camelCase field names

| Field | Detail |
|---|---|
| **Priority** | High |
| **Type** | Compatibility |
| **Status** | ⏳ Pending |

**Steps:**
```json
{
  "externalAccountId": "user_001",
  "accountId": 000011,
  "presignedCsvDownloadUrl": "https://real-csv-url.com/file.csv"
}
```

**Expected:** HTTP 201 — identical to TC-01

---

### TC-06 — Metadata saved correctly in database

| Field | Detail |
|---|---|
| **Priority** | High |
| **Type** | Positive |
| **Status** | ✅ Pass |

**Steps:** Inspect 201 response from TC-01.

**Expected:** All six fields present — `ylhqId`, `hqdatahubId`, `csvPath`, `originalFileName`, `createdAt`, `status: "downloaded"`

---

### TC-07 — End-to-end trigger from HQ Data Hub

| Field | Detail |
|---|---|
| **Priority** | Critical |
| **Type** | End-to-End |
| **Status** | ✅ Pass |

**Steps:**
1. Log into `https://app.hqdatahub.com/Marketing/Leads/Index/Property#/`
2. Select leads and trigger CSV export
3. Navigate to `https://www.yellowletterhq.com/hq-datahub`

**Expected:** CSV appears automatically with correct filename, quantity, status, and action buttons

---

## Module 2 — Error Handling & Validation

---

### TC-08 — Missing AccountId returns 400

| Field | Detail |
|---|---|
| **Priority** | High |
| **Type** | Negative |
| **Status** | ⏳ Pending |

```json
{ "ExternalAccountId": "user_001", "DownloadLink": "https://real-csv-url.com/file.csv" }
```
**Expected:** HTTP 400

---

### TC-09 — Missing DownloadLink returns 400

| Field | Detail |
|---|---|
| **Priority** | High |
| **Type** | Negative |
| **Status** | ⏳ Pending |

```json
{ "ExternalAccountId": "user_001", "AccountId": 000011 }
```
**Expected:** HTTP 400

---

### TC-10 — Invalid DownloadLink returns 502

| Field | Detail |
|---|---|
| **Priority** | High |
| **Type** | Negative |
| **Status** | ⏳ Pending |

```json
{
  "ExternalAccountId": "user_001",
  "AccountId": 000011,
  "DownloadLink": "https://thisurldoesnotexist.xyz/file.csv"
}
```
**Expected:** HTTP 502

---

### TC-11 — Wrong API key returns 403

| Field | Detail |
|---|---|
| **Priority** | Critical |
| **Type** | Security |
| **Status** | ⏳ Pending |

**Steps:** Send valid payload with `x-api-key: wrongkeytest123`

**Expected:** HTTP 403

---

### TC-12 — Empty payload returns 400

| Field | Detail |
|---|---|
| **Priority** | Medium |
| **Type** | Negative |
| **Status** | ⏳ Pending |

```json
{}
```
**Expected:** HTTP 400

---

## Module 3 — CSV Search Page

---

### TC-13 — CSV Search page displays correct data

| Field | Detail |
|---|---|
| **Priority** | High |
| **Type** | Positive |
| **Status** | ✅ Pass |

**Steps:** Log into YLHQ test account → navigate to `/hq-datahub`

**Expected:** Page title = "CSV Search", each row shows Received date, File name, Quantity, Status, Action buttons

---

### TC-14 — Quantity excludes header row

| Field | Detail |
|---|---|
| **Priority** | High |
| **Type** | Positive |
| **Status** | ✅ Pass |

**Expected:** 74-line file (1 header + 73 data rows) → Quantity = `73`

---

### TC-15 — Download via File column

| Field | Detail |
|---|---|
| **Priority** | High |
| **Type** | Positive |
| **Status** | ✅ Pass |

**Steps:** Click filename in File column

**Expected:** CSV downloads and opens correctly

---

### TC-16 — Download via Action button

| Field | Detail |
|---|---|
| **Priority** | High |
| **Type** | Positive |
| **Status** | ✅ Pass |

**Steps:** Click Download under Action column

**Expected:** Identical result to TC-15

---

## Module 4 — Place Order (Single CSV)

---

### TC-17 — Place Order — full end-to-end flow

| Field | Detail |
|---|---|
| **Priority** | Critical |
| **Type** | End-to-End |
| **Status** | ✅ Pass |

**Steps:**
1. Click Place Order on a CSV
2. Map fields in upload tool mapping window
3. Complete order placement

**Expected:** Order placed successfully — AccuZIP processing completes with correct quantity in merged and disk file

**Verified With:** `20260601-000011-05e5e.csv` — 73 records ✅

---

### TC-18 — Place Order — company-only rows accepted

| Field | Detail |
|---|---|
| **Priority** | High |
| **Type** | Regression |
| **Status** | ⏳ Pending |

**Steps:** Open downloaded CSV → identify rows with blank first/last name but company name present → confirm those rows included in AccuZIP output

**Expected:** Company-only rows accepted — row count before and after AccuZIP consistent

---

### TC-19 — Place Order — missing mailing fields rejected

| Field | Detail |
|---|---|
| **Priority** | High |
| **Type** | Negative |
| **Status** | ⏳ Pending |

**Steps:** Create CSV with rows missing `mailing_address`, `city`, `state`, or `zip` → upload → Place Order

**Expected:** Rows with missing mailing fields fail validation with a clear error message

---

### TC-20 — Multiline quoted fields parse correctly

| Field | Detail |
|---|---|
| **Priority** | High |
| **Type** | Regression |
| **Status** | ⏳ Pending |

**Steps:** Create CSV with line break inside a quoted field → send via webhook → Place Order

**Expected:** Record parsed as 1 row — not split. Quantity accurate.

---

### TC-21 — Place Order generates unique candidateUploadId per CSV

| Field | Detail |
|---|---|
| **Priority** | Medium |
| **Type** | Positive |
| **Status** | ⏳ Pending |

**Steps:** Click Place Order on two different CSVs → compare `candidateUploadId` in each URL

**Expected:** Each file generates a different `candidateUploadId` — no reuse

---

## Module 5 — Place Order (Multiple CSVs)

---

### TC-22 — Select multiple CSVs via checkboxes

| Field | Detail |
|---|---|
| **Priority** | Critical |
| **Type** | Positive |
| **Status** | ⏳ Pending |

**Steps:** Select 2+ CSVs → click Place Order Selected

**Expected:** All selected CSVs recognized — mapping window opens per CSV sequentially

---

### TC-23 — Separate mapping window opens per selected CSV

| Field | Detail |
|---|---|
| **Priority** | Critical |
| **Type** | Positive |
| **Status** | ⏳ Pending |

**Expected:** Each window shows correct headers for its specific CSV — mapping for one does not affect others

---

### TC-24 — Single merged order after all mappings complete

| Field | Detail |
|---|---|
| **Priority** | Critical |
| **Type** | Positive |
| **Status** | ⏳ Pending |

**Expected:** Single merged file created — single order placed — total record count = sum of all selected CSV rows

---

### TC-25 — Select all CSVs and place order

| Field | Detail |
|---|---|
| **Priority** | Medium |
| **Type** | Edge Case |
| **Status** | ⏳ Pending |

**Expected:** All CSVs selected — mapping windows open sequentially — single merged order at the end — no UI freeze or timeout

---

## Module 6 — Agent Details CRUD

---

### TC-26 — Create new agent details

| Field | Detail |
|---|---|
| **Priority** | Critical |
| **Type** | Positive |
| **Status** | ⏳ Pending |

**Steps:** Navigate to Agent Details → Add New Agent → fill all fields (Name, Phone, Address, City, State, Zip) → Save

**Expected:** Agent saved — all six fields stored correctly — appears in agent list

---

### TC-27 — Agent details auto-populate in upload tool mapping window

| Field | Detail |
|---|---|
| **Priority** | Critical |
| **Type** | Integration |
| **Status** | ⏳ Pending |

**Preconditions:** TC-26 completed

**Steps:** Place Order on any CSV → observe mapping window

**Expected:** Agent details section pre-populated with all six saved fields — no manual re-entry required

---

### TC-28 — Read / view existing agent details

| Field | Detail |
|---|---|
| **Priority** | High |
| **Type** | Positive |
| **Status** | ⏳ Pending |

**Expected:** All saved fields display correctly — no truncation — matches TC-26 input exactly

---

### TC-29 — Update existing agent details

| Field | Detail |
|---|---|
| **Priority** | High |
| **Type** | Positive |
| **Status** | ⏳ Pending |

**Steps:** Edit agent → change Phone and City → Save → open Place Order mapping window

**Expected:** Updated values saved — mapping window reflects updated details, not old ones

---

### TC-30 — Delete agent details

| Field | Detail |
|---|---|
| **Priority** | High |
| **Type** | Positive |
| **Status** | ⏳ Pending |

**Steps:** Delete agent → confirm → open Place Order mapping window

**Expected:** Agent removed from list — mapping window no longer auto-populates

---

## Module 7 — Multi-CSV Bugs & Edge Cases

---

### TC-31 — Select All checkbox active without prior individual selection

| Field | Detail |
|---|---|
| **Priority** | Medium |
| **Type** | Bug Verification |
| **Status** | ❌ Fail — BUG-05 |
| **Linked Bug** | BUG-05 |

**Steps:** Navigate to `/hq-datahub` → without selecting any CSV, click Select All

**Actual Result:** Select All inactive on page load — only activates after one manual selection

**Expected Result:** Select All clickable immediately on page load

---

### TC-32 — Multi-CSV Place Order passes correct files to upload tool

| Field | Detail |
|---|---|
| **Priority** | Critical |
| **Type** | Bug Verification |
| **Status** | ❌ Fail — BUG-07 |
| **Linked Bug** | BUG-07 |

**Steps:** Select 2 CSVs → Place Order Selected → observe upload tool

**Actual Result:** 6 instances of the same file shown — second CSV never appeared

**Expected Result:** 2 distinct files — one per selected CSV

---

### TC-33 — Select 3 CSVs — upload tool shows 3 distinct files

| Field | Detail |
|---|---|
| **Priority** | High |
| **Type** | Edge Case |
| **Status** | ⏳ Pending |
| **Linked Bug** | BUG-07 |

**Expected:** Exactly 3 distinct files — no duplicates, no missing files

---

### TC-34 — Single shared mapping across all HQ Data Hub CSVs

| Field | Detail |
|---|---|
| **Priority** | Medium |
| **Type** | Bug Verification |
| **Status** | ❌ Fail — BUG-08 |
| **Linked Bug** | BUG-08 |

**Actual Result:** Separate mapping window opens per CSV — user repeats identical mapping N times

**Expected Result (desired):** Single mapping window — applied to all selected CSVs automatically

**Note:** Standard upload tool behavior for regular orders — dedicated change needed for HQ DataHub flow

---

### TC-35 — Select All then Place Order — correct total count in merged order

| Field | Detail |
|---|---|
| **Priority** | High |
| **Type** | Edge Case |
| **Status** | ⏳ Pending |

**Preconditions:** BUG-07 fixed

**Expected:** Total record count = sum of all individual CSV quantities — single order placed — no records dropped or duplicated

---

## Summary Table

| TC | Module | Test Case | Priority | Status |
|---|---|---|---|---|
| TC-01 | Webhook Core | Successful webhook with ExternalAccountId | Critical | ✅ Pass |
| TC-02 | Webhook Core | File stored under ExternalAccountId folder | Critical | ✅ Pass |
| TC-03 | Webhook Core | Filename follows correct format | High | ✅ Pass |
| TC-04 | Webhook Core | Fallback — ExternalAccountId absent | High | ⏳ Pending |
| TC-05 | Webhook Core | Backward compatibility — camelCase fields | High | ⏳ Pending |
| TC-06 | Webhook Core | Metadata saved correctly in DB | High | ✅ Pass |
| TC-07 | Webhook Core | End-to-end trigger from HQ Data Hub | Critical | ✅ Pass |
| TC-08 | Error Handling | Missing AccountId → 400 | High | ⏳ Pending |
| TC-09 | Error Handling | Missing DownloadLink → 400 | High | ⏳ Pending |
| TC-10 | Error Handling | Invalid DownloadLink → 502 | High | ⏳ Pending |
| TC-11 | Error Handling | Wrong API key → 403 | Critical | ⏳ Pending |
| TC-12 | Error Handling | Empty payload → 400 | Medium | ⏳ Pending |
| TC-13 | CSV Search Page | Page displays correct data | High | ✅ Pass |
| TC-14 | CSV Search Page | Quantity excludes header row | High | ✅ Pass |
| TC-15 | CSV Search Page | Download via File column | High | ✅ Pass |
| TC-16 | CSV Search Page | Download via Action button | High | ✅ Pass |
| TC-17 | Place Order — Single | Full end-to-end Place Order flow | Critical | ✅ Pass |
| TC-18 | Place Order — Single | Company-only rows accepted | High | ⏳ Pending |
| TC-19 | Place Order — Single | Missing mailing fields rejected | High | ⏳ Pending |
| TC-20 | Place Order — Single | Multiline quoted fields parse correctly | High | ⏳ Pending |
| TC-21 | Place Order — Single | Unique candidateUploadId per CSV | Medium | ⏳ Pending |
| TC-22 | Place Order — Multi | Select multiple CSVs via checkboxes | Critical | ⏳ Pending |
| TC-23 | Place Order — Multi | Separate mapping window per CSV | Critical | ⏳ Pending |
| TC-24 | Place Order — Multi | Single merged order after all mappings | Critical | ⏳ Pending |
| TC-25 | Place Order — Multi | Select all CSVs and place order | Medium | ⏳ Pending |
| TC-26 | Agent CRUD | Create new agent details | Critical | ⏳ Pending |
| TC-27 | Agent CRUD | Agent details auto-populate in mapping window | Critical | ⏳ Pending |
| TC-28 | Agent CRUD | Read / view existing agent details | High | ⏳ Pending |
| TC-29 | Agent CRUD | Update existing agent details | High | ⏳ Pending |
| TC-30 | Agent CRUD | Delete agent details | High | ⏳ Pending |
| TC-31 | Multi-CSV Bugs | Select All active without prior selection | Medium | ❌ Fail |
| TC-32 | Multi-CSV Bugs | Multi-CSV passes correct files to upload tool | Critical | ❌ Fail |
| TC-33 | Multi-CSV Bugs | Select 3 CSVs — 3 distinct files in upload tool | High | ⏳ Pending |
| TC-34 | Multi-CSV Bugs | Single shared mapping across all HQ DataHub CSVs | Medium | ❌ Fail |
| TC-35 | Multi-CSV Bugs | Select All — correct total count in merged order | High | ⏳ Pending |

---

> *Prepared by Hassan (QA Engineer — Darkthorn Lab) for client Yellow Letter HQ. Sensitive credentials excluded.*
