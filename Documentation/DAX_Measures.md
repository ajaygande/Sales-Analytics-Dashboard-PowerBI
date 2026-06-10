# DAX Measures Documentation

## Overview
All measures are stored in a single `_Measures` table for clean organization.
Total measures built: 47

---

## Base Sales Measures

| Measure | DAX Formula | Purpose |
|---|---|---|
| Total Revenue | `SUM(Sales[Sales Amount])` | Total gross revenue |
| Total Cost | `SUM(Sales[Total Product Cost])` | Total product cost |
| Total Profit | `[Total Revenue] - [Total Cost]` | Gross profit |
| Profit Margin % | `DIVIDE([Total Profit], [Total Revenue], 0)` | Profit as % of revenue |
| Total Orders | `DISTINCTCOUNT(Sales[SalesOrderLineKey])` | Unique order count |
| Total Quantity Sold | `SUM(Sales[Order Quantity])` | Total units sold |
| Avg Order Value | `DIVIDE([Total Revenue], [Total Orders], 0)` | Average revenue per order |
| Avg Unit Price | `AVERAGE(Sales[Unit Price])` | Average selling price |

---

## Customer Measures

| Measure | DAX Formula | Purpose |
|---|---|---|
| Total Customers | `DISTINCTCOUNT(Sales[CustomerKey])` | Unique customer count |
| Revenue per Customer | `DIVIDE([Total Revenue], [Total Customers], 0)` | Average revenue per customer |
| Customer Revenue Rank | `RANKX(ALL(Customer[Customer]), [Total Revenue],, DESC, DENSE)` | Ranks customers by revenue |
| Top Customer Name | `CALCULATE(SELECTEDVALUE(Customer[Customer]), TOPN(1, ALL(Customer[Customer]), [Total Revenue]))` | Name of top revenue customer |
| Revenue by City | `CALCULATE([Total Revenue], ALL(Customer[City]))` | Revenue filtered by city |
| Revenue by Country | `CALCULATE([Total Revenue], ALL(Customer[Country-Region]))` | Revenue filtered by country |
| % Revenue by Country | `DIVIDE([Total Revenue], CALCULATE([Total Revenue], ALL(Customer[Country-Region])), 0)` | Country share of total revenue |

---

## Product Measures

| Measure | DAX Formula | Purpose |
|---|---|---|
| Total Products Sold | `DISTINCTCOUNT(Sales[ProductKey])` | Unique products sold |
| Total Standard Cost | `SUMX(Sales, Sales[Order Quantity] * RELATED(Product[Standard Cost]))` | Total standard cost |
| Avg List Price | `AVERAGEX(Sales, RELATED(Product[List Price]))` | Average list price of sold products |
| Product Revenue Rank | `RANKX(ALL(Product[Product]), [Total Revenue],, DESC, DENSE)` | Ranks products by revenue |
| Category Revenue Rank | `RANKX(ALL(Product[Category]), [Total Revenue],, DESC, DENSE)` | Ranks categories by revenue |
| Subcategory Revenue Rank | `RANKX(ALL(Product[Subcategory]), [Total Revenue],, DESC, DENSE)` | Ranks subcategories by revenue |
| Top 10 Products Flag | `IF(RANKX(ALL(Product[Product]), [Total Revenue],, DESC, DENSE) <= 10, "Top 10", "Others")` | Flags top 10 products |
| % of Total Revenue | `DIVIDE([Total Revenue], CALCULATE([Total Revenue], ALL(Sales)), 0)` | Product share of total revenue |
| % of Category Revenue | `DIVIDE([Total Revenue], CALCULATE([Total Revenue], ALL(Product[Product])), 0)` | Product share of category revenue |
| Revenue by Category | `CALCULATE([Total Revenue], ALL(Product[Category]))` | Revenue filtered by category |
| Revenue by Subcategory | `CALCULATE([Total Revenue], ALL(Product[Subcategory]))` | Revenue filtered by subcategory |
| Product Profitability Tier | `SWITCH(TRUE(), [Profit Margin %] >= 0.40, "High", [Profit Margin %] >= 0.20, "Medium", [Profit Margin %] >= 0, "Low", "Loss Making")` | Classifies products by margin tier |

---

## Time Intelligence Measures

| Measure | DAX Formula | Purpose |
|---|---|---|
| Sales Revenue MTD | `TOTALMTD([Total Revenue], 'Date'[Date])` | Month to date revenue |
| Sales Revenue QTD | `TOTALQTD([Total Revenue], 'Date'[Date])` | Quarter to date revenue |
| Sales Revenue YTD | `TOTALYTD([Total Revenue], 'Date'[Date])` | Year to date revenue |
| Sales Revenue PYTD | `CALCULATE([Total Revenue], SAMEPERIODLASTYEAR('Date'[Date]))` | Prior year to date revenue |
| Sales Revenue Previous Month | `CALCULATE([Total Revenue], DATEADD('Date'[Date], -1, MONTH))` | Previous month revenue |
| Sales Revenue Previous Quarter | `CALCULATE([Total Revenue], DATEADD('Date'[Date], -1, QUARTER))` | Previous quarter revenue |
| Sales YoY Growth | `[Total Revenue] - [Sales Revenue PYTD]` | Year over year revenue change |
| Sales YoY Growth % | `DIVIDE([Sales YoY Growth], [Sales Revenue PYTD], BLANK())` | Year over year growth percentage |
| Sales MoM Growth % | `DIVIDE([Total Revenue] - [Sales Revenue Previous Month], [Sales Revenue Previous Month], 0)` | Month over month growth percentage |
| Sales by Fiscal Year | `CALCULATE([Total Revenue], ALL('Date'[Fiscal Year]))` | Revenue by fiscal year |
| Sales by Fiscal Quarter | `CALCULATE([Total Revenue], ALL('Date'[Fiscal Quarter]))` | Revenue by fiscal quarter |
| Sales by Month | `CALCULATE([Total Revenue], ALL('Date'[Month]))` | Revenue by month |
| Sales Rolling 3M | `CALCULATE([Total Revenue], DATESINPERIOD('Date'[Date], LASTDATE('Date'[Date]), -3, MONTH))` | Rolling 3 month revenue |
| Sales Rolling 6M | `CALCULATE([Total Revenue], DATESINPERIOD('Date'[Date], LASTDATE('Date'[Date]), -6, MONTH))` | Rolling 6 month revenue |

---

## KPI Target Measures

| Measure | DAX Formula | Purpose |
|---|---|---|
| Revenue Target | `[Sales Revenue PYTD] * 1.10` | 10% growth target vs prior year |
| Revenue vs Target | `[Total Revenue] - [Revenue Target]` | Variance from target |
| Revenue vs Target % | `DIVIDE([Revenue vs Target], [Revenue Target], 0)` | Variance as percentage |
| Revenue Target Status | `IF([Revenue vs Target] >= 0, "On Track", "Below Target")` | Target status label |
| Revenue Color Flag | `IF([Sales YoY Growth %] >= 0, "Green", "Red")` | Color indicator for growth |
| Avg YoY Growth % | `AVERAGEX(FILTER(VALUES('Date'[Fiscal Year]), [Sales YoY Growth %] <> BLANK() && [Total Revenue] > 0), [Sales YoY Growth %])` | Average YoY growth excluding blank years |

---

## Dynamic Title Measures

| Measure | DAX Formula | Purpose |
|---|---|---|
| Selected Year Title | `"Sales Performance — " & SELECTEDVALUE('Date'[Fiscal Year], "All Years")` | Dynamic page title by year |
| Selected Month Title | `"Month: " & SELECTEDVALUE('Date'[Month], "All Months")` | Dynamic title by month |
| Selected Category Title | `"Category: " & SELECTEDVALUE(Product[Category], "All Categories")` | Dynamic title by category |
| Selected Country Title | `"Region: " & SELECTEDVALUE(Customer[Country-Region], "All Regions")` | Dynamic title by country |

---

## Advanced Measures

| Measure | DAX Formula | Purpose |
|---|---|---|
| Profitability Tier | `SWITCH(TRUE(), [Profit Margin %] >= 0.40, "High", [Profit Margin %] >= 0.20, "Medium", [Profit Margin %] >= 0, "Low", "Loss Making")` | Classifies overall profitability |

---

## Key Notes

- All measures stored in `_Measures` table
- Date table marked as Date Table — required for time intelligence
- Active date relationship uses `OrderDateKey`
- `ShipDateKey` and `DueDateKey` relationships are inactive
- Use `USERELATIONSHIP()` to activate inactive date relationships in DAX
- `BLANK()` used as fallback in YoY measures to avoid misleading 0 values
- FY2021 exists in Date table but has no sales data — excluded from averages
