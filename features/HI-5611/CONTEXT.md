---
epic: HI-5611
title: HI Borrower One-Time Payment & Pre-payment Scheduling in Agent Tool
spec_url: https://credify.atlassian.net/wiki/spaces/PROD/pages/4697620576/HI+Borrower+One-Time+Payment+Pre-payment+scheduling+in+Agent+Tool
status: in-validation
last_refreshed: 2026-05-26
test_checklist_ticket: HI-7146
confluence_page_id: ""
---

# HI-5611 — HI Borrower One-Time Payment & Pre-payment Scheduling in Agent Tool

## Spec Summary

This epic adds VQ (Verification Queue / Agent Tool) capabilities so Upgrade phone agents (CCPs) can manage One-Time Payments (OTP) and No/No Installment Pre-payment Schedules on behalf of HI borrowers, matching what borrowers can do in their own dashboard. Previously, agents had to rely on Spectrum directly, causing ~500 payment support calls/month averaging 5 minutes each.

**Five core user stories** are scoped: (1) View all future scheduled OTPs and installment pre-payments — with date, type, amount, status, linked bank account, and an "Installment Pre-Payment" flag; (2) Schedule an OTP in three modes — Custom Amount, Total Payoff Balance, or Installment Pre-payment Schedule — including a calendar modal (default today before 7pm PT else next day, window = next possible date + 31 days), bank account selection, and agent-read consent language before confirmation; (3) Schedule No/No Installment Pre-payments for borrowers with at least 3 months of promo remaining — system calculates and displays monthly amount, estimated interest saved, and payoff date; (4) Cancel OTP with same-day cutoff logic (allow before EOD starts at ~3pm PT, disable after), future-dated cancellations allowed up to 1 business day before scheduled date; (5) Cancel a full Installment Pre-payment Schedule — auto-selects payments >3 business days out, shows a warning and partial-cancel option for payments within 3 business days.

Four email notifications are tied to agent actions: OTP Scheduled (HI_OneTimePaymentConfirmationNoAutoPay_C), OTP Cancelled (HI_C_OneTimePaymentCancelled), Installment Scheduled (HI_C_InstallmentPaymentsScheduled), Installment Cancelled (HI_C_InstallmentPaymentCancelled), plus a reminder (HI_C_InstallmentPaymentReminder). A Spectrum activity is logged for all cancellations. VQ role access (read vs. write) is TBD in the spec. The borrower dashboard gets a companion update (HI-7003) to reuse a new SavingsSummaryBanner component.

## Ticket Map

| Ticket | Summary | Status | Assignee | UT | IT | E2E | PRs |
|---|---|---|---|---|---|---|---|
| HI-7146 | Test Coverage Checklist | Open | Yogesh Chauhan | N/A | N/A | N/A | none |
| HI-7133 | [FE][VQGO] Add permission for OTP feature in VQ | In Validation | — | N/A | N/A | GAP | verification-queue-graphql-orch#1389 |
| HI-7003 | [FE][BD] Use new SavingsSummaryBanner + useHomeImprovementOneTimePay hook | In Validation | — | N/A | N/A | PARTIAL | borrower-dashboard-ui#7519 |
| HI-6824 | [FE][CCP] List existing Scheduled Installment Pre-payments | In Validation | — | N/A | N/A | GAP | app-by-phone-ui#3653, #3643 |
| HI-6804 | [FE][CCP] List existing One-Time Payments | In Validation | — | N/A | N/A | GAP | app-by-phone-ui#3644, #3643 |
| HI-6802 | [FE][CCP] Cancel OTP and No/No Installment Pre-payment Schedules | In Validation | — | N/A | N/A | GAP | app-by-phone-ui#3670, #3643 |
| HI-6801 | [FE][CCP] Schedule Prepayments During Promo Period | In Validation | — | N/A | N/A | GAP | app-by-phone-ui#3740, #3643 |
| HI-6800 | [FE][CCP] Schedule One-Time Payment | In Validation | — | N/A | N/A | GAP | app-by-phone-ui#3679, #3725, #3727, #3643 |
| HI-6787 | [FE][CCP] Extract reusable AutopayItem and centralize autopay logic | In Validation | — | N/A | N/A | GAP | app-by-phone-ui#3753, #3643 |
| HI-7034 | [FE][BD] Fixes and improvements to amount selection | Blocked | — | N/A | N/A | GAP | none found |
| HI-7024 | [FE][CCP] Payment Details for One-time Payment in servicing | Blocked | — | N/A | N/A | GAP | none found |
| HI-7123 | [FE][CCP] Add permission validation | Closed | — | N/A | N/A | GAP | app-by-phone-ui#3779 |
| HI-7078 | [FE][CCP] Design/Product updates | Closed | — | N/A | N/A | GAP | app-by-phone-ui#3771 |
| HI-7021 | [FE][CCP] Amount selection fixes and improvements | Closed | — | N/A | N/A | GAP | app-by-phone-ui#3762 |
| HI-6965 | [FE] Create shared components and hooks in HI components lib | Closed | — | N/A | N/A | GAP | none found (Closed) |
| HI-6903 | [FE][CCP] Add New Bank Account flow for Payment From selection | Closed | — | N/A | N/A | GAP | app-by-phone-ui#3769 |
| HI-6852 | [FE] Updates to use new BE API | Closed | — | N/A | N/A | GAP | app-by-phone-ui#3669 |
| HI-6829 | [BE] Expose additional ScheduledPayment data to FE in HICS GraphQL schema | Closed | — | Done | Done | Done | home-improvement-creditline-srvc#4150, qa-automation#34703, qa-automation-graphql#799 |
| HI-7002 | [FE] Cleanups (post-deployment) | Open | — | N/A | N/A | GAP | none found |

**PR Classification Summary:** 20 unique PRs found across 6 repos.
- UI repos (N/A for UT/IT): app-by-phone-ui (14 PRs), borrower-dashboard-ui (2 PRs) = 16 PRs
- Service repos checked: home-improvement-creditline-srvc#4150 (UT Done, IT Done), verification-queue-graphql-orch#1389 (JS snapshot updated = Done) = 2 PRs
- QA repos (N/A — these ARE the E2E tests): qa-automation#34703, qa-automation-graphql#799 = 2 PRs

## Coverage Matrix

| Requirement Area | UT | IT | E2E | Notes |
|---|---|---|---|---|
| View scheduled OTP list in VQ (date, type, amount, status, bank acct) | N/A | N/A | GAP | FE-only (app-by-phone-ui), no VQ E2E tests |
| View installment pre-payment flag in scheduled list | N/A | N/A | GAP | HI-6824, FE-only |
| Empty state when no OTP/installment scheduled | N/A | N/A | GAP | Spec requirement, no E2E |
| Schedule OTP — Custom Amount (borrower dashboard) | N/A | N/A | PARTIAL | Borrower-side covered (AllureIds 16235, 16228, 16269, 16270, 16239, etc.) |
| Schedule OTP — Custom Amount in VQ/Agent Tool | N/A | N/A | GAP | HI-6800; no VQ-side E2E tests exist |
| Schedule OTP — Total Payoff Balance in VQ | N/A | N/A | GAP | HI-6800; no E2E tests |
| Calendar modal date restrictions (7pm PT cutoff, +31 days) | N/A | N/A | GAP | Spec requirement; not tested in VQ |
| Bank account selection for OTP in VQ | N/A | N/A | GAP | HI-6903 merged; no E2E |
| Agent consent language display (OTP) | N/A | N/A | GAP | Spec requirement; no E2E |
| OTP scheduled confirmation email | N/A | N/A | GAP | HI_OneTimePaymentConfirmationNoAutoPay_C; no VQ-side E2E |
| New bank account flow from Payment From selector | N/A | N/A | GAP | HI-6903; no E2E |
| Schedule Installment Pre-payments (VQ, ≥3 months promo) | N/A | N/A | GAP | HI-6801; no E2E tests |
| Promo calculation display (monthly amount, interest saved, payoff date) | N/A | N/A | GAP | HI-6801; no E2E |
| ACH authorization checkbox (when autopay not active) | N/A | N/A | GAP | Spec requirement; no E2E |
| Recalculate schedule when start date changes | N/A | N/A | GAP | Spec requirement; no E2E |
| Installment scheduled email (HI_C_InstallmentPaymentsScheduled) | N/A | N/A | GAP | No VQ E2E; borrower API test exists (AllureId 19709) |
| Cancel OTP — before EOD (same-day, allow) | N/A | N/A | GAP | HI-6802; no E2E |
| Cancel OTP — after EOD begun (disable + message) | N/A | N/A | GAP | HI-6802; spec requirement |
| Cancel OTP — future-dated (≥1 business day) | N/A | N/A | GAP | HI-6802; no E2E |
| OTP cancel consent language | N/A | N/A | GAP | Spec requirement; no E2E |
| OTP cancelled email (HI_C_OneTimePaymentCancelled) | N/A | N/A | GAP | No E2E |
| Cancel installment pre-payment schedule (full cancel) | N/A | N/A | GAP | HI-6802; no E2E |
| Partial cancel (payment <3 business days from now) — dynamic message | N/A | N/A | GAP | Spec edge case; no E2E |
| Last payment <3 business days — disable cancel CTA | N/A | N/A | GAP | Spec edge case; no E2E |
| Installment cancel email (HI_C_InstallmentPaymentCancelled) | N/A | N/A | GAP | No E2E |
| VQ permission gating (read vs. write roles) | N/A | N/A | GAP | HI-7133, HI-7123; role TBD in spec |
| Spectrum activity logging for cancellations | N/A | N/A | GAP | Technical req; no E2E |
| scheduledPaymentHistory GraphQL query (HICS) | Done | Done | Done | HI-6829; AllureIds 60001, 60002, 60003 |
| Overpayment error when scheduled payment > balance | N/A | N/A | PARTIAL | AllureId 19693 (borrower side); VQ side GAP |
| SavingsSummaryBanner (borrower dashboard) | N/A | N/A | GAP | HI-7003; no E2E for new component |

## Active Gaps

### Critical — Core VQ Feature: No E2E Tests at All
This feature is nearly fully deployed (most FE tickets are Closed/In Validation) but has **zero VQ-side E2E tests**. All existing tests cover the borrower dashboard flow, not the agent tool. The main umbrella PR `app-by-phone-ui#3643` is still open, indicating final integration work is ongoing.

- [HIGH] **View scheduled payments in VQ** — no E2E test; agents cannot verify list renders correctly with correct data (HI-6804, HI-6824)
- [HIGH] **Schedule OTP (Custom Amount) via VQ** — no E2E test; primary user story with consent flow, date picker, bank selection
- [HIGH] **Schedule OTP (Total Payoff Balance) via VQ** — no E2E; payoff calculation must match Spectrum
- [HIGH] **Schedule Installment Pre-payments via VQ** — no E2E; complex calculation with promo eligibility check (HI-6801)
- [HIGH] **Cancel OTP via VQ** — no E2E; EOD cutoff logic is time-sensitive and hard to validate manually (HI-6802)
- [HIGH] **Cancel Installment Schedule via VQ** — no E2E; partial cancel edge case is complex (HI-6802)

### High — EOD Cutoff & Business Day Logic (Hard to Test Manually)
- [HIGH] **Same-day OTP cancel — EOD boundary** — cancel allowed before ~3pm PT, disabled after EOD begins; no automated validation of this time-based state change
- [HIGH] **Future OTP cancel — 1 business day cutoff** — "1 business day" must exclude weekends/holidays; no E2E validation
- [HIGH] **Installment cancel — 3 business day cutoff** — partial cancel triggers dynamic message; no E2E for this path

### Medium — Notification Emails (Unverified for VQ Path)
Borrower-side email tests exist but VQ-triggered emails are separate email templates:
- [MEDIUM] **OTP Scheduled email** via VQ (HI_OneTimePaymentConfirmationNoAutoPay_C) — no E2E asserting email sent when agent schedules OTP
- [MEDIUM] **OTP Cancelled email** via VQ (HI_C_OneTimePaymentCancelled) — no E2E
- [MEDIUM] **Installment Scheduled email** via VQ (HI_C_InstallmentPaymentsScheduled) — borrower API test exists (AllureId 19709) but not VQ-triggered path
- [MEDIUM] **Installment Cancelled email** via VQ (HI_C_InstallmentPaymentCancelled) — no E2E

### Medium — Spec Requirement Gaps (Role Access TBD)
- [MEDIUM] **VQ write role** — spec says "Roles: [TBD]" for write access; once defined, role-based gate tests needed
- [MEDIUM] **VQ read role** — spec says "Roles: [TBD]" for read access
- [MEDIUM] **Spectrum activity logging** — no E2E verifies the `CANPMT IB` / `CT CANPMT` Spectrum activity is logged with correct comment format after cancellation

### Low — Spec Details Not Yet Tested
- [LOW] **Empty state display** — "empty state with prompt to initiate prepayment" when no OTP/schedule exists
- [LOW] **Installment Pre-Payment flag** — "Installment Pre-Payment" label shown in payment list when payment is part of a series
- [LOW] **Consent language accuracy** — agent reads specific text per spec; no automated verification of exact consent copy
- [LOW] **Calendar default date (7pm PT cutoff)** — system shows today vs. next day based on time; requires time-mocking or prod-only timing

### Spec Requirement Gaps
- **Collision case (borrower and agent schedule simultaneously)** — spec lists as open question; no ticket covers handling
- **Overpayment on installment schedule** — spec mentions 110% overpayment limit causes OTP to fail if balance drops; edge case under "Questions" section; no E2E (see also AllureId 19693 for borrower side)
- **Promo installment ACH failure handling** — "Autopay turns off after 3 failed payments" — no E2E for installment-caused ACH failure cascade
- **Installment Pre-Payment Reminder email** (HI_C_InstallmentPaymentReminder) — no E2E trigger or validation

## PR Analysis

### home-improvement-creditline-srvc#4150 — [HI-6829] Expose Additional ScheduledPayment Data to Frontend
**Tickets**: HI-6829
**Functionality**: Exposes new fields on the `scheduledPaymentHistory` GraphQL query — payment ID, type, status, bank account info, effective date — required by both VQ and borrower dashboard to render scheduled payment lists.

**Unit Tests** (Done):
- `ScheduledPaymentFacadeTest.java` — service-layer unit tests for the facade

**Integration Tests** (Done):
- `ScheduledPaymentQueryIT.java` — integration test verifying the GraphQL query returns correct data

**E2E Coverage**:
- Existing tests: AllureIds 60001, 60002, 60003 (in `HomeImprovementBorrowerPrePaymentOptionsTest`)
- Tests cover: no-filter returns all, status filter, bank account populated for ACH payment

---

### verification-queue-graphql-orch#1389 — [HI-7133] Add home improvement scheduled payment permissions in VQ
**Tickets**: HI-7133
**Functionality**: Adds permission entries for the new OTP/pre-payment features in the VQ GraphQL orchestration layer. Controls which agent roles can read vs. write scheduled payments.

**Unit Tests** (Done — snapshot):
- `src/schema/query/user/__tests__/__snapshots__/permissions.test.js.snap` updated; Jest snapshot tests verify permission registry integrity

**E2E Coverage**:
- GAP — no E2E tests verify that VQ UI correctly restricts/allows actions based on the new permissions

---

### app-by-phone-ui#3643 (open umbrella), #3644, #3653, #3669, #3670, #3679, #3725, #3727, #3740, #3753, #3762, #3769, #3771, #3779 — HI-5611 VQ (CCP) feature PRs
**Tickets**: HI-6800, HI-6801, HI-6802, HI-6804, HI-6824, HI-6787, HI-6852, HI-6903, HI-7021, HI-7078, HI-7123
**Functionality**: Full VQ UI implementation — schedule/cancel/view OTP and installment pre-payments in the app-by-phone (CCP tool).

**UT/IT**: N/A (UI repo — React/TypeScript)

**E2E Coverage**: GAP — no qa-automation E2E tests cover any VQ-side payment flows

---

### qa-automation#34703 — [HI-6829] Add tests for scheduledPaymentHistory query
**Tickets**: HI-6829
**Functionality**: Adds three API-level E2E tests for the new `scheduledPaymentHistory` GraphQL query.
- AllureId 60001: No filter returns all scheduled payments
- AllureId 60002: Status filter (SCHEDULED/CANCELLED) works correctly
- AllureId 60003: Bank account field populated for ACH payments

## Key Decisions

- **VQ role access is TBD in spec** — roles for write vs. read access are placeholder "[TBD]"; tests for role gating cannot be written until roles are finalized (consult PM/legal)
- **Borrower-side tests already exist** — `HomeImprovementBorrowerPrePaymentOptionsTest.java` has 20+ tests for the borrower dashboard OTP/promo-payoff flows; VQ tests should be a separate test class (e.g., `HomeImprovementAgentOTPTest.java` or similar)
- **EOD timing dependency** — the 7pm PT cutoff and EOD processing window (~3pm PT) make same-day cancel tests time-sensitive; consider testing via API state manipulation rather than wall-clock time
- **1 business day / 3 business day rules** — these exclude weekends and holidays; test data setup should use a known future business day, not just "now + 1 day"
- **Email template IDs are specified** — OTP Scheduled: `88fdea6d-2f91-42e3-b200-b5d6373f3557`, OTP Cancelled: `e6fd4af7-5070-48be-8da2-e391c7f545e5`, Installment Scheduled: `710c5b75-2cd7-4ed2-9b6c-13e3f704ee58`, Installment Cancelled: `ab51c9cd-c65a-4f1c-80d5-ba2b0bfc4177`, Reminder: `43e0cecd-2465-42b0-afba-4f9bb6730ecd`
- **Spectrum activity logging required** — cancellations must log `CANPMT IB` (phone) or `CT CANPMT` (chat) activity with specific comment format including operator name, channel, scheduled amount/date

## Notes for SDET

- **Existing coverage to extend**: `HomeImprovementBorrowerPrePaymentOptionsTest.java` contains `scheduledPaymentHistoryWithNoFilterReturnsAllPayments` (AllureId 60001), `scheduledPaymentHistoryWithStatusFilterReturnsMatchingPayments` (60002), `scheduledPaymentHistoryBankAccountIsPopulatedForAchPayment` (60003) — these test the BE; VQ UI tests need a new class
- **Page object exists**: `HomeImprovementOneTimePaymentPage.java` is in the framework but covers the borrower dashboard; a new VQ page object for `app-by-phone-ui` is needed
- **Data setup utilities available**: `scheduleHomeImprovementPayment()`, `schedulePromoPayoff()`, `cancelPromoPayoff()` in `HomeImprovementCreditLineServiceUtils` — reusable for test state setup
- **Open qa-auto PR for bug fix**: `qa-automation#34754` (branch `fix/1690851-prepayment-otp-assertions`) fixes a radio button assertion bug in borrower promo payoff tests — this is not new coverage
- **Test checklist ticket**: HI-7146 already exists with VQ/agent scenarios — use it as the test scenario source
- **Priority for E2E**: (1) View scheduled payments list, (2) Schedule OTP via VQ (happy path), (3) Cancel OTP before EOD, (4) Schedule installment pre-payment, (5) Cancel installment schedule
