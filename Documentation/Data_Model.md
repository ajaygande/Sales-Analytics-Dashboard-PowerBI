# Data Model Documentation

## Overview
The data model follows a **Star Schema** design with 2 fact tables 
and 5 dimension tables. All measures are stored in a separate 
`_Measures` table for clean organization.

---
## Schema Diagram

```
DIMENSION TABLES                         FACT TABLE              DIMENSION TABLE
─────────────────────────────────────────────────────────────────────────────────

[Customer]       ──── CustomerKey ──────▶
[Product]        ──── ProductKey ────────▶
[Date]           ──── OrderDateKey ──────▶ [Sales] ◀──── SalesTerritoryKey ──── [SalesTerritory]
[Reseller]       ──── ResellerKey ───────▶
[SalesOrder]     ──── SalesOrderKey ─────▶

─────────────────────────────────────────────────────────────────────────────────
INACTIVE RELATIONSHIPS (activate using USERELATIONSHIP in DAX)

[Date] ──── ShipDateKey ──▶ [Sales]
[Date] ──── DueDateKey ───▶ [Sales]

─────────────────────────────────────────────────────────────────────────────────
CARDINALITY    One (1) to Many (*)
FILTER         Single direction on all relationships
```
    
## Tables

### Fact Tables

#### Sales
Primary transaction table containing all sales records.

| Column | Type | Description |
|---|---|---|
| SalesOrderLineKey | Integer | Unique order line identifier |
| OrderDateKey | Integer | FK to Date table (Active) |
| ShipDateKey | Integer | FK to Date table (Inactive) |
| DueDateKey | Integer | FK to Date table (Inactive) |
| CustomerKey | Integer | FK to Customer table |
| ResellerKey | Integer | FK to Reseller table |
| ProductKey | Integer | FK to Product table |
| SalesTerritoryKey | Integer | FK to SalesTerritory table |
| Sales Amount | Decimal | Gross sales revenue |
| Total Product Cost | Decimal | Total cost of product |
| Order Quantity | Integer | Units ordered |
| Unit Price | Decimal | Price per unit |
| Unit Price Discount Pct | Decimal | Discount percentage (all zeros in dataset) |

#### SalesOrder
Order level details linked to Sales fact table.

| Column | Type | Description |
|---|---|---|
| SalesOrderLineKey | Integer | FK to Sales table |

---

### Dimension Tables

#### Customer
Customer demographic and location information.

| Column | Type | Description |
|---|---|---|
| CustomerKey | Integer | Primary key |
| Customer | Text | Full customer name |
| Customer ID | Text | Unique customer identifier |
| City | Text | Customer city |
| State-Province | Text | Customer state or province |
| Country-Region | Text | Customer country |
| Postal Code | Text | Customer postal code |

#### Product
Product details, pricing, and categorization.

| Column | Type | Description |
|---|---|---|
| ProductKey | Integer | Primary key |
| Product | Text | Full product name |
| SKU | Text | Stock keeping unit |
| Model | Text | Product model name |
| Category | Text | Product category |
| Subcategory | Text | Product subcategory |
| Color | Text | Product color |
| Standard Cost | Decimal | Standard manufacturing cost |
| List Price | Decimal | Recommended retail price |

#### Date
Calendar and fiscal period information.

| Column | Type | Description |
|---|---|---|
| DateKey | Integer | Primary key |
| Date | Date | Full date value |
| Full Date | Date | Full date alternate format |
| Month | Text | Month name |
| MonthKey | Integer | Month sort order |
| Fiscal Quarter | Text | Fiscal quarter label |
| Fiscal Year | Text | Fiscal year label |

#### Reseller
Reseller and B2B channel information.

| Column | Type | Description |
|---|---|---|
| ResellerKey | Integer | Primary key |

#### SalesTerritory
Geographic sales territory information.

| Column | Type | Description |
|---|---|---|
| SalesTerritoryKey | Integer | Primary key |

---

## Relationships

| From Table | From Column | To Table | To Column | Cardinality | Status |
|---|---|---|---|---|---|
| Date | DateKey | Sales | OrderDateKey | 1 → * | Active |
| Date | DateKey | Sales | ShipDateKey | 1 → * | Inactive |
| Date | DateKey | Sales | DueDateKey | 1 → * | Inactive |
| Customer | CustomerKey | Sales | CustomerKey | 1 → * | Active |
| Product | ProductKey | Sales | ProductKey | 1 → * | Active |
| SalesTerritory | SalesTerritoryKey | Sales | SalesTerritoryKey | 1 → * | Active |
| Reseller | ResellerKey | Sales | ResellerKey | 1 → * | Active |
| SalesOrder | SalesOrderLineKey | Sales | SalesOrderLineKey | 1 → * | Active |

---

## Key Design Decisions

### Date Table Marked as Date Table
The Date table is marked as a Date Table in Power BI 
using the `Date` column. This is required for all time 
intelligence DAX functions to work correctly including 
SAMEPERIODLASTYEAR, TOTALYTD, TOTALMTD, and TOTALQTD.

### Active vs Inactive Relationships
Sales table has three date foreign keys:
- `OrderDateKey` → **Active** — used for all standard reporting
- `ShipDateKey` → **Inactive** — activated using USERELATIONSHIP()
- `DueDateKey` → **Inactive** — activated using USERELATIONSHIP()

### CustomerKey = -1
Rows where CustomerKey = -1 represent reseller 
transactions not direct customer sales. These rows 
are filtered out of customer specific visuals using 
visual level filters.

### All Measures in _Measures Table
All 47 DAX measures are stored in a dedicated 
`_Measures` table using the Home Table property 
in Power BI. This keeps the model clean and makes 
measures easy to find during development and review.

---

## Data Limitations

| Limitation | Detail |
|---|---|
| FY2021 has no sales data | Date table contains FY2021 but Sales table has no transactions — hidden from slicers |
| Unit Price Discount Pct is all zeros | No discount data available in this dataset version |
| CustomerKey and ResellerKey both populated | Every Sales row has both keys filled — B2B vs B2C segmentation based on CustomerKey = -1 |

---

## Tools Used

| Tool | Purpose |
|---|---|
| Power BI Desktop | Dashboard development |
| DAX | Measure calculations |
| Power Query | Data loading and transformation |
| Microsoft AdventureWorks | Source dataset |
