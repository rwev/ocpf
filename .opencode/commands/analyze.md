---
description: Deep-dive analysis on a single ticker — creates or updates watchlist/{TICKER}.md
agent: build
---

# Ticker Analysis: $1

Perform a deep-dive analysis on ticker **$1**. Fetch current financial data, news, competitive positioning, technical indicators, management quality, and market sentiment from the web. Write or update the ticker's watchlist file with a complete investment thesis, scenario valuation, and verdict.

## Context

Portfolio configuration:
!`cat CONFIG.md`

Current portfolio state (needed to determine if $1 is currently held):
!`cat PORTFOLIO.md`

Existing watchlist file (may not exist yet):
!`cat watchlist/$1.md 2>/dev/null || echo "No existing watchlist file for $1. This is a new analysis."`

Current watchlist index:
!`cat watchlist/_index.md`

## Output Template

All watchlist files MUST conform to this format:
!`cat watchlist/_template.md`

**If `watchlist/$1.md` exists:** This is an update. Preserve the original Added date and Screening Reason. Append to the ANALYZE History section rather than replacing it.

**If `watchlist/$1.md` does NOT exist:** This is a new direct analysis. Create the file from scratch. Set Added to today's date and Screening Reason to "Direct analysis requested by user".

## Workflow

### Step 1: Fetch Live Financial Data

Fetch all data points needed to fill the Key Metrics table, Company Overview, Growth Trajectory table, and Earnings Quality & Balance Sheet table. This includes:

**Company Overview data:** sector, industry, market cap, current price, 52-week range, average volume

**Key Metrics data:** Revenue TTM, revenue growth YoY and QoQ, net income TTM, EPS, P/E trailing and forward, P/S ratio, gross margin, operating margin, FCF margin, debt/equity, FCF TTM, ROE, dividend yield

**Growth Trajectory data (multi-year):** Revenue growth YoY, EPS growth YoY, gross margin, and operating margin for current year, prior year, and 2 years ago

**Earnings Quality data:** FCF/Net Income ratio, accruals ratio, current ratio, interest coverage, net debt/EBITDA, cash & equivalents

Use sources listed in CONFIG.md's Price Data section.

### Step 2: Fetch News and Catalysts

Search for recent news, press releases, and analyst commentary about the company. Identify:
- Upcoming earnings dates
- Product launches or major announcements
- Regulatory developments
- Industry trends affecting the company
- Recent analyst upgrades/downgrades

### Step 3: Technical Profile

Fetch technical data for the Technical Profile section:
- Current price relative to 50-day and 200-day moving averages
- RSI (14-day)
- 3-month price performance relative to SPY (relative strength)
- Recent average volume vs 3-month average volume (accumulation/distribution signal)

Write a 1-sentence technical assessment. Note: the trend filter (50-day MA gate) is enforced by `/decide` at trade time, not here — this step is informational context for the thesis.

### Step 4: Competitive Analysis

Assess:
- Who are the main competitors?
- What is this company's competitive advantage (moat type — network effects, switching costs, scale, brand, IP)?
- How does it compare on key metrics (growth, margins, market share) vs peers?

### Step 5: Management & Capital Allocation

Assess management quality and capital allocation discipline:
- CEO name and tenure
- Insider ownership percentage
- Recent insider buying/selling activity (past 6 months)
- Capital allocation track record: buybacks vs dilution, M&A history, R&D spending as % of revenue
- Guidance reliability: pattern of beats, meets, or misses over recent quarters
- Stock-based compensation as % of revenue — flag if excessive

### Step 6: Identify Themes

Identify 1-3 investment themes this company is exposed to (e.g., AI/Cloud, Digital Advertising, EV Supply Chain, Cybersecurity, Healthcare Innovation). These will be recorded in the Themes section and used by `/decide` for thematic concentration checks. Check the existing watchlist index and held positions to see what themes are already represented in the portfolio.

### Step 7: Growth Trajectory Analysis

Using the multi-year data fetched in Step 1, fill in the Growth Trajectory table and assess:
- **Trend direction:** Is revenue growth accelerating, stable, or decelerating? Same for margins.
- **TAM & Penetration:** Estimate the total addressable market and the company's current penetration. How much runway remains?
- **Revenue Quality:** What is the mix of recurring vs one-time revenue? Organic vs acquired growth? Customer concentration risk?

This section is critical for a growth portfolio — the trajectory matters more than any single data point.

### Step 8: Earnings Quality & Balance Sheet

Using the data fetched in Step 1, fill in the Earnings Quality & Balance Sheet table and assess:
- **Cash conversion:** Is FCF/Net Income above 1.0 (strong) or below 0.7 (weak)?
- **Accruals quality:** High accruals ratio (>10%) is a red flag for earnings sustainability
- **Balance sheet health:** For profitable companies — debt levels, interest coverage. For pre-profit companies — cash runway at current burn rate, upcoming financing needs.
- **Red flags:** One-time items inflating earnings, aggressive revenue recognition, excessive SBC diluting shareholders

### Step 9: Scenario Valuation

Estimate fair value using three scenarios — this replaces single-point valuation:

1. **Bull case (assign probability %):** Growth accelerates, margins expand, market re-rates the multiple upward. What is the stock worth?
2. **Base case (assign probability %):** Current trajectory continues. Growth, margins, and multiples stay roughly on trend.
3. **Bear case (assign probability %):** Growth disappoints, competitive pressure increases, macro headwinds compress the multiple.

Probabilities must sum to 100%. Calculate the **probability-weighted fair value**.

Also assess:
- **Valuation method used** (DCF, comparable multiples, PEG-based, or other)
- **Margin of Safety** — how far below the weighted fair value the current price is:
  - Adequate: >15% below fair value
  - Thin: 5-15% below fair value
  - None: at or above fair value
- **Historical valuation context:** Where does the current forward P/E or P/S sit vs its own 3-5 year range?
- **Key sensitivities:** Identify the 2 variables that most impact fair value (e.g., revenue growth rate, margin trajectory)

A Buy verdict should generally require at least a Thin margin of safety. Adequate margin of safety strengthens conviction. No margin of safety should result in Pass unless the growth trajectory is exceptional and well-supported.

### Step 10: Evaluate Fit with Portfolio Objective

Assess whether this company fits the portfolio's objective and risk level. Consider alignment with the growth objective, appropriateness for the risk level, diversification benefit (both sector and thematic), and any disqualifying concerns. This is a qualitative judgment informed by all the quantitative work above.

### Step 11: Write Investment Thesis

Write a 2-3 paragraph investment thesis covering:
1. **Bull case:** Why this is a good investment given the portfolio's objective
2. **Business quality:** Competitive position, management quality, financial health, growth trajectory
3. **Valuation:** Is the current price reasonable relative to the probability-weighted fair value? What is the margin of safety? How does the risk/reward stack up?

### Step 12: Determine Verdict

Check PORTFOLIO.md to determine if $1 is currently held. If held, use Hold (thesis intact) or Reject (thesis broken). If not held, use Buy, Pass, or Reject.

For the **Target Weight**, use CONFIG.md's Position Sizing table — look up the conviction level and use its default target weight. Adjust within the allowed range if warranted (e.g., lower end for higher-volatility names, higher end for strong margin of safety).

The **Price Target** should be derived from the probability-weighted fair value from the Scenario Valuation.

The **Expected Return** is the percentage difference between the current price and the probability-weighted fair value. This is used by `/decide` to rank capital allocation across candidates.

### Step 13: Populate Scale-Up Tracking (Updates Only)

If $1 is currently held at Initial (half-weight) stage (check PORTFOLIO.md Stage column):
- Populate the Scale-Up Tracking section with current data
- Calculate days held, price vs entry
- Check if any earnings have been reported since entry
- Check if any catalysts from the original analysis have been realized
- Note whether the thesis was reaffirmed in this reanalysis
- Assess whether the 2-of-4 confirmation criteria from CONFIG.md are met

If $1 is not held or is at Full stage, leave the Scale-Up Tracking section blank with the template placeholder text.

### Step 14: Write Watchlist File

Write the complete analysis to `watchlist/$1.md` conforming exactly to the output template loaded above. Fill in every section with data from the analysis.

### Step 15: Update Watchlist Index

Update the row for $1 in `watchlist/_index.md`:
- If the ticker already has a row, update: Last Analyzed, Verdict, Conviction, Target Weight
- If the ticker is new, append a row

Update the **Active Items** count and **Last Updated** date.
