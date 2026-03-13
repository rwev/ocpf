---
description: Update prices, check constraints, assess risk, and produce a review summary
agent: build
---

# Portfolio Review

Conduct a comprehensive review of the current portfolio. Update all prices to live market data, verify adherence to portfolio constraints, assess risk, and produce a dated review summary.

## Context

Portfolio configuration:
!`cat CONFIG.md`

Current portfolio state:
!`cat PORTFOLIO.md`

Watchlist index (for cross-referencing held position theses and recommendations):
!`cat watchlist/_index.md`

## Output Templates

Review summary format:
!`cat log/reviews/_template.md`

Risk assessment format:
!`cat log/risk/_template.md`

## Workflow

### Step 1: Fetch Live Prices

Fetch the current market price from the web for every position in PORTFOLIO.md and the benchmark ticker from CONFIG.md.

### Step 2: Recalculate Portfolio State

Recalculate all position-level metrics (market value, weight, gain/loss $, gain/loss %) and portfolio-level metrics (invested, total value, cash weight, total return, high watermark, drawdown from HWM) using live prices. Rebuild the Allocation Summary table by GICS sector.

### Step 3: Update PORTFOLIO.md

Rewrite PORTFOLIO.md with all recalculated values. Set "Last Updated" to today's date.

### Step 4: Check Constraints and Circuit Breakers

Check every constraint in CONFIG.md's Portfolio Constraints section, including:
- Position count (min/max)
- Max Single Position Weight
- Min Cash Reserve
- **Max Cash Weight** — if cash exceeds the Max Cash Weight, flag a **cash drag warning**
- Max Sector Weight
- **Max Thematic Concentration** — identify the themes of each held position (from their watchlist files) and calculate total weight per theme. Flag any theme exceeding the limit.
- Position Stop-Loss (check each position individually)
- Drawdown Alert

Record each as PASS or VIOLATION with details, using the Constraint Check Results table in the risk template.

**Check circuit breakers:** Calculate drawdown from HWM. If it exceeds the Defensive Mode or Hard Stop thresholds in CONFIG.md, prominently flag the current mode and its implications in the review summary. Recommend immediate `/decide` execution if a circuit breaker is active.

### Step 5: Cross-Reference Watchlist

For each held position, check `watchlist/_index.md` for the current thesis status:
- Note any held positions where conviction has dropped to Low
- Note any held positions where verdict has changed to Pass or Reject
- Note any held positions with stale Last Analyzed dates (exceeding Stale Threshold in CONFIG.md)
- Note any held positions that do NOT appear in the watchlist index (missing thesis)
- Note any held positions still at **Initial stage** (half weight) that may be ready for scale-up confirmation
- Note any held positions where the current price has reached or exceeded the **Price Target** in the watchlist file

Include these findings in the review summary and recommended actions.

### Step 6: Risk Assessment

Summarize the portfolio's risk posture:
- Current drawdown from high watermark and **circuit breaker status** (Normal / Defensive / Hard Stop)
- Largest single-position risk (biggest weight, biggest loser)
- Sector concentration risk
- **Thematic concentration risk** — list the top 3 themes by portfolio weight
- Any stop-loss triggers approaching or breached (flag positions within 5% of the threshold)
- Any positions that have hit their price target
- Top 3 risk factors for the portfolio right now

**Assign urgency to each recommended action:**
- **URGENT** — requires immediate `/decide` execution (circuit breaker active, stop-loss breached, constraint violation)
- **SOON** — should be addressed in the next `/decide` cycle (stale thesis, approaching stop-loss, price target reached)
- **MONITOR** — informational, watch on next review (low conviction, theme crowding)

Write findings to `log/risk/{YYYY-MM-DD}.md` using the risk assessment template loaded above.

### Step 7: Update Performance Tracking

Append today's data as a new row to `performance/SNAPSHOTS.md` and `performance/BENCHMARK.md`, following each file's existing column headers.

### Step 8: Generate Review Summary

Write a review summary to `log/reviews/{YYYY-MM-DD}.md` using the review template loaded above. Include watchlist cross-reference findings in the Recommended Actions section.
