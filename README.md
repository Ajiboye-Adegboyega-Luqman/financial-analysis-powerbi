# Financial Analysis Dashboard — Power BI

An end-to-end financial reporting solution built in Power BI over a general ledger dataset,
covering the **Balance Sheet, Income Statement, Sales Profile, Profitability waterfall
(GP → EBITDA → Operating Profit → EBIT → Net Profit)** and **Financial Ratios** for
FY2018–FY2020.

The goal: turn a raw chart-of-accounts + GL extract into the five statements a finance
team actually reads, with drill-down by year, region, country and account class.

![Balance Sheet](images/01-balance-sheet.png)

---

## Table of contents

- [Key results](#key-results)
- [Report pages](#report-pages)
- [Data model](#data-model)
- [Core DAX measures](#core-dax-measures)
- [Findings](#findings)
- [Repository structure](#repository-structure)
- [How to run it](#how-to-run-it)
- [Skills demonstrated](#skills-demonstrated)
- [Full written report](#full-written-report)

---

## Key results

| Metric | FY2020 value | Read |
|---|---|---|
| Total Assets | £14.17M | Asset base tripled since 2018 (£4.46M) |
| Total Liabilities | £2.27M | Liabilities-to-assets ≈ 16% — conservative leverage |
| Equity | £12.0M | Equity-funded balance sheet |
| Total Revenue | £19.67M | Growth in every year of the series |
| Gross Profit | £13.45M | ≈ 68% gross margin |
| EBITDA | £6.20M | |
| Operating Profit | £4.30M | |
| EBIT | £4.48M | |
| Net Profit | £3.70M | ≈ 19% net margin |
| Current Ratio | 8.08 | Very liquid — arguably *over*-liquid |
| Quick Ratio | 6.80 | |
| Asset Turnover | 1.39 | Moderate asset efficiency |
| Gearing Ratio | 0.19 | Low financial risk |

---

## Report pages

### 1. Balance Sheet
`images/01-balance-sheet.png`

KPI cards for Total Assets / Total Liabilities / Equity, an asset breakdown bar chart,
a liabilities-and-equity column chart, a balance-sheet-value donut by subclass, a
hierarchical matrix (Class → Subclass → Account) across 2018–2020, and a treemap of
balance sheet value by account. Filtered by a `Date` slicer.

### 2. Income Statement
![Income Statement](images/02-income-statement.png)

Revenue, Gross Profit and Net Profit cards; a funnel of expense categories; an operating
expenses donut (utilities, advertising, office supplies, telephone, professional services,
travel, staff costs, commissions, entertainment); an Operating Profit card; and a
three-year income statement matrix broken into Interest & Tax, Non-operating and
Operating accounts. Sliceable by country.

### 3. Sales Profile
![Sales Profile](images/03-sales-profile.png)

Sales FTP, Gross Profit and Sales TTD cards, a year-on-year sales revenue column chart
(2018 → 2020), a subclass contribution donut (sales ≈ 89.67% of total), a gauge tracking
progress against a £56.67M target, and a territory map for regional performance.

### 4. Profitability — GP, EBITDA, OP, EBIT, NP
![Profitability](images/04-profitability.png)

Four KPI-with-trend visuals (EBITDA, Gross Profit, Operating Profit, EBIT) each showing
the 2018/2019/2020 trajectory, plus a Net Income by quarter combo chart — Q4 is
materially the strongest quarter.

### 5. Financial Ratios
![Financial Ratios](images/05-financial-ratios.png)

Current Ratio, Quick Ratio, Asset Turnover and Gearing Ratio cards; a stacked ratio bar
by year; a ratio matrix by year; a region slicer (Asia / Europe / North America); and a
ratio-contribution pie.

---

## Data model

A star-ish schema built around the GL fact table:

| Table | Role | Notes |
|---|---|---|
| `GL` | Fact | Transaction-level general ledger amounts |
| `Chart of Accounts` | Dimension | Account → Subclass → Class hierarchy |
| `Financial Structure` | Dimension | Statement mapping (Assets, Liabilities, Equity, P&L lines) |
| `Calendar` | Date dimension | Marked as a date table; drives Year / Quarter / YTD logic |
| `Territory` | Dimension | Region and country for geographic slicing |

The `Financial Structure` table is what makes a single GL feed both the balance sheet and
the income statement — each account is tagged to a statement, class and sign so that
one set of measures can render either statement.

---

## Core DAX measures

Representative patterns used in the model (adjust column names to match your copy):

```dax
Total Revenue =
CALCULATE (
    SUM ( GL[Amount] ),
    'Chart of Accounts'[Class] = "Revenue"
) * -1

Cost of Sales =
CALCULATE ( SUM ( GL[Amount] ), 'Chart of Accounts'[SubClass] = "Cost of Sales" )

Gross Profit = [Total Revenue] - [Cost of Sales]

Operating Expenses =
CALCULATE ( SUM ( GL[Amount] ), 'Chart of Accounts'[Class] = "Operating Expenses" )

Operating Profit = [Gross Profit] - [Operating Expenses]

EBITDA = [Operating Profit] + [Depreciation] + [Amortization]

EBIT = [Operating Profit] + [Non-Operating Income]

Net Profit = [EBIT] - [Interest Expense] - [Taxation]

Total Assets =
CALCULATE ( SUM ( GL[Amount] ), 'Financial Structure'[Class] = "Assets" )

Total Liabilities =
CALCULATE ( SUM ( GL[Amount] ), 'Financial Structure'[Class] = "Liabilities" ) * -1

Equity = [Total Assets] - [Total Liabilities]

Current Ratio  = DIVIDE ( [Current Assets], [Current Liabilities] )
Quick Ratio    = DIVIDE ( [Current Assets] - [Inventory], [Current Liabilities] )
Asset Turnover = DIVIDE ( [Total Revenue], [Total Assets] )
Gearing Ratio  = DIVIDE ( [Total Liabilities], [Equity] )

Sales TTD =
CALCULATE ( [Total Revenue], DATESYTD ( 'Calendar'[Date] ) )
```

---

## Findings

**Position.** Assets of £14.17M against £2.27M of liabilities — a ~16% liabilities-to-assets
ratio. The company is funded almost entirely by equity (£12.0M), of which retained
earnings are a large slice. Low insolvency risk, but also an unused lever: a moderate
amount of debt would raise return on equity without threatening solvency.

**Performance.** £19.67M revenue at a ~68% gross margin, converting to £3.70M net profit.
Administration is the heaviest operating expense line, which is where margin improvement
would come from first.

**Growth.** Revenue rises in each year 2018 → 2020, and every profitability measure
(GP, EBITDA, OP, EBIT) rises with it. Net income is concentrated in Q4 — seasonality worth
planning working capital around.

**Liquidity.** A current ratio of 8.08 and quick ratio of 6.80 are far above the
conventional 1.5–3.0 comfort band. Short-term obligations are covered many times over,
but that much idle cash and receivable balance is capital not earning a return.

**Efficiency.** Asset turnover of 1.39 is moderate — £1.39 of revenue per £1 of assets.
The clearest improvement path: deploy the excess liquidity into revenue-generating assets
rather than holding it.

---

## Repository structure

```
financial-analysis-powerbi/
├── README.md
├── .gitignore
├── dashboard/
│   └── financial-analysis-dashboard.pbix
├── images/
│   ├── 01-balance-sheet.png
│   ├── 02-income-statement.png
│   ├── 03-sales-profile.png
│   ├── 04-profitability.png
│   └── 05-financial-ratios.png
└── report/
    └── financial-analysis-report.pdf
```

---

## How to run it

1. Install [Power BI Desktop](https://powerbi.microsoft.com/desktop/) (free).
2. Clone the repo:
   ```bash
   git clone https://github.com/<your-username>/financial-analysis-powerbi.git
   ```
3. Open `dashboard/financial-analysis-dashboard.pbix`.
4. **Refresh** to reload the data. If the source path is broken, go to
   *Transform data → Data source settings → Change Source* and point it at your local copy.
5. Some pages use custom visuals from AppSource (territory map, KPI donut, smart slicer,
   annotated bar). Power BI Desktop will prompt to enable them on first open.
6. The Azure Maps visual on the Sales Profile page requires signing in to a Power BI account.

> **Note on file size:** the `.pbix` is ~7.4 MB. That is fine for a normal Git push, but if
> you iterate on it a lot, consider [Git LFS](https://git-lfs.com/) so history stays light:
> ```bash
> git lfs install
> git lfs track "*.pbix"
> git add .gitattributes
> ```

---

## Skills demonstrated

- Financial statement modelling from a raw general ledger
- Star schema design with a dedicated financial-structure dimension
- DAX: `CALCULATE`, `DIVIDE`, time intelligence (`DATESYTD`), ratio and margin measures
- Hierarchical matrix reporting (Class → Subclass → Account)
- KPI card design, trend visuals, gauges against target, geographic analysis
- Multi-page report navigation with cross-page filtering and drill-through
- Translating dashboard output into a written analytical report

---

## Full written report

`report/financial-analysis-report.pdf` contains the full academic analysis — balance sheet,
income statement, sales, profitability and ratio commentary, with a reference list.

**Author:** Ajiboye Luqman Adegboyega
**Module:** Business Intelligence using Power BI — University of Salford
**Date:** April 2026

---

## Tech stack

`Power BI Desktop` · `DAX` · `Power Query (M)` · `Star schema modelling`
