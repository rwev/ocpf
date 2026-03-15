# AGENTS.md — OCPF Repository Guide

## What This Is

OCPF (Open Code Portfolio Framework) is a **no-code, LLM-driven** portfolio
management system. Everything is markdown — commands, config, templates, and
output. The LLM is the runtime. Never create scripts, programs, or executables.

## Build / Lint / Test

There are none. This repository contains only markdown files. No code to compile,
lint, or test. Validation is done by the LLM at execution time.

## Directory Structure

```
CONFIG.md                        # Single source of truth for all parameters
PORTFOLIO.md                     # Current portfolio state (gitignored)
_portfolio_template.md           # Template for PORTFOLIO.md format
watchlist/
  _template.md                   # Template for ticker analysis files
  _index.md                      # Summary table of active items (gitignored)
  {TICKER}.md                    # Individual analyses (gitignored)
closed/                          # Sold positions (mv'd from watchlist/)
passed/                          # Pruned items never bought (mv'd from watchlist/)
log/
  decisions/
    _template.md                 # Trade decision log format
    _prune_template.md           # Watchlist pruning log format
    _summary_template.md         # /decide execution summary format
  reviews/_template.md           # Review summary format
  risk/_template.md              # Risk assessment format
performance/
  _template.md                   # Performance report format
  SNAPSHOTS.md                   # Portfolio value snapshots (gitignored)
.opencode/commands/              # All slash commands (the runtime)
```

## Command File Format

Commands live in `.opencode/commands/*.md` and are invoked as slash commands
(e.g., `/screen`, `/analyze AAPL`).

### Frontmatter

Every command starts with YAML frontmatter:
```yaml
---
description: One-line description of what the command does
agent: build
---
```

### Shell Injection and Arguments

- `!`cat FILE`` — inlines file contents into the prompt at runtime
- `$1` / `$ARGUMENTS` — positional parameter substitution (only `/analyze` uses args)
- Conditional reads: `!`cat watchlist/$1.md 2>/dev/null || echo "No file."``

### Required Structure

Every command follows this pattern:
1. **Context section** — inlines CONFIG.md, PORTFOLIO.md, and other state files
2. **Numbered Step sections** — `### Step N:` with clear action labels
3. **Output Template section** — inlines relevant `_template.md` files

### Commands

| Command | Description | Args |
|---------|-------------|------|
| `/screen` | Find new candidates, apply quantitative filters, create watchlist stubs | None |
| `/analyze` | Deep-dive analysis on a single ticker (15-step workflow) | `$1` = TICKER |
| `/reanalyze` | Parallel refresh of all active watchlist items | None |
| `/decide` | Evaluate portfolio, execute trades, prune watchlist | None |
| `/review` | Update prices, fetch macro data, check constraints, assess risk | None |
| `/report` | Generate performance report with benchmark comparison | None |

## Critical Rules

### CONFIG.md Is the Single Source of Truth

All constraints, thresholds, and objectives live in CONFIG.md. Commands load it
via `!`cat CONFIG.md`` and reference it — never duplicate or hardcode values.
If a parameter changes, it changes in one place only. This includes:
- Portfolio constraints and position sizing
- Screening filters (quantitative hard gates)
- Technical requirements (trend filter, relative strength)
- Benchmark definition (SPY)
- Scale-up confirmation criteria (2-of-4 rule)

### Templates Drive Output Format

Every output directory has a `_template.md`. Commands load the template and write
output conforming to it. Commands must NOT inline template content or re-enumerate
fields the template already defines.

### Live Prices Are Mandatory

CONFIG.md requires fetching CURRENT prices from the web. Prices in files are
historical records only. Commands do not need to restate this — CONFIG.md
enforces it globally.

### File Lifecycle

- Active analyses live in `watchlist/`
- Sold positions: `mv watchlist/{TICKER}.md closed/{TICKER}.md`
- Pruned items (never bought): `mv watchlist/{TICKER}.md passed/{TICKER}.md`
- Files are moved, never deleted
- Before moving, add a dated entry to the file's ANALYZE History section
- Commands that add, remove, or modify watchlist files must update `_index.md`

### PORTFOLIO.md vs Watchlist Files

PORTFOLIO.md determines what is currently held. Watchlist files exist regardless
of whether the ticker is held — the verdict (Buy/Hold/Pass/Reject) is thesis
status, not holding status.

### High Watermark

HWM is NOT stored anywhere. It is derived at runtime by finding the max Total
Value across all rows in `performance/SNAPSHOTS.md`.

## Key Workflow Rules

### /screen Quantitative Gates

`/screen` applies hard quantitative filters from CONFIG.md's Screening Filters
section. Candidates must pass ALL gates (min market cap, min revenue growth,
max forward P/E, min ROE or positive FCF, max debt/equity, min volume). Any
single failure eliminates the candidate. This ensures consistency across
screening sessions.

### /analyze Depth

`/analyze` is a 15-step workflow that produces:
- Key financial metrics with qualitative assessments
- **Growth trajectory** — multi-year trend data (accelerating/decelerating), TAM, revenue quality
- **Earnings quality & balance sheet** — cash conversion, accruals, liquidity, debt health
- **Management & capital allocation** — insider ownership, guidance track record, SBC, M&A discipline
- **Technical profile** — 50/200-day MA, RSI, relative strength vs SPY, volume trend
- **Scenario valuation** — bull/base/bear cases with probabilities, weighted fair value, historical context
- **Expected return** — probability-weighted upside used for capital allocation ranking
- **Scale-up tracking** — for held Initial positions, tracks confirmation criteria

### /decide Guards (checked in order)

1. **Circuit breaker** — check drawdown from HWM first. Hard Stop = liquidate.
2. **Macro context** — fetch macro backdrop (inform-only, no hard gates).
3. **Stale thesis** — don't buy if Last Analyzed exceeds the Stale Threshold.
4. **Missing file** — held position with no watchlist file = flag for SELL.
5. **Re-entry cooldown** — check Sold date in `closed/{TICKER}.md` history.
6. **Valuation** — don't buy if price exceeds the watchlist Price Target.
7. **Trend filter** — don't buy if price is below the 50-day moving average.
8. **Thematic concentration** — don't breach Max Thematic Concentration.
9. **Scale-up confirmation** — require 2 of 4 criteria met before scaling Initial to Full.
10. **Expected return ranking** — rank buy candidates by expected return, not just conviction.

### /review Macro and Benchmark Awareness

`/review` fetches macro indicators (Fed Funds rate, 10Y yield, VIX, market
regime) and benchmarks portfolio performance against SPY. Macro context is
inform-only — it does not gate decisions but informs risk assessment and
urgency of recommended actions.

### Selective Reads, Not Globs

`/decide` reads individual `watchlist/{TICKER}.md` files based on what _index.md
and PORTFOLIO.md indicate are relevant. Never glob `watchlist/*.md` — that pulls
in templates and the index, polluting context.

### Parallel Subagents

`/reanalyze` launches one Task-tool subagent per ticker simultaneously. Each runs
`/analyze {TICKER}`, reusing the full analysis workflow rather than duplicating it.

### Position Sizing and Scale-Up

New positions enter at **half** the target weight (initial/starter). Scale to full
weight only after formalized confirmation — at least 2 of 4 criteria must be met:
price confirmation (20+ trading days above entry), earnings confirmation (beat or
guidance raise), thesis confirmation (reaffirmed on reanalysis), or catalyst
confirmation (material catalyst realized). These are tracked in the Scale-Up
Tracking section of each watchlist file and checked by `/decide`.

### Scenario Valuation

Single-point price targets are replaced by three-scenario valuation (bull/base/bear)
with probability weights. The probability-weighted fair value drives the Price Target
and Expected Return fields. This produces better capital allocation decisions and
makes the margin of safety assessment more robust.

### Benchmark Tracking

SPY is tracked alongside portfolio value in `performance/SNAPSHOTS.md`. `/review`
and `/report` calculate alpha (portfolio return - SPY return), per-position relative
performance, and return attribution (stock selection effect vs sector allocation
effect).

## Editing Commands

When modifying a command file:
- Preserve the `---` frontmatter block
- Preserve all `!`cat`` injections in Context and Output Template sections
- Keep workflow steps numbered sequentially
- Keep guard logic explicit — don't abstract away safety checks
- Reference CONFIG.md and templates, don't inline their content

**Commands SHOULD contain:** workflow logic, guard conditions, circuit breaker
checks, trend filter gates, scale-up confirmation checks, expected return ranking
logic, macro context fetching, benchmark calculations, file mutation instructions,
output summaries.

**Commands SHOULD NOT contain:** values from CONFIG.md, field lists from
templates, output format specs the template already shows.

## Git Conventions

### Tracked

Commands (`.opencode/commands/*.md`), CONFIG.md, AGENTS.md, and all `_template.md`
files. These define system behavior.

### Ignored

All runtime state: PORTFOLIO.md, watchlist analyses, _index.md, logs, performance
data, closed/, passed/. See `.gitignore`.

### Commits

No conventional commit tooling. Use clear, descriptive messages reflecting what
system behavior changed (e.g., "add thematic concentration guard to /decide").
