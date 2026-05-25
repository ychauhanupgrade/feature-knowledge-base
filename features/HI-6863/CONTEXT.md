---
epic: HI-6863
title: Merchant Reporting Improvements
spec_url: https://credify.atlassian.net/wiki/spaces/PROD/pages/5301993558
status: in-development
last_refreshed: 2026-05-24
test_checklist_ticket: HI-7137
confluence_page_id: "5683183849"
---

# HI-6863 — Merchant Reporting Improvements

## Spec Summary

The Merchant Reporting Improvements epic addresses six improvements to HI merchant and parent portal reporting, driven by merchant feedback.

**Requirements:** (1) Transactions Report CSV defaults to funded draws only (removes pending payment requests), ordered by most recent funding date descending; an opt-in toggle allows including pending requests sorted by request date — applies to both merchant and holding-company (parent) portals. (2) Add `merchant_memo` column to Merchant Transactions CSV download — the field already appears in the UI but was missing from downloads. (3) Bug fix: when a merchant belongs to multiple groups, the Parent Overview Report showed duplicate rows (one per group); fix shows a single row with "Multiple Groups" in the Group column. (4) Bug fix: Parent Sales Enrollment Report showed `null` merchant name for Parent Sales reps; fix populates the Parent's own name. (5) Bug fix: null Lead Reference Number showed "No Referral" in Merchant Transactions Report but showed "----" in the Parent portal — align both to display "----". (6) UI enhancement: display a visible reporting-date label on each report tab (Draw = Funding Date or Request Creation Date; Loan = App Creation Date; Prequal Leads = Prequalified Date with 30/60/90 days options only); each report tab gets an independent date range selector that persists across tab switches. Both merchant and parent portals are affected by requirement 6.

Backend changes live in `hi-merchant-reporting-srvc` (PR#2265, open) — covers sort-by-funding-date, multi-group deduplication, and EmployeeDirectory null-name fix. Frontend work is distributed across `merchant-dashboard-ui`, `merchant-parent-dashboard-ui`, and `home-improvement-components-ui`.

## Ticket Map

| Ticket | Summary | Status | Assignee | UT | IT | E2E | PRs |
|---|---|---|---|---|---|---|---|
| HI-6911 | [BE] Sort Holding Co. and Merchant Transaction reports by funding date | In Validation | Hannan Ahsan | Done | Done | GAP | hi-merchant-reporting-srvc#2265, offline-report-client#2433, upflow2-home-improvement-dags#410 |
| HI-6912 | [BE] Misc fixes (multi-group dedup, EmployeeDirectory null name) | Closed | Hannan Ahsan | Done | Done | GAP | hi-merchant-reporting-srvc#2265 (combined), upflow2-home-improvement-dags#410 |
| HI-6875 | [FE][MD] Lead Ref Number null → "----" | Closed | Marcus Silva | N/A | N/A | GAP | merchant-dashboard-ui#761 (merged) |
| HI-6871 | [FE] Per-report date ranges (Merchant + Parent portals) | Closed | Marcus Silva | N/A | N/A | GAP | home-improvement-components-ui#231 (merged), merchant-parent-dashboard-ui#153 (merged), merchant-dashboard-ui#763 (merged) |
| HI-6876 | [FE] Manage Email Reports per-report date ranges | In Validation | Marcus Silva | N/A | N/A | GAP | merchant-dashboard-ui#774 (open), merchant-parent-dashboard-ui#157 (open), home-improvement-components-ui#239 (merged), home-improvement-components-ui#232 (merged) |
| HI-7027 | [FE][MD] Select funding type for transaction downloads | In Validation | Marcus Silva | N/A | N/A | GAP | No impl PR identified (only k8s deploy PRs found) |
| HI-7136 | Test Coverage Checklist (prior, superceded) | Closed | — | N/A | N/A | N/A | — |
| HI-7137 | Test Coverage Checklist (QA task, current) | Open | Yogesh Chauhan | N/A | N/A | IN DEV | — |

**PR Classification Summary:** 1 service PR (hi-merchant-reporting-srvc#2265 — UT Done, IT Done), 1 client PR (offline-report-client#2433 — N/A), 2 infra/DAGs PRs (N/A), 9 UI PRs across merchant-dashboard-ui / merchant-parent-dashboard-ui / home-improvement-components-ui (N/A).

## Coverage Matrix

| Requirement Area | UT | IT | E2E | Notes |
|---|---|---|---|---|
| Merchant Transactions CSV: funded draws only by default, sorted by funding date desc | Done | Done | GAP | HI-6911; TransactionReportResolverIT + TransactionReportServiceTest in PR#2265 |
| Merchant Transactions CSV: include-pending option sorted by request date | Done | Done | GAP | HI-6911; same PR |
| Parent (Holding Co.) Transactions CSV: funded-only default, sorted by funding date | Done | Done | GAP | HI-6912; combined with PR#2265 |
| Parent Transactions CSV: include-pending option | Done | Done | GAP | HI-6912; same PR |
| Merchant Transactions CSV: `merchant_memo` column present with correct value | Done | Done | GAP | HI-6911/6912; EmployeeDirectoryReportMapperTest covers field mapping |
| Parent Overview Report: multi-group merchant → single row, "Multiple Groups" label | Done | Done | GAP | HI-6912; dedup logic in PR#2265 |
| Parent Sales Enrollment Report: Parent Sales rep → Parent name (not null) | Done | Done | GAP | HI-6912; EmployeeDirectoryReportMapper fix |
| Merchant Transactions UI + CSV: null Lead Ref Number → "----" (not "No Referral") | N/A | N/A | GAP | HI-6875; FE-only fix in merchant-dashboard-ui#761 (merged) |
| Merchant Reporting UI: Draw/Loan report date labels | N/A | N/A | GAP | HI-6871; merchant-dashboard-ui#763 (merged) |
| Merchant Reporting UI: Prequal Leads → "Prequalified Date" with 30/60/90 options only | N/A | N/A | GAP | HI-6871; date restriction specific to prequal tab |
| Parent Reporting UI: Draw/Loan/Prequal date labels | N/A | N/A | GAP | HI-6871; merchant-parent-dashboard-ui#153 (merged) |
| Manage Email Reports: per-report date range | N/A | N/A | GAP | HI-6876; PRs still open |
| UI: per-tab date range selector independence (changing one tab doesn't reset others) | N/A | N/A | GAP | HI-6871/6876 |
| Funding type selector in transaction downloads (UI) | N/A | N/A | GAP | HI-7027; no impl PR identified |

## Active Gaps

### Immediate QA Gaps (In Validation — needs testing now)
- [HIGH] **HI-7027 has no identified implementation PR** — "Select a funding type" is In Validation but no merchant-dashboard-ui or component PR found. Requires Jira Development panel auth to identify actual PRs. Cannot confirm E2E scope without them.
- [HIGH] **E2E for Req 1 (CSV sort / funded-only filter)** — Core deliverable of HI-6911 (In Validation); funded-only default and include-pending toggle need API-level E2E (scenarios 1–4 in HI-7137)
- [HIGH] **E2E for Req 3 (multi-group Overview dedup)** — Bug fix; single-row behavior needs E2E validation (scenarios 6–7 in HI-7137)

### High — Missing E2E Coverage (Merged PRs, No Tests)
- [HIGH] **Req 2: memo field in CSV** — PR merged (HI-6875); no E2E test (scenario 5 in HI-7137)
- [HIGH] **Req 4: null Sales Enrollment name fix** — HI-6912 closed; fix merged but no E2E (scenario 8 in HI-7137)
- [HIGH] **Req 5: Lead Ref Number "----" display** — HI-6875 fully merged; no E2E test added (scenarios 9–10 in HI-7137)

### Medium — UI Feature Gaps
- [MEDIUM] **Req 6: Reporting date labels in Merchant UI** — HI-6871 closed/merged; no E2E for Draw/Loan/Prequal labels (scenarios 11–12 in HI-7137)
- [MEDIUM] **Req 6: Reporting date labels in Parent UI** — HI-6871 merged; no E2E (scenario 13 in HI-7137)
- [MEDIUM] **Per-tab date range independence** — no test for cross-tab interaction (scenario 14 in HI-7137)

### Spec Requirement Gaps
- [MEDIUM] **Prequal Leads 30/60/90 constraint** — spec restricts date range to 3 options; confirm HI-6871 enforces this at UI level (not just a label change)
- [LOW] **HI-7027 scope unclear** — "Select a funding type" may be bundled in another ticket's PR; verify with team

## PR Analysis

### hi-merchant-reporting-srvc#2265 — HI-6911 + HI-6912: FUNDED_DRAWS report variants
**Tickets**: HI-6911, HI-6912
**Status**: OPEN (In Validation)
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
**E2E Gap**: No qa-automation tests yet. Branch `HI-MerchantReportingImpovementChanges` exists with 0 commits ahead of master.

## Key Decisions

- **Funded draws = default** — pending payment requests excluded from default CSV download; inclusion requires opt-in. Intentional UX: merchants reconcile against funded transactions in bank statements.
- **"Multiple Groups" label** — single row replaces duplicate rows when merchant is in multiple groups in Parent Overview.
- **Lead Ref null alignment** — "----" (not "No Referral") aligns merchant portal display with Parent portal.
- **Prequal Leads date range** restricted to 30/60/90 days only — arbitrary date ranges unsupported for Prequal.
- **HI-6911 + HI-6912 share one backend PR** — hi-merchant-reporting-srvc#2265 covers both tickets.

## Notes for SDET

- **Active qa-automation branch**: `HI-MerchantReportingImpovementChanges` (0 commits ahead — start here)
- **Test checklist**: HI-7137 (14 scenarios across 6 requirements)
- **Primary service under test**: `hi-merchant-reporting-srvc` (GraphQL API) — use `HiMerchantReportingService`
- **Existing tests to extend**:
  - 52544 `verifyExportCsvWithDateRangeApiTest` — extend for funded-only vs include-pending
  - 52545 `verifyCsvDataWithinDateRangeApiTest` — extend for memo field verification
  - 51262 `verifyTransactionReportMetricsAccuracy` — verify multi-group row dedup
  - 51263 `verifyEmployeeDirectoryReportMetricsAccuracy` — verify Sales Enrollment name fix
- **Jira dev info script blocked**: `jira-basic-auth` keychain entry exists but is empty. Fix: `security add-generic-password -a "$USER" -s "jira-basic-auth" -w "$(echo -n 'ychauhan@upgrade.com:YOUR_API_TOKEN' | base64)"`
- **HI-7027 PR gap**: "Select funding type" UI ticket (In Validation) has no identified GitHub PR. Confirm with Marcus Silva.
