---
epic: HI-7540
title: HI Account Closure in Agent Tool
spec_url: https://credify.atlassian.net/wiki/spaces/PROD/pages/5614305300/HI+Account+Closure+in+Agent+Tool
status: in-development
last_refreshed: 2026-11-17
qa_automation_pr: ""
test_checklist_ticket: ""
confluence_page_id: "6287327249"
---

# HI-7540 — HI Account Closure in Agent Tool

## Spec Summary

Today, closing a Home Improvement loan happens in Spectrum and bundles two activities: bureau reporting (CBR) and Spectrum closure. Accounts get reopened when they should not be (batch work whenever HIMS is out of sync), and when a merchant refund lands after closure agents must reopen, apply the refund, and re-close — the final close sometimes gets missed. This epic moves the flow into VQ (home-improvement-servicing-ui) and splits the two activities: bureau reporting first, Spectrum closure later as a separate step.

Three closure types, each with a different trigger: **Paid Off** (automation reports to the bureau on the 8th business day after PIF, CBR status 13; VQ shows a "Seasoning Period" until PIF + 15 days; agent Close Account only after Day 15; automation closes at Day 60), **Loan Cancellation/Withdrawal** and **Dispute** (agent reports via the VQ dropdown with a reason of Loan cancellation or Dispute; the system derives With Draws vs No Draws and maps to CBR code DA; Close Account is unlocked after reporting, and for With Draws the agent waits for the merchant refund to settle). Close Account uses a confirm modal with an acknowledgement checkbox and moves the row to Closed with timestamp and reason. VQ also shows a "pending closure" flag from Activity 1 until Activity 2 completes.

Surfaces: Credit Bureau dropdown options (P1), Spectrum Closure button state (P1), pending-closure flag (P1), Paid Off pending state (P1). VQ permissions and audit tracking are listed in the spec but left empty.

## Ticket Map

| Ticket | Summary | Status | PR(s) | UT | IT | E2E |
|---|---|---|---|---|---|---|
| HI-8321 | [BE] activity-events: HI closure and CBR-reported event types | In Validation | none linked | GAP | GAP | GAP |
| HI-8322 | [BE] creditline-srvc: split bureau reporting from Spectrum closure, reason-aware | In Validation | home-improvement-creditline-srvc#5273 (OPEN) | Done | Done | GAP |
| HI-8323 | [BE] creditline-srvc: closure-state query and seasoningEndDateTime resolver | In Validation | none linked (closure-state code is in #5273) | Partial | Partial | GAP |
| HI-8324 | [BE] merchant-srvc: adopt reason on project-expiry close | Open | none | GAP | GAP | GAP |
| HI-8325 | [BE] spicedb-schemas: permissions for report-to-bureau and close-account mutations | In Validation | none linked in Jira; spicedb-schemas#1938 (OPEN) is referenced as the dependency of #5273 | N/A | N/A | GAP |
| HI-8326 | [BE] verification-queue-graphql-orch: permission keys for HI closure actions | Open | none | GAP | GAP | GAP |
| HI-8327 | [FE] app-by-phone-ui: account closure actions and row state machine | Open | none | N/A | N/A | GAP |
| HI-8328 | [FE] home-improvement-servicing-ui: port closure changes from app-by-phone-ui | Open | none | N/A | N/A | GAP |
| HI-8329 | [BE] upflow2-home-improvement-dags: spike, exclude gated accounts from payoff selection | Open | none | N/A | N/A | GAP |
| HI-8330 | [QA] [qa-automation] E2E for account closure paths | Open | none | N/A | N/A | GAP |

Classification of the PRs found: 1 service PR checked (creditline#5273), 1 schema/infra PR referenced (spicedb-schemas#1938, N/A). Total 2 unique PRs.

## Coverage Matrix

| Requirement | UT | IT | E2E |
|---|---|---|---|
| Report to bureau split from Spectrum close (reason-aware) | Done | Done | GAP |
| Closure state exposed to VQ (pending closure flag, Close enabled/disabled) | Partial | Partial | GAP |
| Paid Off: Seasoning Period display with end date (PIF + 15d) | Partial | Partial | GAP |
| Paid Off: automation CBR on 8th business day (status 13) | GAP | GAP | GAP |
| Paid Off: agent Close after Day 15; automation close at Day 60 | GAP | GAP | GAP |
| Cancellation/Dispute With Draws vs No Draws derivation, CBR code DA | Done | Done | GAP |
| Close Account confirm modal and acknowledgement checkbox | N/A | N/A | GAP |
| Closed status with timestamp and reason | Partial | Partial | GAP |
| Permissions for report-to-bureau and close-account | N/A | N/A | GAP |
| Activity events for closure and CBR-reported | GAP | GAP | GAP |
| Project-expiry close adopts reason | GAP | GAP | GAP |
| Payoff selection excludes gated accounts (spike) | N/A | N/A | GAP |

## Active Gaps

### High Priority
- No E2E exists anywhere for this epic (none on master, no open qa-automation PR). HI-8330 is open and unassigned to any PR.
- Backend tickets HI-8321, HI-8323, HI-8325 are In Validation but Jira shows no linked PR. Either PRs exist without the ticket key in branch or title, or the work is bundled into #5273. Confirm before trusting the Done/Partial values above.
- HI-8324, HI-8326 have no PR and no tests (permission keys and project-expiry reason are untested).

### Medium Priority
- Paid Off automation (8th business day CBR, Day 60 close) lives in upflow2 DAGs; HI-8329 is only a spike. E2E here needs date-lever support, see Notes.
- spicedb-schemas#1938 is titled "HI-6685 | Add trigger_master_line_closure permission", not an HI-8325 title. Verify it actually covers the report-to-bureau and close-account permissions.

### Low Priority
- Frontend repos (app-by-phone-ui, home-improvement-servicing-ui) have separate unit tests that are out of scope for this map.

### Spec Requirement Gaps
- [HIGH] Spec contradiction on With Draws seasoning: Context and the "Spectrum Closure button state" row say the button unlocks only after a 10-day seasoning; Key Rules says there is no hard seasoning gate and waiting is agent procedure. Needs a PM decision before E2E assertions are written.
- [HIGH] Spec contradiction on Paid Off bureau timing: "How it works" says Day 15; the table, user flow and Key Rules say the 8th business day.
- [MEDIUM] VQ Permissions and audit tracking section is empty. No requirement to test yet.
- [MEDIUM] Reason modal (Loan cancellation or Dispute) and system derivation of With/No Draws: no E2E.
- [LOW] "Questions" section is empty.

## PR Analysis

### home-improvement-creditline-srvc#5273 (OPEN) — HI 8322 split closure and reporting
**Author**: arudenkoupgrade. **Tickets**: HI-8322 (body references the whole epic HI-7540). Depends on spicedb-schemas#1938.
**Functionality**: adds the agent closure inputs (`AgentClosureInputSto`, `AgentClosureReason`, `CloseAgentMasterLineInputSto`), `MasterLineClosureState` and its service, `MasterLineCloseValidator`, `MasterLineClosing.initiatedBy` (Liquibase `HI-8322-add-initiated-by-to-master-line-closing`), GraphQL schema and error-code changes, and a feature flag in `ServerFeatureFlags`.

**Unit Tests**: Done
- MasterLineCloseValidatorTest, MasterLineClosingServiceTest, MasterLineClosureStateServiceTest, CustomerPerformanceSyncServiceTest, GraphQLEnumConsistencyTest

**Integration Tests**: Done
- MasterLineMutationResolverIT

**UT/IT gaps**: not yet reviewed line by line. Run `/feature-context analyze-pr HI-7540 home-improvement-creditline-srvc#5273` for the detailed pass.

## Key Decisions
- The repo used is `ychauhanupgrade/feature-knowledge-base` (the command text says `amishraupgrade`, which is not this user's account).
- Atlassian reads went through the claude.ai Atlassian connector because the `atlassian` MCP is on the retired v1 SSE endpoint.

## Notes for SDET
- Closure is gated by a feature flag added in `ServerFeatureFlags` (#5273). Confirm its name and value per environment before E2E.
- Paid Off paths need backdated PIF dates and automation timing; see the HICS statement gate and backdating notes before designing those tests.
- Follow the VQ conventions in memory: Form/Select drop `data-auto`, use the servicing-ui under `/hi-servicing`.
