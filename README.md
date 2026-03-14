# Data-Engineering-Project

# 🏗️ Azure End-to-End Data Engineering Pipeline
### AdventureWorks Sales Analytics | Medallion Architecture (Bronze → Silver → Gold)

---

## 📌 Project Overview

This project demonstrates a **production-style data engineering pipeline** built entirely on Microsoft Azure. Raw sales data is ingested from GitHub via the GitHub API into Azure Data Lake, progressively transformed through Bronze, Silver, and Gold layers, and finally served as a live Power BI dashboard.

**Dataset:** AdventureWorks — a Microsoft sample database simulating a bicycle manufacturing company's sales operations across territories, products, and customers (2015–2017).

---

## 🏛️ Architecture
```
GitHub (CSV files)
      │
      ▼  [GitHub API]
Azure Data Factory (ADF)
      │
      ▼  [Raw ingestion]
ADLS Gen2 — Bronze Layer
(raw CSVs stored as-is)
      │
      ▼  [PySpark transformations]
Azure Databricks — Silver Layer
(cleaned, typed, deduplicated)
      │
      ▼  [Aggregated business views]
Azure Synapse Analytics — Gold Layer
(star schema, analytics-ready)
      │
      ▼  [Live connection]
Power BI Dashboard
(KPIs, trends, territory analysis)
```

---

## 📁 Dataset — 10 Tables

| File | Description | Rows (approx) |
|------|-------------|---------------|
| `AdventureWorks_Sales_2015.csv` | Transaction-level sales data | ~7,000 |
| `AdventureWorks_Sales_2016.csv` | Transaction-level sales data | ~9,000 |
| `AdventureWorks_Sales_2017.csv` | Transaction-level sales data | ~12,000 |
| `AdventureWorks_Customers.csv` | Customer demographics | ~18,000 |
| `AdventureWorks_Products.csv` | Product master with pricing | ~293 |
| `AdventureWorks_Product_Categories.csv` | 4 top-level categories | 4 |
| `AdventureWorks_Product_Subcategories.csv` | 37 subcategories | 37 |
| `AdventureWorks_Returns.csv` | Return transactions by territory | ~1,800 |
| `AdventureWorks_Territories.csv` | Sales territory master | 10 |
| `AdventureWorks_Calendar.csv` | Date dimension table | ~1,461 |

---

## ⚙️ Pipeline — Step by Step

### 🥉 Bronze Layer — Raw Ingestion (Azure Data Factory)
- Created an ADF pipeline using the **GitHub API** as the HTTP source
- Parameterized the pipeline to loop through all 10 CSV files dynamically
- Stored raw files in **Azure Data Lake Storage Gen2** with folder structure:
```
  adls/
  └── bronze/
      ├── sales/
      ├── customers/
      ├── products/
      └── ...
```
- No transformations at this layer — raw data preserved for audit/reprocessing

### 🥈 Silver Layer — Transformation (Azure Databricks + PySpark)
See: [`SILVER_LAYER.ipynb`](./SILVER_LAYER.ipynb)

Key transformations applied:
- **Schema enforcement** — cast columns to correct data types (dates, integers, decimals)
- **Null handling** — identified and handled missing values in customer and product tables
- **Deduplication** — removed duplicate order rows in sales data
- **Column standardization** — renamed columns to consistent snake_case
- **Date formatting** — parsed multiple date formats into uniform `yyyy-MM-dd`
- **Sales union** — merged 2015, 2016, 2017 sales tables into a single `fact_sales` table
- Output written as **Parquet** back to ADLS Gen2 Silver zone

### 🥇 Gold Layer — Business-Ready Views (Azure Synapse Analytics)
- Created external tables in Synapse pointing to Silver Parquet files
- Built aggregated views for reporting:
  - `vw_sales_by_territory` — revenue and order count by region
  - `vw_product_performance` — sales and return rate by category/subcategory
  - `vw_customer_segments` — order frequency and revenue by customer
  - `vw_monthly_trend` — month-over-month revenue trend 2015–2017

### 📊 Power BI Dashboard
Live connection to Synapse Gold views via DirectQuery.

Dashboard pages:
- **Sales Overview** — Total revenue, orders, AOV by year
- **Territory Analysis** — Map view of revenue by region
- **Product Performance** — Category/subcategory breakdown with return rates
- **Customer Insights** — Top customers, regional distribution

---

## 💡 Key Business Insights from the Data

- **Bikes accounted for ~96% of total revenue** despite being outnumbered by Accessories in SKU count
- **The Southwest territory** consistently led in revenue across all 3 years
- **December and June** were peak sales months — consistent B2C seasonality
- **Accessories had the highest return rate** relative to order volume
- Revenue grew **~25% YoY from 2015 to 2017**

---

## 🛠️ Tech Stack

| Layer | Tool |
|-------|------|
| Source | GitHub API (HTTP connector) |
| Orchestration | Azure Data Factory |
| Storage | Azure Data Lake Storage Gen2 |
| Transformation | Azure Databricks (PySpark) |
| Serving | Azure Synapse Analytics |
| Visualization | Power BI |
| Notebooks | Jupyter / Databricks Notebooks |
| IAM | Azure Role-Based Access Control |

---

## 📂 Repo Structure
```
📦 Data-Engineering-Project
 ├── AdventureWorks_Sales_2015.csv
 ├── AdventureWorks_Sales_2016.csv
 ├── AdventureWorks_Sales_2017.csv
 ├── AdventureWorks_Customers.csv
 ├── AdventureWorks_Products.csv
 ├── AdventureWorks_Product_Categories.csv
 ├── AdventureWorks_Product_Subcategories.csv
 ├── AdventureWorks_Returns.csv
 ├── AdventureWorks_Territories.csv
 ├── AdventureWorks_Calendar.csv
 ├── SILVER_LAYER.ipynb         ← PySpark transformation notebook
 └── README.md
```

---

## 🚀 How to Reproduce

1. Upload the CSV files to your Azure Data Lake Gen2 (Bronze container)
2. Create an ADF pipeline with HTTP connector pointing to this GitHub repo's raw file URLs
3. Run the `SILVER_LAYER.ipynb` notebook in Azure Databricks
4. Create external tables in Synapse Analytics pointing to the Silver Parquet output
5. Connect Power BI to Synapse via DirectQuery

---

## 👩‍💻 Author

**Archita Rai** — Data Analyst | Business Analyst | Insurance & FinTech  
[LinkedIn](https://linkedin.com/in/archita-rai-68344122b) · [GitHub](https://github.com/Raiarchita)

---

*Built as part of a portfolio project to demonstrate enterprise-grade data engineering on Microsoft Azure.*
