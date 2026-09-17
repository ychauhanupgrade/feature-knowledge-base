---
epic: HI-7449
title: HI Fraud Triggers at Payment Request
spec_url: https://credify.atlassian.net/wiki/spaces/PROD/pages/5598543889
tech_design_url: https://credify.atlassian.net/wiki/spaces/HI/pages/5777719448
status: in-development
last_refreshed: 2026-09-17
test_checklist_ticket: HI-7920
confluence_page_id: "6055952391"
---

# HI-7449 — HI Fraud Triggers at Payment Request

## Spec Summary

Fraudulent HI loan volume has risen, and the business does not want to slow issuance to fix it upstream in merchant underwriting. Instead this epic adds alerting and controls **at payment request**: real-time fraud rules evaluated on payment-request / project-completion events, project-level flags so HI Ops can monitor disbursement trends, and (per the product spec) VQ agent UX plus a revived 5-day delayed-payment queue for flagged requests. The product spec (PROD/5598543889) describes four workstreams: (1) richer device fingerprinting fed into `device-identification-srvc` to cut collision rates, (2) additional enrichment sources into the Fraud Engine (IP Capture, device-id, the HI Conditioning Policy EXEMPT trusted-merchant list, LIR fraud-review outcome, related-merchant graph matching on EIN/TIN, business email/website/address/name, owner and control-party identity, IP and device IDs), (3) six new fraud rules (four P1, two P2 "needs further evaluation"), and (4) project-level flags for Tableau reporting plus auto-created Jira tickets for Ops on any trigger except Quick Project.

**The delivered architecture is materially narrower and different from the product spec, and the spec was never updated to match.** Per the tech design (HI/5777719448 §5.1) and the shipped code, the epic became four independent Flink jobs in the `hi-fraud-flink-job` monorepo emitting `PaymentRequestFraudRuleTriggeredEvent` on `event.hiFraud.paymentRequestFraudRuleTriggered`, which `home-improvement-merchant-srvc` (HIMS) consumes into a single `fraud_rule_hit` audit table. The originally designed detection/audit hub in `fraud-monitoring-srvc` (FMS) — with its own `fraudmonitoring` Postgres DB, `fraud_signal`/`fraud_evaluation` tables, a `PaymentRequestFraudSignal` Avro contract and a GraphQL audit query — **was abandoned mid-epic without reopening its tickets**. The product spec's headline P1 rule, *Quick Project* (final payment request within 72h of line opening, project amount > $20,000), **was dropped** and replaced by two category-aware completion rules taken from the Risk-Ops P1 backlog. No VQ UX, no delayed-payment-queue change, no Jira-ticket automation, and no related-merchant graph work exists anywhere in this epic's ticket tree or PR set.

Only **2 of the 5 implemented P1 rules are shipped** (R2 Fraud Review Very High, R4/R5 Rapid Project Completion — both deployed through prod). R1 Same Device is a scaffold on master with the real implementation still in open PRs. R3 Employee Email is actively being **deleted** (six open removal PRs) even though its ticket is Closed and the spec still lists it as P1. EXEMPT trusted-merchant handling — an explicit spec requirement and its own Closed ticket (HI-7620) — **is not present in master at all**: the class merged by `hi-fraud-flink-job#47` was subsequently removed, the shipped Avro event carries no `exempt` field, and the shipped `fraud_rule_hit` table has no `exempt` column.

**E2E automation for this epic is effectively zero.** The single qa-automation PR (#37369, merged) extended `AllureId 21161` to assert `merchantId` on the `projectStatusChanged` Kafka event — a data-plumbing prerequisite for R2, not a fraud rule. No test anywhere exercises a rule firing, the `fraud_rule_hit` row, the evidence payload, idempotency, or exempt behaviour. Service-level UT/IT coverage inside `hi-fraud-flink-job`, by contrast, is genuinely strong (per-rule evaluator, filter, state-retention and pipeline-harness tests plus `RapidProjectCompletionIT` / `SameDeviceIT`).

## Ticket Map

| Ticket | Summary | Status | UT | IT | E2E | PRs |
|---|---|---|---|---|---|---|
| HI-7449 | (epic) HI Fraud Triggers at Payment Request | Open | — | — | — | — |
| HI-7460 | Tech Design — HI Fraud Triggers at Payment Request | Closed | N/A | N/A | N/A | — (design only; HI/5777719448) |
| HI-7553 | [BE] tech Design | Closed | N/A | N/A | N/A | — (design only) |
| HI-7463 | [Spike] Evaluate 11 HI Risk Ops P1 triggers for inclusion in scope | Closed | N/A | N/A | N/A | — (spike; outcome = 4 in scope, #10/#11 pulled in as R4/R5) |
| HI-7462 | UAT — HI Fraud Triggers at Payment Request | Closed | N/A | N/A | N/A | — (no PRs; UAT evidence not linked) |
| HI-7461 | Deployment Checklist | Open | N/A | N/A | N/A | — |
| HI-7455 | [BE][flink-hi-fraud-job] Flink fraud-check job — 5 P1 rules | Closed | Done | Done | GAP | hi-fraud-flink-job#2 (declined), #56; avro-home-improvement-lib#986, #951, #923; k8s-template#281929, #281862 |
| HI-7586 | [BE][avro-fraud-monitoring-lib] Add PaymentRequestFraudSignal Avro schema | Closed | N/A | N/A | N/A | avro-fraud-monitoring-lib#559 (**still OPEN — schema never released**) |
| HI-7456 | [Infra] Provision fraudmonitoring Postgres DB + DMS → Redshift | Closed | N/A | N/A | N/A | github-terraform#4931 (**DECLINED — DB never provisioned**); architecture-decision-records#346 |
| HI-7457 | [BE][fraud-monitoring-srvc] FMS audit hub — fraud_signal/fraud_evaluation, listener, GraphQL audit query | Closed | GAP | GAP | GAP | **no fraud-monitoring-srvc PR exists** — only github-terraform#4931 (declined) + avro-fraud-monitoring-lib#559 (open) |
| HI-7458 | [BE][HIMS] Flag consumer + storage — PR-specific fraud_flag (flag only) | Closed | GAP | GAP | GAP | **no HIMS PR exists** — only avro-fraud-monitoring-lib#559 (open). Superseded by HI-8116 (`fraud_rule_hit`) |
| HI-7459 | [BE][device-identification-srvc] Fingerprint fields + fullHashV3 + Fraud Engine enrichment wiring | Closed | GAP | GAP | GAP | **no device-identification-srvc PR exists** — only architecture-decision-records#346 |
| HI-7620 | [BE] EXEMPT trusted-merchant handling for HI fraud triggers | Closed | GAP | GAP | GAP | hi-fraud-flink-job#47 (merged, **but `ExemptMerchants` is absent from master — since removed**) |
| HI-7621 | [BE] Add lastPayment to HomeImprovementPaymentRequestUpdatedEvent | Closed | GAP | GAP | GAP | home-improvement-merchant-srvc#5803 (**no tests**); avro-home-improvement-lib#923, #924 |
| HI-7626 | Rule 2 — Fraud Review Very High — make required data available | Closed | Done | N/A | COVERED | home-improvement-merchant-srvc#5820; avro-home-improvement-lib#928; **qa-automation#37369** |
| HI-7627 | Rule 3 — Employee Email — make required data available | Closed | N/A | N/A | N/A | **implementation DECLINED** (HIMS#6061, avro-home-improvement-lib#1013); **removal OPEN**: hi-fraud-flink-job#57, stage/preprod/prod/ondemand_terraform, github-terraform#5128 |
| HI-7628 | Rules 4 & 5 — Rapid Project Completion — make required data available | Closed | Done | N/A | GAP | home-improvement-merchant-srvc#5939; avro-home-improvement-lib#962 |
| HI-7625 | [BE][hi-fraud-flink-job] Rule 1 — Same Device — Flink job implementation | Ready for CodeReview | Done | Done | GAP | hi-fraud-flink-job#58 (**OPEN**), #65 (merged); home-improvement-merchant-srvc#6180 (**OPEN**); avro-home-improvement-lib#1034 |
| HI-7960 | [BE][hi-fraud-flink-job] Rules 4 & 5 — Rapid Project Completion — Flink job implementation | Closed | Done | Done | GAP | hi-fraud-flink-job#8, #10; k8s-template#284065, #282695, #282630, #282570; terraform_modules#7406 (declined), stage_terraform#4343 (draft), k8s-template-charts#5063 (declined) |
| HI-8040 | [BE][flink-lib] Resolved config objects + Kafka header capture | Closed | N/A | N/A | N/A | — (no PR surfaced on this ticket; flink-lib#86 referenced in HI-8046 text) |
| HI-8046 | [BE][hi-fraud-flink-job] Rule 2 — Fraud Review Very High — Flink job implementation | Resolved | Done | Done | GAP | hi-fraud-flink-job#48; home-improvement-merchant-srvc#6069; avro-home-improvement-lib#1017; k8s-template#290947 (prod), #290678 (preprod), #289400 (stage), #287896, #286045 |
| HI-8116 | [BE][HIMS] Consume fraud rule triggered events into a new table | Resolved | Done | GAP | GAP | home-improvement-merchant-srvc#6101; hi-fraud-flink-job#59 (**no tests**); avro-home-improvement-lib#1032; k8s-template#288321 |
| HI-7920 | HI Fraud Triggers at Payment Request (P1): Test coverage checklist | Open | N/A | N/A | N/A | — (checklist doc, owned by YCHAUHAN) |

**PR classification summary:** 51 unique PRs across 64 ticket-PR rows. **17 service PRs checked** (`hi-fraud-flink-job` ×10, `home-improvement-merchant-srvc` ×7), 11 schema PRs (N/A — `avro-home-improvement-lib` ×10, `avro-fraud-monitoring-lib` ×1), 22 infra PRs (N/A — `k8s-template` ×12, the five `*_terraform` repos ×6, `github-terraform` ×2, `k8s-template-charts` ×1, `architecture-decision-records` ×1), 1 QA PR (N/A — `qa-automation#37369`). Sum = 51. ✔
`hi-fraud-flink-job` is treated as a **service repo** for UT/IT purposes despite not ending in `-srvc` — it holds all the fraud rule business logic.
Service PRs with **no test files**: `hi-fraud-flink-job#10` (pure package-rename refactor — acceptable), `#57` (module deletion — acceptable), `#59` (**behaviour change to both shipped evaluators' timestamp handling, no test update — real gap**), `home-improvement-merchant-srvc#5803` (**`lastPayment` population on a shared published event, 2 files, no test — real gap**).

## Coverage Matrix

| Requirement Area | UT | IT | E2E | Notes |
|---|---|---|---|---|
| R1 Same Device — PR created + approved on same device id | Done | Done | GAP | HI-7625. `SameDeviceEvaluatorTest`, `PaymentRequestDeviceMapperTest`, `SameDeviceFiltersTest`, `StateRetentionTest`, `SameDeviceIT` — **all in OPEN PR #58**. Master has only the `SameDeviceJob.java` scaffold, no processor, no tests. Not deployed to any env. |
| R2 Fraud Review Very High — merchant > 10 `FRAUD_REVIEW_VERY_HIGH` accounts in rolling 7 days | Done | Done | GAP | HI-8046/HI-7626. `HighRiskLoansEvaluatorTest`, `HighRiskLoansPipelineTest`, `VeryHighAccountAttributionJoinTest`, `HighRiskLoansFiltersTest`, `VeryHighFactDeserializerTest`, `StateRetentionTest`. Deployed stage/preprod/prod (no ondemand). |
| R2 data prerequisite — `merchantId` on `ProjectStatusChangedEvent` | Done | N/A | COVERED | HI-7626. `AllureId 21161` `projectStatusChangedEventIsSentWhenProjectStatusUpdatedTest` in `HomeImprovementMerchantProjectTest` asserts merchantId on PENDING→OPEN and OPEN→CLOSED via `verifyKafkaEventAttributes`. The only real E2E in the epic. |
| R3 Employee Email — borrower email matches a merchant employee email | N/A | N/A | N/A | HI-7627. Implementation PRs **DECLINED**; six **OPEN** PRs delete the module and its infra. Master retains only an `EmployeeEmailJob.java` scaffold. Treat as descoped — but the product spec still lists it P1. |
| R4 Rapid completion, non-HVAC ≥ $50,000 within 2 days | Done | Done | GAP | HI-7960. `RapidCompletionEvaluatorTest`, `MasterLineProjectJoinTest`, `RapidCompletionFiltersTest`, `SnapshotTypesTest`, `StateRetentionTest`, `RapidProjectCompletionIT`. Rule id `RAPID_COMPLETION_NON_HVAC`. Deployed ondemand/stage/preprod/prod. |
| R5 Rapid completion, HVAC ≥ $30,000 within 2 days | Done | Done | GAP | HI-7960. Rule id `RAPID_COMPLETION_HVAC`. **`HVAC_CATEGORIES = Set.of("HVAC")` — `HVAC_PLUS` is deliberately excluded and falls to the R4 $50K threshold** (`hi-fraud-flink-job#56`), contradicting HI-7455 AC4 and HI-7960's own rule text. |
| Rule trigger semantics (day-delta, threshold, single-fire) | Done | Done | GAP | `RapidCompletionEvaluator`: `ChronoUnit.DAYS.between(contractDate, approvedAt in UPGRADE_ZONE_ID)`, `0 <= delta <= 2`, `fired` ValueState guards re-fire. Trigger is **final payment request APPROVED**, not `ProjectStatusChanged COMPLETED` as HI-7455 AC1/HI-7960 AC1/AC2 state. |
| Evidence payload on the emitted event | Done | Done | GAP | R4/R5 evidence keys: `contractDate`, `approvedAt`, `daysToCompletion`, `projectAmount`, `minimumAmount`, `category`. No E2E asserts any of them. |
| EXEMPT trusted-merchant handling | GAP | GAP | GAP | HI-7620 (Closed). **No `ExemptMerchants` class on master**; `RapidCompletionEvaluator` has no exempt branch; the shipped Avro event has **no `exempt` field**; `fraud_rule_hit` has **no `exempt` column**. HI-8116 explicitly declares exemption out of scope. Spec requires "still fund and just log the violation" — unimplementable as built. |
| HIMS consumer + `fraud_rule_hit` storage | Done | GAP | GAP | HI-8116. `FraudRuleHitEventHandlerTest`, `FraudRuleHitServiceTest` (UT only — no consumer IT with `@EmbeddedKafka`). Table `fraud_rule_hit` (`HI-8116-create-fraud-rule-hit-table.changelog.xml`): merchant_id, project_id, payment_request_id, occurred_at, triggered_at, rule_id, evidence jsonb. |
| HIMS consumer idempotency | Done | GAP | GAP | `FraudRuleHitEventHandler` catches `DataIntegrityViolationException` and swallows only `fraud_rule_hit_uuid_uk` violations. UT-level only; no IT redelivers a duplicate event through Kafka, no E2E. |
| `lastPayment` on `HomeImprovementPaymentRequestUpdatedEvent` | GAP | GAP | GAP | HI-7621. `home-improvement-merchant-srvc#5803` changed `PaymentRequestService` with **zero tests**. Shared-schema change consumed by R4/R5's final-PR detection. |
| `previousStatus` on payment-request event | Done | N/A | GAP | HI-7625. `PaymentRequestServiceTest` in **OPEN** HIMS#6180; `hi-fraud-flink-job#65` classifies transitions on it. |
| `projectAmount` / `category` / `approvalMethod` on fraud source events | Done | N/A | GAP | HI-7628. `ProjectAmountsTest`, `OpenProjectStrategyTest`, `PendingProjectStrategyTest`, `PaymentRequestServiceTest`, `ProjectStatusChangedEventPublisherTest` (HIMS#5939). |
| FMS audit hub — `fraud_signal`/`fraud_evaluation`, Kafka listener, `fraudSignalsForPaymentRequest` GraphQL, `READ_FRAUD_SIGNAL` authz | GAP | GAP | GAP | HI-7457 Closed with **no `fraud-monitoring-srvc` PR in existence**. Design abandoned in favour of the direct Flink→HIMS path. |
| `fraudmonitoring` Postgres DB + DMS → Redshift | GAP | GAP | GAP | HI-7456 Closed; `github-terraform#4931` **DECLINED**. DB was never provisioned. |
| `PaymentRequestFraudSignal` Avro contract | GAP | GAP | GAP | HI-7586 Closed; `avro-fraud-monitoring-lib#559` **still OPEN**. Superseded by `PaymentRequestFraudRuleTriggeredEvent` in `avro-home-improvement-lib` (`eventId`/`ruleId`/`sourceJob`/`paymentRequestId`/`projectId`/`merchantId`/`accountId`/`evidence` map/`occurredAt`/`triggeredAt`) — which has **no `signalId` and no `exempt`**, breaking HI-7586 AC2. |
| `paymentRequestFraudDetected` downstream event | GAP | GAP | GAP | HI-7457 AC2. Never built (FMS descoped). |
| Device fingerprint fields + `full_hash_v3` + Fraud Engine enrichment | GAP | GAP | GAP | HI-7459 Closed with **no `device-identification-srvc` PR in existence**. ~18 net-new fields from the spec are unimplemented. |
| Project-level flags for Tableau reporting | Partial | GAP | GAP | Satisfied indirectly by `fraud_rule_hit` (HI-8116) as a queryable HIMS table, but there is no project-level flag column, no exempt marker, and no confirmed Redshift/Tableau consumer story. |
| Jira ticket auto-created per trigger (except Quick Project) | GAP | GAP | GAP | Spec "[NEW]" requirement. **No ticket, no PR, nothing.** |
| 5-day delayed-payment queue modification for flagged requests | GAP | GAP | GAP | Spec Context workstream. **No ticket, no PR.** HI-7920 notes the existing `HI_DELAY_PAYMENTS` feature (`HomeImprovementRequestPaymentsTest`) should be extended rather than duplicated — nothing has been. |
| VQ agent UX for reviewing why a PR was flagged | GAP | GAP | GAP | Spec Context workstream; spec's own Designs field says "TBD". **No ticket, no PR.** |
| Related-merchant graph matching (EIN/TIN, business email/website/address/name, owner + control party, IP, device IDs) | GAP | GAP | GAP | Spec enrichment requirement, incl. 4 NEW match keys. **No ticket, no PR.** |
| Store of IP addresses / cookies / devices per borrower | GAP | GAP | GAP | Spec enrichment requirement. **No ticket, no PR.** |
| Quick Project rule (final PR ≤ 72h of line open, project > $20,000) | N/A | N/A | N/A | **Dropped** per tech design §5.1 and HI-7960; replaced by R4/R5. Spec never updated. |
| P2 — Email/Phone Mismatch (Emailage first-seen < 30d OR Whitepages no name match) | N/A | N/A | N/A | Out of scope per tech design (P2/P3 out) and HI-7920's scope note. |
| P2 — Vulnerable Population + New Email (borrower ≥ 70y and Emailage first-seen < 30d) | N/A | N/A | N/A | Out of scope per tech design (P2/P3 out) and HI-7920's scope note. |

## Active Gaps

### High Priority

- [HIGH] **Zero E2E coverage of any fraud rule firing.** Two rules (R2, R4/R5) are live in **production** with no end-to-end test that a trigger produces a `fraud_rule_hit` row. The only E2E in the epic (`AllureId 21161`) asserts a `merchantId` field on an upstream event. Minimum viable set: drive a non-HVAC ≥$50K project to final-PR-approved within 2 days of contract date → assert one `fraud_rule_hit` row with `rule_id = RAPID_COMPLETION_NON_HVAC` and the six evidence keys; same for `RAPID_COMPLETION_HVAC` at ≥$30K; and an R2 run that crosses the >10 `FRAUD_REVIEW_VERY_HIGH` threshold in 7 days.
- [HIGH] **EXEMPT trusted-merchant handling does not exist in shipped code**, despite being an explicit spec requirement with a Closed ticket (HI-7620) and a merged PR (`hi-fraud-flink-job#47`). `ExemptMerchants` is absent from master, the Avro event has no `exempt` field, and `fraud_rule_hit` has no `exempt` column. Any exempt-merchant test written today would have nothing to assert. Needs a product/eng decision: re-land it, or formally descope it and correct the spec.
- [HIGH] **Five Closed tickets have no shipped implementation** — HI-7457 (FMS audit hub), HI-7456 (`fraudmonitoring` DB, terraform PR declined), HI-7458 (HIMS `fraud_flag`), HI-7459 (device fingerprinting), HI-7586 (`PaymentRequestFraudSignal`, PR still open). The epic pivoted to Flink→HIMS without reopening or restating them, so Jira materially overstates delivery. Coverage planning against the ticket tree alone will be wrong.
- [HIGH] **`hi-fraud-flink-job#59` changed both shipped evaluators' timestamp handling with no test update.** It retyped fraud-trigger timestamps to `timestamp-with-offset-millis` across `HighRiskLoansEvaluator` and `RapidCompletionEvaluator` — the exact fields R4/R5's `daysToCompletion` day-delta is computed from — while touching zero test files. A timezone/offset regression here silently changes which projects trigger.
- [HIGH] **R3 Employee Email is being deleted while the spec still lists it as a P1 rule.** Six open removal PRs (`hi-fraud-flink-job#57`, four `*_terraform`, `github-terraform#5128`) against a Closed ticket whose implementation PRs were declined. Product has not confirmed the descope in the spec; HI-7920's checklist still includes it.

### Medium Priority

- [MEDIUM] **No integration test for the HIMS Kafka consumer.** `FraudRuleHitEventHandlerTest` and `FraudRuleHitServiceTest` are UT-only. The `fraud_rule_hit_uuid_uk` idempotency path — a `DataIntegrityViolationException` narrowed by constraint name — is exactly the kind of logic that needs a real `@EmbeddedKafka`/`@DataJpaTest` redelivery, not a mock.
- [MEDIUM] **`home-improvement-merchant-srvc#5803` (HI-7621 `lastPayment`) shipped with no tests** on a shared event schema consumed by R4/R5's final-payment detection. `lastPayment` wrong or unset means the rapid-completion rules never fire.
- [MEDIUM] **`HVAC_PLUS` routing contradicts the tickets.** Shipped code excludes `HVAC_PLUS` from R5 and applies the R4 $50K threshold; HI-7455 AC4 and HI-7960 both say `primaryCategory IN (HVAC, HVAC_PLUS)` → $30K. One of the two is wrong. Confirm with product, then pin it with a test — an HVAC_PLUS project between $30K and $50K is the discriminating case.
- [MEDIUM] **Rule trigger semantics diverge from the ACs.** HI-7455 AC1 and HI-7960 AC1/AC2 describe firing on `ProjectStatusChangedEvent newStatus=COMPLETED` with `DATEDIFF(contractDate, completion updatedAt)`. Shipped code fires on **final payment request APPROVED** and measures to the approval timestamp. Tests written from the ACs will not match behaviour.
- [MEDIUM] **R2 has no ondemand deployment** (`k8s-template/v2/applications/hi-fraud-flink-job-high-risk-loans` has stage/preprod/prod only), so R2 E2E cannot run on an ondemand stack. R4/R5 does have ondemand. Plan R2 E2E for stage or preprod.
- [MEDIUM] **R1 Same Device lands with good service tests but no E2E plan.** `SameDeviceIT` plus five UT classes sit in open PR #58 alongside HIMS#6180. This is the cheapest moment to add E2E — the null-device-id tolerance path (SMS/tacit/auto approval, HI-7455 AC2) is a behaviour no other rule has.

### Low Priority

- [LOW] `HI-7462` (UAT) is Closed with no linked evidence or PRs — no record of what was actually UAT'd.
- [LOW] `HI-8040` (flink-lib config + Kafka header capture) surfaced no PR on its own ticket; `flink-lib#86` is referenced only in HI-8046's prose. Header capture is load-bearing for R2 event time (fraud-review events carry no timestamp), so its provenance is worth pinning down.
- [LOW] `stage_terraform#4343` is still a **DRAFT** and `terraform_modules#7406` was **DECLINED** for rapid-completion onboarding, yet the job is deployed to prod via k8s-template. Worth confirming there is no half-finished Terraform state.

### Spec Requirement Gaps

Requirements present in the product spec (PROD/5598543889) with **no ticket and no PR anywhere in this epic**:

- [HIGH] **Jira ticket auto-creation on any P1 trigger except Quick Project** — spec's own "[NEW]" flagged requirement, the primary Ops-facing output. Nothing built.
- [HIGH] **5-day delayed-payment queue modification** for flagged payment requests — one of the five workstreams in the spec's Context. Nothing built. The existing `HI_DELAY_PAYMENTS` feature is untouched.
- [HIGH] **VQ agent UX** showing why a payment request was flagged plus data points to investigate — one of the five workstreams. Nothing built; spec's Designs field is still "TBD" and only a Satori mockup link exists.
- [MEDIUM] **Related-merchant graph integration into enrichment**, including the four NEW match keys (Business Name secondary, Control Party Name + Email, IP Address, Device IDs) — nothing built.
- [MEDIUM] **Borrower-associated store of IP addresses, cookies and devices** ("will be used for future rules") — nothing built.
- [MEDIUM] **Device fingerprint metadata expansion** (~18 net-new fields across device properties, browser/environment, network, mobile-specific, behavioural) and Fraud Engine enrichment wiring — HI-7459 is Closed but no `device-identification-srvc` PR exists.
- [MEDIUM] **HI Conditioning Policy EXEMPT list integration into FMS** — see the HIGH exempt gap above; neither the FMS side nor the rule-skip side exists.
- [LOW] **LIR Fraud Review outcome as an enrichment source** — partially satisfied incidentally by R2 consuming `fraudReviewTaskCreated`/`fraudReviewTaskUpdated`, but not as a general enrichment input.
- [LOW] **Stale spec text**: the spec still lists *Quick Project* (72h / $20K) as P1 and *Employee Email* as P1. The first was dropped for R4/R5; the second is being deleted. The spec has not been revised and is now actively misleading as a test-design source — use the tech design (HI/5777719448 §5.1) instead.

## PR Analysis

### hi-fraud-flink-job#8, #10 — HI-7960: Rapid Project Completion (R4/R5)
**Functionality**: First real rule implementation, in the `hi-fraud-flink-job-rapid-completion` module. `#10` then nested job main classes under per-module `hifraud` sub-packages (rename only, no tests — acceptable).
**Unit Tests**: `RapidProjectCompletionJobTest`, `RapidCompletionHarnessTest`, `SnapshotTypesTest`, `StateRetentionTest`, `RapidCompletionFiltersTest`, `Fixtures`.
**Integration Tests**: `RapidProjectCompletionHappyPathIT` (later `RapidProjectCompletionIT`).
**E2E**: GAP.

### hi-fraud-flink-job#47 — HI-7620: exempt-merchant handling for R4/R5
**Functionality**: Added exempt-merchant handling plus `config/ExemptMerchantsTest`. **Merged, but `ExemptMerchants` no longer exists on master** — subsequently removed with no ticket recording the reversal. The single most important discrepancy in the epic.

### hi-fraud-flink-job#48 — HI-8046: Fraud Review Very High (R2)
**Functionality**: `hi-fraud-flink-job-high-risk-loans` module. Counts distinct accounts with a `FRAUD_REVIEW_VERY_HIGH` review attributed to a merchant in a trailing 7-day window, > 10 fires. Event time comes from the `EVENT_CREATED` Kafka header (fraud-review events carry no timestamp) via flink-lib header capture. Merchant attribution learned from `ProjectStatusChangedEvent` on **any** transition so declined-and-never-opened loans still count.
**Unit Tests**: `HighRiskLoansEvaluatorTest`, `HighRiskLoansPipelineTest`, `VeryHighAccountAttributionJoinTest`, `StateRetentionTest`, `HighRiskLoansFiltersTest`, `HighRiskLoansSourcesTest`, `VeryHighFactDeserializerTest`, `Fixtures`, `HighRiskLoansPipelineHarness`.
**Integration Tests**: covered by the pipeline harness rather than a named `*IT`.
**E2E**: GAP. Deployed to prod (`k8s-template#290947`).
**Open decisions carried in the ticket**: count at task creation vs only at settlement (VERIFIED/WAIVED); fire-on-first vs every subsequent PR.

### hi-fraud-flink-job#56 — Exclude HVAC_PLUS from the R5 HVAC threshold bucket
**Functionality**: Narrowed `HVAC_CATEGORIES` to `Set.of("HVAC")`, sending `HVAC_PLUS` to the R4 $50K threshold. Directly contradicts HI-7455 AC4 / HI-7960's rule text. Also moved `ExemptMerchantsTest` into `hi-fraud-flink-job-common/config`.
**Unit Tests**: `RapidCompletionEvaluatorTest`, `RapidProjectCompletionJobTest`, `ExemptMerchantsTest`.
**Integration Tests**: `RapidProjectCompletionIT`.

### hi-fraud-flink-job#58 (OPEN) — HI-7625: Rule 1 Same Device
**Functionality**: The real R1 implementation — `PaymentRequestDeviceMapper`, `SameDeviceEvaluator`, filters, state retention, plus `RequestContextTest` in common.
**Unit Tests**: `SameDeviceJobTest`, `PaymentRequestDeviceMapperTest`, `SameDeviceEvaluatorTest`, `StateRetentionTest`, `SameDeviceFiltersTest`, `Fixtures`.
**Integration Tests**: `SameDeviceIT`.
**E2E**: GAP — best opportunity to add E2E before merge. Paired with `home-improvement-merchant-srvc#6180` (OPEN, `previousStatus`).

### hi-fraud-flink-job#59 — HI-8116: publish OffsetDateTime fraud trigger timestamps
**Functionality**: Retyped trigger timestamps to `timestamp-with-offset-millis`, editing `HighRiskLoansEvaluator` and `RapidCompletionEvaluator` — both shipped rules.
**UT Gaps**: [HIGH] **no test files at all**. The changed fields feed R4/R5's `ChronoUnit.DAYS.between(contractDate, approvedAt)` day-delta; an offset regression changes rule outcomes silently.

### hi-fraud-flink-job#65 — HI-7625: classify transitions on previousStatus
**Functionality**: Uses the new `previousStatus` field to classify payment-request transitions.
**Unit Tests**: `PaymentRequestDeviceMapperTest`, `SameDeviceEvaluatorTest`, `SameDeviceFiltersTest`, `Fixtures`. **Integration Tests**: `SameDeviceIT`.

### hi-fraud-flink-job#57 (OPEN) — Remove employee-email module
**Functionality**: Deletes `Dockerfile.employee-email`, the module, its pom entry and README references. No tests (deletion). Part of a six-PR removal set across the Terraform repos and `github-terraform#5128`.

### home-improvement-merchant-srvc#6101 — HI-8116: consume fraud rule triggered events
**Functionality**: The whole downstream audit path. `FraudRuleHitEventHandler` (`@KafkaListener` on `HiFraudTopics.PAYMENT_REQUEST_FRAUD_RULE_TRIGGERED`), `FraudRuleHitService`, `FraudRuleHit` entity, `FraudRuleHitRepository`, `FraudConfiguration`, and `HI-8116-create-fraud-rule-hit-table.changelog.xml`. Ingestion + storage only, no GraphQL exposure; exemption explicitly out of scope.
**Unit Tests**: `FraudRuleHitEventHandlerTest`, `FraudRuleHitServiceTest`.
**IT Gaps**: [MEDIUM] no consumer IT — the duplicate-`uuid` idempotency branch keys off the constraint name `fraud_rule_hit_uuid_uk` and is only mock-tested.
**E2E**: GAP — this table is the natural assertion point for every rule E2E.

### home-improvement-merchant-srvc#5803 — HI-7621: lastPayment on published events
**Functionality**: Sets `lastPayment` on `HomeImprovementPaymentRequestUpdatedEvent` from `paymentRequestSto.isLastPayment()`. Two files: `PaymentRequestService.java` and `pom.xml`.
**UT Gaps**: [MEDIUM] **no tests**. Shared schema, all `paymentRequestUpdated` consumers affected, and R4/R5 depend on it to identify the final PR.

### home-improvement-merchant-srvc#5820 / #5939 / #6069 — HI-7626 / HI-7628 / HI-8046 data groundwork
**Functionality**: `merchantId` on `ProjectStatusChangedEvent` (#5820); `projectAmount`, `category`, `approvalMethod` on the fraud source events (#5939); `merchantId` on `HomeImprovementPaymentRequestUpdatedEvent` (#6069).
**Unit Tests**: `ProjectStatusChangedEventPublisherTest`, `ProjectServiceEventPublishingTest`, `ProjectExpirationServiceEventPublishingTest`, `ProjectServiceTest`, `ProjectAmountsTest`, `OpenProjectStrategyTest`, `PendingProjectStrategyTest`, `PaymentRequestServiceTest`.
**E2E**: `#5820` is the one area with real E2E — `AllureId 21161` via `qa-automation#37369`.

### home-improvement-merchant-srvc#6061 (DECLINED) — HI-7627: employee/borrower emails for R3
**Functionality**: Would have published `employeeUpdated` and `projectBorrowerUpdated` carrying encrypted email. Had substantial tests (`EmployeeUpdatedEventPublisherTest`, `ProjectBorrowerUpdatedEventPublisherTest`, `EmployeeEmailUpdateServiceTest`, `ProjectBorrowerServiceTest`, `ActorServiceTest`, `HomeImprovementMerchantGivens`). **Declined** alongside `avro-home-improvement-lib#1013` — R3 abandoned.

### qa-automation#37369 — HI-7626: verify merchantId on projectStatusChanged
**Functionality**: The epic's only QA PR. Extended `HomeImprovementMerchantProjectTest` (`AllureId 21161`, `@Jira("HI-5464")`, owner AMISHRA) to assert `merchantId` through `verifyKafkaEventAttributes` on PENDING→OPEN and OPEN→CLOSED, plus a `HomeImprovementMerchantServiceUtils` helper. Merged to master.
**E2E Coverage**: COVERED for the R2 data prerequisite only. No rule-firing coverage.

### avro-fraud-monitoring-lib#559 (OPEN) — HI-7586: PaymentRequestFraudSignal
**Functionality**: The abandoned FMS contract. Linked from four Closed tickets (HI-7455, HI-7457, HI-7458, HI-7586) yet never merged. Superseded by `PaymentRequestFraudRuleTriggeredEvent` in `avro-home-improvement-lib#986`.

## Key Decisions

1. **Quick Project (72h / >$20K) was dropped** and replaced by the two Risk-Ops completion rules R4 (non-HVAC ≥$50K ≤2d) and R5 (HVAC ≥$30K ≤2d). Tech design HI/5777719448 §5.1; HI-7960. The product spec was never updated.
2. **The FMS audit hub was abandoned.** The Flink→FMS→HIMS design (`PaymentRequestFraudSignal`, `fraudmonitoring` DB, `fraud_signal`/`fraud_evaluation`, GraphQL audit query, `paymentRequestFraudDetected`) gave way to Flink→HIMS direct, with `PaymentRequestFraudRuleTriggeredEvent` on `event.hiFraud.paymentRequestFraudRuleTriggered` persisted into HIMS `fraud_rule_hit`. Tickets HI-7456/7457/7458/7586 were Closed rather than restated.
3. **Table and contract naming moved three times**: `payment_request_fraud_flag` (HI-7458) → `payment_request_fraud_trigger` (HI-8116 description) → **`fraud_rule_hit`** (shipped). Use the shipped name.
4. **`HVAC_PLUS` takes the non-HVAC $50K threshold**, not R5's $30K (`hi-fraud-flink-job#56`) — contrary to the ticket ACs. Unconfirmed with product.
5. **R4/R5 fire on final payment request APPROVED**, not on `ProjectStatusChanged COMPLETED`, and the day-delta is measured `contractDate → approval updateDateTime` in `DefaultCalendar.UPGRADE_ZONE_ID`, inclusive of 0 and 2.
6. **R2 event time is the `EVENT_CREATED` Kafka header**, because fraud-review events carry no timestamp (HI-8040 flink-lib header capture). Merchant attribution is learned from `ProjectStatusChangedEvent` on any transition, not only OPEN.
7. **Exempt handling was merged then removed.** No `exempt` field on the event, no `exempt` column in `fraud_rule_hit`, no `ExemptMerchants` on master.
8. **P2 rules (Email/Phone Mismatch, Vulnerable Population + New Email) are out of scope** per the tech design and HI-7920's scope note.
9. **Rule ids are string constants in the evaluators**: `RAPID_COMPLETION_NON_HVAC`, `RAPID_COMPLETION_HVAC`. R1/R2 ids live in their own evaluators; there is no shared rule-id enum.

## Notes for SDET

- **Assert against HIMS `fraud_rule_hit`**, not FMS — FMS has no database. Columns: `merchant_id`, `project_id`, `payment_request_id`, `occurred_at`, `triggered_at`, `rule_id`, `evidence` (jsonb), plus `uuid` with unique constraint `fraud_rule_hit_uuid_uk`. There is **no exempt column** — do not plan assertions on it.
- **R4/R5 evidence keys** to assert: `contractDate`, `approvedAt`, `daysToCompletion`, `projectAmount`, `minimumAmount`, `category`.
- **Discriminating test cases**: day-delta exactly 2 fires / 3 does not; `HVAC_PLUS` between $30K and $50K (should *not* fire per shipped code, *should* per the ACs — this case decides the discrepancy); amount aggregation across sub-lines for split projects; `fired` ValueState means a second qualifying PR must not re-emit.
- **Env reach**: R4/R5 (`rapid-completion`) has ondemand/stage/preprod/prod. R2 (`high-risk-loans`) has **stage/preprod/prod only — no ondemand**. R1 and R3 have no deployment at all.
- **Backdating caveat**: R4/R5 compare `contractDate` against the approval timestamp. Existing HI backdate helpers write HI Postgres only and never sync Spectrum, and master/sub-line contract dates must stay day-based with master ≤ subline — see the existing HI backdating notes before constructing a ≤2-day window.
- **The product spec is not a safe test-design source.** It still lists the dropped Quick Project rule and the being-deleted Employee Email rule, and describes an FMS/exempt/VQ/Jira architecture that was never built. Design from the tech design (HI/5777719448 §5.1) and the shipped evaluators.
- **HI-7920 already carries a scenario table** built on the spec's four P1 rules and assumes signal/flag/Jira-ticket/delayed-review outputs. Several of its rows are untestable as built (Jira ticket, delayed-payment routing, exempt skip, device-fingerprint fields). It needs reconciling against the shipped design before it drives work.
- **Flink jobs are not Spring Boot services** — no actuator, no GraphQL, no REST. E2E has to drive them through Kafka and observe the HIMS table; there is no synchronous handle on rule evaluation.
