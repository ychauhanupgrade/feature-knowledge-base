---
epic: HI-5418
title: Merchant Pricebooks
status: Near Complete
last_refreshed: 2026-06-24
spec_url: https://credify.atlassian.net/wiki/spaces/PROD/pages/4250140727
confluence_page_id: 5569576986
test_checklist_ticket: HI-6505
---

# Merchant Pricebooks (HI-5418)

**Status:** Near Complete &nbsp;&nbsp; **Last Refreshed:** 2026-06-24 &nbsp;&nbsp; **Spec:** [Merchant Pricebooks](https://credify.atlassian.net/wiki/spaces/PROD/pages/4250140727) &nbsp;&nbsp; **Epic:** [HI-5418](https://credify.atlassian.net/browse/HI-5418)

## Spec Summary

Merchant Pricebooks allows merchants operating in multiple categories (e.g., pool installation + hot tub) to manage distinct configurations from a single account instead of maintaining separate accounts. Each pricebook has its own plan selection, approved loan amounts, stage funding, and merchant category — all within one merchant account.

Key flows: Merchant admins create pricebooks via the dashboard, selecting plans from their assigned preset and checking the "same business/EIN" consent box. New pricebooks requesting a new category or stage funding route to an underwriting queue (with time-in-business re-verification, bank statements, and doc requirements). Pricebooks apply across all locations. Parent/regional admins can create pricebooks within child accounts. Employees select a pricebook when creating applications, using the payment calculator, generating customer/plan-specific links, and filtering the home loan list. The pricebook dropdown/filter is only shown when more than one active pricebook exists. Underwriters/Merchant Risk on VQ can assign category, stage funding, and purchase window per pricebook. Pricebooks can be activated/deactivated with a **180-day** reactivation window. Default pricebook cannot be deactivated. When a preset changes, overlapping plans are preserved; pricebooks with 0 remaining plans revert to PENDING_PLAN_SELECTION and notify the admin. During plan revision, in-progress apps may switch pricebook (offers regenerated via CDS); approved apps are locked to their pricebook; plan revision is blocked for projects on an inactive pricebook.

Technical scope: home-improvement-merchant-srvc (entity, CRUD, plan selection, deactivation/reactivation, application flow, notifications, preset migration, VQ category eligibility query), hi-merchant-review-srvc (underwriting queue), merchant-dashboard-ui (pricebook creation, filters, dropdowns, plan selection UI, payment calculator, reporting), app-by-phone-ui / home-improvement-servicing-ui (ABP flow, VQ servicing, underwriting), notification/email services (pricebook status emails), hi-merchant-reporting-srvc + upflow2-home-improvement-dags (reporting with pricebook filters — BE merged 2026-05-27; CSV detail still in dev under HI-7301).

## Ticket Progress Summary

| Status | Count |
| --- | --- |
| Closed / Resolved | 40 |
| In Development | 1 |
| Open | 3 |
| In Definition | 0 |
| **Total** | **45** |

**Major progress since 2026-05-22 refresh:**
- **E2E test PR [qa-auto#33611](https://github.com/Credify/qa-automation/pull/33611) MERGED 2026-06-11** (~96 tests). All E2E coverage is now on master — every `IN DEV` E2E flips to `COVERED`.
- Two follow-up E2E PRs merged: [qa-auto#35895](https://github.com/Credify/qa-automation/pull/35895) "New pricebook tests" (2026-06-23) and [qa-auto#36035](https://github.com/Credify/qa-automation/pull/36035) "resolve plans against default pricebook / fail fast on empty selection" (2026-06-24).
- **HI-6753** (reporting): In Development → **Closed**. [hi-report#2151](https://github.com/Credify/hi-merchant-reporting-srvc/pull/2151) and [upflow2-hi-dags#381](https://github.com/Credify/upflow2-home-improvement-dags/pull/381) both merged 2026-05-27.
- **HI-6505** (test checklist) and **HI-6654** (UAT) → **Closed**.
- New tickets since last refresh: HI-7139 (Closed, VQ category query), HI-7140 (Closed, FE post-merge fixes), HI-7301 (In Development, pricebook info in report CSV), HI-7306 (Open, VQ pricebook config/edit access).

## Ticket Map

| Ticket | Title | Status | PR(s) | UT | IT | E2E |
| --- | --- | --- | --- | --- | --- | --- |
| [HI-5586](https://credify.atlassian.net/browse/HI-5586) | [HIMS] Pricebook Entity and CRUD | Closed | [hi-merchant#4350](https://github.com/Credify/home-improvement-merchant-srvc/pull/4350), [4351](https://github.com/Credify/home-improvement-merchant-srvc/pull/4351), [4532](https://github.com/Credify/home-improvement-merchant-srvc/pull/4532) | Done | Done | COVERED |
| [HI-5587](https://credify.atlassian.net/browse/HI-5587) | [HIMS] Plan Selection Support | Closed | [hi-merchant#4484](https://github.com/Credify/home-improvement-merchant-srvc/pull/4484), [4600](https://github.com/Credify/home-improvement-merchant-srvc/pull/4600), [qa-graphql#699](https://github.com/Credify/qa-automation-graphql/pull/699), [qa-auto#33611](https://github.com/Credify/qa-automation/pull/33611) | Done | Done | COVERED |
| [HI-5589](https://credify.atlassian.net/browse/HI-5589) | [HIMS] Category and Stage Funding Overrides | Closed | [hi-merchant#4351](https://github.com/Credify/home-improvement-merchant-srvc/pull/4351) | Done | GAP | COVERED |
| [HI-5590](https://credify.atlassian.net/browse/HI-5590) | [HIMS] Plan-Specific Link Generation | Closed | [avro-hi-lib#577](https://github.com/Credify/avro-home-improvement-lib/pull/577) | N/A | N/A | COVERED |
| [HI-5592](https://credify.atlassian.net/browse/HI-5592) | [FE][MD] Payment Calculator pricebook support | Closed | [md-ui#745](https://github.com/Credify/merchant-dashboard-ui/pull/745), [757](https://github.com/Credify/merchant-dashboard-ui/pull/757), [759](https://github.com/Credify/merchant-dashboard-ui/pull/759) | N/A | N/A | COVERED |
| [HI-5593](https://credify.atlassian.net/browse/HI-5593) | [FE][MD] Restrict pricebook creation and access | Closed | [md-ui#744](https://github.com/Credify/merchant-dashboard-ui/pull/744) | N/A | N/A | COVERED |
| [HI-5594](https://credify.atlassian.net/browse/HI-5594) | [VQ] Agent Pricebook Management | Closed | [abp-ui#3461](https://github.com/Credify/app-by-phone-ui/pull/3461) | N/A | N/A | COVERED |
| [HI-5595](https://credify.atlassian.net/browse/HI-5595) | [FE][CCP] Pricebooks in underwriting queue | Closed | [abp-ui#3599](https://github.com/Credify/app-by-phone-ui/pull/3599) | N/A | N/A | COVERED |
| [HI-5597](https://credify.atlassian.net/browse/HI-5597) | [BE] Pricebook Feature Flag | Closed | — | Done | Done | COVERED |
| [HI-5510](https://credify.atlassian.net/browse/HI-5510) | [BE] Tech Design | Closed | — | N/A | N/A | N/A |
| [HI-5596](https://credify.atlassian.net/browse/HI-5596) | [FE] QA feature branches / pricebook fixes | Closed | [md-ui#760](https://github.com/Credify/merchant-dashboard-ui/pull/760), [773](https://github.com/Credify/merchant-dashboard-ui/pull/773), [776](https://github.com/Credify/merchant-dashboard-ui/pull/776), [782](https://github.com/Credify/merchant-dashboard-ui/pull/782), [788](https://github.com/Credify/merchant-dashboard-ui/pull/788), [abp-ui#3741](https://github.com/Credify/app-by-phone-ui/pull/3741) | N/A | N/A | COVERED |
| [HI-5705](https://credify.atlassian.net/browse/HI-5705) | [BE] GraphQL Schema changes | Closed | [hi-merchant#4350](https://github.com/Credify/home-improvement-merchant-srvc/pull/4350), [4351](https://github.com/Credify/home-improvement-merchant-srvc/pull/4351), [4369](https://github.com/Credify/home-improvement-merchant-srvc/pull/4369), [4532](https://github.com/Credify/home-improvement-merchant-srvc/pull/4532), [spicedb#817](https://github.com/Credify/spicedb-schemas/pull/817) | Done | Done | COVERED |
| [HI-5859](https://credify.atlassian.net/browse/HI-5859) | [HIMS] Pricebook configuration mutations | Closed | [hi-merchant#4351](https://github.com/Credify/home-improvement-merchant-srvc/pull/4351), [4532](https://github.com/Credify/home-improvement-merchant-srvc/pull/4532) | Done | Done | COVERED |
| [HI-5860](https://credify.atlassian.net/browse/HI-5860) | [HIMS] Pricebook deactivation/reactivation | Closed | [hi-merchant#4532](https://github.com/Credify/home-improvement-merchant-srvc/pull/4532) | Done | Done | COVERED |
| [HI-5925](https://credify.atlassian.net/browse/HI-5925) | [BE] Pricebook underwriting | Closed | [hi-review#2132](https://github.com/Credify/hi-merchant-review-srvc/pull/2132), [2144](https://github.com/Credify/hi-merchant-review-srvc/pull/2144), [hi-merchant#4939](https://github.com/Credify/home-improvement-merchant-srvc/pull/4939) | Done | Done | COVERED |
| [HI-5926](https://credify.atlassian.net/browse/HI-5926) | [BE] Pricebook notifications/emails | Closed | [hi-merchant#5105](https://github.com/Credify/home-improvement-merchant-srvc/pull/5105), [notif-pref#3420](https://github.com/Credify/notification-preference-srvc/pull/3420), [hi-notif-sub#1601](https://github.com/Credify/hi-notification-subscription-srvc/pull/1601) (DB only), [email-mgt#4350](https://github.com/Credify/email-mgt-srvc/pull/4350) (DB only) | Done | Done | COVERED |
| [HI-5927](https://credify.atlassian.net/browse/HI-5927) | [BE] Pricebook migration | Closed | [hi-merchant#4369](https://github.com/Credify/home-improvement-merchant-srvc/pull/4369), [k8s-template#182044](https://github.com/Credify/k8s-template/pull/182044) | Done | Done | COVERED |
| [HI-6081](https://credify.atlassian.net/browse/HI-6081) | [BE] Pricebook queries/mutation implementation | Closed | [hi-merchant#4532](https://github.com/Credify/home-improvement-merchant-srvc/pull/4532), [4563](https://github.com/Credify/home-improvement-merchant-srvc/pull/4563) | Done | Done | COVERED |
| [HI-6088](https://credify.atlassian.net/browse/HI-6088) | [FE][CCP] Pricebook selection in ABP flow | Closed | [hi-servicing-ui#51](https://github.com/Credify/home-improvement-servicing-ui/pull/51) | N/A | N/A | COVERED |
| [HI-6129](https://credify.atlassian.net/browse/HI-6129) | Merchant preset migration pricebook support | Closed | [hi-merchant#5105](https://github.com/Credify/home-improvement-merchant-srvc/pull/5105) | Done | Done | COVERED |
| [HI-6131](https://credify.atlassian.net/browse/HI-6131) | Pricebook support for application flow | Closed | [avro-hi-lib#666](https://github.com/Credify/avro-home-improvement-lib/pull/666), [hi-merchant#5105](https://github.com/Credify/home-improvement-merchant-srvc/pull/5105) | Done | Done | COVERED |
| [HI-6135](https://credify.atlassian.net/browse/HI-6135) | Merchant configuration for Pricebook | Closed | [hi-merchant#4615](https://github.com/Credify/home-improvement-merchant-srvc/pull/4615) | Done | Partial | COVERED |
| [HI-6156](https://credify.atlassian.net/browse/HI-6156) | [FE] Sync and create epic stories | Closed | — | N/A | N/A | N/A |
| [HI-6226](https://credify.atlassian.net/browse/HI-6226) | [FE][MD] Table Filters (home & payment history) | Closed | — | N/A | N/A | COVERED |
| [HI-6227](https://credify.atlassian.net/browse/HI-6227) | [FE][MD] Pricebook dropdown for New application | Closed | [md-ui#740](https://github.com/Credify/merchant-dashboard-ui/pull/740) | N/A | N/A | COVERED |
| [HI-6228](https://credify.atlassian.net/browse/HI-6228) | [FE][MD] Reporting integration | Closed | — | N/A | N/A | COVERED |
| [HI-6229](https://credify.atlassian.net/browse/HI-6229) | [FE][MD] Customer application links integration | Closed | [md-ui#740](https://github.com/Credify/merchant-dashboard-ui/pull/740) | N/A | N/A | COVERED |
| [HI-6230](https://credify.atlassian.net/browse/HI-6230) | [FE][MD] Plan specific links integration | Closed | [md-ui#721](https://github.com/Credify/merchant-dashboard-ui/pull/721) | N/A | N/A | COVERED |
| [HI-6231](https://credify.atlassian.net/browse/HI-6231) | [FE][MD] Plan selection integration | Closed | [md-ui#730](https://github.com/Credify/merchant-dashboard-ui/pull/730) | N/A | N/A | COVERED |
| [HI-6232](https://credify.atlassian.net/browse/HI-6232) | [FE][MD] Plan revision integration | Closed | [md-ui#739](https://github.com/Credify/merchant-dashboard-ui/pull/739) | N/A | N/A | COVERED |
| [HI-6233](https://credify.atlassian.net/browse/HI-6233) | [FE][MD] Pricebook creation and management | Closed | [md-ui#696](https://github.com/Credify/merchant-dashboard-ui/pull/696), [706](https://github.com/Credify/merchant-dashboard-ui/pull/706), [709](https://github.com/Credify/merchant-dashboard-ui/pull/709), [hi-components-ui#209](https://github.com/Credify/home-improvement-components-ui/pull/209) | N/A | N/A | COVERED |
| [HI-6234](https://credify.atlassian.net/browse/HI-6234) | [FE][CCP] Underwriting new pricebook | Closed | [hi-components-ui#228](https://github.com/Credify/home-improvement-components-ui/pull/228), [abp-ui#3664](https://github.com/Credify/app-by-phone-ui/pull/3664) | N/A | N/A | COVERED |
| [HI-6235](https://credify.atlassian.net/browse/HI-6235) | [FE][CCP] Merchant servicing integration | Closed | [abp-ui#3555](https://github.com/Credify/app-by-phone-ui/pull/3555) | N/A | N/A | COVERED |
| [HI-6505](https://credify.atlassian.net/browse/HI-6505) | Merchant Pricebooks: Test coverage checklist | Closed | — | — | — | — |
| [HI-6654](https://credify.atlassian.net/browse/HI-6654) | Pricebooks UAT | Closed | — | — | — | — |
| [HI-6753](https://credify.atlassian.net/browse/HI-6753) | Support for pricebooks in hi-merchant-report-srvc | Closed | [hi-report#2151](https://github.com/Credify/hi-merchant-reporting-srvc/pull/2151) (merged 2026-05-27), [upflow2-hi-dags#381](https://github.com/Credify/upflow2-home-improvement-dags/pull/381) (merged 2026-05-27) | Done | Done | COVERED |
| [HI-6785](https://credify.atlassian.net/browse/HI-6785) | [FE][MD] Restore plan description filter + creation consent check | Closed | [md-ui#750](https://github.com/Credify/merchant-dashboard-ui/pull/750), [753](https://github.com/Credify/merchant-dashboard-ui/pull/753) | N/A | N/A | COVERED |
| [HI-6879](https://credify.atlassian.net/browse/HI-6879) | [FE][MD] Normalize "Send Application Link" behavior | Closed | [md-ui#762](https://github.com/Credify/merchant-dashboard-ui/pull/762) | N/A | N/A | COVERED |
| [HI-7099](https://credify.atlassian.net/browse/HI-7099) | Bug: Admin/finance-rep bypass for tier-aware plan resolution | Closed | [hi-merchant#5480](https://github.com/Credify/home-improvement-merchant-srvc/pull/5480) | Done | Done | COVERED |
| [HI-7139](https://credify.atlassian.net/browse/HI-7139) | VQ query for available categories per merchant | Closed | [hi-merchant#5498](https://github.com/Credify/home-improvement-merchant-srvc/pull/5498), [hi-common-lib#606](https://github.com/Credify/hi-common-lib/pull/606) | Done | Done | COVERED |
| [HI-7140](https://credify.atlassian.net/browse/HI-7140) | [FE] Post merge fixes | Closed | [md-ui#801](https://github.com/Credify/merchant-dashboard-ui/pull/801), [abp-ui#3784](https://github.com/Credify/app-by-phone-ui/pull/3784), [hi-components-ui#241](https://github.com/Credify/home-improvement-components-ui/pull/241) | N/A | N/A | COVERED |
| [HI-7301](https://credify.atlassian.net/browse/HI-7301) | Pricebook information in report CSV | In Development | — (no linked PR yet) | TBD | TBD | GAP |
| [HI-7306](https://credify.atlassian.net/browse/HI-7306) | Merchant pricebook config & edit access on VQ | Open | [abp-ui#3816](https://github.com/Credify/app-by-phone-ui/pull/3816) (OPEN) | N/A | N/A | GAP |
| [HI-7018](https://credify.atlassian.net/browse/HI-7018) | [FE][MD][v2] Stage funding series with identical stages | Open | — (no PRs yet) | N/A | N/A | GAP |
| [HI-7066](https://credify.atlassian.net/browse/HI-7066) | [FE][HISUI][v2] Pricebooks without ABP plans when creating loan | Open | — (no PRs yet) | N/A | N/A | GAP |

## E2E Coverage — qa-automation (MERGED to master)

All pricebook E2E coverage is now merged. Primary PR [qa-auto#33611](https://github.com/Credify/qa-automation/pull/33611) "HI-6421: E2E tests for Merchant Pricebooks" (branch `HI-5587-pricebook-plan-selection-e2e`) **merged 2026-06-11**, plus follow-ups [qa-auto#35895](https://github.com/Credify/qa-automation/pull/35895) (2026-06-23) and [qa-auto#36035](https://github.com/Credify/qa-automation/pull/36035) (2026-06-24).

Test classes on master (`qa-automation-tests/.../regression/homeimprovement/`):

| Test Class | Coverage |
| --- | --- |
| HomeImprovementPricebookLifecycleTest | Pricebook CRUD, deactivation/reactivation, 180-day limit, underwriting (approve/decline), preset changes, feature flag toggle, title uniqueness, aggregator guard, VQ available-categories query |
| HomeImprovementPricebookPlanSelectionTest | Pricebook-scoped plan selection (all roles), app link create/delete, borrower registration via pricebook link, TIER_TWO with pricebook, sales-rep link count, preset-migration link invalidation |
| HomeImprovementPricebookUnderwritingUiTest | VQ agent approves pricebook underwriting; **assigns purchase window per pricebook** (`testVqAgentAssignsPurchaseWindowPerPricebook`) |
| HomeImprovementMerchantDashboardSettingsPricebooksTest | Pricebook dropdown UI, plan selection via UI (all roles), create/deactivate/reactivate via UI, pricebook filter on home page, preset change status, **EIN consent checkbox**, Update Offer with pricebook modal |
| HomeImprovementMerchantReportingTest (additions) | **Draws report filtered by pricebook** (`verifyFilterDrawsByPriceBookTest`), non-default pricebook reporting lookups |
| HomeImprovementMerchantControlsInVqTest (additions) | VQ agent enable/disable pricebooks, toggle status in presets, ABP dropdown visibility rules |
| HomeImprovementPlanRevisionTest (additions) | Plan revision blocked by inactive pricebook, completes after feature disabled, uses approved pricebook plans |
| HomeImprovementPlanSelectionOngoingAppTest (addition) | Regenerate offers for ongoing app after pricebook switch |

## Coverage Matrix

| Requirement | Ticket(s) | UT | IT | E2E | Status |
| --- | --- | --- | --- | --- | --- |
| Pricebook entity CRUD | HI-5586, HI-5859 | Done | Done | COVERED | COVERED |
| Pricebook plan selection (all roles) | HI-5587, HI-6231 | Done | Done | COVERED | COVERED |
| Category & stage funding overrides | HI-5589 | Done | GAP | COVERED | PARTIAL (IT gap) |
| Plan-specific link generation | HI-5590, HI-6230 | N/A | N/A | COVERED | COVERED |
| Pricebook feature flag & config | HI-5597, HI-6135 | Done | Partial | COVERED | COVERED |
| Pricebook deactivation/reactivation (180-day) | HI-5860 | Done | Done | COVERED | COVERED |
| Pricebook in application flow | HI-6131 | Done | Done | COVERED | COVERED |
| Pricebook underwriting queue | HI-5925, HI-5595 | Done | Done | COVERED | COVERED |
| Pricebook notifications/emails | HI-5926 | Done | Done | COVERED | COVERED |
| Pricebook dropdown on new application | HI-6227 | N/A | N/A | COVERED | COVERED |
| Customer application links with pricebook | HI-6229 | N/A | N/A | COVERED | COVERED |
| Payment calculator pricebook support | HI-5592 | N/A | N/A | COVERED | COVERED |
| Pricebook creation/access permissions | HI-5593 | N/A | N/A | COVERED | COVERED |
| EIN consent checkbox on creation | HI-6785 | N/A | N/A | COVERED | COVERED |
| VQ agent pricebook management | HI-5594 | N/A | N/A | COVERED | COVERED |
| VQ purchase window per pricebook | HI-5594 (spec §3) | N/A | N/A | COVERED | COVERED |
| VQ available categories query | HI-7139 | Done | Done | COVERED | COVERED |
| ABP pricebook selection flow | HI-6088 | N/A | N/A | COVERED | COVERED |
| Pricebook reporting filters (MD UI) | HI-6228, HI-6226 | N/A | N/A | COVERED | COVERED |
| Pricebook in hi-merchant-reporting-srvc | HI-6753 | Done | Done | COVERED | COVERED |
| Pricebook info in report CSV detail | HI-7301 | TBD | TBD | GAP | IN DEV |
| Preset migration pricebook support | HI-6129 | Done | Done | COVERED | COVERED |
| Plan revision with pricebook | HI-6232 | N/A | N/A | COVERED | COVERED |
| VQ pricebook config / edit-access permissions | HI-7306 | N/A | N/A | GAP | IN DEV (abp-ui#3816 open) |

## Active Gaps

### Spec Requirement Gaps

* **[MEDIUM] HI-7306 open — VQ pricebook config & edit-access permissions.** [abp-ui#3816](https://github.com/Credify/app-by-phone-ui/pull/3816) is still open ("Expand permission check for HI pricebook enabling and activation"). No E2E for the expanded permission matrix yet. Add once merged + deployed.
* **[MEDIUM] HI-7066 open (v2) — Pricebooks without ABP plans when creating loan.** No implementation or tests yet.
* **[MEDIUM] HI-7018 open (v2) — Stage funding series with identical stages and thresholds.** No implementation or tests yet.
* **[LOW] HI-7301 in development — Pricebook information in report CSV.** BE work for adding pricebook columns to the downloadable CSV detail; no linked PR / E2E yet. Reporting *filter* is covered (`verifyFilterDrawsByPriceBookTest`); the CSV *column* detail is the remaining slice.
* **[LOW] Category/stage funding override IT** — hi-merchant#4351 (HI-5589) has no integration tests for the category/stage-funding resolver layer. Covered indirectly by later PRs and E2E.
* **[LOW] Feature flag config IT** — hi-merchant#4615 (HI-6135) has no integration tests for the pricebook config endpoint. Service-layer UT present.

### Resolved Since Last Refresh (2026-05-22)

* ~~Purchase window per pricebook in VQ~~ — **COVERED** by `testVqAgentAssignsPurchaseWindowPerPricebook`.
* ~~EIN consent checkbox~~ — **COVERED** in HomeImprovementMerchantDashboardSettingsPricebooksTest.
* ~~Pricebook reporting (HI-6753) blocked on open PR~~ — **RESOLVED**; reporting PRs merged 2026-05-27, filter E2E on master.

### Other

* **[INFO] DB-migration PRs with no tests** — hi-notif-sub#1601 and email-mgt#4350 are pure Liquibase changelog PRs. GAP expected and acceptable.
* **[INFO] notif-pref#3420** — IT only, no UT. Minimal service logic; acceptable.

## Decisions

* 2026-04-17: Feature context initialized; discovered 40+ PRs across 12 repos; only 1 E2E PR (qa-auto#33611).
* 2026-05-22: Refreshed — 8 tickets moved In Validation → Closed; all HIMS BE PRs merged; UT/IT filled in; E2E PR had ~96 tests (still open).
* 2026-06-24: Refreshed — **E2E PR qa-auto#33611 merged 2026-06-11** (+ follow-ups #35895, #36035); all E2E flipped IN DEV → COVERED. HI-6753 reporting Closed (PRs merged 2026-05-27). HI-6505/HI-6654 Closed. Three prior spec gaps (purchase window, EIN checkbox, reporting filter) verified COVERED on master. New tickets: HI-7139 (Closed, UT+IT+E2E), HI-7140 (Closed FE), HI-7301 (In Dev, report CSV detail — gap), HI-7306 (Open, VQ permission expansion — gap, abp-ui#3816). Remaining: HI-7301, HI-7306, plus v2 tickets HI-7018/HI-7066.
