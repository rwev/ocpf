---
description: Deep-dive analysis on a single ticker — creates or updates watchlist/{TICKER}.md
agent: build
---

# Ticker Analysis: $1

Perform a deep-dive analysis on ticker **$1**. Fetch current financial data, news, competitive positioning, and market sentiment from the web. Write or update the ticker's watchlist file with a complete investment thesis and verdict.

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

Fetch all data points needed to fill every section of the watchlist template — Company Overview, Key Metrics table, 52-week range, and market data. Use sources listed in CONFIG.md's Price Data section.

### Step 2: Fetch News and Catalysts

Search for recent news, press releases, and analyst commentary about the company. Identify:
- Upcoming earnings dates
- Product launches or major announcements
- Regulatory developments
- Industry trends affecting the company
- Recent analyst upgrades/downgrades

### Step 3: Competitive Analysis

Briefly assess:
- Who are the main competitors?
- What is this company's competitive advantage (moat)?
- How does it compare on key metrics vs peers?

### Step 4: Identify Themes

Identify 1-3 investment themes this company is exposed to (e.g., AI/Cloud, Digital Advertising, EV Supply Chain, Cybersecurity, Healthcare Innovation). These will be recorded in the Themes section of the watchlist file and used by `/decide` for thematic concentration checks. Check the existing watchlist index and held positions to see what themes are already represented in the portfolio.

### Step 5: Valuation Assessment

Estimate a fair value for the stock using at least one method (DCF, comparable multiples, PEG-based, or other). Fill in the Valuation Assessment section of the template:
- **Fair Value Estimate** — the price you believe reflects intrinsic value
- **Margin of Safety** — how far below fair value the current price is:
  - Adequate: >15% below fair value
  - Thin: 5-15% below fair value
  - None: at or above fair value
- A Buy verdict should generally require at least a Thin margin of safety. Adequate margin of safety strengthens conviction. No margin of safety should result in Pass unless the growth trajectory is exceptional and well-supported.

### Step 6: Evaluate Fit with Portfolio Objective

Assess whether this company fits the portfolio's objective and risk level. Consider alignment with the growth objective, appropriateness for the risk level, diversification benefit (both sector and thematic), and any disqualifying concerns. This is a qualitative judgment.

### Step 7: Write Investment Thesis

Write a 2-3 paragraph investment thesis covering:
1. **Bull case:** Why this is a good investment given the portfolio's objective
2. **Business quality:** Competitive position, management, financials
3. **Valuation:** Is the current price reasonable relative to the fair value estimate? What is the margin of safety?

### Step 8: Determine Verdict

Check PORTFOLIO.md to determine if $1 is currently held. If held, use Hold (thesis intact) or Reject (thesis broken). If not held, use Buy, Pass, or Reject.

For the **Target Weight**, use CONFIG.md's Position Sizing table — look up the conviction level and use its default target weight. Adjust within the allowed range if warranted (e.g., lower end for higher-volatility names, higher end for strong margin of safety).

The **Price Target** should be derived from the valuation analysis — typically the fair value estimate or a growth-adjusted target. This is the price at which `/decide` will consider trimming.

### Step 9: Write Watchlist File

Write the complete analysis to `watchlist/$1.md` conforming exactly to the output template loaded above. Fill in every section with data from the analysis, including the new Themes and Valuation Assessment sections.

### Step 10: Update Watchlist Index

Update the row for $1 in `watchlist/_index.md`:
- If the ticker already has a row, update: Last Analyzed, Verdict, Conviction, Target Weight
- If the ticker is new, append a row

Update the **Active Items** count and **Last Updated** date.
