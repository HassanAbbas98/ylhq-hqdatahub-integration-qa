# Phase 2 — HQ DataHub Signup & Checkout | Bug Reports

**Project:** HQ Data Hub × YLHQ Integration
**Phase:** 2 — Signup, Checkout & Realeflow Partner API Account Creation
**Prepared by:** Hassan (QA Engineer — Darkthorn Lab)
**Client:** Yellow Letter HQ (YLHQ)
**Last Updated:** 2026-07-26
**Total Bugs:** 8

---

## Legend

| Status | Meaning |
|---|---|
| 🔴 Open | Reported — fix not yet applied |
| 🟡 In Progress | Fix in progress |
| ✅ Closed | Fixed and confirmed by QA |

| Severity | Meaning |
|---|---|
| Critical | Blocks core functionality / financial impact |
| High | Major feature broken or security concern |
| Medium | Feature partially broken, workaround exists |
| Low | Minor / cosmetic |

---

## Bug Index

| ID | Title | Severity | Status |
|---|---|---|---|
| BUG-10 | HQ DataHub navigation link visible to users without HQ DataHub role | High | 🔴 Open |
| BUG-11 | Change Package link redirects to 404 page | Medium | 🔴 Open |
| BUG-12 | Checkout fails for Packages 1239 and 1241 — invalid subscription billing period | Critical | 🔴 Open |
| BUG-13 | User with active HQ DataHub account can reach checkout and be charged again | Critical | 🔴 Open |
| BUG-14 | Order total missing on checkout page for Packages 1239 and 1241 | High | 🔴 Open |
| BUG-15 | Invalid PlanId sent to Realeflow Partner API for Package 1239 | Critical | 🔴 Open |
| BUG-16 | Realeflow account creation fails with entity validation error | High | 🔴 Open |
| BUG-17 | Expired subscription still blocks re-signup on HQ DataHub signup page | High | 🔴 Open |

---

## BUG-10 — HQ DataHub Navigation Link Visible to Users Without HQ DataHub Role

| Field | Detail |
|---|---|
| **Severity** | High |
| **Priority** | High |
| **Reported By** | Hassan (QA Engineer) |
| **Status** | 🔴 Open |
| **Area** | Navigation — Role-Based Access Control |

### Steps to Reproduce
1. Create a new YLHQ user without the HQ DataHub role
2. Log in → observe the navigation bar

### Actual Result
HQ DataHub nav link visible — user can attempt to access the section without a subscription or role

### Expected Result
HQ DataHub nav link only visible to users with the HQ DataHub role explicitly assigned

### Impact
Unauthorized users can see and attempt to access HQ DataHub. Confusing for users who have not subscribed.

---

## BUG-11 — Change Package Link Redirects to 404 Page

| Field | Detail |
|---|---|
| **Severity** | Medium |
| **Priority** | High |
| **Reported By** | Hassan (QA Engineer) |
| **Status** | 🔴 Open |
| **Area** | HQ DataHub Signup Flow — Step 2 |

### Steps to Reproduce
1. Navigate to `https://www.yellowletterhq.com/hq-datahub-signup-dev/`
2. Click **Change Package**

### Actual Result
Redirected to `https://www.yellowletterhq.com/hq-datahub-info/` — 404 Page Not Found

### Expected Result
Redirected to package selection page

### Root Cause
`/hq-datahub-info/` page does not exist — Change Package button points to wrong URL

---

## BUG-12 — Checkout Fails for Packages 1239 and 1241 — Invalid Subscription Billing Period

| Field | Detail |
|---|---|
| **Severity** | Critical |
| **Priority** | Critical |
| **Reported By** | Hassan (QA Engineer) |
| **Status** | 🔴 Open |
| **Area** | HQ DataHub Signup — Checkout |

### Steps to Reproduce
1. Select Package 1239 or 1241
2. Proceed to checkout and place order

### Actual Result
```json
{
  "result": "failure",
  "messages": "Invalid subscription billing period given."
}
```

### Expected Result
Checkout completes successfully — same as Package 1240

### Root Cause (Suspected)
WooCommerce product configuration issue — billing period missing, blank, or invalid on products 1239 and 1241 in wp-admin. Package 1240 is correctly configured.

### Impact
Two out of three packages completely broken at checkout

---

## BUG-13 — User With Active HQ DataHub Account Can Reach Checkout and Be Charged Again

| Field | Detail |
|---|---|
| **Severity** | Critical |
| **Priority** | Critical |
| **Reported By** | Hassan (QA Engineer) |
| **Status** | 🔴 Open |
| **Area** | HQ DataHub Signup — Pre-Checkout Gate |

### Description
A user with an already active and linked HQ DataHub account was able to proceed to checkout and complete payment. The Realeflow API correctly rejected duplicate account creation — but the user was already charged.

### Steps to Reproduce
1. Log in as user with active HQ DataHub account (User ID: 22721, hqdatahubId: 351300)
2. Select any package → proceed to checkout → complete payment

### Actual Result
- Payment taken — `paymentStatus: "paid"`
- Realeflow rejects: *"This email address is already in use on an active account"*
- User charged — no new account created — no automatic refund

### Expected Result
Pre-checkout gate detects active account — blocks checkout — shows clear message to user

### Evidence
```json
"hqdatahubId": "351300"
"paymentStatus": "paid"
"processingStatus": "failed"
"lastError": "This email address is already in use on an active account."
```

### Impact
User charged for a subscription they cannot receive. Chargeback and support risk. This edge case was explicitly required in the original specification.

---

## BUG-14 — Order Total Missing on Checkout Page for Packages 1239 and 1241

| Field | Detail |
|---|---|
| **Severity** | High |
| **Priority** | High |
| **Reported By** | Hassan (QA Engineer) |
| **Status** | 🔴 Open |
| **Area** | HQ DataHub Signup — Checkout Page |

### Description
Packages 1239 and 1241 show the product name on checkout but display no pricing, subtotal, processing fee, or total. Package 1240 displays correctly.

### Actual Result
Product name visible — all pricing fields blank for 1239 and 1241

### Expected Result
Full order summary for all packages before user confirms payment

### Note
Likely caused by same WooCommerce misconfiguration as BUG-12. Retest both together after fix.

### Impact
User has no visibility into what they will be charged — serious UX and compliance issue

---

## BUG-15 — Invalid PlanId Sent to Realeflow Partner API for Package 1239

| Field | Detail |
|---|---|
| **Severity** | Critical |
| **Priority** | Critical |
| **Reported By** | Hassan (QA Engineer) |
| **Status** | 🔴 Open |
| **Area** | HQ DataHub Signup — Post-Payment Account Creation |

### Description
After a successful payment for Package 1239, the YLHQ backend calls Realeflow with `planId: 1239`. Realeflow rejects this — the WooCommerce product ID is not the same as the Realeflow Plan ID. PlanId mapping is incorrect or missing for Package 1239.

### Evidence
```json
"planId": "1239"
"paymentStatus": "paid"
"processingStatus": "failed"
"realeflowAccountId": null
"lastError": "Invalid PlanId specified"
```
Order: 225337 | User: 22166

### Expected Result
Correct Realeflow Plan ID passed — account created — linked to YLHQ user

### Impact
User charged but no account created. Manual support intervention and refund required. Same issue likely affects Package 1241.

---

## BUG-16 — Realeflow Account Creation Fails With Entity Validation Error

| Field | Detail |
|---|---|
| **Severity** | High |
| **Priority** | High |
| **Reported By** | Hassan (QA Engineer) |
| **Status** | 🔴 Open |
| **Area** | HQ DataHub Signup — Post-Payment Account Creation |

### Description
A new user's Package 1240 signup failed after payment with a Realeflow entity validation error. PlanId is correct — the issue is specific to this user's profile data failing Realeflow's server-side validation.

### Evidence
```json
"planId": "1240"
"paymentStatus": "paid"
"processingStatus": "failed"
"realeflowAccountId": null
"lastError": "Validation failed for one or more entities."
"retryCount": 0
```
Order: 225400 | User: 22733

### Recommended Fix
Add pre-call payload validation on YLHQ side — catch missing or invalid fields before calling Realeflow API, rather than letting the API reject it post-payment

---

## BUG-17 — Expired Subscription Still Blocks Re-Signup on HQ DataHub Signup Page

| Field | Detail |
|---|---|
| **Severity** | High |
| **Priority** | High |
| **Reported By** | Hassan (QA Engineer) |
| **Status** | 🔴 Open |
| **Area** | HQ DataHub Signup — Active Subscription Gate |

### Steps to Reproduce
1. Complete a successful HQ DataHub signup
2. In wp-admin, change subscription status to **Expired**
3. Navigate to `https://www.yellowletterhq.com/hq-datahub-signup-dev/`

### Actual Result
Page still shows *"You already have an active HQ DataHub subscription"* — re-signup blocked

### Expected Result
Expired subscription allows user to re-subscribe through normal signup flow

### Root Cause (Suspected)
Active subscription gate reads `hqdatahubId` user meta or `accountStatus` DB field — neither updates when WooCommerce subscription expires. Gate needs to query live WooCommerce subscription status.

### Impact
Any user with an expired subscription is permanently locked out of re-subscribing without manual support intervention. Directly contradicts the PM-specified edge case requirement.

---

> *Prepared by Hassan (QA Engineer — Darkthorn Lab) for client Yellow Letter HQ. Sensitive credentials excluded.*
