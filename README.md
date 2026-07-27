# 📈 BSE Indices Analytics Dashboard — Tableau

> **3-dashboard Tableau workbook analyzing BSE equity index performance: Executive KPIs · Risk-Return Matrix · Sector Comparison**
> Tools: `Tableau` · `LOD Expressions` · `Table Calculations` · `BSE Historical Data`

---

## 📌 Project Overview

Investors and market analysts tracking Indian equity markets need a fast, visual way to compare index performance, assess risk-adjusted returns, and identify which BSE indices have delivered the most consistent growth relative to their volatility. This project builds a 3-dashboard Tableau analytics platform on BSE Daily Historical Index Data — covering price performance, drawdown risk, volatility, and cross-index comparison.

**Live Dashboard (Tableau Public):** [Add your Tableau Public link here]

---

## 📦 Dataset Profile

| Attribute | Value |
|-----------|-------|
| Source | BSE India — Daily Historical Index Data |
| Table | HistoricalData (up to 10,000 rows) + QueryIndices (21 indices) |
| Date Dimension | Daily granularity (`tdate`) |
| Price Fields | Open, High, Low, Close (`I_open`, `I_high`, `I_low`, `I_close`) |
| Market Metrics | P/E Ratio (`I_pe`), P/B Ratio (`I_pb`), Dividend Yield (`I_yl`) |
| Volume & Turnover | `Volume`, `Turnover`, `TOTAL_SHARES_TRADED` |
| Daily Change | Absolute (`Chg`) and Percentage (`ChgPer`) |
| Indices Covered | 21 BSE Indices (Sensex, BSE 100, BSE 200, BSE 500, sector indices, etc.) |

---

## 🏗️ Dashboard Architecture

### Dashboard 1 — Executive Overview

**Purpose**: Single-screen command center for monitoring current index health across 4 critical dimensions.

**KPI Sheets embedded**:
| Sheet | Metric | Calculation Method |
|-------|--------|-------------------|
| KPI Latest Close | Most recent closing price per index | LOD Fixed expression: `{ FIXED [I_name] : MAX([tdate]) }` then SUM of close on that date |
| KPI Daily Change | Today's price change and % change | Lookup-based daily return: `(Close_today - Close_yesterday) / Close_yesterday` |
| KPI Max Drawdown | Worst peak-to-trough decline | Running max then `(Close - Running_Peak) / Running_Peak` — minimum fixed per index |
| KPI Volatility | 30-day annualized realized volatility | `WINDOW_STDEV(Daily_Return, -29, 0) × √252` |

**Trend Timeline** visualization showing price trajectory over the full history — filtered to show only latest active indices.

---

### Dashboard 2 — Risk-Return Matrix

**Purpose**: Scatter plot positioning each BSE index on a risk (volatility) vs. return (annualized gain) quadrant — the standard framework for investment decision-making.

**Key calculated fields**:
```
Scatter Annualized Return =
(Close_latest - Close_first) / Close_first
[using FIXED LOD to pin base price per index]

Scatter Volatility =
{ FIXED [I_name] : STDEV([ChgPer] / 100) } × √252
[cross-sectional volatility, annualized with √252 trading days]
```

**Reading the matrix**:
- **Top-left quadrant**: High return, low volatility — optimal risk-adjusted performance
- **Bottom-right quadrant**: High volatility, low return — poorest risk-adjusted performance
- Each dot = one BSE index, labeled by `I_name`

---

### Dashboard 3 — Sector Comparison (Normalized Returns Base 100)

**Purpose**: Apples-to-apples comparison of indices that started at different price levels, by rebasing all to 100 at the start date.

**Key calculation**:
```
Normalized Return Base 100 =
SUM([I_close]) / 
ATTR({ FIXED [I_name] : SUM(IF [tdate] = { FIXED [I_name] : MIN([tdate]) } THEN [I_close] END) }) × 100
```

This converts every index to a "₹100 invested at start → worth X today" format — making performance directly comparable regardless of index level differences (Sensex at 80,000 vs a sector index at 5,000).

---

## ⚙️ Technical Highlights

### LOD (Level of Detail) Expressions Used

LOD expressions allow calculations at a granularity different from the view — a powerful Tableau-specific feature used extensively here:

```tableau
// Latest Close — fix to index level, get value on most recent date
{ FIXED [I_name] : SUM(IF [tdate] = { FIXED [I_name] : MAX([tdate]) } THEN [I_close] END) }

// Historical Peak — max close price ever recorded per index
{ FIXED [I_name] : MAX([I_close]) }

// Max Drawdown — worst decline ever from peak, per index
{ FIXED [I_name] : MIN(IF [I_close] > 0 
  THEN ([I_close] - {FIXED [I_name]: MAX([I_close])}) / {FIXED [I_name]: MAX([I_close])} 
  END) }

// Cross-Sectional Volatility — annualized std dev of daily returns, per index
{ FIXED [I_name] : STDEV([ChgPer] / 100) } * SQRT(252)
```

### Table Calculations Used

```tableau
// Daily Return — percentage change from previous trading day
Daily Return (%) = 
  (ZN(SUM([I_close])) - LOOKUP(ZN(SUM([I_close])), -1)) 
  / LOOKUP(ZN(SUM([I_close])), -1)

// Running Peak — highest close reached up to current date
Running Peak = RUNNING_MAX(SUM([I_close]))

// Drawdown — current distance from running peak
Drawdown % = (SUM([I_close]) - RUNNING_MAX(SUM([I_close]))) / RUNNING_MAX(SUM([I_close]))

// 30D Realized Volatility — rolling 30-day window, annualized
Annualized Volatility 30D = WINDOW_STDEV([Daily_Return], -29, 0) * SQRT(252)

// Normalized Return Base 100 — for cross-index comparison
Chart2_Normalized_Base_100 = 
  (SUM([I_close]) / LOOKUP(SUM([I_close]), FIRST())) * 100
```

### Financial Year Handling (Indian Market Convention)

Indian fiscal year runs April–March, not January–December. Custom field:
```tableau
Financial Year = 
  IF MONTH([tdate]) >= 4 THEN YEAR([tdate]) 
  ELSE YEAR([tdate]) - 1 END
```

---

## 📊 Key Insights This Dashboard Surfaces

| Question | Dashboard | How to Read |
|----------|-----------|-------------|
| Which BSE index is at its highest/lowest right now? | Executive Overview | KPI Latest Close card, sorted by value |
| Which index had the worst crash? | Executive Overview | KPI Max Drawdown — most negative % wins/loses |
| Which index gives the best return per unit of risk? | Risk-Return Matrix | Furthest top-left on the scatter plot |
| Which sector recovered fastest post-correction? | Sector Comparison | Steepest slope in normalized returns chart |
| How volatile is a specific index right now? | Executive Overview | KPI Volatility — 30D annualized |

---

## 🛠️ Tech Stack

| Layer | Technology |
|-------|-----------|
| Data Source | BSE India Historical Index Data (Excel) |
| Visualization Tool | Tableau Desktop / Tableau Public |
| Advanced Calculations | LOD Expressions, Table Calculations, Window Functions |
| Financial Metrics | Drawdown, Volatility (realized), Normalized Returns, Risk-Return |

---

## 📁 Repository Structure

```
bse-tableau-dashboard/
│
├── BSE_Analysis.twbx           # Tableau packaged workbook (full project)
├── data/
│   └── BSEIndicesDailyHistoricalData.xlsx  # Source dataset
├── screenshots/
│   ├── executive_overview.png
│   ├── risk_return_matrix.png
│   └── sector_comparison.png
└── README.md
```

---

## 🎯 Skills Demonstrated

`Tableau Desktop` · `LOD Expressions (FIXED)` · `Table Calculations` · `Window Functions` · `Financial Analytics` · `Drawdown Analysis` · `Realized Volatility` · `Risk-Return Framework` · `BSE Market Data` · `Equity Index Analytics` · `Normalized Returns`

---

## 🔗 Links

- **Tableau Public**: [(https://public.tableau.com/views/BSEAnalysis_17850002261240/SectorComparison?:language=en-US&:sid=&:redirect=auth&:display_count=n&:origin=viz_share_link)]
- **Portfolio**: [github.com/sahiln15]
- **LinkedIn**: [linkedin.com/in/sahilnavale]
