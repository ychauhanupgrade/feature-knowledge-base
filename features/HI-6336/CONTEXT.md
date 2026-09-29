---
epic: HI-6336
title: Merchant Base Grade
spec_url: https://credify.atlassian.net/wiki/spaces/PROD/pages/4923457872/Home+Improvement+Base+Grade
status: in-development
last_refreshed: 2026-09-29
test_checklist_ticket: HI-7886
confluence_page_id: "5952569412"
---

# HI-6336 — Merchant Base Grade

## Spec Summary

HI Credit wants a fine-grained risk categorization ("base grade") with merchant-level line-amount caps. The formula (HIRM1, BK1, and other credit variables) produces one of 10 (addressable range 1–20) base grades per borrower, scored against 3 FICO bands (780–850, 660–779, 300–659). Each merchant gets a base grade "preset" — a matrix mapping (base grade × FICO band) → {Tier 1 / Tier 2 / Decline} plus a max loan amount, capped at the tier's overall ceiling ($200k Tier 1, $35k Tier 2 in v1; Tier 3/4 are explicit future scope). VQ agents create presets (Default / High-Risk, or custom), assign them to merchants (single preset per merchant, not per-tier), and must be able to view a preset before assigning it. A hard business rule guards assignment: a preset referencing a Tier must not be assignable to a merchant that doesn't have that Tier enabled + a pricing preset assigned for it. Base grade is locked at application creation — reassigning a merchant's preset must not affect outstanding offers, only go-forward applications. Changes must be logged and queryable via EDW. During decisioning, CDS calculates the borrower's grade, runs Tier 1/2 decisioning, cascades to higher tiers on decline until an offer/counteroffer is produced, caps the max loan amount from the base grade before accommodations run, and still applies a global MaxHIRM1 check. ARIX field16 gains 5 new reporting fields (consumer_final_grade, base_grade_preset_id/range_id, loan_amount_to_hh_income, payment_to_income).

The epic spans 6 services (MPDS, HMS, LACS, HIDS, activity-events, CDS) plus 2 UI repos (`home-improvement-servicing-ui`, `ccp-portal-components`). Per HI-7859 (the epic's own retroactive tech-design ticket), the epic was originally handed over as "backend done, frontend remaining" — that framing was wrong: MPDS and LACS had **no epic ticket at all** until HI-7855/HI-7856 were opened to cover real defects found in review (silent domain-boundary bugs, a write returning HTTP 200 on a dropped update, duplicate-race errors, ID serialization). The true remaining shape of the work is backend hardening + a from-scratch VQ UI + QA automation, not "frontend only."

**Backend E2E now exists; UI E2E still does not.** `qa-automation#38176` (HI-6775) merged 2026-09-29 and added `HomeImprovementMerchantBaseGradeTest` (5 API tests, AllureIds 84685–84688 and 84960). Three of the five carry `@SkipUntil` gates (see Active Gaps — two cite blockers that have since cleared). There are still no page objects or Playwright tests for the VQ base-grade screens. The separate Decisioning-owned coverage (`CreditDecisionHiclLockingTest`, `CreditDecisionMerchantPlanDefinitionMockHelper`, under `CRD-17733`/`CRD-18201`) validates CDS's WireMock-mocked consumption/locking of a preset and is not tied to any HI-6336 ticket.

## Ticket Map

| Ticket | Summary | Status | UT | IT | E2E | PRs |
|---|---|---|---|---|---|---|
| HI-6636 | [BE] Tech Design | Closed | N/A | N/A | N/A | — (design only) |
| HI-6646 | [BE] MPDS implementation (preset CRUD, score groups/bands) | Closed | Done | Done | COVERED | merchant-plan-definition-srvc#2321, hi-common-lib#516, home-improvement-merchant-srvc#5188/#5122 — E2E 84685 (qa-automation#38176) |
| HI-6656 | [BE] HMS merchant assignment | Closed | Done | Done | COVERED | home-improvement-merchant-srvc#5188, activity-events#1410, spicedb-schemas#1355 — E2E 84685 |
| HI-6657 | [BE] HIDS ARIX field16 | In Validation | Done | Done | PARTIAL | home-improvement-disbursement-srvc#3715 (still open) — E2E 84688 exists but is skipped on every env |
| HI-6658 | [BE] LACS merchant project field | Closed | Done | Partial | PARTIAL | loan-app-creation-srvc#8680, loan-app-creation-client#2634 — E2E 84686 (skipped on stage/preprod/main) |
| HI-6768 | [BE] activity-events new activity types | Closed | N/A | GAP | GAP | activity-events#1410 |
| HI-6774 | Test coverage checklist (Phase 1) | Closed | N/A | N/A | N/A | — (checklist doc) |
| HI-6775 | QA automation E2E tests | Ready for CodeReview | N/A | N/A | COVERED (API) / GAP (UI) | qa-automation#38176 (MERGED 2026-09-29, merge `e0d8617ceb`), qa-automation-graphql#1165 (merged), qa-jenkins-jobs#3434 (open — registers the `home-improvement-merchant-base-grade-tests` suite) |
| HI-6776 | Base Grade Config — Navigation & Entry Point | Ready for CodeReview | N/A | N/A | GAP | home-improvement-servicing-ui#154 (draft), ccp-portal-components-ui#304 (open), #87 (merged) |
| HI-6777 | Manage Base Grade Configurations — List View | In Validation | N/A | N/A | GAP | home-improvement-servicing-ui#100 |
| HI-6778 | Create New Base Grade Configuration | In Validation | N/A | N/A | GAP | home-improvement-servicing-ui#87 (app-by-phone-ui#3688/#3693/#3905 declined) |
| HI-6779 | View Base Grade Configuration — Read-Only Detail | In Validation | N/A | N/A | GAP | home-improvement-servicing-ui#123 |
| HI-6780 | Duplicate Base Grade Configuration | In Validation | N/A | N/A | GAP | home-improvement-servicing-ui#127 |
| HI-6781 | Assign to Merchants — Search & Selection | In Validation | N/A | N/A | GAP | home-improvement-servicing-ui#138 (merged) |
| HI-6782 | Assign — Overwrite Handling & Assigned Merchants View | In Validation | N/A | N/A | GAP | home-improvement-servicing-ui#144 (open) |
| HI-6798 | Seed first base grade preset via liquibase | Closed | N/A | Partial | GAP | merchant-plan-definition-srvc#2391 |
| HI-7590 | [BE] Block Tier 2 preset assignment without Tier 2 enabled | In Validation | Done | Partial | COVERED | home-improvement-merchant-srvc#5791 (merged), merchant-plan-definition-srvc#2727 (merged) — E2E 84687 |
| HI-7723 | [ccp-portal-components] Add nav entry | In Validation | N/A | N/A | GAP | ccp-portal-components-ui#304 (open) |
| HI-7774 | Archive a configuration | Blocked | N/A | N/A | N/A | none — no archive mutation exists in either backend, product decision pending |
| HI-7854 | [BE] MPDS require explicit grade 1–20 coverage + DB constraint | Closed | Done | Done | GAP | folded into merchant-plan-definition-srvc#2833 |
| HI-7855 | [BE] MPDS backend gaps (6 items: domain constraint, duplicate-race, ID serialization, schedule seed, tier-cap query, enum-drift) | In Validation | Partial | Partial | GAP | merchant-plan-definition-srvc#2833 (merged) |
| HI-7856 | [BE] LACS write-once + stop 200-on-dropped-write | In Validation | Done | Done | GAP | loan-app-creation-srvc#9337 (merged; deployed to prod via k8s-template#286559) |
| HI-7857 | [BE] HMS follow-ups (producer wiring, 2nd tier condition, rejection detail, franchise clone) | In Validation | Partial | Partial | COVERED (gated) | home-improvement-merchant-srvc#5951 (merged) — E2E 84960 (skipped on stage/preprod/main) |
| HI-7859 | [Design] Tech design set (HLD, design, impl notes, test plan, spike) | Ready for CodeReview | N/A | N/A | N/A | docs.credify.tech#196 (**DECLINED** — design set has no open PR now) |
| HI-7886 | Test coverage checklist (Phase 2) | Open | N/A | N/A | N/A | — (checklist doc, populated 2026-08-20) |
| HI-8087 | Merchant Servicing — Base Grade Configuration row & edit (NEW) | In Validation | N/A | N/A | GAP | home-improvement-servicing-ui#155 (open) |
| HI-8096 | [Bug] CDS `ranges()` omitted `baseGradePresetId` — 500 on HICL decisions (NEW) | In Validation | Done | GAP | GAP | credit-decision-srvc#10227 (merged; re-land of CRD-20596, revert of revert #10052) |

**PR classification summary (2026-09-29 refresh, 38 unique PRs):** 13 service PRs (merchant-plan-definition-srvc ×4, home-improvement-merchant-srvc ×4, home-improvement-disbursement-srvc ×1, loan-app-creation-srvc ×2, credit-decision-srvc ×1 **new**, plus 1 excluded false-positive), 12 UI PRs (N/A — home-improvement-servicing-ui ×8, ccp-portal-components-ui ×1, app-by-phone-ui ×3 declined/superseded by the servicing-ui port), 3 client/lib PRs (N/A — loan-app-creation-client, hi-common-lib, activity-events), 6 infra/schema PRs (N/A — spicedb-schemas, github-terraform, k8s-template ×4), 1 docs PR (N/A — docs.credify.tech#196, now DECLINED), 3 QA repo PRs (qa-automation#38176 merged, qa-automation-graphql#1165 merged, qa-jenkins-jobs#3434 open). 13+12+3+6+1+3 = 38. `hi-application-srvc#527` was surfaced by the Dev panel for HI-6656/HI-6658 but its title/diff ("[HI-5515] Add application domain data model") is unrelated to base grade — excluded as a false positive.

## Coverage Matrix

| Requirement Area | UT | IT | E2E | Notes |
|---|---|---|---|---|
| MPDS: preset CRUD, score groups/bands, name/content duplicate detection | Done | Done | COVERED | HI-6646; E2E 84685 (create via MPDS, read back); `BaseGradePresetCreateIT`, `BaseGradeQueryIT`, `BaseGradeValidatorTest`, `BaseGradePresetServiceTest` |
| MPDS: seed initial preset via Liquibase | N/A | Partial | GAP | HI-6798; IT covers validation/create/query, no dedicated test for `BackfillBaseGradePresetHashContentJob` itself |
| MPDS: grade 1–20 domain bound + DB CHECK constraint | Done | Done | GAP | HI-7854 (closed), folded into PR#2833; `BaseGradePresetRangeConstraintIT` new in that PR |
| MPDS: duplicate-race error mapping, URN serialization, GraphQL enum-drift guard, tier-cap query, real schedule seed | Partial | Partial | GAP | HI-7855 (open PR#2833); `GraphQLEnumConsistencyTest` new; DB-constraint IT present; **duplicate-race concurrency test and URN-serialization assertion not confirmed by filename alone — verify PR#2833's IT bodies directly** |
| MPDS: `containedTiers` field | Done | Done | GAP | HI-7590 (MPDS half, merged PR#2727) |
| HMS: assign/batch-assign/remove mutations, federation fields | Done | Done | COVERED | HI-6656; E2E 84685 covers the list-form assign mutation + MerchantConfiguration read-back (remove is not exercised); `BaseGradePresetMutationIT`, `MerchantConfigurationServiceTest` |
| HMS: activity events on assign/remove | Done | GAP | GAP | HI-6768/HI-6656; `ActivityCommandPublisherTest` (UT only) — still no IT verifying the Kafka event actually publishes, same gap HI-6774 already flagged as T6 |
| HMS: Tier 2 assignment guard (reject/allow, batch fail-fast) | Done | Partial | COVERED | HI-7590 (HMS half, PR#5791 now merged); E2E 84687 covers the reject path, the structured `ineligibleMerchants` extension, `containedTiers` on the created preset, and that no assignment persists after rejection (the allow path and a multi-merchant batch are not asserted); `BaseGradePresetMutationIT` covers the core guard, but per HI-7857's own AC4 note, this IT `@MockitoBean`s `MerchantPlanDefinitionService` so the nonexistent-preset / MPDS-unreachable paths are not really exercised |
| HMS: producer wiring (MerchantProjectFactory sets baseGradePresetId) | Done | Partial | GAP | HI-7857 (open PR#5951); `MerchantProjectFactoryTest` present; per-merchant rejection detail (`ineligibleMerchants`) and franchise-clone re-validation not confirmed present |
| LACS: baseGradePresetId snapshot at application creation | Done | Partial | PARTIAL | HI-6658; E2E 84686 asserts the snapshot on `funnel.merchant_project` but is skipped on stage/preprod/main (runs on ondemand only); `MerchantAtoMapperTest`/`MerchantProjectMapperTest` present, but no dedicated IT for DB persistence — same gap HI-6774 already flagged |
| LACS: write-once enforcement + stop-200-on-dropped-write | Done | Done | GAP | HI-7856 (open PR#9337); `MerchantProjectControllerIT`, `MerchantProjectFacadeTest`, `MerchantProjectServiceTest` all present — best-covered of the open PRs |
| HIDS: ARIX field16 additions | Done | Done | PARTIAL | HI-6657 (open PR#3715); `Field16PayloadSizeTest`, `MasterLineOnboardingServiceIT` present; still depends on CDS populating the source fields |
| CDS: grade calculation, Tier cascade, max-loan-amount cap, MaxHIRM1 | N/A (Decisioning-owned) | N/A (Decisioning-owned) | PARTIAL (separate track) | Owned by Decisioning team, tracked under `CRD-17733`/`CRD-18201`, not this epic's ticket tree; `CreditDecisionHiclLockingTest` (merged) validates grade-locking via WireMock-mocked MPDS responses, `CreditDecisionMerchantPlanDefinitionMockHelper` provides the mock scaffolding — real cross-service (live MPDS+HMS+LACS+CDS) flow is still untested |
| VQ UI: nav entry, list view, create/view/duplicate, assign-to-merchants, archive | N/A | N/A | GAP | HI-6776–6782, HI-7723, HI-7774; all merged/in-review UI work has **zero E2E automation** — no qa-automation PR, no page objects |
| Base grade locked at application creation (spec rule) | N/A | Partial (LACS write-once, HI-7856) | PARTIAL | Decisioning-side locking is tested (`lockingMerchantBaseGradePresetId` pattern exists per the `lockingMerchantTrustStatus` sibling test in `CreditDecisionHiclLockingTest`), but no test exercises the full live chain: reassign merchant preset → verify an in-flight application's already-created offer is unaffected |
| EDW-queryable audit log of merchant base-grade setting changes | N/A | N/A | GAP | No story or PR found addressing this spec requirement at all — needs a BE/data card before it is testable |

## Active Gaps

### High Priority
- [HIGH] **Zero E2E automation for the entire VQ UI** (HI-6776–6782, HI-7723) — 6 shipped/in-review screens (nav, list, create, view, duplicate, assign) with no Playwright coverage in qa-automation. This is the single largest gap in the epic.
- [HIGH] **Stale `@SkipUntil` gates in `HomeImprovementMerchantBaseGradeTest` (qa-automation#38176).** All three carry `skipBefore = "2027-12-31"`, so they are effectively permanent skips until edited:
  - **84686** (snapshot onto `merchant_project`; skipped stage/preprod/main) cites CRD-20596 unmerged. The fix is now merged (credit-decision-srvc#10227 / HI-8096) — gate can likely be lifted once #10227 is confirmed deployed on the target env.
  - **84960** (clone excludes preset; skipped stage/preprod/main) cites hms PR-5951 unmerged. It is now merged (HI-7857) — same: lift after confirming deployment.
  - **84688** (full MPDS→HMS→LACS→CDS→HIDS→ARIX journey; skipped on **every** env including ondemand) cites CRD-17736 (engine returns null base-grade outcome, so HIDS onboards without `finalGrade`) and HIDS PR-3715. #3715 is **still open** and CRD-17736 status was not checked this refresh — this gate is still legitimate.
- [HIGH] **Live CDS defect from HI-7857 AC1 is fixed, not yet proven end-to-end.** The `ranges()` missing-argument 500 (HI-8096) is fixed by credit-decision-srvc#10227 (merged), and HI-7857 producer wiring (#5951) is merged. HI-8096 is still In Validation, and no E2E has run against a deployed fix — 84686 is the natural verifier.
- [HIGH] **Still no live test of "base grade locked at application; reassigning the merchant's preset doesn't affect outstanding offers."** 84686 checks the snapshot is written, but nothing assigns preset A → creates an app → reassigns preset B → verifies the existing app/offer still references A.

### Medium Priority
- [MEDIUM] **HI-6774's T6 gap is still open**: no IT verifies the Kafka activity event actually publishes on assign/remove (only UT via mocked publisher).
- [MEDIUM] **HI-6774's LACS persistence gap is still open**: no IT verifies `baseGradePresetId` is actually persisted to the `merchant_project` row end-to-end (only mapper-level UT).
- [MEDIUM] **HI-7855's duplicate-race and URN-serialization ACs are unconfirmed** — PR#2833 has the right test *files* (`BaseGradePresetCreateIT`, `BaseGradePresetServiceTest`) but a filename-level check can't confirm the concurrency scenario or the URN-format assertion are actually in there; needs a direct read of the IT bodies.
- [MEDIUM] **HI-7857 AC5 (per-merchant rejection detail) is now covered at E2E** by 84687 (asserts `ineligibleMerchants` in the GraphQL error extension). **AC6 (franchise-clone re-validation) has no confirmed test**; 84960 covers only the clone *exclusion* of the preset, and is gated.
- [MEDIUM] **HI-8096 has UT but no IT** — credit-decision-srvc#10227 changes only `HiClCreditDecisionServiceTest` and `MerchantPlanDefinitionServiceTest`; nothing in that PR exercises the `get-base-grade-preset-query.graphql` argument against a real or contract-tested MPDS, which is how the original defect escaped.
- [MEDIUM] **qa-jenkins-jobs#3434 (suite registration) is still open**, so the merged E2E class has no Jenkins regression job yet.
- [MEDIUM] **No test ticket or PR addresses EDW-queryable change logging** for merchant base-grade settings — a spec requirement with no owner anywhere in the ticket tree.

### Low Priority
- [LOW] **HI-7774 (Archive)** is blocked on a product decision (what happens to assigned merchants on archive) — no BE mutation exists yet, nothing to test.
- [LOW] **HI-6781/HI-6782/HI-7723 have no dedicated PR** discoverable via the Jira Dev panel yet (still Open/In Development) — re-run `/feature-context refresh HI-6336` once they're opened.

### Spec Requirement Gaps
- [HIGH] EDW audit logging of merchant base-grade setting changes — spec requirement, no ticket covers it (see above).
- [MEDIUM] Full cascading-tier CDS decisioning logic (Tier 1→2→3 fallback until an offer, max-loan-amount cap before accommodations, global MaxHIRM1 check) — owned by the separate CRD-17733/CRD-18201 track; worth confirming with the Decisioning team whether their coverage is considered complete for this epic's purposes, since it's invisible to this ticket tree.
- [LOW] Archive lifecycle (HI-7774) — open product questions (restore? filter/tab for archived configs? effect on outstanding offers?) mean this can't be scoped for testing yet.

## PR Analysis

### merchant-plan-definition-srvc#2321 — HI-6646: base grade preset create mutation
**Status**: MERGED. Full preset CRUD, score groups/bands, entities, validator, GraphQL layer.
**Tests**: `BaseGradeConfigIT`, `BaseGradePresetTest`, `BaseGradePresetMapperTest`, resolver tests (×5), `BaseGradeMutationResolverTest`, `BaseGradePresetCreateIT`, `BaseGradeQueryIT`, `BaseGradeQueryResolverTest`, `BaseGradePresetServiceTest`, `BaseGradeScoreServiceTest`, `BaseGradeValidatorTest`.
**Verdict**: UT/IT Done — the most thoroughly tested PR in the epic.

### merchant-plan-definition-srvc#2391 — HI-6798: seed first preset via Liquibase
**Status**: MERGED. Adds `BackfillBaseGradePresetHashContentJob` and the seed changelog.
**Tests**: `BaseGradePresetValidationIT`, `BaseGradePresetCreateIT`, `BaseGradeQueryIT` (updated).
**Verdict**: Partial — the seed data path is covered indirectly; the backfill job itself has no dedicated test.

### merchant-plan-definition-srvc#2727 — HI-7590 (MPDS half): add `containedTiers`
**Status**: MERGED. **Tests**: `BaseGradePresetResolverTest`, `BaseGradeQueryIT` updated. **Verdict**: Done.

### merchant-plan-definition-srvc#2833 — HI-7855/HI-7854: domain + decision-consistency integrity
**Status**: OPEN. Adds the grade ≤20 DB CHECK constraint, GraphQL enum-drift guard, decision-consistency changes.
**Tests**: `GraphQLEnumConsistencyTest` (new), `BaseGradePresetRangeConstraintIT` (new), `BaseGradePresetCreateIT`, `BaseGradePresetServiceTest`, `BaseGradeValidatorTest`.
**Verdict**: Partial — AC1 (DB constraint) and AC6 (enum guard) look directly covered by name; AC3 (duplicate-race) and AC7 (URN serialization) need the IT bodies read directly to confirm.

### home-improvement-merchant-srvc#5188 — HI-6656/HI-6646: assignment + moved enum
**Status**: MERGED. **Tests**: `BaseGradePresetMutationIT`, `MerchantConfigurationServiceTest`, `ActivityCommandPublisherTest`, `ProjectServiceTest`, plus broad refactor-safety tests from the `MerchantPlanTierName` move. **Verdict**: Done.

### home-improvement-merchant-srvc#5791 — HI-7590 (HMS half): Tier 2 guard
**Status**: OPEN. **Tests**: `BaseGradePresetMutationIT`, `MerchantPlanDefinitionServiceTest`, `MerchantPlanTierServiceTest`, `MerchantConfigurationServiceTest`.
**Verdict**: Partial — core reject/allow guard looks covered; per HI-7857's own review note, the existing IT mocks `MerchantPlanDefinitionService`, so the "nonexistent preset" and "MPDS unreachable" paths are not truly exercised despite passing.

### home-improvement-merchant-srvc#5951 — HI-7857: producer wiring + tier-guard fixes
**Status**: OPEN. **Tests**: `MerchantProjectFactoryTest`, `MerchantConfigurationServiceTest`, `MerchantPlanDefinitionServiceTest`.
**Verdict**: Partial — producer-wiring unit coverage present; no confirmed test for per-merchant rejection detail (`ineligibleMerchants`) or franchise-clone re-validation (ACs 5–6).

### home-improvement-disbursement-srvc#3715 — HI-6657: ARIX field16
**Status**: OPEN. **Tests**: `Field16PayloadSizeTest`, `ApplicantAdditionalInformationStoEnricherTest`, `ApplicationAdditionalInformationStoEnricherTest`, `CrbFeatureFlagBindingTest`, `MasterLineOnboardingServiceIT`. **Verdict**: Done at the HIDS layer; still depends on CDS populating the underlying fields.

### loan-app-creation-srvc#8680 — HI-6658: baseGradePresetId on merchant project
**Status**: MERGED. **Tests**: `MerchantAtoMapperTest`, `MerchantProjectMapperTest` (mapper UT only). **Verdict**: Partial — no IT for DB persistence of the new column, matching the gap HI-6774 already flagged.

### loan-app-creation-srvc#9337 — HI-7856: write-once + stop-200-on-dropped-write
**Status**: OPEN. **Tests**: `MerchantProjectControllerIT` (new), `MerchantProjectServiceTest`, `MerchantProjectFacadeTest`, `MerchantProjectMapperTest`. **Verdict**: Done — controller-level IT plus service/facade UT, the most complete of the currently-open PRs.

### home-improvement-servicing-ui#87, #100, #123, #127 — HI-6776/6778/6779/6780, HI-6777
**Status**: All MERGED (create flow ported from app-by-phone-ui, list view, view detail, duplicate). **Tests**: N/A per classification (frontend). **E2E**: GAP — nothing in qa-automation exercises any of these screens.

### docs.credify.tech#196 — HI-7859: tech design set
**Status**: OPEN. Publishes `hld.md`, `design.md`, `impl-notes.md`, `test_plan.md`, `spike-finding-verification.md` under `teams/homeimprovement/features/HI-6336_merchant-base-grade/`. **Verdict**: N/A (docs) — but useful as a secondary source of truth once merged; the ticket's own framing ("backend done, frontend remaining" was wrong) is the reason HI-7855/HI-7856/HI-7857 exist at all.

### qa-automation (Decisioning-owned, separate track) — CRD-17733/CRD-18201
**Status**: MERGED (PRs #33971, #34066, #34483, #34395 on master). **Tests**: `CreditDecisionHiclLockingTest` (`@Owner(DecisioningTeam.MSELA)`), `CreditDecisionMerchantPlanDefinitionMockHelper`, `ApplicationExtendedAttributesSto.lockedMerchantBaseGradePresetId`. **Verdict**: Real but narrow — validates CDS's own locking/consumption of a mocked base grade preset. Does not cover MPDS/HMS/LACS real integration, nor any HI-6336-ticket-tracked scenario. Not tied to any ticket in this epic's tree.

### qa-automation#38176 — HI-6775: cross-service E2E for Merchant Base Grade (added 2026-09-29 refresh)
**Status**: MERGED 2026-09-29T14:04Z (merge `e0d8617ceb`); 8 files, +768/−1. Also touches `HomeImprovementMerchantService`, `MerchantPlanDefinitionService`, `DecisioningQueries` (+ `.properties`), `HomeImprovementTeam`, `HomeImprovementFeature`, and adds suite `home-improvement-merchant-base-grade-tests.xml`. Companion: qa-automation-graphql#1165 (merged), qa-jenkins-jobs#3434 (open).
**Tests** (`HomeImprovementMerchantBaseGradeTest`, `@Owner(HomeImprovementTeam.BRIMAN)`, `@Jira("HI-6775")`):
- 84685 — MPDS preset authoring → HMS assignment → MerchantConfiguration read-back. Ungated.
- 84687 — TIER_TWO preset assigned to a merchant without Tier 2 is rejected; asserts structured `ineligibleMerchants` extension and that no assignment persists. Ungated.
- 84686 — HMS producer → LACS snapshot onto `funnel.merchant_project`. `@SkipUntil` stage/preprod/main (CRD-20596 — now fixed).
- 84960 — merchant clone does not inherit the preset. `@SkipUntil` stage/preprod/main (hms#5951 — now merged).
- 84688 — full MPDS→HMS→LACS→CDS→HIDS→ARIX field16 journey. `@SkipUntil` on all envs (CRD-17736, HIDS#3715 open).
**Verdict**: 2 of 5 run everywhere; 2 have stale gates; 1 is legitimately blocked. API/backend only — no UI coverage.

### credit-decision-srvc#10227 — HI-8096 / CRD-20596: re-land base-grade preset query fix (added 2026-09-29 refresh)
**Status**: MERGED (re-land of the fix reverted in #10052). Adds the required `baseGradePresetId` argument to `ranges()` in `get-base-grade-preset-query.graphql`, plus `ServiceErrorCode` and `MerchantPlanDefinitionService` changes.
**Tests**: `HiClCreditDecisionServiceTest`, `MerchantPlanDefinitionServiceTest` (UT only).
**Verdict**: UT Done, IT GAP — no integration/contract test exercises the query against MPDS.

### home-improvement-servicing-ui#138/#144/#154/#155 and ccp-portal-components-ui#304 — HI-6781/6782/6776/8087/7723 (added 2026-09-29 refresh)
**Status**: #138 MERGED (manage-merchants search/selection/assignment); #144 OPEN (assigned-merchants tab); #154 DRAFT (nav entry in servicing-ui); #155 OPEN (config row + edit modal in Merchant Servicing, HI-8087); ccp-portal-components-ui#304 OPEN (Tools nav entry). **Tests**: N/A per classification. **E2E**: GAP for all.

## Key Decisions

- **One preset per merchant, not per-tier.** `merchant_configuration` holds exactly one `base_grade_preset_id`; tiers live inside the preset's ranges (`containedTiers`). Resolved via HI-6781's investigation — no per-tier assignment UI or mutation exists or is planned.
- **Presets are immutable and content-hashed.** There is no update/edit mutation — "Duplicate" is create-with-prefill (HI-6780), and an unmodified duplicate is *guaranteed* to collide on the content hash (name is not part of the hash).
- **No archive mutation exists in either backend** (MPDS or HMS) — HI-7774 is blocked on a product decision before a BE card can even be written.
- **MPDS and LACS had no epic ticket until this design review** — HI-7855 and HI-7856 were opened specifically to close that gap, per HI-7859's own framing correction.
- **HI-7857 AC1 (producer wiring) was sequenced behind CDS — both have now merged.** #5951 (producer wiring) and credit-decision-srvc#10227 (the `ranges()` argument fix, HI-8096) are merged, but HI-8096 is still In Validation. Confirm the fix is deployed on the target env before treating the live CDS fetch as safe to exercise.
- **Grade domain is 1–20** (not open-ended) — HI-7854 (closed) added the coverage requirement and DB CHECK; HI-6778's create form was sequenced to ship *after* HI-7854 so it would submit `maxBaseGrade=20` per row correctly.
- **CDS/Decisioning-side coverage is a separate track**, owned by the Decisioning team under `CRD-17733`/`CRD-18201`, not reachable via this epic's Jira ticket tree or Dev-panel links — do not assume "CDS blocked, not started" (as HI-6774/HI-6775 both say) without checking that track directly; some CDS-side mocked coverage already exists and is merged.

## Notes for SDET

- **Backend E2E is merged** (qa-automation#38176, 2026-09-29) — new work should branch from current master and extend `HomeImprovementMerchantBaseGradeTest`, not start a parallel class. There is no open qa-automation PR for this epic; UI E2E (VQ base-grade screens) would need a new branch and page objects.
- **First follow-ups**: (1) confirm credit-decision-srvc#10227 and hms#5951 are deployed to stage/preprod, then drop the stale `@SkipUntil` on 84686 and 84960 — and consider whether the remaining 84688 gate should stay, since the standing preference is to let tests fail rather than gate them on unshipped features; (2) get qa-jenkins-jobs#3434 merged so the suite runs on Jenkins.
- **Test checklist**: HI-7886 (Open) — Phase 2 checklist covering the Tier 2 guard, HMS/MPDS/LACS hardening, and the full VQ UI, written 2026-08-20. HI-6774 (Closed) is the Phase 1 checklist for the original 4 backend stories — still useful for the gaps it flagged that remain open (T6 activity-event IT, LACS persistence IT).
- **VQ UI lives in** `home-improvement-servicing-ui` (list/create/view/duplicate/assign pages) **and** `ccp-portal-components` (the Tools nav entry, HI-7723) — two repos, not one.
- **Owner conventions observed in the code**: HMS base-grade work uses `@Owner` values tied to the HI merchant team; the CDS-side mocked locking test uses `@Owner(DecisioningTeam.MSELA)` — a different team entirely, worth knowing before assuming "the epic's tests" include it.
- **HI-7590/HI-7857 PRs (#5791, #5951) are merged** as of 2026-09-29. Before exercising the producer-wiring path on a shared env, confirm CDS's query fix (#10227) is deployed there (see Key Decisions).
