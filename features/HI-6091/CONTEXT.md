---
epic: HI-6091
title: Cross Sell
spec_url: https://credify.atlassian.net/wiki/spaces/PROD/pages/4494065719
spec_url_2: https://credify.atlassian.net/wiki/spaces/PROD/pages/5856264352
status: in-development
last_refreshed: 2026-09-30
test_checklist_ticket: HI-6478
confluence_page_id: "5605752862"
qa_automation_pr: Credify/qa-automation#38403
---

# Cross Sell

## Spec Summary

The Cross Sell program offers existing Upgrade customers (PL, PCL, Deposit, HI, FlexPay) a Home Improvement loan through the merchant network. Eligible borrowers see an NBA/ITA banner on their dashboard (Directory, HI Home, Manage Payments pages), explore a contractor network filtered by zip code (configurable max radius, up to 150 miles system-wide; FE slider up to 50 miles), select up to 5 merchants (max 3 per category), complete a pre-qualification form (soft credit pull, pre-populated PI1, state disclosures where required), and share contact details with selected merchants. Merchants receive leads in an "Upgrade Leads" tab (now restricted by role, see below), can view lead details, update lead stages, and initiate loan applications directly. Merchant eligibility requires `cross_sell_enabled`, serviceable zip codes, and optionally a Google Places ID.

**The E2E suite is now a PR: `Credify/qa-automation#38403`** (branch `HI-CrossSellDirectoryPLTests`, head `08b5beea36`, OPEN, mergeable). The "local branch, not a PR" framing of the last two refreshes is obsolete. The PR adds **67 genuinely new `@Test` methods across 14 classes** (BorrowerUiTest 19, MerchantUiTest 12, BatchPreQualTest 5, MerchantBrazeEventTest 5, BorrowerApplicationTest 4, HomeImprovementPreQualificationTest 4, BrazeEventTest 3, MerchantSearchLimitsTest 3, ExplorePaginationTest 3, DeclineCooldownTest 3, StateDisclosureTest 2 (5 data rows), PersonalLoanCtaTest 2 (4 data rows), MerchantCrossSellTest 1, PreQualApiTest 1), registers every class in `home-improvement-cross-sell-tests.xml`, and has **zero `@SkipUntil` in the cross-sell package** (direct scan of the PR head: 92 `@Test` methods in `crosssell/`, none gated or disabled). `qa-automation` **master** still carries the old V1 suite: 29 `@Test` methods across 10 classes, 28 of them `@SkipUntil(... skipBefore="2050-12-31")`-gated (only `[74459]` parent report tab runs). In this document **IN DEV** means "in #38403"; **COVERED\*** means "on master but skip-gated."

**Preprod status of #38403:** the full cross-sell run (TestOps launch 2060464) was 85 passed / 14 failed. After fixes, 9 of the 14 pass (8 via Jenkins #7738, 8/8; `[87630]` after the QA DB user was granted UPDATE on `actor.actor_website` via `grant-qa-db-privileges`, stage #3351 / preprod #3352). **5 failures are expected and the tests are correct as written**: `[90473]` (waits on the HI-8259 permission stack), `[87232]`/`[87234]` (Load More breaks on a merchant with an unloadable actor profile, fix HI-8253 in `home-improvement-merchant-srvc#6302`), and two **product defects**, `[85541]` (cross-sell NBA keeps showing after the HI application is created) and `[83443]` (APP_CREATED lead reverts to EXPIRED on prequal expiry). A third defect, 0-prefixed zip codes never matching `zip_code_geo_encoding`, has no dedicated test. CT tagging in #38403: `HI_APPLICATION_SRVC` 16 API tests, `MERCHANT_DASHBOARD_ONBOARDING_UI` 6, `MERCHANT_CORE` 3, and a new group `HI_BORROWER_DASHBOARD_UI` 17 for `home-improvement-borrower-dashboard-ui`, whose CT job is `qa-jenkins-jobs#3446` (OPEN, HI-8075).

**Scope added since 2026-09-08**: 18 new child tickets (epic is now 111). Most are post-launch borrower-UI polish (explore pagination, zip persistence, View Contractor Details to the merchant website, legacy category mapping, mobile WebView header), a 5-merchant BE connection cap (HI-8108), an on-demand prequal feature flag (HI-8255), Google Places egress (HI-8152), and a **merchant role-restriction workstream** (HI-8259/HI-8260/HI-8267/HI-8298): cross-sell leads and the Cross Sell tab are limited to authorized roles through a new `read_cross_sell_lead` SpiceDB permission (`spicedb-schemas#1980`, MERGED). The V1 spec moved v26 to **v28 on 2026-09-30** (merchant Lead Management page marked "Admin", merchant new-lead email copy added, "Batch Email" listed as future P2, a "mortgage tradeline" exploration note). See Spec Requirement Gaps. The Omni spec is unchanged at v13.

**Omni Pre-Qual (CRD-19822)**: all `[X-Sell Omni Prequal][BE]` workstreams (W1, W2, W3, W5, W7, W_EXP, W_SHARE) are now Closed (Done). The model, Branch A (valid batch prequal, "you're pre-qualified" tile) vs Branch B (on-demand ITA), is unchanged from the 2026-09-08 summary and is IN DEV-covered by `[84225]`/`[84226]`/`[84236]`/`[85075]`/`[85068]`. O18 (re-decision against a locked policy version) is still unowned; HI-8098 is still Open with no PRs.

## Ticket Map

| Ticket | Title | Type | Status | PR(s) | UT/IT | E2E | Gap |
| --- | --- | --- | --- | --- | --- | --- | --- |
| HI-6395 | [BE] Design | Story | Closed | -- | -- | -- | Design-only |
| HI-6396 | [FE][Borrower] Add NBA banner image on BD | Story | Closed | bd-ui#7082 (superseded), bd-ui#7826 (MERGED, asset re-add) | N/A (FE) | COVERED* + IN DEV | Interactive banner now lives in new-repo#9 |
| HI-6469 | [BE] Google Places API Integration | Story | Closed | hi-merchant#5436 (MERGED) | Done | COVERED* + IN DEV (`[57005]`, `[57009]`) | -- |
| HI-6470 | [BE] Merchant serviceable zip code list support | Story | Closed | hi-merchant#5436 (MERGED); hi-merchant#5649 (DECLINED) | Done | COVERED* + IN DEV (`[57002]`, `[57004]`, `[74451]`) | **Defect: 0-prefixed zips (CT/MA/ME/NH/NJ/RI/VT/PR) never match `zip_code_geo_encoding`.** No test asserts it |
| HI-6494 | [FE][Borrower] Contractors exploration page + list | Story | Closed | bd-ui#7151 (superseded), new-repo#9 (MERGED) | N/A (FE) | COVERED* + IN DEV | FE now in new repo |
| HI-6495 | [FE][Borrower] Filter section for contractors exploration | Story | Closed | bd-ui#7121 (superseded), new-repo#9 | N/A (FE) | COVERED* + IN DEV (`[74452]`, `[87631]`) | -- |
| HI-6496 | [FE][Borrower] Contractors listing API | Story | Closed | bd-ui#7277 (superseded), new-repo#9 | N/A (FE) | COVERED* + IN DEV | -- |
| HI-6497 | [FE][Borrower] Pre-qualification page + form/modal | Story | Closed | bd-ui#7194 (superseded), new-repo#9; qa#37388 (MERGED) | N/A (FE) | COVERED* + IN DEV | -- |
| HI-6498 | [FE][Borrower] Pre-qualification success page | Story | Closed | bd-ui#7202 (superseded), new-repo#9 | N/A (FE) | COVERED* + IN DEV (`[74454]`) | -- |
| HI-6499 | [FE][Borrower] Info sent page | Story | Closed | -- | N/A | N/A | -- |
| HI-6506 | [BE] Cross Sell pre-qual lead design | Story | Closed | spicedb#1235 (MERGED) | N/A (schema) | -- | -- |
| HI-6533 | [BE] Check Borrower Eligibility | Story | Closed (Done) | avro-hi-lib#792, upflow2-hi-dags#407/#531, k8s ×4 (MERGED); hi-app#583, hi-merchant#5649, avro-hi-lib#797, k8s#259049 (DECLINED) | Done (DAG + avro event) | IN DEV (`[74461]`, `[85068]`/`[85070]`) | Eligibility lives in upflow2 DAGs |
| HI-6534 | [BE] Start cross sell pre-qual | Story | Closed | hi-app#570 (MERGED) | Done | COVERED* + IN DEV (`[55001]`) | -- |
| HI-6535 | [BE] Submit cross sell pre-qual | Story | Closed | hi-app#570 (MERGED) | Done | COVERED* (approved); IN DEV `[86216]` `hi_declined` Braze on a declined prequal | AAN path Won't Do (HI-7294) |
| HI-6536 | [BE] Share contact with merchant | Story | Closed | hi-app#570 (MERGED) | Done | COVERED* + IN DEV (`[74448]`, `[74450]`, `[82961]`) | -- |
| HI-6537 | [BE] Merchant user lead query | Story | Closed (Duplicate) | -- | N/A | N/A | Not a gap |
| HI-6538 | [BE] Merchant manage pre-qual lead stage | Story | Closed | hi-app#570 (MERGED) | Done | COVERED* + IN DEV (`[74457]`, `[83196]`) | -- |
| HI-6539 | [BE] Borrower Eligibility management | Story | Closed (Duplicate) | -- | N/A | N/A | Not a gap |
| HI-6540 | [BE] Borrower notification | Story | Closed (Done) | hi-app#570 (MERGED) | Done | IN DEV (`[78105]`, `[85077]`, `[83513]`, `[83516]`, `[86215]`, `[86216]`) | `[78105]`/`[85077]` updated to the `prequalDecisions` shape by qa#38324 (MERGED) |
| HI-6541 | [BE] Merchant Notifications | Story | Closed (Done) | qa#37317 (MERGED); hi-app#571 (DECLINED) | Done | IN DEV (`[86105]` new-lead, `[83515]` app-submitted, `[84335]` expiring-lead) | V1 spec v28 adds new-lead email copy (Braze template, not E2E-observable) |
| HI-6542 | [BE] Parent Portal (Phase 1) | Story | Closed | upflow2#363 (MERGED) | N/A (infra) | **COVERED** (`[74459]` runs on master, not skip-gated) | -- |
| HI-6543 | [BE][P2] Reporting & Metrics - Funnel Metrics | Story | Open | hi-app#570 (MERGED, partial) | Partial | GAP | Scope still open |
| HI-6544 | [BE] VQ Application Lookup | Story | Closed (Done) | -- | -- | -- | **Not built** per the HI-6478 audit (11/r1-2). Product should strike or confirm |
| HI-6546 | [BE] Scheduled job to remind merchants about lead expiry | Story | Closed | hi-app#570, k8s#285334 (MERGED) | Done | IN DEV (`[84335]`) | -- |
| HI-6631 | [FE] Move ContactDetailsCard to URC | Story | Closed | -- | N/A | N/A | -- |
| HI-6642 | [FE][Borrower] Share contact page | Story | Closed | bd-ui#7203 (superseded), new-repo#9 | N/A (FE) | COVERED* + IN DEV (`[82812]`) | -- |
| HI-6647 | [FE][Borrower] Featured merchant badge/sorting | Story | Closed | bd-ui#7208 (DECLINED) | N/A (FE) | **IN DEV** (`[86217]`, `[86218]`) | Was GAP. Re-implemented under HI-6720 |
| HI-6659 | [BE] CrossSell priority config for merchant | Story | Closed | hi-merchant#5102 (MERGED) | Done | IN DEV (`[84304]`) | -- |
| HI-6720 | [FE][Borrower] Featured merchant ordering API | Story | Closed | bd-ui#7494 (superseded), new-repo#9 | N/A (FE) | **IN DEV** (`[86217]` ribbon + tooltip follow `crossSellPriority`; `[86218]` no featured partner hides both) | Was PARTIAL. Ribbon behaviour is now asserted, not just used as setup |
| HI-6743 | [FE][Borrower] Google reviews API integration | Story | Closed | bd-ui#7281 (superseded), new-repo#9 | N/A (FE) | PARTIAL (`[57009]` API layer) | No UI review-modal assertion |
| HI-6745 | [FE][Borrower] Modify mobile filters | Story | Closed | bd-ui#7420 (superseded), new-repo#9 | N/A (FE) | GAP | -- |
| HI-6750 | [FE][Borrower] BE API at sharing contact flow | Story | Closed | bd-ui#7395 (superseded), new-repo#9 | N/A (FE) | COVERED* + IN DEV | -- |
| HI-6751 | [FE][Borrower] BE API at pre-qualification | Story | Closed | bd-ui#7395 (superseded), new-repo#9 | N/A (FE) | COVERED* + IN DEV | -- |
| HI-6754 | [FE][Merchant] List Upgrade Leads on homepage | Story | Closed | md-ui#747/748/751 (MERGED) | N/A (FE) | COVERED* + IN DEV (`[74458]`, `[85824]`) | -- |
| HI-6755 | [FE][Merchant] Lead Details page for Upgrade Leads | Story | Closed | md-ui#748/749 (MERGED) | N/A (FE) | COVERED* + IN DEV (`[74463]`, `[84148]`) | -- |
| HI-6756 | [FE][Merchant] New Create Application page for leads | Story | Closed (Not Needed) | -- | N/A | IN DEV (`[83127]`, `[83128]`, `[83168]`, `[83194]`, `[83195]`, `[83197]`, `[83200]`, `[83443]`) | `[83443]` fails on a **product defect**, see Active Gaps |
| HI-6766 | [FE][CCP] Cross Sell config in Merchant Features | Story | Closed | abp-ui#3622/3630 (MERGED) | N/A (FE) | COVERED* + IN DEV (`[57001]`) | -- |
| HI-6769 | [BE] NBA configuration for Cross Sell | Story | Closed (Done) | hi-app#634, nba#3398, nba#3418 (MERGED); **hi-app#1455 (MERGED 2026-10-01 UTC)** | Done (#1455: 8 UT + `PreQualificationServiceIT`) | IN DEV (`[82676]`, `[82677]`, `[84225]`, `[84226]`, `[84236]`); `[85541]` **expected fail** | **#1455 fixes only the eligibility half of the `[85541]` defect.** Verified on ondemand: with #1455 the pre-qualified tile still shows after app creation and after offer confirmation, because no next-best-action-srvc rule checks `hasStartedHiApplication` |
| HI-6770 | [FE][CCP] Borrower Servicing Zip Code | Story | Closed | abp-ui#3630/3632 (MERGED) | N/A (FE) | COVERED* + IN DEV (`[57002]`) | -- |
| HI-6771 | [FE][CCP] Places ID for Google Reviews | Story | Closed | abp-ui#3630/3631 (MERGED) | N/A (FE) | COVERED* + IN DEV (`[57005]`) | Place ID help link **not built** (HI-6478 9/r4) |
| HI-6828 | [FE][MD] Updates required by Design/Product | Task | Closed | md-ui#756 (MERGED) | N/A (FE) | N/A | -- |
| HI-6887 | [FE][Parent] Updates to support Cross Sell | Story | Closed | mpd-ui#163/166 (MERGED) | N/A (FE) | COVERED (`[74459]`) | -- |
| HI-6930 | [FE][CCP] Split Cross Sell eligibility/priority configs | Task | Closed | abp-ui#3747 (MERGED) | N/A | PARTIAL (`[84304]`, API layer) | -- |
| HI-7004 | [FE][CCP] Updates for Servicing Zip Codes config | Task | Closed | abp-ui#3766 (MERGED) | N/A | GAP | -- |
| HI-7012 | [FE][MD] Conditionally display Pre-qual features in Reporting | Task | Closed | mpd-ui#165 (MERGED) | N/A | GAP | -- |
| HI-7017 | [FE][CCP] Display Zip/Places configs if eligibleForCrossSell | Task | Closed | abp-ui#3746 (MERGED) | N/A | GAP | -- |
| HI-7036 | [FE] Cleanups (post-deployment) | Task | Open | -- | N/A (FE) | GAP | Still no PRs |
| HI-7077 | [FE][Borrower] NBA and Resumption tiles on explore contractors | Story | Closed | folded into new-repo#9 | N/A (FE) | PARTIAL (`[84987]`) | Resumption tile unasserted |
| HI-7158 | [BE] Sign Agreements when submitting cross sell | Story | Closed | hi-app#570 (MERGED) | Done | **IN DEV** (`[74455]` consent display; `[85811]` an opened agreement is signed from the read HTML uuid) | Was PARTIAL |
| HI-7223 | [BE] Add PreQualificationContact under Lead | Story | Closed | hi-app#570 (MERGED) | Done (UT) | IN DEV (`[83285]`, `[84148]`) | -- |
| HI-7243 | Heap analytics trackings | Story | Closed | new-repo#9 (MERGED) | N/A (FE, unit-tested) | N/A (not E2E-observable) | -- |
| HI-7294 | [BE] Handle AAN | Story | Closed (Won't Do) | None | N/A | N/A | Not a gap; `[85067]` guards the ineligible-copy path |
| HI-7369 | Improvement on google photo delivery | Story | Closed | hi-merchant#5631 (MERGED) | Done (4 UT + `MerchantReviewsIT`, `MerchantServiceableZipCodesIT`, re-verified) | GAP | -- |
| HI-7397 | Add idempotency-lib | Story | Closed | hi-app#810 (MERGED) | Done | COVERED* (`[55002]`) + IN DEV (`[85073]`) | -- |
| HI-7411 | [FE] New repo for HI Portal (Cross Sell) | Story | Closed | auth-sdk-ui#291, auth-ui#487, github-terraform#4729, hibdui#1-9, k8s ×5 | N/A (infra) | N/A | -- |
| HI-7441 | Verify required info sent to CDS from pre-qual | Story | Closed | hi-merchant#5777 (MERGED, UT only), lac#9187 (MERGED, **no tests**, enum only) | Partial (re-verified) | GAP | Audit: application path hard-codes `crossSellPrequal(false)` (8/r10); prequal policy version never reaches CDS (7/r11) |
| HI-7504 | Add Google maps attribution | Story | Closed | in new-repo#9 | N/A (FE) | GAP | -- |
| HI-7505 | Extra merchant placeholder images by category | Story | Resolved | **hibdui#12 (now MERGED)** | N/A (FE) | GAP | Status/PR mismatch resolved |
| HI-7506 | Hide filter categories if no merchants | Story | Closed | in new-repo#9 | N/A (FE) | **IN DEV** (`[87631]` filter offers only categories available in the area) | Was GAP |
| HI-7704 | [BE] Minimal backend defense-in-depth | Story | Closed | hi-merchant#5904 (MERGED, UT only, re-verified) | Partial | GAP | Low risk |
| HI-7754 | [Omni][BE] W7 decline record + 90-day cooldown | Story | **Closed (Done)** | hi-app#1105, #1102 (MERGED); qa#38324 (MERGED) | Done | IN DEV (`[85068]`, `[85070]`, `[85071]`) | Was In Validation. O18 still unowned |
| HI-7755 | [Omni][BE] W1 batch consumer + applicant hydration | Story | **Closed (Done)** | hi-app#1102, #1105, **#1126 (now MERGED)** | Done | COVERED* (`qa#37758`) + IN DEV (`[82282]`, `[82283]`, `[82292]`, `[82293]`, `[85073]`, `[85074]`, `[85868]`) | `[85076]` was renumbered `[85868]` (Gold Star decision leaves ACTIVE batch cross-sell untouched) |
| HI-7756 | [Omni][BE] W2 activation rendezvous | Story | **Closed (Done)** | hi-app#1102 (MERGED) | Done | IN DEV (`[82283]`, `[85072]`, `[85075]`) | -- |
| HI-7757 | [Omni][BE] W3 prequalDecisionUuid to CDS | Story | Closed (Done) | lac#9287 (MERGED) | Partial (UT only, re-verified) | GAP | Audit: prequal policy version is never passed to CDS at application (7/r11) |
| HI-7758 | [Omni][BE] W_SHARE editable contact | Story | Closed (Done) | hi-app#1038, qa#37912, qa-gql#1085 (MERGED) | Done | IN DEV (`[85004]`, `[85003]`, `[83285]`, `[84148]`) | -- |
| HI-7759 | [Omni][BE] W_EXP 45-day expiry | Story | Closed (Done) | hi-app#1102 (now linked) | Done | PARTIAL (`[83197]`, `[84387]`, `[83443]`) | Audit: batch prequal expiry is **45 days, on-demand is 30**. Two windows by design; see S29 |
| HI-7760 | [Omni][BE] W5 tests / flag / observability | Story | **Closed (Done)** | **hi-app#1126 (now MERGED)**, k8s#287607 (MERGED, flag on in non-prod) | Partial (#1126: UT only, no IT) | GAP | No E2E for flag-off |
| HI-7761 | [Omni][BE] W0 avro-decisioning-lib bump | Story | Closed (Self-Resolved) | -- | N/A | N/A | -- |
| HI-7762 | [Omni][BE] W_AMT prequal amount on borrower surfaces | Story | Closed (**resolution now Self-Resolved**, was Won't Do) | None | N/A | N/A | Title now reads "prequal amount on borrower surfaces is needed". Superseded in practice by HI-7765 |
| HI-7765 | [FE][Borrower] Hide prequal amount on explore title/banner | Story | Closed | hibdui#25 (MERGED) | N/A (FE) | PARTIAL (disclaimer-marker assertion) | Other surfaces unasserted |
| HI-7766 | [FE][Borrower] Redesign "How it works" | Story | Resolved | **hibdui#26 (now MERGED)** | N/A (FE) | IN DEV (`[84987]`) | Status/PR mismatch resolved |
| HI-7767 | [FE][Borrower] Personal-loan cross-sell CTA on explore contractors | Story | **Closed** | **hibdui#29, #40 (MERGED)**; hibdui#46 (DECLINED, OD test build) | N/A (FE) | **IN DEV** (PersonalLoanCtaTest `[88232]`-`[88236]`: CTA after at most 5 contractors, no-results card offers the PL and clears filters, links to `<public site>/funnel/new`) | Was SPEC GAP. O26 now covered |
| HI-7802 | [FE][Borrower] Pre-fill application screen (Omni) | Story | Closed (Done) | -- | N/A (FE) | PARTIAL (`[85371]` PI2 requires employment status + SSN; `[85373]`/`[85374]` TIL + agreements) | Was GAP; the BorrowerApplicationTest class exercises the post-prequal application screens |
| HI-7909 | Null prequalDecisionUuid for on-demand prequal | Story | **Closed (Done)** | hi-app#1098 (DECLINED) | Partial | GAP | Was In Validation. Closed with its only PR declined. Presumed absorbed by #1102; not verified |
| HI-7918 | Clean up pre-qual dead code | Task | Closed (Done) | hi-app#1189, qa#38091 (MERGED) | Done | N/A (cleanup) | -- |
| HI-7927 | [FE][Borrower] Re-send htmlDmsDocumentUuid + agreementReadDateTime | Story | **Resolved** | **hibdui#27 (MERGED)** | N/A (FE) | **IN DEV** (`[85811]`) | Was GAP |
| HI-7928 | [FE][Borrower] Editable contact at share step | Story | **Resolved** | **hibdui#28 (MERGED)** | N/A (FE) | IN DEV (`[85003]`, `[85067]`) | -- |
| HI-7938 | Report pulls borrower contact by applicant presence | Story | **Closed (Done)** | **upflow2-hi-dags#545 (MERGED)** | N/A (infra/DAG) | GAP | Was In Development, no PRs |
| HI-8024 | [FE][MD] Update Cross Sell tab disclaimer | Task | **Closed** | **md-ui#912 (MERGED)** | N/A (FE) | GAP | Disclaimer/empty-state copy not asserted |
| HI-8025 | [FE][MD] Contact source on Lead Details | Task | **Closed (Done)** | md-ui#899 (MERGED) | N/A (FE) | IN DEV (`[85004]`, `[83285]`, `[84148]`) | -- |
| HI-8026 | [FE][MD] Show Cross Sell tab with no projects | Task | **Closed** | **md-ui#908 (MERGED)** | N/A (FE) | GAP | Shipped. Branch helpers still give merchants a project first; the workaround can now be removed |
| HI-8027 | [FE][Borrower] Income validation ($0..$3M) | Story | **Closed** | hibdui#30 (MERGED) | N/A (FE) | IN DEV (`[85069]`, `[85078]`) | -- |
| HI-8036 | [BE] Merchant read applicant | Story | Closed (Done) | spicedb#1899, applicant-srvc#2353, hi-app#1178, k8s#284060, applicant-srvc#2249 (SB4 migration, 1 UT + IT harness) (MERGED) | Done | Indirect (`[85004]`/`[83285]`/`[84148]`) | Deny path untested |
| HI-8075 | Setup confidence score and CI gate for new cross sell services | Task | Open | **qa-jenkins-jobs#3446 (OPEN)** | N/A (CI) | N/A | #38403 adds the CT tags. The confidence-score flag in k8s-template is still not set |
| HI-8085 | [BE] CONTACT_GRANT_GIVEN is not recorded | Bug | Closed (Done) | hi-app#1201 (MERGED); hi-app#1304 (OPEN, HI-8184) | Done | GAP | No E2E on the grant marker |
| HI-8093 | [FE][Borrower] State Disclosures on prequal agreements | Story | **Resolved** | **hibdui#33 (MERGED)**, hi-app#1222 (MERGED) | N/A (FE) | **IN DEV** (`[86893]` API ×5 states, required + signed, signing-time UTC/gap assertions; `[86899]` WV borrower signs on the UI form) | Was SPEC GAP [HIGH] |
| HI-8094 | [FE][Borrower] projectNotes validation (2000 chars) | Story | **Closed** | **hibdui#34, #36 (MERGED)** | N/A (FE) | GAP | Shipped; still unasserted |
| HI-8095 | [BE] Support State disclosure | Story | **Closed (Done)** | **hi-app#1222 (MERGED)** | **Done** (`PreQualificationDocumentServiceTest`, `PreQualificationStateDisclosureConfigSyncTest`; `CrossSellMutationsIT`, `CrossSellIdempotencyIT`, `PreQualificationServiceIT`) | **IN DEV** (`[86893]`, `[86899]`) | Was SPEC GAP [HIGH] |
| HI-8097 | [BE] hi_prequal_offer into prequalDecisions array | Story | **Closed** | hi-app#1203, #1381, eaes#4443, eaes#4570 (CC-2381), avro-funnel#835, k8s#288175, **qa#38324** (all MERGED); hi-app#1352 (OPEN, HI-7530) | Done | **COVERED** on master (qa#38324 updated `[78105]`/`[85077]`) + IN DEV (`[86215]` Gold Star and cross-sell decisions coexist and dedupe) | **The 2026-09-08 HIGH scheduling risk is closed**: the E2E update merged 2026-09-10 alongside the BE |
| HI-8098 | [BE] HI prequal on prequal-decision V2 API in NBA | Story | Open | -- | GAP (no PRs) | GAP | Still the plausible home for O18 |
| HI-8103 | [BE] Optimize contactGrantRevocationJob | Story | Ready for CodeReview | hi-app#1206 (OPEN) | Done | GAP | -- |
| HI-8106 | [FE][Borrower] Block the 6th contractor | Story | **Closed** | **hibdui#36 (MERGED)** | N/A (FE) | **IN DEV** (`[88059]` connecting a sixth contractor shows the limit screen) | Was GAP |
| HI-8108 | [BE] Max allowed merchants to connect is 5 | Story | Closed (Done) | hi-app#1209 (MERGED) | **Done** (`PreQualificationFacadeTest`, `CrossSellMutationsIT`) | IN DEV (`[88059]`) | **New ticket** |
| HI-8144 | [FE][Borrower] Hide web header/footer in mobile WebView | Story | Resolved | hibdui#39 (MERGED) | N/A (FE) | GAP | **New ticket.** Mobile WebView is outside this suite's drivers; LOW |
| HI-8145 | [FE][Borrower] Remember last searched zip | Story | Closed | hibdui#40 (MERGED) | N/A (FE) | IN DEV (`[88112]`) | **New ticket** |
| HI-8146 | [FE][Borrower] View Contractor Details to profileUrl | Story | Closed (Done) | hibdui#44 (MERGED); hibdui#41 (DECLINED) | N/A (FE) | IN DEV (`[87630]`) | **New ticket.** `[87630]` needed the `actor.actor_website` DB grant |
| HI-8147 | [FE][Borrower] Map legacy merchant categories | Story | Closed | hibdui#42 (MERGED); hibdui#46 (DECLINED) | N/A (FE) | GAP | **New ticket.** CUTTING_TREES and other legacy categories not asserted |
| HI-8148 | [FE][Borrower] Cap explore at 3 merchants per category | Story | Closed (Done) | hibdui#43 (DECLINED) | N/A (FE) | IN DEV (`[85413]` BE cap at 3 per category) | **New ticket.** FE PR declined; the cap is enforced in the BE search |
| HI-8149 | [BE] Paginate merchants by service area + website | Story | Closed | hi-merchant#6137, qa-gql#1202, hibdui#44 (MERGED) | **Done** (`MerchantsByServiceAreaResolverTest`, `ActorServiceTest`, `MerchantGeoLocationServiceTest`; `MerchantServiceableZipCodesIT`) | IN DEV (`[85372]`, `[85375]`, `[87232]`-`[87234]`) | **New ticket** |
| HI-8150 | [FE][Borrower] Paginate explore with merchantsPaged | Story | Closed | hibdui#44 (MERGED) | N/A (FE) | IN DEV (`[87232]`, `[87233]`, `[87234]`, `[87631]`); `[87232]`/`[87234]` **expected fail** until HI-8253 | **New ticket** |
| HI-8152 | Google Reviews never resolve (Places egress) | Bug | In Validation | hi-merchant#6242 (MERGED); k8s ×7 (MERGED) | Partial (#6242: `GooglePlacesConfigurationTest` UT only) | COVERED* + IN DEV (`[57009]` `merchantReviews` populated) | **New ticket** |
| HI-8184 | [BE] IO-blocking tasks on a dedicated pool | Story | Ready for CodeReview | hi-app#1304 (OPEN) | Partial (4 UT; IT base-class changes only) | N/A (perf) | **New ticket** |
| HI-8253 | Resolve merchant list when one merchant is invalid | Bug | Ready for CodeReview | hi-merchant#6302 (OPEN) | Partial (3 UT incl. `MerchantGeoLocationServiceTest`, no IT) | IN DEV (`[87232]`, `[87234]`, expected fail until merged) | **New ticket** |
| HI-8255 | [BE] Feature flag for on-demand prequal | Story | Closed (Done) | hi-app#1401, k8s#297411 (MERGED) | Partial (`FeatureFlagAspectTest`, `PreQualificationServiceTest`; no IT) | GAP | **New ticket.** No E2E for flag-off |
| HI-8259 | [BE] Dedicated permission for manage cross sell lead | Story | Ready for CodeReview | hi-merchant#6302, md-ui#929 (OPEN) | Partial (#6302 UT only) | IN DEV (`[90473]`, expected fail until the stack merges) | **New ticket.** Requirement: hide the tab for Supervisor/Sales Rep/Installer; Repeat Customer (Gold Star) tab unchanged (`[57845]`) |
| HI-8260 | [FE][MD] Add permission per Leads | Story | In Validation | md-ui#929 (OPEN) | N/A (FE) | IN DEV (`[90473]`) | **New ticket** |
| HI-8261 | Cross Sell general adjustments (reviews link, card, share success) | Story | In Validation | hibdui#49 (OPEN) | N/A (FE) | GAP | **New ticket** |
| HI-8267 | Restrict Upgrade Lead data in lead queries to authorized roles | Story | Ready for CodeReview | hi-app#1450 (OPEN), spicedb#1980 (MERGED) | **Done** (#1450: 2 UT + `MerchantUserPreQualificationLeadsIT`, `MerchantUserPreQualificationLeadsCountIT`, `PreQualificationServiceIT`) | IN DEV (`[90473]` UI only) | **New ticket.** API-level deny for an unauthorized role is not asserted |
| HI-8298 | Restrict Upgrade Lead rows in report + CSV to authorized roles | Story | Open | spicedb#1980 (MERGED) | N/A (schema only so far) | GAP | **New ticket** |
| HI-8300 | Deployment plan and signoff - Cross Sell | Task | Open | -- | N/A | N/A | **New ticket** |
| HI-6478 | Cross-Sell Program: Test Coverage Checklist | Task | **In Development** (was Reopened) | qa#34134 (MERGED), qa#38403 (OPEN) | N/A (QA) | N/A | Updated today (18 AllureId fills, 85076→85868 fix, comment 4837532); description is at Jira's size limit |

\* **COVERED\*** = the test is on `qa-automation` master but carries `@SkipUntil(... skipBefore="2050-12-31")`. #38403 removes every such gate.
**IN DEV** = the test is in the open PR `Credify/qa-automation#38403` (unskipped; included in preprod launch 2060464). **COVERED** = runs on master today.

**PR Classification Summary (this refresh):** 160 unique PRs across 30+ repos are now linked to the epic (was 123). 39 are genuinely new since 2026-09-08: **12 service** (hi-app #1209/#1222/#1304/#1352/#1381/#1401/#1450/#1455, hi-merchant #6137/#6242/#6302, eaes#4570), **12 UI** (hibdui #36/#39/#40/#41/#42/#43/#44/#46/#49, md-ui #908/#912/#929), **12 infra** (k8s ×10, spicedb#1980, qa-jenkins-jobs#3446), **3 QA** (qa#38324, qa#38403, qa-gql#1202). 12 + 12 + 12 + 3 = 39. The other 36 newly visible links are historic (mostly DECLINED/superseded PRs from April-July) that the Development panel now lists; none changes a verdict. **Every one of the 12 new service PRs has tests**: 7 have UT + IT, 4 have UT only (hi-app#1401, hi-merchant#6242, #6302, eaes#4570), and #1381 is an IT fix. No new PR has zero tests. Re-checked GAP/Partial tickets (hi-merchant#5777/#5631/#5904, lac#9187/#9287, hi-app#570/#1126/#1206, applicant-srvc#2249): no verdict changes; lac#9187 remains NO TESTS (one enum file).

## PR Analysis

_(Entries from the 2026-06-19, 2026-08-21, 2026-08-31 and 2026-09-08 refreshes are carried forward verbatim below the new ones.)_

### Credify/qa-automation#38403 — Omni Prequal tests (the cross-sell E2E suite) — OPEN

_Analyzed: 2026-09-30 (head `08b5beea36`)_

**Tickets**: HI-6470, HI-7754, HI-7767, HI-8025, HI-8093, HI-8253, HI-8259 (Development panel); in practice covers most of the epic.
**Functionality**: Promotes the `HI-CrossSellDirectoryPLTests` branch to a PR. 54 files: 31 framework (page objects for explore/connect/prequal/declined/TIL/employment, the merchant Upgrade Leads and Lead pages, Braze/applicant/actor/HI DB query utilities), 22 test files, and the suite XML. Removes the root-level `HomeImprovementCrossSellBrazeEventTest` and recreates it in `crosssell/`.

**Tests**: 67 genuinely new `@Test` methods across 14 classes, 96 cross-sell test methods in total (92 in `crosssell/` plus 4 in `HomeImprovementPreQualificationTest`). No `@SkipUntil`, `enabled=false` or `SkipException` gates. New classes: `HomeImprovementCrossSellBorrowerApplicationTest`, `HomeImprovementCrossSellExplorePaginationTest`, `HomeImprovementCrossSellMerchantSearchLimitsTest`, `HomeImprovementCrossSellPersonalLoanCtaTest`, `HomeImprovementCrossSellStateDisclosureTest`, plus the three introduced on the branch on 2026-09-08 (`DeclineCooldownTest`, `MerchantBrazeEventTest`, relocated `BrazeEventTest`).

**CT tags**: `HI_APPLICATION_SRVC` 16 API tests, `MERCHANT_DASHBOARD_ONBOARDING_UI` 6, `MERCHANT_CORE` 3, new `HI_BORROWER_DASHBOARD_UI` 17.

**Preprod result**: launch 2060464 = 85 passed / 14 failed; 9 of the 14 since fixed (Jenkins #7738 8/8, plus `[87630]` after the DB grant). 5 expected failures: `[90473]`, `[87232]`, `[87234]`, `[85541]`, `[83443]`.

**Gaps**: [MEDIUM] Nothing asserts projectNotes validation (HI-8094), legacy category mapping (HI-8147), the Cross Sell tab with no projects (HI-8026), the HI-8024 disclaimer copy, or an API-level deny for unauthorized merchant roles (HI-8267).

### hi-application-srvc#1455 — Keep linked cross-sell prequal active in borrower eligibility (HI-6769) — MERGED 2026-10-01 UTC

_Analyzed: 2026-09-30_

**Functionality**: Fixes the eligibility half of the `[85541]` defect. Before: `PreQualificationSpecification.notLinkedToApplicationPredicate` stopped counting a linked prequal as active, so `evaluateCrossSellBorrowerEligibility` fell through to the still-eligible `borrower_ita_eligibility` row and returned `isEligible=true, hasActivePrequal=false`, which is GETPREQUAL targeting. #1455 keeps the linked prequal active. It also touches pre-allocation (`PreAllocationRequestMapper`, `HiclOfferPreAllocationService`) and every create-application pipeline config.
**Unit Tests**: 8 (`PreQualificationServiceTest`, `PreAllocationRequestMapperTest`, `HiclOfferPreAllocationServiceTest`, `PreAllocateOffersStepTest`, `EnrichMerchantPlanStepTest`, three pipeline-config tests).
**Integration Tests**: `PreQualificationServiceIT`.
**E2E Coverage**: `[85541]` (IN DEV) is the regression test, and it still fails. Verified on an ondemand stack with #1455: the pre-qualified tile (`ITA_HI_PREQUAL_XSELL_1` / `XSELL_HI_PREQUAL_1`) still shows after app creation and after offer confirmation (merchant- or borrower-confirmed), because no next-best-action-srvc targeting rule checks `hasStartedHiApplication`.
**Gaps**: [HIGH] Needs an NBA rule change or a product decision. The pipeline-config changes in #1455 reach every create-application channel (agent, aggregator, borrower, merchant), not just cross-sell.

### hi-application-srvc#1222 + home-improvement-borrower-dashboard-ui#33 — State disclosures at cross-sell pre-qualification (HI-8093/HI-8095) — MERGED

_Analyzed: 2026-09-30_

**Unit Tests**: `PreQualificationDocumentServiceTest`, `PreQualificationStateDisclosureConfigSyncTest`.
**Integration Tests**: `CrossSellMutationsIT`, `CrossSellIdempotencyIT`, `PreQualificationServiceIT`.
**E2E Coverage**: `[86893]` (API, 5 state data rows: disclosure required and signed, with signing-time UTC and gap assertions) and `[86899]` (WV borrower signs the disclosure on the UI form). Closes the 2026-09-08 HIGH gap N1.
**Gaps**: [LOW] Non-disclosure states are asserted only through the data rows chosen; no exhaustive state sweep.

### hi-application-srvc#1209 — Cap a cross-sell offer at five connected merchants (HI-8108) — MERGED

**UT/IT**: Done (`PreQualificationFacadeTest`, `CrossSellMutationsIT`). **E2E**: `[88059]` (UI limit screen on the sixth contractor). API-level rejection of a sixth connection is not separately asserted. The per-category limit is enforced in the merchant search (`[85413]`).

### home-improvement-merchant-srvc#6137 + qa-automation-graphql#1202 — Paginate merchants by service area, add website (HI-8149) — MERGED

**UT/IT**: Done (3 UT + `MerchantServiceableZipCodesIT`). **E2E**: `[85372]` max-results cap, `[85413]` 3 per category, `[85375]` full first page with Load More, `[87232]`-`[87234]` paging reaches every contractor exactly once and filtering resets to page 1, `[87630]` View Contractor Details links to the merchant website.

### home-improvement-merchant-srvc#6302 — Lead-tab permissions (HI-8259) + skip merchants with unloadable actor profiles (HI-8253) — OPEN

**UT/IT**: Partial: 3 UT (`HoldingCompanyEmployeeServiceTest`, `MerchantEmployeeServiceTest`, `MerchantGeoLocationServiceTest`), **no IT** for either change. **E2E**: `[87232]`/`[87234]` and `[90473]` fail until this merges. **Gaps**: [MEDIUM] No IT for the skip-invalid-merchant path, the behaviour that currently breaks Load More on preprod.

### hi-application-srvc#1450 + spicedb-schemas#1980 + merchant-dashboard-ui#929 — read_cross_sell_lead role restriction (HI-8259/HI-8260/HI-8267/HI-8298) — #1980 MERGED, others OPEN

**UT/IT**: #1450 Done (2 UT + `MerchantUserPreQualificationLeadsIT`, `MerchantUserPreQualificationLeadsCountIT`, `PreQualificationServiceIT`); spicedb/md-ui N/A. **E2E**: `[90473]` asserts the cross-sell leads tab is hidden for Supervisor, Sales Rep and Installer, and passes on an ondemand stack carrying all four PRs. `[57845]` covers the unchanged Repeat Customer (Gold Star) tab. **Gaps**: [MEDIUM] No API-level E2E asserting an unauthorized role gets no Upgrade Lead rows from the lead query (HI-8267) or the report/CSV (HI-8298, no implementation PR yet).

### hi-application-srvc#1401 — Feature flag for on-demand cross-sell prequal eligibility (HI-8255) — MERGED

**UT/IT**: Partial (`FeatureFlagAspectTest`, `PreQualificationServiceTest`; no IT). Flag enabled in non-prod by k8s#297411. **E2E**: none for flag-off. [MEDIUM].

### qa-automation#38324 — Put hi_prequal_offer in array prequalDecisions (HI-8097) — MERGED 2026-09-10

**Functionality**: The E2E half of HI-8097. Updates `HomeImprovementCrossSellBrazeEventTest` and `HomeImprovementCrossSellBatchPreQualTest` on master to the `prequalDecisions` array shape, with `PrequalDecisionQueries` and Kafka-utility support. **Closes the 2026-09-08 HIGH scheduling risk** that `[78105]`/`[85077]` would break when the BE merged. eaes#4570 (CC-2381, CC marketing/program code on prequal decisions, UT only) and hi-app#1381 (IT stabilization) followed.

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
| Cross-sell pre-qual creation | HI-6534 | Y | Y | master (SkipUntil) + #38403 | IN DEV |
| Submit cross-sell pre-qual + decision (approved path) | HI-6535, HI-6540 | Y | Y | master (SkipUntil) + #38403 | IN DEV |
| Declined path: `hi_declined` Braze + ineligible copy guard | HI-6535, HI-7294 | -- | -- | #38403 `[86216]`, `[85067]` | PARTIAL (AAN is Won't Do) |
| Share contact with merchant | HI-6536 | Y | Y | master (SkipUntil) + #38403 | IN DEV |
| Max 5 connected merchants | HI-8108, HI-8106 | Y | Y | #38403 `[88059]` (UI limit screen) | **IN DEV** (was GAP) |
| Max 3 merchants per category / max search results | HI-8148, HI-8149 | Y | Y | #38403 `[85413]`, `[85372]`, `[85375]` | **IN DEV** (new) |
| Explore pagination (Load More) | HI-8149, HI-8150, HI-8253 | Y | Partial | #38403 `[87232]`-`[87234]` | **IN DEV**, `[87232]`/`[87234]` expected fail until hi-merchant#6302 |
| Explore filter offers only available categories | HI-7506, HI-8150 | N/A | N/A | #38403 `[87631]` | **IN DEV** (was GAP) |
| Zip persistence across the session | HI-8145 | N/A | N/A | #38403 `[88112]` | **IN DEV** (new) |
| View Contractor Details to the merchant website | HI-8146 | N/A | N/A | #38403 `[87630]` | **IN DEV** (new) |
| Featured-partner ribbon + tooltip by crossSellPriority | HI-6647, HI-6720, HI-6659 | Y | Y | #38403 `[86217]`, `[86218]`, `[84304]` | **IN DEV** (was PARTIAL) |
| Legacy merchant category mapping | HI-8147 | N/A | N/A | -- | GAP (new) |
| Lead stage / lifecycle | HI-6538 | Y | Y | #38403 `[74457]`, `[83196]` | IN DEV |
| APP_CREATED lead must not revert to EXPIRED | HI-6756, HI-7759 | -- | -- | #38403 `[83443]` | **IN DEV, expected fail: PRODUCT DEFECT** |
| Merchant suspension: lead hidden + restored | HI-6541 | Y | Y | #38403 `[74449]`, `[83516]` | IN DEV |
| Serviceable zip codes + Google Places | HI-6469, HI-6470, HI-8152 | Y | Y | master (SkipUntil) + #38403 `[57002]`, `[57004]`, `[57009]` | IN DEV; **0-prefixed zip defect untested** |
| Borrower ITA eligibility persistence | HI-6533 | Y | Y | #38403 `[74461]`, `[74462]` | IN DEV (PL/PCL; Deposit/FlexPay still gap) |
| Borrower Braze notifications | HI-6540, HI-8097 | Y | Y | master `[78105]`/`[85077]` (qa#38324, new shape) + #38403 `[83513]`, `[83516]`, `[86215]`, `[86216]` | IN DEV |
| Merchant Braze notifications | HI-6541 | Y | Y | #38403 `[86105]`, `[83515]`, `[84335]` | IN DEV |
| Merchant lead-expiry reminder job | HI-6546 | Y | Y | #38403 `[84335]` | IN DEV |
| Merchant-initiated create-application flow | HI-6756 | N/A | N/A | #38403 ×8 | IN DEV |
| Post-prequal application screens (PI2, TIL, agreements) | HI-7802 | N/A | N/A | #38403 `[85371]`, `[85373]`, `[85374]` | **IN DEV** (new) |
| NBA/ITA config + definition copy | HI-6769 | Y | Y | #38403 `[82676]`, `[82677]`, `[84225]`, `[84226]`, `[84236]` | IN DEV |
| Cross-sell NBA stops once the HI application is created | HI-6769 (hi-app#1455) | Y | Y | #38403 `[85541]` | **IN DEV, expected fail: PRODUCT DEFECT** (NBA rule missing) |
| Personal-loan cross-sell CTA on explore (O26) | HI-7767 | N/A | N/A | #38403 `[88232]`-`[88236]` | **IN DEV** (was SPEC GAP) |
| State disclosures on prequal agreements | HI-8093, HI-8095 | Y | Y | #38403 `[86893]` ×5 states, `[86899]` | **IN DEV** (was HIGH SPEC GAP) |
| Agreement signing from the read HTML uuid | HI-7158, HI-7927 | Y | Y | #38403 `[74455]`, `[85811]` | **IN DEV** (was PARTIAL) |
| Merchant reads borrower applicant (authz) | HI-8036 | Y | Y | transitive via `[85004]`/`[83285]`/`[84148]` | PARTIAL (deny path untested) |
| Cross-sell leads restricted to authorized merchant roles | HI-8259, HI-8260, HI-8267, HI-8298 | Partial | Y (#1450) | #38403 `[90473]` (UI), `[57845]` (Gold Star unchanged) | **IN DEV, expected fail** until hi-app#1450, hi-merchant#6302, md-ui#929 merge; report/CSV (HI-8298) GAP |
| Merchant Lead Details shows edited contact | HI-7758, HI-8025 | Y | Y | #38403 `[85004]`, `[83285]`, `[84148]` | IN DEV |
| Pre-qualification income bounds ($0, $3M cap) | HI-8027 | N/A | N/A | #38403 `[85069]`, `[85078]` | IN DEV |
| projectNotes validation (2000 chars, valid text) | HI-8094 | N/A | N/A | -- | GAP |
| Cross Sell tab shown with no projects; disclaimer copy | HI-8026, HI-8024 | N/A | N/A | -- | GAP |
| On-demand prequal feature flag | HI-8255 | Y | -- | -- | GAP |
| Batch consumer dark-launch flag | HI-7760 | Y | -- | -- | GAP |
| Reporting & funnel metrics | HI-6543, HI-7938 | Partial | Partial | -- | GAP |
| Omni: Branch A/B routing (O1) | HI-7755, HI-7756, HI-6769 | Y | Y | #38403 `[84225]`, `[84226]`, `[84236]`, `[85075]`, `[85068]` | IN DEV |
| Omni: Eligibility ≥3 merchants/150mi (O2/S26) | Spec only | -- | -- | -- | PARTIAL |
| Omni: Amount suppression on borrower surfaces (O3) | HI-7765 | -- | -- | indirect | PARTIAL |
| Omni: Repeat-customer category exclusion (O4/O5) | Spec only / S28 | -- | -- | qa#35594 + setup only | PARTIAL |
| Omni: Monthly batch decisioning + prequal fields (O6/O7) | HI-7755, HI-7756 | Y | Y | `qa#37758` + #38403 `[82282]`, `[82283]`, `[82292]`, `[82293]` | IN DEV |
| Omni: Decline suppression / 90-day cooldown (+O29) | HI-7754 | Y | Y | #38403 `[85068]`, `[85070]`, `[85071]` | IN DEV |
| Omni: 45-day batch vs 30-day on-demand expiry (O8/S29) | HI-7759 | -- | -- | #38403 `[83197]`, `[84387]`, `[83443]` | PARTIAL |
| Omni: Sub-cohort treatment, 4 states (O9) | HI-7755/HI-7756 | Y | Y | #38403 | IN DEV |
| Omni: Latest-decision-wins (O20) | HI-7755 | Y | Y | #38403 `[85073]` (approve→approve) | PARTIAL |
| Omni: Batch backs off on in-flight on-demand | HI-7755 | Y | Y | #38403 `[85074]` | IN DEV |
| Omni: Gold Star / cross-sell coexistence | HI-7755, HI-8097 | Y | Y | #38403 `[82293]`, `[85868]`, `[86215]` | IN DEV |
| Omni: prequal-id selection at application (O17) | HI-7757 | Y (UT) | -- | #38403 `[83200]` | PARTIAL; CDS gets `crossSellPrequal(false)` on the application path |
| Omni: Re-decision against locked policy version (O18) | Unowned (HI-8098?) | -- | -- | -- | GAP; prequal policy version never passed to CDS |
| Omni: O10, O13, O14, O23, O28 | Spec only | -- | -- | -- | GAP |
| Braze persona `prequalDecisions` array | HI-8097 | Y | Y | master (qa#38324) + #38403 `[86215]` | **COVERED** (was GAP + scheduling risk) |
| Contact-grant marker lifecycle | HI-8085, HI-8103 | Y | Y | -- | GAP |

## Spec Requirement Gaps

**The V1 spec changed: v26 (2026-08-05) to v28 (2026-09-30, two edits today).** The Omni spec (`PROD/5856264352`) is unchanged at v13 (2026-07-29). Diff of v26 against v28:

### NEW REQUIREMENT (V1 spec v27/v28, 2026-09-30)

| # | Change | Priority | E2E Status |
| --- | --- | --- | --- |
| NR1 | Merchant "Lead Management List Page" now annotated **Admin** (the role that sees it). Matches the HI-8259/HI-8267/HI-8298 role-restriction workstream. The merchant email rows still say "User: Admin, Finance rep" | MEDIUM | IN DEV `[90473]` (Supervisor/Sales Rep/Installer hidden, expected fail until the stack merges). **Ask product:** the spec says Admin only; `[90473]` was written to the HI-8259 rule. Is Finance Rep in or out? |
| NR2 | Merchant "New Leads Received" email copy added: subject "You have a new lead from Upgrade", body, CTA "View My Lead" | LOW | Braze template copy is not E2E-observable; the `hi_merchant_lead_prequal` event is IN DEV (`[86105]`) |
| NR3 | "Batch Email: Future, P2" row added to merchant notifications | Info | Future scope, no test needed |
| NR4 | Lead-expiry email priority P2 to **TBD**; app-started and suspended rows priority **TBD**; the "Offer expiry date" dynamic field removed from the Day 7 / Day 20 borrower reminder | LOW | `[84335]` covers the merchant expiring-lead alert and notice. Confirm whether the borrower reminder still sends an expiry date |
| NR5 | "Explore mortgage tradeline" note added under FlexPay/Deposit eligibility | Info | Exploratory, nothing to test yet |
| NR6 | Borrower Braze rows gained a "Copy" column with Google Doc links (`hi_prequal_offer` and `hi_declined` marked N/A) | Info | Copy lives in Braze |

### Requirement areas introduced by tickets (2026-09-08 list, status now)

| # | Requirement | Status now |
| --- | --- | --- |
| N1 | State disclosures on prequal agreements (HI-8093/HI-8095) | **IN DEV** `[86893]`, `[86899]`. Was HIGH SPEC GAP |
| N2 | 6th contractor blocked with a max-selection screen (HI-8106/HI-8108) | **IN DEV** `[88059]` |
| N3 | projectNotes validation (HI-8094) | **GAP**, shipped (hibdui#34/#36) but still unasserted |
| N4 | Braze `prequalDecisions` array (HI-8097) | **COVERED** (qa#38324 on master) |
| N5 | Contact-grant marker survives rollback; revoked on expiry (HI-8085/HI-8103) | GAP |

### HI-6478 checklist audit (2026-09-30): rows describing behaviour that was NOT built

Product should strike or confirm these rows. They are not E2E gaps until product confirms the behaviour is wanted.

| HI-6478 row | Requirement as written | Finding |
| --- | --- | --- |
| 3/r21, 5/r3, 3/r23 | Multi-select contractor list | Not built |
| 1/r11 | Business-address ZIP fallback for merchant matching | Not built |
| 16/r13 | "Branch B fail-closed resolver" | Not built |
| 9/r4 | Place ID help link in CCP | Not built |
| 11/r1-2 | VQ application-lookup flags (HI-6544) | Not built |
| 12/r8, part of 12/r9 | Google metrics | Not built |
| 18/r6 | Separate repeat-customer report tab | Not built |

### CDS / prequal behaviour found in code (2026-09-30 audit)

| HI-6478 row | Finding | Priority | E2E |
| --- | --- | --- | --- |
| 8/r10 | The application path hard-codes `crossSellPrequal(false)` in hi-application-srvc `CreditDecisionRequestMapper` | HIGH | GAP |
| 7/r11 | The prequal's policy version is never passed to CDS for the application (ties to O18) | HIGH | GAP |
| -- | Batch prequal expiry is 45 days; on-demand is 30 (two windows, explains S29) | MEDIUM | PARTIAL |
| -- | CDS credit-report reuse window is **10 days**, not the 30 the checklist assumes | MEDIUM | GAP |
| -- | The batch path never sets `maxHirm1` | MEDIUM | GAP |
| 15/r7 | Application matching compares **year of birth only** | MEDIUM | GAP, untested |
| 16/r15 | An ACTIVE cross-sell borrower who is later declined still shows as pre-qualified | HIGH | GAP, untested |

### Original spec (PROD/4494065719): S1-S29

| # | Requirement | Priority | E2E Status |
| --- | --- | --- | --- |
| S1-S25 | _(unchanged; see Confluence page history)_ | -- | V1 E2E on master is skip-gated; #38403 unskips it |
| S26 | Zip-match eligibility threshold 5 to **3** merchants within 150mi | MEDIUM | SPEC GAP |
| S27 | Cease & Desist borrowers excluded from **NBA only** | LOW | SPEC GAP |
| S28 | HI+Goldstar customers no longer excluded | MEDIUM | PARTIAL |
| S29 | 45 vs 30 days | MEDIUM (was HIGH) | **Mostly explained**: the code has two windows, 45 days for batch prequals and 30 for on-demand. Remaining ask: product confirms that split is intended, and that the HI-8024 merchant disclaimer (md-ui#912, MERGED) matches it |

### Omni spec (PROD/5856264352): O1-O35

Text unchanged (v13). Status deltas since 2026-09-08: **O26 SPEC GAP → IN DEV** (`[88232]`-`[88236]`). O18 is still unowned, and the audit adds a concrete code fact: the prequal policy version never reaches CDS. Everything else is as in the 2026-09-08 table (O1, O6/O7, O9, O24, O29 IN DEV; O3, O8, O17, O20 PARTIAL; O2, O4, O5, O10, O12-O16, O19, O21-O23, O25, O27, O28, O30, O31 unchanged; O11, O32-O35 informational).

## Active Gaps

### Confirmed non-gaps (deliberately deprioritized)

1. **HI-7294 AAN**: Won't Do.
2. **HI-7762 W_AMT**: resolution changed Won't Do → Self-Resolved; amount suppression delivered by HI-7765.
3. **HI-6537, HI-6539**: Duplicate.
4. **The HI-6478 not-built rows** (multi-select contractor list, ZIP fallback, Branch B fail-closed resolver, Place ID help link, VQ lookup flags, Google metrics, separate repeat-customer report tab): pending a product strike/confirm. Not test gaps.

### Critical / HIGH

1. **[HIGH, PRODUCT DEFECT] `[85541]`: the cross-sell NBA keeps showing after the HI application is created.** hi-app#1455 (MERGED 2026-10-01 UTC) fixes the eligibility half. Verified on ondemand: the pre-qualified tile (`ITA_HI_PREQUAL_XSELL_1`/`XSELL_HI_PREQUAL_1`) still shows after app creation and after offer confirmation, because no next-best-action-srvc rule checks `hasStartedHiApplication`. Needs an NBA rule change or a product decision.
2. **[HIGH, PRODUCT DEFECT] `[83443]`: an APP_CREATED lead reverts to EXPIRED on prequal expiry.** `PreQualificationExpirationService.expireLeadsByIds` lacks the APP_CREATED exclusion that `PreQualificationService.expireLeadsFor` has.
3. **[HIGH, PRODUCT DEFECT] 0-prefixed zip codes never match `zip_code_geo_encoding`.** Borrowers in CT/MA/ME/NH/NJ/RI/VT/PR see no contractors. No test asserts it yet.
4. **[HIGH, OPERATIONAL] Merge #38403.** The epic's coverage still runs only from an open PR; master's 28 skip-gated tests do not run. CI gating also needs HI-8075 (qa-jenkins-jobs#3446 OPEN; confidence-score flag in k8s-template not set).
5. **[HIGH] CDS handoff findings**: the application path sends `crossSellPrequal(false)` (8/r10), and the prequal policy version never reaches CDS (7/r11, the O18 wiring). O18 is still unowned; HI-8098 is still Open with no PRs.
6. **[HIGH] 16/r15**: an ACTIVE cross-sell borrower who is later declined still shows as pre-qualified. Untested, and next to O20's approve→decline branch, which is also untested.
7. **[HIGH] O14 / O10 / O23**: score-gate cutoffs, null-score fairness and the restricted decline-reason set are still untested.
8. **[CARRIED OVER, HIGH] S1**: Deposit, FlexPay and existing-HI eligibility untested.

### Medium Priority

1. **[MEDIUM, WAITING ON DEV] `[90473]`**: cross-sell tab hidden for Supervisor/Sales Rep/Installer. Waits on hi-app#1450, hi-merchant#6302 and md-ui#929 (all OPEN; spicedb#1980 MERGED). Passes on an ondemand stack with all four. NR1 role question (Admin only vs Admin + Finance Rep) is open.
2. **[MEDIUM, WAITING ON DEV] `[87232]`/`[87234]`**: Load More fails on a merchant with an unloadable actor profile; fix HI-8253 is inside hi-merchant#6302 (OPEN, UT only, no IT).
3. **[MEDIUM]** No API-level deny test for unauthorized merchant roles (HI-8267 lead query, HI-8298 report/CSV), and no deny test for `read_applicant` (HI-8036).
4. **[MEDIUM]** HI-8094 projectNotes validation shipped and is unasserted.
5. **[MEDIUM]** HI-8026 (tab with no projects) and HI-8024 (disclaimer copy) shipped and are unasserted. The "give the merchant a project first" helper workaround can now be removed.
6. **[MEDIUM]** Feature flags without flag-off E2E: HI-8255 (on-demand prequal, hi-app#1401) and HI-7760 (batch consumer, hi-app#1126). Both UT only.
7. **[MEDIUM]** Contact-grant marker lifecycle (HI-8085/HI-8103) has no E2E.
8. **[MEDIUM]** Audit findings: CDS credit-report reuse window is 10 days, not 30; the batch path never sets `maxHirm1`; application matching compares year of birth only (15/r7).
9. **[MEDIUM]** HI-8147 legacy category mapping untested.
10. **[MEDIUM]** S26/O2, O4/S28, O12/O13, O16, O21, O25, O28, O31; HI-7441 and HI-7757 still UT-only; HI-6543 funnel metrics.

### Lower Priority

1. **[LOW]** S27, O11, O15, O19, O22, O27, O30.
2. **[LOW]** HI-7704 (UT only), HI-6743 (reviews API layer only), HI-7369, HI-7504, HI-7505, HI-6745 (FE, no E2E).
3. **[LOW]** HI-8144 mobile WebView header/footer: outside this suite's drivers. HI-8261 adjustments (hibdui#49 OPEN).
4. **[LOW]** NR2/NR4 copy and priority changes in the V1 spec.

### New / Watch

1. **[WATCH]** hi-app#1455's create-application pipeline changes reach every channel (agent, aggregator, borrower, merchant). Watch non-cross-sell HI application suites on the next run.
2. **[WATCH]** HI-7909 closed Done with its only PR (hi-app#1098) declined; presumed absorbed by #1102.
3. **[WATCH]** `nba#3418` ITA/banner definitions were gated at `starts_at=2050-01-01`; the E2E activates them. Confirm the prod date before signoff (HI-8300).
4. **[WATCH]** HI-8300 (deployment plan and signoff) is open with no PRs.

### Changes Since Last Refresh (2026-09-08 → 2026-09-30)

* **The E2E suite became a PR**: `qa-automation#38403` (OPEN, head `08b5beea36`). 67 new test methods across 14 classes, no skip gates, CT tags for four groups including a new `HI_BORROWER_DASHBOARD_UI`. Preprod launch 2060464: 85/14; 9 of the 14 fixed since; 5 expected failures (2 product defects, 3 waiting on open dev PRs).
* **20 ticket status changes** (19 status moves plus HI-7762's resolution change). All Omni BE workstreams (W1, W2, W5, W7) and HI-7909 moved In Validation → Closed. HI-7767, HI-7938, HI-8024, HI-8026, HI-8027, HI-8094, HI-8095, HI-8097 and HI-8106 moved to Closed. HI-7927, HI-7928 and HI-8093 moved to Resolved. HI-6478 moved Reopened → In Development. HI-7762's resolution changed Won't Do → Self-Resolved.
* **18 new child tickets** (epic 93 → 111): HI-8108, HI-8144-HI-8150, HI-8152, HI-8184, HI-8253, HI-8255, HI-8259, HI-8260, HI-8261, HI-8267, HI-8298, HI-8300.
* **39 new PRs** (12 service, 12 UI, 12 infra, 3 QA). Every new service PR carries tests.
* **Closed gaps**: state disclosures (HIGH, N1), PL cross-sell CTA (O26), 6th-contractor block (N2), explore pagination, merchant search caps, featured-partner ribbon, View Contractor Details, category filter, zip persistence, agreement HTML-uuid signing, merchant new-lead Braze, Gold Star/cross-sell `prequalDecisions` dedupe, `hi_declined` Braze. **The HI-8097 breaking-change risk closed**: qa#38324 merged with the BE on 2026-09-10.
* **New defects**: `[85541]` NBA-after-application (root cause verified in code; hi-app#1455 merged as a partial fix, NBA rule still missing), `[83443]` APP_CREATED → EXPIRED, 0-prefixed zips.
* **V1 spec changed** (v26 → v28 on 2026-09-30): NR1-NR6 above. Omni spec unchanged.
* **HI-6478 audit**: 39 rows had no recorded coverage. Lower-level coverage found for several; 9 rows describe behaviour that was not built; 7 CDS/prequal code findings recorded.
* **Infra**: QA DB user granted UPDATE on `actor.actor_website` (stage #3351, preprod #3352). Google Places egress allowlisted (HI-8152, k8s ×7). On-demand prequal flag enabled in non-prod (k8s#297411).

### Changes Since Last Refresh (2026-08-31 → 2026-09-08)

_(Carried forward; full entry in Confluence page history v14.)_ Omni core merged (hi-app#1102); O1 correction; md-ui#899 fixed the Lead Details contact source; the HI-8036 authz chain completed; O3 shipped (hibdui#25); HI-8027 shipped and covered; HI-8085 found and fixed; the local branch grew to 67 tests; 14 tickets added to the map.

### Changes Since Last Refresh (2026-08-21 → 2026-08-31)

_(Carried forward.)_ W7/HI-7754 rescope discovered; `qa#37758` and `qa#37912` merged; the merchant Lead Details contact bug found; 9 new child tickets.

### Changes Since Last Refresh (2026-06-19 → 2026-08-21)

_(Carried forward.)_ Epic grew 54 → 75 tickets; V1 shipped via `qa#34134` with a blanket `@SkipUntil`; borrower FE moved to `home-improvement-borrower-dashboard-ui`; Omni spec distilled into O1-O35; S26-S29 added.

## Deployment — Feature Flags & Config (E2E stack)

_(V1 flag table carried forward from the 2026-06-19 refresh; see Confluence history.)_

**Updated 2026-09-30:**
* `qa-automation` master: the V1 cross-sell suite stays skip-gated. #38403 removes the gate.
* `hi-application-srvc#1126` (W5 batch-consumer dark-launch flag) **MERGED**; enabled in non-prod by k8s#287607.
* `hi-application-srvc#1401` (HI-8255 on-demand prequal flag) **MERGED**; enabled in non-prod by k8s#297411.
* Google Places egress: k8s#291077 (prod), #291090 (non-prod), #294725 (stage/preprod proxy), #294736 (prod override, empty). hi-merchant#6242 adds a Referer on Places calls.
* QA DB user has UPDATE on `actor.actor_website` (grant-qa-db-privileges stage #3351, preprod #3352). `[87630]` needs it.
* `[90473]` needs spicedb-schemas#1980 (MERGED) plus hi-app#1450, hi-merchant#6302 and md-ui#929 (OPEN). Until those merge, run it on an ondemand stack with all four.
* CT for `home-improvement-borrower-dashboard-ui`: qa-jenkins-jobs#3446 (OPEN). Confidence-score flag in k8s-template not yet set (HI-8075).
* `nba#3418` definitions shipped with `starts_at=2050-01-01`; the E2E activates them.

## Decisions

_(2026-04-16 through 2026-08-31 decisions carried forward; see Confluence page history for the full list.)_

* 2026-07-09/07-16: New standalone repo `home-improvement-borrower-dashboard-ui`; borrower cross-sell FE migrated there.
* 2026-07-22: Omni Pre-Qual spec (`PROD/5856264352`) added.
* 2026-07-24: `qa-automation#34134` merged with a blanket `@SkipUntil`.
* 2026-08-21: HI-7294 and HI-7762 confirmed "Won't Do"; W7 rescoped to decline suppression only.
* 2026-08-31: Merchant Lead Details found to source contact from `Actor.profile`.
* 2026-09-01/02: The merchant applicant-read authz chain landed (`spicedb-schemas#1899` dedicated `read_applicant`; `applicant-srvc#2353`; `hi-application-srvc#1178`).
* 2026-09-02: `merchant-dashboard-ui#899` resolved the contact-source bug for `UPGRADE_LEAD` leads.
* 2026-09-03: O3 amount suppression shipped (`hibdui#25`).
* 2026-09-08: `hi-application-srvc#1102` merged; Branch A/B routing real; O18 alone stays unowned.
* 2026-09-08 (QA): remove every `@SkipUntil` from the cross-sell package.
* **2026-09-10: HI-8097's BE and E2E merged together** (`qa#38324`), so the `prequalDecisions` restructure landed without a red suite.
* **2026-09 (product, HI-8259)**: hide the cross-sell leads tab for Supervisor, Sales Rep and Installer, using a dedicated `read_cross_sell_lead` permission (`spicedb-schemas#1980` "add read_cross_sell_lead; revert supervisor lead read"). The Repeat Customer (Gold Star) tab is unchanged. On 2026-09-30 the V1 spec marked the Lead Management page "Admin".
* **2026-09-30 (QA)**: `qa-automation#38403` opened as the single cross-sell E2E PR. Tests that fail on product defects (`[85541]`, `[83443]`) or on unmerged dev PRs (`[90473]`, `[87232]`, `[87234]`) stay as written. No `@SkipUntil` is added for unshipped features.
* **2026-09-30 (QA)**: HI-6478 rows describing behaviour that was not built go back to product to strike or confirm; they are not tracked as test gaps.
* **2026-09-30 (code fact)**: two expiry windows exist by design in code: 45 days for batch prequals, 30 for on-demand. This mostly explains S29.
