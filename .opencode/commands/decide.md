---
description: Review portfolio + watchlist theses, execute trades, and prune watchlist
agent: build
---

# Portfolio Decision & Execution

Review the current portfolio state and all watchlist theses, then autonomously execute portfolio management decisions: buy new positions, sell or trim existing positions, and prune the watchlist.

## Context

Portfolio configuration:
!`cat CONFIG.md`

Current portfolio state:
!`cat PORTFOLIO.md`

Watchlist index (summary of all active watchlist items):
!`cat watchlist/_index.md`

Portfolio value history (used to derive High Watermark for circuit breaker checks):
!`cat performance/SNAPSHOTS.md`

Recently closed positions (to check for re-entry guard — note filenames and dates):
!`ls -l closed/ 2>/dev/null || echo "No closed positions."`

After reviewing the index, read the FULL watchlist file for every ticker you need to evaluate. Read each file individually:
- For held positions: read `watchlist/{TICKER}.md` for each ticker in PORTFOLIO.md
- For buy candidates: read `watchlist/{TICKER}.md` for any ticker in _index.md with a Buy verdict
- For scale-up candidates: read `watchlist/{TICKER}.md` for any held position still at initial (half) weight
- For pruning candidates: read `watchlist/{TICKER}.md` for items that look stale or low-conviction in the index

Do NOT glob `watchlist/*.md`. Read files selectively based on the index and portfolio.

## Output Templates

Trade decision format:
!`cat log/decisions/_template.md`

Watchlist pruning format:
!`cat log/decisions/_prune_template.md`

Decision execution summary format:
!`cat log/decisions/_summary_template.md`

Portfolio state format:
!`cat _portfolio_template.md`

## Workflow

### Step 0: Check Circuit Breakers

Before any other evaluation, derive the High Watermark (HWM) from `performance/SNAPSHOTS.md` — it is the maximum Total Value across all snapshot rows. Then calculate the current portfolio drawdown: (Current Total Value - HWM) / HWM.

- **Hard Stop (drawdown exceeds Hard Stop threshold in CONFIG.md):** Liquidate ALL positions immediately. Skip Steps 2–4 entirely — proceed directly to Step 5 with SELL orders for every held position. No new buys are permitted until drawdown recovers above the Defensive Mode threshold.
- **Defensive Mode (drawdown exceeds Defensive Mode threshold in CONFIG.md):** No new BUY orders are permitted in this execution. Step 3 is skipped entirely. In Step 2, additionally flag positions for trimming to raise cash to the Defensive Mode cash target specified in CONFIG.md, prioritizing lowest-conviction positions first.
- **Normal Mode:** Proceed with all steps.

Record the current mode (Normal / Defensive / Hard Stop) in the decision summary.

### Step 1: Fetch Live Prices

Fetch current market prices from the web for every held position and every Buy-verdict ticker in the watchlist index.

### Step 2: Evaluate Existing Positions (Sell/Trim/Hold)

For each position currently held in PORTFOLIO.md:

1. **Find the watchlist file:** Read `watchlist/{TICKER}.md`
   - **If no file exists:** The position has no backing thesis. Flag for **SELL**.
2. **Check thesis integrity:**
   - Is the verdict Buy or Hold? → Thesis intact
   - Has the verdict changed to Pass or Reject? → Thesis broken → **SELL**
3. **Check stop-loss:** Calculate current gain/loss % from cost basis. If loss exceeds the Position Stop-Loss threshold in CONFIG.md:
   - **Default action is SELL.** To override and HOLD, the watchlist file must contain an explicit section in the thesis addressing why the loss is temporary, with a revised price target. Log the override justification in the decision log.
   - If no such justification exists in the thesis → **SELL** (mandatory).
4. **Check price target:** If the current price has reached or exceeded the Price Target in the watchlist file's Verdict section:
   - **TRIM** by at least half the position (take profits).
   - If conviction has dropped to Low or the thesis doesn't support further upside → **SELL** the full position.
   - Log the price target trigger in the decision log.
5. **Check portfolio constraints:**
   - If position weight exceeds Max Single Position Weight → **TRIM** to target weight
   - If sector weight exceeds Max Sector Weight → **TRIM** weakest conviction position in that sector
   - If thematic concentration exceeds Max Thematic Concentration in CONFIG.md → **TRIM** the weakest-conviction position in that theme
6. **Opportunity-cost evaluation:** After evaluating all held positions and all buy candidates (Step 3), compare: if a held position has Low conviction and there is a buy candidate with High conviction that cannot be funded, flag the Low-conviction position for **SELL** to free capital. Only do this when the conviction gap is at least two tiers (Low held vs High candidate).

Compile a list of SELL and TRIM actions with share quantities and reasoning.

### Step 3: Evaluate Watchlist for Buys

**Skip this step entirely if in Defensive Mode or Hard Stop.**

First, check the **cash drag warning**: if cash weight exceeds the Max Cash Weight in CONFIG.md, note this in the summary and prioritize deploying capital (lower the bar slightly — Medium conviction is sufficient, not just High).

For each ticker in `watchlist/_index.md` with Verdict = **Buy**:

1. **Check stale threshold:** If Last Analyzed date exceeds the Stale Threshold in CONFIG.md, **SKIP** this ticker. Do not buy based on a stale thesis.
2. **Check conviction:** Rank remaining candidates by conviction level (High > Medium > Low). Low conviction candidates are only eligible if cash drag warning is active.
3. **Check re-entry guard:** For any ticker that also has a file in `closed/`, check the Sold date in the file's ANALYZE History (the last dated entry before it was moved). If the sale occurred within the Re-Entry Cooldown period defined in CONFIG.md, **SKIP** to avoid sell-then-rebuy churn.
4. **Check valuation discipline:** Read the watchlist file. The current price must be at or below the Price Target. If the stock has already run past its target, **SKIP** — do not chase. Prefer candidates trading meaningfully below their price target (larger margin of safety).
5. **Check portfolio fit:**
   - Would adding this position violate Max Positions?
   - Would adding this position violate Max Sector Weight?
   - Would adding this position violate Max Thematic Concentration? (Check the Themes field in the watchlist file against themes of existing positions.)
   - Is there enough cash (after reserving Min Cash Reserve)?
6. **Calculate position size using conviction tiers from CONFIG.md:**
   - Look up the conviction level and its corresponding target weight range in CONFIG.md's Position Sizing table
   - **New positions enter at half the target weight** (initial/starter position) per CONFIG.md's scaling-in rule
   - Ensure half-weight does not exceed Max Single Position Weight
   - Calculate number of shares: floor(Initial Position Value / Current Price)
   - Record in the decision log that this is an **initial position** (to be scaled up later)

For **existing positions at initial (half) weight** that have confirmed (price holding above entry, positive development, or thesis reaffirmed on reanalysis):
   - These are eligible for **scale-up** to full target weight
   - Calculate additional shares needed: floor((Full Target Value - Current Position Value) / Current Price)
   - Record in the decision log that this is a **scale-up to full position**

Rank all eligible buys and scale-ups by conviction and portfolio fit. Select the top candidates that fit within cash and constraints.

### Step 4: Prune Watchlist

Review all items in `watchlist/_index.md` and evaluate for pruning:

1. **Reject items with broken thesis:** The investment case is no longer valid or the company's situation has materially deteriorated
2. **Reject items that are a poor fit:** No longer align with the portfolio's objective or risk level
3. **Reject stale items:** Last Analyzed date exceeds the Stale Threshold in CONFIG.md and the item has a Pass or low-conviction verdict
4. **Reject items that are a pass or low conviction**: A **Pass** verdict or **Low** conviction should be immediately pruned.

For **pruned watchlist items** (never bought, rejected): Add a dated rejection reason to the file's ANALYZE History, then `mv watchlist/{TICKER}.md passed/{TICKER}.md`. Remove row from _index.md.

### Step 5: Execute All Trades

Process in order: **SELLS first**, then **TRIMS**, then **BUYS/SCALE-UPS** (sells free cash for buys).

For each trade, update PORTFOLIO.md's Positions table and Cash accordingly:
- **SELL:** Remove position, add proceeds to Cash. Add a dated closing reason to the file's ANALYZE History, then `mv watchlist/{TICKER}.md closed/{TICKER}.md`. Remove row from _index.md.
- **TRIM:** Reduce shares in position, add proceeds to Cash.
- **BUY (initial):** Add new position at half target weight (record Ticker, Shares, Avg Cost, Current Price, Date Acquired today, Position Stage: Initial). Subtract cost from Cash.
- **BUY (scale-up):** Increase existing position to full target weight (update Shares, recalculate Avg Cost, Position Stage: Full). Subtract cost from Cash.

After all trades, recalculate all portfolio metrics and verify all constraints in CONFIG.md are satisfied.

### Step 6: Log Every Decision

For EACH trade executed, create a decision log file at `log/decisions/{YYYY-MM-DD}-{ACTION}-{TICKER}.md` using the trade decision template loaded above. For stop-loss overrides, include the justification. For price-target trims, note the target and current price. For opportunity-cost sells, name the higher-conviction replacement. For scale-ups, note "Scale-up from initial to full position."

If any items were pruned, create `log/decisions/{YYYY-MM-DD}-PRUNE-WATCHLIST.md` using the pruning template loaded above.

### Step 7: Final Portfolio and Index Update

After all trades and logging:
1. Rewrite PORTFOLIO.md with the final state, conforming to the portfolio state template loaded above
2. Update all portfolio-level metrics (Total Return, etc.)
3. Rewrite `watchlist/_index.md` — remove rows for tickers moved to closed/ or passed/, update the Active Items count and Last Updated date
4. Append a snapshot row to `performance/SNAPSHOTS.md`

### Step 8: Decision Summary Output

Output a complete summary of all actions taken, conforming to the decision execution summary template loaded above.
