---
description: Search for new investment candidates and add stubs to watchlist/
agent: build
---

# Stock Screener

Search for new investment candidates that align with the portfolio's objective and risk level. Produce lightweight stub files in `watchlist/` for each candidate found. This command does NOT perform deep analysis — it identifies candidates for `/analyze` to evaluate.

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

Select the top 3-5 candidates after filtering out tickers already in `watchlist/_index.md` or held in PORTFOLIO.md. Rank by fit with the objective, diversification benefit (sector and thematic), and valuation attractiveness.

### Step 2: Create Watchlist Stubs

For each selected candidate, create a new file `watchlist/{TICKER}.md` using the template loaded above. For stub entries:
- Set **Watchlist Status** to Active
- Set **Added** to today's date
- Set **Last Analyzed** to Pending
- Fill in **Screening Reason**, **Sector**, **Industry**, **Market Cap**, **Price**
- Set **Thesis** to "Pending deep analysis. See Screening Reason above."
- Leave Key Metrics, Catalysts, Risks, Competitive Position, and Verdict sections as placeholders

### Step 3: Update Watchlist Index

Append a new row to `watchlist/_index.md` for each candidate added:
```
| {TICKER} | {Sector} | {today} | Pending | Pending | Pending | Pending |
```

Update the **Active Items** count and **Last Updated** date at the top of _index.md.

### Step 4: Summary Output

After creating all stubs, output a summary listing:
- How many candidates were screened
- How many passed filters
- Which tickers were added to watchlist/
- Brief rationale for each
