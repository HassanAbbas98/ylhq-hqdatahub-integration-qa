# RealFlow CSV Webhook — QA Testing Project

> **Role:** QA Engineer — Darkthorn Lab (Client: Yellow Letter HQ)
> **Project Type:** API Integration Testing + End-to-End Functional Testing

---

## Project Overview

This project covers end-to-end QA testing of the **HQ Data Hub × YLHQ** integration — a multi-phase project spanning webhook ingestion, CSV management, order placement, subscription signup, and List Organizer integration.

**Phase 1** covers the webhook integration between **HQ Data Hub** (a RealFlow white-label platform at `hqdata.realeflow.com`) and **Yellow Letter HQ (YLHQ)** (`yellowletterhq.com`) — allowing users to export lead CSVs directly into YLHQ for print mail order placement.

**Phase 2** covers the HQ DataHub signup and checkout flow — allowing users to select a subscription package, complete payment, and automatically create and link a Realeflow account via the Partner API.

**Phase 3** covers the Add to List Organizer feature — allowing users to send HQ DataHub CSVs directly into their YLHQ List Organizer from the CSV Search page.

---

## System Architecture

```
┌─────────────────────┐         Webhook POST          ┌──────────────────────────┐
│   HQ Data Hub        │ ────────────────────────────► │  YLHQ Webhook Endpoint   │
│ (hqdata.realeflow.com)│    ExternalAccountId         │  /api/realflow/csv-webhook│
│                      │    AccountId                  │                          │
│  User exports leads  │    DownloadLink               │  1. Validates API key    │
│  → triggers webhook  │                               │  2. Downloads CSV        │
└─────────────────────┘                               │  3. Stores file + metadata│
                                                       └──────────┬───────────────┘
                                                                  │
                                                                  ▼
                                                       ┌──────────────────────────┐
                                                       │   YLHQ Frontend          │
                                                       │   /hq-datahub            │
                                                       │                          │
                                                       │  User sees CSV list      │
                                                       │  → clicks Place Order    │
                                                       │  → order placement flow  │
                                                       └──────────────────────────┘
```

---

## Webhook Specification

| Property | Value |
|---|---|
| **Endpoint** | `POST https://apiorders.yellowletterhq.com/api/realflow/csv-webhook` |
| **Auth** | `x-api-key` header (shared secret) |
| **Success Response** | HTTP 201 |

### Request Payload

```json
{
  "ExternalAccountId": "user_001",
  "AccountId": 000011,
  "DownloadLink": "https://presigned-url/file.csv"
}
```

### Field Mapping

| Webhook Field | Stored As | Purpose |
|---|---|---|
| `ExternalAccountId` | `ylhqId` | YLHQ-side user identifier (preferred mapping) |
| `AccountId` | `hqdatahubId` | HQ Data Hub internal account ID (fallback) |
| `DownloadLink` | — | Presigned URL — backend downloads CSV from this |

### File Storage Logic

```
ExternalAccountId present  →  uploads/csvs/{ylhqId}/filename.csv
ExternalAccountId absent   →  uploads/csvs/{hqdatahubId}/filename.csv
```

### Filename Format (post-fix)
```
YYYYMMDD-{accountId}-{5-char-uuid}.csv
Example: 20260601-000011-05e5e.csv
```

---

## Account ID Mapping

| ID | Value | System |
|---|---|---|
| `ExternalAccountId` (ylhqId) | `user_001` | YLHQ WordPress user ID |
| `AccountId` (hqdatahubId) | `000011` | HQ Data Hub internal account ID |

Mapping is stored in WordPress user meta on the YLHQ side. When the webhook fires, YLHQ queries CSVs by both `ylhqId` and `hqdatahubId` to ensure records are returned regardless of which ID was used during storage.

---

## Features Tested

| # | Feature |
|---|---|
| 1 | Webhook receives payload and downloads CSV from presigned URL |
| 2 | CSV stored on server with correct folder structure |
| 3 | Metadata saved to DB (ylhqId, hqdatahubId, csvPath, originalFileName, createdAt, status) |
| 4 | New filename format: `YYYYMMDD-{accountId}-{5charId}.csv` |
| 5 | Quantity column (header row excluded from count) |
| 6 | Page renamed from "RealFlow CSVs" to "CSV Search" |
| 7 | CSV download via File column click |
| 8 | CSV download via Action button |
| 9 | Place Order button — triggers existing YLHQ order placement flow |
| 10 | Quoted multiline field parsing fix |
| 11 | Relaxed Place Order validation (company-only rows accepted) |
| 12 | Backward compatibility with legacy camelCase field names |
| 13 | Multi-CSV selection via checkboxes — Place Order Selected button |
| 14 | Agent Details CRUD — create, read, update, delete saved agent records |
| 15 | Agent details auto-populate in upload tool mapping window (ylhq-saved-agent role) |
| 16 | Role-based access control for Saved Agent UI (`ylhq-saved-agent` role gate) |
| 17 | HQ DataHub signup flow — package selection page (UI + mobile responsiveness) |
| 18 | HQ DataHub signup flow — account activation status page (UI + mobile responsiveness) |
| 19 | HQ DataHub signup — Realeflow Partner API account creation via post-payment webhook |
| 20 | HQ DataHub subscription gate on CSV Search page (login, role, active status checks) |
| 21 | Add to List Organizer — popup flow and list import from CSV Search page |

---

## Bugs Reported

| ID | Title | Severity | Status |
|---|---|---|---|
| BUG-01 | Upload field mapping missing on first two webhook CSV files | Critical | ✅ Closed |
| BUG-02 | Quoted multiline fields break CSV row parsing | High | ✅ Closed |
| BUG-03 | Place Order rejects company-only rows with blank first/last name | High | ✅ Closed |
| BUG-04 | CSV downloaded from /hq-datahub contains UTF-8 BOM and 3 empty lines | Medium | 🔴 Open |
| BUG-05 | Select All checkbox inactive on initial page load | Low | 🔴 Open |
| BUG-06 | Upload tool displays internal temp ID instead of dashboard filename | Low | 🔴 Open |
| BUG-07 | Multi-CSV Place Order — wrong file count and duplicate file reference in upload tool | Critical | 🔴 Open |
| BUG-08 | Multi-CSV Place Order — multiple mapping windows instead of single shared mapping | Medium | 🔴 Open |
| BUG-09 | Saved Agent UI accessible to Administrator role without explicit saved-agent role | High | ✅ Closed |
| BUG-10 | HQ DataHub navigation link visible to users without HQ DataHub role | High | 🔴 Open |
| BUG-11 | Change Package link redirects to 404 page | Medium | 🔴 Open |
| BUG-12 | Checkout fails for Packages 1239 and 1241 — invalid subscription billing period | Critical | 🔴 Open |
| BUG-13 | User with active HQ DataHub account can reach checkout and be charged again | Critical | 🔴 Open |
| BUG-14 | Order total missing on checkout page for Packages 1239 and 1241 | High | 🔴 Open |
| BUG-15 | Invalid PlanId sent to Realeflow Partner API for Package 1239 | Critical | 🔴 Open |
| BUG-16 | Realeflow account creation fails with entity validation error | High | 🔴 Open |
| BUG-17 | Expired subscription still blocks re-signup on HQ DataHub signup page | High | 🔴 Open |
| BUG-18 | Property data not showing in List Organizer — column header mapping mismatch | High | 🔴 Open |

> Full bug details including steps to reproduce, root cause, and fix verification are in [`bug-reports/realflow-csv-webhook-bugs.md`](bug-reports/realflow-csv-webhook-bugs.md)

---

## Test Results Summary

| TC | Test Case | Result |
|---|---|---|
| TC-01 | Webhook receives payload and downloads CSV | ✅ Pass |
| TC-02 | Filename format: `YYYYMMDD-{accountId}-{5charId}.csv` | ✅ Pass |
| TC-03 | Place Order — previously failing files now pass (25 records each) | ✅ Pass |
| TC-06 | Quantity column excludes header row | ✅ Pass |
| TC-07 | Page renamed to "CSV Search" | ✅ Pass |
| TC-08 | Download via File column | ✅ Pass |
| TC-09 | Download via Action button | ✅ Pass |
| TC-11 | Webhook fallback — no ExternalAccountId | ⏳ Pending API key |
| TC-12 | Backward compatibility — camelCase field names | ⏳ Pending API key |
| TC-13 | Missing AccountId → HTTP 400 | ⏳ Pending API key |
| TC-14 | Missing DownloadLink → HTTP 400 | ⏳ Pending API key |
| TC-15 | Invalid DownloadLink → HTTP 502 | ⏳ Pending API key |
| TC-16 | Wrong API key → HTTP 403 | ⏳ Pending API key |
| TC-17 | Empty payload → HTTP 400 | ⏳ Pending API key |
| TC-04 | Company-only rows accepted in Place Order | ⏳ Pending |
| TC-05 | Missing mailing fields rejected in Place Order | ⏳ Pending |
| TC-10 | Multiline quoted fields parse correctly | ⏳ Pending |
| TC-18 | Quantity excludes header — verified with known row count | ⏳ Pending |
| TC-19 | Multiple CSVs for same account all appear on page | ⏳ Pending |
| TC-20 | Dual ID query — CSVs returned by both ylhqId and hqdatahubId | ⏳ Pending |
| TC-21 | Place Order generates unique candidateUploadId per CSV | ⏳ Pending |
| TC-22 | Downloaded CSV integrity — row count and data match source | ⏳ Pending |
| TC-22 | Select multiple CSVs via checkboxes | ⏳ Pending |
| TC-23 | Separate mapping window per selected CSV | ⏳ Pending |
| TC-24 | Single merged order after all mappings complete | ⏳ Pending |
| TC-25 | Select all CSVs and place order | ⏳ Pending |
| TC-26 | Create new agent details | ⏳ Pending |
| TC-27 | Agent details auto-populate in mapping window | ⏳ Pending |
| TC-28 | Read / view existing agent details | ⏳ Pending |
| TC-29 | Update existing agent details | ⏳ Pending |
| TC-30 | Delete agent details | ⏳ Pending |
| TC-31 | Select All checkbox active without prior selection | ❌ Fail — BUG-05 |
| TC-32 | Multi-CSV Place Order passes correct files to upload tool | ❌ Fail — BUG-07 |
| TC-33 | Select 3 CSVs — upload tool shows 3 distinct files | ⏳ Pending |
| TC-34 | Single shared mapping across all HQ Data Hub CSVs | ❌ Fail — BUG-08 |
| TC-35 | Select All — correct total count in merged order | ⏳ Pending |
| TC-36 | Add to List Organizer button visible on each CSV row | ⏳ Pending |
| TC-37 | Add to List Organizer popup displays correct options | ⏳ Pending |
| TC-38 | Confirming popup adds list to List Organizer | ⏳ Pending |
| TC-39 | Property data displays correctly in List Organizer after import | ❌ Fail — BUG-18 |
| TC-40 | Cancelling popup does not add list to List Organizer | ⏳ Pending |

> Full test case details in [`test-cases/`](test-cases/)

---

## Tools Used

| Tool | Purpose |
|---|---|
| **Postman** | API/webhook testing, negative test cases, HTTP response validation |
| **Browser DevTools** | Network inspection, request/response capture, account ID discovery |
| **HQ Data Hub** (`app.hqdatahub.com`) | End-to-end trigger source for webhook |
| **YLHQ wp-admin** | User meta verification, account ID lookup, role verification |
| **Excel / Google Sheets** | CSV integrity validation, row count verification |

---

## Skills Demonstrated

- **API Testing** — Webhook endpoint validation, HTTP status code verification, authentication testing
- **Integration Testing** — End-to-end flow across two separate third-party platforms
- **Bug Analysis** — Root cause investigation using server file path logic and encoding knowledge (UTF-8 BOM)
- **Security Testing** — Role-based access control bypass identification and verification
- **Test Case Design** — Critical path first, then edge cases; both positive and negative scenarios
- **Documentation** — Structured bug reports, Slack updates, requirement-to-test mapping
- **CSV/Data Validation** — RFC 4180 parsing issues, header row exclusion, field mapping verification
- **WordPress / WooCommerce** — User meta inspection, role assignment, capability gate verification, WooCommerce Subscriptions product configuration and status management
- **Payment Flow Testing** — Subscription checkout, post-payment webhook validation, duplicate payment prevention, idempotency verification
- **Third-Party API Testing** — Realeflow Partner API account creation, Plan ID mapping, entity validation error analysis
- **Mobile Responsiveness** — UI testing across iPhone 14 Pro Max viewport, layout and spacing issue identification

---

## Repository Structure

```
ylhq-hqdatahub-integration-qa/
│
├── README.md
│
test-cases/
├── phase1-webhook-csv-test-cases.md
├── phase2-signup-checkout-test-cases.md
└── phase3-list-organizer-test-cases.md

bug-reports/
├── phase1-webhook-csv-bugs.md
├── phase2-signup-checkout-bugs.md
└── phase3-list-organizer-bugs.md
```

---

> *This project was executed as part of QA work at Darkthorn Lab for client Yellow Letter HQ. All account IDs, API keys, and sensitive credentials have been excluded from this repository.*
