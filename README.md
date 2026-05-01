# 📈 Stock Market Analysis — Power BI Dashboard
**Data Visualisation Tools & Techniques | CIE-3 Project**
School of Computer Science and Engineering

---

## 📌 Problem Statement

This project analyses historical stock market data to uncover price trends, volatility patterns, moving average crossovers, and trading signals across multiple companies. The core analytical questions addressed are:

- How have stock closing prices trended over time across companies?
- What do 20-day and 50-day Simple Moving Averages reveal about price momentum?
- What is the daily volatility profile of each stock, and how does it vary by month?
- When does technical analysis suggest a Buy or Sell signal based on SMA crossovers?
- How does traded volume compare across companies and time periods?

**Objectives:**
1. Compute technical indicators (SMA_20, SMA_50, Daily Volatility, Price Change %, Buy/Sell Signal) using explicit DAX measures.
2. Build an interactive, single-page Power BI dashboard with 11 visuals covering trend, signal, volume, and volatility analysis.
3. Enable per-company drill-down via a Company slicer.

---

## 🗂️ Dataset

| Attribute | Detail |
|-----------|--------|
| **Table Name** | `StockData` |
| **Key Columns** | `Date`, `Company`, `Open`, `High`, `Low`, `Close`, `Volume` |
| **Granularity** | Daily OHLCV trading records |
| **Date Hierarchy** | Year → Quarter → Month → Day (auto-generated) |

---

## 🔧 Data Sourcing, Cleaning & Modelling

All transformations were applied in **Power Query Editor** prior to loading.

### Cleaning Steps
- **Type casting** — `Date` cast to `Date`; `Close`, `Open`, `High`, `Low` to `Decimal Number`; `Volume` to `Whole Number`.
- **Null handling** — Rows with missing `Close` or `Date` values removed or forward-filled.
- **Date hierarchy** — Power BI auto-generated Year → Quarter → Month → Day drill-down on the `Date` column.
- **Duplicate check** — Verified no duplicate `(Date, Company)` pairs exist.

### Data Model
- **Single fact table** architecture — `StockData` holds all raw price and volume data.
- `TypeDetectionEnabled` and `RelationshipImportEnabled` were active during import (confirmed from report `Settings`).
- Report created on **Power BI Cloud** (release `2026.04`, Desktop version `2.152.856.0`).

---

## 📐 DAX Measures

Six explicit DAX measures are defined, all operating on the `StockData` table.

### 1. `Price_Change_%`
Calculates the percentage change between the chronologically first and last closing price within the current filter context.

```dax
Price_Change_% =
VAR FirstClose = 
    CALCULATE(MIN(StockData[Close]), 
    FILTER(ALL(StockData), 
    StockData[Company] = MAX(StockData[Company])))
VAR LastClose  = 
    CALCULATE(MAX(StockData[Close]), 
    FILTER(ALL(StockData), 
    StockData[Company] = MAX(StockData[Company])))
RETURN DIVIDE(LastClose - FirstClose, FirstClose) * 100
```

---

### 2. `SMA_20` — 20-Day Simple Moving Average
Average closing price over the 20 most recent trading days, scoped per company.

```dax
SMA_20 = AVERAGEX(FILTER(
        ALL(StockData),
        StockData[Company] = MAX(StockData[Company]) &&
        StockData[Date] <= MAX(StockData[Date]) &&
        StockData[Date] >= MAX(StockData[Date]) - 20
    ),
    StockData[Close])
```

---

### 3. `SMA_50` — 50-Day Simple Moving Average
Same pattern as `SMA_20`, extended to 50 trading days. Used alongside `SMA_20` to detect:
- **Golden Cross** — SMA_20 crosses above SMA_50 → bullish signal
- **Death Cross** — SMA_20 crosses below SMA_50 → bearish signal

---

### 4. `Buy_Sell_Signal`
Conditional KPI returning `"BUY"`, `"SELL"`, or `"HOLD"` based on SMA crossover logic.

```dax
Buy_Sell_Signal = IF([SMA_20] > [SMA_50], "BUY", "SELL")
```

---

### 5. `Total_Volume`
```dax
Total_Volume = SUM(StockData[Volume])
```

---

### 6. `Daily_Volatility`
Measures intraday price spread as a proxy for daily risk.

```dax
Daily_Volatility = MAX(StockData[High]) - MAX(StockData[Low])
```

---

## 🖥️ Dashboard Layout & Visuals

The report is a **single page ("Page 1")** with **11 visuals**, arranged to flow from KPI summary → trend analysis → comparative breakdown.

### KPI Cards
| # | Visual | Measure | Purpose |
|---|--------|---------|---------|
| 1 | Card | `Price_Change_%` | Overall price performance in selected context |
| 2 | Card | `Total_Volume` | Total shares traded |
| 6 | Card (labelled **SIGNAL**) | `Buy_Sell_Signal` | Live Buy / Sell / Hold recommendation |

### Slicer
| # | Visual | Field | Purpose |
|---|--------|-------|---------|
| 3 | Slicer | `Company` | Filters all visuals to selected company/companies |

### Trend & Moving Average Visuals
| # | Visual | Title | Key Fields |
|---|--------|-------|------------|
| 4 | Line Chart | Average Close By Year | Date Hierarchy (drillable), `Sum(Close)`, `Company` |
| 5 | Line + Column Combo Chart | SMA_20, SMA_50 & Average Volume By Year | Date Hierarchy, `SMA_20`, `SMA_50`, `Sum(Volume)` |
| 9 | Clustered Bar Chart | SMA_20 & SMA_50 By Month | Month, Day, `SMA_20`, `SMA_50` |

### Volatility & Volume Distribution
| # | Visual | Title | Key Fields |
|---|--------|-------|------------|
| 10 | Treemap | Total Volume of Company By Month | `Company`, `Total_Volume`, Month |
| 11 | Pie Chart | Daily Volatility By Month | `Daily_Volatility`, Month |

### Detail / Reference Tables
| # | Visual | Key Fields |
|---|--------|------------|
| 7 | Pivot Table | Date Hierarchy, `SMA_20`, `SMA_50`, `Company` |
| 8 | Multi-Row Card | Year, Month, `Avg(High)`, `Avg(Low)` |

### Interactivity
- **Company Slicer** — cross-filters all 11 visuals simultaneously.
- **Date hierarchy drill-down** — Year → Quarter → Month → Day on Line and Bar charts.
- **Cross-filtering** — clicking any visual's data point filters the rest of the page.

---

## 📊 Key Insights

1. **Price Trend** — The Average Close by Year line chart reveals long-term trajectories per company, making bull and bear phases easily identifiable.
2. **SMA Crossover Signal** — When `SMA_20` rises above `SMA_50`, the SIGNAL card flips to `BUY`, flagging a potential Golden Cross entry point.
3. **Volatility Seasonality** — The Pie Chart of Daily Volatility by Month shows which months historically carry higher intraday risk, aiding risk-aware position sizing.
4. **Volume Distribution** — The Treemap gives an immediate visual of which companies dominate trading volume across months.
5. **High/Low Range** — The Multi-Row Card's monthly average High and Low values define the price channel, useful for support/resistance analysis.

---

## 🛠️ Tools & Technologies

| Tool | Purpose |
|------|---------|
| **Power BI Cloud** | Report authoring (release 2026.04) |
| **Power BI Desktop** | Local editing (version 2.152.856.0) |
| **Power Query (M)** | Data cleaning, type casting, date hierarchy |
| **DAX** | 6 custom measures — SMA_20, SMA_50, Buy_Sell_Signal, Daily_Volatility, Total_Volume, Price_Change_% |

---

## 📁 Project Structure

```
📦 StockMarket-PowerBI/
├── 📊 CIE3.pbix                    ← Main Power BI report file
├── 📄 README.md                    ← This file
├── 📁 data/
│   └── stock_data.csv              ← Raw OHLCV dataset
└── 📁 report/
    └── CIE3_Report.pdf             ← Submitted PDF with visual explanations
```

---

## 🚀 How to Open

1. Download `CIE3.pbix` from this repository.
2. Open in **Power BI Desktop** (v2.152.x or later) or Power BI Cloud.
3. If prompted, reconnect the data source to your local CSV file.
4. Click **Refresh** in the Home ribbon to reload data.
5. Use the **Company slicer** to filter by individual stocks.
6. Use the **drill-down arrows** on Line/Bar charts to navigate Year → Quarter → Month → Day.

---