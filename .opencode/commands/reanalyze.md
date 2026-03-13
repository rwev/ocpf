---
description: Refresh all active watchlist items by running analysis in parallel for each ticker
agent: build
---

# Reanalyze All Watchlist Items

Refresh all active watchlist items by running analysis in parallel for every active ticker. Each ticker gets its own isolated subagent with its own context. After all complete, rebuild the watchlist index.

## Context

Portfolio configuration:
!`cat CONFIG.md`

Current portfolio state:
!`cat PORTFOLIO.md`

Current watchlist index:
!`cat watchlist/_index.md`

Watchlist output format (all updated files must conform to this):
!`cat watchlist/_template.md`

## Workflow

### Step 1: Identify Tickers to Reanalyze

Read `watchlist/_index.md`. For each row:
- Include the ticker if it has an active entry (any Verdict value — these are all active watchlist items)
- The index only contains active items, so include all rows

Compile the list of tickers to reanalyze.

### Step 2: Launch Parallel Analysis

For EVERY ticker identified in Step 1, launch a **separate** parallel subagent using the Task tool. Each subagent runs the `/analyze` command for its ticker.

**Launch all subagents simultaneously.** Do not wait for one to complete before starting the next.

The prompt for each subagent:

```
/analyze {TICKER}
```

This reuses the full `/analyze` workflow (data fetch, news, competitive analysis, themes, valuation, thesis, verdict, index update) — ensuring reanalysis is always identical to a fresh analysis in depth and methodology. The `/analyze` command already handles the update-vs-new distinction and appends to ANALYZE History.

### Step 3: Verify Completion

After all parallel subagents complete:
1. Read each updated `watchlist/{TICKER}.md`
2. Verify that **Last Analyzed** dates have been updated to today
3. Report any failures (tickers that could not be updated)

### Step 4: Rebuild Watchlist Index

Rebuild `watchlist/_index.md` from scratch by reading every `watchlist/{TICKER}.md` file (excluding `_template.md` and `_index.md`). For each file, extract:
- Ticker, Sector, Added date, Last Analyzed date, Verdict, Conviction, Target Weight

Rewrite `watchlist/_index.md` with the authoritative data:

```markdown
# Watchlist Index

**Last Updated:** {today}
**Active Items:** {count} / 30 max

| Ticker | Sector | Added | Last Analyzed | Verdict | Conviction | Target Weight |
|--------|--------|-------|---------------|---------|------------|---------------|
| ... | ... | ... | ... | ... | ... | ... |
```

### Step 5: Summary Output

Produce a summary:

```
## Reanalysis Complete — {YYYY-MM-DD}

**Tickers Updated:** {count}
**Failures:** {count}

| Ticker | Previous Verdict | Updated Verdict | Conviction Change | Key Change |
|--------|-----------------|-----------------|-------------------|------------|

### Notable Changes
- {any significant verdict or conviction changes}
- {any new risks identified}
```
