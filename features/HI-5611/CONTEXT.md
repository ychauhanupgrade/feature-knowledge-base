---
epic: HI-5611
title: HI Borrower One-Time Payment & Pre-payment Scheduling in Agent Tool
spec_url: https://credify.atlassian.net/wiki/spaces/PROD/pages/4697620576/HI+Borrower+One-Time+Payment+Pre-payment+scheduling+in+Agent+Tool
status: in-validation
last_refreshed: 2026-06-26
test_checklist_ticket: HI-7146
confluence_page_id: "5686558776"
---

# HI-5611 — HI Borrower One-Time Payment & Pre-payment Scheduling in Agent Tool

## Spec Summary

This epic adds VQ (Verification Queue / Agent Tool) capabilities so Upgrade phone agents (CCPs) can manage One-Time Payments (OTP) and No/No Installment Pre-payment Schedules on behalf of HI borrowers, matching what borrowers can do in their own dashboard. Previously, agents had to rely on Spectrum directly, causing ~500 payment support calls/month averaging 5 minutes each.

**Five core user stories** are scoped: (1) View all future scheduled OTPs and installment pre-payments — with date, type, amount, status, linked bank account, and an "Installment Pre-Payment" flag (requires ≥1 draw; only upcoming/unprocessed OTP shown, installments show all states); (2) Schedule an OTP in three modes — Custom Amount, Total Payoff Balance, or Installment Pre-payment Schedule — including a calendar modal (default today before 7pm PT else next day, window = next possible date + 31 days, due date highlighted), bank account selection, and agent-read consent language before confirmation; (3) Schedule No/No Installment Pre-payments for borrowers with at least 3 months of promo remaining — system calculates and displays monthly amount, estimated interest saved, and payoff date, recalculates when start date changes, and requires an ACH-authorization checkbox unless autopay is already active; (4) Cancel OTP with same-day cutoff logic (allow before EOD starts at ~3pm PT, disable after with "begun to process" message), future-dated cancellations allowed up to 1 business day before scheduled date; (5) Cancel a full Installment Pre-payment Schedule — auto-selects payments >3 business days out, shows a dynamic warning + partial-cancel option for payments within 3 business days, and disables the CTA when the last payment is within 3 business days. A single payment of a series cannot be cancelled in isolation — the whole series is cancelled and re-generated.

Four email notifications are tied to agent actions: OTP Scheduled (HI_OneTimePaymentConfirmationNoAutoPay_C), OTP Cancelled (HI_C_OneTimePaymentCancelled), Installment Scheduled (HI_C_InstallmentPaymentsScheduled), Installment Cancelled (HI_C_InstallmentPaymentCancelled), plus a reminder (HI_C_InstallmentPaymentReminder). A Spectrum activity (CANPMT IB for phone, CT CANPMT for chat) is logged for all cancellations with a prescribed comment format including operator, channel, scheduled amount/date. VQ role access (read vs. write) is still "[TBD]" in the spec, but implementation migrated the schedule/cancel mutations to SpiceDB `@access` (HI-7367) — only the borrower and CUSTOMER_SUPPORT_OPS can write. The borrower dashboard got a companion update (HI-7003) to reuse a new SavingsSummaryBanner component.

## Ticket Map

| Ticket | Summary | Status | Assignee | UT | IT | E2E | PRs |
|---|---|---|---|---|---|---|---|
| HI-7146 | Test Coverage Checklist | Open | Yogesh Chauhan | N/A | N/A | N/A | none |
| HI-7395 | Update scheduleHomeImprovementPayment permission to deprecate externalBankAccountId | Open | — | GAP | GAP | GAP | none found (no impl yet) |
| HI-7360 | Handle Spectrum processed payments with effective date in the past on VQ | Open | — | GAP | GAP | GAP | none found (no impl yet) |
| HI-7002 | [FE] Cleanups (post-deployment) | Open | Ruan Mer | N/A | N/A | N/A | none found |
| HI-7034 | [FE][BD] fixes and improvements to the amount selection | In Development | Ruan Mer | N/A | N/A | GAP | none found |
| HI-7399 | [FE][VQ] Fix validation when fields change in SelectStep for OTP | Closed | Ruan Mer | N/A | N/A | COVERED | app-by-phone-ui#3841, qa-automation#35854 |
| HI-7367 | Move scheduleHomeImprovementPayment & cancelHomeImprovementScheduledPayment to SpiceDB | Closed | Milton Chambi | Done | N/A | COVERED | home-improvement-creditline-srvc#4458, spicedb-schemas#1627 |
| HI-7358 | [FE][VQ] Show OTP button where principal is zero but interest is non-zero | Closed | Ruan Mer | N/A | N/A | COVERED | app-by-phone-ui#3837 |
| HI-7148 | [FE][CCP] Add AutopayItem for One-time Payment Payment Details | Closed | Ruan Mer | N/A | N/A | COVERED | app-by-phone-ui#3788 |
| HI-7133 | [FE][VQGO] Add permission for managing the OTP feature in VQ | Closed | Ruan Mer | N/A | N/A | COVERED | verification-queue-graphql-orch#1389 |
| HI-7123 | [FE][CCP] Add permission validation | Closed | Ruan Mer | N/A | N/A | COVERED | app-by-phone-ui#3779 |
| HI-7078 | [FE][CCP] Design/Product updates | Closed | Ruan Mer | N/A | N/A | COVERED | app-by-phone-ui#3771 |
| HI-7024 | [FE][CCP] Payment Details for One-time Payment in servicing | Closed | Ruan Mer | N/A | N/A | COVERED | app-by-phone-ui (servicing) |
| HI-7021 | [FE][CCP] fixes and improvements to the amount selection | Closed | Ruan Mer | N/A | N/A | COVERED | app-by-phone-ui#3762 |
| HI-7003 | [FE][BD] use new SavingsSummaryBanner + payoff-quote hook | Closed | Ruan Mer | N/A | N/A | COVERED | borrower-dashboard-ui#7519 |
| HI-6965 | [FE] Create shared components and hooks in HI components lib | Closed | Ruan Mer | N/A | N/A | N/A | shared lib (no app-level test) |
| HI-6903 | [FE][CCP] Add New Bank Account flow for Payment From selection | Closed | Ruan Mer | N/A | N/A | COVERED | app-by-phone-ui#3769 |
| HI-6852 | [FE] Updates to use new BE API | Closed | Ruan Mer | N/A | N/A | COVERED | app-by-phone-ui#3669 |
| HI-6829 | [BE] Expose additional ScheduledPayment data to FE in HICS GraphQL schema | Closed | Milton Chambi | Done | Done | COVERED | home-improvement-creditline-srvc#4150, qa-automation#34703, qa-automation-graphql#799 |
| HI-6824 | [FE][CCP] List existing Scheduled Installment Pre-payments | Closed | Ruan Mer | N/A | N/A | COVERED | app-by-phone-ui#3653, #3643 |
| HI-6804 | [FE][CCP] List existing One-Time Payments | Closed | Ruan Mer | N/A | N/A | COVERED | app-by-phone-ui#3644, #3643 |
| HI-6802 | [FE][CCP] Cancel OTP and No/No Installment Pre-payment Schedules | Closed | Ruan Mer | N/A | N/A | COVERED | app-by-phone-ui#3670, #3643 |
| HI-6801 | [FE][CCP] Schedule Prepayments During Promo Period | Closed | Ruan Mer | N/A | N/A | COVERED | app-by-phone-ui#3740, #3643 |
| HI-6800 | [FE][CCP] Schedule One-Time Payment | Closed | Ruan Mer | N/A | N/A | COVERED | app-by-phone-ui#3679, #3725, #3727, #3643 |
| HI-6787 | [FE][CCP] Extract reusable AutopayItem and centralize autopay logic | Closed | Ruan Mer | N/A | N/A | COVERED | app-by-phone-ui#3753, #3643 |

**Related (not a child of HI-5611):** HI-7466 — "Assert payment-timing disclosure on HI one-time-payment SelectStep" — adds plan-type disclosure assertions to the VQ OTP SelectStep. IN DEV via **qa-automation#36128** (open, branch `HI-7466--UpdateDisclaimerValidation`).

**PR Classification Summary:** ~26 unique PRs across 8 repos.
- UI repos (N/A for UT/IT): app-by-phone-ui (~16 PRs), borrower-dashboard-ui (HI-7003) — all N/A
- Service repos checked: home-improvement-creditline-srvc#4150 (UT Done, IT Done — HI-6829), home-improvement-creditline-srvc#4458 (UT Done — `AuthorizationChecksTest`, HI-7367)
- Infra/schema (N/A): verification-queue-graphql-orch#1389 (JS snapshot), spicedb-schemas#1627 (schema)
- QA repos (N/A — these ARE the E2E): qa-automation#34703, #35854, #36128 (open), qa-automation-graphql#799

## Coverage Matrix

| Requirement Area | UT | IT | E2E | Notes |
|---|---|---|---|---|
| View scheduled OTP/installment list in VQ (date, type, amount, status, bank, flag) | N/A | N/A | COVERED | 64213 (payment details non-promo), 64536 (autopay status via tab nav); scheduling tests render the list |
| Empty state when no OTP/installment scheduled | N/A | N/A | GAP | Spec requirement; not explicitly asserted |
| Only upcoming OTP shown / installments show all states | N/A | N/A | GAP | Spec display rule; not explicitly asserted |
| Schedule OTP — Custom Amount in VQ | N/A | N/A | COVERED | 64077 |
| Schedule OTP — Total Payoff Balance in VQ | N/A | N/A | COVERED | 64490, 64491 (No/No plan); payoff breakdown via HomeImprovementAgentPayoffQuoteTest |
| Payoff Amount info-icon breakdown (principal/interest/fee) | N/A | N/A | COVERED | 56846, 64736 (aggregates sublines), 64739–64750 (payoff quote suite) |
| Calendar modal — default date (7pm PT), +31-day window, due-date highlight | N/A | N/A | GAP | Calendar-specific date rules not asserted in VQ |
| Bank account selection / add new bank for OTP in VQ | N/A | N/A | COVERED | 64537, 63615 (add bank activates autopay) |
| Agent consent / script language (OTP) | N/A | N/A | COVERED | 64479, 64480, 64481, 64483, 66701 (scripts flip with autopay) |
| Confirmation/processing pop-up language | N/A | N/A | COVERED | 64483 (confirm + success scripts) |
| OTP overpayment / amount-exceeds-balance error | N/A | N/A | COVERED | 64537, 64540, 64546, 19693 |
| Schedule Installment Pre-payments (VQ, ≥3 months promo) | N/A | N/A | COVERED | 64079 |
| Installment option hidden when promo < 3 months | N/A | N/A | COVERED | 64538 |
| Promo calc display (monthly amount, interest saved, payoff date) | N/A | N/A | PARTIAL | Schedule + confirm script covered (64079, 64484); explicit interest-saved/payoff-date value assertion borrower-side (16562) |
| Recalculate schedule when start date changes | N/A | N/A | GAP | Spec requirement; not asserted in VQ |
| ACH authorization checkbox (gated by autopay state) | N/A | N/A | COVERED | 64555 (blocked when autopay off), 64556 (allowed when on) |
| Plan-type disclosures on SelectStep | N/A | N/A | IN DEV | HI-7466 / qa-automation#36128 (open) |
| Cancel OTP — before EOD (same-day, allow) | N/A | N/A | COVERED | 64078, 64492 (No/No) |
| Cancel OTP — after EOD / within 1 business day (disable + message) | N/A | N/A | COVERED | 64559 |
| OTP cancel consent language + decline keeps payment | N/A | N/A | COVERED | 64486 (reads back name), 64548 (decline) |
| OTP cancelled email (HI_C_OneTimePaymentCancelled) | N/A | N/A | COVERED | 64558 |
| Cancel installment schedule (full cancel, auto-select >3 days) | N/A | N/A | COVERED | 64080 |
| Partial cancel (<3 business days) — dynamic "cancel remaining" message | N/A | N/A | COVERED | 64553 |
| Last payment <3 business days — disable cancel CTA | N/A | N/A | COVERED | 64552 |
| Installment cancel email (HI_C_InstallmentPaymentCancelled) + decline | N/A | N/A | COVERED | 64550, 64551 (decline keeps schedule) |
| VQ permission gating (read-only role cannot schedule/cancel) | N/A | N/A | COVERED | 64488; backed by HI-7367 SpiceDB @access |
| Cannot schedule for closed loan | N/A | N/A | COVERED | 64081 |
| Zero-principal-but-interest-accrued OTP button shown | N/A | N/A | COVERED | 66015, 68039, 68664 (delinquent due-amount) |
| OTP Scheduled email (HI_OneTimePaymentConfirmationNoAutoPay_C) | N/A | N/A | GAP | VQ-triggered scheduled email not asserted (cancellation email is) |
| Installment Scheduled email (HI_C_InstallmentPaymentsScheduled) | N/A | N/A | GAP | VQ-triggered scheduled email not asserted; borrower API exists (19709) |
| Installment Reminder email (HI_C_InstallmentPaymentReminder) | N/A | N/A | GAP | No trigger/validation |
| Spectrum activity logging for cancellations (CANPMT IB / CT CANPMT) | N/A | N/A | GAP | Technical req; no E2E asserts the activity/comment format |
| scheduledPaymentHistory GraphQL query (HICS) | Done | Done | COVERED | HI-6829; 60001, 60002, 60003 |
| SpiceDB @access auth migration (schedule/cancel mutations) | Done | N/A | COVERED | HI-7367; `AuthorizationChecksTest` (UT); 64488 (E2E negative) |
| SavingsSummaryBanner (borrower dashboard) | N/A | N/A | COVERED | HI-7003; borrower prepayment-options suite |

## Active Gaps

Coverage flipped this refresh — the VQ One-Time-Payment / Pre-payment E2E suite (`HI_BORROWER_PREPAYMENT_OPTIONS_VQ`, 35+ tests, AllureIds 64077–64559 / 66015 / 66701 / 68664) plus the agent payoff-quote (56846, 64736–64750) and agent-autopay (63613–63615) suites have **merged to master**. Nearly every requirement previously flagged GAP is now COVERED. Remaining gaps are narrow:

### Medium — Notification & Spectrum-Activity (asymmetric coverage)
- [MEDIUM] **OTP Scheduled email via VQ** (HI_OneTimePaymentConfirmationNoAutoPay_C) — cancellation email is asserted (64558) but the *scheduled* confirmation email is not
- [MEDIUM] **Installment Scheduled email via VQ** (HI_C_InstallmentPaymentsScheduled) — cancellation email is asserted (64550) but the *scheduled* email is not (borrower API path covered by 19709)
- [MEDIUM] **Spectrum activity logging for cancellations** — no E2E verifies the `CANPMT IB` (phone) / `CT CANPMT` (chat) activity is logged with the prescribed comment format (operator, channel, amount, date)

### Medium — Calendar & Recalculation Mechanics
- [MEDIUM] **Calendar modal date rules** — default-date 7pm PT cutoff, next-possible-date + 31-day editable window, and due-date highlight are not asserted in VQ (time-sensitive; consider state/clock control)
- [MEDIUM] **Installment recalculation on start-date change** — spec: moving the start date later reduces remaining months and raises the monthly amount; not asserted

### Low — Display Rules & Reminder
- [LOW] **Empty state** — "initiate a prepayment/OTP" prompt when none scheduled
- [LOW] **List filtering rule** — only upcoming/unprocessed OTP shown; installments show all states (processed/cancelled)
- [LOW] **Promo calc value assertion in VQ** — explicit interest-saved / payoff-date dollar values shown to the agent (schedule + script are covered; exact displayed values are borrower-side)
- [LOW] **Installment Reminder email** (HI_C_InstallmentPaymentReminder) — no trigger or validation

### New Tickets — No Implementation Yet (E2E GAP, future)
- [MEDIUM] **HI-7360 (Open)** — "Handle Spectrum processed payments with effective date in the past on VQ" — no impl/PR found; new edge case for past-effective-date processed payments in the VQ list. No E2E.
- [MEDIUM] **HI-7395 (Open)** — "Update scheduleHomeImprovementPayment permission to deprecate externalBankAccountId" — no impl/PR found; permission/contract change. Will need regression once landed.
- [LOW] **HI-7034 (In Development)** — "[FE][BD] fixes and improvements to amount selection" — borrower-dashboard amount-selection polish; no PR yet.

### Spec Requirement Gaps / Open Questions (carried from spec)
- **Collision case (borrower + agent schedule simultaneously)** — spec open question ("check how PL handles"); no ticket/test covers handling
- **Overpayment after balance reduced (110% limit)** — spec "Need to test in pre-prod"; COVERED by 64540 (OTP exceeding overpayment limit rejected after balance reduced) and 64546
- **Promo installment ACH failure cascade** — "Autopay turns off after 3 failed payments"; no E2E for installment-caused ACH failure
- **Role access still "[TBD]" in spec** — implementation landed via HI-7367 SpiceDB (borrower + CUSTOMER_SUPPORT_OPS write); 64488 asserts read-only role is denied. A dedicated positive/negative matrix across other ops roles (CREDIT_OPS / ACCOUNT_SERVICING denied) would harden this — [LOW]

## PR Analysis

### home-improvement-creditline-srvc#4150 — [HI-6829] Expose Additional ScheduledPayment Data to Frontend
**Tickets**: HI-6829
**Functionality**: Exposes new fields on the `scheduledPaymentHistory` GraphQL query — payment ID, type, status, bank account info, effective date — required by both VQ and borrower dashboard to render scheduled payment lists.

**Unit Tests** (Done): `ScheduledPaymentFacadeTest.java`
**Integration Tests** (Done): `ScheduledPaymentQueryIT.java`
**E2E Coverage**: AllureIds 60001, 60002, 60003 in `HomeImprovementBorrowerPrePaymentOptionsTest` (no-filter, status filter, bank-account-populated)

---

### home-improvement-creditline-srvc#4458 — [HI-7367] Migrate schedule/cancel HI payment mutations to SpiceDB @access
**Tickets**: HI-7367
**Functionality**: Moves `scheduleHomeImprovementPayment` and `cancelHomeImprovementScheduledPayment` from legacy authorization to SpiceDB `@access`. Only the borrower and CUSTOMER_SUPPORT_OPS may write; CREDIT_OPS / ACCOUNT_SERVICING are denied. Paired with spicedb-schemas#1627 (schema permissions).

**Unit Tests** (Done): `AuthorizationChecksTest.java` — asserts the new access checks
**E2E Coverage**: 64488 (`verifyVqReadOnlyRoleCannotScheduleOrCancelOtpTest`). NOTE (from prior investigation): the VQ "merchant ops" token actually maps to the CUSTOMER_SUPPORT_OPS role despite its name — qa-automation#35854 switched the VQ OTP tests to use it.

---

### verification-queue-graphql-orch#1389 — [HI-7133] Add HI scheduled-payment permissions in VQ
**Tickets**: HI-7133
**Functionality**: Permission entries for OTP/pre-payment read vs. write in the VQ GraphQL orchestration layer.
**Unit Tests** (N/A — JS snapshot): `permissions.test.js.snap` updated
**E2E Coverage**: COVERED indirectly via 64488 (read-only role gating)

---

### app-by-phone-ui (HI-5611 VQ/CCP feature PRs)
#3643 (umbrella), #3644, #3653, #3669, #3670, #3679, #3725, #3727, #3740, #3753, #3762, #3769, #3771, #3779, #3788 (HI-7148), #3837 (HI-7358), #3841 (HI-7399)
**Tickets**: HI-6800, HI-6801, HI-6802, HI-6804, HI-6824, HI-6787, HI-6852, HI-6903, HI-7021, HI-7078, HI-7123, HI-7148, HI-7358, HI-7399
**Functionality**: Full VQ UI — schedule/cancel/view OTP and installment pre-payments in the app-by-phone (CCP) tool; AutopayItem for payment details; zero-principal-but-interest OTP button; SelectStep field-change validation fix.
**UT/IT**: N/A (React/TypeScript UI repo)
**E2E Coverage**: COVERED — the `HI_BORROWER_PREPAYMENT_OPTIONS_VQ` suite (below) exercises these flows.

---

### qa-automation#34703 — [HI-6829] scheduledPaymentHistory query tests
**Tickets**: HI-6829
- 60001: no-filter returns all; 60002: status filter; 60003: bank account populated for ACH

### qa-automation#35854 — [HI-7399] Use Merchant Ops login for VQ OTP (merged)
**Tickets**: HI-7399 (supports HI-7367 auth change)
**Functionality**: Switches the VQ OTP E2E tests to authenticate with the merchant-ops (= CUSTOMER_SUPPORT_OPS) token so the SpiceDB-gated schedule/cancel mutations succeed.

### qa-automation#36128 — [HI-7466] Assert payment-timing disclosure on SelectStep (OPEN, IN DEV)
**Tickets**: HI-7466 (related; not a child of HI-5611)
**Functionality**: Adds plan-type payment-timing disclosure assertions to the VQ OTP SelectStep. Current working branch `HI-7466--UpdateDisclaimerValidation`.

---

### VQ E2E Suite on master — `HomeImprovementBorrowerPrePaymentOptionsTest` (`HI_BORROWER_PREPAYMENT_OPTIONS_VQ`)
| AllureId | Test |
|---|---|
| 64077 | verifyVqAgentCanScheduleCustomAmountOtp |
| 64078 | verifyVqAgentCanCancelScheduledOtp |
| 64079 | verifyVqAgentCanScheduleInstallmentPrePayment |
| 64080 | verifyVqAgentCanCancelInstallmentPrePaymentSchedule |
| 64081 | verifyVqAgentCannotSchedulePaymentForClosedLoan |
| 64213 | verifyVqPaymentDetailsForNonPromoOpenLoan |
| 64479 | verifyVqSelectStepAgentScriptsForYesPaymentPlan |
| 64480 | verifyVqPaymentApplicationScriptWithAutopay |
| 64481 | verifyVqPayoffBalanceScriptForZeroInterestPlan |
| 64482 | verifyVqSelectStepAgentScriptsForNoNoPromoPlan |
| 64483 | verifyVqConfirmAndSuccessAgentScriptsForOtp |
| 64484 | verifyVqConfirmAgentScriptForInstallment |
| 64486 | verifyVqCancelPromoModalReadsBackBorrowerName |
| 64487 | verifyVqDelinquentIntroAgentScript |
| 64488 | verifyVqReadOnlyRoleCannotScheduleOrCancelOtp |
| 64490 | verifyVqAgentCanSchedulePayoffBalanceOtp |
| 64491 | verifyVqAgentCanSchedulePayoffBalanceOtpForNoNoPlan |
| 64492 | verifyVqAgentCanCancelOtpForNoNoPlan |
| 64536 | verifyVqAutopayStatusShownOnPaymentDetailsViaTabNavigation |
| 64537 | verifyVqAmountAndBankValidationErrorsAndAddBankAccount |
| 64538 | verifyVqInstallmentPrePaymentOptionHiddenWhenPromoUnderThreeMonths |
| 64540 | verifyOtpExceedingOverpaymentLimitRejectedAfterBalanceReduced |
| 64546 | verifyVqOverpaymentScheduledErrorWhenSchedulingExceedsBalance |
| 64548 | verifyVqOtpCancelDeclineKeepsPayment |
| 64550 | verifyVqInstallmentScheduleCancellationEmailSent |
| 64551 | verifyVqInstallmentCancelDeclineKeepsSchedule |
| 64552 | verifyVqInstallmentLastPaymentCannotBeCancelledWithinThreeDays |
| 64553 | verifyVqInstallmentProcessingShowsCancelRemainingMessage |
| 64555 | verifyVqRecurringPaymentBlockedWhenAutopayOff |
| 64556 | verifyVqRecurringPaymentAllowedWhenAutopayOn |
| 64558 | verifyVqOtpCancellationEmailSent |
| 64559 | verifyVqOtpCannotBeCancelledWithinOneBusinessDay |
| 66015 | verifyVqOtpScheduleButtonShownWhenZeroPrincipalButInterestAccrued |
| 66701 | verifyVqPaymentApplicationScriptFlipsWithAutopayState |
| 68664 | verifyVqOtpScheduleDueAmountPaymentForDelinquentZeroPrincipalButInterestAccrued |

### Agent payoff-quote suite — `HomeImprovementAgentPayoffQuoteTest` (`HI_PAYOFF_QUOTE`)
56846, 64736 (aggregate across sublines), 64737, 64738, 64739, 64740, 64741, 64742, 64743, 64745, 64746, 64747, 64748, 64749, 64750

### Agent autopay suite — `HomeImprovementAgentAutopayTest` (`HI_SAVING_CALCULATOR_AND_AUTOPAY`)
63613 (bank dropdown + enable/disable emails), 63614 (cutoff warning + next effective date within 3 days), 63615 (add new bank in VQ activates autopay)

## Key Decisions

- **Coverage flipped to COVERED** — as of this refresh the VQ OTP/pre-payment feature is broadly E2E-covered on master; the prior "zero VQ E2E tests" assessment is obsolete.
- **VQ "merchant ops" token = CUSTOMER_SUPPORT_OPS role** — despite the name; this is the write-eligible role after the HI-7367 SpiceDB migration. Tests authenticate with it (qa-automation#35854).
- **VQ tests live in `HomeImprovementBorrowerPrePaymentOptionsTest`** under the `HI_BORROWER_PREPAYMENT_OPTIONS_VQ` feature — not a separate class as originally planned.
- **Page objects**: `AbpHomeImprovementOneTimePaymentPage`, `AbpHomeImprovementPaymentCalculatorPage`, `AbpHomeImprovementAccountSummaryPage` (app-by-phone servicing) drive the VQ flows.
- **EOD / business-day timing** — same-day cancel disable and 1-/3-business-day cutoffs are driven via DB/state manipulation (e.g. scheduled_payment.status=IN_PROGRESS) rather than wall-clock, since these are time-sensitive.
- **Scheduled vs. cancellation emails** — cancellation emails (OTP 64558, installment 64550) are asserted; the *scheduled* confirmation emails are the remaining notification gap.
- **Email template IDs** — OTP Scheduled: `88fdea6d-2f91-42e3-b200-b5d6373f3557`; OTP Cancelled: `e6fd4af7-5070-48be-8da2-e391c7f545e5`; Installment Scheduled: `710c5b75-2cd7-4ed2-9b6c-13e3f704ee58`; Installment Cancelled: `ab51c9cd-c65a-4f1c-80d5-ba2b0bfc4177`; Reminder: `43e0cecd-2465-42b0-afba-4f9bb6730ecd`.
- **Spectrum activity logging required** — cancellations must POST `CANPMT IB` (phone) / `CT CANPMT` (chat) with the prescribed comment format; no E2E asserts this yet.

## Notes for SDET

- **Remaining E2E to add (priority)**: (1) Spectrum cancellation-activity assertion (CANPMT IB / CT CANPMT comment format), (2) OTP/installment *Scheduled* email assertions, (3) calendar default-date / +31-day window / due-date highlight, (4) installment recalculation when start date changes, (5) empty-state + list-filtering display rules.
- **Watch the new Open tickets**: HI-7360 (past-effective-date processed payments) and HI-7395 (deprecate externalBankAccountId on the schedule permission) have no implementation yet — they will need fresh E2E once PRs land.
- **HI-7466 (qa-automation#36128, open)** adds payment-timing disclosure assertions to the SelectStep — track it to merge.
- **Data setup utilities**: `scheduleHomeImprovementPayment()`, `schedulePromoPayoff()`, `cancelPromoPayoff()`, `backDateSubLineInHIAndSpectrum`, `ensureMonthlyBillExists`, savings-calculator preset setup — all reusable for VQ state setup.
- **Test checklist ticket**: HI-7146 — use as the scenario source.
