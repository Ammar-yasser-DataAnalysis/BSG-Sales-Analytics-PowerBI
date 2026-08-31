# 📊 Business Sales Group — Sales Analytics Dashboard

> A comprehensive Power BI analytics solution for **Business Sales Group (BSG)**, built on a multi-source dataset using Power Query (M Language) for ETL, a Star Schema data model, and advanced DAX measures — delivering executive-level financial insights and strategic market analysis.

---

## 📌 Project Overview

This project analyzes sales data for a wholesale distribution company across multiple U.S. states and product categories. The full pipeline covers data ingestion from CSV and Excel files, cleaning and transformation in Power Query, star schema modeling in Power BI, and a 3-page interactive dashboard with a dedicated Data Quality & Audit page.

---

## 🛠️ Tech Stack

| Layer | Tool |
|---|---|
| **Data Sources** | CSV files + Excel workbook |
| **ETL & Transformation** | Power Query (M Language — Advanced Editor) |
| **Data Modeling** | Power BI Star Schema |
| **Visualization** | Power BI Desktop |
| **Measures** | DAX |

---

## 🗂️ Dashboard Pages

### 1. Executive Financial Summary
High-level KPIs for leadership:

| KPI | Value |
|---|---|
| Total Sales | **$22.86M** |
| Total Profit | **$9.92M** |
| Average Order Value (AOV) | **$865.82** |
| Profit Margin | **43.42%** |

**Visuals:**
- Top 5 Cities by Sales (Akhiok, San Jacinto, Fieldbrook, Wapinitia, Haycock)
- Total Profit by State (California $2.09M leads)
- Sales by Buying Group — Wingtip Toys (40.53%), Tailspin Toys (25.42%)
- Sales & Profit Year Over Year (2013–2016)
- Total Sales: Holidays vs. Working Days (monthly breakdown)

### 2. Strategic Market Analysis
Sales performance and product-level insights:

| KPI | Value |
|---|---|
| Active Salespersons | **49** |
| Total Customers | **402** |
| Sales per Active Customer | **12.19%** |
| Total Orders | **26K** |
| Total Quantity | **1M** |
| Avg Population | **13.18K** |

**Visuals:**
- Total Sales per Salesman (Sophia Hinton leads at $2.55M)
- Profit vs. Sales gauge ($9.92M / $22.86M)
- Top 5 Products by Sales (Air cushion machine (Blue) — $1.38M)
- Total Profit by Product (Top: 20mm Double — $0.7M)
- AOV by State Province (treemap — Oregon $915.20 highest)

### 3. Data Quality & Audit
Transparency and data integrity reporting:

| KPI | Value |
|---|---|
| Total Tax | **$2.98M** |
| Undelivered Orders | **13** |
| Sales per Working Day | **$20.11M** |
| Profit Margin | **43.42%** |
| Sales per Holiday | **$2.75M** |

**Visuals:**
- Total Sales (incl. tax) & Unit Price by Color — N/A dominates at $10M
- Count by State Province (California 1,187 — highest population)
- Top 5 Customers by Sales & Profit

---

## ⚙️ Power Query Transformations (Advanced Editor)

### 📦 DimStockItem
Key cleaning steps:
- Removed top row and promoted headers
- Split `Recommended Retail Price` by `?` delimiter → kept valid part → replaced ` -` with `0`
- Removed `Photo` and `Discount` columns (irrelevant/empty)
- Split `Valid From` and `Valid To` by position → extracted time component → replaced corrupted `Valid To` values (`###...`) with `null`
- Final type casting to proper numeric and time types

```m
#"Split Column by Delimiter" = Table.SplitColumn(#"Filtered Rows", "Recommended Retail Price",
    Splitter.SplitTextByDelimiter("?", QuoteStyle.Csv), {"Recommended Retail Price.1", "Recommended Retail Price.2"}),
#"Replaced Value1" = Table.ReplaceValue(#"Changed Type4",
    "#############...", null, Replacer.ReplaceValue, {"Valid To.1"})
```

---

### 🏙️ DimCity
Key cleaning steps:
- Promoted headers from first row
- Set correct data types for all 7 columns
- No major issues — clean source

---

### 👤 DimCustomer
Key cleaning steps:
- Double header issue — promoted headers twice to reach actual column names
- Split `Credit Limit` by `?` delimiter → kept valid part → replaced ` -` with `0`
- Split `Valid From` by position → extracted time → removed `Valid To` column
- Replaced all `N/A` values with `null` across: Bill To Customer, Category, Buying Group, Primary Contact, Postal Code

```m
#"Replaced Value1" = Table.ReplaceValue(#"Changed Type5","N/A",null,Replacer.ReplaceValue,{"Bill To Customer"}),
#"Replaced Value2" = Table.ReplaceValue(#"Replaced Value1","N/A",null,Replacer.ReplaceValue,{"Category"}),
...
```

---

### 📅 DimDate
Key cleaning steps:
- Extracted `DayName` from `Date` column using `Date.DayOfWeekName()`
- Removed redundant columns: `Day Number`, `Month`, `Calendar Month Number`, `Calendar Year Label`, etc.
- Merged with a holidays reference sheet (`Sheet1`) via Left Outer Join on `Date`
- Renamed `Holiday Name` column to `Is Weekend` for dashboard use

```m
#"Duplicated Column" = Table.DuplicateColumn(#"Changed Type", "Date", "Date - Copy"),
#"Extracted Day Name" = Table.TransformColumns(#"Duplicated Column",
    {{"Date - Copy", each Date.DayOfWeekName(_), type text}}),
#"Merged Queries" = Table.NestedJoin(#"Changed Type1", {"Date"}, Sheet1, {"Date"}, "Sheet1", JoinKind.LeftOuter)
```

---

### 👨‍💼 DimEmployee
Key cleaning steps:
- Source: Excel workbook (`.xlsx`)
- Removed `Photo` column
- Replaced corrupted `Valid To` value (`2958466`) with `null`
- Renamed `Salesperson Key` → `Employee Key` for model consistency

```m
#"Replaced Value" = Table.ReplaceValue(#"Removed Columns",2958466,null,Replacer.ReplaceValue,{"Valid To"})
```

---

### 💰 FactSale
Key cleaning steps:
- Renamed `Salesperson Key` → `Employee Key` to match `DimEmployee`
- Added engineered column `Duration` = `Delivery Date Key` − `Invoice Date Key` (in days)
- Reordered columns for readability

```m
#"Inserted Date Subtraction" = Table.AddColumn(#"Changed Type1", "Subtraction",
    each Duration.Days([Delivery Date Key] - [Invoice Date Key]), Int64.Type),
#"Renamed Columns1" = Table.RenameColumns(#"Reordered Columns",{{"Subtraction", "Duration"}})
```

---

## 🗃️ Data Model (Star Schema)

```
         DimCity
            |
DimEmployee ──── FactSale ──── DimStockItem
            |         |
       DimCustomer  DimDate
                      |
                   USA Date (holidays reference)
                   Measure (DAX measures table)
```
## Data Model (Star Schema) Screenshots (Click to enlarge) :
<img src="https://github.com/Ammar-yasser-DataAnalysis/BSG-Sales-Analytics-PowerBI/blob/main/Data%20Modeling.jpg">

---

| Table | Type | Key |
|---|---|---|
| `FactSale` | Fact | Sale Key |
| `DimCity` | Dimension | City Key |
| `DimCustomer` | Dimension | Customer Key |
| `DimEmployee` | Dimension | Employee Key |
| `DimStockItem` | Dimension | Stock Item Key |
| `DimDate` | Dimension | Date |
| `USA Date` | Reference | Date (holidays) |
| `Measure` | DAX Table | — |

---
## Dashboard Screenshots (Click to enlarge) :
<img src="https://github.com/Ammar-yasser-DataAnalysis/BSG-Sales-Analytics-PowerBI/blob/main/Executive%20Financial%20Summary.jpg">
<img src="https://github.com/Ammar-yasser-DataAnalysis/BSG-Sales-Analytics-PowerBI/blob/main/Srategic%20Market%20Analysis.jpg">
<img src="https://github.com/Ammar-yasser-DataAnalysis/BSG-Sales-Analytics-PowerBI/blob/main/Data%20Quality%20%26%20Audit.jpg">
<img src="https://github.com/Ammar-yasser-DataAnalysis/BSG-Sales-Analytics-PowerBI/blob/main/Conclusion.jpg">
---

## 🔍 Data Quality Findings

Documented transparently in the **Data Quality & Audit** page:

| Issue | Detail |
|---|---|
| Missing Customer Key | 9,077 of 26,397 sales (34.4%) have Customer Key = 0 |
| Missing Brand | 605 of 673 items (90%) missing Brand |
| Missing Color | 312 items (46%) missing Color |
| Missing Barcode | 656 items (97%) missing Barcode |
| Zero Population | 3,734 of 13,028 cities (28.7%) report population = 0 |
| Undelivered Orders | 13 invoices missing Delivery Date (undelivered or cancelled) |

---

## 📁 Repository Structure

```
BSG-Sales-Analytics-Dashboard/
│
├── README.md
├── BSG_Dashboard.pbix               # Power BI dashboard file
├── Screenshots/
│   ├── 01_Executive_Financial_Summary.png
│   ├── 02_Strategic_Market_Analysis.png
│   ├── 03_Data_Quality_Audit.png
│   ├── Data_Modeling.png
│   └── Conclusion.png
└── Dataset/
    ├── DimCity.csv
    ├── DimCustomer.csv
    ├── DimDate.csv
    ├── DimEmployee.xlsx
    ├── DimStockItem.csv
    └── FactSale.csv
```

---

## 🚀 Getting Started

### Prerequisites
- Power BI Desktop (latest version)
- Dataset files in a local folder

### Steps

1. **Clone the repository**
   ```bash
   git clone https://github.com/your-username/BSG-Sales-Analytics-Dashboard.git
   ```

2. **Update data source path**
   In Power BI Desktop → Transform Data → Data Source Settings → update the folder path to your local dataset location.

3. **Refresh data**
   Click **Home → Refresh All** to load the data.

4. **Explore the dashboard**
   Navigate between the 3 pages using the arrows on the dashboard.

---

## 💡 Key Business Insights

- **California** is the top state by both profit ($2.09M) and city count — priority market
- **Wingtip Toys** dominates buying groups at 40.53% of total sales
- **Sophia Hinton** is the top salesperson at $2.55M — 23% above average
- **Holiday sales ($2.75M)** are significantly lower than working day sales ($20.11M) — promotions opportunity
- **43.42% profit margin** is strong but 34.4% of sales have no linked customer — CRM data gap
- **Air cushion machine (Blue)** is the #1 product by revenue at $1.38M

---

## 📄 License

This project is for educational and portfolio purposes, developed as part of the Digital Hub × Orange training program.
