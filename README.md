# 🛒 Superstore Sales Analytics — Power BI Report

![Power BI](https://img.shields.io/badge/Power%20BI-F2C811?style=for-the-badge&logo=powerbi&logoColor=black)
![DAX](https://img.shields.io/badge/DAX-0078D4?style=for-the-badge&logo=microsoft&logoColor=white)
![Power Query](https://img.shields.io/badge/Power%20Query-217346?style=for-the-badge&logo=microsoft&logoColor=white)
![TMDL](https://img.shields.io/badge/TMDL-0078D4?style=for-the-badge&logo=microsoft&logoColor=white)

A fully documented, production-grade Power BI semantic model and report built on the classic Superstore dataset. The project covers end-to-end BI development — from data modelling and ETL to advanced DAX, interactive visuals, tariff scenario analysis, and a dynamic self-service data dictionary.

---

## 📊 Report Pages

| Page | Description |
|------|-------------|
| **Overview** | High-level KPI summary with revenue, profit, margin and cost cards with sparklines |
| **Geography** | Revenue by state map, city efficiency scatter plot and city-level details matrix |
| **Products** | Product and sub-category performance with clustering and trend analysis |
| **Time Intelligence** | YTD/QTD/MTD dynamic switching with YoY growth and target tracking |
| **Tariff Analysis** | What-if scenario modelling for tariff rates and price increase impacts |
| **Dynamic Data Dictionary** | Searchable, filterable documentation of all model objects |

---

## 🗂️ Data Model

### Star Schema
The model follows a **normalised star schema** with a single fact table and dedicated dimension tables.

```
DateTable ──────────────────────────────┐
                                        │
Subcategory ── Product ── Sales ── Customer
                              │
                         Target_Table ── DateTable
                              │
                         Subcategory
```

### Tables

| Table | Type | Description |
|-------|------|-------------|
| `Sales` | Fact | Transactional sales data — revenue, cost, profit, quantity, shipping |
| `Customer` | Dimension | Customer details — region, state, city, segment |
| `Product` | Dimension | Product hierarchy — sub-category, product name, clusters |
| `Subcategory` | Dimension | Product category and sub-category hierarchy |
| `DateTable` | Calculated | Date dimension generated via `CALENDARAUTO()` |
| `Target_Table` | Reference | Annual targets for revenue, profit, orders and costs |
| `Scenarios` | Parameter | Tariff rate and price increase combinations for what-if analysis |
| `TimeFilter` | Parameter | YTD / QTD / MTD period switcher |
| `Parameter` | Field Parameter | Dynamic metric switcher (Revenue, Profit, Margin, Cost, Shipping) |
| `Dimension Switcher` | Field Parameter | Dynamic dimension switcher (Region, State, City, Category, Sub-Category) |
| `Price Increase Rate` | Calculated | GENERATESERIES 0–100% in steps of 10 |
| `Tariff Rate` | Calculated | GENERATESERIES 0–100% in steps of 10 |
| `ClusterMappingTable` | Calculated | K-Means clusters for products (6 clusters, 2025 data) |
| `ClusterMappingTable 2` | Calculated | K-Means clusters for cities (4 clusters, 2025 data) |
| `Model Tables` | Metadata | `INFO.VIEW.TABLES()` with injected descriptions |
| `Model Columns` | Metadata | `INFO.VIEW.COLUMNS()` with descriptions |
| `Model Measures` | Metadata | `INFO.VIEW.MEASURES()` |
| `Model Relationships` | Metadata | `INFO.VIEW.RELATIONSHIPS()` with injected descriptions |
| `Data Dictionary` | Calculated | Union of all model objects for the dynamic documentation page |

### Relationships

| From | To | Cardinality | Active |
|------|----|-------------|--------|
| `Sales[Customer ID]` | `Customer[Customer ID]` | Many-to-one | ✅ |
| `Sales[Product ID]` | `Product[Product ID]` | Many-to-one | ✅ |
| `Sales[Order Date]` | `DateTable[Date]` | Many-to-one | ✅ |
| `Sales[Ship Date]` | `DateTable[Date]` | Many-to-one | ❌ Inactive |
| `Product[Sub-Category]` | `Subcategory[Sub-Category]` | Many-to-one | ✅ |
| `Target_Table[Start of Month]` | `DateTable[Date]` | Many-to-one | ✅ |
| `Target_Table[Sub-Category]` | `Subcategory[Sub-Category]` | Many-to-one | ✅ |

---

## ⚙️ ETL — Power Query

All tables share a single staging query `Superstore_Source` which handles the CSV connection, type changes and base transformations. Dimension tables (`Customer`, `Product`) reference this staging query and select only their relevant columns — ensuring a **single source of truth** with no version mismatches.

```
Superstore_Source (disabled load)
    ├── Sales         → filters, sorts, selects fact columns
    ├── Customer      → deduplicates by Customer ID
    └── Product       → deduplicates by Product ID, removes Category
```

### Source Data
- **Format:** CSV (`,` delimiter, UTF-8)
- **Rows:** 9,994 transactions
- **Columns:** 27 source columns
- **Coverage:** US only, 49 states (excludes Alaska and Hawaii)
- **Date Range:** 2018–2025 (42 rows with 2026 Ship Dates are filtered out)

---

## 📐 DAX Measures

The model contains **80+ measures** organised into display folders across two measure tables:

### `_Key Metrics`
| Folder | Measures |
|--------|----------|
| Revenue | Net Revenue, Gross Revenue, Dynamic Revenue, Revenue Target, YoY %, MoM %, Sparklines, Progress bars |
| Profit | Total Profit, Dynamic Profit, Profit Target, YoY %, Sparklines, Progress bars |
| Profit Margin % | Profit Margin %, 3M MA, Dynamic, Target, Gap, Sparklines |
| Total Cost | Total Cost, Dynamic Total Cost, Target, YoY %, Sparklines |
| Orders | Orders Count, Dynamic Orders, Target, YoY %, Progress bars |
| Quantity | Quantity Sold, Sparklines |
| Tariffs | All adjusted metrics under tariff/price increase scenarios |

### `_Other Measures`
| Measure | Description |
|---------|-------------|
| `AVG Cost per Product` | Average cost per unit across all products |
| `AVG Price per Product` | Average selling price per unit |
| `City AOV` | Average Order Value by city |
| `Customer Count` | Distinct customer count |
| `Freight %` | Shipping costs as % of revenue |
| `Shipping Costs` | Total shipping costs |
| `Last Refreshed` | Timestamp of last data refresh |
| `Time-Intelligence *` | Dynamic titles for cards, donuts, treemaps and tables |
| `Word Cloud Title` | Dynamic word cloud title based on selected parameter |
| `TOP Subcategory` | Top sub-category by Net Revenue |
| `Total Value Discounted` | Sum of discounted sales value |

---

## 🎯 Key Features

### Dynamic Time Intelligence
All key metrics support **YTD / QTD / MTD** switching via the `TimeFilter` parameter table, using `TOTALYTD`, `TOTALQTD`, `TOTALMTD` with `SAMEPERIODLASTYEAR` for YoY comparisons.

### Tariff Scenario Analysis
A dedicated what-if page allows users to model the impact of **tariff rates (0–100%)** and **price increases (0–100%)** on adjusted revenue, cost, profit and margin — with a waterfall chart showing the decomposition of profit impact.

### K-Means Clustering
Products and cities are segmented using **Power BI's built-in K-Means clustering**:
- **Products** → 6 clusters: Star Products, High Volume Core, Low Volume Core, Loss Drivers, Volume Fillers, Contributors
- **Cities** → 4 clusters: Flagship Cities, Contributors, Margin Risks, Small Markets

### SVG Sparklines & Progress Bars
All KPI cards include inline **SVG sparklines** rendered as `ImageUrl` calculated measures, with gradient fills and endpoint dots. Progress bars use colour-coded fill to show target achievement at sub-category level.

### Dynamic Data Dictionary
A fully searchable **semantic model documentation page** powered by `INFO.VIEW.*` functions, combining columns, measures, tables and relationships into a single `UNION`-based calculated table with injected descriptions.

---

## 📁 Repository Structure

```
📦 Superstore-PowerBI
 ┣ 📂 TMDL
 ┃ ┣ 📄 Measures_TMDL.txt          # All measures with /// descriptions
 ┃ ┣ 📄 Columns_TMDL.txt           # Model Columns table
 ┃ ┣ 📄 Tables_TMDL.txt            # Model Tables table
 ┃ ┣ 📄 Sales_Columns_TMDL.txt     # Sales table columns with descriptions
 ┃ ┣ 📄 Product_Columns_TMDL.txt   # Product table columns with descriptions
 ┃ ┗ 📄 Datetable_Columns_TMDL.txt # DateTable columns with descriptions
 ┣ 📂 Data
 ┃ ┗ 📄 Superstore_Prod_v2.csv     # Source data
 ┗ 📄 README.md
```

---

## 🛠️ Technical Stack

| Tool | Usage |
|------|-------|
| **Power BI Desktop** | Report development, semantic model |
| **TMDL** | Model documentation, descriptions, version control |
| **Power Query (M)** | ETL, data normalisation, staging query pattern |
| **DAX** | Measures, calculated columns, calculated tables |
| **Tabular Editor** | Model metadata, annotations |
| **INFO.VIEW functions** | Self-documenting data dictionary |
| **K-Means Clustering** | Product and city segmentation |

---

## 🚀 Getting Started

1. Clone the repository
2. Update the file path in the `Superstore_Source` Power Query staging query to point to your local copy of `Superstore_Prod_v2.csv`
3. Refresh the dataset
4. All tables, measures and relationships will load automatically

> **Note:** The file path is centralised in `Superstore_Source` — you only need to update it in one place.

---

## 👤 Author

**José Pinto Lima dos Santos**

---

## 📄 License

This project uses the Superstore sample dataset which is publicly available for educational and portfolio purposes.
