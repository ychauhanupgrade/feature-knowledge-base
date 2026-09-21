---
epic: HI-7449
title: HI Fraud Triggers at Payment Request
spec_url: https://credify.atlassian.net/wiki/spaces/PROD/pages/5598543889
tech_design_url: https://credify.atlassian.net/wiki/spaces/HI/pages/5777719448
status: in-development
last_refreshed: 2026-09-21
qa_automation_pr: "38621"
test_checklist_ticket: HI-7920
confluence_page_id: "6055952391"
---

# HI-7449 — HI Fraud Triggers at Payment Request

## Spec Summary

Fraudulent HI loan volume has risen, and the business does not want to slow issuance to fix it upstream in merchant underwriting. Instead this epic adds alerting and controls **at payment request**: real-time fraud rules evaluated on payment-request / project-completion events, project-level flags so HI Ops can monitor disbursement trends, and (per the product spec) VQ agent UX plus a revived 5-day delayed-payment queue for flagged requests. The product spec (PROD/5598543889) describes four workstreams: (1) richer device fingerprinting fed into `device-identification-srvc` to cut collision rates, (2) additional enrichment sources into the Fraud Engine (IP Capture, device-id, the HI Conditioning Policy EXEMPT trusted-merchant list, LIR fraud-review outcome, related-merchant graph matching on EIN/TIN, business email/website/address/name, owner and control-party identity, IP and device IDs), (3) six new fraud rules (four P1, two P2 "needs further evaluation"), and (4) project-level flags for Tableau reporting plus auto-created Jira tickets for Ops on any trigger except Quick Project.

**The delivered architecture is materially narrower and different from the product spec, and the spec was never updated to match.** Per the tech design (HI/5777719448 §5.1) and the shipped code, the epic became four independent Flink jobs in the `hi-fraud-flink-job` monorepo emitting `PaymentRequestFraudRuleTriggeredEvent` on `event.hiFraud.paymentRequestFraudRuleTriggered`, which `home-improvement-merchant-srvc` (HIMS) consumes into a single `fraud_rule_hit` audit table. The originally designed detection/audit hub in `fraud-monitoring-srvc` (FMS) — with its own `fraudmonitoring` Postgres DB, `fraud_signal`/`fraud_evaluation` tables, a `PaymentRequestFraudSignal` Avro contract and a GraphQL audit query — **was abandoned mid-epic without reopening its tickets**. The product spec's headline P1 rule, *Quick Project* (final payment request within 72h of line opening, project amount > $20,000), **was dropped** and replaced by two category-aware completion rules taken from the Risk-Ops P1 backlog. No VQ UX, no delayed-payment-queue change, no Jira-ticket automation, and no related-merchant graph work exists anywhere in this epic's ticket tree or PR set.

**3 of the 5 implemented P1 rules are shipped to prod, and a fourth is live below prod.** R2 Fraud Review Very High and R4/R5 Rapid Project Completion are deployed through prod. **R1 Same Device fully landed since the last refresh** — `hi-fraud-flink-job#58` merged 2026-09-18 after its HIMS `previousStatus` prerequisite deployed, and `k8s-template#292681`/`#292954` put it on ondemand, stage and preprod; **there is no prod deployment**. R3 Employee Email is actively being **deleted** (six open removal PRs) even though its ticket is Closed and the spec still lists it as P1. EXEMPT trusted-merchant handling — an explicit spec requirement with its own Closed ticket (HI-7620) — **is still not present in master at all**: the class merged by `hi-fraud-flink-job#47` was subsequently removed, the shipped Avro event carries no `exempt` field, and the shipped `fraud_rule_hit` table has no `exempt` column. A shared `com.credify.hi.common.fraud.FraudRuleId` enum now exists in hi-common-lib, replacing the per-evaluator rule-id string constants.

**E2E automation now covers every shipped rule, and all four are validated on preprod.** qa-automation#38621 (open, awaiting review) adds one E2E per rule — R1 `AllureId 88541`, R2 `87944`, R4 `87945`, R5 `87920` — plus the shared scaffolding (`fraud_rule_hit` queries with evidence keys lifted into flat `evidence_*` columns, a `HomeImprovementFraudRule` enum mirroring hi-common-lib's `FraudRuleId`, a `waitForFraudRuleHits` poll helper, the `HI_FRAUD_TRIGGERS_AT_PAYMENT_REQUEST` feature constant, a merchant-category overload, and an `extraHeaders` path through `GraphQLService`/`BaseService` so R1 can send `X-UPG-DEVICE-ID`). Each test asserts its rule's full evidence payload, and R1's asserts that the `deviceId` in evidence equals the UUID the test minted — exercising the whole `X-UPG-DEVICE-ID` → request context → `REQUEST_CONTEXT` header → Flink chain, not just the rule. The only prior qa-automation PR (#37369, merged) extended `AllureId 21161` to assert `merchantId` on the `projectStatusChanged` Kafka event — a data-plumbing prerequisite for R2, not a fraud rule. Service-level UT/IT inside `hi-fraud-flink-job` remains genuinely strong (185 test methods across the four rule modules plus 11 in common), but it runs against a Flink test harness with synthetic events — which is precisely why the four preprod passes matter.

**One rule remains deliberately uncovered:** R3 Employee Email, which is being **deleted** by six open PRs. Per Charles Chartrand's review of this page (2026-09-17), the target is at most one E2E per rule, so threshold-boundary cases, the `HVAC_PLUS` discriminator, R2's exactly-10 boundary and its 7-day window, and R1's null-device-id tolerance all stay at the unit layer rather than becoming additional E2Es — each is either unobservable downstream or disproportionately expensive to drive end to end (R2's boundary alone would need 10 seeded accounts at roughly a minute each to prove an off-by-one already pinned in `HighRiskLoansEvaluatorTest`).

## Ticket Map

| Ticket | Summary | Status | UT | IT | E2E | PRs |
|---|---|---|---|---|---|---|
| HI-7449 | (epic) HI Fraud Triggers at Payment Request | Open | — | — | — | — |
| HI-7460 | Tech Design — HI Fraud Triggers at Payment Request | Closed | N/A | N/A | N/A | — (design only; HI/5777719448) |
| HI-7553 | [BE] tech Design | Closed | N/A | N/A | N/A | — (design only) |
| HI-7463 | [Spike] Evaluate 11 HI Risk Ops P1 triggers for inclusion in scope | Closed | N/A | N/A | N/A | — (spike; outcome = 4 in scope, #10/#11 pulled in as R4/R5) |
| HI-7462 | UAT — HI Fraud Triggers at Payment Request | Closed | N/A | N/A | N/A | — (no PRs; UAT evidence not linked) |
| HI-7461 | Deployment Checklist | Closed | N/A | N/A | N/A | — |
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
| HI-7625 | [BE][hi-fraud-flink-job] Rule 1 — Same Device — Flink job implementation | Resolved | Done | Done | COVERED | hi-fraud-flink-job#58 (**MERGED 2026-09-18**), #65; home-improvement-merchant-srvc#6180; avro-home-improvement-lib#1034; **k8s-template#292681 (ondemand+stage), #292954 (preprod)**. Fully landed. **E2E `AllureId 88541` passed on preprod (Jenkins 6925)**. Not deployed to prod. |
| HI-7960 | [BE][hi-fraud-flink-job] Rules 4 & 5 — Rapid Project Completion — Flink job implementation | Closed | Done | Done | COVERED | hi-fraud-flink-job#8, #10; k8s-template#284065, #282695, #282630, #282570; terraform_modules#7406 (declined), stage_terraform#4343 (draft), k8s-template-charts#5063 (declined); **qa-automation#38621 (open, AllureId 87920 + 87945 — both passed on preprod)** |
| HI-8040 | [BE][flink-lib] Resolved config objects + Kafka header capture | Closed | N/A | N/A | N/A | — (no PR surfaced on this ticket; flink-lib#86 referenced in HI-8046 text) |
| HI-8046 | [BE][hi-fraud-flink-job] Rule 2 — Fraud Review Very High — Flink job implementation | Closed | Done | Done | COVERED | hi-fraud-flink-job#48; home-improvement-merchant-srvc#6069; avro-home-improvement-lib#1017; k8s-template#290947 (prod), #290678 (preprod), #289400 (stage), #287896, #286045; **qa-automation#38621 (open, AllureId 87944 — passing on preprod)** |
| HI-8116 | [BE][HIMS] Consume fraud rule triggered events into a new table | Closed | Done | GAP | COVERED | home-improvement-merchant-srvc#6101; hi-fraud-flink-job#59 (**no tests**); avro-home-improvement-lib#1032; k8s-template#288321 |
| HI-7920 | HI Fraud Triggers at Payment Request (P1): Test coverage checklist | In Development | N/A | N/A | N/A | — (checklist doc, owned by YCHAUHAN; **rewritten 2026-09-21 against shipped code**) |

**PR classification summary:** 54 unique PRs across 73 ticket-PR rows (+3 since last refresh). **17 service PRs checked** (`hi-fraud-flink-job` ×10, `home-improvement-merchant-srvc` ×7 — unchanged; `hi-fraud-flink-job#58` flipped OPEN→MERGED), 11 schema PRs (N/A — `avro-home-improvement-lib` ×10, `avro-fraud-monitoring-lib` ×1), 24 infra PRs (N/A — `k8s-template` ×14 incl. the two new same-device deploys, the five `*_terraform` repos ×6, `github-terraform` ×2, `k8s-template-charts` ×1, `architecture-decision-records` ×1), 2 QA PRs (N/A — `qa-automation#37369`, `#38621`). Sum = 54. ✔
`hi-fraud-flink-job` is treated as a **service repo** for UT/IT purposes despite not ending in `-srvc` — it holds all the fraud rule business logic.
Service PRs with **no test files** (unchanged): `hi-fraud-flink-job#10` (pure package-rename refactor — acceptable), `#57` (module deletion — acceptable), `#59` (**behaviour change to both shipped evaluators' timestamp handling, no test update — real gap**), `home-improvement-merchant-srvc#5803` (**`lastPayment` population on a shared published event, 2 files, no test — real gap**).

## Coverage Matrix

| Requirement Area | UT | IT | E2E | Notes |
|---|---|---|---|---|
| R1 Same Device — PR created + approved on same device id | Done | Done | COVERED | HI-7625. `SameDeviceEvaluatorTest`, `PaymentRequestDeviceMapperTest`, `SameDeviceFiltersTest`, `StateRetentionTest`, `SameDeviceIT`, `SameDeviceJobTest` — **53 test methods, merged to master in #58 (2026-09-18)**. Deployed ondemand/stage/preprod — **not prod**. **E2E `AllureId 88541` (qa-automation#38621) passed on preprod 2026-09-21 (Jenkins 6925)**, asserting the evidence `deviceId` equals the UUID the test minted. Classification is on the `status`/`previousStatus` transition pair, not `approvalMethod`. |
| R2 Fraud Review Very High — merchant > 10 `FRAUD_REVIEW_VERY_HIGH` accounts in rolling 7 days | Done | Done | COVERED | HI-8046/HI-7626. `HighRiskLoansEvaluatorTest`, `HighRiskLoansPipelineTest`, `VeryHighAccountAttributionJoinTest`, `HighRiskLoansFiltersTest`, `VeryHighFactDeserializerTest`, `StateRetentionTest`. Deployed stage/preprod/prod (no ondemand). **E2E `AllureId 87944` (qa-automation#38621) passed on preprod 2026-09-17** — 11 distinct VERY_HIGH accounts then a payment request, asserting `veryHighCount=11`, `threshold=10`, `windowDays=7`. |
| R2 data prerequisite — `merchantId` on `ProjectStatusChangedEvent` | Done | N/A | COVERED | HI-7626. `AllureId 21161` `projectStatusChangedEventIsSentWhenProjectStatusUpdatedTest` in `HomeImprovementMerchantProjectTest` asserts merchantId on PENDING→OPEN and OPEN→CLOSED via `verifyKafkaEventAttributes`. The only real E2E in the epic. |
| R3 Employee Email — borrower email matches a merchant employee email | N/A | N/A | N/A | HI-7627. Implementation PRs **DECLINED**; six **OPEN** PRs delete the module and its infra. Master retains only an `EmployeeEmailJob.java` scaffold. Treat as descoped — but the product spec still lists it P1. |
| R4 Rapid completion, non-HVAC ≥ $50,000 within 2 days | Done | Done | COVERED | HI-7960. `RapidCompletionEvaluatorTest`, `MasterLineProjectJoinTest`, `RapidCompletionFiltersTest`, `SnapshotTypesTest`, `StateRetentionTest`, `RapidProjectCompletionIT`. Rule id `RAPID_COMPLETION_NON_HVAC`. Deployed ondemand/stage/preprod/prod. **E2E `AllureId 87945` passed on preprod (Jenkins 6917, 6920)**, seeding a PLUMBING merchant via the new category overload. Note the test sits exactly on its threshold: $50,000 is the credit ceiling for the arix-eligible profile, so the offer lands at $50,000 against a `>= 50000` rule with no margin. |
| R5 Rapid completion, HVAC ≥ $30,000 within 2 days | Done | Done | COVERED | HI-7960. Rule id `RAPID_COMPLETION_HVAC`. **`HVAC_CATEGORIES = Set.of("HVAC")` — `HVAC_PLUS` is deliberately excluded and falls to the R4 $50K threshold** (`hi-fraud-flink-job#56`), contradicting HI-7455 AC4 and HI-7960's own rule text. **E2E `AllureId 87920` passed on preprod (Jenkins 6868, 6920, 6925)**, asserting evidence `category=HVAC` and `minimumAmount=30000`. |
| Shared rule-id enum | Done | N/A | COVERED | **NEW since last refresh**: `com.credify.hi.common.fraud.FraudRuleId` now exists in `hi-common-lib` with exactly four values (`RAPID_COMPLETION_HVAC`, `RAPID_COMPLETION_NON_HVAC`, `FRAUD_REVIEW_VERY_HIGH`, `PAYMENT_REQUEST_SAME_DEVICE`), replacing the per-evaluator string constants. `FraudRuleIdTest` pins the set. Mirrored in qa-automation by `HomeImprovementFraudRule`. |
| Rule trigger semantics (day-delta, threshold, single-fire) | Done | Done | GAP | `RapidCompletionEvaluator`: `ChronoUnit.DAYS.between(contractDate, approvedAt in UPGRADE_ZONE_ID)`, `0 <= delta <= 2`, `fired` ValueState guards re-fire. Trigger is **final payment request APPROVED**, not `ProjectStatusChanged COMPLETED` as HI-7455 AC1/HI-7960 AC1/AC2 state. |
| Evidence payload on the emitted event | Done | Done | COVERED | R4/R5 evidence keys: `contractDate`, `approvedAt`, `daysToCompletion`, `projectAmount`, `minimumAmount`, `category`. R2: `threshold`, `windowDays`, `veryHighCount`. R1: `deviceId`, `merchantActionStatus`, `merchantActionAt`, `approvalStatus`, `approvedAt`, optional `approvalMethod`. **All four E2Es now assert their rule's evidence keys** — qa-automation#38621 lifts them into flat `evidence_*` columns in `select.fraud_rule_hit.by.project_id.and.rule_id` so assertions run on the query-result handle. |
| EXEMPT trusted-merchant handling | GAP | GAP | GAP | HI-7620 (Closed). **No `ExemptMerchants` class on master**; `RapidCompletionEvaluator` has no exempt branch; the shipped Avro event has **no `exempt` field**; `fraud_rule_hit` has **no `exempt` column**. HI-8116 explicitly declares exemption out of scope. Spec requires "still fund and just log the violation" — unimplementable as built. |
| HIMS consumer + `fraud_rule_hit` storage | Done | GAP | COVERED | HI-8116. `FraudRuleHitEventHandlerTest`, `FraudRuleHitServiceTest` (UT only — no consumer IT with `@EmbeddedKafka`). Table `fraud_rule_hit` (`HI-8116-create-fraud-rule-hit-table.changelog.xml`): merchant_id, project_id, payment_request_id, occurred_at, triggered_at, rule_id, evidence jsonb. **The four E2Es exercise this consumer against a real broker**, which is the closest thing to the missing consumer IT. |
| HIMS consumer idempotency | Done | GAP | GAP | `FraudRuleHitEventHandler` catches `DataIntegrityViolationException` and swallows only `fraud_rule_hit_uuid_uk` violations. UT-level only; no IT redelivers a duplicate event through Kafka, no E2E. |
| `lastPayment` on `HomeImprovementPaymentRequestUpdatedEvent` | GAP | GAP | GAP | HI-7621. `home-improvement-merchant-srvc#5803` changed `PaymentRequestService` with **zero tests**. Shared-schema change consumed by R4/R5's final-PR detection. |
| `previousStatus` on payment-request event | Done | N/A | COVERED | HI-7625. `PaymentRequestServiceTest` in HIMS#6180; `hi-fraud-flink-job#65` classifies transitions on it. Exercised end to end by `AllureId 88541` — R1 cannot fire at all unless `previousStatus` is populated, so the E2E pass is positive proof the field is live on preprod. |
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

- [RESOLVED 2026-09-21] **E2E coverage of rule firing is complete for all four shipped rules.** Every rule has passed end to end on preprod with its evidence assertions: R1 `88541` (Jenkins 6925), R2 `87944` (6862/6868/6925), R4 `87945` (6917/6920), R5 `87920` (6868/6920/6925). The remaining work on qa-automation#38621 is review and merge, not authoring.
- [HIGH, ESCALATED] **HI master-line opening stalls on preprod too, not just stage — it is the single biggest source of noise in this suite.** `HomeImprovementUtils.waitForEntryInAccountCandidateTable` failed **5 of 8 preprod runs** during E2E development (Jenkins 6851 stage, 6862, 6868, 6869, 6920, 6925), hitting R5 twice, R4 twice and R1 once with **no correlation to any test's inputs**. In every case the loan reached APPROVED, both batch jobs COMPLETED, and `loanreview.account_candidate` simply never received a row inside the 300s + one-retry budget. The variance is bimodal — the row arrives in seconds or not at all — which points at stuck lines rather than slow ones. The timeout was deliberately **not** widened: it would slow every HI suite that shares the helper, and if these are stuck lines it would only make them fail later. **Next step is a preprod query on whether the row ever materialises for a timed-out account** (e.g. 137760428, 137760119); that answer decides timeout-tune vs product bug. Shared framework/platform territory, not this epic — needs its own ticket.
- [HIGH] **Existing HI tests trip R5 at random, polluting `fraud_rule_hit`.** `getHomeImprovementArixEligibleRandomPerson`, the most-used HI borrower helper, draws `desiredLoanAmount` from `generateRandomBigDecimalFromRange(6000, 50000)`, and test merchants default to HVAC primary category. Roughly 45% of runs therefore clear R5's $30,000 threshold and, completing the same day, fire the rule. Nothing asserts on it, so it buys no coverage — but stage and preprod accumulate unpredictable `fraud_rule_hit` rows. If Ops reporting or Tableau is built on that table, test traffic is already polluting it.
- [HIGH] **R1 Same Device is live on preprod but not deployed to prod**, while R2/R4/R5 all are. `k8s-template/v2/applications/hi-fraud-flink-job-same-device` has ondemand/stage/preprod only (#292681, #292954) — no prod directory. HI-7625 is Resolved and HI-7461 (Deployment Checklist) is now Closed, so nothing in the ticket tree is tracking the missing prod rollout. Confirm whether prod is intentionally deferred.
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
- [RESOLVED 2026-09-21] **R1 Same Device is fully landed and covered.** #58 merged 2026-09-18, deployed ondemand/stage/preprod, and `AllureId 88541` passes on preprod. The null-device-id tolerance path (SMS/tacit/auto approval, HI-7455 AC2) stays at the unit layer in `PaymentRequestDeviceMapperTest`/`SameDeviceIT` — a null device id is unobservable downstream, so an E2E could only assert the absence of a row, which any unrelated failure also produces.

- [MEDIUM] **qa-automation#38621 is still open and carries a core-framework change.** Beyond the four tests it adds an `extraHeaders` overload to `GraphQLService`/`BaseService` and device-aware overloads of the create/approve payment-request mutations in `FederatedGatewayPublicService` (needed because R1 keys on the `X-UPG-DEVICE-ID` header). Every existing caller keeps the header-free path. Needs review attention on the framework diff, not just the tests. A CHANGELOG entry is included.
- [MEDIUM] **The new suite `home-improvement-fraud-triggers-tests.xml` has no Jenkins job** in `qa-jenkins-jobs`, so nothing runs these four tests on a schedule. Until that exists the coverage is real but unmonitored.
- [MEDIUM] **R2's seed loop is race-prone.** `merchantOverTenVeryHighFraudLoansTriggersFraudRuleHitTest` registers 11 accounts sequentially, giving 11 rolls per run at async backend races. Two distinct failures were seen during development (`conditioning_status` read as `SUBMIT_CALLED` not `FULL_FRAUD_CALLED`; `loan_in_review` returning zero rows straight after submission), both in shared helpers rather than the test. It passes more often than not, but expect intermittent red.

### Low Priority

- [LOW] `HI-7462` (UAT) is Closed with no linked evidence or PRs — no record of what was actually UAT'd.
- [LOW] `HI-8040` (flink-lib config + Kafka header capture) surfaced no PR on its own ticket; `flink-lib#86` is referenced only in HI-8046's prose. Header capture is load-bearing for R2 event time (fraud-review events carry no timestamp), so its provenance is worth pinning down.
- [LOW] `stage_terraform#4343` is still a **DRAFT** and `terraform_modules#7406` was **DECLINED** for rapid-completion onboarding, yet the job is deployed to prod via k8s-template. Worth confirming there is no half-finished Terraform state.

### Spec Requirement Gaps

**Spec unchanged since last refresh** — PROD/5598543889 is still v24 (2026-08-03) and the tech design HI/5777719448 still v4 (2026-06-25), both predating the 2026-09-17 refresh. No new or altered PM requirements this cycle; the list below carries forward verbatim.

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

### qa-automation#38621 (OPEN) — HI-7920: first E2E coverage for the shipped fraud rules
**Tickets**: HI-7920 (checklist), HI-8046 (R2), HI-7960 (R4/R5)
**Functionality**: The epic's first E2E coverage plus the scaffolding it needs.

**Framework additions**
- `fraud_rule_hit` queries (by project, by payment request, by project + rule) and accessors on `HomeImprovementMerchantQueries`. The rule-scoped query lifts the `evidence` jsonb keys into flat `evidence_*` columns so assertions chain on the query-result handle per `.cursor/BUGBOT.md`, instead of substring-matching raw JSON.
- `HomeImprovementFraudRule` enum — the service side has no shared rule-id enum, the ids are string constants inside each evaluator, so these must be kept in step by hand.
- `waitForFraudRuleHits` / `selectFraudRuleHits` / `selectAllFraudRuleHits` in `HomeImprovementDbUtils`, polling via `waitForCondition` with a Hamcrest matcher (the deprecated `waitingForCondition` was rejected by Bugbot).
- `createNewMerchantEmployeeUnderNewMerchant(role, isPrimaryLocation, canManageSalesPlan, primaryCategory, secondaryCategory)` overload. Needed because a project inherits the merchant's `primaryCategory` at onboarding and that value is snapshotted on the OPEN transition — it cannot be patched afterwards. The two public overloads funnel through one private method so the employee-wiring body is not duplicated.
- `HI_FRAUD_TRIGGERS_AT_PAYMENT_REQUEST` feature constant.

**E2E tests** (no TestNG group — the setup is far too long for a continuous gate)
- `merchantOverTenVeryHighFraudLoansTriggersFraudRuleHitTest` (`AllureId 87944`, R2) — **PASSING on preprod**. Registers 11 applications reusing one blocklisted SSN to force `FRAUD_REVIEW_VERY_HIGH` via the "Related Loan Fraud Blacklisted" signal, then creates a payment request on a clean project under the same merchant. Two deliberate cost cuts, both derived from the evaluator source: the very-high loans are registered but **never opened** (merchant attribution comes from `ProjectStatusChangedEvent` on any transition), and the payment request is **created but not approved** (`processElement2` fires on the first `paymentRequestUpdated` per payment request). That brings it to roughly 24 minutes against the ~1.5h Charles measured.
- `nonHvacRapidProjectCompletionTriggersFraudRuleHitTest` (`AllureId 87945`, R4) — written, not yet run. PLUMBING merchant at $60,000; asserts `evidence_category = PLUMBING` and `evidence_minimum_amount = 50000`, which also pins the category-routing branch disputed against HI-7455 AC4.
- `hvacRapidProjectCompletionTriggersFraudRuleHitTest` (`AllureId 87920`, R5) — written, not yet green.

**Fixes found by running it**
- `setHomeImprovementProjectAmount` returns `INVALID_LOAN_STATUS` once the loan is APPROVED, and `createOpenedHIMasterline` leaves it there — so the project amount cannot be set after opening. The mutation was dropped; `desiredLoanAmount` already drives the approved offer, and the threshold is now asserted as a precondition so a partial approval fails on its real cause.
- Jenkins can build a stale revision when a run is triggered within a minute of pushing: build 6858 ran `4cf501f` (2 tests) rather than `e795be3` (3 tests), which made an already-fixed failure look live.

**E2E Coverage**: R2 COVERED; R4/R5 IN DEV; R1 and R3 deliberately absent.

### hi-fraud-flink-job#58 — HI-7625: Rule 1 Same Device (MERGED 2026-09-18)

**Supersedes the earlier "(OPEN)" entry above.** Merged after `home-improvement-merchant-srvc#6180` deployed, per its own hold note — merged earlier, `previousStatus` would have been null on every event, every transition would have read as a merchant action, and the rule would have gone quiet rather than simply not improving.

**Functionality**: flags a payment request created and approved from the same device — the merchant approving in the borrower's place. Keyed by payment request; merchant actions and the borrower approval are both held in state, so whichever side arrives second makes the comparison. Classification is driven by the `status`/`previousStatus` transition pair, **not** `approvalMethod`, which is a sticky persisted column rather than a record of the current transition. `PRE_APPROVED_REVIEWERS` are excluded throughout — they approve inside the merchant's own request, so their device matches by construction. State retention is 31 days, matching Camunda's `payment.request.expiry.days`.

**Tests**: 53 test methods — `SameDeviceFiltersTest` (every transition pair incl. both stale-`approvalMethod` traps), `SameDeviceEvaluatorTest` (device matching, out-of-order arrival, fire-once, retention, key isolation), `SameDeviceIT` (full topology on a bounded source, plus the ops-agent false-positive guard and SMS/auto/no-device exclusions), `SameDeviceJobTest` (operator uid/name stability for savepoint restore), `PaymentRequestDeviceMapperTest`, `StateRetentionTest`.

**UT/IT Gaps**: none identified.

**E2E**: `AllureId 88541` in qa-automation#38621 — passed preprod 2026-09-21.

### k8s-template#292681, #292954 — HI-7625: same-device deployment

`#292681` deploys `hi-fraud-flink-job-same-device` to ondemand and stage; `#292954` adds preprod. **No prod deployment exists** — see the HIGH gap above. This is the only shipped rule not in prod.

### qa-automation#38621 (OPEN) — HI-7920: E2E for all four shipped rules

**Supersedes the earlier entry above**, which described the PR when it held one test.

**Functionality**: four E2Es, one per shipped rule, plus the scaffolding they share — `fraud_rule_hit` queries with evidence keys lifted into flat `evidence_*` columns, a `HomeImprovementFraudRule` enum mirroring hi-common-lib's `FraudRuleId`, a `waitForFraudRuleHits` poll helper (the Flink path is async and out of band of any API response), and the `HI_FRAUD_TRIGGERS_AT_PAYMENT_REQUEST` feature constant.

**Core framework change**: R1 classifies on `x_upg_device_id`, read off the event's `REQUEST_CONTEXT` Kafka header, which service-core-lib populates from the `X-UPG-DEVICE-ID` HTTP header. Nothing in the GraphQL client layer could add a header without rebuilding `Authorization` and `Origin` by hand, so `GraphQLService`/`BaseService` gain an `extraHeaders` overload and `FederatedGatewayPublicService` gains device-aware overloads of the create and approve mutations. Existing callers keep the header-free path, so the rule skips their transitions rather than firing on unrelated tests. CHANGELOG entry included.

**Two defects found and fixed during validation**:
- `setProjectDetails` populates `projectAmount` from `project.estimated_amount` — what the borrower asked for — but the payment request is capped by available credit, and the amount the rule evaluates is `ProjectAmounts.resolve` in HIMS (first non-null of adjusted, offer, estimated; for a fresh project, the offer). R4 failed with `INSUFFICIENT_AVAILABLE_CREDIT` requesting 60,000 against a 50,000 line. Both rapid-completion tests now derive one `ruleAmount` from the offer and use it for both the precondition assertion and the request. **R5 carried the same latent bug** — a partial approval below threshold would have passed its assertion and then surfaced as a missing hit.
- The same-device approval helper originally ran an arix status sync it did not need; HIMS publishes the `paymentRequestUpdated` event the rule keys on as part of the approval itself, so the sync only coupled R1 to a batch service that returned 503 on two of three consecutive runs.

**E2E coverage**: `88541` (R1), `87944` (R2), `87945` (R4), `87920` (R5) — all passed on preprod. Boundary and discriminator cases stay at the unit layer per Charles Chartrand's one-E2E-per-rule guidance.

**Status**: open, awaiting review.

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
- **`setHomeImprovementProjectAmount` only works pre-approval.** Drive the project amount from `desiredLoanAmount` before opening the line and assert it afterwards; calling the mutation post-open returns `INVALID_LOAN_STATUS`.
- **A non-HVAC project needs the merchant seeded that way.** Use the `createNewMerchantEmployeeUnderNewMerchant` overload with an explicit `primaryCategory` — the project inherits it at onboarding and it is snapshotted on OPEN, so patching `project.category` later has no effect on the rule.
- **Stage cannot currently open HI master lines** (`loanreview.account_candidate` never populates). Run HI fraud E2E on preprod until that is fixed.
- **Wait a couple of minutes after pushing before triggering Jenkins**, or confirm the build's revision — a run triggered immediately after a push can execute the previously indexed commit.
- **R2 costs ~24 minutes** and must not be given a TestNG group; it is a manual or nightly test, not a CI gate.
- **Flink jobs are not Spring Boot services** — no actuator, no GraphQL, no REST. E2E has to drive them through Kafka and observe the HIMS table; there is no synchronous handle on rule evaluation.
