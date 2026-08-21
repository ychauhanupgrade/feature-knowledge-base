---
epic: HI-6091
title: Cross Sell
spec_url: https://credify.atlassian.net/wiki/spaces/PROD/pages/4494065719
spec_url_2: https://credify.atlassian.net/wiki/spaces/PROD/pages/5856264352
status: in-development
last_refreshed: 2026-08-21
test_checklist_ticket: HI-6478
confluence_page_id: "5605752862"
---

# Cross Sell

## Spec Summary

The Cross Sell program offers existing Upgrade customers (PL, PCL, Deposit, HI, FlexPay) a Home Improvement loan through the merchant network. Eligible borrowers see an NBA/ITA banner on their dashboard (Directory, HI Home, Manage Payments pages), explore a contractor network filtered by zip code (configurable max radius, up to 150 miles system-wide; FE slider up to 50 miles), select up to 5 merchants (max 3 per category), complete a pre-qualification form (soft credit pull, pre-populated PI1), and share contact details with selected merchants. Merchants receive leads in an "Upgrade Leads" tab, can view lead details, update lead stages, and initiate loan applications directly. Merchant eligibility requires `cross_sell_enabled`, serviceable zip codes, and optionally a Google Places ID.

**V1 is now fully built and merged** (`qa-automation#34134` merged to master 2026-07-24; borrower-side FE migrated to a brand-new standalone repo, `home-improvement-borrower-dashboard-ui`, created 2026-06-24). **However, the entire V1 cross-sell E2E suite (9 test classes / 22 `@Test` methods under `com.upgrade.tests.regression.homeimprovement.crosssell`, plus `HomeImprovementCrossSellBrazeEventTest`) carries `@SkipUntil(envToSkip = {"main","stage","preprod"}, skipBefore = "2050-12-31", reason = "More changes in cross sell are planned from business")` on every single `@Test` method — confirmed by direct grep of master.** This means none of these tests currently execute in CI on the three key environments; "COVERED (master)" in this document means "code exists and is wired into `home-improvement-cross-sell-tests.xml`," not "passing in CI." The only cross-sell E2E tests that run unconditionally today live in the decisioning layer: `PrequalDecisionHiclCrossSellTest` and `PrequaDecisionlHiclGoldstarTest` (no `@SkipUntil`).

**New this refresh — "Omni Pre-Qual" initiative (epic CRD-19822, tracked as `[X-Sell Omni Prequal][BE]` sub-tickets under HI-6091):** a second, newly-added spec (`PROD/5856264352`, created 2026-07-22) flips V1's on-demand-only pre-qualification model. Instead of running the prequal only when a borrower clicks through, CDS now generates a generic HI cross-sell prequal upfront during the existing monthly Goldstar-style bureau refresh for consenting, eligible customers. Borrowers split into **Branch A** (valid non-expired APPROVED omni prequal exists → shown a "you're pre-qualified" tile, no dollar amount) or **Branch B** (no valid prequal → on-demand ITA at lower priority, captures consent, graduates into the next month's batch). This is a workstream-based build-out: W0 (avro-decisioning-lib bump), W1 (batch consumer routing), W2 (activation rendezvous APPROVED→ACTIVE), W3 (send prequalDecisionUuid to CDS), W5 (tests/flag/observability), W7 (prequal-decision-srvc client + Branch-A/B gating), W_AMT (amount suppression), W_EXP (45-day expiry), W_SHARE (editable contact at share step). **W7 and W2 — the architectural core that actually implements the Branch A/B split and activation state machine — have not been started** (HI-7754, HI-7756 both status Open, no PRs).

## Ticket Map

| Ticket | Title | Type | Status | PR(s) | UT/IT | E2E | Gap |
| --- | --- | --- | --- | --- | --- | --- | --- |
| HI-6395 | [BE] Design | Story | Closed | -- | -- | -- | Design-only |
| HI-6396 | [FE][Borrower] Add NBA banner image on BD | Story | Closed | bd-ui#7082 (superseded), bd-ui#7826 (MERGED, asset re-add) | N/A (FE) | COVERED* | Interactive banner now lives in new-repo#9 |
| HI-6469 | [BE] Google Places API Integration | Story | Closed | hi-merchant#5436 (MERGED) | Done | COVERED* | -- |
| HI-6470 | [BE] Merchant serviceable zip code list support | Story | Closed | hi-merchant#5436 (MERGED); hi-merchant#5649 (CLOSED, no tests, abandoned) | Done | COVERED* | -- |
| HI-6494 | [FE][Borrower] Contractors exploration page + list | Story | Closed | bd-ui#7151 (superseded), new-repo#9 (MERGED) | N/A (FE) | COVERED* | FE now in new repo |
| HI-6495 | [FE][Borrower] Filter section for contractors exploration | Story | Closed | bd-ui#7121 (superseded), new-repo#9 | N/A (FE) | COVERED* | FE now in new repo |
| HI-6496 | [FE][Borrower] Contractors listing API | Story | Closed | bd-ui#7277 (superseded), new-repo#9 | N/A (FE) | COVERED* | FE now in new repo |
| HI-6497 | [FE][Borrower] Pre-qualification page + form/modal | Story | Closed | bd-ui#7194 (superseded), new-repo#9; qa#37388 (MERGED, nav fix) | N/A (FE) | COVERED* | FE now in new repo |
| HI-6498 | [FE][Borrower] Pre-qualification success page | Story | Closed | bd-ui#7202 (superseded), new-repo#9 | N/A (FE) | COVERED* | FE now in new repo |
| HI-6499 | [FE][Borrower] Info sent page | Story | Closed | -- | N/A | N/A | -- |
| HI-6506 | [BE] Cross Sell pre-qual lead design | Story | Closed | spicedb#1235 (MERGED) | N/A (schema) | -- | -- |
| HI-6534 | [BE] Start cross sell pre-qual | Story | Closed | hi-app#570 (MERGED) | Done | COVERED* | -- |
| HI-6535 | [BE] Submit cross sell pre-qual | Story | Closed | hi-app#570 (MERGED) | Done | COVERED* (approved path); decline path S3 GAP | Decline/AAN path never delivered — see HI-7294 |
| HI-6536 | [BE] Share contact with merchant | Story | Closed | hi-app#570 (MERGED) | Done | COVERED* | -- |
| HI-6537 | [BE] Merchant user lead query | Story | Closed | -- | -- | -- | No PRs found |
| HI-6538 | [BE] Merchant manage pre-qual lead stage | Story | Closed | hi-app#570 (MERGED) | Done | COVERED* | -- |
| HI-6539 | [BE] Borrower Eligibility management | Story | Closed | -- | -- | -- | No PRs found |
| HI-6542 | [BE] Parent Portal (Phase 1) | Story | Closed | upflow2#363 (MERGED) | N/A (infra) | GAP | -- |
| HI-6543 | [BE][P2] Reporting & Metrics - Funnel Metrics | Story | Open | hi-app#570 (MERGED, partial) | Partial | GAP | Scope still open |
| HI-6544 | [BE] VQ Application Lookup | Story | Closed | -- | -- | -- | -- |
| HI-6631 | [FE] Move ContactDetailsCard to URC | Story | Closed | -- | N/A | N/A | -- |
| HI-6642 | [FE][Borrower] Share contact page | Story | Closed | bd-ui#7203 (superseded), new-repo#9 | N/A (FE) | COVERED* | -- |
| HI-6647 | [FE][Borrower] Featured merchant badge/sorting | Story | Closed | bd-ui#7208 (DECLINED) | N/A (FE) | GAP | Re-implemented under HI-6720 |
| HI-6659 | [BE] CrossSell priority config for merchant | Story | Closed | hi-merchant#5102 (MERGED) | Done | GAP | Still no E2E |
| HI-6720 | [FE][Borrower] Featured merchant ordering API | Story | Closed | bd-ui#7494 (superseded), new-repo#9 | N/A (FE) | PARTIAL | Force-ranking/badge assertions not confirmed this refresh |
| HI-6743 | [FE][Borrower] Google reviews API integration | Story | Closed | bd-ui#7281 (superseded), new-repo#9 | N/A (FE) | GAP | -- |
| HI-6745 | [FE][Borrower] Modify mobile filters | Story | Closed | bd-ui#7420 (superseded), new-repo#9 | N/A (FE) | GAP | -- |
| HI-6750 | [FE][Borrower] BE API at sharing contact flow | Story | Closed | bd-ui#7395 (superseded), new-repo#9 | N/A (FE) | COVERED* | -- |
| HI-6751 | [FE][Borrower] BE API at pre-qualification | Story | Closed | bd-ui#7395 (superseded), new-repo#9 | N/A (FE) | COVERED* | -- |
| HI-6754 | [FE][Merchant] List Upgrade Leads on homepage | Story | Closed | md-ui#747/748/751 (MERGED) | N/A (FE) | COVERED* | -- |
| HI-6755 | [FE][Merchant] Lead Details page for Upgrade Leads | Story | Closed | md-ui#748/749 (MERGED) | N/A (FE) | COVERED* | -- |
| HI-6756 | [FE][Merchant] Create Application page for leads | Story | Closed | -- | N/A | GAP | No PRs |
| HI-6766 | [FE][CCP] Cross Sell config in Merchant Features | Story | Closed | abp-ui#3622/3630 (MERGED) | N/A (FE) | COVERED* | -- |
| HI-6769 | [BE] NBA configuration for Cross Sell | Story | In Validation | hi-app#634 (MERGED), nba#3398 (MERGED), nba#3418 (OPEN) | Done | GAP | NBA delivery E2E not found |
| HI-6770 | [FE][CCP] Borrower Servicing Zip Code | Story | Closed | abp-ui#3630/3632 (MERGED) | N/A (FE) | COVERED* | -- |
| HI-6771 | [FE][CCP] Places ID for Google Reviews | Story | Closed | abp-ui#3630/3631 (MERGED) | N/A (FE) | COVERED* | -- |
| HI-6828 | [FE][MD] Updates required by Design/Product | Task | Closed | md-ui#756 (MERGED) | N/A (FE) | N/A | -- |
| HI-6887 | [FE][Parent] Updates to support Cross Sell | Story | Closed | mpd-ui#163/166 (MERGED) | N/A (FE) | COVERED* | -- |
| HI-6930 | [FE][CCP] Split Cross Sell eligibility/priority configs | Task | Closed | abp-ui#3747 (MERGED) | N/A | GAP | -- |
| HI-7004 | [FE][CCP] Updates for Servicing Zip Codes config | Task | Closed | abp-ui#3766 (MERGED) | N/A | GAP | -- |
| HI-7012 | [FE][MD] Conditionally display Pre-qual features in Reporting | Task | Closed | mpd-ui#165 (MERGED) | N/A | GAP | -- |
| HI-7017 | [FE][CCP] Display Zip/Places configs if eligibleForCrossSell | Task | Closed | abp-ui#3746 (MERGED) | N/A | GAP | -- |
| HI-7036 | [FE] Cleanups (post-deployment) | Task | Open | -- | N/A (FE) | GAP | Still no PRs |
| HI-7077 | [FE][Borrower] NBA and Resumption tiles on explore contractors | Story | Closed | No dedicated PR found — Jira resolution confirms **Done**; folded into new-repo#9 rollup | N/A (FE) | GAP | Delivered but not independently verified/tested |
| HI-7158 | [BE] Sign Agreements when submitting cross sell | Story | Closed | hi-app#570 (MERGED) | Done | PARTIAL | Consent display asserted; full signing-on-submit not independently re-verified |
| HI-7223 | [BE] Add PreQualificationContact under Lead | Story | Closed | hi-app#570 (MERGED) | Done (UT) | GAP | -- |
| HI-7243 | Heap analytics trackings | Story | Closed | new-repo#9 (`heap-events.test.js`, MERGED) | N/A (FE, unit-tested) | GAP | No E2E |
| HI-7294 | [BE] Handle AAN (Adverse Action Notice) | Story | Closed | **None — Jira resolution = "Won't Do"** | N/A | N/A (deliberately not built) | **Not a gap to fix — confirmed deprioritized.** Declined/AAN path (spec S3) remains genuinely unimplemented by product decision |
| HI-7369 | Improvement on google photo delivery | Story | Closed | hi-merchant#5631 (MERGED) | Done | GAP | -- |
| HI-7397 | Add idempotency-lib | Story | Closed | hi-application-srvc#810 (MERGED) | Done (`PreQualificationMutationResolverTest`, `CrossSellIdempotencyIT`) | COVERED* (idempotency test [55002] on master) | -- |
| HI-7411 | [FE] New repo for HI Portal (Cross Sell) | Story | Closed | auth-sdk-ui#291, auth-ui#487, github-terraform#4729, home-improvement-borrower-dashboard-ui#1-9 (all MERGED), k8s#255981/262255/269432 | N/A (infra/bootstrap) | N/A | Confirmed new standalone repo, live since 2026-06-24 |
| HI-7441 | Verify required info sent to CDS from pre-qual | Story | Closed | hi-merchant#5777 (MERGED, UT only), loan-app-creation-srvc#9187 (MERGED, no tests) | Partial | GAP | No IT, no E2E |
| HI-7504 | Add Google maps attribution | Story | Closed | No dedicated PR; code confirmed in new-repo#9 (`GoogleMapsAttribution` + test) | N/A (FE) | GAP | -- |
| HI-7505 | Extra merchant placeholder images by category | Story | In Validation | home-improvement-borrower-dashboard-ui#12 (**still OPEN**) | N/A (FE) | GAP | Status/PR mismatch — WATCH |
| HI-7506 | Hide filter categories if no merchants | Story | Closed | No dedicated PR; likely bundled in new-repo#9 FilterSection | N/A (FE) | GAP | -- |
| HI-7704 | [BE] Minimal backend defense-in-depth | Story | Closed | hi-merchant#5904 (MERGED, UT only) | Partial | GAP | No IT, no E2E |
| HI-7754 | [Omni Prequal][BE] W7 — prequal-decision-srvc client + Branch-A/B gating | Story | Open | -- | GAP (no PRs) | SPEC GAP | **Not started — this is the architectural core of Omni (O1)** |
| HI-7755 | [Omni Prequal][BE] W1 — Batch consumer routing + applicant hydration | Story | In Development | -- | GAP (no PRs) | SPEC GAP | Not started at code level |
| HI-7756 | [Omni Prequal][BE] W2 — Activation rendezvous (APPROVED→ACTIVE) | Story | Open | -- | GAP (no PRs) | SPEC GAP | Not started |
| HI-7757 | [Omni Prequal][BE] W3 — Send prequalDecisionUuid to CDS | Story | In Validation | loan-app-creation-srvc#9287 (OPEN, UT only) | Partial | GAP | No IT, no E2E |
| HI-7758 | [Omni Prequal][BE] W_SHARE — Editable contact at share step | Story | Ready for CodeReview | hi-application-srvc#1038 (OPEN, Done UT+IT), qa#37409 (OPEN/DRAFT, utils only, no test methods yet), qa-gql#1085 (OPEN/DRAFT) | Done (BE) | IN DEV (draft, incomplete) | E2E scaffolding started, no assertions yet |
| HI-7759 | [Omni Prequal][BE] W_EXP — 45-day cross-sell prequal expiry | Story | Closed | Jira resolution = Done (no PR captured in scanned repos) | Done (per resolution) | PARTIAL | S29: spec self-contradicts 30 vs 45 days in different sections — needs product clarification on Branch B |
| HI-7760 | [Omni Prequal][BE] W5 — Tests/feature flag/observability | Story | Open | -- | GAP (no PRs) | SPEC GAP | Not started |
| HI-7761 | [Omni Prequal][BE] W0 — avro-decisioning-lib bump | Story | Closed | Jira resolution = "Self-Resolved" — likely a dependency-bump PR in `avro-decisioning-lib`, not in our tracked repo set | N/A (dependency) | N/A | Not independently verified — different repo scope |
| HI-7762 | [Omni Prequal][BE] W_AMT — Suppress prequal amount on borrower surfaces | Story | Closed | **None — Jira resolution = "Won't Do"** | N/A | SPEC GAP | **Not a gap to fix by this ticket — deliberately deprioritized.** Amount-suppression logic (O3) remains unimplemented; overlaps unresolved with HI-7765 (Blocked) |
| HI-7765 | [FE][Borrower][placeholder] Hide pre-qual amounts on success/cards | Story | Blocked | -- | N/A (FE) | SPEC GAP | Real successor to HI-7762; still Blocked, no PRs — O3 remains a live HIGH gap |
| HI-7766 | [FE][Borrower][placeholder] Carousel copy changes at Explore Contractors | Story | Blocked | -- | N/A (FE) | GAP | No PRs |
| HI-7767 | [FE][placeholder] NBA component at bottom of explore contractors (new PL placement) | Story | In Development | -- | N/A (FE) | SPEC GAP | Corresponds to O26 (PL cross-sell fallback) — not started |
| HI-7802 | [FE][Borrower][placeholder] Pre-fill application screen (Omni flow) | Story | Blocked | -- | N/A (FE) | SPEC GAP | Part of Omni Branch A confirm/edit step (O1-adjacent) |
| HI-7909 | Set prequalDecisionUuid null for on-demand prequal | Story | Ready for CodeReview | hi-application-srvc#1098 (OPEN, IT only) | Partial | GAP | No dedicated new UT |
| HI-6478 | Cross-Sell Program: Test Coverage Checklist | Task | Reopened | qa#34134 (MERGED) | N/A (QA) | N/A | Checklist ticket — revisit now that #34134 is merged; new Omni scope should be added |

\* **COVERED** in this table means the test code exists and is wired into the suite on `qa-automation` master — see the SkipUntil caveat in Spec Summary above. None of these V1 tests are currently exercised by CI on main/stage/preprod.

**PR Classification Summary:** ~40 service-repo PRs checked for UT/IT across `hi-application-srvc`, `home-improvement-merchant-srvc`, `next-best-action-srvc`, `loan-app-creation-srvc`; ~50 UI/infra/schema/client PRs marked N/A per classification rules; qa-automation/qa-automation-graphql PRs are the E2E layer itself.

## PR Analysis

_(Carried forward verbatim from the 2026-06-19 refresh — see full historical detail on Confluence. New entries below.)_

### hi-application-srvc#810 — Add idempotency-lib (HI-7397)

_Analyzed: 2026-08-21_

**Changes**: Adds idempotency protection to cross-sell mutations.

**UT/IT**: `PreQualificationMutationResolverTest` (UT), `CrossSellIdempotencyIT` (IT).

**E2E**: Covered by existing idempotency test [55002] on qa-automation master (pre-existing, from HomeImprovementCrossSellPreQualApiTest).

**Gaps**: None identified for this specific PR's scope.

### hi-application-srvc#1038 — Editable contact at share step (HI-7758, W_SHARE)

_Analyzed: 2026-08-21_

**Changes**: Allows editing contact info at the confirm-share step of the Omni pre-qual flow.

**UT/IT**: `PreQualificationApplicantServiceTest`, `PreQualificationFacadeTest`, `PreQualificationLeadServiceShareContactTest`, `ApplicantServiceTest` (UT) + `CrossSellMutationsIT`, `PreQualificationServiceIT` (IT).

**E2E**: `qa-automation#37409` (OPEN, DRAFT) touches only `HomeImprovementCrossSellUtils.java` — no test methods yet.

**Gaps**: [MEDIUM] E2E scaffolding started but no assertions written.

### hi-application-srvc#1098 — Null prequalDecisionUuid for on-demand prequal (HI-7909)

_Analyzed: 2026-08-21_

**Changes**: Ensures `prequalDecisionUuid` is set to null when the prequal is generated on-demand (Branch B) rather than from the Omni batch (Branch A).

**UT/IT**: Only IT files touched (`CrossSellIdempotencyIT`, `CrossSellMutationsIT`, `PreQualificationServiceIT`) — no dedicated new unit test.

**E2E**: None found.

**Gaps**: [MEDIUM] No unit-level regression test isolating this null-vs-set branch logic; no E2E.

### home-improvement-merchant-srvc#5777 — Send marketingSegment/modelVersion to CDS (HI-7441)

_Analyzed: 2026-08-21_

**UT/IT**: `LoanApplicationCreationFactoryTest`, `PreQualificationServiceTest` (UT only, no IT).

**E2E**: None found.

**Gaps**: [MEDIUM] No integration-level test for the CDS payload contract; no E2E.

### home-improvement-merchant-srvc#5904 — Minimal backend defense-in-depth (HI-7704)

_Analyzed: 2026-08-21_

**UT/IT**: `GooglePlaceDetailsMapperTest` (UT only, rejects non-https URLs from Google Places).

**E2E**: None found.

**Gaps**: [LOW] No IT, no E2E, but low business risk (defensive validation only).

### loan-app-creation-srvc#9287 — Send prequalDecisionUuid to CDS at application (HI-7757, W3)

_Analyzed: 2026-08-21_

**UT/IT**: `HomeImprovementCreditDecisionModelFactoryTest` (UT only, no IT).

**E2E**: None found.

**Gaps**: [HIGH] This is the wiring for O17/O18 (prequal-id-driven re-decision) — currently only unit-tested; no integration or E2E coverage for the actual CDS handoff.

### home-improvement-borrower-dashboard-ui#8, #9 — New repo foundation + full cross-sell migration (HI-7411, HI-6494-6498, HI-6642, HI-6720, HI-6743, HI-6745, HI-6750, HI-6751, HI-7077, HI-7243, HI-7504, HI-7506)

_Analyzed: 2026-08-21_

**Changes**: `#8` (MERGED 2026-07-09) scaffolds a brand-new standalone FE repo. `#9` (MERGED 2026-07-16, 135 files, +34,217/−9,366) migrates the entire borrower-side cross-sell feature from `borrower-dashboard-ui` to this new repo — explore-contractors, pre-qualification, connect-with-contractor screens, NBA/resumption tiles, heap analytics events, Google Maps attribution, category-filter-hiding logic.

**UT/IT**: N/A (FE repo per classification rules), but extensive Jest unit tests/snapshots exist for every migrated component — real coverage exists even though it's out of scope for this document's UT/IT column.

**E2E**: Covered by `qa-automation#34134` (now merged to master) which was updated to target the new repo's routes/selectors; `qa#37388` is a follow-up nav fix.

**Gaps**: The old `borrower-dashboard-ui#70xx/71xx/72xx` PR links throughout this Ticket Map are now historical — the live source of truth for all borrower-side cross-sell FE code is this new repo.

## Coverage Matrix

_(V1 rows carried forward from the 2026-06-19 refresh with the master-merge + SkipUntil caveat applied; see Spec Requirement Gaps below for the full S1-S29 / O1-O35 granular list.)_

| Requirement | Ticket(s) | UT | IT | E2E | Status |
| --- | --- | --- | --- | --- | --- |
| Cross-sell pre-qual creation | HI-6534 | Y | Y | On master (SkipUntil-disabled) | COVERED* |
| Submit cross-sell pre-qual + decision (approved path) | HI-6535, HI-6540 | Y | Y | On master (SkipUntil-disabled) | PARTIAL |
| Submit cross-sell pre-qual (declined path / AAN) | HI-6535, HI-7294 | -- | -- | GAP | GAP (confirmed Won't Do — not a bug, a scope decision) |
| Share contact with merchant | HI-6536 | Y | Y | On master (SkipUntil-disabled) | COVERED* |
| Lead stage / lifecycle | HI-6538 | Y | Y | On master (SkipUntil-disabled) | COVERED* |
| Merchant suspension → lead hidden + restored | HI-6541 | Y | Y | On master (SkipUntil-disabled) | COVERED* |
| Serviceable zip codes + Google Places | HI-6469, HI-6470 | Y | Y | On master (SkipUntil-disabled) | COVERED* |
| Borrower ITA eligibility persistence | HI-6533 | Y | Y | On master — `HomeImprovementCrossSellEligibilityTest` [74461/74462] (SkipUntil-disabled) | COVERED* (PL only; multi-product still gap) |
| Borrower Braze notification (`hi_prequal_offer`) | HI-6540 | Y | Y | On master — `HomeImprovementCrossSellBrazeEventTest` [78105] (SkipUntil-disabled) | PARTIAL (1 of 6 events, and disabled) |
| Merchant Braze notifications | HI-6541 | Y | Y | GAP — qa#37317 test actually asserts the borrower-side event, mislabeled | GAP |
| NBA config + eligibility retriever | HI-6769 | Y | Y | GAP | PARTIAL |
| Reporting & funnel metrics | HI-6543 | Partial | Partial | GAP | GAP |
| **Omni: Branch A/B gating logic (O1)** | HI-7754 (W7, not started) | -- | -- | SPEC GAP | GAP |
| **Omni: Eligibility ≥3 merchants/150mi (O2/S26)** | Spec only | -- | -- | PARTIAL (V1 radius logic reused, 3-vs-5 threshold not confirmed) | PARTIAL |
| **Omni: Amount suppression on borrower surfaces (O3)** | HI-7762 (Won't Do), HI-7765 (Blocked) | -- | -- | SPEC GAP | GAP (confirmed unimplemented, actively at risk) |
| **Omni: Repeat-customer category exclusion + Goldstar overlap (O4/O5)** | Spec only / S28 | -- | -- | `qa#35594` "HICL cross-sell prequal exemption from goldstar eligibility" (MERGED) | COVERED (exemption logic); category-display UI not confirmed |
| **Omni: Monthly batch decisioning + prequal fields (O6/O7)** | HI-7755 (W1, in dev) | -- | -- | `qa#37553` "Add QA coverage for HI cross-sell prequal decision" (MERGED); `qa#37576` validates `prequalType=GOLD_STAR` | PARTIAL |
| **Omni: 45-day expiry / 30-day contradiction (O8/S29)** | HI-7759 (Done) | -- | -- | Unverified whether E2E asserts both branch expiries | PARTIAL |
| **Omni: Sub-cohort treatment (4 states) (O9)** | Spec only, no ticket | -- | -- | SPEC GAP | GAP |
| **Omni: Null-score-vs-decline fairness (O10)** | Spec open question, no ticket | -- | -- | SPEC GAP | GAP |
| **Omni: Offer rounding rule (O13)** | Spec only | -- | -- | SPEC GAP | GAP |
| **Omni: Score-gate cutoff logic (O14)** | Spec only | -- | -- | SPEC GAP | GAP |
| **Omni: prequal-id selection (Goldstar vs Cross-Sell) at application (O17)** | HI-7757 (W3), HI-7758 (W_SHARE) | Y (UT) | Partial | IN DEV (draft, no assertions) | PARTIAL |
| **Omni: Re-decision against locked policy/merchant-config-at-application-time (O18)** | HI-7754 (W7, not started) | -- | -- | SPEC GAP | GAP |
| **Omni: Latest-decision-wins (approve→higher / approve→decline / approve→no-refresh) (O20)** | Spec only, no ticket | -- | -- | SPEC GAP | GAP |
| **Omni: Restricted decline-reason set at re-decision (O23)** | Spec only, no ticket | -- | -- | SPEC GAP | GAP |
| **Omni: PL cross-sell fallback on Explore Contractors (O26)** | HI-7767 (In Development) | -- | -- | SPEC GAP | GAP |
| **Omni: Repeat-customer merchant reporting tab (O28)** | Spec only, no ticket | -- | -- | SPEC GAP | GAP |

## Spec Requirement Gaps

### Original spec (PROD/4494065719) — S1-S25 carried over, S26-S29 new this refresh

| # | Requirement | Source | Priority | E2E Status |
| --- | --- | --- | --- | --- |
| S1-S25 | _(unchanged from 2026-06-19 refresh — see Confluence page version history for full text; all carry over as-is except where superseded below)_ | Spec | -- | See prior refresh; V1 E2E for these now exists on master but is SkipUntil-disabled |
| S26 | Zip-match eligibility threshold lowered from 5 to **3** merchants within 150mi | Spec - Borrower Eligibility table (edited v26) | MEDIUM | SPEC GAP — not specifically asserted (V1 tests may still assume 5) |
| S27 | Cease & Desist borrowers excluded from **NBA only**; still receive email/SMS | Spec - Borrower Eligibility table | LOW | SPEC GAP |
| S28 | HI+Goldstar customers **no longer excluded** from cross-sell eligibility (strikethrough removes prior exclusion) | Spec - Borrower Eligibility table | MEDIUM | PARTIAL — decisioning exemption tested (`qa#35594`); UI category-display not confirmed |
| S29 | **Spec self-contradiction**: Decisioning section says pre-qual offer valid **45 days**; Product Flow / Pre-qualified Page section still says **30 days**, unedited | Spec - Decisioning vs Product Flow sections | HIGH | Needs product clarification — 45 days confirmed real for Omni/batch path (HI-7759 Done); unclear if Branch B on-demand path is meant to stay 30 |

### NEW spec — Omni Pre-Qual (PROD/5856264352), O1-O35

| # | Requirement | Source | Priority | E2E Status |
| --- | --- | --- | --- | --- |
| O1 | Branch A/B gating: route to Branch A if a valid, non-expired, APPROVED Omni prequal + CPA consent exists; else Branch B (on-demand, lower priority) | Product Flow #2, Decisioning sub-cohort table | **HIGH** | SPEC GAP — owning ticket HI-7754 (W7) not started |
| O2 | Eligibility: ≥3 partnered merchants within 150-mile radius (same as S26) | Product Flow #1 | MEDIUM | PARTIAL |
| O3 | Amount suppression: borrower never sees the pre-qualified dollar amount on **any** surface (tile, funnel, email, SMS); merchant sees it in the lead | Decisioning notes, flow-chart sticky note | **HIGH** | SPEC GAP — HI-7762 Won't Do, successor HI-7765 Blocked. Spec itself flags this as an engineering risk ("tile and lead resolve through the same amount-bearing type today") |
| O4 | Category exclusion: repeat HI borrower sees every contractor category except their previous project category | Product Flow #3, "Repeat customer exclusion" note | MEDIUM | SPEC GAP |
| O5 | HI+Goldstar borrowers included; Goldstar covers the excluded category so there's no overlap with Cross-Sell | Product Flow #3, S28 | MEDIUM | COVERED — `qa#35594` |
| O6 | Monthly batch decisioning: CDS generates a generic HI cross-sell prequal during the monthly bureau refresh for consenting, eligible customers, on each customer's own anniversary date (not a single global batch date) | Decisioning intro, Questions section | MEDIUM | PARTIAL — `qa#37553` general coverage exists; per-customer cadence timing not confirmed |
| O7 | Prequal record fields: `prequal_id`, product type `HOME_IMPROVEMENT_CREDIT_LINE`, prequal type (Cross-Sell vs Goldstar), `cross_sell_prequal` flag, prequal amount | Decisioning - Branch A | MEDIUM | PARTIAL — `qa#37576` validates `prequalType=GOLD_STAR`; full field set not confirmed |
| O8 | 45-day expiry for Branch A (Omni batch) vs 30-day expiry for Branch B (on-demand) — two coexisting expiry windows | Decisioning - Branch A / Branch B | **HIGH** | PARTIAL — same underlying issue as S29 |
| O9 | Sub-cohort treatment table: (a) Consented+Approved→Branch A, (b) Approved-no-CPA→on-demand-as-consent-capture (spec itself flags as an open question whether this state can exist), (c) Declined→suppress, don't route to on-demand, (d) No decision produced (not in monthly TU refresh)→on-demand with CPA+TU refresh | Decisioning sub-cohort table | **HIGH** | SPEC GAP — no ticket owns this explicitly |
| O10 | Suppression fairness: a decline caused only by a null required score (data gap) should not suppress the same as a true credit decline — spec explicitly flags needing a decline-reason field to distinguish these | Flow-chart sticky note (SUP) | **HIGH** | SPEC GAP — open question, not yet ticketed |
| O11 | Cross-Sell policy = Goldstar policy + cross-sell carve-out, generic merchant inputs (not a new policy) | Policy and defaults | LOW | N/A — policy-definition, not independently E2E-testable |
| O12 | Default policy inputs: Merchant Category=Remodel, MQG=5, Requested Amount=$100,000 (capped), 84mo@11.49%, required scores HIRM1/IR5/EDQHIRM1 | Default inputs | MEDIUM | PARTIAL — likely embedded in decisioning tests, not independently confirmed |
| O13 | Offer rounding: amount >$10k rounds down to nearest $5k; <$10k rounds to nearest $1k | Default inputs | MEDIUM | SPEC GAP |
| O14 | Score gate: cutoffs on HIRM1/IR5/EDQHIRM1 per MQG+FICO; gate = "score present AND under cutoff" — a score above cutoff OR null both decline (root cause of O10) | Score rules | **HIGH** | SPEC GAP |
| O15 | Proxy rules if scores unavailable | Score rules | LOW | N/A — spec itself incomplete ("add here" placeholders), not yet testable |
| O16 | Application matching: borrower-initiated via actor+prequal id; merchant-initiated via first/last name+DOB (spec marks "to confirm") | Application and re-decision | MEDIUM | SPEC GAP — spec itself not finalized |
| O17 | Prequal-id selection: HI sends CDS the specific locked `prequal_id`; if borrower holds both Goldstar and Cross-Sell prequals, HI picks based on which merchant `account_id` was selected | Application and re-decision | **HIGH** | PARTIAL — HI-7758 (W_SHARE) has BE Done, E2E draft/incomplete |
| O18 | Re-decision: CDS re-decisions against the locked prequal_id + locked policy version, using merchant configs current **at application time**, credit report reused if pulled within 30 days else fresh pull | Application and re-decision | **HIGH** | SPEC GAP — HI-7754 (W7) not started |
| O19 | "Acquisition channel = Repeat Customer" tag on new app project page when started by same merchant [P2] | Application and re-decision | LOW | SPEC GAP |
| O20 | Latest-decision-wins: 3 explicit scenarios — (a) approved→approved-higher (newest wins for marketing display), (b) approved→declined (prior offer goes stale/hidden), (c) approved→no-refresh-next-month (original offer stays valid until its own expiry since no overriding decision exists) | Expired pre-qualified lead | **HIGH** | SPEC GAP — none of the 3 branches found tested |
| O21 | Merchant-initiated app on expired Omni lead: allowed to proceed with warning + fresh credit pull; "honor prequal for 45 days and not override expiry" explicitly marked "(to be confirmed)" in spec | Expired pre-qualified lead | MEDIUM | SPEC GAP — spec itself open |
| O22 | Leads shown in merchant reporting 90 days post-expiry (same rule as V1 S13) | Merchant Reporting | LOW | GAP (carries over from S13) |
| O23 | Restricted decline-reason set at re-decision: only fair reasons are credit-got-worse, fraud, already-took-another-loan, or real-project-very-different-from-assumed (anti bait-and-switch) | Flow-chart sticky note (NO) | **HIGH** | SPEC GAP — compliance rule, zero test coverage |
| O24 | New Omni ITA copy/targeting: "You're pre-qualified..." on Directory+Dashboard, targeted at opted-into-marketing + `cross_sell_omniprequal` ACTIVE + no HI app started, priority P1 | Placements & Emails table | MEDIUM | SPEC GAP — HI-7767/HI-7802 not started |
| O25 | Resume-application NBA: "Don't let your project stall" targeted at passed-decisioning + no-app-after-lead-sent, stops 30 days after lead sent, triggers 7 days after interest submitted, priority P1 | Placements & Emails table | MEDIUM | SPEC GAP |
| O26 | PL cross-sell fallback: shown on Explore Contractors when no contractor found / end of list, gated by PL-prequal flag from CDS, else redirect to PL landing page — resolves V1's old open question definitively | Placements & Emails table | MEDIUM | SPEC GAP — net-new flow, HI-7767 In Development, no PRs |
| O27 | Omni-specific borrower ITA notification: P2 marketing, triggered by `active_prequal` HI cross-sell, 1x/month | Borrower notification table (Omni) | LOW | SPEC GAP |
| O28 | Merchant reporting: new "repeat customer tabs" for prequalified leads (distinct from V1's active/expired lead report) | Merchant Reporting | MEDIUM | SPEC GAP — no ticket found |
| O29 | Un-suppression cadence: suppress only until the *next* monthly refresh returns a new decision, not permanently — "one bad month should not lock someone out" | Flow-chart sticky note (SN) | MEDIUM | SPEC GAP |
| O30 | Prequal log persisted per customer↔merchant combination, tracking-only, not used for decisioning | Questions section | LOW | N/A — infra/logging, not independently gap-worthy |
| O31 | Monthly refresh runs on each customer's own **application-date anniversary**, not one global batch date | Questions section | MEDIUM | SPEC GAP — testing implication: fixtures can't assume a single fixed batch date |
| O32 | No minimum-amount policy change — uses current program minimums (explicit confirmation) | Questions section | N/A | Informational — not a gap |
| O33 | No CRB reporting needed for Omni prequal (explicit negative confirmation) | Questions section | N/A | Informational — not a gap |
| O34 | No monthly credit-score-change notice required unless account opened (explicit negative confirmation) | Questions section | N/A | Informational — not a gap |
| O35 | Prequal policy version reuses current full policy version, no separate bank-approval versioning coupling | Questions section | N/A | Informational — not a gap |

**Workstream ↔ requirement mapping:** W0/HI-7761→O11-O12 (policy plumbing/dependency bump); W1/HI-7755→O6 (batch consumer routing); W2/HI-7756→O1 (activation rendezvous, APPROVED→ACTIVE state machine); W3/HI-7757→O17-O18 (prequalDecisionUuid to CDS); W5/HI-7760→O9-O10, O14 (tests/flag/observability — the workstream most directly responsible for closing the HIGH-priority fairness/gating gaps); W7/HI-7754→O1, O18 (Branch-A/B gating + re-decision — the architectural core, not started); W_AMT/HI-7762→O3 (Won't Do); W_EXP/HI-7759→O8 (Done); W_SHARE/HI-7758→O17 (in code review, E2E incomplete).

## Active Gaps

### Confirmed non-gaps (deliberately deprioritized — do not chase these as bugs)

1. **HI-7294 [BE] Handle AAN** — Jira resolution "Won't Do." The declined/AAN path (spec S3) was never built by product decision, not an oversight. No test should be expected here unless product reopens it.
2. **HI-7762 [Omni][BE] W_AMT — Suppress prequal amount** — Jira resolution "Won't Do." Amount-suppression logic (O3) is not implemented by this ticket; the live successor is HI-7765 (Blocked, FE placeholder). O3 remains a real, tracked SPEC GAP — just not attributable to HI-7762 anymore.

### Critical / HIGH (E2E or implementation needed)

1. **[HIGH]** O1 — Branch A/B gating not started (HI-7754/W7 Open, no PRs) — the entire Omni architecture hinges on this
2. **[HIGH]** O3 — Amount suppression on borrower surfaces unimplemented (HI-7762 Won't Do, HI-7765 Blocked) — spec itself flags this as an unresolved engineering risk
3. **[HIGH]** O9 — 4-way sub-cohort treatment (consent × batch-decision combinations) has no ticket or test coverage
4. **[HIGH]** O10 — Null-score-vs-true-decline suppression fairness is an open compliance question with no ticket
5. **[HIGH]** O14 — Score-gate cutoff logic (HIRM1/IR5/EDQHIRM1) untested at E2E
6. **[HIGH]** O17/O18 — Prequal-id selection (Goldstar vs Cross-Sell) + re-decision against locked policy/current-merchant-config — BE in progress (HI-7757/HI-7758), no E2E assertions yet
7. **[HIGH]** O20 — Latest-decision-wins re-solicitation logic (3 branches: approve→higher, approve→decline, approve→no-refresh) entirely untested
8. **[HIGH]** O23 — Restricted decline-reason enumeration at re-decision (anti bait-and-switch) has zero test coverage
9. **[HIGH]** S29 — 30-day vs 45-day offer-validity contradiction needs product clarification before it can be tested correctly
10. **[CARRIED OVER, HIGH]** S1 — Borrower eligibility: only PL covered at E2E; PCL/Deposit/HI/FlexPay untested
11. **[OPERATIONAL, HIGH]** The entire V1 E2E suite (9 classes, 22 tests) is `@SkipUntil`-disabled on main/stage/preprod until 2050 — none of the "COVERED" V1 items above are currently a meaningful CI signal. Needs a decision: re-enable as-is, or hold frozen pending W7 (since V1's on-demand-only assumptions may not survive Branch-A).

### Medium Priority

1. **[MEDIUM]** S26/O2 — Merchant-count eligibility threshold changed 5→3; not confirmed re-asserted in tests
2. **[MEDIUM]** S28/O5 — HI+Goldstar category-display in UI (exemption logic itself is COVERED via `qa#35594`)
3. **[MEDIUM]** O4 — Repeat-customer category exclusion logic — no dedicated ticket found
4. **[MEDIUM]** O6/O7 — Batch decisioning cadence + full prequal field set — partial coverage via `qa#37553`/`qa#37576`
5. **[MEDIUM]** O12/O13 — Default policy inputs + offer rounding rule untested
6. **[MEDIUM]** O16 — Application-matching rules still marked "to confirm" in spec itself
7. **[MEDIUM]** O21 — Merchant-initiated app on expired Omni lead — spec itself has an open "(to be confirmed)" item
8. **[MEDIUM]** O24/O25 — New Omni ITA + resume-application NBA placements — HI-7767/HI-7802 not started
9. **[MEDIUM]** O26 — PL cross-sell fallback on Explore Contractors — net-new flow, zero coverage
10. **[MEDIUM]** O28 — Repeat-customer merchant reporting tab — no ticket found
11. **[MEDIUM]** O29 — Un-suppression retry cadence rule — buried in flow-chart notes only, easy to miss
12. **[MEDIUM]** O31 — Per-customer monthly-anniversary batch cadence has direct test-fixture implications
13. **[CARRIED OVER, MEDIUM]** HI-6541 — merchant-side Braze notification E2E — `qa#37317` actually tests the *borrower*-side event despite the ticket label; genuinely untested
14. **[CARRIED OVER, MEDIUM]** HI-6769 — NBA config E2E gap
15. **[CARRIED OVER, MEDIUM]** HI-7441 — CDS marketing-segment forwarding has no IT/E2E
16. **[CARRIED OVER, MEDIUM]** S5-S8, S10-S12, S18, S21-S22 — see prior refresh detail on Confluence; status unchanged, V1 E2E exists but SkipUntil-disabled

### Lower Priority

1. **[LOW]** S27 — Cease & Desist NBA-only exclusion untested
2. **[LOW]** O11, O15, O19, O22, O27, O30 — see table above
3. **[LOW]** HI-7704 — defense-in-depth UT only, low business risk
4. **[LOW]** HI-6543/S19 — funnel metrics reporting

### Informational (confirmed, no test action needed)

- O32-O35 — explicit negative confirmations from the Omni spec's Questions section (no minimum-amount change, no CRB reporting, no monthly score-change notice, no separate policy versioning)

### New / Watch

1. **[WATCH]** HI-7505 — status "In Validation" but its PR (new-repo#12) is still OPEN — status/PR mismatch
2. **[WATCH]** HI-7759 — Jira "Done" but no PR captured in our tracked repos; verify against `avro-decisioning-lib` or wherever the actual change landed
3. **[WATCH]** HI-7761 — resolution "Self-Resolved"; likely a dependency bump in `avro-decisioning-lib`, outside our tracked repo set — not independently verified

### Changes Since Last Refresh (2026-06-19 → 2026-08-21)

* **Epic grew from 54 to 75 child tickets** — 20 net-new tickets, almost entirely the new "X-Sell Omni Prequal" workstream (HI-7754/7755/7756/7757/7758/7759/7760/7761/7762, HI-7909) plus supporting FE placeholders (HI-7765/7766/7767/7802) and misc BE/FE cleanup (HI-7411, 7441, 7504, 7505, 7506, 7704).
* **V1 shipped**: `qa-automation#34134` merged to master 2026-07-24. Borrower-side FE fully migrated to a **brand-new standalone repo**, `home-improvement-borrower-dashboard-ui` (created 2026-06-24) — confirmed via `gh repo view` and PR #8/#9 (135-file, +34k/-9k line migration). All prior `bd-ui#70xx/71xx/72xx` PR links are now historical.
* **Critical operational finding**: every V1 cross-sell E2E test method (9 classes, 22 tests) carries `@SkipUntil(envToSkip={"main","stage","preprod"}, skipBefore="2050-12-31")` — confirmed by direct grep of master. None of it runs in CI on the three key environments today.
* **18 tickets flipped to Closed** since last refresh (HI-6396, 6494-6498, 6534-6538, 6720, 6754/6755, 6766, 6770/6771, 6887, 7158, 7223, 7077, plus more) — near-total status progression on the V1 backlog.
* **2 tickets confirmed "Won't Do"** via direct Jira lookup: HI-7294 (Handle AAN) and HI-7762 (W_AMT amount suppression) — both deliberately deprioritized, not implementation gaps to chase.
* **New Omni Pre-Qual spec read in full** (`PROD/5856264352`, created 2026-07-22, v13 as of 2026-07-29): extracted 35 granular requirements (O1-O35). Architectural core (Branch A/B gating, W7/HI-7754; activation rendezvous, W2/HI-7756) **not started**. Highest-risk gaps: amount suppression (O3, Won't Do), sub-cohort fairness (O9/O10), score-gate logic (O14), re-decision/prequal-id selection (O17/O18), latest-decision-wins (O20), restricted decline-reasons (O23) — all HIGH priority, all currently untested.
* **Primary spec also edited** (v26, 2026-08-05): 4 new granular requirements (S26-S29) bleeding in from the Omni work — merchant-count threshold 5→3, C&D NBA-only exclusion, HI+Goldstar no-longer-excluded, and a live 30-vs-45-day offer-validity self-contradiction between the Decisioning and Product Flow sections.
* **3 new qa-automation E2E PRs**: `#37317` (MERGED, adds `HomeImprovementCrossSellBrazeEventTest`, mislabeled as merchant notification but actually asserts the borrower-side event), `#37388` (MERGED, nav fix for new-repo selectors), `#37409` (OPEN/DRAFT, utils only, no test methods yet). Separately, 3 Omni-decisioning E2E PRs landed and merged: `#37553`, `#35594`, `#37576` — these test `PrequalDecisionHiclCrossSellTest`/`PrequaDecisionlHiclGoldstarTest`, which run **unconditionally** (no SkipUntil).
* **New service PRs checked for UT/IT**: hi-application-srvc#810 (Done), #1038 (Done), #1098 (Partial — IT only); home-improvement-merchant-srvc#5777 (Partial — UT only), #5904 (Partial — UT only); loan-app-creation-srvc#9287 (Partial — UT only). None reach full UT+IT+E2E coverage yet.

## Deployment — Feature Flags & Config (E2E stack)

_(Carried forward from 2026-06-19 refresh — see Confluence history for full table.)_ **New operational note (2026-08-21): the entire V1 crosssell TestNG suite is currently `@SkipUntil`-gated off on main/stage/preprod until 2050-12-31 regardless of flag state — flags alone will not make these tests run.** Omni Pre-Qual workstreams (W0-W7) do not yet have documented flags since W7 (the gating logic) has not been built.

## Decisions

_(2026-04-16 through 2026-06-19 decisions carried forward verbatim — see Confluence page history.)_

* 2026-07-09/07-16: New standalone repo `home-improvement-borrower-dashboard-ui` created and the entire borrower-side cross-sell FE migrated there from `borrower-dashboard-ui` (PR#8 foundation, PR#9 full migration). All future borrower FE cross-sell work happens in this repo.
* 2026-07-22: New "Omni Pre-Qual" spec (`PROD/5856264352`) added to the initiative — flips V1's on-demand-only prequal to an upfront monthly-batch model reusing Goldstar's decisioning machinery. Tracked via epic CRD-19822 and 9 `[X-Sell Omni Prequal][BE]` workstream tickets (W0/W1/W2/W3/W5/W7/W_AMT/W_EXP/W_SHARE).
* 2026-07-24: `qa-automation#34134` merged to master — V1 cross-sell E2E is code-complete but shipped with a blanket `@SkipUntil` disabling all 22 tests on main/stage/preprod until 2050, per the PR's own annotation reason: "More changes in cross sell are planned from business."
* 2026-08-05: Primary spec edited (v26) — eligibility merchant-count threshold lowered 5→3, HI+Goldstar exclusion removed, C&D exclusion scoped to NBA only, and a 45-day expiry introduced in the Decisioning section that was not mirrored into the (still-30-day) Product Flow section.
* 2026-08-21: **Confirmed via direct Jira lookup**: HI-7294 (Handle AAN) and HI-7762 (W_AMT amount suppression) both closed with resolution "Won't Do" — the declined/AAN path and amount-suppression logic are deliberately out of scope, not overlooked. HI-7765 (Blocked) is the live successor tracking amount-suppression FE work.
* 2026-08-21: Refresh discovered this CONTEXT.md had never been committed to the `feature-knowledge-base` git repo despite 10 versions of Confluence sync history — reconstructed from the Confluence page body and re-established as the git source of truth.
