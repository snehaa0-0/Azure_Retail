# 🛒 Azure Retail Analytics — End-to-End ELT Pipeline
### Medallion Architecture (Bronze → Silver → Gold) using Azure Synapse Analytics

---

## 📌 Project Overview

This project demonstrates a complete, production-style **ELT (Extract, Load, Transform) data pipeline** built on **Azure Synapse Analytics**. Raw retail transaction data is ingested from an HTTP source, cleaned and transformed using **PySpark notebooks**, and finally exposed as a queryable **SQL External Table** using Synapse Serverless SQL Pool.

The pipeline follows the **Medallion Architecture** pattern — a best practice in modern data engineering — ensuring data quality and reliability at every stage before it reaches business users.

---

## 🏗️ Architecture

```
[HTTP Source — Raw JSON Data]
            │
            ▼
[Copy Data Pipeline — Synapse Integrate Tab]
            │
            ▼
[🥉 Bronze Layer — ADLS Gen2 /bronze folder]
   Raw Parquet files, no transformations
            │
            ▼
[🥈 Silver Layer — PySpark Notebook]
   Filter: purchases only
   Clean: drop nulls (customer_id, amount)
   Transform: fix timestamps, cast data types
            │
            ▼
[🥇 Gold Layer — PySpark Notebook]
   Aggregate: daily_revenue, total_purchases per day
   Output: clean Parquet ready for reporting
            │
            ▼
[Serverless SQL Pool — External Table]
   SELECT * FROM daily_revenue
```

> 📁 Architecture diagram image coming soon

---

## 🛠️ Tech Stack

| Tool | Purpose |
|---|---|
| Azure Synapse Analytics | Core platform — pipelines, notebooks, SQL |
| Azure Data Lake Storage Gen2 | Storage for all three layers |
| Apache Spark (PySpark) | Silver and Gold layer transformations |
| Synapse Serverless SQL Pool | SQL presentation layer |
| Parquet File Format | Columnar storage for efficient querying |
| Azure Blob Storage | Source storage account |
| Database Master Key + Scoped Credentials | Secure storage authentication |

---

## 📂 Project Structure

```
azure-synapse-retail-analytics/
│
├── README.md
├── architecture/
│   └── architecture-diagram.png
│
├── 01_pipeline/
│   └── bronze_ingestion_pipeline.json    # Copy Data pipeline export
│
├── 02_notebooks/
│   ├── silver_cleaning.ipynb             # PySpark cleaning notebook
│   └── gold_aggregation.ipynb            # PySpark aggregation notebook
│
├── 03_sql/
│   ├── 01_create_database.sql
│   ├── 02_create_master_key.sql
│   ├── 03_create_external_datasource.sql
│   ├── 04_create_file_format.sql
│   ├── 05_create_external_table.sql
│   └── 06_query_daily_revenue.sql
│
└── screenshots/
    ├── 01_pipeline_success.png
    ├── 02_storage_folders.png
    ├── 03_silver_notebook_output.png
    ├── 04_gold_folder.png
    └── 05_sql_query_results.png
```

---

## 🔄 Pipeline Walkthrough

### 🥉 Phase 1 — Bronze Layer (Raw Ingestion)
- Created a **Copy Data pipeline** in the Synapse Integrate tab
- Source: HTTP dataset pointing to raw JSON retail transaction data
- Sink: ADLS Gen2, saved as **Parquet** in the `/bronze` folder
- Handled data type mapping issues (e.g. `amount` column mapped to `Double` to prevent conversion errors)

### 🥈 Phase 2 — Silver Layer (Data Cleaning with PySpark)
- Created an **Apache Spark Pool** to run PySpark notebooks
- Loaded raw Parquet from the Bronze layer
- Applied the following transformations:
  - Filtered rows to keep only `event_type = 'purchase'`
  - Dropped rows with null values in `customer_id` and `amount`
  - Converted `event_timestamp` to proper Date format using `to_date()`
  - Cast `amount` to Float for accuracy
- Saved cleaned output to `/silver` folder as Parquet

### 🥇 Phase 3 — Gold Layer (Aggregation with PySpark)
- Loaded cleaned Silver data into a new notebook
- Grouped data by `event_date`
- Calculated:
  - `daily_revenue` → `sum(amount)`
  - `total_purchases` → `count(*)`
- Saved final aggregated output to `/gold` folder as Parquet

### 🗄️ Phase 4 — SQL Presentation Layer
- Created a dedicated database `retailpoc` in Synapse Serverless SQL Pool
- Configured security using **Database Master Key** and **Database Scoped Credentials**
- Created an **External Data Source** pointing to the ADLS Gen2 storage account
- Defined a **Parquet External File Format**
- Created an **External Table** (`daily_revenue`) mapped to the Gold layer files
- Verified results with `SELECT * FROM daily_revenue`

---

## 💡 Key Skills Demonstrated

- **Medallion Architecture** — Bronze/Silver/Gold data layering
- **Pipeline Orchestration** — Copy Data activity with schema mapping
- **Data Type Troubleshooting** — Resolved ingestion errors via manual column mapping
- **PySpark Transformations** — Filtering, null handling, type casting, aggregations
- **Serverless SQL** — Cost-efficient querying with no dedicated infrastructure
- **External Tables** — Virtual SQL tables over Data Lake files
- **Storage Security** — Database Master Key, Scoped Credentials, External Data Source

---

## 🚀 How to Replicate This Project

1. Create an Azure Synapse Analytics workspace with an ADLS Gen2 storage account
2. Create three folders in your container: `bronze`, `silver`, `gold`
3. Run the pipeline in `01_pipeline/` to ingest raw data into the Bronze layer
4. Attach a Spark Pool and run `02_notebooks/silver_cleaning.ipynb` to clean the data
5. Run `02_notebooks/gold_aggregation.ipynb` to produce the final aggregated table
6. Run the SQL scripts in `03_sql/` in order (01 through 06) to expose the Gold data as a queryable external table

---

## 👩‍💻 Author

**Sneha Sharon**
- 💼 [LinkedIn](https://www.linkedin.com/in/) ← *add your LinkedIn URL here*
- 🐙 [GitHub](https://github.com/) ← *add your GitHub profile URL here*

---

> *This project was built as a Proof of Concept (POC) to demonstrate hands-on experience with Azure Synapse Analytics and modern data engineering practices.*
