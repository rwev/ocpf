# AGENTS.md — OCPF Repository Guide

## What This Is

OCPF (Open Code Portfolio Framework) is a **no-code, LLM-driven** portfolio management system. There is no source code, no build system, no tests.
The entire system is markdown files, templates, and OpenCode CLI commands. The LLM
is the runtime.

## Build / Lint / Test

There are none. This repository contains only markdown files. There is no code to
compile, lint, or test. Validation is done by the LLM at execution time.

## System Architecture

### Runtime

Commands live in `.opencode/commands/*.md` and are invoked as slash commands in
OpenCode (e.g., `/screen`, `/analyze AAPL`). They use OpenCode's command features:
- `!`cat FILE`` — shell injection that inlines file contents into the prompt
- `$1` / `$ARGUMENTS` — positional parameter substitution
- Frontmatter (`description`, `agent`, `subtask`, `model`) for command metadata

### Data Flow

```
/screen → watchlist stubs → /analyze TICKER → full thesis → /decide → trades
                                   ↑                              ↓
                            /reanalyze (parallel refresh)    PORTFOLIO.md
                                                                  ↓
                                                    /review → risk + snapshots
                                                    /report → performance report
```

### Directory Structure

```
CONFIG.md                    # Master config: objective, constraints, risk level
PORTFOLIO.md                 # Current portfolio state (gitignored)
_portfolio_template.md       # Template for PORTFOLIO.md format
watchlist/
  _template.md               # Template for all ticker analysis files
  _index.md                   # Summary table of active items (gitignored)
  {TICKER}.md                # Individual ticker analyses (gitignored)
closed/                      # Sold positions (files mv'd from watchlist/)
passed/                      # Pruned items never bought (files mv'd from watchlist/)
log/
  decisions/
    _template.md             # Trade decision log format
    _prune_template.md       # Watchlist pruning log format
    _summary_template.md     # /decide execution summary format
  reviews/_template.md       # Review summary format
  risk/_template.md          # Risk assessment format
performance/
  _template.md               # Performance report format
  RETURNS.md                 # Return history (gitignored)
  BENCHMARK.md               # Benchmark tracking (gitignored)
  SNAPSHOTS.md               # Portfolio value snapshots (gitignored)
.opencode/commands/          # All slash commands
```

## Critical Rules

### 1. No Code

Everything is markdown. Never create scripts, programs, or executable files.
Commands describe workflows for the LLM to follow — they are prompts, not code.

### 2. CONFIG.md Is the Single Source of Truth for Parameters

All constraints, thresholds, data source lists, and objectives live in CONFIG.md.
Commands load it via `!`cat CONFIG.md`` and reference it — they must NOT duplicate
or hardcode values from it. If a constraint changes, it changes in one place.

### 3. Templates Drive Output Format

Every output directory has a `_template.md`. Commands load the relevant template
via `!`cat`` and write output conforming to it. Commands must NOT inline template
content or re-enumerate fields the template already defines.

### 4. Live Prices Are Mandatory

CONFIG.md's Price Data section requires fetching CURRENT prices from the web.
Prices stored in files are historical records only. Commands do not need to repeat
this rule — CONFIG.md is always loaded in context and enforces it.

### 5. File Lifecycle

- Active analyses live in `watchlist/`
- Sold positions: `mv watchlist/{TICKER}.md closed/{TICKER}.md`
- Pruned items (never bought): `mv watchlist/{TICKER}.md passed/{TICKER}.md`
- Files are moved, never deleted
- Before moving, add a dated entry to the file's ANALYZE History section

### 6. watchlist/_index.md

A summary table of all active watchlist items. Commands that add, remove, or
modify watchlist files must update the index. `/reanalyze` does a full authoritative
rebuild from all individual files.

### 7. PORTFOLIO.md Is Source of Truth for Holdings

Active watchlist files exist regardless of whether the ticker is held.
PORTFOLIO.md determines what is currently held. The watchlist file's verdict
(Buy/Hold/Pass/Reject) is the thesis status, not the holding status. The Positions
table includes a **Stage** column (Initial / Full) to track scaling-in progress.

### 8. High Watermark Is Derived from SNAPSHOTS.md

The High Watermark (HWM) is NOT stored in PORTFOLIO.md. It is computed at runtime
by finding the maximum Total Value across all rows in `performance/SNAPSHOTS.md`.
`/decide` and `/review` both derive HWM this way when checking circuit breakers
and calculating drawdown. This prevents stale or corrupted HWM values from
persisting across rewrites.

## Risk Management Rules

### Position Sizing

CONFIG.md defines conviction-tiered default target weights with allowed ranges.
Commands must look up the conviction level and use the corresponding weight — they
must NOT invent arbitrary weights. New positions enter at **half** the target weight
(initial/starter position) and scale up to full weight after confirmation.

### Stop-Loss Is Mandatory

The Position Stop-Loss in CONFIG.md is a **hard rule**, not advisory. When a
position breaches the threshold, the default action is SELL. The only way to
override is if the watchlist file's thesis explicitly addresses why the loss is
temporary and includes a revised price target. The override must be logged.

### Circuit Breakers

Two drawdown-based circuit breakers are defined in CONFIG.md:

1. **Defensive Mode** — triggered at the first drawdown threshold. Halts all new
   buys, trims weakest positions, raises cash to the defensive target.
2. **Hard Stop** — triggered at the second drawdown threshold. Liquidates all
   positions. No new buys until drawdown recovers past the Defensive Mode level.

`/decide` checks circuit breakers in Step 0 before any other evaluation.
`/review` flags the current circuit breaker status in every review.

### Sell Discipline

Positions are sold for any of these reasons (checked in `/decide` Step 2):
1. No backing thesis (missing watchlist file)
2. Thesis broken (verdict changed to Pass/Reject)
3. Stop-loss breached (mandatory unless explicitly overridden in thesis)
4. **Price target reached** — trim at least half; full sell if conviction is Low
5. Constraint violation (overweight position, sector, or thematic concentration)
6. **Opportunity-cost upgrade** — sell Low-conviction to fund High-conviction
   (requires a two-tier conviction gap)
7. Circuit breaker liquidation

### Buy Discipline

Positions are only bought when (checked in `/decide` Step 3):
1. Thesis is fresh (within Stale Threshold)
2. Conviction is sufficient (Low only eligible during cash drag)
3. Re-entry cooldown has passed (Re-Entry Cooldown period in CONFIG.md, checked
   via Sold date in the closed file's ANALYZE History)
4. **Valuation discipline** — current price must be at or below the Price Target;
   prefer candidates with larger margin of safety
5. Portfolio fit (position count, sector weight, thematic concentration, cash)
6. Not in Defensive Mode or Hard Stop

### Thematic Concentration

Beyond GICS sector limits, CONFIG.md defines a Max Thematic Concentration limit.
Themes (e.g., AI/Cloud, Digital Advertising, EV Supply Chain) are recorded in each
watchlist file's Themes section and used by `/decide` and `/review` for cross-portfolio
concentration checks. `/screen` factors thematic diversification into candidate selection.

### Cash Management

CONFIG.md defines both a Min Cash Reserve (floor) and a **Max Cash Weight** (ceiling).
Exceeding the cash ceiling triggers a cash drag warning — the system should prioritize
screening and deploying capital when too much cash is sitting idle.

## Command Conventions

### Context Loading Pattern

Every command starts with a Context section that inlines required files:

```markdown
## Context

Portfolio configuration:
!`cat CONFIG.md`

Current portfolio state:
!`cat PORTFOLIO.md`
```

### What Goes in a Command vs. What Doesn't

**Commands SHOULD contain:**
- Workflow logic — the sequence of decisions, evaluations, and actions
- Guard conditions (stale thesis guard, re-entry guard, missing file guard)
- Circuit breaker checks and mode determination
- File mutation instructions (what to create, update, move)
- Output summaries to display to the user

**Commands SHOULD NOT contain:**
- Values that belong in CONFIG.md (thresholds, source URLs, constraint values)
- Field lists that the template already defines (metric names, table columns)
- Output format specifications that the template already shows
- "CRITICAL" warnings about live prices (CONFIG.md enforces this globally)
- Outputs sections that re-list what the workflow already describes

### Selective Reads, Not Globs

`/decide` reads individual `watchlist/{TICKER}.md` files based on what _index.md
and PORTFOLIO.md indicate are relevant. It does NOT glob `watchlist/*.md` because
that would pull in `_template.md` and `_index.md`, polluting context.

### Parallel Subagents in /reanalyze

`/reanalyze` launches one Task-tool subagent per ticker, all simultaneously.
Each subagent runs `/analyze {TICKER}` — reusing the full analysis workflow
rather than inlining a duplicate prompt. The parent command handles orchestration
(identify tickers, launch, verify, rebuild index) while `/analyze` handles the
analysis logic. This ensures reanalysis is always identical in depth to a fresh
analysis and avoids maintaining two copies of the analysis workflow.

## Guards in /decide

Guards prevent bad trades:

1. **Circuit breaker guard:** Check drawdown from HWM before anything else.
   Hard Stop = liquidate all. Defensive Mode = no new buys.
2. **Stale thesis guard:** Don't buy if Last Analyzed exceeds CONFIG.md's
   Stale Threshold. Stale data = no trade.
3. **Missing file guard:** If a held position has no `watchlist/{TICKER}.md`,
   flag it for SELL — no backing thesis.
4. **Re-entry guard:** Don't re-buy a ticker within the Re-Entry Cooldown
   period in CONFIG.md. Check the Sold date in `closed/{TICKER}.md`'s
   ANALYZE History.
5. **Valuation guard:** Don't buy if current price exceeds the Price Target
   in the watchlist file — do not chase.
6. **Thematic concentration guard:** Don't buy if adding the position would
   breach the Max Thematic Concentration limit.

## Valuation in /analyze

`/analyze` and `/reanalyze` must fill the Valuation Assessment section of the
watchlist template. This includes a Fair Value Estimate, current price vs fair
value comparison, and a Margin of Safety classification (Adequate >15% below,
Thin 5-15% below, None at/above). A Buy verdict generally requires at least a
Thin margin of safety.

## Git Conventions

### What's Tracked

Commands (`.opencode/commands/*.md`), CONFIG.md, AGENTS.md, and all `_template.md`
files. These define the system's behavior and are version-controlled.

### What's Ignored

All state and output: PORTFOLIO.md, watchlist analyses, _index.md, logs,
performance data, closed/, passed/. See `.gitignore` for the full list.

### Commit Style

This repo has no conventional commit tooling. Use clear, descriptive messages
that reflect what system behavior changed (e.g., "simplify /review constraint
checking to reference risk template").

## Editing Commands

When modifying a command file:
- Preserve the frontmatter (`---` block with `description`, `agent`)
- Preserve all `!`cat`` context injections in the Context section
- Preserve all `!`cat`` template loads in the Output Template section
- Keep workflow steps numbered sequentially
- Keep guard logic explicit — don't abstract away safety checks
- Refer to CONFIG.md and templates by reference, don't inline their content
