# DataXLAbs_Intern_Task-3
Dashboard Creation for business stakeholders 
# Sales & Financial Performance Dashboard — Power BI

An interactive Power BI dashboard built for business stakeholders to track sales, profit, and growth performance across regions, categories, and time. Built on the Kaggle Superstore dataset as a hands-on exercise in KPI selection, data modeling, DAX, and dashboard UX.

---

## Objective

Design an interactive business dashboard that helps stakeholders make data-driven decisions by surfacing the right KPIs, enabling self-service filtering, and presenting time-series trends and regional/category breakdowns clearly.

## Outcome

Hands-on experience with the full BI workflow: cleaning raw transactional data, modeling it for analysis (calendar table, relationships), writing DAX measures for KPIs and YoY growth, and designing a multi-page, navigable dashboard with consistent visual language.

---

## Tools Used

- **Power BI Desktop** — data modeling, DAX, visualization
- **Power Query** — data cleaning and transformation
- **DAX (Data Analysis Expressions)** — KPI and time-intelligence measures

---

## Dataset

**Source:** [Superstore Dataset (Final) — Kaggle](https://www.kaggle.com/datasets/vivek468/superstore-dataset-final)

A retail transactions dataset (~9,994 rows) covering 4 years of order history.

| Column | Description |
|---|---|
| Order ID | Unique order identifier |
| Order Date / Ship Date | Transaction and fulfillment dates |
| Customer Name, Segment | Customer info (Consumer / Corporate / Home Office) |
| Region, State, City | Geographic dimensions |
| Category, Sub-Category, Product Name | Product hierarchy |
| Sales, Quantity, Discount, Profit | Core financial metrics |

---

## Project Files

```
├── Sample - Superstore.csv          # Raw dataset (from Kaggle)
├── Poer BI Dashboard 4 pages pdf    # PDF
├── README.md                        # This file
```

---

## 1. Data Preparation (Power Query)

1. Imported `Sample - Superstore.csv` via **Get Data → Text/CSV**.
2. Set column data types: `Sales`, `Profit`, `Discount` → Decimal Number; `Quantity` → Whole Number; remaining identifier/text fields → Text.
3. **Date format fix:** the source file is formatted MM/DD/YYYY (US convention). On a non-US system locale, Power BI's default type conversion misreads these and throws errors on any date where the "month" value exceeds 12. Fixed by selecting `Order Date` and `Ship Date` together → **Right-click → Change Type → Using Locale → Date, English (United States)**. Both columns must be reset to Text and reconverted together in the same step to avoid inconsistent per-column parsing.
4. Removed unused identifier columns (`Row ID`),(`Postal Code`)  where not needed for analysis.
---

## 2. Data Model

### Tables created

| Table | Purpose | How it was created |
|---|---|---|
| `Superstore` | Main fact table (imported transactional data) | Power Query import |
| `Date` | Calendar/date dimension for time intelligence | DAX calculated table |
| `_Measures` | Container table to hold all DAX measures separately from data tables | DAX calculated table |

### `Date` table

```DAX
Date = CALENDAR(MIN(Superstore[Order Date]), MAX(Superstore[Order Date]))
```

Calculated columns added on `Date`:

```DAX
Year      = YEAR('Date'[Date])
Month     = FORMAT('Date'[Date], "MMM")
MonthNo   = MONTH('Date'[Date])
Quarter   = "Q" & QUARTER('Date'[Date])
YearMonth = FORMAT('Date'[Date], "MMM YYYY")
```

### `_Measures` table

```DAX
_Measures = {BLANK()}
```
(A measures table needs at least one column; `{}` alone is invalid since an empty table has zero columns. The resulting `Value` column is hidden in report view and never used directly.)

### Relationships

| From | To | Cardinality | Direction |
|---|---|---|---|
| `Date[Date]` | `Superstore[Order Date]` | One-to-many (1:*) | Single, Date → Orders |

Built in **Model view** by dragging `Date[Date]` onto `Superstore[Order Date]`.

---

## 3. DAX Measures

All measures live in the `_Measures` table.

```DAX
Total Sales        = SUM(Superstore[Sales])
Total Profit       = SUM(Superstore[Profit])
Total Orders       = DISTINCTCOUNT(Orders[Order ID])
Total Quantity     = SUM(Superstore[Quantity])
Profit Margin %    = DIVIDE([Total Profit], [Total Sales], 0)
Avg Order Value    = DIVIDE([Total Sales], [Total Orders], 0)
Avg Discount %     = AVERAGE(Superstore[Discount])

Sales LY            = CALCULATE([Total Sales], SAMEPERIODLASTYEAR('Date'[Date]))
Profit LY           = CALCULATE([Total Profit], SAMEPERIODLASTYEAR('Date'[Date]))
Sales YoY %          = DIVIDE([Total Sales] - [Sales LY], [Sales LY], 0)
Profit YoY %         = DIVIDE([Total Profit] - [Profit LY], [Profit LY], 0)

Loss Orders         = CALCULATE([Total Orders], Superstore[Profit] < 0)
```

---

## 4. Dashboard Pages

### Page 1 — Overview
- KPI cards: Total Sales, Total Profit, Profit Margin %, Total Orders, Sales YoY %
- Line chart: Monthly Sales & Profit trend (`Date[YearMonth]` on X, `Total Sales` + `Total Profit` as series)
- Bar chart: Sales by Category
- Donut chart: Sales by Segment
- Slicers: Year

### Page 2 — Sales Analysis
- Treemap: Sales by Sub-Category
- Top 10 Products by Sales (Top N filter on `Product Name`, ranked by `Total Sales`)
- Multi-line chart: Sales trend by Category
- Scatter chart: Discount vs. Sales
  
### Page 3 — Profit Analysis
- Bar chart: Profit by Sub-Category, conditional formatting (red for negative)
- Matrix: Profit Margin % by Region × Category, color-scale conditional formatting
- Card: Loss-Making Orders count
- Bottom 10 Products by Profit (Top N filter, "Bottom" direction, ranked by `Total Profit`, sorted ascending)
- Line chart: Profit Margin % trend over time

### Page 4 — Regional View
- Filled map: Sales by State
- Clustered column chart: Sales & Profit by Region
- Customer-level summary table: Customer Name, Total Sales, Total Profit, Total Orders
- Top region callouts by Sales and by Profit Margin

---

## 5. Color Theme

| Role | Hex |
|---|---|
| Primary (headers/dark elements) | `#0B1F3A` |
| Secondary | `#1E293B` |
| Accent / positive | `#0D9488` |
| Negative / alerts | `#DC2626` |
| Background | `#F8FAFC` |
| Text | `#1E293B` |

Applied globally via **View → Themes → Customize current theme**, used consistently across every visual and page.

---

## Troubleshooting Log

Issues encountered during the build and their fixes, kept here for reference:

| Issue | Cause | Fix |
|---|---|---|
| Dates showing MM/DD/YYYY errors instead of converting cleanly | System locale (DD/MM/YYYY) conflicting with source file format (MM/DD/YYYY) | Converted `Order Date`/`Ship Date` together using **Change Type → Using Locale → English (United States)** |
| `Order Date` and `Ship Date` erroring on different rows | Inconsistent type-conversion steps applied separately to each column | Reset both columns to Text, then reapplied the locale conversion to both simultaneously |
| `An Evaluate statement cannot return a table without columns` | `_Measures = {}` creates a zero-column table, which DAX rejects | Used `_Measures = {BLANK()}` instead, then hid the resulting `Value` column |

---

## Key Business Insights
Furniture is underperforming: 2% profit margin vs. 17% for Technology/Office Supplies; even negative (-2%) in Central.
Tables sub-category loses money outright — the only one below zero on total profit.
Top customer by sales is unprofitable: Sean Miller ($25K sales, -$1,980 profit). Sales ≠ value.
~20% of orders lose money (1,000 of 5,009).
West leads on both sales and profit margin; South lags on both.
Profit margin flat at ~10-14% for 4 years — no improvement despite sales growth.
Consumer segment = 51% of sales, heavy reliance vs. Corporate (31%) and Home Office (19%).
