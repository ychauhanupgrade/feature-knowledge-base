---
epic: HI-6863
title: Merchant Reporting Improvements
spec_url: https://credify.atlassian.net/wiki/spaces/PROD/pages/5301993558
status: in-development
last_refreshed: 2026-06-10
test_checklist_ticket: HI-7137
confluence_page_id: "5683183849"
---

# HI-6863 — Merchant Reporting Improvements

## Spec Summary

The Merchant Reporting Improvements epic addresses six improvements to HI merchant and parent portal reporting, driven by merchant feedback.

**Requirements:** (1) Transactions Report CSV defaults to funded draws only (removes pending payment requests), ordered by most recent funding date descending; an opt-in toggle allows including pending requests sorted by request date — applies to both merchant and holding-company (parent) portals. (2) Add `merchant_memo` column to Merchant Transactions CSV download — the field already appears in the UI but was missing from downloads. (3) Bug fix: when a merchant belongs to multiple groups, the Parent Overview Report showed duplicate rows (one per group); fix shows a single row with "Multiple Groups" in the Group column. (4) Bug fix: Parent Sales Enrollment Report showed `null` merchant name for Parent Sales reps; fix populates the Parent's own name. (5) Bug fix: null Lead Reference Number showed "No Referral" in Merchant Transactions Report but showed "----" in the Parent portal — align both to display "----". (6) UI enhancement: display a visible reporting-date label on each report tab (Draw = Funding Date or Request Creation Date; Loan = App Creation Date; Prequal Leads = Prequalified Date with 30/60/90 days options only); each report tab gets an independent date range selector that persists across tab switches. Both merchant and parent portals are affected by requirement 6.

Backend changes live in `hi-merchant-reporting-srvc` (PR#2265, merged) — covers sort-by-funding-date, multi-group deduplication, and EmployeeDirectory null-name fix. Frontend work is distributed across `merchant-dashboard-ui`, `merchant-parent-dashboard-ui`, and `home-improvement-components-ui`. **E2E coverage now lives in `qa-automation#35131` (open, branch `HI-MerchantReportingImpovementChanges`).**

## Ticket Map

| Ticket | Summary | Status | Assignee | UT | IT | E2E | PRs |
|---|---|---|---|---|---|---|---|
| HI-6911 | [BE] Sort Holding Co. and Merchant Transaction reports by funding date | In Validation | Hannan Ahsan | Done | Done | IN DEV | hi-merchant-reporting-srvc#2265, offline-report-client#2433, upflow2-home-improvement-dags#410, qa-automation#35131 (62003, 66449, 65369, 65506, 52544, 52545) |
| HI-6912 | [BE] Misc fixes (multi-group dedup, EmployeeDirectory null name) | Closed | Hannan Ahsan | Done | Done | IN DEV | hi-merchant-reporting-srvc#2265 (combined), upflow2-home-improvement-dags#410, qa-automation#35131 (66453, 66452) |
| HI-6875 | [FE][MD] Lead Ref Number null → "----" | Closed | Marcus Silva | N/A | N/A | IN DEV | merchant-dashboard-ui#761 (merged), qa-automation#35131 (65372) |
| HI-6871 | [FE] Per-report date ranges (Merchant + Parent portals) | Closed | Marcus Silva | N/A | N/A | IN DEV | home-improvement-components-ui#231, merchant-parent-dashboard-ui#153, merchant-dashboard-ui#763 (all merged), qa-automation#35131 (65369, 65370, 65371, 62001, 62002, 66454) |
| HI-6876 | [FE] Manage Email Reports per-report date ranges | Closed | Marcus Silva | N/A | N/A | IN DEV | merchant-dashboard-ui#774, merchant-parent-dashboard-ui#157, home-improvement-components-ui#239, home-improvement-components-ui#232 (merged), qa-automation#35131 (65506, 52541) |
| HI-7027 | [FE][MD] Select funding type for transaction downloads | Closed | Marcus Silva | N/A | N/A | IN DEV | No FE impl PR identified (Dev panel auth blocked); E2E in qa-automation#35131 (62003, 66449, 62004) |
| HI-7304 | [FE] Manage Email Reports success-banner copy update | Closed | Marcus Silva | N/A | N/A | PARTIAL | merchant-dashboard-ui#807 (merged), merchant-parent-dashboard-ui#176 (merged); banner asserted indirectly by scheduling tests 52537–52543 |
| HI-7278 | [BE] Consolidate Parent TRANSACTION + Merchant DRAW report variants (FUNDING_STATUS_FILTER) | Open | — | GAP | GAP | GAP | No impl PR — backend tech-debt (blocked on v2 optional NULL param support) |
| HI-7297 | Consolidate RidgeTop Reports to use same DAGs as MD (incl. date→datetime + merchant-memo CSV change) | Open | — | GAP | GAP | GAP | No impl PR — DAG consolidation; pending RidgeTop confirmation |
| HI-7136 | Test Coverage Checklist (prior, superceded) | Closed | — | N/A | N/A | N/A | — |
| HI-7137 | Test Coverage Checklist (QA task, current) | In Development | Yogesh Chauhan | N/A | N/A | IN DEV | qa-automation#35131 |

**PR Classification Summary:** 1 service PR (hi-merchant-reporting-srvc#2265 — UT Done, IT Done), 1 client PR (offline-report-client#2433 — N/A), 2 infra/DAGs PRs (upflow2-home-improvement-dags#410 — N/A), 11 UI PRs across merchant-dashboard-ui / merchant-parent-dashboard-ui / home-improvement-components-ui (N/A), 1 QA E2E PR (qa-automation#35131 — these ARE the E2E tests). HI-7278 and HI-7297 are open BE/DAG consolidation tech-debt with no implementation PRs yet.

## Coverage Matrix

| Requirement Area | UT | IT | E2E | Notes |
|---|---|---|---|---|
| Merchant Transactions CSV: funded draws only by default, sorted by funding date desc | Done | Done | IN DEV | HI-6911; E2E 62003/66449 (funding type → legacy vs funded), 52544/52545 (date range), 65369 (DRAWS label) |
| Merchant Transactions CSV: include-pending option sorted by request date | Done | Done | IN DEV | HI-6911; E2E 65506 (schedule All Draws/pending), 62003, 66449 |
| Parent (Holding Co.) Transactions CSV: funded-only default, sorted by funding date | Done | Done | IN DEV | HI-6912; E2E 62001/62004 |
| Parent Transactions CSV: include-pending option | Done | Done | IN DEV | HI-6912; E2E 62004 (Pending → legacy TRANSACTION report) |
| Merchant Transactions CSV: `merchant_memo` column present with correct value | Done | Done | IN DEV | Req 2; **NEW** merchant-portal E2E 66526 asserts memo column present in DRAWS CSV + cell value matches GQL `paymentRequestMerchantMemo`; parent-side covered by 51267 (column 18). Gap closed. |
| Parent Overview Report: multi-group merchant → single row, "Multiple Groups" label | Done | Done | IN DEV | HI-6912; E2E 66453 |
| Parent Sales Enrollment Report: Parent Sales rep → Parent name (not null) | Done | Done | IN DEV | HI-6912; E2E 66452 |
| Merchant Transactions UI + CSV: null Lead Ref Number → "----" (not "No Referral") | N/A | N/A | IN DEV | HI-6875; E2E 65372 (dash placeholder, not "No Referral") |
| Merchant Reporting UI: Draw/Loan report date labels | N/A | N/A | IN DEV | HI-6871; E2E 65369 (DRAWS Funding/Request Date label toggle) |
| Merchant Reporting UI: Prequal Leads → "Prequalified Date" with 30/60/90 options only | N/A | N/A | IN DEV | HI-6871; E2E 65370 (date range restricted to 30/60/90) |
| Parent Reporting UI: Draw/Loan/Prequal date labels | N/A | N/A | IN DEV | HI-6871; E2E 62001, 66454 |
| Manage Email Reports: per-report date range | N/A | N/A | IN DEV | HI-6876; E2E 65506, 52541 |
| UI: per-tab date range selector independence (changing one tab doesn't reset others) | N/A | N/A | IN DEV | HI-6871/6876; E2E 65371 (merchant), 62002 (parent) |
| Funding type selector in transaction downloads (UI) | N/A | N/A | IN DEV | HI-7027; E2E 62003, 66449, 62004 |
| Manage Email Reports success-banner copy | N/A | N/A | PARTIAL | HI-7304; banner asserted indirectly by scheduling tests 52537–52543; no dedicated copy assertion |
| BE report-variant consolidation (FUNDING_STATUS_FILTER) | GAP | GAP | GAP | HI-7278 (Open, no impl PR) — future tech-debt |
| RidgeTop DAG consolidation (date→datetime, merchant-memo) | GAP | GAP | GAP | HI-7297 (Open, no impl PR) — future |

## Active Gaps

### Immediate QA Gaps (In Validation — needs testing now)
- [MEDIUM] **HI-6911 still In Validation** — only ticket not yet Closed. E2E exists in qa-automation#35131 (open); run the suite on the validation env to close it out.
- [LOW] **HI-7027 has no identified FE implementation PR** — ticket is now Closed and E2E exists (62003/66449/62004), but the front-end PR was never surfaced (Jira Dev-panel auth blocked). Cosmetic gap only; confirm with Marcus Silva if needed.

### Medium — E2E partials to harden
- [RESOLVED 2026-06-10] **Req 2: merchant_memo in MERCHANT-portal CSV** — closed by new E2E 66526 (`verifyMerchantTransactionsCsvIncludesMerchantMemoColumnTest`), which asserts the memo column is present in the merchant-portal DRAWS CSV download and the cell value matches the GQL source memo. No further action needed beyond running the suite.
- [LOW] **HI-7304: success-banner copy** — banner is exercised by existing scheduling tests (52537–52543) but no test asserts the exact updated copy. Add a copy assertion if the wording is contractually important. (See launch 1769611 RCA — prior banner-copy failures were preprod UI deploy lag, not assertion bugs.)

### Low — Future tech-debt (no implementation yet, not QA-blocking)
- [LOW] **HI-7278 (Open)** — BE consolidation of funded/non-funded report variants into one report + `FUNDING_STATUS_FILTER` param; blocked on v2 GraphQL optional-NULL parameter support. No impl PR. Re-evaluate E2E scope when implemented.
- [LOW] **HI-7297 (Open)** — RidgeTop DAG consolidation; includes a CSV change (date → datetime) and merchant-memo addition pending RidgeTop confirmation. No impl PR. Watch for CSV-format regressions when implemented.

### Spec Requirement Gaps
- **None outstanding.** As of 2026-06-10, all spec requirements (1, 2, 3, 4, 5, 6 incl. prequal 30/60/90 and per-tab independence) have IN DEV E2E coverage in qa-automation#35131 for both merchant and parent portals. Req 2 merchant-portal memo column — previously the only sub-requirement without direct merchant-side coverage — is now covered by 66526.

## PR Analysis

### hi-merchant-reporting-srvc#2265 — HI-6911 + HI-6912: FUNDED_DRAWS report variants
**Tickets**: HI-6911, HI-6912
**Status**: MERGED (backend Done)
**Functionality**: Adds `FUNDED_DRAWS` / `FUNDED_TRANSACTION` report variants for merchant and holding-company CSV downloads; sorts by funding date; deduplicates multi-group merchant rows in Parent Overview; fixes EmployeeDirectory null merchant name for Parent Sales reps.

**Unit Tests** (4 files):
- `OrsEventHandlerTest.java`
- `MerchantReportMutationResolverTest.java`
- `TransactionReportServiceTest.java`
- `EmployeeDirectoryReportMapperTest.java`

**Integration Tests** (3 files):
- `ScheduledReportIT.java`
- `TransactionReportResolverIT.java`
- `GenerateMerchantReportIT.java`

**UT/IT Result**: Done — comprehensive service-layer test coverage.
**E2E Gap**: Closed by qa-automation#35131 (see below).

### qa-automation#35131 — HI-6863: E2E tests for merchant reporting improvements
**Tickets**: HI-6911, HI-6912, HI-6875, HI-6871, HI-6876, HI-7027
**Status**: OPEN — branch `HI-MerchantReportingImpovementChanges` (active, several commits ahead of master)
**Functionality**: End-to-end coverage for funding-type selection, per-report date-range labels/independence, prequal 30/60/90 restriction, multi-group dedup, Sales Enrollment Parent-name fix, and null Lead-Ref dash rendering — across both merchant and parent portals (UI + GraphQL API).

**New E2E tests — Merchant** (`HomeImprovementMerchantReportingTest`):
- 65369 `verifyDrawsDateRangeLabelChangesByFundingTypeTest` — DRAWS label "Funding Date Range" → "Request Date Range" on Pending (Req 6 + Req 1)
- 65370 `verifyPrequalLeadsDateRangeIsRestrictedTest` — Prequal Leads restricted to Last 30/60/90 (Req 6c)
- 65371 `verifyIndependentDateRangePerReportTypeTest` — per-report independent date range (Req 6 / per-tab independence)
- 65372 `verifyNullLeadRefNumberDisplaysDashTest` — null Lead Ref → dash, not "No Referral" (Req 5)
- 65506 `verifyAdminCanScheduleAllDrawsReportTest` — schedule "All Draws (pending)" report type (Req 1 include-pending)
- 62003 `verifyDrawsPendingFundingTypePersistsLegacyDrawsTest` — Pending persists legacy DRAWS scheduled report (Req 1 / HI-7027)
- 66449 `verifyDrawsPendingFundingTypeGeneratesLegacyCsvReportTest` — Pending generates legacy DRAWS CSV (Req 1 / HI-7027)
- 66526 `verifyMerchantTransactionsCsvIncludesMerchantMemoColumnTest` (**NEW 2026-06-10**) — merchant-portal DRAWS CSV download includes `merchant_memo` column AND cell value matches GQL `paymentRequestMerchantMemo` (Req 2 merchant-side). `@Jira("HI-6863")`, `@LimitRunToEnv(stage, preprod)`. Closes the prior Req 2 merchant-portal gap.

**New E2E tests — Parent** (`HomeImprovementParentReportingTest`):
- 62001 `verifyTransactionDateRangeLabelChangesByFundingTypeTest` — TRANSACTION date-range label reflects funding type (Req 1/6 parent)
- 62002 `verifyIndependentDateRangePerReportTypeParentTest` — independent per-report date range (Req 6 parent)
- 62004 `verifyTransactionPendingFundingTypeGeneratesLegacyReportTest` — Pending generates legacy TRANSACTION report (Req 1 parent)
- 66452 `verifySalesEnrollmentShowsParentNameForParentSalesRepTest` — Sales Enrollment shows Parent name for Parent sales reps (Req 4)
- 66453 `verifyMultiGroupMerchantTransactionShowsSingleRowWithMultipleGroupsLabelTest` — multi-group → single "Multiple Groups" row (Req 3)
- 66454 `verifyCsvModalDateRangeLabelsPerReportTypeTest` — per-report date-range labels + prequal options in parent CSV modal (Req 6 parent)
- 51267 `verifyCSVDownload` (pre-existing, extended) — asserts Merchant Memo column 18 against GQL (Req 2, parent-side)

**UT/IT**: N/A (E2E repo). **Coverage verdict**: IN DEV — covers Req 1, 2, 3, 4, 5, 6 for both portals. Req 2 now has direct merchant-portal coverage (66526) in addition to parent-side (51267).

## Key Decisions

- **Funded draws = default** — pending payment requests excluded from default CSV download; inclusion requires opt-in. Intentional UX: merchants reconcile against funded transactions in bank statements.
- **"Multiple Groups" label** — single row replaces duplicate rows when merchant is in multiple groups in Parent Overview.
- **Lead Ref null alignment** — "----" (not "No Referral") aligns merchant portal display with Parent portal.
- **Prequal Leads date range** restricted to 30/60/90 days only — arbitrary date ranges unsupported for Prequal.
- **HI-6911 + HI-6912 share one backend PR** — hi-merchant-reporting-srvc#2265 covers both tickets.
- **Funded vs legacy reports kept as separate variants** — HI-7278 (Open) tracks the future consolidation into a single report + `FUNDING_STATUS_FILTER` param; blocked on v2 GraphQL optional-NULL support, so the duplicated funded/non-funded report definitions remain for now.
- **Pending funding type persists/generates the LEGACY report** (DRAWS / TRANSACTION), not the new FUNDED variant — asserted by 62003/66449 (merchant) and 62004 (parent).

## Notes for SDET

- **Active qa-automation branch / PR**: `HI-MerchantReportingImpovementChanges` → qa-automation#35131 (OPEN). All E2E work lives here — do NOT create a new branch.
- **Test checklist**: HI-7137 (In Development) — 14 scenarios across 6 requirements.
- **Primary service under test**: `hi-merchant-reporting-srvc` (GraphQL API) — use `HiMerchantReportingService` / `HiMerchantReportingServiceUtils`.
- **Test classes**: `HomeImprovementMerchantReportingTest`, `HomeImprovementParentReportingTest`; page objects under `playwright/pages/homeimprovement/merchantdashboard/reporting/` and `playwright/pages/homeimprovement/parent/`.
- **REAL DATA ONLY rule** (see memory) — reporting-table inserts must use real funded/draw data, no synthetic rows; multi-group SQL uses `to_jsonb`; null fields render as em-dash.
- **Remaining work**: (1) ~~add merchant-portal CSV memo-column assertion (Req 2)~~ — DONE 2026-06-10 via 66526; (2) optionally assert HI-7304 banner copy; (3) close out HI-6911 by running the suite on the validation env.
- **Jira dev info script blocked**: `jira-basic-auth` keychain entry is MISSING (lookup returns "item could not be found"). PR discovery this refresh used `gh search prs` + Atlassian remote-links instead. Fix: `security add-generic-password -a "$USER" -s "jira-basic-auth" -w "$(echo -n 'ychauhan@upgrade.com:YOUR_API_TOKEN' | base64)"`
- **HI-7027 FE PR gap**: front-end "Select funding type" PR never surfaced (Dev-panel auth blocked); ticket is Closed and E2E exists, so this is cosmetic only.
