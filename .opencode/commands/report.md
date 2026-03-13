---
description: Generate a comprehensive portfolio performance report
agent: build
---

# Portfolio Performance Report

Generate a comprehensive performance report for the portfolio. Compile returns, benchmark comparison, risk metrics, trade history, and portfolio analytics.

## Context

Portfolio configuration:
!`cat CONFIG.md`

Current portfolio state:
!`cat PORTFOLIO.md`

Watchlist index:
!`cat watchlist/_index.md`

Performance data:
!`cat performance/RETURNS.md`
!`cat performance/BENCHMARK.md`
!`cat performance/SNAPSHOTS.md`

Recent decision logs:
!`ls log/decisions/ 2>/dev/null || echo "No decision logs yet."`

Recent review logs:
!`ls log/reviews/ 2>/dev/null || echo "No review logs yet."`

Recent risk logs:
!`ls log/risk/ 2>/dev/null || echo "No risk logs yet."`

Closed positions:
!`ls closed/ 2>/dev/null || echo "No closed positions."`

Passed items:
!`ls passed/ 2>/dev/null || echo "No passed items."`

Read the most recent files from each log directory for context.

## Output Template

Report format:
!`cat performance/_template.md`

## Workflow

### Step 1: Fetch Current Data

Fetch current market prices from the web for all held positions and the benchmark ticker from CONFIG.md. Recalculate current portfolio value with live prices.

### Step 2: Calculate All Report Metrics

Calculate all metrics needed to fill the report template. Use:
- PORTFOLIO.md for current positions and cost basis
- SNAPSHOTS.md for historical portfolio values and max drawdown
- Decision logs for trade statistics (count by type, win rate, avg gain/loss)
- Closed position files for realized return analysis

### Step 3: Generate Report

Output the report using the template loaded above. Fill in all sections with calculated data. For the Watchlist Status section, use counts from `watchlist/_index.md`, `closed/`, and `passed/`.

### Step 4: Update Returns Log

Append today's return data as a new row to `performance/RETURNS.md`, following the file's existing column headers.
