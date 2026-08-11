---
epic: HI-5039
title: HI Merchant Multi-Account
spec_url: https://credify.atlassian.net/wiki/spaces/PROD/pages/4365615309/HI+Merchant+Multi-Account
status: in-development
last_refreshed: 2026-08-11
test_checklist_ticket: HI-7135
confluence_page_id: "5683675314"
---

# HI-5039 — HI Merchant Multi-Account

## Spec Summary

The HI Merchant Multi-Account feature enables mid-sized merchants to operate multiple accounts under a single password-based login, without requiring SSO. This is explicitly distinct from the Parent Portal (which uses Okta SSO, merchant groups, and IT-level integrations). Each account under a multi-account maintains independent bank accounts, plan selections (pricebooks), locations, settings, employees, and reporting. The feature introduces a new **Supervisor role** that mirrors the Admin role but cannot manage bank accounts or add Admin users, and can only add Supervisor or Sales Rep users.

Key business rules: three-tier settings granularity (account-level, pricebook-level, multi-account-level); email conflict resolution when adding users whose emails already exist as standalone merchant logins (deactivation prompt for child merchants, hard error for non-children); Data Sharing Agreement required when merging accounts with different owners; password reset via forgot-password flow (no Okta). The dashboard is always scoped to the currently selected account — only the aggregated reporting view crosses account boundaries, with an added Account column and filter. Finance Rep and Group Admin roles (which exist at merchant and Parent Portal levels respectively) are explicitly absent from Multi-Account. Spec page re-read at version 41 (last edited 2026-08-07) — no substantive new requirements found versus the 2026-07-30 refresh; content matches the tables and rules already tracked below.

**Status as of 2026-08-11**: this is a major-progress refresh. **qa-automation#36790 (HI-7135, 71 E2E tests, 9 classes) merged to master on 2026-08-11 (`c899cc7880`)** — the entire E2E suite that was previously "unmerged, at risk of loss" is now permanently on master and gates regressions. All three long-lived FE integration PRs (`app-by-phone-ui#3793`, `merchant-parent-dashboard-ui#182`, `merchant-dashboard-ui#819`) also merged, as did the two previously-open service PRs (`home-improvement-merchant-srvc#5529`, `hi-merchant-reporting-srvc#2493`). 36 child tickets now exist (up from 34; two new Closed bugs, HI-7773/HI-7791). The epic is functionally complete for every requirement **except the two P0 boundaries flagged since the last refresh**: the **Supervisor role** (BE PR `hims#5844` and `spicedb-schemas#1730` are still DRAFT; a related FE integration PR `merchant-dashboard-ui#838` is open and explicitly blocked on the BE draft; zero E2E coverage) and **DSA enforcement on cross-owner merge** (HI-7167 still Open with **no implementation PR anywhere**). Both remain fully untested at every layer above unit.

## Ticket Map

| Ticket | Summary | Status | Assignee | UT | IT | E2E | PRs |
|---|---|---|---|---|---|---|---|
| HI-6726 | [BE] Tech Design | Closed | Devin Lafrenière | N/A | N/A | N/A | none |
| HI-6975 | [FE][VQ] Multi-Account profiles and search | Closed | Ruan Mer | N/A | N/A | COVERED | abp-ui#3793 |
| HI-6998 | [BE] Support Multi-Account / Parent types (user/pass vs SSO) | Closed | Devin Lafrenière | Done | Done | COVERED | hims#5529, spicedb#1648 |
| HI-6999 | [BE] Multi-Account employee invitation flow | Closed | Devin Lafrenière | Partial | Partial | COVERED | auth-ui#492, apsrvc#3226, emailmgt#4635, hims#5720, login-srvc#6429, spicedb#1568/#1648/#1661 |
| HI-7000 | [V2][BE] Allow parent accounts to support user management | Open | Devin Lafrenière | GAP | GAP | GAP | none |
| HI-7135 | Test Coverage Checklist (QA task) | In Development | Yogesh Chauhan | N/A | N/A | COVERED | qa-auto#36790 (MERGED) |
| HI-7141 | [FE][VQ] Child Merchant Profile | Closed | Ruan Mer | N/A | N/A | COVERED | abp-ui#3793 |
| HI-7142 | [FE][VQ] Multi-Account Application | Closed | Ruan Mer | N/A | N/A | COVERED | abp-ui#3793 |
| HI-7143 | [P2][FE][VQ] Merchant Underwriting | Blocked | Ruan Mer | N/A | N/A | GAP | abp-ui#3793 |
| HI-7144 | [FE][MPD] Home/Dashboard | Resolved | Ruan Mer | N/A | N/A | COVERED | mpd-ui#182 |
| HI-7161 | [P1.5][BE] Supervisor role — definition, permissions & API enforcement | Ready for CodeReview | Devin Lafrenière | Done | GAP | **GAP** | hims#5844 (DRAFT, updated today), spicedb#1730 (DRAFT) |
| HI-7162 | [P1.5][FE][MPD] Supervisor — restricted nav, hidden bank UI, scoped invite picker | In Validation | Ruan Mer | N/A | N/A | **GAP** | mpd-ui#182, mpd-ui#184 (MERGED), mpd-ui#198 (MERGED, flag gate) |
| HI-7163 | [P1.5][FE][VQ] Supervisor visibility in Merchant Profile & employee assignment | In Validation | Ruan Mer | N/A | N/A | **GAP** | abp-ui#3793, abp-ui#3847 (role-constant refactor only — visibility feature itself not yet found in diff) |
| HI-7164 | [BE] Multi-Account onboarding, first admin email takeover | Closed | Devin Lafrenière | Done | Done | COVERED | hims#5529 (MERGED) |
| HI-7165 | [P2][FE][MPD] Add Account flow — redirect to Merchant Application w/ sponsor_id | Blocked | Ruan Mer | N/A | N/A | COVERED | none |
| HI-7166 | [FE][MPD] Email-conflict modal — deactivate child login & reassign | Resolved | Ruan Mer | N/A | N/A | COVERED | none linked (still untraceable — presumed folded into mpd-ui#182) |
| HI-7167 | [BE] DSA enforcement gate on cross-owner account merge | Open | Devin Lafrenière | GAP | GAP | **GAP** | none — unchanged since last refresh |
| HI-7168 | [FE][VQ] DSA prompt during cross-owner merge | Closed | Ruan Mer | N/A | N/A | **GAP** | none linked (still untraceable) |
| HI-7169 | [TO BE REUSED] (placeholder) | Open | Devin Lafrenière | N/A | N/A | N/A | none |
| HI-7170 | [P2][BE] Migrate Merchant to Multi-Account | Open | Devin Lafrenière | GAP | GAP | GAP | none |
| HI-7171 | [BE] Aggregated Loans & Transactions reporting across accounts | Closed | Devin Lafrenière | Done | GAP | Partial | himr#2493 (MERGED), offline-report-client#2572, upflow2-dags#508, k8s-template#274765 (deploy) |
| HI-7172 | [FE][MPD] Aggregated reporting UI — account column + filter, admin-only | Resolved | Ruan Mer | N/A | N/A | Partial | mpd-ui#182 |
| HI-7173 | [BE] Add merchant to Multi-Account | Closed | Devin Lafrenière | Done | Done | COVERED | hims#5529, hims#5763, spicedb#1753 |
| HI-7174 | [BE] ABP Search for Multi-Account | Closed | Devin Lafrenière | GAP | GAP | COVERED | none linked (still untraceable, presumed folded into hims#5529) |
| HI-7175 | [BE] Merchant Email Takeover flow for Multi-Account (Employee Invite) | Closed | Devin Lafrenière | Done | Done | COVERED | hims#5529, hims#5736 |
| HI-7217 | UAT - Multi Account Hierarchy | Open | Ryan Jung | N/A | N/A | N/A | none |
| HI-7218 | Deployment plan and signoff | Open | Yogesh Chauhan | N/A | N/A | N/A | none |
| HI-7323 | [FE][MPD] Users | Resolved | Ruan Mer | N/A | N/A | COVERED | mpd-ui#182 |
| HI-7404 | [P1.5][FE][MD] Supervisor — restricted nav, hidden bank UI, scoped invite picker | In Validation | Ruan Mer | N/A | N/A | **GAP** | md-ui#819 (MERGED, no supervisor code), md-ui#837 (MERGED, HI-7648 sub-task), **md-ui#838 (OPEN, depends on hims#5844)** |
| HI-7487 | [P2][BE] Multi-Account Merchant Underwriting | Open | unassigned | GAP | GAP | GAP | none |
| HI-7560 | [FE] Cleanups (post-deployment) | Open | Ruan Mer | N/A | N/A | N/A | none |
| HI-7584 | [P2][FE][MPD] Requested Documents | Blocked | Ruan Mer | N/A | N/A | GAP | none |
| HI-7710 | [FE][MPD] Profile | Resolved | Ruan Mer | N/A | N/A | COVERED | mpd-ui#182 |
| HI-7722 | [FE][MPD] Redirect Merchant users logging into Parent Portal | Resolved | Ruan Mer | N/A | N/A | COVERED | mpd-ui#210 |
| HI-7773 *(new)* | [FE][MPD] Bug — stale bootstrap state after multi-account login | Closed | Ruan Mer | N/A | N/A | N/A | none found (searched mpd-ui/abp-ui commits and PR titles — untraceable) |
| HI-7791 *(new)* | [FE][VQ] Bug — handle bad-data edge cases in multi-account hierarchy UI | Closed | Ruan Mer | N/A | N/A | N/A | abp-ui#3929 (MERGED, jest UT only) |

**PR Classification Summary:** 25 unique PRs (up from 22). 9 service PRs (UT/IT checked — unchanged set), 8 UI PRs (N/A — added mpd-ui#184, mpd-ui#198, md-ui#837, md-ui#838, abp-ui#3847, abp-ui#3929), 7 infra/schema PRs (N/A — added spicedb-schemas#1661, k8s-template#274765), 1 client PR (N/A). 9 + 8 + 7 + 1 = 25 ✓. **None of the 4 net-new PRs are service repos**, so no new UT/IT checks were required by the classification gate; all new PRs are UI/infra.

### Service PR UT/IT detail

| PR | Ticket | State | UT | IT | Evidence |
|---|---|---|---|---|---|
| home-improvement-merchant-srvc#5529 | 6998/7164/7173/7175 | **MERGED** (was OPEN) | Done | Done | 73 files; IT: OwnerRepositoryIT, HoldingCompanyEmployeeMutationIT, ParentMutationResolverIT, ParentQueryResolverIT; UT: HoldingCompanyTypeGuardTest, MerchantEmployeeServiceTest, EmployeeFacadeTest, LoginServiceTest, HoldingCompanyFacadeTest, HoldingCompanyGroupServiceTest, HoldingCompanyServiceTest, MerchantOnboardingServiceTest, OwnerServiceTest, AccessRelationshipUpdaterTest, EmployeeTokenServiceTest, GraphQLEnumConsistencyTest |
| home-improvement-merchant-srvc#5736 | 7175 | MERGED | Done | Done | unchanged from last refresh — 3 IT + 6 UT |
| home-improvement-merchant-srvc#5763 | 7173 | MERGED | Done | Done | unchanged — 2 IT + 4 UT |
| home-improvement-merchant-srvc#5720 | 6999 | MERGED | Done | N/A | unchanged — HoldingCompanyEmployeeServiceTest, IT not warranted |
| home-improvement-merchant-srvc#5844 | 7161 | **DRAFT** (31 files, updated 2026-08-11 — actively worked today) | Done | **GAP** | 9 test files now incl. new `BackfillMerchantUserManagerSupervisorRelationshipTaskletTest`, `EmployeeValidatorTest`, `ProjectBorrowerSearchServiceTest`, `MerchantUserManagedRefundsFacadeTest`, `MerchantAccessRelationshipUpdaterTest`. The two IT files touched (`ParentMutationResolverIT`, `ParentQueryResolverIT`) show only 8/19-line diffs with **zero Supervisor-related assertions** in the patch — confirmed no IT exists yet for Supervisor API permission enforcement (bank-account denial, admin-invite denial). |
| authentication-provider-srvc#3226 | 6999 | MERGED | Done | Done | unchanged |
| login-srvc#6429 | 6999 | MERGED | Done | GAP | unchanged — LoginPromotionServiceTest only, no IT |
| hi-merchant-reporting-srvc#2493 | 7171 | **MERGED** (was OPEN) | Done | GAP | unchanged test files (HoldingCompanyResolverTest, ProjectReportServiceTest); still no IT |
| email-mgt-srvc#4635 | 6999 | MERGED | **GAP** | **GAP** | unchanged — NO TESTS, Liquibase changelog only |

**Supervisor role — cross-surface implementation status (new this refresh, discovered via broader PR search since Jira's dev-panel only tracks direct parent links, not sub-tasks):**

| Surface | Repo | State | Notes |
|---|---|---|---|
| BE permissions | home-improvement-merchant-srvc#5844 | DRAFT | UT only, no IT. Actively updated today (2026-08-11). |
| BE SpiceDB schema | spicedb-schemas#1730 | DRAFT | Unchanged since last refresh. |
| FE — Merchant Parent Dashboard (MPD) | merchant-parent-dashboard-ui#184, #198 | **MERGED** | Full Supervisor role + flag-gating merged to master. Jest UT: `SupervisorMerchantList.test.js`, `EmployeeRoleSelect.test.js`, `usePermissions.test.jsx`. This surface is FE-complete. |
| FE — Merchant Dashboard (MD) | merchant-dashboard-ui#837 (sub-task HI-7648, MERGED) + **#838 (OPEN)** | **Blocked** | #838 is titled "HI-5039 Multi-Account implementation" and its PR body explicitly states `Depends on: home-improvement-merchant-srvc/pull/5844`. Identical file set to #837 (UserEdit.js/.test.js, EmployeeRoleSelect.js, usePermissions.js, employeeRoles.js + jest tests) — this is the integration-to-master PR still waiting on the BE draft. |
| FE — VQ (app-by-phone-ui) | abp-ui#3793, #3847 | **Not actually implemented** | #3847 (sub-tasks HI-7442/HI-7464/HI-7476, all Closed) is a generic Employee Role constants refactor pulled from a shared HICUI lib — no "supervisor" string anywhere in its diff. HI-7163 (Supervisor visibility in Merchant Profile & employee assignment) has no traceable implementation yet despite its sub-tasks being Closed. |

Net effect: Supervisor is **not one uniform gap** — MPD is FE-done, MD is FE-complete-but-unmerged-and-BE-blocked, VQ has no real implementation yet, and BE enforcement itself is still a draft with zero IT. E2E is zero everywhere.

## E2E Coverage

**All 71 multi-account E2E tests (up from 65) are now on `Credify/qa-automation` master**, merged via [qa-automation#36790](https://github.com/Credify/qa-automation/pull/36790) at commit `c899cc7880` on 2026-08-11. The `HI-MultiAccountAuth` branch's risk of loss is fully retired — the suite now gates regressions like any other master test. No open qa-automation PR carries additional multi-account coverage (searched for Supervisor/DSA/HI-7161/HI-7167 by title — none found).

| Test class | Tests | AllureId range | Area |
|---|---|---|---|
| HomeImprovementMultiAccountAuthTest | 10 | 74490, 74778, 75871-75878 | Invite link, password onboarding, smart login, token expiry, cross-merchant invite scoping |
| HomeImprovementMultiAccountDashboardTest | 17 (+2) | 74542-74546, 74669, 75087, 75088, 75299, 75556, 75868-75870, 75879, 75880, 77492, 77495 | Account switcher, scoped projects, role-based dashboard access, API permission parity, general/plan-specific application-link timing vs. email takeover |
| HomeImprovementMultiAccountEmployeeInviteTest | 5 (methods; 1 uses a 2-row DataProvider covering AllureIds 76060/76061) | 75881, 76059-76063 | Invite role validation, email-collision rejection, resend semantics, non-admin-role invite denial |
| HomeImprovementMultiAccountMyProfileTest | 4 (+1) | 76072, 76073, 76127, 76576 | Self-service name/email update, report-viewer restriction, canUpdateOwnDetails guard |
| HomeImprovementMultiAccountNewAccountOnboardingTest | 1 | 75886 | New EIN → Merchant Application w/ sponsor_id |
| HomeImprovementMultiAccountOnboardingEmailTakeoverTest | 9 (+1) | 74883, 74884, 74890, 74891, 74894, 74895, 74938, 76055, 77984 | API-level first-admin email takeover matrix |
| HomeImprovementMultiAccountReportingTest | 6 (+4) | 76074, 76075, 77557, 77559, 78272, 78394 | Loans tab, Loans CSV, CSV cross-project aggregation, email report preferences, Pre-Qualified Leads tab — **still no Transactions report test** |
| HomeImprovementMultiAccountUserManagementTest | 4 | 75124, 75167, 75205, 75206 | Email-conflict modal, deactivate + reassign, already-used alerts |
| HomeImprovementMultiAccountVqProfileTest | 15 (+3) | 74780, 74784-74788, 74879, 74896, 74934, 74940, 76114, 76115, 77467, 77468, 78140 | VQ wizard onboarding, takeover prompt, search, relationship editor, ID verification, zero/real-owner holding-company connection |

**Confirmed still absent from every test file (grepped for "Supervisor" and DSA-related terms):** zero Supervisor-role assertions (one comment at `HomeImprovementMultiAccountDashboardTest.java:649` documents the deliberate exclusion behind the feature flag) and zero Data Sharing Agreement assertions.

## Coverage Matrix

| Requirement Area | UT | IT | E2E | Notes |
|---|---|---|---|---|
| Multi-Account entity creation (VQ) | Done | Done | COVERED | hims#5529 now MERGED; E2E 74934, 74879 |
| Multi-Account / Parent type distinction (user/pass vs SSO) | Done | Done | COVERED | HI-6998 Closed; E2E 74891 asserts PARENT does not migrate |
| Multi-Account admin login & password onboarding | Done | Done | COVERED | HI-6999 Closed; E2E 74490, 75871-75873 |
| Password reset via forgot-password | N/A | N/A | COVERED | E2E 74778 |
| **Supervisor role CRUD & permissions** | Done | **GAP** | **GAP** | BE `hims#5844` still DRAFT (updated today); MPD FE fully merged, MD FE merged-but-unintegrated (blocked on BE), VQ FE not implemented; no IT, no E2E anywhere |
| Role set validation — no Group Admin | Done | Done | COVERED | E2E 75870, 75882 |
| Role set validation — no Finance Rep | Done | Done | **GAP** | Unchanged — no negative test |
| Email conflict — first admin takeover (VQ/API) | Done | Done | COVERED | Best-covered area, unchanged: 74879, 74884, 74938, 74940, 74896, 74890, 74891, 74894, 74895, 76055 |
| Email conflict — self-serve user management | Done | Done | COVERED | E2E 75124, 75167, 75205, 75206 |
| Multi-Account employee invitation flow | Partial | Partial | COVERED | email-mgt-srvc#4635 still has no tests; E2E 75881, 76059-76063 |
| User management (email uniqueness check) | Done | Done | COVERED | E2E 75201-equivalent behavior, 75884/75885 folded into current invite tests |
| New account onboarding (new EIN, sponsor_id) | GAP | GAP | COVERED | HI-7165 still Blocked (P2); E2E 75886 targets a flow still blocked upstream |
| Aggregated **Loans** reporting w/ account column | Done | GAP | COVERED | E2E 76074, 76075, 77557, 77559 |
| Aggregated **Transactions** reporting | Done | GAP | **GAP** | Unchanged — spec calls for "Loans *and Transactions* Reports"; still zero tests reference transactions despite Reporting test class growing from 2→6 tests |
| Reporting admin-only access | N/A | N/A | Partial | Unchanged |
| Account-level settings isolation | GAP | GAP | **GAP** | Unchanged — ~15 settings enumerated in spec table 2; zero coverage at any layer |
| Pricebook isolation (stays at account level on merge) | GAP | GAP | **GAP** | Unchanged |
| Data Sharing Agreement on cross-owner merge | GAP | GAP | **GAP** | Unchanged — HI-7167 still Open, still no PR anywhere |
| VQ agent edits account/pricebook/multi-account settings | GAP | GAP | **GAP** | Unchanged — no ticket, no test |
| VQ agent sees plans + employees per account | GAP | GAP | **GAP** | Unchanged |
| Cross-account data isolation (sales rep) | Done | Done | COVERED | E2E 74545, 74546, 75869 |
| Merchant→Parent-Portal redirect | N/A | N/A | COVERED | mpd-ui#210 merged; E2E via incorrect-access assertions |
| Document Center (multi-account) | GAP | GAP | GAP | HI-7584 still Blocked (P2) |
| Notifications Management (multi-account) | GAP | GAP | GAP | P2, no ticket |
| WIP cap (account + multi-account) | GAP | GAP | GAP | P2, no ticket |
| Multi-Account merchant underwriting | GAP | GAP | GAP | HI-7143 Blocked, HI-7487 Open (both P2) |

## Active Gaps

### Critical — P0 Boundaries (must pass before UAT sign-off)

- [HIGH] **Supervisor cannot change bank accounts** — HI-7161. BE PR `hims#5844` is still DRAFT (actively updated 2026-08-11) with **UT only, no IT**. Zero E2E. This remains the single hardest permission boundary in the feature with no executable proof above unit test.
- [HIGH] **Supervisor cannot add Admin users** — HI-7161/7162/7404. Role-creation gate must reject at API level, not just hide UI. No IT, no E2E. Note: MPD's UI-side hiding is done (mpd-ui#184/#198 merged); MD's is coded but sitting in an unmerged, BE-blocked PR (md-ui#838); VQ's isn't implemented at all yet (HI-7163).
- [HIGH] **Supervisor positive capabilities** — spec grants start loan apps, request payments, request refunds, view Merchant + Multi-Account reporting. None verified at any layer.
- [HIGH] **Data Sharing Agreement gate on cross-owner merge** — HI-7167 is still Open with **no PR anywhere** (checked broadly — no implementation PR in any repo references it), even though the FE ticket HI-7168 is now Closed. This is a legal/compliance requirement: data must not be shared before DSA is signed. The Jira-link gate (E2E 74784/74780) is an internal approval control, *not* the DSA — do not conflate them.

### Resolved since 2026-07-30 refresh

- ~~Entire E2E suite is unmerged~~ — **RESOLVED**. qa-automation#36790 merged 2026-08-11 (`c899cc7880`); all 71 tests are on master.
- ~~Three long-lived FE integration PRs stuck In Validation~~ — **RESOLVED**. `abp-ui#3793`, `mpd-ui#182`, `md-ui#819` all merged.
- ~~hims#5529 / himr#2493 open for extended period~~ — **RESOLVED**. Both merged.

### High — Core feature gaps

- [HIGH] **Aggregated Transactions report untested** — HI-7171 ships Loans and Transactions; only Loans has E2E (76074/76075/77557/77559) and the service PR has **no IT**. The Reporting test class grew from 2→6 tests this cycle but none of the additions touch Transactions.
- [HIGH] **Account-level settings isolation completely untested** — the spec's second table enumerates ~15 settings that must stay account-scoped. Zero coverage at any layer. A leak here silently mis-prices loans across a merchant's accounts.
- [MEDIUM] **Finance Rep exclusion not asserted** — spec is explicit that Finance Reps do not exist for multi-accounts; still no negative test.
- [MEDIUM] **HI-7404 (MD Supervisor) blocked on BE** — `merchant-dashboard-ui#838` cannot merge until `hims#5844` merges; if BE timeline slips, this FE work sits idle regardless of its own readiness.
- [MEDIUM] **HI-7163 (VQ Supervisor visibility) has no real implementation** — its Closed sub-tasks (HI-7442/HI-7464/HI-7476) only refactored role constants; the actual Merchant Profile / employee-assignment visibility feature does not appear in any abp-ui diff yet.

### Medium — Implementation gaps

- [MEDIUM] **email-mgt-srvc#4635 has zero tests** (Liquibase changelog only) — unchanged, merged as part of HI-6999.
- [MEDIUM] **hi-merchant-reporting-srvc#2493 has no IT** — now MERGED but still resolver + service UT only; exactly where an IT would catch a cross-account scoping bug.
- [MEDIUM] **login-srvc#6429 has no IT** for login promotion — unchanged.
- [MEDIUM] **HI-7174 (ABP Search) and HI-7168/HI-7166/HI-7773 (bug fix) are Closed with no traceable PR** — implementation presumably folded into larger merged PRs (hims#5529, mpd-ui#182) but not traceable from the ticket itself.
- [MEDIUM] **Pricebook-stays-at-account-level on merge** — explicit spec rule, still no test.

### Spec Requirement Gaps (no ticket mapping)

- [MEDIUM] VQ agents editing pricebook-level / account-level / multi-account-level settings — spec requirement under "Servicing / Underwriting"; still no ticket exists.
- [MEDIUM] VQ agents seeing plans opted-in per account, and which employees are assigned to each account — no ticket; app-by-phone-ui still does not render the combined view (documented in the 74787 test's Javadoc).
- [LOW] Reporting must **not** expose rep directory or onboarding reports (Parent Portal-only) — negative requirement, no test.
- [LOW] Multi-Account → Parent Portal promotion path (owner match or DSA) — future-state, no ticket.

## PR Analysis

*(append-only)*

### 2026-07-30 — bulk classification of 22 PRs

First refresh with PR data (previous refresh had no Jira token). Full service-PR test-file audit recorded in the "Service PR UT/IT detail" table above. Notable structural observation: `home-improvement-merchant-srvc#5529` is a long-lived open PR carrying **four tickets** (HI-6998, HI-7173, HI-7175, HI-7164) and 71 files, while narrower slices of the same work (#5736 HI-7175, #5763 HI-7173) have already merged separately. Any environment not running #5529 will exhibit pre-HI-7164 behaviour.

Likewise `app-by-phone-ui#3793` carries five FE tickets and `merchant-parent-dashboard-ui#182` carries five — ticket-level status ("In Validation") therefore does not imply independently shippable increments.

### 2026-08-11 — refresh: E2E suite merged, FE integration PRs merged, Supervisor cross-surface audit

`qa-automation#36790` merged (`c899cc7880`), taking the E2E suite from "at-risk, unmerged" to "on master, gating regressions" in one step, and growing from 65→71 tests along the way (Dashboard, MyProfile, EmailTakeover, Reporting, and VqProfile each picked up new methods; none touch Supervisor or DSA). The three long-lived FE integration PRs (`abp-ui#3793`, `mpd-ui#182`, `md-ui#819`) and the two previously-open service PRs (`hims#5529`, `himr#2493`) all merged in the same window — this epic went from "FE largely In Validation" to "FE/BE done except Supervisor+DSA" in about two weeks.

The dev-panel query on `parent = HI-5039` undercounts PRs for the Supervisor tickets because Jira's dev panel only surfaces PRs linked to the exact ticket queried, not to that ticket's sub-tasks. A targeted `gh pr list --search supervisor` per repo surfaced sub-task PRs (HI-7648 → `md-ui#837`, HI-7650 → `mpd-ui#198`, HI-7442/7464/7476/7484 → refactor-only PRs) that the standard workflow's dev-info script missed entirely. This revealed that Supervisor implementation status is **not uniform**: MPD is fully merged, MD is coded-but-blocked-on-BE (`md-ui#838` open, explicit dependency on `hims#5844`), and VQ has no real implementation despite its sub-tasks being Closed. Future refreshes of Supervisor-adjacent epics should query sub-tasks (`parent in (<story-keys>)`) in addition to the epic-level query, and should search each FE repo directly for the feature keyword rather than relying solely on the dev-info script.

## Key Decisions

- Multi-Account uses **password login**, not Okta SSO — architecturally distinct from Parent Portal (HI-6998)
- **Finance Rep role is explicitly absent** from Multi-Account per spec; roles are Admin, Supervisor (new), Sales Rep
- **Group Admin** (Parent Portal) must NOT appear in Multi-Account user management
- **Pricebooks remain at account level** even after merge
- **Data Sharing Agreement required** before access is granted when merging accounts with different owners
- Dashboard is **always account-scoped**; aggregated view only in reporting, Admins only
- **HI-7164 (2026-07-30)**: taking over the email of a merchant's *sole active administrator* no longer deletes that employee. It is retained and its **login username** is moved to a plus-alias carrying the merchant account id (`info@x.com` → `info+123@x.com`), freeing the original address. The actor's preferred email is untouched. Taking over any other employee still deletes it. This reversed two previously-disabled "known issue" tests (74938, 74940) into positive coverage.
- **Supervisor is FE-flag-gated** behind `REACT_APP_HI_MULTI_ACCOUNT_SUPERVISOR_ENABLED`; E2E role assertions deliberately exclude it today (see HomeImprovementMultiAccountDashboardTest.java:649).
- **Supervisor rollout is per-surface, not monolithic (2026-08-11)**: MPD (merchant-parent-dashboard-ui) shipped Supervisor to master already; MD (merchant-dashboard-ui) has the FE code ready in an open PR explicitly gated on the BE draft merging first; VQ (app-by-phone-ui) has not implemented the feature yet despite ticket sub-tasks showing Closed (those sub-tasks were role-constant refactors, not the visibility feature).

## Notes for SDET

- The E2E suite is on **master** now — no more working off `HI-MultiAccountAuth`/qa-automation#36790. Any new HI-5039 test work (Supervisor, DSA) should be a **fresh branch off current master**, per the standard E2E workflow (no existing open qa-automation PR to attach to).
- Tests asserting HI-7164 alias behaviour (74879, 74938, 74940, 75879, 75299) no longer have an environment caveat — the dependency (`hims#5529`) is merged to master.
- Existing tests to extend/reuse: AllureId 2827 (Sales Rep scoping), 1370 (password reset), 59027 (pricebook Admin access), 13939/13940 (WIP cap)
- The current qa-automation branch `HI-SupervisorRole` (checked 2026-08-11) has no commits yet relative to master — it's a clean starting point for exactly this work.
- Highest-value next tests, in order: (1) Supervisor bank-account denial at API level — but this cannot be written as a real (non-mocked) E2E test until `hims#5844` merges; consider starting with a test skeleton against the draft PR's ondemand deploy, (2) Supervisor cannot invite an ADMINISTRATOR, (3) DSA gate on cross-owner merge — also blocked on a BE implementation that doesn't exist yet (HI-7167 has no PR at all), (4) aggregated Transactions report (this one IS unblocked — himr#2493 is merged, just needs an IT + E2E), (5) one account-level settings isolation test as a template for the other ~14.
