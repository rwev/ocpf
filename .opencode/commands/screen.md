---
description: Search for new investment candidates and add stubs to watchlist/
agent: build
---

# Stock Screener

Search for new investment candidates that align with the portfolio's objective and risk level. Apply quantitative hard gates to filter candidates, then produce lightweight stub files in `watchlist/` for each candidate that passes. This command does NOT perform deep analysis — it identifies candidates for `/analyze` to evaluate.

## Context

Portfolio configuration:
!`cat CONFIG.md`

Current portfolio (to avoid screening for held tickers):
!`cat PORTFOLIO.md`

Current watchlist index (to avoid duplicates):
!`cat watchlist/_index.md`

## Output Template

Watchlist file format (fill in only the Status, Company Overview, and stub Thesis fields — leave the rest as placeholders for `/analyze` to complete):
!`cat watchlist/_template.md`

## Workflow

### Step 1: Find Candidates

Check PORTFOLIO.md cash weight against the Max Cash Weight in CONFIG.md. If cash is above this threshold, note the **cash drag warning** — the portfolio needs to deploy capital, so screening is a priority.

Search publicly available sources for US equities that fit the portfolio's objective. Review current portfolio sector weights AND thematic exposure to identify areas that are underweight or overrepresented and factor this into your search. Favor candidates that bring thematic diversification — avoid screening for more of what the portfolio already has concentrated exposure to.

Identify 10-15 potential candidates before filtering.

### Step 2: Apply Quantitative Filters

Apply every hard gate from CONFIG.md's Screening Filters section to each candidate. A candidate must pass **ALL** filters to proceed:
- Min Market Cap
- Min Revenue Growth (YoY)
- Max Forward P/E
- Min ROE (or positive FCF if pre-profit)
- Max Debt/Equity
- Min Avg Daily Volume

Fetch the required data points for each candidate from publicly available financial sources. Reject any candidate that fails any single filter. Track which filter eliminated each rejected candidate for the summary.

Also filter out tickers already in `watchlist/_index.md` or held in PORTFOLIO.md.

### Step 3: Select Top Candidates

From the candidates that passed all quantitative gates, select the top 3-5. Rank by:
1. Fit with the portfolio's objective
2. Diversification benefit (sector and thematic)
3. Valuation attractiveness
4. Relative strength vs the market (prefer candidates showing positive recent momentum)

### Step 4: Create Watchlist Stubs

For each selected candidate, create a new file `watchlist/{TICKER}.md` using the template loaded above. For stub entries:
- Set **Watchlist Status** to Active
- Set **Added** to today's date
- Set **Last Analyzed** to Pending
- Fill in **Screening Reason**, **Sector**, **Industry**, **Market Cap**, **Price**
- Set **Thesis** to "Pending deep analysis. See Screening Reason above."
- Leave all analysis sections (Key Metrics, Growth Trajectory, Earnings Quality, Management, Catalysts, Risks, Competitive Position, Technical Profile, Themes, Scenario Valuation, Verdict, Scale-Up Tracking) as placeholders for `/analyze` to complete

### Step 5: Update Watchlist Index

Append a new row to `watchlist/_index.md` for each candidate added:
```
| {TICKER} | {Sector} | {today} | Pending | Pending | Pending | Pending |
```

Update the **Active Items** count and **Last Updated** date at the top of _index.md.

### Step 6: Summary Output

After creating all stubs, output a summary listing:
- How many candidates were initially identified
- Quantitative filter results — how many passed, how many rejected, and for each rejection: which filter(s) they failed
- Which tickers were added to watchlist/
- Brief rationale for each selected candidate
- Any cash drag warning status
