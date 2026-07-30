---
epic: HI-5039
title: HI Merchant Multi-Account
spec_url: https://credify.atlassian.net/wiki/spaces/PROD/pages/4365615309/HI+Merchant+Multi-Account
status: in-development
last_refreshed: 2026-07-30
test_checklist_ticket: HI-7135
confluence_page_id: "5683675314"
---

# HI-5039 — HI Merchant Multi-Account

## Spec Summary

The HI Merchant Multi-Account feature enables mid-sized merchants to operate multiple accounts under a single password-based login, without requiring SSO. This is explicitly distinct from the Parent Portal (which uses Okta SSO, merchant groups, and IT-level integrations). Each account under a multi-account maintains independent bank accounts, plan selections (pricebooks), locations, settings, employees, and reporting. The feature introduces a new **Supervisor role** that mirrors the Admin role but cannot manage bank accounts or add Admin users, and can only add Supervisor or Sales Rep users.

Key business rules: three-tier settings granularity (account-level, pricebook-level, multi-account-level); email conflict resolution when adding users whose emails already exist as standalone merchant logins (deactivation prompt for child merchants, hard error for non-children); Data Sharing Agreement required when merging accounts with different owners; password reset via forgot-password flow (no Okta). The dashboard is always scoped to the currently selected account — only the aggregated reporting view crosses account boundaries, with an added Account column and filter. Finance Rep and Group Admin roles (which exist at merchant and Parent Portal levels respectively) are explicitly absent from Multi-Account.

**Status as of 2026-07-30**: the epic has moved decisively out of pre-implementation. 34 child tickets (up from 6 at last refresh), 22 unique PRs across 11 repos, and **65 E2E tests written across 9 test classes**. Core BE (onboarding, email takeover, add-merchant, invitation flow, ABP search) is merged or in code review; FE is largely In Validation on three long-lived integration PRs. The dominant remaining risk is the **Supervisor role (HI-7161/7162/7163/7404), which has zero E2E coverage** and whose BE PR is still a draft, plus **DSA enforcement (HI-7167/7168), which has neither implementation nor tests**.

## Ticket Map

| Ticket | Summary | Status | Assignee | UT | IT | E2E | PRs |
|---|---|---|---|---|---|---|---|
| HI-6726 | [BE] Tech Design | Closed | Devin Lafrenière | N/A | N/A | N/A | none |
| HI-6975 | [FE][VQ] Multi-Account profiles and search | In Validation | Ruan Mer | N/A | N/A | IN DEV | abp-ui#3793 |
| HI-6998 | [BE] Support Multi-Account / Parent types (user/pass vs SSO) | In Validation | Devin Lafrenière | Done | Done | IN DEV | hims#5529, spicedb#1648 |
| HI-6999 | [BE] Multi-Account employee invitation flow | Closed | Devin Lafrenière | Partial | Partial | IN DEV | auth-ui#492, apsrvc#3226, emailmgt#4635, hims#5720, login-srvc#6429, spicedb#1568/#1648/#1661 |
| HI-7000 | [V2][BE] Allow parent accounts to support user management | Open | Devin Lafrenière | GAP | GAP | GAP | none |
| HI-7135 | Test Coverage Checklist (QA task) | In Development | Yogesh Chauhan | N/A | N/A | IN DEV | qa-auto#36790 |
| HI-7141 | [FE][VQ] Child Merchant Profile | In Validation | Ruan Mer | N/A | N/A | IN DEV | abp-ui#3793 |
| HI-7142 | [FE][VQ] Multi-Account Application | In Validation | Ruan Mer | N/A | N/A | IN DEV | abp-ui#3793 |
| HI-7143 | [P2][FE][VQ] Merchant Underwriting | Blocked | Ruan Mer | N/A | N/A | GAP | abp-ui#3793 |
| HI-7144 | [FE][MPD] Home/Dashboard | In Validation | Ruan Mer | N/A | N/A | IN DEV | mpd-ui#182 |
| HI-7161 | [P1.5][BE] Supervisor role — definition, permissions & API enforcement | In Development | Devin Lafrenière | Done | GAP | **GAP** | hims#5844 (DRAFT), spicedb#1730 (DRAFT) |
| HI-7162 | [P1.5][FE][MPD] Supervisor — restricted nav, hidden bank UI, scoped invite picker | In Validation | Ruan Mer | N/A | N/A | **GAP** | mpd-ui#182, mpd-ui#184 |
| HI-7163 | [P1.5][FE][VQ] Supervisor visibility in Merchant Profile & employee assignment | In Validation | Ruan Mer | N/A | N/A | **GAP** | abp-ui#3793 |
| HI-7164 | [BE] Multi-Account onboarding, first admin email takeover | Ready for CodeReview | Devin Lafrenière | Done | Done | IN DEV | hims#5529 |
| HI-7165 | [P2][FE][MPD] Add Account flow — redirect to Merchant Application w/ sponsor_id | Blocked | Ruan Mer | N/A | N/A | IN DEV | none |
| HI-7166 | [FE][MPD] Email-conflict modal — deactivate child login & reassign | In Validation | Ruan Mer | N/A | N/A | IN DEV | none linked |
| HI-7167 | [BE] DSA enforcement gate on cross-owner account merge | Open | Devin Lafrenière | GAP | GAP | **GAP** | none |
| HI-7168 | [FE][VQ] DSA prompt during cross-owner merge | In Validation | Ruan Mer | N/A | N/A | **GAP** | none linked |
| HI-7169 | [TO BE REUSED] (placeholder) | Open | Devin Lafrenière | N/A | N/A | N/A | none |
| HI-7170 | [P2][BE] Migrate Merchant to Multi-Account | Open | Devin Lafrenière | GAP | GAP | GAP | none |
| HI-7171 | [BE] Aggregated Loans & Transactions reporting across accounts | Ready for CodeReview | Devin Lafrenière | Done | GAP | Partial | himr#2493, offline-report-client#2572, upflow2-dags#508 |
| HI-7172 | [FE][MPD] Aggregated reporting UI — account column + filter, admin-only | In Validation | Ruan Mer | N/A | N/A | Partial | mpd-ui#182 |
| HI-7173 | [BE] Add merchant to Multi-Account | Ready for CodeReview | Devin Lafrenière | Done | Done | IN DEV | hims#5529, hims#5763, spicedb#1753 |
| HI-7174 | [BE] ABP Search for Multi-Account | Closed | Devin Lafrenière | GAP | GAP | IN DEV | none linked |
| HI-7175 | [BE] Merchant Email Takeover flow for Multi-Account (Employee Invite) | Ready for CodeReview | Devin Lafrenière | Done | Done | IN DEV | hims#5529, hims#5736 |
| HI-7217 | UAT - Multi Account Hierarchy | Open | Ryan Jung | N/A | N/A | N/A | none |
| HI-7218 | Deployment plan and signoff | Open | Yogesh Chauhan | N/A | N/A | N/A | none |
| HI-7323 | [FE][MPD] Users | In Validation | Ruan Mer | N/A | N/A | IN DEV | mpd-ui#182 |
| HI-7404 | [P1.5][FE][MD] Supervisor — restricted nav, hidden bank UI, scoped invite picker | In Validation | Ruan Mer | N/A | N/A | **GAP** | md-ui#819 |
| HI-7487 | [P2][BE] Multi-Account Merchant Underwriting | Open | unassigned | GAP | GAP | GAP | none |
| HI-7560 | [FE] Cleanups (post-deployment) | Open | Ruan Mer | N/A | N/A | N/A | none |
| HI-7584 | [P2][FE][MPD] Requested Documents | Blocked | Ruan Mer | N/A | N/A | GAP | none |
| HI-7710 | [FE][MPD] Profile | In Validation | Ruan Mer | N/A | N/A | IN DEV | mpd-ui#182 |
| HI-7722 | [FE][MPD] Redirect Merchant users logging into Parent Portal | In Validation | Ruan Mer | N/A | N/A | IN DEV | mpd-ui#210 |

**PR Classification Summary:** 22 unique PRs (34 ticket-PR links). 9 service PRs (UT/IT checked), 6 UI PRs (N/A), 6 infra/schema PRs (N/A — spicedb-schemas ×5, upflow2-dags ×1), 1 client PR (N/A). 9 + 6 + 6 + 1 = 22 ✓

### Service PR UT/IT detail

| PR | Ticket | State | UT | IT | Evidence |
|---|---|---|---|---|---|
| home-improvement-merchant-srvc#5529 | 6998/7164/7173/7175 | OPEN | Done | Done | 71 files; 4 IT (OwnerRepositoryIT, HoldingCompanyEmployeeMutationIT, ParentMutationResolverIT, ParentQueryResolverIT) + 12 UT incl. HoldingCompanyTypeGuardTest, MerchantEmployeeServiceTest |
| home-improvement-merchant-srvc#5736 | 7175 | MERGED | Done | Done | 3 IT + 6 UT (EmployeeFacadeTest, HoldingCompanyEmployeeServiceTest, MerchantOnboardingServiceTest, OwnerServiceTest) |
| home-improvement-merchant-srvc#5763 | 7173 | MERGED | Done | Done | 2 IT + 4 UT |
| home-improvement-merchant-srvc#5720 | 6999 | MERGED | Done | N/A | 2 files; HoldingCompanyEmployeeServiceTest (email-param change, IT not warranted) |
| home-improvement-merchant-srvc#5844 | 7161 | **DRAFT** | Done | GAP | 8 UT incl. BackfillMerchantUserManagerSupervisorRelationshipTaskletTest, EmployeeValidatorTest; **no IT for Supervisor API enforcement** |
| authentication-provider-srvc#3226 | 6999 | MERGED | Done | Done | FactorRegistrationControllerIT + PasswordRegistrationServiceTest |
| login-srvc#6429 | 6999 | MERGED | Done | GAP | LoginPromotionServiceTest only |
| hi-merchant-reporting-srvc#2493 | 7171 | OPEN | Done | GAP | HoldingCompanyResolverTest + ProjectReportServiceTest; no IT |
| email-mgt-srvc#4635 | 6999 | MERGED | **GAP** | **GAP** | NO TESTS — Liquibase changelog only (HI-4241 merchant-activity-to-holding-company notification) |

## E2E Coverage

**All 65 multi-account E2E tests live exclusively on open PR [qa-automation#36790](https://github.com/Credify/qa-automation/pull/36790) (branch `HI-MultiAccountAuth`). The `multiaccount` package does not exist on qa-automation master.** Nothing is merged; a revert or abandonment of that PR loses the entire suite.

| Test class | Tests | AllureId range | Area |
|---|---|---|---|
| HomeImprovementMultiAccountAuthTest | 10 | 74490, 74778, 75871-75878 | Invite link, password onboarding, smart login, token expiry, cross-merchant invite scoping |
| HomeImprovementMultiAccountDashboardTest | 15 | 74542-74546, 74669, 75087, 75088, 75299, 75556, 75868-75870, 75879, 75880 | Account switcher, scoped projects, role-based dashboard access, API permission parity |
| HomeImprovementMultiAccountEmployeeInviteTest | 10 | 75201, 75881-75885, 76059, 76062-76064 | Invite role validation, email-collision rejection, resend semantics |
| HomeImprovementMultiAccountMyProfileTest | 3 | 76072, 76073, 76127 | Self-service name/email update, report-viewer restriction |
| HomeImprovementMultiAccountNewAccountOnboardingTest | 1 | 75886 | New EIN → Merchant Application w/ sponsor_id |
| HomeImprovementMultiAccountOnboardingEmailTakeoverTest | 8 | 74883, 74884, 74890, 74891, 74894, 74895, 74938, 76055 | API-level first-admin email takeover matrix |
| HomeImprovementMultiAccountReportingTest | 2 | 76074, 76075 | Loans tab, Loans CSV report type |
| HomeImprovementMultiAccountUserManagementTest | 4 | 75124, 75167, 75205, 75206 | Email-conflict modal, deactivate + reassign, already-used alerts |
| HomeImprovementMultiAccountVqProfileTest | 12 | 74780, 74784-74788, 74879, 74896, 74934, 74940, 76114, 76115 | VQ wizard onboarding, takeover prompt, search, relationship editor, ID verification |

## Coverage Matrix

| Requirement Area | UT | IT | E2E | Notes |
|---|---|---|---|---|
| Multi-Account entity creation (VQ) | Done | Done | IN DEV | hims#5529; E2E 74934, 74879 |
| Multi-Account / Parent type distinction (user/pass vs SSO) | Done | Done | IN DEV | HI-6998; E2E 74891 asserts PARENT does not migrate |
| Multi-Account admin login & password onboarding | Done | Done | IN DEV | HI-6999 merged; E2E 74490, 75871-75873 |
| Password reset via forgot-password | N/A | N/A | IN DEV | E2E 74778 |
| **Supervisor role CRUD & permissions** | Done | **GAP** | **GAP** | HI-7161 BE still DRAFT; no IT, no E2E; FE flag-gated `REACT_APP_HI_MULTI_ACCOUNT_SUPERVISOR_ENABLED` |
| Role set validation — no Group Admin | Done | Done | IN DEV | E2E 75870, 75882 |
| Role set validation — no Finance Rep | Done | Done | **GAP** | FINANCE_REPRESENTATIVE only appears as a *merchant* role in takeover data; no test asserts it cannot be created in a Multi-Account |
| Email conflict — first admin takeover (VQ/API) | Done | Done | IN DEV | Best-covered area: 10 E2E (74879, 74884, 74938, 74940, 74896, 74890, 74891, 74894, 74895, 76055) |
| Email conflict — self-serve user management | Done | Done | IN DEV | E2E 75124, 75167, 75205, 75206 |
| Multi-Account employee invitation flow | Partial | Partial | IN DEV | email-mgt-srvc#4635 has no tests; E2E 75881-75885, 76059-76064 |
| User management (email uniqueness check) | Done | Done | IN DEV | E2E 75201, 75884, 75885 |
| New account onboarding (new EIN, sponsor_id) | GAP | GAP | IN DEV | HI-7165 Blocked (P2); E2E 75886 exists but targets a flow still blocked |
| Aggregated **Loans** reporting w/ account column | Done | GAP | IN DEV | E2E 76074, 76075 |
| Aggregated **Transactions** reporting | Done | GAP | **GAP** | Spec calls for "Loans *and Transactions* Reports"; zero tests reference transactions |
| Reporting admin-only access | N/A | N/A | Partial | 75869/75880 cover sales-rep/report-viewer nav; no explicit cross-account aggregation-scope assertion |
| Account-level settings isolation | GAP | GAP | **GAP** | 16 settings enumerated in spec (Merchant Preset, RTP, Skip Payment Approval, Tacit Payment Approval, CU flows, KBYG, minFICO/maxHIRM, ...); no test touches any |
| Pricebook isolation (stays at account level on merge) | GAP | GAP | **GAP** | No test asserts a merged account's pricebook stays account-scoped |
| Data Sharing Agreement on cross-owner merge | GAP | GAP | **GAP** | HI-7167 Open, no PR; E2E 74784/74780 cover the *Jira link* gate, which is not the DSA |
| VQ agent edits account/pricebook/multi-account settings | GAP | GAP | **GAP** | Spec requirement; no ticket, no test |
| VQ agent sees plans + employees per account | GAP | GAP | **GAP** | app-by-phone-ui does not yet render the combined list (noted in 74787 Javadoc) |
| Cross-account data isolation (sales rep) | Done | Done | IN DEV | E2E 74545, 74546, 75869 |
| Merchant→Parent-Portal redirect | N/A | N/A | IN DEV | mpd-ui#210; E2E via incorrect-access assertions in 74879 |
| Document Center (multi-account) | GAP | GAP | GAP | HI-7584 Blocked (P2) |
| Notifications Management (multi-account) | GAP | GAP | GAP | P2, no ticket |
| WIP cap (account + multi-account) | GAP | GAP | GAP | P2, no ticket |
| Multi-Account merchant underwriting | GAP | GAP | GAP | HI-7143 Blocked, HI-7487 Open (both P2) |

## Active Gaps

### Critical — P0 Boundaries (must pass before UAT sign-off)

- [HIGH] **Supervisor cannot change bank accounts** — HI-7161. BE PR `hims#5844` is still a DRAFT with **UT only, no IT**. Zero E2E. This is the single hardest permission boundary in the feature and currently has no executable proof at any layer above unit.
- [HIGH] **Supervisor cannot add Admin users** — HI-7161/7162. Role-creation gate must reject at API level, not just hide UI. No IT, no E2E.
- [HIGH] **Supervisor positive capabilities** — spec grants start loan apps, request payments, request refunds, view Merchant + Multi-Account reporting. None verified.
- [HIGH] **Data Sharing Agreement gate on cross-owner merge** — HI-7167 is Open with **no PR at all**, though the FE ticket HI-7168 is In Validation. Legal/compliance requirement: data must not be shared before DSA is signed. The existing Jira-link gate (74784/74780) is an internal approval control, *not* the DSA — do not mistake one for the other.

### High — Core feature gaps

- [HIGH] **Entire E2E suite is unmerged** — all 65 tests sit on qa-automation#36790. Nothing gates a regression on master today.
- [HIGH] **Aggregated Transactions report untested** — HI-7171 ships Loans and Transactions; only Loans has E2E (76074/76075) and the service PR has **no IT**.
- [HIGH] **Account-level settings isolation completely untested** — the spec's second table enumerates 16 settings that must stay account-scoped. Zero coverage at any layer. A leak here silently mis-prices loans across a merchant's accounts.
- [MEDIUM] **Finance Rep exclusion not asserted** — spec is explicit that Finance Reps do not exist for multi-accounts; no negative test.

### Medium — Implementation gaps

- [MEDIUM] **email-mgt-srvc#4635 has zero tests** (Liquibase changelog only) — merged as part of HI-6999.
- [MEDIUM] **hi-merchant-reporting-srvc#2493 has no IT** — resolver + service UT only for a cross-account aggregation query, which is exactly where an IT would catch a scoping bug.
- [MEDIUM] **login-srvc#6429 has no IT** for login promotion.
- [MEDIUM] **HI-7174 (ABP Search) is Closed with no linked PR** and GAP UT/IT — implementation is presumably folded into hims#5529 but is not traceable from the ticket.
- [MEDIUM] **Pricebook-stays-at-account-level on merge** — explicit spec rule, no test.

### Spec Requirement Gaps (no ticket mapping)

- [MEDIUM] VQ agents editing pricebook-level / account-level / multi-account-level settings — spec requirement under "Servicing / Underwriting"; no ticket exists.
- [MEDIUM] VQ agents seeing plans opted-in per account, and which employees are assigned to each account — no ticket; app-by-phone-ui does not render the combined view yet (documented in the 74787 Javadoc).
- [LOW] Reporting must **not** expose rep directory or onboarding reports (Parent Portal-only) — negative requirement, no test.
- [LOW] Multi-Account → Parent Portal promotion path (owner match or DSA) — future-state, no ticket.

## PR Analysis

*(append-only)*

### 2026-07-30 — bulk classification of 22 PRs

First refresh with PR data (previous refresh had no Jira token). Full service-PR test-file audit recorded in the "Service PR UT/IT detail" table above. Notable structural observation: `home-improvement-merchant-srvc#5529` is a long-lived open PR carrying **four tickets** (HI-6998, HI-7173, HI-7175, HI-7164) and 71 files, while narrower slices of the same work (#5736 HI-7175, #5763 HI-7173) have already merged separately. Any environment not running #5529 will exhibit pre-HI-7164 behaviour.

Likewise `app-by-phone-ui#3793` carries five FE tickets and `merchant-parent-dashboard-ui#182` carries five — ticket-level status ("In Validation") therefore does not imply independently shippable increments.

## Key Decisions

- Multi-Account uses **password login**, not Okta SSO — architecturally distinct from Parent Portal (HI-6998)
- **Finance Rep role is explicitly absent** from Multi-Account per spec; roles are Admin, Supervisor (new), Sales Rep
- **Group Admin** (Parent Portal) must NOT appear in Multi-Account user management
- **Pricebooks remain at account level** even after merge
- **Data Sharing Agreement required** before access is granted when merging accounts with different owners
- Dashboard is **always account-scoped**; aggregated view only in reporting, Admins only
- **HI-7164 (2026-07-30)**: taking over the email of a merchant's *sole active administrator* no longer deletes that employee. It is retained and its **login username** is moved to a plus-alias carrying the merchant account id (`info@x.com` → `info+123@x.com`), freeing the original address. The actor's preferred email is untouched. Taking over any other employee still deletes it. This reversed two previously-disabled "known issue" tests (74938, 74940) into positive coverage.
- **Supervisor is FE-flag-gated** behind `REACT_APP_HI_MULTI_ACCOUNT_SUPERVISOR_ENABLED`; E2E role assertions deliberately exclude it today (see HomeImprovementMultiAccountDashboardTest:652).

## Notes for SDET

- E2E branch: `HI-MultiAccountAuth` → [qa-automation#36790](https://github.com/Credify/qa-automation/pull/36790). Per the E2E workflow, keep all HI-5039 test work on this branch — do not open a second branch.
- Tests asserting HI-7164 alias behaviour (74879, 74938, 74940, 75879, 75299) require an environment running `home-improvement-merchant-srvc#5529`. Against plain master they fail on the old delete-branch behaviour, which is an environment problem, not a test defect.
- Existing tests to extend/reuse: AllureId 2827 (Sales Rep scoping), 1370 (password reset), 59027 (pricebook Admin access), 13939/13940 (WIP cap)
- Highest-value next tests, in order: (1) Supervisor bank-account denial at API level, (2) Supervisor cannot invite an ADMINISTRATOR, (3) DSA gate on cross-owner merge, (4) aggregated Transactions report, (5) one account-level settings isolation test as a template for the other 15.
