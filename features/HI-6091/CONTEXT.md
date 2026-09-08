---
epic: HI-6091
title: Cross Sell
spec_url: https://credify.atlassian.net/wiki/spaces/PROD/pages/4494065719
spec_url_2: https://credify.atlassian.net/wiki/spaces/PROD/pages/5856264352
status: in-development
last_refreshed: 2026-09-08
test_checklist_ticket: HI-6478
confluence_page_id: "5605752862"
---

# Cross Sell

## Spec Summary

The Cross Sell program offers existing Upgrade customers (PL, PCL, Deposit, HI, FlexPay) a Home Improvement loan through the merchant network. Eligible borrowers see an NBA/ITA banner on their dashboard (Directory, HI Home, Manage Payments pages), explore a contractor network filtered by zip code (configurable max radius, up to 150 miles system-wide; FE slider up to 50 miles), select up to 5 merchants (max 3 per category), complete a pre-qualification form (soft credit pull, pre-populated PI1), and share contact details with selected merchants. Merchants receive leads in an "Upgrade Leads" tab, can view lead details, update lead stages, and initiate loan applications directly. Merchant eligibility requires `cross_sell_enabled`, serviceable zip codes, and optionally a Google Places ID.

**V1 is fully built and merged** (`qa-automation#34134` merged to master 2026-07-24; borrower-side FE lives in the standalone `home-improvement-borrower-dashboard-ui` repo, created 2026-06-24). On `qa-automation` **master**, the V1 cross-sell suite is 27 `@Test` methods across 10 classes, and **every one of them still carries `@SkipUntil(envToSkip = {"main","stage","preprod"}, skipBefore = "2050-12-31", reason = "More changes in cross sell are planned from business")`** — 36 `SkipUntil` occurrences across 9 files, confirmed by direct grep of `origin/master` this refresh. "COVERED\*" in this document means "code exists and is wired into `home-improvement-cross-sell-tests.xml` on master," **not** "passing in CI." The only HI cross-sell E2E that runs unconditionally on master lives in the decisioning layer (`PrequalDecisionHiclCrossSellTest`, `PrequaDecisionlHiclGoldstarTest`).

**The largest change this refresh is on the local `HI-CrossSellDirectoryPLTests` branch of `qa-automation` (still not a PR).** It has grown from the 12 tests reported on 2026-08-31 to **67 `@Test` methods across 13 classes — 39 net-new AllureIds, 3 net-new test classes, and zero `@SkipUntil` annotations anywhere in the package.** The branch deletes the blanket 2050 skip gate that Active Gap #14 of the last two refreshes flagged as the epic's biggest operational risk, and adds first-ever E2E coverage for decline cooldown, merchant-side Braze notifications, lead-expiry reminders, the merchant-initiated create-application flow, both NBA/ITA definition variants and their flip, income validation bounds, and batch supersession/backoff. **None of it is independently verifiable or CI-visible until it becomes a PR** — that is now the single highest-leverage action item in this epic.

**"Omni Pre-Qual" initiative (epic CRD-19822, `[X-Sell Omni Prequal][BE]` sub-tickets under HI-6091):** flips V1's on-demand-only pre-qualification model. CDS generates a generic HI cross-sell prequal upfront during the monthly Goldstar-style bureau refresh for consenting, eligible customers. Borrowers split into **Branch A** (valid non-expired APPROVED omni prequal exists → "you're pre-qualified" tile, no dollar amount) or **Branch B** (no valid prequal → on-demand ITA at lower priority, captures consent, graduates into the next month's batch). Workstreams: W0, W1 (batch consumer routing), W2 (activation rendezvous APPROVED→ACTIVE), W3 (prequalDecisionUuid to CDS), W5 (tests/flag/observability), W7 (rescoped to decline suppression), W_AMT, W_EXP (45-day expiry), W_SHARE (editable contact at share step).

**CORRECTION TO THE 2026-08-31 REFRESH — O1 (Branch A/B gating) is implemented; it just never belonged to W7.** The prior refresh concluded, from HI-7754's rescope note ("Dropped, not deferred"), that Branch A/B routing was *unowned and unimplemented*. That conclusion was about ticket ownership and it over-reached on implementation. **`hi-application-srvc#1102` (W1+W2) merged 2026-09-08** and delivers the batch consumer, the APPROVED→ACTIVE rendezvous, and on-demand reuse of a batch-APPROVED prequal; **`next-best-action-srvc#3418` merged 2026-09-02** and ships the two competing ITA/banner definitions (`ITA_HI_GETPREQUAL_1` for Branch B, `ITA_HI_PREQUAL_XSELL_1` for Branch A, plus their banner twins). Branch-A/B routing is the emergent behaviour of those two pieces, and the local branch now proves it end-to-end on a live stack: tests `[84225]`/`[84226]`/`[84236]` assert the two definitions are mutually exclusive and flip on prequal activation, `[85075]` asserts on-demand create adopts a batch-APPROVED row instead of re-deciding, and `[85068]` asserts a DECLINED batch decision suppresses the ITA entirely. What genuinely remains dropped from W7 is **O18** (re-decision against a locked policy version via a prequal-decision-srvc REST client) — that is still unowned. One caveat worth carrying: `#3418`'s definitions ship gated at `starts_at=2050-01-01`, so the E2E has to activate them, and they are not live for real borrowers yet.

## Ticket Map

| Ticket | Title | Type | Status | PR(s) | UT/IT | E2E | Gap |
| --- | --- | --- | --- | --- | --- | --- | --- |
| HI-6395 | [BE] Design | Story | Closed | -- | -- | -- | Design-only |
| HI-6396 | [FE][Borrower] Add NBA banner image on BD | Story | Closed | bd-ui#7082 (superseded), bd-ui#7826 (MERGED, asset re-add) | N/A (FE) | COVERED* | Interactive banner now lives in new-repo#9 |
| HI-6469 | [BE] Google Places API Integration | Story | Closed | hi-merchant#5436 (MERGED) | Done | COVERED* | -- |
| HI-6470 | [BE] Merchant serviceable zip code list support | Story | Closed | hi-merchant#5436 (MERGED); hi-merchant#5649 (DECLINED, no tests, abandoned) | Done | COVERED* | -- |
| HI-6494 | [FE][Borrower] Contractors exploration page + list | Story | Closed | bd-ui#7151 (superseded), new-repo#9 (MERGED) | N/A (FE) | COVERED* | FE now in new repo |
| HI-6495 | [FE][Borrower] Filter section for contractors exploration | Story | Closed | bd-ui#7121 (superseded), new-repo#9 | N/A (FE) | COVERED* | FE now in new repo |
| HI-6496 | [FE][Borrower] Contractors listing API | Story | Closed | bd-ui#7277 (superseded), new-repo#9 | N/A (FE) | COVERED* | FE now in new repo |
| HI-6497 | [FE][Borrower] Pre-qualification page + form/modal | Story | Closed | bd-ui#7194 (superseded), new-repo#9; qa#37388 (MERGED, nav fix) | N/A (FE) | COVERED* | FE now in new repo |
| HI-6498 | [FE][Borrower] Pre-qualification success page | Story | Closed | bd-ui#7202 (superseded), new-repo#9 | N/A (FE) | COVERED* | FE now in new repo |
| HI-6499 | [FE][Borrower] Info sent page | Story | Closed | -- | N/A | N/A | -- |
| HI-6506 | [BE] Cross Sell pre-qual lead design | Story | Closed | spicedb#1235 (MERGED) | N/A (schema) | -- | -- |
| HI-6533 | [BE] Check Borrower Eligibility | Story | Closed (Done) | avro-hi-lib#792 (MERGED), upflow2-hi-dags#407/#531 (MERGED), k8s#230588/#255901/#256579/#274770 (MERGED); hi-app#583, hi-merchant#5649, avro-hi-lib#797, k8s#259049 (all DECLINED) | Done (DAG + avro event) | **COVERED (branch)** — `[74461]` `borrowerItaEligibilityPersistedTest` (master, SkipUntil) plus `[85068]`/`[85070]` ITA-suppression assertions on the branch | Row added this refresh — was referenced in the Coverage Matrix but never had a Ticket Map entry |
| HI-6534 | [BE] Start cross sell pre-qual | Story | Closed | hi-app#570 (MERGED) | Done | COVERED* | -- |
| HI-6535 | [BE] Submit cross sell pre-qual | Story | Closed | hi-app#570 (MERGED) | Done | COVERED* (approved path); decline path S3 GAP | Decline/AAN path never delivered — see HI-7294 (Won't Do) |
| HI-6536 | [BE] Share contact with merchant | Story | Closed | hi-app#570 (MERGED) | Done | COVERED* | -- |
| HI-6537 | [BE] Merchant user lead query | Story | Closed (Duplicate) | -- | N/A | N/A | Closed as duplicate — not a gap |
| HI-6538 | [BE] Merchant manage pre-qual lead stage | Story | Closed | hi-app#570 (MERGED) | Done | COVERED* + **branch** `[83196]` per-merchant stage isolation | -- |
| HI-6539 | [BE] Borrower Eligibility management | Story | Closed (Duplicate) | -- | N/A | N/A | Closed as duplicate of HI-6533 — not a gap |
| HI-6540 | [BE] Borrower notification | Story | Closed (Done) | hi-app#570 (MERGED); hi-app#545/#555 (DECLINED) | Done | **COVERED (branch)** — `[78105]` + `[85077]` (batch-created-ACTIVE variant) + `[83513]` `hi_prequal_lead_submitted` to borrower + `[83516]` `hi_borrower_merchant_suspended` | Row added this refresh. Borrower Braze coverage went from 1 event to 4 on the branch |
| HI-6541 | [BE] Merchant Notifications | Story | Closed (Done) | qa#37317 (MERGED, mislabeled — asserts the *borrower* event); hi-app#571 (DECLINED) | Done | **COVERED (branch)** — new class `HomeImprovementCrossSellMerchantBrazeEventTest`: `[83514]` `hi_merchant_lead_prequal`, `[83515]` `hi_merchant_app_submitted`, `[84335]` expiring-lead alert + notice | Row added this refresh. **This was a standing MEDIUM gap since the 2026-06-19 refresh and is now closed on the branch** |
| HI-6542 | [BE] Parent Portal (Phase 1) | Story | Closed | upflow2#363 (MERGED) | N/A (infra) | COVERED* — `[74459]` parent Pre-qualified Leads report tab | -- |
| HI-6543 | [BE][P2] Reporting & Metrics - Funnel Metrics | Story | Open | hi-app#570 (MERGED, partial) | Partial | GAP | Scope still open; see also HI-7938 |
| HI-6544 | [BE] VQ Application Lookup | Story | Closed (Done) | -- | -- | -- | -- |
| HI-6546 | [BE] Scheduled job to remind merchants about lead expiry | Story | Closed | hi-app#570 (MERGED), k8s#285334 (MERGED — schedules the job, enables cross-sell notices in non-prod) | Done | **COVERED (branch)** — `[84335]` `verifyMerchantSeesAndIsNotifiedOfExpiringLeadTest` asserts both the "Lead Expires: 7d Left" dashboard alert and the merchant Braze notice | Row added this refresh |
| HI-6631 | [FE] Move ContactDetailsCard to URC | Story | Closed | -- | N/A | N/A | -- |
| HI-6642 | [FE][Borrower] Share contact page | Story | Closed | bd-ui#7203 (superseded), new-repo#9 | N/A (FE) | COVERED* | -- |
| HI-6647 | [FE][Borrower] Featured merchant badge/sorting | Story | Closed | bd-ui#7208 (DECLINED) | N/A (FE) | GAP | Re-implemented under HI-6720 |
| HI-6659 | [BE] CrossSell priority config for merchant | Story | Closed | hi-merchant#5102 (MERGED) | Done | **COVERED (branch)** — `[84304]` asserts priority can be set and a negative value is rejected | Was "GAP — still no E2E" for three consecutive refreshes; now closed on the branch |
| HI-6720 | [FE][Borrower] Featured merchant ordering API | Story | Closed | bd-ui#7494 (superseded), new-repo#9 | N/A (FE) | PARTIAL | Branch exercises priority-driven ranking as a *setup* mechanism (`createTopRankedCrossSellMerchant` demotes rivals then pins priority 0), but never asserts badge/force-ranking as behaviour under test |
| HI-6743 | [FE][Borrower] Google reviews API integration | Story | Closed | bd-ui#7281 (superseded), new-repo#9 | N/A (FE) | PARTIAL | `[57009]` asserts `merchantReviews` is populated at the API layer; no UI-level review-modal assertion |
| HI-6745 | [FE][Borrower] Modify mobile filters | Story | Closed | bd-ui#7420 (superseded), new-repo#9 | N/A (FE) | GAP | -- |
| HI-6750 | [FE][Borrower] BE API at sharing contact flow | Story | Closed | bd-ui#7395 (superseded), new-repo#9 | N/A (FE) | COVERED* | -- |
| HI-6751 | [FE][Borrower] BE API at pre-qualification | Story | Closed | bd-ui#7395 (superseded), new-repo#9 | N/A (FE) | COVERED* | -- |
| HI-6754 | [FE][Merchant] List Upgrade Leads on homepage | Story | Closed | md-ui#747/748/751 (MERGED) | N/A (FE) | COVERED* | -- |
| HI-6755 | [FE][Merchant] Lead Details page for Upgrade Leads | Story | Closed | md-ui#748/749 (MERGED) | N/A (FE) | COVERED* + **branch** `[84148]` original-contact case | -- |
| HI-6756 | [FE][Merchant] New Create Application page for leads | Story | Closed (**Not Needed**) | -- | N/A | **COVERED (branch)** — `[83127]`, `[83128]`, `[83168]`, `[83194]`, `[83195]`, `[83197]`, `[83200]`, `[83443]` | Ticket closed as not needed — the flow reuses the Gold Star Leads create-application pattern. The flow is live in-product; master still has **zero** coverage for it. 8 branch tests cover merchant- and borrower-initiated app creation, lead status→APP_CREATED, per-merchant lead isolation, App-Started button suppression, prequal-expiry propagation, batch-originated lead parity, and the APP_CREATED-must-not-revert-to-EXPIRED regression |
| HI-6766 | [FE][CCP] Cross Sell config in Merchant Features | Story | Closed | abp-ui#3622/3630 (MERGED) | N/A (FE) | COVERED* | -- |
| HI-6769 | [BE] NBA configuration for Cross Sell | Story | **Closed (Done)** | hi-app#634 (MERGED), nba#3398 (MERGED), **nba#3418 (MERGED 2026-09-02)** | Done | **COVERED (branch)** — `[82676]`/`[82677]` Directory banner for PL/PCL borrowers; `[84225]`/`[84226]` ITA get-prequal → prequal-xsell flip with exact spec copy on all 7 declared ITA placements; `[84236]` same flip on both banner placements | Was "IN DEV (local branch)"; now the strongest-covered area of the epic. **Operational caveat: `#3418`'s definitions ship gated at `starts_at=2050-01-01`** — the E2E activates them, so passing tests do not imply the ITAs are live for real borrowers |
| HI-6770 | [FE][CCP] Borrower Servicing Zip Code | Story | Closed | abp-ui#3630/3632 (MERGED) | N/A (FE) | COVERED* | -- |
| HI-6771 | [FE][CCP] Places ID for Google Reviews | Story | Closed | abp-ui#3630/3631 (MERGED) | N/A (FE) | COVERED* | -- |
| HI-6828 | [FE][MD] Updates required by Design/Product | Task | Closed | md-ui#756 (MERGED) | N/A (FE) | N/A | -- |
| HI-6887 | [FE][Parent] Updates to support Cross Sell | Story | Closed | mpd-ui#163/166 (MERGED) | N/A (FE) | COVERED* | -- |
| HI-6930 | [FE][CCP] Split Cross Sell eligibility/priority configs | Task | Closed | abp-ui#3747 (MERGED) | N/A | PARTIAL | `[84304]` now covers the priority half of the split at the API layer |
| HI-7004 | [FE][CCP] Updates for Servicing Zip Codes config | Task | Closed | abp-ui#3766 (MERGED) | N/A | GAP | -- |
| HI-7012 | [FE][MD] Conditionally display Pre-qual features in Reporting | Task | Closed | mpd-ui#165 (MERGED) | N/A | GAP | -- |
| HI-7017 | [FE][CCP] Display Zip/Places configs if eligibleForCrossSell | Task | Closed | abp-ui#3746 (MERGED) | N/A | GAP | -- |
| HI-7036 | [FE] Cleanups (post-deployment) | Task | Open | -- | N/A (FE) | GAP | Still no PRs |
| HI-7077 | [FE][Borrower] NBA and Resumption tiles on explore contractors | Story | Closed | No dedicated PR — folded into new-repo#9 rollup | N/A (FE) | PARTIAL | `[84987]` covers the "How it works" in-banner state for a not-yet-prequalified borrower; resumption tile itself still unasserted |
| HI-7158 | [BE] Sign Agreements when submitting cross sell | Story | Closed | hi-app#570 (MERGED) | Done | PARTIAL | Consent display asserted (`[74455]`); full signing-on-submit not independently re-verified. See HI-7927 for the htmlDmsDocumentUuid re-send |
| HI-7223 | [BE] Add PreQualificationContact under Lead | Story | Closed | hi-app#570 (MERGED) | Done (UT) | **COVERED (branch)** — `[83285]` and `[84148]` read the contact block off the merchant Lead Details page in both edited and unedited states | -- |
| HI-7243 | Heap analytics trackings | Story | Closed | new-repo#9 (`heap-events.test.js`, MERGED) | N/A (FE, unit-tested) | GAP | No E2E (analytics events are not E2E-observable from this framework) |
| HI-7294 | [BE] Handle AAN (Adverse Action Notice) | Story | Closed | **None — resolution "Won't Do"** | N/A | N/A (deliberately not built) | **Not a gap to fix.** But see `[85067]` on the branch, which asserts an *ineligible* borrower is not misrouted to the declined page's "processing, check your email" copy — the nearest thing to AAN-path coverage that exists |
| HI-7369 | Improvement on google photo delivery | Story | Closed | hi-merchant#5631 (MERGED) | Done | GAP | -- |
| HI-7397 | Add idempotency-lib | Story | Closed | hi-application-srvc#810 (MERGED) | Done (`PreQualificationMutationResolverTest`, `CrossSellIdempotencyIT`) | COVERED* (`[55002]` on master) + **branch** `[85073]` batch replay idempotency | -- |
| HI-7411 | [FE] New repo for HI Portal (Cross Sell) | Story | Closed | auth-sdk-ui#291, auth-ui#487, github-terraform#4729, home-improvement-borrower-dashboard-ui#1-9 (MERGED), k8s#255981/262255/269432 | N/A (infra/bootstrap) | N/A | Standalone repo live since 2026-06-24 |
| HI-7441 | Verify required info sent to CDS from pre-qual | Story | Closed | hi-merchant#5777 (MERGED, UT only), loan-app-creation-srvc#9187 (MERGED, no tests) | Partial | GAP | No IT, no E2E |
| HI-7504 | Add Google maps attribution | Story | Closed | No dedicated PR; code confirmed in new-repo#9 (`GoogleMapsAttribution` + test) | N/A (FE) | GAP | -- |
| HI-7505 | Extra merchant placeholder images by category | Story | Resolved | home-improvement-borrower-dashboard-ui#12 (**still OPEN**) | N/A (FE) | GAP | Status/PR mismatch persists a second refresh — WATCH |
| HI-7506 | Hide filter categories if no merchants | Story | Closed | No dedicated PR; bundled in new-repo#9 FilterSection | N/A (FE) | GAP | -- |
| HI-7704 | [BE] Minimal backend defense-in-depth | Story | Closed | hi-merchant#5904 (MERGED, UT only) | Partial | GAP | No IT, no E2E; low business risk |
| HI-7754 | [Omni][BE] W7 — local cross-sell decline record + 90-day cooldown (rescoped 2026-08-21) | Story | **In Validation** | hi-application-srvc#1105 (MERGED), #1102 (**MERGED 2026-09-08**) | **Done** — `CrossSellDeclineServiceTest`, `CrossSellDeclineRepositoryIT`, `CrossSellDeclineActorChangeStrategyTest`, `PrequalDecisionComputedEventHandlerTest`/`IT` | **COVERED (branch)** — new class `HomeImprovementCrossSellDeclineCooldownTest`: `[85068]` decline recorded + `DECLINED_COOLDOWN` rejection + ITA suppressed, `[85070]` later approval ends cooldown pre-activation, `[85071]` replayed/out-of-order older decline never shortens a live cooldown | **Standing HIGH gap from the last refresh — now closed.** O18 (re-decision against locked policy) remains dropped from this ticket and unowned |
| HI-7755 | [Omni][BE] W1 — Batch consumer routing + applicant hydration | Story | **In Validation** | hi-application-srvc#1102 (**MERGED 2026-09-08**), #1126 (OPEN, W5 flag), #1105 (MERGED) | **Done** — 23 test files touched incl. `PrequalDecisionComputedEventHandlerTest`/`IT`, `CrossSellBorrowerEligibilityIT`, `PreQualificationServiceIT` | **COVERED** — `qa#37758` (master, SkipUntil) + branch `[82282]`, `[82283]`, `[82292]`, `[82293]`, `[85073]` supersession, `[85074]` in-flight backoff, `[85076]` Gold Star non-supersession | BE PR merged this refresh; the E2E contract the branch assumes is now the real one |
| HI-7756 | [Omni][BE] W2 — Activation rendezvous (APPROVED→ACTIVE) | Story | **In Validation** | hi-application-srvc#1102 (**MERGED 2026-09-08**) | **Done** (shared with W1) | **COVERED (branch)** — `[82283]` ITA-completes-rendezvous, `[85072]` consent-arrives-after-ITA (reverse order), `[85075]` on-demand create adopts a batch-APPROVED row with no new credit decision | Both rendezvous trigger orders now asserted |
| HI-7757 | [Omni][BE] W3 — Send prequalDecisionUuid to CDS | Story | Closed (Done) | loan-app-creation-srvc#9287 (MERGED) | Partial (UT only — `HomeImprovementCreditDecisionModelFactoryTest`) | GAP | No IT, no E2E. This is the CDS handoff wiring for O17/O18 |
| HI-7758 | [Omni][BE] W_SHARE — Editable contact at share step | Story | Closed (Done) | hi-application-srvc#1038 (MERGED), qa#37912 (MERGED), qa-gql#1085 (MERGED) | Done | **COVERED (branch)** — `[85004]` borrower edits then merchant sees it; `[85003]` batch borrower with no applicant yet edits actor-profile-sourced contact; `[83285]`/`[84148]` merchant-side edited vs unedited | The Actor-vs-Applicant visibility gap this ticket exposed is fixed — see HI-8025 |
| HI-7759 | [Omni][BE] W_EXP — 45-day cross-sell prequal expiry | Story | Closed (Done) | No PR captured in scanned repos (resolution = Done) | Done (per resolution) | PARTIAL — branch `[83197]` asserts expiry propagates across all merchant leads; `[84387]` asserts a merchant can still start an app on an expired batch lead; `[83443]` asserts an APP_CREATED lead is not reverted to EXPIRED | S29: spec still self-contradicts 30 vs 45 days. Branch constant `CROSS_SELL_EXPIRATION_DAYS` encodes one value; the two-window question is still open |
| HI-7760 | [Omni][BE] W5 — Tests / feature flag / observability | Story | **In Validation** | hi-application-srvc#1126 (**still OPEN**, dark-launch flag) | Partial (PR open; 23 test files touched) | GAP | Flag exists in-PR; no E2E asserting flag-off (consumer no-ops) vs flag-on |
| HI-7761 | [Omni][BE] W0 — avro-decisioning-lib bump | Story | Closed (Self-Resolved) | Not in tracked repo set | N/A (dependency) | N/A | Not independently verified |
| HI-7762 | [Omni][BE] W_AMT — Suppress prequal amount | Story | Closed (**Won't Do**) | None | N/A | N/A | Superseded by HI-7765, which shipped — see below |
| HI-7765 | [FE][Borrower] Hide the pre-qualification amount on explore-contractors title and banner | Story | **Closed** | **home-improvement-borrower-dashboard-ui#25 (MERGED 2026-09-03)** | N/A (FE) | PARTIAL | **Status flip this refresh: was Blocked with no PRs; O3 is now implemented on the borrower title/banner.** Branch `verifyDefinitionCopy(..., expectsAmountDisclaimer)` asserts the prequal-state NBA carries only a `$$CODE-1$$` disclaimer marker, not a rendered amount — indirect but real evidence. A direct "no dollar figure on any borrower surface (tile, funnel, email, SMS)" assertion still does not exist |
| HI-7766 | [FE][Borrower] Redesign the "How it works" section on Explore Contractors | Story | **Resolved** | home-improvement-borrower-dashboard-ui#26 (OPEN) | N/A (FE) | **COVERED (branch)** — `[84987]` asserts the "How it works" steps render inside the banner for a not-yet-prequalified borrower | Status/PR mismatch: Resolved while #26 is still open — WATCH |
| HI-7767 | [FE][Borrower] NBA component at bottom of explore contractors (new PL placement) | Story | In Development | **home-improvement-borrower-dashboard-ui#29 (OPEN)** | N/A (FE) | SPEC GAP | O26 (PL cross-sell fallback). Was "no PRs"; now has one in flight |
| HI-7802 | [FE][Borrower] Pre-fill application screen (Omni flow) | Story | **Closed (Done)** | No dedicated PR captured | N/A (FE) | GAP | Was Blocked; now Done with no PR in the tracked set and no E2E — WATCH |
| HI-7909 | Set prequalDecisionUuid null for on-demand prequal | Story | In Validation | hi-application-srvc#1098 (**CLOSED unmerged**) | Partial | GAP | **PR was closed without merging while the ticket sits In Validation.** The behaviour was likely absorbed into #1102's rewrite of the same handlers — worth confirming with the author rather than assuming |
| HI-7918 | Clean up pre-qual dead code | Task | **Closed (Done)** | **hi-application-srvc#1189 (MERGED 2026-09-02)**, **qa#38091 (MERGED 2026-09-02)** | **Done** — `PreQualificationEventHandlerTest`/`IT`, `PreQualificationMapperTest` | N/A (cleanup) | Was "Open, no PRs". `HomeImprovementPreQualifiedEvent` consumption path removed on both sides |
| HI-7927 | [FE][Borrower] Re-send htmlDmsDocumentUuid and agreementReadDateTime on pre-qual agreements | Story | Ready for CodeReview | home-improvement-borrower-dashboard-ui#27 (OPEN) | N/A (FE) | GAP | **New ticket this refresh.** Agreement-signing payload fidelity; relates to HI-7158 |
| HI-7928 | [FE][Borrower] Editable contact at share-contact step, sourced from Applicant then Actor | Story | **In Validation** | **home-improvement-borrower-dashboard-ui#28 (OPEN)** | N/A (FE) | **COVERED (branch)** — `[85003]` asserts a batch borrower with no applicant row yet gets actor-profile-sourced contact at confirm-share and can edit it; `[85067]` asserts an ineligible borrower is not misrouted to the declined page | Was "Ready For Eng, no PRs, SPEC GAP" — now has a PR and E2E ahead of it |
| HI-7938 | Update report to pull borrower contact info correctly based on if applicant is created | Story | In Development | -- | GAP (no PRs) | GAP | Reporting-side counterpart to HI-7928/HI-8025 |
| HI-8024 | [FE][MD] Update Cross Sell tab disclaimer | Task | Blocked | -- | N/A (FE) | GAP | Copy still says "Leads expire 30 days after pre-qualification" — needs reconciling with the 45-day Omni expiry (S29/O8) |
| HI-8025 | [FE][MD] Handle the different sources for contact info in the Lead Details page | Task | **Resolved** | **merchant-dashboard-ui#899 (MERGED 2026-09-02)** — "Read contact info from the applicant version for `UPGRADE_LEAD` leads" | N/A (FE) | **COVERED (branch)** — `[85004]` and `[83285]` assert the merchant sees the borrower's *edited* address/phone; `[84148]` asserts the unedited case still shows the original | **The bug reported in the 2026-08-31 refresh is fixed.** The Lead Details page no longer sources `UPGRADE_LEAD` contact from `Actor.profile`; it reads the applicant version |
| HI-8026 | [FE][MD] Show the Cross Sell tab even if merchant has no projects | Task | Blocked | -- | N/A (FE) | GAP | Still affects QA setup: branch helpers give merchants a project first (`registerCustomerInitiatedHomeImprovementLoan`, `findOrCreateSimpleCompletedProjectForPrequal`). Do not remove that workaround before this ships |
| HI-8027 | [FE][Borrower] Pre-qualification income validation (allow $0, cap at $3M) | Story | **In Validation** | **home-improvement-borrower-dashboard-ui#30 (MERGED 2026-09-08)** | N/A (FE) | **COVERED (branch)** — `[85069]` $0 individual income accepted and persisted through the subgraph; `[85078]` income above the $3,000,000 cap blocks submit | Was "Open, no PRs, GAP" — shipped and covered inside one refresh cycle |
| HI-8036 | [BE] Merchant read applicant | Story | **Closed (Done)** | **spicedb-schemas#1899 (MERGED 2026-09-01)**, **applicant-srvc#2353 (MERGED 2026-09-02)**, **hi-application-srvc#1178 (MERGED 2026-09-02)** | **Done** — `LeadAccessRelationshipUpdaterTest` (19/19), `LeadAccessRelationshipUpdaterIT` (4/4), `LeadContactGrantMarkerIT`, `ApplicantQueryResolverIT` | Indirect — the merchant-side contact tests `[85004]`/`[83285]`/`[84148]` only pass because this authz chain works | **The full 3-repo sequenced authz fix landed this refresh.** The "Missing privilege READ_APPLICANT" 403 that blocked a merchant from reading a shared lead's applicant is resolved end to end |
| HI-8075 | Setup confidence score and CI gate for new cross sell services | Task | Open | -- | N/A (CI) | N/A | **New ticket this refresh.** Directly relevant: this is the ticket under which the branch's `@SkipUntil` removal and CI gating should land |
| HI-8085 | [BE] CONTACT_GRANT_GIVEN is not recorded | Bug | **Closed (Done)** | **hi-application-srvc#1201 (MERGED 2026-09-04)** — "record CONTACT_GRANT_GIVEN from a clean transaction" | **Done** — `LeadAccessRelationshipUpdaterTest`, `LeadAccessRelationshipUpdaterIT`, `LeadContactGrantMarkerIT` | GAP | **New ticket this refresh — a real production bug, found and fixed inside the cycle.** Contact-grant marker was lost when written inside a failing/rolled-back transaction. No E2E asserts the marker itself |
| HI-8093 | [FE][Borrower] Add State Disclosures to the cross-sell pre-qualification agreements | Story | Blocked | home-improvement-borrower-dashboard-ui#33 (DRAFT) | N/A (FE) | **SPEC GAP [HIGH]** | **New ticket this refresh.** Legal/compliance — state-specific disclosure copy at prequal consent. Blocked on HI-8095 |
| HI-8094 | [FE][Borrower] Handle projectNotes validation errors on connect-with-contractor (2000 chars + valid text) | Story | Ready for CodeReview | home-improvement-borrower-dashboard-ui#34 (OPEN) | N/A (FE) | GAP | **New ticket this refresh.** Direct analogue of HI-8027's income bounds, which the branch already covers — the same test pattern applies |
| HI-8095 | [BE] Support State disclosure | Story | Open | -- | GAP (no PRs) | **SPEC GAP [HIGH]** | **New ticket this refresh.** BE half of HI-8093. Legal/compliance, not started |
| HI-8097 | [BE] Put hi_prequal_offer in array prequalDecisions | Story | In Development | **hi-application-srvc#1203 (OPEN)**, **external-actor-engagement-srvc#4443 (OPEN)**, **avro-funnel-lib#835 (OPEN)** | **Done (all three)** — hi-app: `PreQualificationCustomerEngagementServiceTest`, `PreQualificationLeadEventListenerIT`; eaes: `PrequalDecisionMapperTest`, `PrequalDecisionBuilderTest`, `BrazePersonaMapperTest`, `PrequalDecisionComputedEventHandlerIT`; avro: codegen test | GAP | **New ticket this refresh.** Restructures the Braze persona payload from a single `hi_prequal_offer` field to a `prequalDecisions` array covering GOLD_STAR and cross-sell. **This will break branch tests `[78105]`/`[85077]`, which assert the current single-field shape** — schedule the E2E update alongside the merge |
| HI-8098 | [BE] Upgrade HI prequal to use prequal-decision V2 API in NBA | Story | Open | -- | GAP (no PRs) | GAP | **New ticket this refresh.** Watch: this is where a prequal-decision-srvc client re-enters the picture, i.e. the natural home for the O18 scope W7 dropped |
| HI-8103 | [BE] Optimize preQualificationContactGrantRevocationJob | Story | Ready for CodeReview | **hi-application-srvc#1206 (OPEN)** | **Done** — `PreQualificationContactGrantRevocationJobTest`, `PreQualificationExpirationServiceTest`/`IT`, `LeadContactGrantMarkerIT` | GAP | **New ticket this refresh.** Performance work on the grant-revocation job; no E2E asserts revocation-on-expiry behaviour at all |
| HI-8106 | [FE][Borrower] Block the 6th contractor with the max-selection screen | Story | In Development | -- | N/A (FE) | GAP | **New ticket this refresh.** The "max 5 merchants / 3 per category" rule (S-series) has never had an E2E asserting the block |
| HI-6478 | Cross-Sell Program: Test Coverage Checklist | Task | Reopened | qa#34134 (MERGED) | N/A (QA) | N/A | Checklist ticket — updated by this refresh |

\* **COVERED\*** = the test code exists and is wired into `home-improvement-cross-sell-tests.xml` on `qa-automation` **master**, but carries `@SkipUntil(... skipBefore="2050-12-31")` and therefore does **not** run in CI on main/stage/preprod.
**COVERED (branch)** = the test exists only on the local, unpushed `HI-CrossSellDirectoryPLTests` branch, where the `@SkipUntil` gate has been removed entirely. Not verifiable by anyone but the branch owner until it becomes a PR.

**PR Classification Summary (this refresh):** 123 unique PRs across 27 repos are now linked to this epic. This refresh newly checked 9 service-repo PRs for UT/IT (`hi-application-srvc` #1102/#1105/#1126/#1178/#1189/#1201/#1203/#1206, `external-actor-engagement-srvc` #4443) plus `applicant-srvc#2353`; **all 10 carry both unit and integration tests** — the strongest UT/IT showing of any refresh in this epic. UI repos (`home-improvement-borrower-dashboard-ui` ×8, `merchant-dashboard-ui` ×1), schema/avro repos (`avro-funnel-lib#835`, `spicedb-schemas#1899`), and infra (`k8s-template` ×4, `upflow2-home-improvement-dags` ×2) are N/A per classification rules; `qa-automation`/`qa-automation-graphql` PRs are the E2E layer itself.

## PR Analysis

_(Entries from the 2026-06-19, 2026-08-21 and 2026-08-31 refreshes are carried forward verbatim below the new ones — see Confluence page history for the full historical detail.)_

### hi-application-srvc#1102 — Batch cross-sell prequal consumer + activation rendezvous (HI-7755 W1, HI-7756 W2) — **MERGED 2026-09-08**

_Analyzed: 2026-09-08 (status change from OPEN → MERGED; UT/IT verified for the first time)_

**Changes**: Implements W1 batch consumer routing and W2 activation rendezvous (APPROVED→ACTIVE) together. Merged the same day as this refresh.

**UT/IT**: **Done.** 23 test files touched, including `PrequalDecisionComputedEventHandlerTest` + `PrequalDecisionComputedEventHandlerIT`, `PreQualificationEligibilityCheckedEventHandlerTest` + `IT`, `CrossSellBorrowerEligibilityIT`, `CrossSellMutationsIT`, `CrossSellIdempotencyIT`, `PreQualificationServiceTest`/`IT`, `CrossSellDeclineServiceTest`, `CrossSellDeclineRepositoryIT`, `BorrowerItaEligibilityServiceTest`, `ConsentStatusChangedServiceTest`, `CrossSellAccountStatusSyncHandlerTest`.

**E2E**: `qa#37758` (master, SkipUntil-disabled) plus 5 net-new branch tests: `[85072]` reverse-order rendezvous (consent after ITA), `[85073]` monthly-rerun supersession with lead repointing and account carry-forward, `[85074]` batch backs off while an on-demand prequal is in flight, `[85075]` on-demand create adopts a batch-APPROVED row without a new credit decision, `[85076]` a Gold Star decision leaves an ACTIVE batch cross-sell row untouched.

**Resolves the prior refresh's open risk** that "E2E merged ahead of the BE PR it depends on." The BE contract is now merged and the branch tests were written against it.

**Gaps**: [MEDIUM] The dark-launch flag that gates this consumer (`#1126`) is still open and unmerged — no E2E asserts flag-off behaviour.

### hi-application-srvc#1178 + applicant-srvc#2353 + spicedb-schemas#1899 — Merchant applicant-read authz chain (HI-8036) — **ALL MERGED**

_Analyzed: 2026-09-08_

**Changes**: The 3-repo sequenced fix tracked as DRAFT/OPEN last refresh is complete. `spicedb-schemas#1899` (MERGED 2026-09-01) defines a new `read_applicant` permission on `account` — deliberately not the existing `read` permission, which ~15 unrelated services already hold. `applicant-srvc#2353` (MERGED 2026-09-02) enforces the check on applicant read. `hi-application-srvc#1178` (MERGED 2026-09-02) writes the relationship tuples when a cross-sell lead is shared.

**UT/IT**: **Done.** `LeadAccessRelationshipUpdaterTest` (19/19), `LeadAccessRelationshipUpdaterIT` (4/4), `LeadContactGrantMarkerIT`, plus `ApplicantQueryResolverIT` and a new `applicant-query.graphql` fixture on the applicant-srvc side. `spicedb-schemas#1899` ships `tests/home-improvement/assertions.yaml` + `relationships.txt`.

**E2E**: No test asserts the permission directly, but the branch's merchant-side contact tests (`[85004]`, `[83285]`, `[84148]`) exercise it transitively — a merchant reading a shared lead's applicant contact is exactly the call that used to 403.

**Gaps**: [LOW] No negative test asserting a merchant *without* a shared lead is still denied. Worth adding — this is a permission grant, and only the allow path is covered.

### merchant-dashboard-ui#899 — Read contact info from the applicant version for UPGRADE_LEAD leads (HI-8025) — **MERGED 2026-09-02**

_Analyzed: 2026-09-08_

**Changes**: Fixes the bug reported in the 2026-08-31 refresh. The Lead Details page previously sourced borrower address/phone/email from `Actor.profile` for all lead types, so a borrower's edit at the confirm-share step (which writes to applicant-srvc's `Applicant`) was invisible to the merchant. `UPGRADE_LEAD` leads now read the applicant version.

**UT/IT**: N/A (UI repo per classification rules).

**E2E**: Branch `[85004]` (borrower edits → merchant sees the edit) and `[83285]` (merchant-side assertion of the same) are the regression tests for exactly this defect; `[84148]` guards the complementary case, that an *unedited* lead still shows the original contact — which is the failure mode a naive fix would introduce.

**Gaps**: None for this PR's scope. Both directions of the branch are covered.

### home-improvement-borrower-dashboard-ui#25 — Hide the pre-qualified amount on explore-contractors title (HI-7765, O3) — **MERGED 2026-09-03**

_Analyzed: 2026-09-08_

**Changes**: Removes the pre-qualified dollar amount from the explore-contractors page title and banner. This is the live successor to HI-7762 (Won't Do) and the first real implementation of O3.

**UT/IT**: N/A (UI repo).

**E2E**: Indirect. The branch's `verifyDefinitionCopy(..., expectsAmountDisclaimer)` asserts the prequal-state NBA title carries only a `$$<CODE>-1$$` disclaimer marker — the API-visible proof that a disclaimer referencing `${...qualifiedAmount:amountInteger}` is attached, rather than a rendered amount. The merchant side is separately asserted to *have* the amount (`[74463]` `upgradeLeadDetailShowsChannelAmountAndContact`), which is the correct asymmetry.

**Gaps**: [MEDIUM] O3's actual claim is "the borrower never sees the amount on **any** surface (tile, funnel, email, SMS)." Only the tile/title surface is now implemented and (indirectly) covered. Funnel, email and SMS surfaces are unasserted, and HI-8097's Braze payload restructure touches exactly that risk area.

### hi-application-srvc#1203 + external-actor-engagement-srvc#4443 + avro-funnel-lib#835 — prequalDecisions array in the Braze persona (HI-8097) — ALL OPEN

_Analyzed: 2026-09-08_

**Changes**: Replaces the single `hi_prequal_offer` Braze persona field with a `prequalDecisions` array carrying both GOLD_STAR and cross-sell offers. Three coordinated PRs: the avro schema (`ExternalActorEngagementPrequalDecision.avsc` + a change to `ExternalActorEngagementSetProfileCommand.avsc`), the eaes consumer, and the hi-application-srvc publisher.

**UT/IT**: **Done on all three.** hi-app: `PreQualificationCustomerEngagementServiceTest`, `PreQualificationLeadEventListenerIT`. eaes: `PrequalDecisionMapperTest`, `PrequalDecisionBuilderTest`, `BrazePersonaMapperTest`, `BrazePersonaFacadeTest`, `PrequalDecisionComputedEventHandlerIT`, `ExternalActorEngagementTrackHandlerIT`. avro-funnel-lib: `ExternalActorEngagementSetProfileCommandCodegenTest`.

**E2E**: None yet — **and existing E2E will break.** Branch tests `[78105]` and `[85077]` assert the current single-`hi_prequal_offer` persona shape. They must be updated in the same window these three PRs merge, or the cross-sell Braze suite goes red.

**Gaps**: [HIGH, scheduling] Cross-repo breaking change to a contract two E2E tests already assert. This is the clearest near-term action item to sequence.

### hi-application-srvc#1201 — Record CONTACT_GRANT_GIVEN from a clean transaction (HI-8085) — MERGED 2026-09-04

_Analyzed: 2026-09-08_

**Changes**: Bug fix. The `CONTACT_GRANT_GIVEN` marker was being written inside a transaction that could roll back, so the grant was silently lost.

**UT/IT**: **Done** — `LeadAccessRelationshipUpdaterTest`, `LeadAccessRelationshipUpdaterIT`, `LeadContactGrantMarkerIT`, plus `AbstractApplicationServerIT`/`AbstractApplicationServiceIT` harness changes.

**E2E**: None. No cross-sell E2E asserts the contact-grant marker or its lifecycle (grant on share → revoke on expiry, the latter being HI-8103's territory).

**Gaps**: [MEDIUM] A silently-lost authz marker is exactly the class of defect E2E catches and unit tests do not. Worth one test: share contact → assert grant marker present → expire → assert revoked.

### hi-application-srvc#1206 — Optimize preQualificationContactGrantRevocationJob (HI-8103) — OPEN

_Analyzed: 2026-09-08_

**UT/IT**: **Done** — `PreQualificationContactGrantRevocationJobTest`, `PreQualificationExpirationServiceTest` + `PreQualificationExpirationServiceIT`, `LeadContactGrantMarkerIT`.

**E2E**: None. Pairs with the HI-8085 gap above — the revocation half of the same lifecycle.

### hi-application-srvc#1189 / qa-automation#38091 — Remove the dead HomeImprovementPreQualifiedEvent path (HI-7918) — BOTH MERGED 2026-09-02

_Analyzed: 2026-09-08_

**Changes**: Coordinated cleanup on both sides. Was listed as "Open, no PRs" last refresh; resolved within two days.

**UT/IT**: **Done** — `PreQualificationEventHandlerTest`, `PreQualificationEventHandlerIT`, `PreQualificationMapperTest`, `Fixtures`.

**E2E**: `qa#38091` is the qa-automation half of the same cleanup — the only HI cross-sell PR merged to qa-automation master since the last refresh.

### hi-application-srvc#1098 — Null prequalDecisionUuid for on-demand prequal (HI-7909) — **CLOSED UNMERGED**

_Status corrected 2026-09-08_

Previously tracked as OPEN with IT-only coverage. The PR was **closed without merging**, while HI-7909 sits at "In Validation." `#1102` rewrote the same event handlers, so the behaviour was most likely absorbed there — but that is an inference, not a verified fact. Flagged under Watch.

_(Prior entries — hi-application-srvc#810, #1038, #1105, #1126; home-improvement-merchant-srvc#5777, #5904; loan-app-creation-srvc#9287; home-improvement-borrower-dashboard-ui#8/#9; qa-automation#37912 — carried forward unchanged from the 2026-08-21 and 2026-08-31 refreshes. See Confluence page history.)_

## Coverage Matrix

| Requirement | Ticket(s) | UT | IT | E2E | Status |
| --- | --- | --- | --- | --- | --- |
| Cross-sell pre-qual creation | HI-6534 | Y | Y | master (SkipUntil) + branch | COVERED* |
| Submit cross-sell pre-qual + decision (approved path) | HI-6535, HI-6540 | Y | Y | master (SkipUntil) + branch | COVERED* |
| Submit cross-sell pre-qual (declined path / AAN) | HI-6535, HI-7294 | -- | -- | branch `[85067]` (ineligible-copy guard only) | GAP (confirmed Won't Do — a scope decision, not a defect) |
| Share contact with merchant | HI-6536 | Y | Y | master (SkipUntil) + branch | COVERED* |
| Lead stage / lifecycle | HI-6538 | Y | Y | master + branch `[83196]` isolation | COVERED (branch) |
| Merchant suspension → lead hidden + restored | HI-6541 | Y | Y | master `[74449]` + branch `[83516]` Braze notice | COVERED (branch) |
| Serviceable zip codes + Google Places | HI-6469, HI-6470 | Y | Y | master (SkipUntil) | COVERED* |
| Merchant cross-sell priority config | HI-6659, HI-6930 | Y | Y | branch `[84304]` (set + negative rejected) | **COVERED (branch)** — was a 3-refresh standing GAP |
| Borrower ITA eligibility persistence | HI-6533 | Y | Y | master `[74461]`/`[74462]` (SkipUntil) | COVERED* (PL/PCL; Deposit/FlexPay still gap) |
| Borrower Braze notifications | HI-6540 | Y | Y | branch `[78105]`, `[85077]`, `[83513]`, `[83516]` | **COVERED (branch)** — 1 event → 4 |
| **Merchant Braze notifications** | HI-6541 | Y | Y | branch `[83514]`, `[83515]`, `[84335]` in the new `HomeImprovementCrossSellMerchantBrazeEventTest` | **COVERED (branch)** — was GAP since 2026-06-19 |
| Merchant lead-expiry reminder job | HI-6546 | Y | Y | branch `[84335]` (dashboard alert + Braze notice) | **COVERED (branch)** — new row |
| Merchant-initiated create-application flow | HI-6756 (Not Needed) | N/A | N/A | branch ×8 (`[83127]`, `[83128]`, `[83168]`, `[83194]`, `[83195]`, `[83197]`, `[83200]`, `[83443]`) | **COVERED (branch)** — master has zero |
| NBA/ITA config + definition copy | HI-6769 | Y | Y | branch `[82676]`, `[82677]`, `[84225]`, `[84226]`, `[84236]` | **COVERED (branch)**; definitions still `starts_at=2050-01-01` gated |
| Merchant reads borrower applicant (authz) | HI-8036 | Y | Y | transitive via `[85004]`/`[83285]`/`[84148]` | COVERED (allow path); deny path untested |
| Merchant Lead Details shows edited contact | HI-7758, HI-8025 | Y | Y | branch `[85004]`, `[83285]`, `[84148]` | **COVERED (branch)** — bug fixed by md-ui#899 |
| Pre-qualification income bounds ($0 allowed, $3M cap) | HI-8027 | N/A | N/A | branch `[85069]`, `[85078]` | **COVERED (branch)** |
| Reporting & funnel metrics | HI-6543, HI-7938 | Partial | Partial | GAP | GAP |
| **Omni: Branch A/B routing (O1)** | HI-7755 (W1), HI-7756 (W2), HI-6769 (nba#3418) — **not** HI-7754 | Y | Y | branch `[84225]`, `[84226]`, `[84236]`, `[85075]`, `[85068]` | **COVERED (branch)** — see the correction in Spec Summary |
| **Omni: Eligibility ≥3 merchants/150mi (O2/S26)** | Spec only | -- | -- | PARTIAL (V1 radius logic reused; the 3-vs-5 threshold is still not asserted) | PARTIAL |
| **Omni: Amount suppression on borrower surfaces (O3)** | HI-7762 (Won't Do), **HI-7765 (MERGED)** | -- | -- | Indirect (disclaimer-marker assertion) | **PARTIAL** — was GAP; title/banner shipped, other surfaces unasserted |
| **Omni: Repeat-customer category exclusion + Goldstar overlap (O4/O5)** | Spec only / S28 | -- | -- | `qa#35594` (MERGED, exemption logic); branch `findBorrowerAllowingCategory`/`excludedCategoryFor` use exclusion as setup, not as an assertion | PARTIAL |
| **Omni: Monthly batch decisioning + prequal fields (O6/O7)** | HI-7755, HI-7756 (both **MERGED**) | Y | Y | `qa#37553`, `qa#37758`, `qa#37576` + branch `[82282]`, `[82283]`, `[82292]`, `[82293]` | COVERED (branch); per-customer anniversary cadence (O31) still unasserted |
| **Omni: Decline suppression / 90-day cooldown** | HI-7754 (**MERGED**) | Y | Y | branch `[85068]`, `[85070]`, `[85071]` | **COVERED (branch)** — was HIGH GAP |
| **Omni: Un-suppression cadence (O29)** | HI-7754 | Y | Y | branch `[85070]` (later approval ends the cooldown pre-activation) | **COVERED (branch)** — was GAP |
| **Omni: 45-day expiry / 30-day contradiction (O8/S29)** | HI-7759 (Done), HI-8024 (Blocked) | -- | -- | branch `[83197]`, `[84387]`, `[83443]` cover expiry propagation and post-expiry behaviour; the two-window question itself is untested | PARTIAL — needs product clarification first |
| **Omni: Sub-cohort treatment, 4 states (O9)** | HI-7755/HI-7756 emergent | Y | Y | branch: (a) `[84226]`, (b) `[85075]`/`[85070]`, (c) `[85068]`, (d) `[84225]` | **COVERED (branch)** — was SPEC GAP |
| **Omni: Latest-decision-wins (O20)** | HI-7755 | Y | Y | branch `[85073]` covers (a) approve→approve supersession + replay idempotency; (b) approve→decline and (c) approve→no-refresh untested | **PARTIAL** — was total GAP |
| **Omni: Batch backs off on in-flight on-demand prequal** | HI-7755 | Y | Y | branch `[85074]` | **COVERED (branch)** — new row |
| **Omni: Gold Star / cross-sell prequal coexistence** | HI-7755 | Y | Y | branch `[82293]`, `[85076]` (both directions) | **COVERED (branch)** — new row |
| **Omni: Null-score-vs-decline fairness (O10)** | Spec open question, no ticket | -- | -- | SPEC GAP | GAP |
| **Omni: Offer rounding rule (O13)** | Spec only | -- | -- | SPEC GAP | GAP |
| **Omni: Score-gate cutoff logic (O14)** | Spec only | -- | -- | SPEC GAP | GAP |
| **Omni: prequal-id selection at application (O17)** | HI-7757 (W3), HI-7758 (W_SHARE) | Y (UT) | Partial | branch `[83200]` asserts a batch-originated lead converts identically to an on-demand one; explicit Goldstar-vs-cross-sell id selection untested | PARTIAL |
| **Omni: Re-decision against locked policy version (O18)** | **Unowned** — dropped from HI-7754; HI-8098 is the likely future home | -- | -- | SPEC GAP | GAP — still unowned |
| **Omni: Restricted decline-reason set at re-decision (O23)** | Spec only, no ticket | -- | -- | SPEC GAP | GAP |
| **Omni: PL cross-sell fallback on Explore Contractors (O26)** | HI-7767 (In Development, **PR #29 open**) | -- | -- | SPEC GAP | GAP — now in flight |
| **Omni: Repeat-customer merchant reporting tab (O28)** | Spec only, no ticket | -- | -- | SPEC GAP | GAP |
| **State disclosures on prequal agreements (NEW)** | HI-8093 (Blocked), HI-8095 (Open) | -- | -- | SPEC GAP | **GAP [HIGH]** — new legal/compliance scope this refresh |
| **Max-selection block at 6th contractor (NEW)** | HI-8106 (In Development) | -- | -- | GAP | GAP — the 5-merchant/3-per-category rule has never been asserted |
| **projectNotes validation (2000 chars, valid text) (NEW)** | HI-8094 (Ready for CodeReview) | -- | -- | GAP | GAP — same test pattern as the covered HI-8027 income bounds |
| **Braze persona `prequalDecisions` array (NEW)** | HI-8097 (3 PRs open) | Y | Y | **Will break `[78105]`/`[85077]`** | GAP + scheduling risk |

## Spec Requirement Gaps

**Both specs are unchanged since the last refresh** — the V1 spec (`PROD/4494065719`) is still at v26 (2026-08-05) and the Omni spec (`PROD/5856264352`) is still at v13 (2026-07-29). No new S- or O-requirements were introduced by PM this cycle. What changed is coverage, and the three *new* requirement areas below came in through Jira tickets rather than spec edits.

### New requirement areas introduced by tickets, not by spec edits (2026-09-08)

| # | Requirement | Source | Priority | E2E Status |
| --- | --- | --- | --- | --- |
| N1 | State-specific disclosures must appear on the cross-sell pre-qualification agreements | HI-8093 (FE, Blocked), HI-8095 (BE, Open) | **HIGH** | SPEC GAP — legal/compliance, neither half started. The V1 spec's state-disclosure language (CA vs VT) was never extended to the cross-sell prequal consent screen |
| N2 | The 6th contractor selection must be blocked with a max-selection screen | HI-8106 (In Development) | MEDIUM | GAP — the "max 5 merchants, max 3 per category" rule is stated in the V1 spec but has never had an E2E asserting the block |
| N3 | `projectNotes` must be validated at 2000 chars and for valid text on connect-with-contractor | HI-8094 (Ready for CodeReview) | MEDIUM | GAP — structurally identical to HI-8027's income bounds, which the branch covers with `[85069]`/`[85078]`; the same pattern transfers directly |
| N4 | Braze persona must carry a `prequalDecisions` array spanning GOLD_STAR and cross-sell, replacing the single `hi_prequal_offer` field | HI-8097 (3 PRs open) | **HIGH (scheduling)** | GAP — and a breaking change to a contract that branch tests `[78105]`/`[85077]` already assert |
| N5 | `CONTACT_GRANT_GIVEN` must survive transaction rollback; contact grants must be revoked when the prequal expires | HI-8085 (Fixed), HI-8103 (in review) | MEDIUM | GAP — no E2E asserts the grant marker's lifecycle in either direction |

### Original spec (PROD/4494065719) — S1-S29

| # | Requirement | Priority | E2E Status |
| --- | --- | --- | --- |
| S1-S25 | _(unchanged — see Confluence page history for full text)_ | -- | V1 E2E exists on master but is SkipUntil-disabled; the branch removes that gate |
| S26 | Zip-match eligibility threshold lowered from 5 to **3** merchants within 150mi | MEDIUM | SPEC GAP — still not specifically asserted |
| S27 | Cease & Desist borrowers excluded from **NBA only**; still receive email/SMS | LOW | SPEC GAP |
| S28 | HI+Goldstar customers **no longer excluded** from cross-sell eligibility | MEDIUM | PARTIAL — decisioning exemption tested (`qa#35594`); UI category-display still not asserted as behaviour |
| S29 | **Spec self-contradiction**: Decisioning says the offer is valid **45 days**; Product Flow / Pre-qualified Page still says **30 days** | **HIGH** | Needs product clarification. Now visible in two more places: HI-8024 (merchant tab disclaimer still says 30 days) and the branch's own `CROSS_SELL_EXPIRATION_DAYS` constant, which has to pick one |

### Omni spec (PROD/5856264352) — O1-O35, status as of this refresh

Requirement text is unchanged from the last refresh; only the E2E column has moved. Summarised deltas:

| # | Prior status (2026-08-31) | Status now | Why |
| --- | --- | --- | --- |
| O1 | SPEC GAP — "unowned, not just unstarted" | **COVERED (branch)** | Implemented by W1+W2 (`hi-app#1102`, MERGED) + `nba#3418` (MERGED), not by W7. Proven by `[84225]`/`[84226]`/`[84236]`/`[85075]`/`[85068]` |
| O3 | SPEC GAP (HIGH) | **PARTIAL** | HI-7765 shipped (`hibdui#25`, MERGED 2026-09-03) — title/banner only |
| O6/O7 | PARTIAL | COVERED (branch) | BE merged; 4 branch tests on batch creation/activation/provisioning |
| O8 | PARTIAL | PARTIAL | Expiry behaviour covered; the 30-vs-45 contradiction still blocks a correct assertion |
| O9 | SPEC GAP (HIGH) — "no ticket owns this" | **COVERED (branch)** | All four sub-cohorts now have a test each |
| O17 | PARTIAL | PARTIAL | `[83200]` adds batch-lead parity; explicit Goldstar-vs-cross-sell id selection still untested |
| O18 | SPEC GAP (HIGH) — unowned | **SPEC GAP — still unowned** | Genuinely dropped from W7. HI-8098 ("use prequal-decision V2 API in NBA") is the plausible future home, but does not claim this scope today |
| O20 | SPEC GAP (HIGH) | **PARTIAL** | `[85073]` covers approve→approve supersession + replay idempotency; approve→decline and approve→no-refresh remain untested |
| O24 | SPEC GAP | **COVERED (branch)** | `[84226]`/`[84236]` assert the exact spec copy ("You're pre-qualified for a home improvement loan" / "Browse contractors") on all declared placements |
| O26 | SPEC GAP | SPEC GAP — in flight | HI-7767 now has `hibdui#29` open |
| O29 | SPEC GAP | **COVERED (branch)** | `[85070]` — a later approval ends the cooldown before activation, so one bad month does not lock a borrower out |
| O2, O4, O5, O10, O12-O16, O19, O21-O23, O25, O27, O28, O30, O31 | unchanged | unchanged | No implementation or coverage movement |
| O11, O32-O35 | Informational / not independently testable | unchanged | -- |

## Active Gaps

### Confirmed non-gaps (deliberately deprioritized — do not chase these as bugs)

1. **HI-7294 [BE] Handle AAN** — resolution "Won't Do." The declined/AAN path (S3) was never built by product decision.
2. **HI-7762 [Omni][BE] W_AMT** — resolution "Won't Do." Amount suppression was delivered instead by **HI-7765, which is now merged**. O3 is no longer attributable to either as a gap; what remains is surface coverage breadth.
3. **HI-6537, HI-6539** — both closed as Duplicate. Not implementation gaps.

### Critical / HIGH

1. **[CRITICAL, OPERATIONAL]** The `HI-CrossSellDirectoryPLTests` branch is **still not a PR**, and it is now carrying the epic's entire coverage story: 67 tests vs master's 27, 39 net-new AllureIds, 3 new classes, and the removal of every `@SkipUntil`. Nine separate gaps in this document read "COVERED (branch)" and are invisible to CI, to reviewers, and to anyone else on the team. **Opening this PR is the single highest-leverage action in the epic** and should precede any further test writing. HI-8075 (Setup confidence score and CI gate for new cross sell services) is the natural home for the CI-gating half.
2. **[HIGH, SCHEDULING]** **HI-8097 is a breaking change to a contract two branch tests already assert.** `hi-application-srvc#1203` + `external-actor-engagement-srvc#4443` + `avro-funnel-lib#835` replace the single `hi_prequal_offer` Braze field with a `prequalDecisions` array. Tests `[78105]` and `[85077]` will fail on merge. Sequence the E2E update into the same window.
3. **[HIGH, NEW]** **State disclosures (N1) — HI-8093 Blocked, HI-8095 Open, neither started.** Legal/compliance requirement on the prequal agreements screen with no implementation and no coverage. Highest-risk *unstarted* item in the epic now that the Omni core has landed.
4. **[HIGH]** **O18 — re-decision against a locked policy version is still unowned.** This is the one piece of the 2026-08-31 "W7 rescope" finding that survives scrutiny: it was genuinely dropped, and no ticket claims it. HI-8098 (prequal-decision V2 API in NBA) is the plausible future home — worth explicitly asking whether it absorbs O18 rather than waiting to find out.
5. **[HIGH]** **S29 — the 30-vs-45-day offer-validity contradiction is still unresolved** and now blocks three things instead of one: the spec's own Product Flow section, HI-8024's merchant tab disclaimer, and the branch's `CROSS_SELL_EXPIRATION_DAYS` constant. Needs a product decision before any of the three can be made correct.
6. **[HIGH]** **O14 — score-gate cutoff logic (HIRM1/IR5/EDQHIRM1)** untested at E2E. **O10 — null-score-vs-true-decline fairness** remains an open compliance question with no ticket. Both sit inside the decisioning layer that the newly-merged batch consumer now depends on.
7. **[HIGH]** **O23 — restricted decline-reason enumeration at re-decision** (anti bait-and-switch) has zero coverage.
8. **[HIGH]** **O20 is only one-third covered.** `[85073]` proves approve→approve supersession. Approve→**decline** (prior offer must go stale/hidden) and approve→**no-refresh** (original offer survives to its own expiry) are both untested, and the decline branch is the one with borrower-facing marketing consequences.
9. **[CARRIED OVER, HIGH]** **S1 — borrower eligibility by source product.** PL and PCL now have branch coverage (`[82676]`, `[82677]`); **Deposit, FlexPay and existing-HI remain untested.**

### Medium Priority

1. **[MEDIUM, NEW]** No E2E asserts the **contact-grant marker lifecycle** (N5) — grant on share, revoke on expiry — despite a production bug (HI-8085) and an optimization PR (HI-8103) both landing in this area within a week.
2. **[MEDIUM, NEW]** **Deny-path authz is untested.** HI-8036's chain grants a merchant `read_applicant` on a shared lead; only the allow path is exercised. A merchant *without* a shared lead should still 403, and nothing asserts it.
3. **[MEDIUM, NEW]** **N2 — the max-5-merchants / max-3-per-category selection block** (HI-8106) has never had an E2E, on master or branch.
4. **[MEDIUM, NEW]** **N3 — projectNotes validation** (HI-8094) is unasserted; the covered HI-8027 income-bounds tests are a direct template.
5. **[MEDIUM]** **O3 breadth** — amount suppression now ships on the explore-contractors title/banner, but the spec's claim is "no borrower surface at all." Funnel, email and SMS are unasserted, and HI-8097 is actively reshaping the email/SMS payload.
6. **[MEDIUM]** **W5 dark-launch flag (`hi-app#1126`) is still open** with no E2E asserting flag-off vs flag-on behaviour of the now-merged batch consumer.
7. **[MEDIUM]** S26/O2 — the 5→3 merchant-count eligibility threshold is still not asserted.
8. **[MEDIUM]** O4/S28 — repeat-customer category exclusion is used by the branch as *setup* (`findBorrowerAllowingCategory`, `excludedCategoryFor`) but never asserted as behaviour under test. The scaffolding to assert it already exists.
9. **[MEDIUM]** O12/O13 — default policy inputs and the offer rounding rule untested.
10. **[MEDIUM]** O16, O21 — spec itself still marks these "to confirm."
11. **[MEDIUM]** O25 — resume-application NBA ("Don't let your project stall") untested; O24's sibling, and the NBA test scaffolding on the branch would extend to it cheaply.
12. **[MEDIUM]** O26 — PL cross-sell fallback, now in flight as `hibdui#29`.
13. **[MEDIUM]** O28, O29-adjacent reporting — repeat-customer merchant reporting tab has no ticket.
14. **[MEDIUM]** O31 — per-customer monthly-anniversary batch cadence has direct fixture implications and is unasserted.
15. **[CARRIED OVER, MEDIUM]** HI-7441 — CDS marketing-segment forwarding still has no IT and no E2E.
16. **[CARRIED OVER, MEDIUM]** HI-7757 (W3) — prequalDecisionUuid to CDS is UT-only; this is the O17/O18 handoff wiring.
17. **[MEDIUM]** HI-6543 / HI-7938 — funnel metrics and contact-source-aware reporting, both still GAP.

### Lower Priority

1. **[LOW]** S27 — Cease & Desist NBA-only exclusion untested.
2. **[LOW]** O11, O15, O19, O22, O27, O30 — see tables above.
3. **[LOW]** HI-7704 — defense-in-depth, UT only, low business risk.
4. **[LOW]** HI-6743/HI-6720 — Google reviews and featured-merchant ordering are asserted at the API layer only.
5. **[LOW]** HI-7243 — Heap analytics events are unit-tested in the FE repo and are not E2E-observable from this framework.

### Informational (confirmed, no test action needed)

- O32-O35 — explicit negative confirmations from the Omni spec's Questions section.

### New / Watch

1. **[WATCH]** **HI-7909 is "In Validation" but its only PR (`hi-app#1098`) was closed unmerged.** `#1102` rewrote the same handlers and probably absorbed the behaviour — confirm rather than assume.
2. **[WATCH]** **HI-7802 flipped Blocked → Closed (Done) with no PR in the tracked repo set** and no E2E. The Omni pre-fill application screen is claimed done; nothing verifies it.
3. **[WATCH]** **HI-7766 is Resolved while `hibdui#26` is still open**; **HI-7505 is Resolved while `hibdui#12` is still open** — the latter for a second consecutive refresh.
4. **[WATCH]** **`nba#3418`'s ITA/banner definitions ship gated at `starts_at=2050-01-01`.** The branch E2E activates them to run. Green tests here do **not** mean the ITAs are live for real borrowers — someone owns flipping that date.
5. **[WATCH]** HI-7761 — resolution "Self-Resolved," outside the tracked repo set, never independently verified.

### Changes Since Last Refresh (2026-08-31 → 2026-09-08)

* **14 tickets added to the Ticket Map**: 10 genuinely new (HI-7927, HI-8075, HI-8085, HI-8093, HI-8094, HI-8095, HI-8097, HI-8098, HI-8103, HI-8106) and 4 that existed but had never been given a row despite appearing in the Coverage Matrix (HI-6533, HI-6540, HI-6541, HI-6546). Epic is now 93 child tickets.
* **The Omni core landed.** `hi-application-srvc#1102` (W1 batch consumer + W2 activation rendezvous) **merged 2026-09-08**, the day of this refresh. It carries 23 test files with both UT and IT.
* **The 2026-08-31 "O1 is unowned and unimplemented" finding is corrected.** Branch A/B routing is implemented — it emerged from W1+W2 plus `next-best-action-srvc#3418` (MERGED 2026-09-02), not from W7. The branch proves all four O9 sub-cohorts end to end. What W7 genuinely dropped, and what remains unowned, is **O18 alone**.
* **The merchant Lead Details contact bug reported last refresh is fixed**: `merchant-dashboard-ui#899` (MERGED 2026-09-02) makes `UPGRADE_LEAD` leads read the applicant version instead of `Actor.profile`. HI-8025 moved Blocked → Resolved.
* **The 3-repo merchant applicant-read authz chain is complete**: `spicedb-schemas#1899`, `applicant-srvc#2353` and `hi-application-srvc#1178` all merged 2026-09-01/02. HI-8036 moved Open → Closed. The "Missing privilege READ_APPLICANT" 403 is resolved.
* **O3 (amount suppression) shipped** via `home-improvement-borrower-dashboard-ui#25` (MERGED 2026-09-03). HI-7765 moved Blocked → Closed — it had no PRs at all a week ago.
* **HI-8027 (income $0..$3M bounds) went Open → shipped → covered inside one cycle**: `hibdui#30` merged 2026-09-08, with branch tests `[85069]`/`[85078]` already written against it.
* **HI-7918 dead-code cleanup completed on both sides** (`hi-app#1189` + `qa#38091`, both merged 2026-09-02) — it was "Open, no PRs" last refresh.
* **A real production bug was found and fixed inside the cycle**: HI-8085, `CONTACT_GRANT_GIVEN` lost when written inside a rolled-back transaction (`hi-app#1201`, merged 2026-09-04).
* **The local `HI-CrossSellDirectoryPLTests` branch roughly quintupled**: from the 12 tests reported last refresh to **67 `@Test` methods across 13 classes — 39 net-new AllureIds and 3 net-new classes** (`HomeImprovementCrossSellDeclineCooldownTest`, `HomeImprovementCrossSellMerchantBrazeEventTest`, plus `HomeImprovementCrossSellBrazeEventTest` relocated into the `crosssell` package and extended). **Every `@SkipUntil` in the package has been removed** — master still has 36 occurrences across 9 files. New coverage closes: decline cooldown + un-suppression (HI-7754, was HIGH GAP), merchant Braze notifications (HI-6541, GAP since June), lead-expiry reminders (HI-6546), merchant cross-sell priority (HI-6659, GAP for 3 refreshes), the merchant create-application flow (HI-6756, 8 tests, master has zero), both NBA/ITA definition variants and their flip (HI-6769), batch supersession/backoff/Gold-Star-coexistence (HI-7755/HI-7756), and income validation bounds (HI-8027).
* **Neither spec changed.** V1 still v26 (2026-08-05), Omni still v13 (2026-07-29). All requirement movement this refresh is implementation and coverage, not scope.
* **New scope arrived through tickets rather than spec edits** (N1-N5): state disclosures on prequal agreements (HI-8093/HI-8095, HIGH, unstarted), the 6th-contractor max-selection block (HI-8106), projectNotes validation (HI-8094), the Braze `prequalDecisions` array restructure (HI-8097, breaking), and contact-grant lifecycle correctness (HI-8085/HI-8103).
* **One PR regressed in status**: `hi-application-srvc#1098` (HI-7909) was closed unmerged while its ticket remains In Validation.

### Changes Since Last Refresh (2026-08-21 → 2026-08-31)

_(Carried forward — see Confluence page history for the full entry.)_ W7/HI-7754 rescope discovered; Omni workstreams advanced; `qa#37758` and `qa#37912` merged; the merchant Lead Details Actor-vs-Applicant contact bug was found by direct FE source inspection; 9 new child tickets appeared (HI-7918, HI-7927, HI-7928, HI-7938, HI-8024, HI-8025, HI-8026, HI-8027, HI-8036).

### Changes Since Last Refresh (2026-06-19 → 2026-08-21)

_(Carried forward — see Confluence page history for the full entry.)_ Epic grew 54 → 75 tickets; V1 shipped via `qa#34134` with a blanket `@SkipUntil`; the borrower FE migrated to the standalone `home-improvement-borrower-dashboard-ui` repo; the Omni Pre-Qual spec was read in full and distilled into O1-O35; the primary spec picked up S26-S29.

## Deployment — Feature Flags & Config (E2E stack)

_(V1 flag table carried forward from the 2026-06-19 refresh — see Confluence history.)_

**Updated 2026-09-08:**
* On `qa-automation` **master**, the V1 cross-sell suite remains `@SkipUntil`-gated off on main/stage/preprod until 2050-12-31 regardless of flag state — flags alone will not make these tests run. **The `HI-CrossSellDirectoryPLTests` branch removes the gate entirely.**
* **`next-best-action-srvc#3418`'s ITA/banner definitions ship with `starts_at=2050-01-01`.** The branch E2E activates them programmatically in order to run. They are not live for real borrowers until that date is changed.
* **`hi-application-srvc#1126` (W5 dark-launch flag) is still open**, so the newly-merged batch cross-sell prequal consumer from `#1102` is not yet flag-gated in the way W5 intends.
* `k8s-template#285334` (MERGED) schedules the lead-expiry reminder job and enables cross-sell notices in non-prod — the config the branch's `[84335]` expiring-lead test depends on.

## Decisions

_(2026-04-16 through 2026-08-31 decisions carried forward — see Confluence page history for the full list.)_

* 2026-07-09/07-16: New standalone repo `home-improvement-borrower-dashboard-ui` created; the entire borrower-side cross-sell FE migrated there.
* 2026-07-22: Omni Pre-Qual spec (`PROD/5856264352`) added — flips V1's on-demand prequal to an upfront monthly-batch model reusing Goldstar's decisioning machinery.
* 2026-07-24: `qa-automation#34134` merged — V1 E2E code-complete but shipped with a blanket `@SkipUntil` disabling all tests on main/stage/preprod until 2050.
* 2026-08-21: HI-7294 and HI-7762 confirmed "Won't Do." W7/HI-7754 rescoped away from Branch-A/B gating to local decline-suppression only.
* 2026-08-31: Merchant Lead Details found to source contact from `Actor.profile` rather than the `Applicant` record HI-7758 writes to.
* **2026-09-01/02: The merchant applicant-read authz chain landed in sequence** — `spicedb-schemas#1899` introduced a dedicated `read_applicant` permission (deliberately *not* reusing the existing `read`, which ~15 unrelated services already hold), `applicant-srvc#2353` enforced it, `hi-application-srvc#1178` granted it on lead share.
* **2026-09-02: `merchant-dashboard-ui#899` resolved the Actor-vs-Applicant contact-source bug** for `UPGRADE_LEAD` leads. The broader HI-7928 (borrower form) and HI-7938 (reporting) halves of the same migration are still in progress.
* **2026-09-03: O3 amount suppression shipped** on the borrower explore-contractors title/banner via `hibdui#25`, superseding the "Won't Do" HI-7762.
* **2026-09-08: `hi-application-srvc#1102` merged**, delivering the Omni batch consumer and activation rendezvous. Combined with `nba#3418`, this makes Branch A/B routing real — correcting the 2026-08-31 conclusion that it was unimplemented. Only O18 remains genuinely dropped and unowned.
* **2026-09-08 (QA decision, pending PR): the `HI-CrossSellDirectoryPLTests` branch removes every `@SkipUntil` from the cross-sell package**, reversing the 2026-07-24 decision to ship V1 E2E disabled. This is the right call now that the Omni core has merged — but it is unreviewed and unmerged, and until the PR is opened the epic's coverage story exists only on one machine.
