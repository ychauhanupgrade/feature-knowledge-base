---
epic: HI-6336
title: Merchant Base Grade
spec_url: https://credify.atlassian.net/wiki/spaces/PROD/pages/4923457872/Home+Improvement+Base+Grade
status: in-development
last_refreshed: 2026-10-14
test_checklist_ticket: HI-7886
confluence_page_id: "5952569412"
---

# HI-6336 — Merchant Base Grade

## Spec Summary

HI Credit wants a fine-grained risk categorization ("base grade") with merchant-level line-amount caps. The formula (HIRM1, BK1, and other credit variables) produces a base grade per borrower, scored against 3 FICO bands (780–850, 660–779, 300–659). Each merchant gets a base grade "preset" — a matrix mapping (base grade × FICO band) → {Tier 1 / Tier 2 / Decline} plus a max loan amount, capped at the tier's overall ceiling ($200k Tier 1, $35k Tier 2 in the spec text; Tier 3/4 are explicit future scope). VQ agents create presets (Default / High-Risk, or custom), assign them to merchants (single preset per merchant, not per-tier), and must be able to view a preset before assigning it. A hard business rule guards assignment: a preset referencing a Tier must not be assignable to a merchant that doesn't have that Tier enabled + a pricing preset assigned for it. Base grade is locked at application creation — reassigning a merchant's preset must not affect outstanding offers, only go-forward applications. Changes must be logged and queryable via EDW. During decisioning, CDS calculates the borrower's grade, runs Tier 1/2 decisioning, cascades to higher tiers on decline until an offer/counteroffer is produced, caps the max loan amount from the base grade before accommodations run, and still applies a global MaxHIRM1 check. ARIX field16 gains 5 new reporting fields (consumer_final_grade, base_grade_preset_id/range_id, loan_amount_to_hh_income, payment_to_income).

**Spec v14 (2026-08-12) internal inconsistencies, found on this refresh.** The prose says "base grades 1-20" (and elsewhere "10 base grades"), but the sample table has grades 1–15 plus a ">=16" row. The sample table also uses Tier 3 (out of v1 scope) and lists 780–850 / grade 7 as Tier 2 at 100,000, which is above the 35,000 Tier 2 cap in the same document. The seeded preset "Default v1" (HI-6798; live on stage and ondemand) does not match the sample table (flat 100k Tier 1, different tiers and decline cutoffs). The backend Tier 2 cap config is 100,000 (TIER_TWO) versus 35,000 in the spec text. The UI form shows 16 rows (MAX_BASE_GRADE=16 in servicing-ui `constants.js`); product has said the form needs 20 rows, UI change pending (tracked under HI-8376), while the backend already supports 20 (`base-grade.max-base-grade=20`, DB check `min_base_grade<=20`, `min-catch-all-grade=11`). The UI merges adjacent identical rows; MPDS does not merge. Open product questions: how base grade is displayed on the VQ offer page, what happens when Tier 2 is turned off while the merchant holds a Tier 2 preset, and Tier 2 batch assignment is all-or-nothing in HMS (fail-fast, detail in `ineligibleMerchants`).

The epic spans 6 services (MPDS, HMS, LACS, HIDS, activity-events, CDS) plus 2 UI repos (`home-improvement-servicing-ui`, `ccp-portal-components-ui`). Per HI-7859 (the epic's own retroactive tech-design ticket, design set now merged as docs.credify.tech#349), the epic was originally handed over as "backend done, frontend remaining" — that framing was wrong: MPDS and LACS had no epic ticket at all until HI-7855/HI-7856 were opened to cover real defects found in review.

**Backend and UI E2E both exist now.** `qa-automation#38176` (HI-6775, merged 2026-09-29) added `HomeImprovementMerchantBaseGradeTest` (API) and `qa-automation#39042` (merged 2026-10-07, merge `23c40dbe69`) added `HomeImprovementMerchantBaseGradeUiTest` (15 Playwright UI tests plus 91953 API). Master therefore holds 21 base-grade E2E tests (6 API: 84685, 84686, 84687, 84688, 84960, 91953; 15 UI: 91368–91375, 92146, 92252, 92672, 92673, 91659, 91660). Only 84688 is still gated (`@SkipUntil` on every env, CRD-17736). Open PR `qa-automation#39405` (branch `HI-6336-BaseGradeEpicUiTests`) adds 93811/93812 (create-form tier cap and validation), DB-vs-UI range verification, removes the UI `@SkipUntil` gates and trims the continuous-test group `home-improvement-servicing-ui` to four UI tests (91368, 91369, 91373, 92672); the other base-grade UI tests run only in the suite `home-improvement-merchant-base-grade-tests.xml` (registered by qa-jenkins-jobs#3434, merged 2026-10-09). A mixed-grid merge test (95207) exists only locally and is not in any PR. The separate Decisioning-owned coverage (`CreditDecisionHiclLockingTest`, `CreditDecisionMerchantPlanDefinitionMockHelper`, under `CRD-17733`/`CRD-18201`) validates CDS's WireMock-mocked consumption/locking of a preset and is not tied to any HI-6336 ticket.

## Ticket Map

UT/IT for UI repos means jest/RTL component tests in `home-improvement-servicing-ui` (listed per ticket); IT is N/A for UI. E2E values: COVERED = on master, IN DEV = open PR or local only.

| Ticket | Summary | Status | UT | IT | E2E | PRs |
|---|---|---|---|---|---|---|
| HI-6636 | [BE] Tech Design | Closed | N/A | N/A | N/A | — (design only) |
| HI-6646 | [BE] MPDS implementation (preset CRUD, score groups/bands) | Closed | Done | Done | COVERED | merchant-plan-definition-srvc#2321, hi-common-lib#516, home-improvement-merchant-srvc#5188/#5122 — E2E 84685 |
| HI-6656 | [BE] HMS merchant assignment | Closed | Done | Done | COVERED | home-improvement-merchant-srvc#5188, activity-events#1410, spicedb-schemas#1355 — E2E 84685 |
| HI-6657 | [BE] HIDS ARIX field16 | Closed (was In Validation) | Done | Done | PARTIAL | home-improvement-disbursement-srvc#3715 (MERGED 2026-09-30), k8s-template#298586 (preprod deploy, merged) — E2E 84688 exists but is @SkipUntil on every env (CRD-17736) |
| HI-6658 | [BE] LACS merchant project field | Closed | Done | Partial | COVERED | loan-app-creation-srvc#8680, loan-app-creation-client#2634 — E2E 84686, 91953 (no persistence IT) |
| HI-6768 | [BE] activity-events new activity types | Closed | N/A (lib, no tests in PR) | GAP | PARTIAL | activity-events#1410 — only `RecordActivityCommand` enum; PRESET_ASSIGNED/REMOVED types are defined but never emitted (HMS emits generic HOME_IMPROVEMENT_MERCHANT_CONFIG_CHANGED) |
| HI-6774 | Test coverage checklist (Phase 1) | Closed | N/A | N/A | N/A | — (checklist doc) |
| HI-6775 | QA automation E2E tests | Ready for CodeReview | N/A | N/A | COVERED (API + UI) / IN DEV | qa-automation#38176 (merged 09-29), #39042 (merged 10-07), #39405 (OPEN), qa-automation-graphql#1165 (merged), qa-jenkins-jobs#3434 (MERGED 10-09) |
| HI-6776 | Base Grade Config — Navigation & Entry Point | Closed (was Ready for CodeReview) | Done (UI jest: Header.test.js in ccp-portal-components-ui) | N/A | COVERED | home-improvement-servicing-ui#154 (MERGED 10-02, dependency bump only), ccp-portal-components-ui#304 (MERGED) — E2E 91659, 91660 |
| HI-6777 | Manage Base Grade Configurations — List View | In Validation | Done (UI jest: BaseGradeConfigurationList index/RowActions/utils/styles) | N/A | COVERED | home-improvement-servicing-ui#100 — E2E 91368 |
| HI-6778 | Create New Base Grade Configuration | In Validation | Done (UI jest: CreateBaseGradeConfiguration index/ApprovalTierCell/MaxLoanAmountCell/ScoreBandSegment/utils) | N/A | COVERED + IN DEV | home-improvement-servicing-ui#87 — E2E 91369, 91370, 91371; IN DEV 93811, 93812 (#39405), 95207 (local) |
| HI-6779 | View Base Grade Configuration — Read-Only Detail | In Validation | Done (UI jest: ViewBaseGradeConfiguration index/SegmentTable/utils) | N/A | COVERED | home-improvement-servicing-ui#123 — E2E 91368, 92252 |
| HI-6780 | Duplicate Base Grade Configuration | In Validation | Done (UI jest: pages/Duplicate index) | N/A | COVERED | home-improvement-servicing-ui#127 — E2E 91372 |
| HI-6781 | Assign to Merchants — Search & Selection | In Validation | Done (UI jest: ManageMerchants index/AssignMerchants/MerchantSearch/ReplaceConfigurationModal/utils) | N/A | COVERED | home-improvement-servicing-ui#138 (merged) — E2E 91373, 92146 |
| HI-6782 | Assign — Overwrite Handling & Assigned Merchants View | Closed (was In Validation) | Done (UI jest: AssignedMerchants.test.js, ReplaceConfigurationModal.test.js) | N/A | COVERED | home-improvement-servicing-ui#144 (MERGED 10-02) — E2E 91374, 92252, 91953 |
| HI-6798 | Seed first base grade preset via liquibase | Closed | N/A | Partial | PARTIAL | merchant-plan-definition-srvc#2391 — BaseGradePresetValidationIT validates the seed CSV; backfill job has no dedicated test; seed does not match spec table |
| HI-7590 | [BE] Block Tier 2 preset assignment without Tier 2 enabled | In Validation | Done | Partial | COVERED | home-improvement-merchant-srvc#5791 (merged), merchant-plan-definition-srvc#2727 (merged) — E2E 84687, 91375, 92673 |
| HI-7723 | [ccp-portal-components] Add nav entry | In Validation | Done (UI jest: Header.test.js) | N/A | COVERED | ccp-portal-components-ui#304 (MERGED) — E2E 91659, 91660 |
| HI-7774 | Archive a configuration | Blocked | N/A | N/A | N/A | none — no archive mutation exists in either backend, product decision pending |
| HI-7854 | [BE] MPDS require explicit grade 1–20 coverage + DB constraint | Closed | Done | Done | N/A (UT/IT) | folded into merchant-plan-definition-srvc#2833 |
| HI-7855 | [BE] MPDS backend gaps (domain constraint, duplicate-race, ID serialization, schedule seed, tier-cap query, enum-drift) | In Validation | Done | Partial | PARTIAL | merchant-plan-definition-srvc#2833 (merged) — URN serialization asserted in BaseGradePresetCreateIT; no concurrency test for the duplicate race |
| HI-7856 | [BE] LACS write-once + stop 200-on-dropped-write | In Validation | Done | Partial | COVERED | loan-app-creation-srvc#9337 (merged; prod deploy k8s-template#286559) — E2E 91953; controller IT covers disallowed status only |
| HI-7857 | [BE] HMS follow-ups (producer wiring, 2nd tier condition, rejection detail, franchise clone) | In Validation | Done | Partial | COVERED | home-improvement-merchant-srvc#5951 (merged) — E2E 84686, 84960, 84687 |
| HI-7859 | [Design] Tech design set | Ready for CodeReview | N/A | N/A | N/A | docs.credify.tech#349 (MERGED; #196 DECLINED) |
| HI-7886 | Test coverage checklist (Phase 2) | In Development (was Open) | N/A | N/A | N/A | — (checklist doc, re-populated 2026-10-14 with UT/IT/UI-unit/E2E AllureId rows) |
| HI-8087 | Merchant Servicing — Base Grade Configuration row & edit | Resolved (was In Validation) | Done (UI jest: BaseGradeConfigurationEditor/Item, RiskControlSection, utils/base-grade-presets) | N/A | COVERED | home-improvement-servicing-ui#155 (MERGED 10-06) — E2E 92672, 92673 |
| HI-8096 | [Bug] CDS `ranges()` omitted `baseGradePresetId` — 500 on HICL decisions | Closed (was In Validation) | Done | GAP | PARTIAL | credit-decision-srvc#10227 (merged; UT only) |
| HI-8376 | **NEW** Merchant base grade: QA check fixes (FE) | In Development | GAP (no PR yet) | N/A | GAP | none yet — replace-button colour, @ValidText name/description validator, name maxLength 255, pending 20-row decision, tier-cap inline validation question |

**PR classification summary (2026-10-14 refresh, 42 unique PRs, +4 new vs 2026-09-29):** 12 service PRs (merchant-plan-definition-srvc ×4, home-improvement-merchant-srvc ×4, home-improvement-disbursement-srvc ×1, loan-app-creation-srvc ×2, credit-decision-srvc ×1) plus 1 excluded false positive, 12 UI PRs (N/A for service UT/IT; jest checked separately — home-improvement-servicing-ui ×8, ccp-portal-components-ui ×1, app-by-phone-ui ×3 declined), 3 client/lib PRs (N/A — loan-app-creation-client, hi-common-lib, activity-events), 7 infra/schema PRs (N/A — spicedb-schemas, github-terraform, k8s-template ×5, **new**: k8s-template#298586), 2 docs PRs (N/A — docs.credify.tech#349 **new, merged**, #196 declined), 5 QA repo PRs (qa-automation#38176 merged, **new** #39042 merged, **new** #39405 open, qa-automation-graphql#1165 merged, qa-jenkins-jobs#3434 now merged). 12+1+12+3+7+2+5 = 42. `hi-application-srvc#527` ("[HI-5515] Add application domain data model") is unrelated to base grade — excluded as a false positive. Epic PR `home-improvement-servicing-ui#164` (branch `HI-6336-Merchant-Base-Grade`, OPEN) is not Jira-linked; it aggregates #144/#154/#155 and its jest files are the union of the per-PR files above. PR status transitions since 2026-09-29: home-improvement-disbursement-srvc#3715, servicing-ui#144/#154/#155, ccp-portal-components-ui#304 and qa-jenkins-jobs#3434 moved to MERGED.

## Coverage Matrix

| Requirement Area | UT | IT | E2E | Notes |
|---|---|---|---|---|
| MPDS: preset CRUD, score groups/bands, name/content duplicate detection | Done | Done | COVERED | HI-6646; E2E 84685, 91369–91371; `BaseGradePresetCreateIT`, `BaseGradeQueryIT`, `BaseGradeValidatorTest`, `BaseGradePresetServiceTest` |
| MPDS: seed initial preset via Liquibase | N/A | Partial | PARTIAL | HI-6798; `BaseGradePresetValidationIT` validates the seed CSV; no dedicated test for `BackfillBaseGradePresetHashContentJob`; seed does not match spec table |
| MPDS: grade 1–20 domain bound + DB CHECK | Done | Done | N/A (UT/IT) | HI-7854/HI-7855 (PR#2833 merged); `BaseGradePresetRangeConstraintIT` (3 tests), `BaseGradeValidatorTest` |
| MPDS: duplicate-race, URN serialization, enum drift, tier-cap, schedule seed | Done | Partial | PARTIAL | URN serialization asserted in `BaseGradePresetCreateIT` (resolved); duplicate race only tested sequentially (no concurrency test); `GraphQLEnumConsistencyTest` present |
| MPDS: `containedTiers` field | Done | Done | COVERED | HI-7590 (PR#2727); `BaseGradePresetResolverTest`; E2E 84687 asserts it |
| HMS: assign/batch-assign/remove mutations | Done | Done | COVERED | HI-6656; `BaseGradePresetMutationIT` (4 tests), `MerchantConfigurationServiceTest`; remove not exercised by E2E |
| HMS: activity/event publication on assign/remove | Done | GAP | PARTIAL | `ActivityCommandPublisherTest`, `MerchantStatusEventPublisherTest` (UT only); no IT proves the Kafka publish |
| HMS: Tier 2 guard (reject/allow, batch fail-fast, detail) | Done | Partial | COVERED | E2E 84687, UI 91375/92673 cover reject + `ineligibleMerchants`; allow-path with Tier 2 enabled and mixed multi-merchant batch not covered above UT |
| HMS: producer wiring + clone exclusion | Done | Partial | COVERED | `MerchantProjectFactoryTest`, `MerchantConfigurationServiceTest.doesNotCopyBaseGradePreset...`; E2E 84686, 84960 now ungated on master |
| LACS: preset snapshot at application creation | Done | Partial | COVERED | E2E 84686, 91953; mapper UT only, no persistence IT |
| LACS: write-once + stop-200-on-dropped-write | Done | Partial | COVERED | `MerchantProjectServiceTest`, `MerchantProjectFacadeTest` UT; `MerchantProjectControllerIT` covers disallowed status only; E2E 91953 covers go-forward behavior |
| HIDS: ARIX field16 additions | Done | Done | PARTIAL | HI-6657 (PR#3715 merged 09-30); E2E 84688 skipped on every env (CRD-17736 In Development) |
| CDS: grade calculation, Tier cascade, max-loan cap, MaxHIRM1 | N/A (Decisioning-owned) | N/A | BLOCKED | CRD-17736 (HI Engine baseGrade policy v33) In Development; `CreditDecisionHiclLockingTest` (mocked) is the only coverage |
| CDS: `ranges()` query fix (HI-8096) | Done | GAP | PARTIAL | #10227 UT only; no contract/IT against MPDS |
| VQ UI: nav, list, create, view, duplicate, assign, assigned tab, merchant servicing row | Done (jest) | N/A | COVERED | 15 UI E2E on master; 93811/93812/95207 IN DEV; UI jest per ticket in Ticket Map |
| VQ UI: archive | N/A | N/A | BLOCKED | HI-7774, no BE mutation |
| VQ UI: FE fixes from QA checks (validator, button colour, maxLength, 20 rows) | GAP | N/A | GAP | HI-8376 In Development, no PR |
| Base grade locked at application creation | Done | Partial | COVERED | E2E 91953 (reassign then verify existing and new application) |
| EDW-queryable audit log of preset changes | Done (event UT) | GAP | PARTIAL | Implemented via generic config-changed event; EDW ingestion unverified; no owning ticket |

### Per-requirement checklist (mirrors HI-7886)

Columns: UT and IT name backend test classes (method names where useful); UI unit means jest files under `home-improvement-servicing-ui/src` (or ccp-portal-components-ui where stated). E2E lists AllureIds in `HomeImprovementMerchantBaseGradeTest` (API) and `HomeImprovementMerchantBaseGradeUiTest` (UI). Status: COVERED = on master, IN DEV = PR #39405 or local, PARTIAL, GAP, BLOCKED.

#### 1. Tier 2 Assignment Guard (HI-7590; HMS PR #5791, MPDS PR #2727)

| Requirement | UT | IT | UI unit | E2E (AllureId) | Status |
|---|---|---|---|---|---|
| Assign a preset containing a TIER_TWO range to a merchant WITHOUT Tier 2 enabled is rejected, merchant not modified | MerchantConfigurationServiceTest.assignBatchRejectsMerchantMissingRequiredTier; MerchantPlanTierServiceTest.findMissingTiers_* | BaseGradePresetMutationIT.assignBaseGradePresetToMerchants_rejectsMerchantWithoutRequiredTier | - | 84687 (API); 91375, 92673 (UI) | COVERED |
| Assign a TIER_TWO preset to a merchant WITH Tier 2 enabled succeeds | MerchantPlanTierServiceTest.hasTiersEnabled_returnsTrue_whenAllNonStandardTiersEnabled | BaseGradePresetMutationIT.assignBaseGradePresetToMerchants (happy path only, no Tier 2 enabled variant) | - | none (84685, 91953, 91373, 92672 all use STANDARD-only presets) | PARTIAL |
| Assign a STANDARD/DECLINED-only preset to any merchant regardless of tier state succeeds | MerchantPlanTierServiceTest.hasTiersEnabled_returnsTrue_whenOnlyStandardRequested | BaseGradePresetMutationIT.assignBaseGradePresetToMerchants | - | 84685, 91953 (API); 91373, 92672 (UI) | COVERED |
| Batch assign: one ineligible merchant rejects the WHOLE batch, no partial assignment | MerchantConfigurationServiceTest.assignBatchReportsMissingTiersForEveryIneligibleMerchant | none (IT rejects a single merchant only) | AssignMerchants.test.js | 84687 and 91375 reject one merchant; mixed multi-merchant batch not asserted | PARTIAL |
| Removing a preset is NOT gated by the tier check | MerchantConfigurationServiceTest (remove, removeWhenAlreadyNull, removeBatchDelegatesToUpdateMerchantConfiguration) | BaseGradePresetMutationIT.removeBaseGradePresetFromMerchants | - | none (no E2E exercises remove) | PARTIAL |
| MPDS unreachable while fetching preset ranges: assignment fails closed | MerchantPlanDefinitionServiceTest.getBaseGradePresetContainedTiersRethrowsUnexpectedErrors | none (IT mocks MerchantPlanDefinitionService) | - | none (needs fault injection) | PARTIAL |
| Re-assigning the same preset to the same merchant still runs the tier guard | none | none | - | none | GAP |

#### 2. HMS Follow-ups (HI-7857; HMS PR #5951)

| Requirement | UT | IT | UI unit | E2E (AllureId) | Status |
|---|---|---|---|---|---|
| MerchantProjectFactory populates baseGradePresetId from the project snapshot; no preset gives null | MerchantProjectFactoryTest.create_populatesBaseGradePresetId_whenProjectHasAnAssignedPreset, create_setsBaseGradePresetIdToNull_whenProjectHasNoAssignedPreset*, create_sourcesBaseGradePresetIdFromProjectSnapshot_notFromMutableMerchantConfig | none | - | 84686, 91953 | COVERED |
| Tier enabled but NO pricing preset assigned for it is rejected (second condition) | MerchantPlanTierServiceTest.hasTiersEnabled_* (tier-enabled condition; pricing-preset condition not identifiable by name) | none | - | none | PARTIAL |
| Multi-tier preset where one tier passes and one fails is rejected, error names the missing tier | MerchantPlanTierServiceTest.findMissingTiers_namesTheTiersNotEnabled_andNeverReportsStandard | none | - | 84687 (single missing tier) | PARTIAL |
| Nonexistent preset id gives a typed 400, not a raw GraphQL exception | MerchantPlanDefinitionServiceTest.getBaseGradePresetContainedTiersTranslatesPresetNotFound | none (IT mocks MerchantPlanDefinitionService) | - | none | PARTIAL |
| Batch rejection carries per-merchant detail (ineligibleMerchants: merchantId, missingTiers) | MerchantConfigurationServiceTest.assignBatchReportsMissingTiersForEveryIneligibleMerchant | BaseGradePresetMutationIT (rejectsMerchantWithoutRequiredTier) | - | 84687 | COVERED |
| Merchant clone does not copy the preset id or its assigned timestamp | MerchantConfigurationServiceTest.doesNotCopyBaseGradePresetOrItsAssignedTimestamp | none | - | 84960 | COVERED |
| New error codes appear in the enum-consistency guard | MPDS GraphQLEnumConsistencyTest.errorCode and merchantPlanTierName (HMS-side equivalent not confirmed) | - | - | N/A | PARTIAL |

#### 3. MPDS Backend Hardening (HI-7854 Closed, HI-7855; MPDS PR #2833 merged)

| Requirement | UT | IT | UI unit | E2E (AllureId) | Status |
|---|---|---|---|---|---|
| Band whose ranges stop below grade 20 is rejected with BASE_GRADE_RANGE_COVERAGE_INVALID | BaseGradeValidatorTest.rejectsLastRangeNotCoveringMinCatchAllGrade, rejectsGapBetweenRanges, rejectsFirstRangeNotStartingAtOne | BaseGradePresetCreateIT.createRejectsGapInCoverage, createRejectsMissingBandCoverage | - | N/A (covered at UT/IT) | COVERED |
| Band covering exactly 1-20 is accepted; top range stored as the catch-all | BaseGradeValidatorTest.acceptsMultipleContiguousRanges, acceptsMinGradeAtDomainCeiling | BaseGradePresetCreateIT.createPresetHappyPath; BaseGradePresetRangeConstraintIT.acceptsMinBaseGradeAtTheDomainCeilingAtTheDatabaseLevel | - | 84685 (API); 91369 (UI create, persisted ranges) | COVERED |
| minBaseGrade = 21 written at the repository layer is rejected by the DB CHECK | - | BaseGradePresetRangeConstraintIT.rejectsMinBaseGradeAboveDomainCeilingAtTheDatabaseLevel, rejectsApprovedRangeWithZeroAmountAtTheDatabaseLevel | - | N/A (IT only by design) | COVERED |
| Concurrent duplicate-content creates: race loser gets the typed duplicate error | BaseGradePresetServiceTest.createRejectsDuplicateName, createRejectsDuplicateContent (sequential) | BaseGradePresetCreateIT.createRejectsDuplicateName, createRejectsDuplicateContent (sequential, no concurrency test) | - | 91370, 91371 (sequential, via UI) | PARTIAL |
| existingPresetId in a conflict error serializes as a URN string | - | BaseGradePresetCreateIT.createRejectsDuplicateName and createRejectsDuplicateContent assert the value starts with urn: | - | 91370 (UI shows the View configuration link built from it) | COVERED |
| GraphQL enum-drift guard covers the base-grade enums | GraphQLEnumConsistencyTest (errorCode, merchantPlanTierName, merchantType) | - | - | N/A | COVERED |
| Tier-cap ceiling enforced per tier (Tier 1 200,000; Tier 2 100,000 in config) | BaseGradeValidatorTest.rejectsAmountExceedingCap, allowsAmountAtCap, rejectsMissingTierCapConfig | BaseGradePresetCreateIT.createRejectsTierCapExceeded | - | 93811 (Tier 1 cap, UI) | IN DEV |
| Seeded product preset is a new immutable preset distinct from Default v1 | - | BaseGradePresetValidationIT (validates seed CSV base_grade_preset_default_v1.csv via repository and GraphQL) | - | none | PARTIAL |

#### 4. LACS Write-Once and Error Handling (HI-7856; LACS PR #9337 merged, PR #8680)

| Requirement | UT | IT | UI unit | E2E (AllureId) | Status |
|---|---|---|---|---|---|
| Stored preset id cannot be changed to a different value; rejected, stored value unchanged | MerchantProjectServiceTest.should_ThrowException_When_UpdateOneWithChangedBaseGradePresetId (parameterized by status); MerchantProjectFacadeTest.should_PropagateException_When_UpsertRejectsChangedBaseGradePresetIdWhileOffered | none (MerchantProjectControllerIT has no preset-specific case) | - | none | PARTIAL |
| Null or omitted preset id over a stored value is rejected, not silently cleared | MerchantProjectServiceTest.should_ThrowException_When_IncomingBaseGradePresetIdIsNullOverAStoredValue | none | - | none | PARTIAL |
| Write in a disallowed loan-app state returns 4xx, not HTTP 200 with an empty body | MerchantProjectFacadeTest (disallowed status cases) | MerchantProjectControllerIT.should_ReturnBadRequest_When_Upsert_WithDisallowedStatus, should_ReturnBadRequest_When_PartialUpdate_WithDisallowedStatus | - | N/A | COVERED |
| First-time assignment and re-sending the same value are accepted (idempotent) | MerchantProjectServiceTest.should_ReturnMerchantProject_When_AssigningBaseGradePresetIdForTheFirstTime, should_ReturnMerchantProject_When_ResendingTheSameBaseGradePresetId | none | - | 84686 (first assignment) | PARTIAL |
| Preset id is mapped onto the merchant project (mapper level) | MerchantProjectMapperTest, MerchantAtoMapperTest (PR #8680) | none (no persistence IT) | - | 84686 (reads funnel.merchant_project) | COVERED |
| A NEW application after the merchant's preset is reassigned picks up the new preset (write-once does not leak) | - | - | - | 91953 | COVERED |

#### 5. VQ UI: Navigation, List View, Archive (HI-6776, HI-7723, HI-6777, HI-7774)

| Requirement | UT | IT | UI unit | E2E (AllureId) | Status |
|---|---|---|---|---|---|
| Tools > Home Improvement entry appears for managers and is hidden without manager access | - | - | ccp-portal-components-ui Header/__tests__/Header.test.js (+snapshot, PR #304); servicing-ui base-grade-presets/__tests__/Outlet.test.js (PR #87) | 91659, 91660 | COVERED |
| Clicking the nav entry loads the Manage base grade configurations list | - | - | pages/BaseGradePresets/List/__tests__/index.test.js | 91659 | COVERED |
| List loads configs; total count label matches rows | - | - | BaseGradeConfigurationList/__tests__/index.test.js, utils.test.js; pages/BaseGradePresets/List/__tests__/index.test.js | 91368 | COVERED |
| Client-side search filters by configuration name | - | - | BaseGradeConfigurationList/__tests__/index.test.js, utils.test.js | 91368 (large-list filtering not covered) | COVERED |
| Merchants Assigned and Creation Date columns sort on the list | - | - | BaseGradeConfigurationList/__tests__/utils.test.js | none (92252 sorts the Assigned tab, not the list columns) | PARTIAL |
| Clicking a configuration name opens the detail page | - | - | pages/BaseGradePresets/Detail/__tests__/index.test.js | 91368 | COVERED |
| Create new configuration button opens the create form | - | - | BaseGradeConfigurationList/__tests__/index.test.js | 91369 | COVERED |
| Manage merchants button opens the assign page for that configuration | - | - | BaseGradeConfigurationList/__tests__/RowActions.test.js | 91373, 91374 | COVERED |
| Kebab menu shows Duplicate (Archive placeholder pending HI-7774) | - | - | BaseGradeConfigurationList/__tests__/RowActions.test.js; base-grade-presets/__tests__/ConfigurationActionsMenu.test.js | 91372 (Duplicate) | PARTIAL |
| Confirmation banner on the list after create and after assignment | - | - | pages/BaseGradePresets/List/__tests__/index.test.js | 91369, 91373 | COVERED |
| Archive a configuration (HI-7774) | - | - | none (no mutation exists) | none | BLOCKED |

#### 6. VQ UI: Create / View / Duplicate (HI-6778, HI-6779, HI-6780)

| Requirement | UT | IT | UI unit | E2E (AllureId) | Status |
|---|---|---|---|---|---|
| Create form shows 3 FICO accordions (780-850, 660-779, 300-659) | - | - | CreateBaseGradeConfiguration/__tests__/ScoreBandSegment.test.js, index.test.js | 91369 | COVERED |
| Each accordion shows the grade rows (UI today: 1-15 plus a >=16 catch-all = 16 rows) | - | - | CreateBaseGradeConfiguration/__tests__/ScoreBandSegment.test.js, utils.test.js | 91369 (grid entered and persisted) | COVERED |
| Approval Tier dropdown offers only Tier 1, Tier 2, Decline (no Tier 3/4 in v1) | - | - | CreateBaseGradeConfiguration/__tests__/ApprovalTierCell.test.js | 91369 selects Tier 1/Tier 2 labels; absence of Tier 3/4 not asserted | PARTIAL |
| Decline row takes no amount (dash instead of input) | - | - | CreateBaseGradeConfiguration/__tests__/MaxLoanAmountCell.test.js | 93812 (PR #39405) | IN DEV |
| Submitting an incomplete grid shows required errors on every segment | - | - | CreateBaseGradeConfiguration/__tests__/index.test.js | 93812 (PR #39405) | IN DEV |
| Max loan amount above the tier cap is rejected on the form; the cap itself is accepted | BaseGradeValidatorTest.rejectsAmountExceedingCap, allowsAmountAtCap | BaseGradePresetCreateIT.createRejectsTierCapExceeded | CreateBaseGradeConfiguration/__tests__/MaxLoanAmountCell.test.js | 93811 (PR #39405) | IN DEV |
| Duplicate grid content shows the duplicate banner with a working View configuration link | BaseGradePresetServiceTest.createRejectsDuplicateContent | BaseGradePresetCreateIT.createRejectsDuplicateContent | pages/BaseGradePresets/Create/__tests__/index.test.js | 91370 | COVERED |
| Duplicate NAME shows the name-conflict banner | BaseGradePresetServiceTest.createRejectsDuplicateName | BaseGradePresetCreateIT.createRejectsDuplicateName | pages/BaseGradePresets/Create/__tests__/index.test.js | 91371 | COVERED |
| Success redirects to the list with the created banner and the new configuration | - | - | pages/BaseGradePresets/Create/__tests__/index.test.js | 91369 | COVERED |
| Cancel returns to the list without saving | - | - | CreateBaseGradeConfiguration/__tests__/index.test.js | 93812 (PR #39405) | IN DEV |
| Detail page shows tier badges and amounts per segment | - | - | ViewBaseGradeConfiguration/__tests__/index.test.js, SegmentTable.test.js, utils.test.js | 91368 (badge colour not asserted) | COVERED |
| Detail page Assigned merchants (N) link opens the Assigned tab | - | - | ViewBaseGradeConfiguration/__tests__/index.test.js | 92252 | COVERED |
| Duplicate pre-fills the form with '(copy)' name and the source grid | - | - | pages/BaseGradePresets/Duplicate/__tests__/index.test.js | 91372 | COVERED |
| Unmodified duplicate is rejected with the content-duplicate error | BaseGradePresetServiceTest.createRejectsDuplicateContent | BaseGradePresetCreateIT.createRejectsDuplicateContent | CreateBaseGradeConfiguration/__tests__/index.test.js | 91372 | COVERED |
| Editing at least one value makes the duplicate succeed and leaves the source unchanged | - | - | pages/BaseGradePresets/Duplicate/__tests__/index.test.js | 91372 | COVERED |
| Mixed grid: adjacent identical rows merge into one range, non-adjacent rows stay separate (UI merges, MPDS does not) | BaseGradeValidatorTest.acceptsMultipleContiguousRanges | - | CreateBaseGradeConfiguration/__tests__/utils.test.js | 95207 (local, uncommitted) | IN DEV |
| Persisted base_grade_preset_range rows match the grid entered in the UI (DB vs UI) | - | - | - | 91368, 91369, 95207 with DB range verification (PR #39405 branch) | IN DEV |
| Name and description validated before submit (@ValidText mirror) and name maxLength 255 (HI-8376) | - | - | none yet (HI-8376, no PR) | none | GAP |
| Replace configuration confirm button is destructive red (HI-8376) | - | - | ManageMerchants/__tests__/ReplaceConfigurationModal.test.js (snapshot, current green state) | 91374 asserts the modal, not the colour | GAP |
| Form offers 20 base-grade rows per spec (UI MAX_BASE_GRADE=16 today; product confirmation pending) | BaseGradeValidatorTest.acceptsMinGradeAtDomainCeiling (backend supports 20) | BaseGradePresetRangeConstraintIT (DB allows up to 20) | none | none | GAP |

#### 7. VQ UI: Assign to Merchants and Merchant Servicing (HI-6781, HI-6782, HI-8087)

| Requirement | UT | IT | UI unit | E2E (AllureId) | Status |
|---|---|---|---|---|---|
| Tier-enablement warning banner shown on Manage merchants with an Enable tiers link | - | - | ManageMerchants/__tests__/index.test.js | 91373 (banner shown; link navigation not asserted) | COVERED |
| Merchant search by name, account ID and email independently | - | - | ManageMerchants/__tests__/MerchantSearch.test.js | 91373 (account ID), 92146 (email, name) | COVERED |
| Results show current assignment (Not assigned or existing configuration name) | - | - | ManageMerchants/__tests__/utils.test.js | 91373, 91374 | COVERED |
| Select all unions into the selection across pages | - | - | ManageMerchants/__tests__/AssignMerchants.test.js | 92146 (single result page only) | PARTIAL |
| Load more merchants paginates with the Relay cursor | - | - | ManageMerchants/__tests__/index.test.js | none | GAP |
| Selected count and Clear selections update | - | - | ManageMerchants/__tests__/AssignMerchants.test.js | 92146 | COVERED |
| Assigning over an existing different configuration shows the Replace configuration modal | - | - | ManageMerchants/__tests__/ReplaceConfigurationModal.test.js | 91374 | COVERED |
| Confirm replaces; Cancel dismisses with no change | - | - | ManageMerchants/__tests__/ReplaceConfigurationModal.test.js | 91374 | COVERED |
| Re-assigning the SAME configuration is a no-op and shows no modal | - | - | none identified | none | GAP |
| Batch including a Tier-2-ineligible merchant: whole batch rejected, selection kept for retry | - | - | ManageMerchants/__tests__/AssignMerchants.test.js | 91375 (single ineligible merchant; mixed batch not asserted) | PARTIAL |
| Assigned merchants tab lists merchants; count updates after assignment; client-side search and Date Assigned sort | - | - | ManageMerchants/__tests__/AssignedMerchants.test.js (PR #144) | 91373, 92252 | COVERED |
| After assignment the list shows the '{Config} was successfully assigned to N merchants' banner and the count updates | - | - | pages/BaseGradePresets/Merchants/__tests__/index.test.js | 91373 | COVERED |
| Merchant Servicing Risk Controls row shows None, edit modal assigns a configuration, row updates (HI-8087) | - | - | merchant-servicing/__tests__/BaseGradeConfigurationEditor.test.js, BaseGradeConfigurationItem.test.js, RiskControlSection.test.js; utils/__tests__/base-grade-presets.test.js (PR #155) | 92672 | COVERED |
| Merchant Servicing edit keeps a Tier 2 rejection inside the modal; modal stays open; Cancel leaves it unchanged (HI-8087) | - | - | merchant-servicing/__tests__/BaseGradeConfigurationEditor.test.js | 92673 | COVERED |

#### 8. Cross-Service and Spec Requirements

| Requirement | UT | IT | UI unit | E2E (AllureId) | Status |
|---|---|---|---|---|---|
| Base grade locked at application creation: reassigning the merchant preset does not change an existing application | MerchantProjectServiceTest (write-once cases) | - | - | 91953 (API); 84686 | COVERED |
| Merchant base-grade setting changes are logged and queryable via EDW | MerchantStatusEventPublisherTest (event carries baseGradePresetId), ActivityCommandPublisherTest | none (no IT proves the Kafka publish; EDW load of these topics unverified) | - | none (activity record asserted via verifyBaseGradeAssignmentPersisted; no EDW check) | PARTIAL |
| CDS calculates the grade, cascades Tier 1 to Tier 2 on decline, caps max loan amount before accommodations, global MaxHIRM1 | HiClCreditDecisionServiceTest, MerchantPlanDefinitionServiceTest (CDS PR #10227, query only) | none | - | 84688 (@SkipUntil all envs, CRD-17736 In Development); CreditDecisionHiclLockingTest (Decisioning track, WireMock) | BLOCKED |
| ARIX field16: consumer_final_grade, base_grade_preset_id/range_id, max amount/tier, loan_amount_to_hh_income, payment_to_income | Field16PayloadSizeTest, ApplicantAdditionalInformationStoEnricherTest, ApplicationAdditionalInformationStoEnricherTest, CrbFeatureFlagBindingTest, SubLineOnboardingServiceTest, CreditDecisionServiceTest | MasterLineOnboardingServiceIT, SubLineOnboardingCommandHandlerIT | - | 84688 (skipped on all envs, CRD-17736) | BLOCKED |
| CDS ranges() sends the required baseGradePresetId (HI-8096) | HiClCreditDecisionServiceTest, MerchantPlanDefinitionServiceTest | none (no contract or integration test against MPDS) | - | none (no E2E drives a HICL decision with a preset; 84688 skipped) | PARTIAL |
| Seeded Default v1 preset matches the spec sample table | - | BaseGradePresetValidationIT validates the seed CSV, not the spec table | - | none | GAP |
| Tier 2 cap: spec text says 35,000, backend config says 100,000 (TIER_TWO) | BaseGradeValidatorTest.rejectsAmountExceedingCap | BaseGradePresetCreateIT.createRejectsTierCapExceeded | - | 93811 covers the Tier 1 cap only | PARTIAL |
| Base grade display on the VQ offer page (open product question) | - | - | - | none | GAP |
| Turning Tier 2 off while the merchant holds a Tier 2 preset (open product question) | - | - | - | none | GAP |


## Active Gaps

### High Priority
- [HIGH] **CRD-17736 (HI Engine baseGrade policy v33) is still In Development**, so the full cross-service journey 84688 is `@SkipUntil` on every env and CDS grade calculation, tier cascade, max-loan cap and ARIX field16 values cannot be proven end to end. HIDS#3715 is now merged (2026-09-30); the remaining blocker is CDS. HI-6657 and the CDS row stay PARTIAL/BLOCKED until then.
- [HIGH] **Spec v14 conflicts with the shipped config and has no owner.** Spec table vs Default v1 seed (HI-6798), 35,000 vs 100,000 Tier 2 cap, Tier 3 in a v1 table, and a Tier 2 amount above the Tier 2 cap. Test expectations cannot be written against the spec until product reconciles it.
- [HIGH] **20 base-grade rows.** Spec prose says 1-20, the form shows 16 rows (MAX_BASE_GRADE=16), backend supports 20. Waiting on product; HI-8376 holds the FE change. When it ships, the UI create/detail E2E (91369, 91368, 95207, 93811, 93812) and the jest ScoreBandSegment/utils tests need updating together.

### Medium Priority
- [MEDIUM] **EDW logging was previously reported as "no story/code exists" — corrected.** HMS tracks `baseGradePresetId` and `baseGradePresetAssignedDateTime` in `MerchantConfigurationUpdatedEvent.TRACKED_FIELDS`, published to Kafka (`MerchantStatusEventPublisher`, `HomeImprovementMerchantConfigurationChangedEvent` on MERCHANT_STATUS_CHANGED_EVENT) and to the Activity service (HOME_IMPROVEMENT_MERCHANT_CONFIG_CHANGED with a `baseGradePresetId` attribute; qa-automation `verifyBaseGradeAssignmentPersisted` checks it). CDS persists `offer.base_grade` (CRD-18293), `lockedMerchantBaseGradePresetId`, merchant base grade max amount/tier; LACS `merchant_project` and the HMS project carry `base_grade_preset_id`; HIDS emits ARIX field16. Not verified: whether those tables/topics are loaded into the warehouse (EDW). No ticket owns the EDW requirement, and HI-6768's dedicated PRESET_ASSIGNED/PRESET_REMOVED activity types are defined but never emitted.
- [MEDIUM] **HI-8376 (new, In Development)** carries four FE fixes with no PR yet and no automated check: Replace configuration button colour, `@ValidText` mirror for name/description, name maxLength 255, and the open question of inline tier-cap validation (BE already rejects; 93811 covers the Tier 1 rejection once #39405 merges).
- [MEDIUM] **PR #39405 not merged**: 93811, 93812, DB-vs-UI range verification, the removal of UI `@SkipUntil` gates and the 4-test continuous group are all still IN DEV; 95207 (mixed-grid merge) is local only and in no PR.
- [MEDIUM] **No IT proves the Kafka activity/config-changed event publishes** on assign/remove (UT with mocked publishers only) — HI-6774's T6 gap, still open.
- [MEDIUM] **HMS IT `BaseGradePresetMutationIT` mocks `MerchantPlanDefinitionService`**, so nonexistent-preset and MPDS-unreachable paths are UT only; the allow path with Tier 2 enabled and a mixed multi-merchant batch have no IT or E2E.
- [MEDIUM] **LACS write-once has no controller-level IT** (only disallowed-status cases in `MerchantProjectControllerIT`); different-value, null-over-stored and idempotent resend are UT only, and no E2E attempts a conflicting write.
- [MEDIUM] **MPDS duplicate-race has no concurrency test** (HI-7855 AC3); duplicates are only tested sequentially. URN serialization (AC7) is confirmed in `BaseGradePresetCreateIT` — the earlier "unconfirmed" gap is closed.
- [MEDIUM] **HI-8096 has UT but no IT/contract test**; no E2E drives a HICL decision with a preset (84688 skipped), so the `ranges()` fix is proven only by unit tests.
- [MEDIUM] **UI E2E holes**: "Load more merchants" pagination, re-assigning the same configuration (no-op), list-column sorting, cross-page select-all union, mixed multi-merchant tier-ineligible batch, Tier 3/4 absence in the dropdown, Enable tiers link navigation.

### Low Priority
- [LOW] **HI-7774 (Archive)** is blocked on a product decision — no BE mutation exists, nothing to test.
- [LOW] **qa-automation 84688 gate text** still cites hms PR-5791 and HIDS PR-3715; both are merged, so only CRD-17736 remains (reason string is stale).
- [LOW] **Continuous-test gate**: only 91368, 91369, 91373, 92672 gate continuous runs once #39405 merges; the other UI tests (13 once #39405 merges, 14 with 95207) run only in the base-grade suite.

### Spec Requirement Gaps
- [HIGH] CDS decisioning (grade calculation, tier cascade, max-loan cap before accommodations, global MaxHIRM1) — NEW STATUS: owned by CRD-17736 (In Development) plus the Decisioning track CRD-17733/CRD-18201; no live E2E possible yet.
- [HIGH] NEW (spec v14 review): sample table vs Default v1 seed, Tier 2 cap 35,000 vs 100,000, Tier 3 and over-cap rows in the spec table — spec/seed contradiction, no ticket.
- [MEDIUM] NEW: 20 base-grade rows in the form (UI shows 16) — HI-8376 pending product.
- [MEDIUM] CHANGED: EDW audit logging — HIGH to MEDIUM. Implemented via the generic config-changed event and activity records; EDW ingestion unverified; still no owning ticket.
- [MEDIUM] NEW: base grade display on the VQ offer page — open product question, no ticket.
- [MEDIUM] NEW: turning Tier 2 off while a Tier 2 preset is assigned — open product question, no ticket (HMS guard only checks at assignment time).
- [MEDIUM] Spec: "Agents can view base grades before assigning" — COVERED (91368, 91373 views the configuration and its assignments).
- [MEDIUM] Spec: "Tier must be enabled and pricing preset assigned before assigning a preset containing it" — Tier-enabled condition COVERED (84687, 91375, 92673); the pricing-preset second condition is PARTIAL (UT only).
- [LOW] Archive lifecycle (HI-7774) — open product questions (restore? archived filter? effect on outstanding offers?).

## PR Analysis

### merchant-plan-definition-srvc#2321 — HI-6646: base grade preset create mutation
**Status**: MERGED. Full preset CRUD, score groups/bands, entities, validator, GraphQL layer.
**Tests**: `BaseGradeConfigIT`, `BaseGradePresetTest`, `BaseGradePresetMapperTest`, resolver tests (×5), `BaseGradeMutationResolverTest`, `BaseGradePresetCreateIT`, `BaseGradeQueryIT`, `BaseGradeQueryResolverTest`, `BaseGradePresetServiceTest`, `BaseGradeScoreServiceTest`, `BaseGradeValidatorTest`.
**Verdict**: UT/IT Done — the most thoroughly tested PR in the epic.

### merchant-plan-definition-srvc#2391 — HI-6798: seed first preset via Liquibase
**Status**: MERGED. Adds `BackfillBaseGradePresetHashContentJob`, the seed changelog and `base_grade_preset_default_v1.csv` test resource.
**Tests**: `BaseGradePresetValidationIT` (validates the seed CSV via repository and GraphQL), `BaseGradePresetCreateIT`, `BaseGradeQueryIT` (updated).
**Verdict**: Partial — seed path covered; the backfill job has no dedicated test. Re-checked 2026-10-14: no tests added.

### merchant-plan-definition-srvc#2727 — HI-7590 (MPDS half): add `containedTiers`
**Status**: MERGED. **Tests**: `BaseGradePresetResolverTest.containedTiers`, `BaseGradeQueryIT` updated. **Verdict**: Done.

### merchant-plan-definition-srvc#2833 — HI-7855/HI-7854: domain + decision-consistency integrity
**Status**: MERGED (the 2026-09-29 entry said OPEN). Adds the grade ≤20 DB CHECK, GraphQL enum-drift guard, tier-cap config, validator changes.
**Tests** (re-checked 2026-10-14 from method names): `BaseGradePresetRangeConstraintIT` (3 tests: rejects min grade above ceiling, rejects zero-amount approved range, accepts min grade at ceiling), `GraphQLEnumConsistencyTest` (errorCode, merchantPlanTierName, merchantType), `BaseGradePresetCreateIT` (+97/−16; asserts the conflict error's `existingPresetId` is a URN string), `BaseGradeValidatorTest` (+120), `BaseGradePresetServiceTest`.
**Verdict**: Partial — AC1, AC6, AC7 confirmed. AC3 (duplicate race) is covered only sequentially; no concurrency test.

### home-improvement-merchant-srvc#5188 — HI-6656/HI-6646: assignment + moved enum
**Status**: MERGED. **Tests**: `BaseGradePresetMutationIT`, `MerchantConfigurationServiceTest`, `ActivityCommandPublisherTest`, `ProjectServiceTest`, plus refactor-safety tests from the `MerchantPlanTierName` move. **Verdict**: Done.

### home-improvement-merchant-srvc#5791 — HI-7590 (HMS half): Tier 2 guard
**Status**: MERGED (2026-09-29 entry said OPEN). **Tests**: `BaseGradePresetMutationIT` (assign, reject without required tier, replacement, remove), `MerchantPlanDefinitionServiceTest` (`getBaseGradePresetContainedTiers*`), `MerchantPlanTierServiceTest` (`hasTiersEnabled_*`, `findMissingTiers_*`), `MerchantConfigurationServiceTest` (`assignBatchRejectsMerchantMissingRequiredTier`, `assignBatchReportsMissingTiersForEveryIneligibleMerchant`).
**Verdict**: Partial — core reject path covered at UT, IT and E2E; the IT mocks `MerchantPlanDefinitionService`, so nonexistent-preset and MPDS-unreachable paths are UT only.

### home-improvement-merchant-srvc#5951 — HI-7857: producer wiring + tier-guard fixes
**Status**: MERGED (2026-09-29 entry said OPEN). **Tests**: `MerchantProjectFactoryTest` (baseGradePresetId populated / null / sourced from project snapshot), `MerchantConfigurationServiceTest` (`doesNotCopyBaseGradePresetOrItsAssignedTimestamp`), `MerchantPlanDefinitionServiceTest`, `CustomerOnboardingServiceTest`, `MerchantPlanTierServiceTest`.
**Verdict**: Partial — UT strong; no IT for per-merchant rejection detail (E2E 84687 covers it) and none for franchise-clone re-validation (the preset is not copied, 84960 covers it).

### home-improvement-disbursement-srvc#3715 — HI-6657: ARIX field16
**Status**: MERGED 2026-09-30 (the 2026-09-29 entry said OPEN). **Tests**: `Field16PayloadSizeTest`, `ApplicantAdditionalInformationStoEnricherTest`, `ApplicationAdditionalInformationStoEnricherTest`, `CrbFeatureFlagBindingTest`, `CreditDecisionServiceTest`, `SubLineOnboardingServiceTest`, `MasterLineOnboardingServiceIT`, `SubLineOnboardingCommandHandlerIT`. **Verdict**: Done at the HIDS layer; E2E still depends on CDS populating `finalGrade` (CRD-17736).

### loan-app-creation-srvc#8680 — HI-6658: baseGradePresetId on merchant project
**Status**: MERGED. **Tests**: `MerchantAtoMapperTest`, `MerchantProjectMapperTest` (mapper UT only). **Verdict**: Partial — no IT for DB persistence of the new column; E2E 84686 and 91953 read `funnel.merchant_project`.

### loan-app-creation-srvc#9337 — HI-7856: write-once + stop-200-on-dropped-write
**Status**: MERGED (2026-09-29 entry said OPEN); prod deploy k8s-template#286559. **Tests**: `MerchantProjectControllerIT` (disallowed-status upsert and partial update return 400; no preset-specific IT case), `MerchantProjectServiceTest` (changed value, null over stored value, first-time assignment, same-value resend), `MerchantProjectFacadeTest`, `MerchantProjectMapperTest`.
**Verdict**: Partial at IT (controller IT has no write-once case), Done at UT.

### credit-decision-srvc#10227 — HI-8096 / CRD-20596: re-land base-grade preset query fix
**Status**: MERGED (HI-8096 Closed). Adds the required `baseGradePresetId` argument to `ranges()` in `get-base-grade-preset-query.graphql`, plus `ServiceErrorCode` and `MerchantPlanDefinitionService` changes.
**Tests**: `HiClCreditDecisionServiceTest`, `MerchantPlanDefinitionServiceTest` (UT only). **Verdict**: UT Done, IT GAP — re-checked 2026-10-14, still no integration/contract test against MPDS.

### activity-events#1410 — HI-6768: base grade preset activity types
**Status**: MERGED. Changes `RecordActivityCommand` and poms only. **Tests**: none in PR (library enum). **Verdict**: N/A for the lib classification, but note the dedicated PRESET_ASSIGNED/REMOVED types are defined and never emitted by HMS.

### home-improvement-servicing-ui#87, #100, #123, #127, #138, #144, #154, #155 — HI-6776/6777/6778/6779/6780/6781/6782/8087
**Status**: all MERGED (#144 and #154 on 2026-10-02, #155 on 2026-10-06); epic PR #164 (`HI-6336-Merchant-Base-Grade`) is OPEN and aggregates them. **UI jest tests found via PR file lists** (paths under `src/components/base-grade-presets/` unless stated):
- #87 create flow: `CreateBaseGradeConfiguration/__tests__/{index,ApprovalTierCell,MaxLoanAmountCell,ScoreBandSegment,utils}.test.js`, `__tests__/Outlet.test.js`, `pages/BaseGradePresets/Create/__tests__/index.test.js`, `pages/BaseGradePresets/List/__tests__/index.test.js`.
- #100 list: `BaseGradeConfigurationList/__tests__/{index,RowActions,utils,styles}.test.js`, `__tests__/styles.test.js`, `pages/BaseGradePresets/Placeholder/__tests__/index.test.js`.
- #123 detail: `ViewBaseGradeConfiguration/__tests__/{index,SegmentTable,utils}.test.js`, `__tests__/ConfigurationActionsMenu.test.js`, `__tests__/utils.test.js`, `pages/BaseGradePresets/Detail/__tests__/index.test.js`.
- #127 duplicate: `pages/BaseGradePresets/Duplicate/__tests__/index.test.js`, `CreateBaseGradeConfiguration/__tests__/{index,utils}.test.js`.
- #138 assign: `ManageMerchants/__tests__/{index,AssignMerchants,MerchantSearch,ReplaceConfigurationModal,utils,styles}.test.js`, `pages/BaseGradePresets/Merchants/__tests__/index.test.js`, `common/TargetedAlerts/__tests__/index.test.js`.
- #144 assigned tab: `ManageMerchants/__tests__/{AssignedMerchants,AssignMerchants,MerchantSearch,index,utils,styles}.test.js`.
- #154 nav: no tests (adds a `package.json`/lockfile dependency bump only); the nav entry itself is `ccp-portal-components-ui#304`.
- #155 merchant servicing: `merchant-profile/sections/merchant-servicing/__tests__/{BaseGradeConfigurationEditor,BaseGradeConfigurationItem,RiskControlSection}.test.js`, `utils/__tests__/base-grade-presets.test.js`.
**Verdict**: UI unit coverage Done for every shipped screen; E2E COVERED for all of them (see Per-requirement checklist).

### ccp-portal-components-ui#304 — HI-7723: Tools nav entry
**Status**: MERGED. **Tests**: `src/Header/__tests__/Header.test.js` (+ snapshot); also changes `featureFlags.js`, `urls.js`, adds a changeset. **Verdict**: Done (UI jest). E2E 91659/91660.

### docs.credify.tech#349 — HI-7859: tech design set (added 2026-10-14 refresh)
**Status**: MERGED (replaces the DECLINED #196). Publishes the HLD, design, impl notes, test plan and finding-verification spike under `teams/homeimprovement/features/HI-6336_merchant-base-grade/`. **Verdict**: N/A (docs) — now a merged secondary source of truth.

### k8s-template#298586 — HI-6657 preprod deploy of home-improvement-disbursement-srvc (added 2026-10-14 refresh)
**Status**: MERGED. Infra only. **Verdict**: N/A.

### qa-automation (Decisioning-owned, separate track) — CRD-17733/CRD-18201
**Status**: MERGED (PRs #33971, #34066, #34483, #34395 on master). **Tests**: `CreditDecisionHiclLockingTest` (`@Owner(DecisioningTeam.MSELA)`), `CreditDecisionMerchantPlanDefinitionMockHelper`, `ApplicationExtendedAttributesSto.lockedMerchantBaseGradePresetId`. **Verdict**: Real but narrow — validates CDS's locking/consumption of a mocked preset. Not tied to any ticket in this epic's tree.

### qa-automation#38176 — HI-6775: cross-service E2E for Merchant Base Grade
**Status**: MERGED 2026-09-29 (merge `e0d8617ceb`). `HomeImprovementMerchantBaseGradeTest` (`@Owner(HomeImprovementTeam.BRIMAN)`, `@Jira("HI-6775")`): 84685 (MPDS authoring, HMS assignment, read-back), 84687 (Tier 2 guard reject, `ineligibleMerchants`), 84686 (HMS producer to LACS snapshot), 84960 (clone excludes preset), 84688 (full journey to ARIX field16). **Update 2026-10-14**: on master the `@SkipUntil` gates on 84686 and 84960 are gone (removed by #39042); only 84688 remains gated on every env.

### qa-automation#39042 — HI-6775: VQ UI E2E for Merchant Base Grade (added 2026-10-14 refresh)
**Status**: MERGED 2026-10-07 (merge `23c40dbe69`). Adds `HomeImprovementMerchantBaseGradeUiTest` with 15 UI tests (91368–91375, 92146, 92252, 92672, 92673, 91659, 91660), page objects for the VQ base-grade screens, nav tests, and API addition 91953 (reassignment does not change an existing application). All 15 carry `@SkipUntil` stage/preprod (to 2026-12-31) and the `home-improvement-servicing-ui` group on master.

### qa-automation#39405 — HI-6775: create form tier cap and validation UI tests (added 2026-10-14 refresh)
**Status**: OPEN (branch `HI-6336-BaseGradeEpicUiTests`, head `6c37710daf`). Adds 93811 (Tier 1 cap rejection and cap accepted) and 93812 (incomplete grid, Decline takes no amount, Cancel), DB-vs-UI range verification (`MerchantPlanDefinitionQueries.selectBaseGradeRangesByPresetId`, `HomeImprovementDbUtils.getBaseGradePresetRanges`, `BaseGradePresetRangeDbo`), distinct per-band amounts, removes all base-grade UI `@SkipUntil` gates (API 84688 keeps its CRD-17736 skip), and keeps only 91368, 91369, 91373, 92672 in the continuous group. Local uncommitted: 95207 (mixed-grid merge test, `BaseGradeRowInput`, page-object edits).

### qa-jenkins-jobs#3434 — HI-6775: register `home-improvement-merchant-base-grade-tests` suite
**Status**: MERGED 2026-10-09 (was open on 2026-09-29).

## Key Decisions

- **One preset per merchant, not per-tier.** `merchant_configuration` holds exactly one `base_grade_preset_id`; tiers live inside the preset's ranges (`containedTiers`).
- **Presets are immutable and content-hashed.** There is no update/edit mutation — "Duplicate" is create-with-prefill, and an unmodified duplicate is guaranteed to collide on the content hash (name is not part of the hash).
- **No archive mutation exists in either backend** — HI-7774 is blocked on a product decision.
- **MPDS and LACS had no epic ticket until the design review** — HI-7855 and HI-7856 were opened to close that gap.
- **Producer wiring and the CDS query fix both shipped**: HMS#5951 and credit-decision-srvc#10227 are merged and HI-8096 is Closed; E2E 84686 and 91953 are ungated on master.
- **Grade domain is 1–20 in the backend** (HI-7854 DB CHECK); the UI form is 16 rows pending a product decision, and the UI merges adjacent identical rows while MPDS stores exactly what it receives.
- **CDS/Decisioning-side coverage is a separate track** (CRD-17733/CRD-18201, now also CRD-17736 for the engine baseGrade policy); do not read "CDS blocked" as "no coverage" — mocked locking coverage exists and is merged.
- **EDW logging is implemented via the generic merchant-config-changed event** (not dedicated PRESET_ASSIGNED/REMOVED activity types); the open question is warehouse ingestion, not code.
- **UI E2E test grouping**: only four UI tests are in the continuous-test group `home-improvement-servicing-ui` after #39405 (91368, 91369, 91373, 92672); the rest run only in the base-grade regression suite.

## Notes for SDET

- **Both backend and UI E2E are on master.** New work should extend `HomeImprovementMerchantBaseGradeTest` / `HomeImprovementMerchantBaseGradeUiTest`, not start parallel classes. The only open qa-automation PR is #39405 (branch `HI-6336-BaseGradeEpicUiTests`); 95207 is local and must be committed to that branch to count.
- **First follow-ups**: (1) merge #39405 (93811/93812, DB-vs-UI verification, 4-test CT group); (2) commit 95207; (3) watch CRD-17736 to un-gate 84688 and update its stale reason text; (4) when product confirms 20 rows, update the UI create/detail tests together with HI-8376; (5) get a decision on the EDW requirement and write a data-validation check against the warehouse.
- **Test checklist**: HI-7886 (In Development) is the authoritative per-requirement checklist (UT/IT/UI unit/E2E AllureIds). HI-6774 (Closed) is the Phase 1 checklist.
- **VQ UI lives in** `home-improvement-servicing-ui` (list/create/view/duplicate/assign pages, Merchant Servicing row) **and** `ccp-portal-components-ui` (Tools nav entry, HI-7723); epic PR is servicing-ui#164.
- **Owner conventions**: base-grade UI tests are `@Owner(HomeImprovementTeam.YCHAUHAN)`, API cross-service tests `@Owner(HomeImprovementTeam.BRIMAN)`, CDS-side locking test `@Owner(DecisioningTeam.MSELA)`.
- **Before exercising the producer-wiring path on a shared env**, confirm credit-decision-srvc#10227 is deployed there.
