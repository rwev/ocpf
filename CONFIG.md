# Portfolio Configuration

## Identity
- **Portfolio Name:** OCPF Growth Portfolio
- **Inception Date:** 2026-03-12
- **Starting Capital:** $50,000
- **Currency:** USD

## Objective
Grow capital aggressive over the 1-5 year term by investing in high-quality US-list equities with strong growth characteristics. Prioritize companies with durable competitive advantages, healthy financials, and secular tailwinds. Manage risk through diversification and position sizing.

## Risk Level
**Aggressive** (Conservative | Moderate | Aggressive)

## Portfolio Constraints
- **Max Positions:** 15
- **Min Positions:** 5
- **Max Single Position Weight:** 15% of portfolio value
- **Min Cash Reserve:** 5% of portfolio value
- **Max Cash Weight:** 30% of portfolio value (unless in Defensive Mode or Hard Stop) — exceeding this triggers a cash drag warning and the system should prioritize screening and deploying capital
- **Max Sector Weight:** 30% of portfolio value in any single GICS sector
- **Max Thematic Concentration:** 40% of portfolio value in correlated positions sharing a common theme (e.g., AI/cloud, digital advertising, EV supply chain). Themes are identified by the LLM during analysis and recorded in watchlist files.
- **Position Stop-Loss:** -20% from cost basis triggers mandatory action. Default is SELL. To HOLD through a stop-loss breach, the watchlist file's thesis must explicitly address why the loss is temporary and set a revised price target. This exception must be logged in the decision log with the justification.
- **Drawdown Alert:** -10% from all-time high watermark — flag in review, increase scrutiny on all positions
- **Circuit Breaker — Defensive Mode:** At -15% drawdown from HWM, enter Defensive Mode: halt all new BUY orders, reduce position sizes to the low end of their conviction range, and prioritize trimming weakest-conviction positions to raise cash to at least 20%
- **Circuit Breaker — Hard Stop:** At -25% drawdown from HWM, liquidate all positions to cash.

## Position Sizing
Default target weights are determined by conviction level. These are starting points — the LLM may adjust within the stated range based on volatility, sector crowding, or portfolio fit, but must stay within the range.

| Conviction | Target Weight | Allowed Range |
|------------|--------------|---------------|
| High | 8% | 6–10% |
| Medium | 5% | 4–7% |
| Low | 3% | 2–4% |

- **Initial Position:** New buys enter at **half** the target weight (a starter position). The remainder is added on confirmation — price holding above entry, positive earnings, or thesis strengthening on reanalysis.
- **Full Position:** After confirmation, scale to full target weight via a second buy.
- **Max Single Position Weight** (from Portfolio Constraints) is the absolute ceiling regardless of conviction.

## Rebalancing
- **Drift Trigger:** Rebalance when any position drifts >5 percentage points from target weight
- **Preference:** Trim overweight positions rather than full liquidation unless thesis is broken
- **Holding Preference:** Favor long-term holds (>1 year)

## Price Data
- **Requirement:** ALL agents MUST fetch CURRENT live prices from the web. Never rely on prices stored in files as current — they are historical records only.
- **Sources:** Yahoo Finance, Google Finance, Finviz, or any publicly available financial data source.

## Watchlist
- **Max Active Items:** 30
- **Stale Threshold:** 30 days since last analysis
- **Re-Entry Cooldown:** 30 days after selling before re-buying the same ticker
