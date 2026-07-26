# Phase 2 — HQ DataHub Signup & Checkout | Test Cases

**Project:** HQ Data Hub × YLHQ Integration
**Phase:** 2 — Signup, Checkout & Realeflow Partner API Account Creation
**Prepared by:** Hassan (QA Engineer — Darkthorn Lab)
**Client:** Yellow Letter HQ (YLHQ)
**Last Updated:** 2026-07-26
**Total Test Cases:** 20

---

## Legend

| Symbol | Meaning |
|---|---|
| ✅ Pass | Executed and passed |
| ❌ Fail | Executed and failed — linked bug included |
| ⏳ Pending | Not yet executed |

---

## Module Index

| Module | TCs |
|---|---|
| [Module 1 — Package Selection Page (UI)](#module-1--package-selection-page-ui) | TC-01 to TC-05 |
| [Module 2 — Checkout Flow](#module-2--checkout-flow) | TC-06 to TC-09 |
| [Module 3 — Account Activation & Partner API](#module-3--account-activation--partner-api) | TC-10 to TC-14 |
| [Module 4 — Subscription Gate on CSV Page](#module-4--subscription-gate-on-csv-page) | TC-15 to TC-17 |
| [Module 5 — Edge Cases](#module-5--edge-cases) | TC-18 to TC-20 |

---

## Module 1 — Package Selection Page (UI)

---

### TC-01 — All three packages display correctly

| Field | Detail |
|---|---|
| **Priority** | High |
| **Type** | Positive / UI |
| **Status** | ✅ Pass |

**Steps:** Navigate to `https://www.yellowletterhq.com/hq-datahub-signup-dev/`

**Expected:**
- Trial: $250 one-time, 90 days full access then $349/mo
- Buy-In: $97/mo after one-time buy-in
- Month-to-Month: $349/mo, no buy-in required

---

### TC-02 — Package selection page mobile responsiveness

| Field | Detail |
|---|---|
| **Priority** | Medium |
| **Type** | UI / Responsive |
| **Status** | ✅ Pass — bugs noted |

**Steps:** Open page in DevTools → iPhone 14 Pro Max (430 × 932)

**Bugs Identified:**
- Cards have excessive side margins — too narrow on mobile
- Buy-In and Month-to-Month card content split across two separate visual cards
- Inconsistent card structure — Trial correct, Buy-In and Month-to-Month broken
- "Select Month-to-Month" button label too long for mobile width
- Insufficient spacing between description text and CTA button on all cards

---

### TC-03 — Select Trial navigates to checkout

| Field | Detail |
|---|---|
| **Priority** | Critical |
| **Type** | Positive |
| **Status** | ✅ Pass |

**Steps:** Click **Select Trial** on Package 1240

**Expected:** Redirected to checkout page with correct product pre-selected

---

### TC-04 — Select Month-to-Month navigates to checkout

| Field | Detail |
|---|---|
| **Priority** | Critical |
| **Type** | Positive |
| **Status** | ✅ Pass |

**Steps:** Click **Select Month-to-Month** on Package 1239

**Expected:** Redirected to checkout with Package 1239 pre-selected

---

### TC-05 — Back to CSV Search button behavior during signup

| Field | Detail |
|---|---|
| **Priority** | Low |
| **Type** | UX Observation |
| **Status** | ⏳ Pending — awaiting PM confirmation |

**Question raised:** Should the Back to CSV Search button be visible during signup before account activation? Flagged to PM for decision.

---

## Module 2 — Checkout Flow

---

### TC-06 — Package 1240 checkout displays full order summary

| Field | Detail |
|---|---|
| **Priority** | Critical |
| **Type** | Positive |
| **Status** | ✅ Pass |

**Expected:**
```
Product: HQ DataHub Trial × 1    $250.00
Subtotal:                         $250.00
Processing Fee (2.9%):            $7.25
Total:                            $257.25
```

---

### TC-07 — Package 1239 checkout displays full order summary

| Field | Detail |
|---|---|
| **Priority** | Critical |
| **Type** | Negative / Bug Verification |
| **Status** | ❌ Fail — BUG-12, BUG-14 |
| **Linked Bugs** | BUG-12, BUG-14 |

**Actual Result:** Product name shown but no pricing, subtotal, or total displayed

**Expected Result:** Full order summary including price, subtotal, processing fee, and total

---

### TC-08 — Package 1241 checkout displays full order summary

| Field | Detail |
|---|---|
| **Priority** | Critical |
| **Type** | Negative / Bug Verification |
| **Status** | ❌ Fail — BUG-12, BUG-14 |
| **Linked Bugs** | BUG-12, BUG-14 |

**Actual Result:** Same as TC-07 — no pricing shown

**Expected Result:** Full order summary

---

### TC-09 — Change Package redirects to package selection page

| Field | Detail |
|---|---|
| **Priority** | Medium |
| **Type** | Negative / Bug Verification |
| **Status** | ❌ Fail — BUG-11 |
| **Linked Bug** | BUG-11 |

**Steps:** On signup page, click **Change Package**

**Actual Result:** Redirected to `/hq-datahub-info/` — 404 Page Not Found

**Expected Result:** Redirected to package selection page

---

## Module 3 — Account Activation & Partner API

---

### TC-10 — Account activation status page UI

| Field | Detail |
|---|---|
| **Priority** | High |
| **Type** | UI |
| **Status** | ✅ Pass |

**Steps:** Navigate to `/hq-datahub-signup-status-dev/?status=processing&package=1240`

**Expected:** Three-step progress indicator, activation checklist, Refresh Status button, Back to Packages link all present and correctly rendered

---

### TC-11 — Package 1240 signup creates Realeflow account successfully

| Field | Detail |
|---|---|
| **Priority** | Critical |
| **Type** | End-to-End |
| **Status** | ✅ Pass |

**Steps:** New user selects Package 1240 → completes checkout → observes account creation

**Expected:** Realeflow account created — `realeflowAccountId` populated — `hqdatahubId` linked to YLHQ user — CSV Search page accessible

**Verified With:** Order #800538, Package 1239 (PlanId 1333 mapping) ✅

---

### TC-12 — Package 1239 signup creates Realeflow account successfully

| Field | Detail |
|---|---|
| **Priority** | Critical |
| **Type** | End-to-End |
| **Status** | ✅ Pass |

**Steps:** New user selects Package 1239 → completes checkout → observes account creation

**Expected:** Realeflow account created with correct PlanId 1333 mapping

---

### TC-13 — Signup failure sends notification email and logs error

| Field | Detail |
|---|---|
| **Priority** | High |
| **Type** | Negative |
| **Status** | ✅ Pass — observed during BUG-15 and BUG-16 |

**Expected:**
- User receives failure notification email with Order ID
- wp-admin order notes contain full error detail including YLHQ User ID, Package, Error, and Error data JSON

---

### TC-14 — Signup webhook is idempotent — duplicate webhook does not create duplicate account

| Field | Detail |
|---|---|
| **Priority** | Critical |
| **Type** | Edge Case |
| **Status** | ⏳ Pending |

**Steps:** Ask dev to trigger the same webhook event twice for the same order

**Expected:** Second webhook event recognized as duplicate — no second Realeflow account created — `idempotencyKey` prevents duplicate processing

---

## Module 4 — Subscription Gate on CSV Page

---

### TC-15 — Active subscriber can access Place Order

| Field | Detail |
|---|---|
| **Priority** | Critical |
| **Type** | Positive |
| **Status** | ✅ Pass |

**Steps:** Log in as user with active HQ DataHub subscription → navigate to `/hq-datahub` → click Place Order

**Expected:** Order placement proceeds normally

---

### TC-16 — User without HQ DataHub account redirected to signup on Place Order

| Field | Detail |
|---|---|
| **Priority** | Critical |
| **Type** | Negative |
| **Status** | ⏳ Pending |

**Steps:** Log in as user with no HQ DataHub account → navigate to `/hq-datahub` → click Place Order

**Expected:** Redirected to package selection/signup page

---

### TC-17 — Expired subscription blocks Place Order and allows re-signup

| Field | Detail |
|---|---|
| **Priority** | High |
| **Type** | Negative / Bug Verification |
| **Status** | ❌ Fail — BUG-17 |
| **Linked Bug** | BUG-17 |

**Steps:** Set subscription to Expired in wp-admin → navigate to `/hq-datahub-signup-dev/`

**Actual Result:** Page still shows "You already have an active HQ DataHub subscription" — re-signup blocked

**Expected Result:** Expired subscription allows user to re-subscribe through normal signup flow

---

## Module 5 — Edge Cases

---

### TC-18 — User with active account cannot reach checkout again

| Field | Detail |
|---|---|
| **Priority** | Critical |
| **Type** | Negative / Bug Verification |
| **Status** | ❌ Fail — BUG-13 |
| **Linked Bug** | BUG-13 |

**Steps:** Log in as user with active HQ DataHub account → select a package → proceed to checkout → complete payment

**Actual Result:** User allowed through checkout — charged — Realeflow rejects with "email already in use"

**Expected Result:** Pre-checkout gate detects active account — blocks checkout — shows clear message

---

### TC-19 — Invalid PlanId returns failure after payment with correct error logging

| Field | Detail |
|---|---|
| **Priority** | Critical |
| **Type** | Negative / Bug Verification |
| **Status** | ❌ Fail — BUG-15 |
| **Linked Bug** | BUG-15 |

**Evidence:**
```json
"planId": "1239"
"lastError": "Invalid PlanId specified"
"paymentStatus": "paid"
"realeflowAccountId": null
```

---

### TC-20 — Missing or invalid user profile data causes entity validation failure

| Field | Detail |
|---|---|
| **Priority** | High |
| **Type** | Negative / Bug Verification |
| **Status** | ❌ Fail — BUG-16 |
| **Linked Bug** | BUG-16 |

**Evidence:**
```json
"lastError": "Validation failed for one or more entities."
"realeflowAccountId": null
"paymentStatus": "paid"
```

**Note:** User-data-specific issue — requires dev to inspect exact API payload for User 22733 / Order 225400

---

## Summary Table

| TC | Module | Test Case | Priority | Status |
|---|---|---|---|---|
| TC-01 | Package Selection UI | All three packages display correctly | High | ✅ Pass |
| TC-02 | Package Selection UI | Mobile responsiveness | Medium | ✅ Pass — bugs noted |
| TC-03 | Package Selection UI | Select Trial navigates to checkout | Critical | ✅ Pass |
| TC-04 | Package Selection UI | Select Month-to-Month navigates to checkout | Critical | ✅ Pass |
| TC-05 | Package Selection UI | Back to CSV Search button behavior | Low | ⏳ Pending PM confirmation |
| TC-06 | Checkout | Package 1240 full order summary | Critical | ✅ Pass |
| TC-07 | Checkout | Package 1239 full order summary | Critical | ❌ Fail — BUG-12, BUG-14 |
| TC-08 | Checkout | Package 1241 full order summary | Critical | ❌ Fail — BUG-12, BUG-14 |
| TC-09 | Checkout | Change Package redirects correctly | Medium | ❌ Fail — BUG-11 |
| TC-10 | Account Activation | Status page UI | High | ✅ Pass |
| TC-11 | Account Activation | Package 1240 Realeflow account created | Critical | ✅ Pass |
| TC-12 | Account Activation | Package 1239 Realeflow account created | Critical | ✅ Pass |
| TC-13 | Account Activation | Failure sends notification and logs error | High | ✅ Pass |
| TC-14 | Account Activation | Idempotent webhook — no duplicate account | Critical | ⏳ Pending |
| TC-15 | Subscription Gate | Active subscriber can Place Order | Critical | ✅ Pass |
| TC-16 | Subscription Gate | No account → redirected to signup | Critical | ⏳ Pending |
| TC-17 | Subscription Gate | Expired subscription blocks re-signup | High | ❌ Fail — BUG-17 |
| TC-18 | Edge Cases | Active account blocked from checkout | Critical | ❌ Fail — BUG-13 |
| TC-19 | Edge Cases | Invalid PlanId — failure after payment | Critical | ❌ Fail — BUG-15 |
| TC-20 | Edge Cases | Entity validation failure — bad user data | High | ❌ Fail — BUG-16 |

---

> *Prepared by Hassan (QA Engineer — Darkthorn Lab) for client Yellow Letter HQ. Sensitive credentials excluded.*
