---
epic: HI-6336
title: Merchant Base Grade
spec_url: https://credify.atlassian.net/wiki/spaces/PROD/pages/4923457872/Home+Improvement+Base+Grade
status: in-development
last_refreshed: 2026-08-20
test_checklist_ticket: HI-7886
confluence_page_id: PENDING
---

# HI-6336 — Merchant Base Grade

## Spec Summary

HI Credit wants a fine-grained risk categorization ("base grade") with merchant-level line-amount caps. The formula (HIRM1, BK1, and other credit variables) produces one of 10 (addressable range 1–20) base grades per borrower, scored against 3 FICO bands (780–850, 660–779, 300–659). Each merchant gets a base grade "preset" — a matrix mapping (base grade × FICO band) → {Tier 1 / Tier 2 / Decline} plus a max loan amount, capped at the tier's overall ceiling ($200k Tier 1, $35k Tier 2 in v1; Tier 3/4 are explicit future scope). VQ agents create presets (Default / High-Risk, or custom), assign them to merchants (single preset per merchant, not per-tier), and must be able to view a preset before assigning it. A hard business rule guards assignment: a preset referencing a Tier must not be assignable to a merchant that doesn't have that Tier enabled + a pricing preset assigned for it. Base grade is locked at application creation — reassigning a merchant's preset must not affect outstanding offers, only go-forward applications. Changes must be logged and queryable via EDW. During decisioning, CDS calculates the borrower's grade, runs Tier 1/2 decisioning, cascades to higher tiers on decline until an offer/counteroffer is produced, caps the max loan amount from the base grade before accommodations run, and still applies a global MaxHIRM1 check. ARIX field16 gains 5 new reporting fields (consumer_final_grade, base_grade_preset_id/range_id, loan_amount_to_hh_income, payment_to_income).

The epic spans 6 services (MPDS, HMS, LACS, HIDS, activity-events, CDS) plus 2 UI repos (`home-improvement-servicing-ui`, `ccp-portal-components`). Per HI-7859 (the epic's own retroactive tech-design ticket), the epic was originally handed over as "backend done, frontend remaining" — that framing was wrong: MPDS and LACS had **no epic ticket at all** until HI-7855/HI-7856 were opened to cover real defects found in review (silent domain-boundary bugs, a write returning HTTP 200 on a dropped update, duplicate-race errors, ID serialization). The true remaining shape of the work is backend hardening + a from-scratch VQ UI + QA automation, not "frontend only."

**Zero E2E automation exists for anything under this epic in `qa-automation`** — no PR, no branch, no page objects for the VQ base-grade screens. The one related E2E coverage that does exist (`CreditDecisionHiclLockingTest`, `CreditDecisionMerchantPlanDefinitionMockHelper`) is owned by the **Decisioning team** under separate `CRD-17733`/`CRD-18201` tickets — it validates CDS's WireMock-mocked consumption/locking of a base grade preset, not the live cross-service flow, and is not tied to any HI-6336 ticket.

## Ticket Map

| Ticket | Summary | Status | UT | IT | E2E | PRs |
|---|---|---|---|---|---|---|
| HI-6636 | [BE] Tech Design | Closed | N/A | N/A | N/A | — (design only) |
| HI-6646 | [BE] MPDS implementation (preset CRUD, score groups/bands) | Closed | Done | Done | GAP | merchant-plan-definition-srvc#2321, hi-common-lib#516, home-improvement-merchant-srvc#5188 |
| HI-6656 | [BE] HMS merchant assignment | Closed | Done | Done | GAP | home-improvement-merchant-srvc#5188, activity-events#1410, spicedb-schemas#1355 |
| HI-6657 | [BE] HIDS ARIX field16 | Ready for CodeReview | Done | Done | PARTIAL | home-improvement-disbursement-srvc#3715 (open) |
| HI-6658 | [BE] LACS merchant project field | Closed | Done | Partial | GAP | loan-app-creation-srvc#8680, loan-app-creation-client#2634 |
| HI-6768 | [BE] activity-events new activity types | Closed | N/A | GAP | GAP | activity-events#1410 |
| HI-6774 | Test coverage checklist (Phase 1) | Closed | N/A | N/A | N/A | — (checklist doc) |
| HI-6775 | QA automation E2E tests | In Design | N/A | N/A | GAP | none — no qa-automation PR exists |
| HI-6776 | Base Grade Config — Navigation & Entry Point | Open | N/A | N/A | GAP | home-improvement-servicing-ui#87 (route exists, nav entry itself pending HI-7723) |
| HI-6777 | Manage Base Grade Configurations — List View | In Validation | N/A | N/A | GAP | home-improvement-servicing-ui#100 |
| HI-6778 | Create New Base Grade Configuration | In Validation | N/A | N/A | GAP | home-improvement-servicing-ui#87 |
| HI-6779 | View Base Grade Configuration — Read-Only Detail | In Validation | N/A | N/A | GAP | home-improvement-servicing-ui#123 |
| HI-6780 | Duplicate Base Grade Configuration | In Validation | N/A | N/A | GAP | home-improvement-servicing-ui#127 |
| HI-6781 | Assign to Merchants — Search & Selection | In Development | N/A | N/A | GAP | no dedicated PR found (Jira Dev panel only surfaced the unrelated list-view PR) |
| HI-6782 | Assign — Overwrite Handling & Assigned Merchants View | Open | N/A | N/A | GAP | none yet |
| HI-6798 | Seed first base grade preset via liquibase | Closed | N/A | Partial | GAP | merchant-plan-definition-srvc#2391 |
| HI-7590 | [BE] Block Tier 2 preset assignment without Tier 2 enabled | Ready for CodeReview | Done | Partial | GAP | home-improvement-merchant-srvc#5791 (open), merchant-plan-definition-srvc#2727 (containedTiers, merged) |
| HI-7723 | [ccp-portal-components] Add nav entry | Open | N/A | N/A | GAP | none identified |
| HI-7774 | Archive a configuration | Blocked | N/A | N/A | N/A | none — no archive mutation exists in either backend, product decision pending |
| HI-7854 | [BE] MPDS require explicit grade 1–20 coverage + DB constraint | Closed | Done | Done | GAP | folded into merchant-plan-definition-srvc#2833 |
| HI-7855 | [BE] MPDS backend gaps (6 items: domain constraint, duplicate-race, ID serialization, schedule seed, tier-cap query, enum-drift) | Ready for CodeReview | Partial | Partial | GAP | merchant-plan-definition-srvc#2833 (open) |
| HI-7856 | [BE] LACS write-once + stop 200-on-dropped-write | Ready for CodeReview | Done | Done | GAP | loan-app-creation-srvc#9337 (open) |
| HI-7857 | [BE] HMS follow-ups (producer wiring, 2nd tier condition, rejection detail, franchise clone) | In Development | Partial | Partial | GAP | home-improvement-merchant-srvc#5951 (open) |
| HI-7859 | [Design] Tech design set (HLD, design, impl notes, test plan, spike) | Ready for CodeReview | N/A | N/A | N/A | docs.credify.tech#196 (open, docs only) |
| HI-7886 | Test coverage checklist (Phase 2) | Open | N/A | N/A | N/A | — (checklist doc, populated 2026-08-20) |

**PR classification summary:** 12 service PRs checked (merchant-plan-definition-srvc ×4, home-improvement-merchant-srvc ×4, home-improvement-disbursement-srvc ×1, loan-app-creation-srvc ×2, plus 1 excluded false-positive), 4 UI PRs (N/A — home-improvement-servicing-ui), 3 declined/abandoned UI PRs (app-by-phone-ui — superseded by the servicing-ui port), 3 client/lib PRs (N/A — loan-app-creation-client, hi-common-lib, activity-events), 1 infra/schema PR (N/A — spicedb-schemas, self-tested via `assertions.yaml`), 1 docs PR (N/A — docs.credify.tech), 0 qa-automation PRs. `hi-application-srvc#527` was surfaced by the Dev panel for HI-6656/HI-6658 but its title/diff ("[HI-5515] Add application domain data model") is unrelated to base grade — excluded as a false positive.

## Coverage Matrix

| Requirement Area | UT | IT | E2E | Notes |
|---|---|---|---|---|
| MPDS: preset CRUD, score groups/bands, name/content duplicate detection | Done | Done | GAP | HI-6646; `BaseGradePresetCreateIT`, `BaseGradeQueryIT`, `BaseGradeValidatorTest`, `BaseGradePresetServiceTest` |
| MPDS: seed initial preset via Liquibase | N/A | Partial | GAP | HI-6798; IT covers validation/create/query, no dedicated test for `BackfillBaseGradePresetHashContentJob` itself |
| MPDS: grade 1–20 domain bound + DB CHECK constraint | Done | Done | GAP | HI-7854 (closed), folded into PR#2833; `BaseGradePresetRangeConstraintIT` new in that PR |
| MPDS: duplicate-race error mapping, URN serialization, GraphQL enum-drift guard, tier-cap query, real schedule seed | Partial | Partial | GAP | HI-7855 (open PR#2833); `GraphQLEnumConsistencyTest` new; DB-constraint IT present; **duplicate-race concurrency test and URN-serialization assertion not confirmed by filename alone — verify PR#2833's IT bodies directly** |
| MPDS: `containedTiers` field | Done | Done | GAP | HI-7590 (MPDS half, merged PR#2727) |
| HMS: assign/batch-assign/remove mutations, federation fields | Done | Done | GAP | HI-6656; `BaseGradePresetMutationIT`, `MerchantConfigurationServiceTest` |
| HMS: activity events on assign/remove | Done | GAP | GAP | HI-6768/HI-6656; `ActivityCommandPublisherTest` (UT only) — still no IT verifying the Kafka event actually publishes, same gap HI-6774 already flagged as T6 |
| HMS: Tier 2 assignment guard (reject/allow, batch fail-fast) | Done | Partial | GAP | HI-7590 (HMS half, open PR#5791); `BaseGradePresetMutationIT` covers the core guard, but per HI-7857's own AC4 note, this IT `@MockitoBean`s `MerchantPlanDefinitionService` so the nonexistent-preset / MPDS-unreachable paths are not really exercised |
| HMS: producer wiring (MerchantProjectFactory sets baseGradePresetId) | Done | Partial | GAP | HI-7857 (open PR#5951); `MerchantProjectFactoryTest` present; per-merchant rejection detail (`ineligibleMerchants`) and franchise-clone re-validation not confirmed present |
| LACS: baseGradePresetId snapshot at application creation | Done | Partial | GAP | HI-6658; `MerchantAtoMapperTest`/`MerchantProjectMapperTest` present, but no dedicated IT for DB persistence — same gap HI-6774 already flagged |
| LACS: write-once enforcement + stop-200-on-dropped-write | Done | Done | GAP | HI-7856 (open PR#9337); `MerchantProjectControllerIT`, `MerchantProjectFacadeTest`, `MerchantProjectServiceTest` all present — best-covered of the open PRs |
| HIDS: ARIX field16 additions | Done | Done | PARTIAL | HI-6657 (open PR#3715); `Field16PayloadSizeTest`, `MasterLineOnboardingServiceIT` present; still depends on CDS populating the source fields |
| CDS: grade calculation, Tier cascade, max-loan-amount cap, MaxHIRM1 | N/A (Decisioning-owned) | N/A (Decisioning-owned) | PARTIAL (separate track) | Owned by Decisioning team, tracked under `CRD-17733`/`CRD-18201`, not this epic's ticket tree; `CreditDecisionHiclLockingTest` (merged) validates grade-locking via WireMock-mocked MPDS responses, `CreditDecisionMerchantPlanDefinitionMockHelper` provides the mock scaffolding — real cross-service (live MPDS+HMS+LACS+CDS) flow is still untested |
| VQ UI: nav entry, list view, create/view/duplicate, assign-to-merchants, archive | N/A | N/A | GAP | HI-6776–6782, HI-7723, HI-7774; all merged/in-review UI work has **zero E2E automation** — no qa-automation PR, no page objects |
| Base grade locked at application creation (spec rule) | N/A | Partial (LACS write-once, HI-7856) | PARTIAL | Decisioning-side locking is tested (`lockingMerchantBaseGradePresetId` pattern exists per the `lockingMerchantTrustStatus` sibling test in `CreditDecisionHiclLockingTest`), but no test exercises the full live chain: reassign merchant preset → verify an in-flight application's already-created offer is unaffected |
| EDW-queryable audit log of merchant base-grade setting changes | N/A | N/A | GAP | No story or PR found addressing this spec requirement at all — needs a BE/data card before it is testable |

## Active Gaps

### High Priority
- [HIGH] **Zero E2E automation for the entire VQ UI** (HI-6776–6782, HI-7723) — 6 shipped/in-review screens (nav, list, create, view, duplicate, assign) with no Playwright coverage in qa-automation. This is the single largest gap in the epic.
- [HIGH] **HI-7857 AC1 (producer wiring) must not merge/enable tests early** — per the ticket's own sequencing note, landing it activates a live CDS query defect (missing `baseGradePresetId` argument on `ScoreBand.ranges`) that will 500 every HICL decision until CDS fixes its query. Any IT covering this path should stay gated behind that CDS fix.
- [HIGH] **No live cross-service test of the "base grade locked at application, doesn't affect outstanding offers" rule.** LACS's write-once guard (HI-7856) and CDS's mocked locking test cover pieces in isolation; nothing exercises: assign preset → create app → reassign merchant's preset → verify the existing app/offer is untouched, end to end.

### Medium Priority
- [MEDIUM] **HI-6774's T6 gap is still open**: no IT verifies the Kafka activity event actually publishes on assign/remove (only UT via mocked publisher).
- [MEDIUM] **HI-6774's LACS persistence gap is still open**: no IT verifies `baseGradePresetId` is actually persisted to the `merchant_project` row end-to-end (only mapper-level UT).
- [MEDIUM] **HI-7855's duplicate-race and URN-serialization ACs are unconfirmed** — PR#2833 has the right test *files* (`BaseGradePresetCreateIT`, `BaseGradePresetServiceTest`) but a filename-level check can't confirm the concurrency scenario or the URN-format assertion are actually in there; needs a direct read of the IT bodies.
- [MEDIUM] **HI-7857's per-merchant rejection detail and franchise-clone re-validation (AC5/AC6) have no confirmed test** in PR#5951.
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

## Key Decisions

- **One preset per merchant, not per-tier.** `merchant_configuration` holds exactly one `base_grade_preset_id`; tiers live inside the preset's ranges (`containedTiers`). Resolved via HI-6781's investigation — no per-tier assignment UI or mutation exists or is planned.
- **Presets are immutable and content-hashed.** There is no update/edit mutation — "Duplicate" is create-with-prefill (HI-6780), and an unmodified duplicate is *guaranteed* to collide on the content hash (name is not part of the hash).
- **No archive mutation exists in either backend** (MPDS or HMS) — HI-7774 is blocked on a product decision before a BE card can even be written.
- **MPDS and LACS had no epic ticket until this design review** — HI-7855 and HI-7856 were opened specifically to close that gap, per HI-7859's own framing correction.
- **HI-7857 AC1 (producer wiring) is sequenced behind CDS.** Landing it activates a currently-dormant CDS query defect; do not treat "producer wiring merged" as a green light for enabling any E2E test that exercises the live CDS fetch.
- **Grade domain is 1–20** (not open-ended) — HI-7854 (closed) added the coverage requirement and DB CHECK; HI-6778's create form was sequenced to ship *after* HI-7854 so it would submit `maxBaseGrade=20` per row correctly.
- **CDS/Decisioning-side coverage is a separate track**, owned by the Decisioning team under `CRD-17733`/`CRD-18201`, not reachable via this epic's Jira ticket tree or Dev-panel links — do not assume "CDS blocked, not started" (as HI-6774/HI-6775 both say) without checking that track directly; some CDS-side mocked coverage already exists and is merged.

## Notes for SDET

- **No qa-automation branch or PR exists for this epic yet.** Starting E2E work means creating a new branch from master (`HI-6336-e2e-tests` or similar) — there is nothing to check out and continue.
- **Test checklist**: HI-7886 (Open) — Phase 2 checklist covering the Tier 2 guard, HMS/MPDS/LACS hardening, and the full VQ UI, written 2026-08-20. HI-6774 (Closed) is the Phase 1 checklist for the original 4 backend stories — still useful for the gaps it flagged that remain open (T6 activity-event IT, LACS persistence IT).
- **VQ UI lives in** `home-improvement-servicing-ui` (list/create/view/duplicate/assign pages) **and** `ccp-portal-components` (the Tools nav entry, HI-7723) — two repos, not one.
- **Owner conventions observed in the code**: HMS base-grade work uses `@Owner` values tied to the HI merchant team; the CDS-side mocked locking test uses `@Owner(DecisioningTeam.MSELA)` — a different team entirely, worth knowing before assuming "the epic's tests" include it.
- **Before writing new E2E tests for the tier guard or producer wiring**, re-check HI-7590/HI-7857's PR status — both were open as of this refresh, and the producer-wiring path specifically must not be exercised until CDS's query fix is confirmed (see Key Decisions).
