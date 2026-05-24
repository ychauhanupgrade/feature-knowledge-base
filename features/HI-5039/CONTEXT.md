---
epic: HI-5039
title: HI Merchant Multi-Account
spec_url: https://credify.atlassian.net/wiki/spaces/PROD/pages/4365615309/HI+Merchant+Multi-Account
status: pre-implementation
last_refreshed: 2026-05-23
test_checklist_ticket: HI-7135
confluence_page_id: ""
---

# HI-5039 — HI Merchant Multi-Account

## Spec Summary

The HI Merchant Multi-Account feature enables mid-sized merchants to operate multiple accounts under a single password-based login, without requiring SSO. This is explicitly distinct from the Parent Portal (which uses Okta SSO, merchant groups, and IT-level integrations). Each account under a multi-account maintains independent bank accounts, plan selections (pricebooks), locations, settings, employees, and reporting. The feature introduces a new **Supervisor role** that mirrors the Admin role but cannot manage bank accounts or add Admin users, and can only add Supervisor or Sales Rep users.

Key business rules: three-tier settings granularity (account-level, pricebook-level, multi-account-level); email conflict resolution when adding users whose emails already exist as standalone merchant logins (deactivation prompt for child merchants, hard error for non-children); Data Sharing Agreement required when merging accounts with different owners; password reset via forgot-password flow (no Okta). The dashboard is always scoped to the currently selected account — only the aggregated reporting view crosses account boundaries, with an added Account column and filter. Finance Rep and Group Admin roles (which exist at merchant and Parent Portal levels respectively) are explicitly absent from Multi-Account.

The feature is currently **pre-implementation** — all 5 implementation tickets are Open/To Do with no merged PRs. Tech design (HI-6726) is complete. Test checklist HI-7135 has 58 scenarios across 10 areas: VQ setup, admin navigation, Supervisor role, role set validation, email conflict resolution, new account onboarding, sales rep experience, reporting, settings/pricebook isolation, WIP cap, and usability/state-refresh. No E2E tests exist on master or any open PRs yet.

## Ticket Map

| Ticket | Summary | Status | Assignee | UT | IT | E2E | PRs |
|---|---|---|---|---|---|---|---|
| HI-6726 | [BE] Tech Design | Closed | Devin Lafrenière | N/A | N/A | N/A | none |
| HI-6975 | [FE] Placeholder | Open | Ruan Mer | N/A | N/A | GAP | none |
| HI-6998 | [BE] Support Multi-Account / Parent types (user/pass vs SSO) | Open | Devin Lafrenière | GAP | GAP | GAP | none |
| HI-6999 | [BE] Multi-Account employee invitation flow | Open | unassigned | GAP | GAP | GAP | none |
| HI-7000 | [BE] Allow parent accounts to support user management | Open | unassigned | GAP | GAP | GAP | none |
| HI-7135 | Test Coverage Checklist (QA task) | Open | Yogesh Chauhan | N/A | N/A | IN DEV | none |

**PR Classification Summary:** 0 total PRs found. All tickets are pre-implementation (no code shipped yet).

## Coverage Matrix

| Requirement Area | UT | IT | E2E | Notes |
|---|---|---|---|---|
| Multi-Account entity creation (VQ) | GAP | GAP | GAP | HI-6998 not started |
| Multi-Account / Parent type distinction (user/pass vs SSO) | GAP | GAP | GAP | HI-6998 not started |
| Multi-Account admin login & navigation | N/A | N/A | GAP | FE not started (HI-6975) |
| Supervisor role CRUD & permissions | GAP | GAP | GAP | No ticket covers Supervisor role impl yet |
| Role set validation (no Finance Rep, no Group Admin) | GAP | GAP | GAP | No ticket yet |
| Email conflict resolution (deactivate/reassign) | GAP | GAP | GAP | HI-6999 not started |
| Multi-Account employee invitation flow | GAP | GAP | GAP | HI-6999 not started |
| User management (email uniqueness check) | GAP | GAP | GAP | HI-7000 not started |
| New account onboarding (new EIN, sponsor_id) | GAP | GAP | GAP | No ticket yet |
| Aggregated reporting with account column/filter | N/A | N/A | GAP | FE not started |
| Account-level settings isolation | GAP | GAP | GAP | No ticket yet |
| Pricebook isolation (stays at account level on merge) | GAP | GAP | GAP | No ticket yet |
| WIP cap (account-level + multi-account P2) | GAP | GAP | GAP | No ticket yet |
| Usability: state & session refresh | N/A | N/A | GAP | FE not started |

## Active Gaps

### Critical — P0 Boundaries (Must Pass Before Any UAT Sign-Off)
- [HIGH] **Supervisor cannot change bank accounts** — no implementation ticket exists; this is the hardest permission boundary to enforce correctly
- [HIGH] **Supervisor cannot add Admin users** — role creation gate; must reject at API level, not just UI
- [HIGH] **Cross-account data isolation** — Sales Rep in Account A must not see Account B data; extends existing `salesRepCanViewOnlyTheProjectsCreatedByThem` pattern (AllureId 2827) across account boundaries

### High — Core Feature Gaps
- [HIGH] **Session invalidation on merge** — after VQ merges a merchant account into Multi-Account, the merchant's standalone session must be revoked; no ticket covers session management
- [HIGH] **Email deactivation does not invalidate active session** — child merchant login deactivated but session token TTL outlives deactivation; must verify immediate 401
- [HIGH] **Supervisor role** — no implementation ticket exists yet for the Supervisor role backend or frontend; entire role is untracked in the backlog

### Medium — Implementation Gaps
- [MEDIUM] **Requirement GAP — no ticket covers new account onboarding from Multi-Account portal** (new EIN redirect with sponsor_id, simplified vs full underwriting path)
- [MEDIUM] **Requirement GAP — no ticket covers role set validation** (Finance Rep absent, Group Admin absent from multi-account invite flow)
- [MEDIUM] **Requirement GAP — no ticket covers pricebook isolation** (pricebook stays at account level, not elevated to multi-account on merge)
- [MEDIUM] **Account switcher state refresh** — no FE ticket details how components re-fetch on account context switch; SPA stale cache risk

### Spec Requirement Gaps (no ticket mapping)
- Supervisor role (entire role — no BE or FE ticket)
- New account onboarding from multi-account portal (new EIN → Merchant Application with sponsor_id)
- Finance Rep / Group Admin role exclusion validation
- Account-level settings isolation across sibling accounts
- Pricebook level isolation during and after account merge
- Data Sharing Agreement enforcement when merging accounts with different owners
- VQ multi-account entity view (parent link on child accounts)
- WIP cap at multi-account aggregate level (P2)
- All usability/state-refresh scenarios (11 scenarios in HI-7135)

## PR Analysis

*No PRs yet — all tickets are pre-implementation as of 2026-05-23.*

## Key Decisions

- Multi-Account uses **password login**, not Okta SSO — this is architecturally distinct from Parent Portal (HI-6998 covers the auth type distinction)
- **Finance Rep role is explicitly absent** from Multi-Account per spec; roles are: Admin, Supervisor (new), Sales Rep only
- **Group Admin** (exists in Parent Portal) must NOT appear in Multi-Account user management
- **Pricebooks remain at account level** even after merge — do not create separate accounts for pricebooks
- **Data Sharing Agreement required** before access is granted when merging accounts with different owners
- Dashboard is **always account-scoped**; aggregated view only available in reporting to Admins

## Notes for SDET

- Existing tests to extend/reuse: AllureId 2827 (Sales Rep scoping), 4870/4871/7854 (Finance Rep), 1370 (password reset), 51276 (multi-tab), 59027 (pricebook Admin access), 13939/13940 (WIP cap)
- Jira Basic Auth token NOT set up — run `.claude/script/get_jira_dev_info.sh` after setting up: `security add-generic-password -a "$USER" -s "jira-basic-auth" -w "$(echo -n 'ychauhan@upgrade.com:YOUR_API_TOKEN' | base64)"`
- No implementation PRs to analyze yet; re-run `/feature-context refresh HI-5039` once HI-6998/6999/7000 move to In Progress
