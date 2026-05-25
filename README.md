# Feature Knowledge Base

QA feature context tracking for Credify/Upgrade — test coverage maps, PR analyses, and active gaps per epic.

## Structure

```
features/
  <EPIC_KEY>/
    CONTEXT.md    — ticket map, coverage matrix, active gaps, PR analysis
```

## Usage

Managed via `/feature-context` commands in Claude Code:
- `/feature-context init <epic_key> <spec_url>` — initialize a new feature context
- `/feature-context refresh <epic_key>` — update ticket statuses and discover new PRs
- `/feature-context status <epic_key>` — view coverage summary
- `/feature-context analyze-pr <epic_key> <repo#pr_number>` — analyze a specific PR

## Tracked Features

| Epic | Title | Status | Last Refreshed |
|---|---|---|---|
| [HI-5039](features/HI-5039/CONTEXT.md) | HI Merchant Multi-Account | pre-implementation | 2026-05-23 |
| [HI-6863](features/HI-6863/CONTEXT.md) | Merchant Reporting Improvements | in-development | 2026-05-24 |
