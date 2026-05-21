# 🏢 SQL Server Data Warehouse — Modern Medallion Architecture

[![SQL Server](https://img.shields.io/badge/SQL_Server-CC2927?style=for-the-badge&logo=microsoft-sql-server&logoColor=white)](https://www.microsoft.com/en-us/sql-server)
[![T-SQL](https://img.shields.io/badge/T--SQL-CC2927?style=for-the-badge&logo=microsoft-sql-server&logoColor=white)](https://docs.microsoft.com/en-us/sql/t-sql/)
[![SSMS](https://img.shields.io/badge/SSMS-CC2927?style=for-the-badge&logo=microsoft-sql-server&logoColor=white)](https://docs.microsoft.com/en-us/sql/ssms/)
[![Data Warehouse](https://img.shields.io/badge/Data_Warehouse-336791?style=for-the-badge&logo=databricks&logoColor=white)](#)
[![ETL Pipeline](https://img.shields.io/badge/ETL_Pipeline-FF6F00?style=for-the-badge&logo=apache-airflow&logoColor=white)](#)
[![Star Schema](https://img.shields.io/badge/Star_Schema-FFD700?style=for-the-badge&logo=postgresql&logoColor=black)](#)

> **A modern data warehouse built from scratch using SQL Server, implementing the Medallion Architecture (Bronze → Silver → Gold) with end-to-end ETL pipelines, dimensional modeling, and analytics-ready data products.**

---

## 📑 Table of Contents

- [🎯 Project Overview](#-project-overview)
- [🏗️ Data Architecture](#️-data-architecture)
- [🛠️ Tech Stack](#️-tech-stack)
- [📁 Project Structure](#-project-structure)
- [🥉 Bronze Layer](#-bronze-layer-raw-data)
- [🥈 Silver Layer](#-silver-layer-cleansed-data)
- [🥇 Gold Layer](#-gold-layer-business-ready)
- [📊 Data Model](#-data-model)
- [🚀 Getting Started](#-getting-started)
- [💡 Key Features](#-key-features)
- [📚 Skills Demonstrated](#-skills-demonstrated)

---

## 🎯 Project Overview

This project demonstrates the design and implementation of a **modern data warehouse on SQL Server** for sales analytics. It consolidates data from two enterprise source systems (CRM and ERP), transforms it through three architectural layers, and produces a dimensional model optimized for business intelligence.

### 🌟 Project Highlights

- ✨ **End-to-end Data Warehouse** built on SQL Server
- ✨ **Medallion Architecture** with three distinct layers
- ✨ **Multi-source integration** (CRM + ERP systems)
- ✨ **Star Schema** dimensional modeling (Kimball methodology)
- ✨ **Stored Procedures** for repeatable ETL execution
- ✨ **Data quality checks** at every layer

### 💼 Business Use Cases

This warehouse enables analytics on:
- 📈 **Sales performance** by customer, product, region
- 👥 **Customer 360** — demographics, purchase history
- 📦 **Product analytics** — categories, trends
- 🌍 **Geographic insights** — sales by country/region
- 📅 **Time-series reporting** — daily/monthly/yearly trends

---

## 🏗️ Data Architecture

The architecture follows the **Medallion (Multi-hop) Architecture** pattern, with three distinct layers:

```
╔══════════════════════════════════════════════════════════════════════╗
║                                                                      ║
║                       📂  SOURCE SYSTEMS                              ║
║                                                                      ║
║         ┌─────────────────┐         ┌─────────────────┐             ║
║         │   CRM SYSTEM    │         │   ERP SYSTEM    │             ║
║         │  ─────────────  │         │  ─────────────  │             ║
║         │ • cust_info     │         │ • CUST_AZ12     │             ║
║         │ • prd_info      │         │ • LOC_A101      │             ║
║         │ • sales_details │         │ • PX_CAT_G1V2   │             ║
║         └────────┬────────┘         └────────┬────────┘             ║
║                  │                            │                      ║
║                  └────────────┬───────────────┘                      ║
║                               │                                      ║
║                  CSV Files (Interface: File-based)                   ║
║                                                                      ║
╚═══════════════════════════════╪══════════════════════════════════════╝
                                │
                                │  📥 BULK INSERT
                                │  Full Load + Truncate
                                ▼
╔══════════════════════════════════════════════════════════════════════╗
║                                                                      ║
║                     🥉  BRONZE LAYER (Raw Zone)                       ║
║                     ───────────────────────────                       ║
║                                                                      ║
║   📌 Purpose:    Land raw data as-is from source systems              ║
║   📌 Schema:     bronze                                               ║
║   📌 Object:     Tables                                               ║
║   📌 Load Type:  Batch Processing → Full Load → Truncate & Insert     ║
║   📌 Transform:  None (preserves source structure)                    ║
║                                                                      ║
║   ┌──────────────────────────────────────────────────────────────┐  ║
║   │  bronze.crm_cust_info        bronze.erp_cust_az12            │  ║
║   │  bronze.crm_prd_info         bronze.erp_loc_a101             │  ║
║   │  bronze.crm_sales_details    bronze.erp_px_cat_g1v2          │  ║
║   └──────────────────────────────────────────────────────────────┘  ║
║                                                                      ║
║   📜 Stored Procedure: bronze.load_bronze                            ║
║                                                                      ║
╚══════════════════════════════╪═══════════════════════════════════════╝
                               │
                               │  🔄 Data Cleansing
                               │  Standardization & Quality Checks
                               ▼
╔══════════════════════════════════════════════════════════════════════╗
║                                                                      ║
║                    🥈  SILVER LAYER (Cleansed Zone)                   ║
║                    ─────────────────────────────                      ║
║                                                                      ║
║   📌 Purpose:    Clean, validate, and standardize data                ║
║   📌 Schema:     silver                                               ║
║   📌 Object:     Tables                                               ║
║   📌 Load Type:  Full Load → Truncate & Insert                        ║
║                                                                      ║
║   🔧 Transformations Applied:                                        ║
║                                                                      ║
║      ✓ Data Cleansing          (TRIM, UPPER, NULL handling)          ║
║      ✓ Data Standardization    (M → Married, S → Single)             ║
║      ✓ Data Normalization      (Country codes → Full names)          ║
║      ✓ Derived Columns         (Calculated fields)                   ║
║      ✓ Data Enrichment         (Cross-source joins)                  ║
║                                                                      ║
║   📜 Stored Procedure: silver.load_silver                            ║
║                                                                      ║
╚══════════════════════════════╪═══════════════════════════════════════╝
                               │
                               │  🎯 Business Logic
                               │  Dimensional Modeling
                               ▼
╔══════════════════════════════════════════════════════════════════════╗
║                                                                      ║
║                    🥇  GOLD LAYER (Business Zone)                     ║
║                    ─────────────────────────────                      ║
║                                                                      ║
║   📌 Purpose:    Business-ready analytical model                      ║
║   📌 Schema:     gold                                                 ║
║   📌 Object:     Views (no data duplication)                          ║
║   📌 Data Model: Star Schema                                          ║
║                                                                      ║
║   ⭐ Star Schema Tables:                                              ║
║                                                                      ║
║          ┌──────────────────────┐                                    ║
║          │  gold.dim_customers  │                                    ║
║          │  ──────────────────  │                                    ║
║          │  • customer_key (PK) │                                    ║
║          │  • customer details  │                                    ║
║          └──────────┬───────────┘                                    ║
║                     │                                                ║
║                     ▼                                                ║
║          ┌──────────────────────┐    ┌──────────────────────┐        ║
║          │   gold.fact_sales    │◄───┤  gold.dim_products   │        ║
║          │   ───────────────    │    │  ──────────────────  │        ║
║          │  • order details     │    │  • product_key (PK)  │        ║
║          │  • measures          │    │  • product details   │        ║
║          └──────────────────────┘    └──────────────────────┘        ║
║                                                                      ║
╚══════════════════════════════╪═══════════════════════════════════════╝
                               │
                               ▼
                  ╔════════════════════════════════╗
                  ║   📊  CONSUMPTION LAYER         ║
                  ║   ─────────────────────         ║
                  ║                                 ║
                  ║   • Power BI Dashboards         ║
                  ║   • Tableau Reports             ║
                  ║   • Ad-hoc SQL Queries          ║
                  ║   • Machine Learning            ║
                  ║                                 ║
                  ╚════════════════════════════════╝
```

---

## 🛠️ Tech Stack

| Layer | Technology |
|-------|------------|
| 💾 **Database** | Microsoft SQL Server |
| 🔤 **Language** | T-SQL (Transact-SQL) |
| 🖥️ **IDE** | SQL Server Management Studio (SSMS) |
| 📦 **Source Files** | CSV (from CRM and ERP systems) |
| 🏗️ **Architecture** | Medallion (Bronze / Silver / Gold) |
| 📐 **Data Model** | Star Schema (Kimball Methodology) |
| 🔧 **ETL Approach** | SQL-based ETL with Stored Procedures |
| 📊 **Visualization** | Draw.io for architecture diagrams |
| 🌿 **Version Control** | Git / GitHub |

---

## 📁 Project Structure

```
data-warehouse-project/
│
├── 📂 datasets/                           # Raw source datasets
│   ├── source_crm/                        # CRM system data
│   │   ├── cust_info.csv
│   │   ├── prd_info.csv
│   │   └── sales_details.csv
│   └── source_erp/                        # ERP system data
│       ├── CUST_AZ12.csv
│       ├── LOC_A101.csv
│       └── PX_CAT_G1V2.csv
│
├── 📂 docs/                               # Documentation & diagrams
│   ├── etl.drawio                         # ETL methods diagram
│   ├── data_architecture.drawio           # High-level architecture
│   ├── data_catalog.md                    # Field descriptions & metadata
│   ├── data_flow.drawio                   # Data flow diagram
│   ├── data_models.drawio                 # Star schema diagram
│   └── naming-conventions.md              # Naming standards
│
├── 📂 scripts/                            # ETL SQL scripts
│   ├── 📂 bronze/                         # Bronze layer scripts
│   │   ├── ddl_bronze.sql                 # Table creation
│   │   └── proc_load_bronze.sql           # Load procedure
│   │
│   ├── 📂 silver/                         # Silver layer scripts
│   │   ├── ddl_silver.sql                 # Table creation
│   │   └── proc_load_silver.sql           # Cleansing procedure
│   │
│   └── 📂 gold/                           # Gold layer scripts
│       └── ddl_gold.sql                   # View creation (star schema)
│
├── 📂 tests/                              # Quality validation scripts
│   ├── quality_checks_silver.sql
│   └── quality_checks_gold.sql
│
├── 📜 README.md                           # This file
├── 📜 LICENSE                             # MIT License
├── 📜 .gitignore                          # Git ignore patterns
└── 📜 requirements.txt                    # Project requirements
```

---

## 🥉 Bronze Layer (Raw Data)

**Goal**: Land raw data from source systems with **zero transformations**.

### 📌 Characteristics

- **Object Type**: Tables
- **Load Pattern**: Batch Processing → Full Load → Truncate & Insert
- **Transformations**: None (as-is)
- **Data Model**: None (mirrors source)

### 🔄 Source Systems

| System | Tables | Purpose |
|--------|--------|---------|
| 📂 **CRM** | `cust_info`, `prd_info`, `sales_details` | Customer & sales data |
| 📂 **ERP** | `CUST_AZ12`, `LOC_A101`, `PX_CAT_G1V2` | Enterprise master data |

### 💻 Sample Load Logic

```sql
-- Truncate & Bulk Insert Pattern
TRUNCATE TABLE bronze.crm_cust_info;

BULK INSERT bronze.crm_cust_info
FROM 'C:\datasets\source_crm\cust_info.csv'
WITH (
    FIRSTROW = 2,
    FIELDTERMINATOR = ',',
    TABLOCK
);
```

---

## 🥈 Silver Layer (Cleansed Data)

**Goal**: Clean, validate, and standardize data for downstream use.

### 📌 Characteristics

- **Object Type**: Tables
- **Load Pattern**: Full Load → Truncate & Insert
- **Transformations**: Multiple (see below)
- **Data Model**: None (mirrors silver, cleaned)

### 🔧 Transformations Applied
'''
| Transformation | Example |
|----------------|---------|
| 🧹 **Data Cleansing** | Remove NULLs, trim whitespace |
| 📏 **Standardization** | `'M'` → `'Married'`, `'S'` → `'Single'` |
| 🌍 **Normalization** | Country codes → Full country names |
| ➕ **Derived Columns** | Full name = first + last |
| 🔗 **Data Enrichment** | Join CRM + ERP data |
| 🚫 **Deduplication** | `ROW_NUMBER()` window function |

'''

## 🥇 Gold Layer (Business Ready)

**Goal**: Create analytics-ready dimensional model for BI tools.

### 📌 Characteristics

- **Object Type**: Views (no data duplication)
- **Load Pattern**: N/A (views, no load)
- **Transformations**: Business logic, aggregations
- **Data Model**: ⭐ **Star Schema**

### 📊 Star Schema Design

```
                    ┌──────────────────────────┐
                    │   gold.dim_customers     │
                    │   ────────────────────   │
                    │   customer_key  (PK)     │
                    │   customer_id            │
                    │   customer_number        │
                    │   first_name             │
                    │   last_name              │
                    │   country                │
                    │   marital_status         │
                    │   gender                 │
                    │   birthdate              │
                    │   create_date            │
                    └──────────┬───────────────┘
                               │
                               │
                               ▼
            ┌──────────────────────────────────┐
            │       gold.fact_sales            │
            │   ────────────────────────       │
            │   order_number      (PK)         │
            │   product_key       (FK) ────────┐
            │   customer_key      (FK)         │
            │   order_date                     │
            │   shipping_date                  │
            │   due_date                       │
            │   sales_amount                   │
            │   quantity                       │
            │   price                          │
            └──────────────────────────────────┘
                               ▲
                               │
                               │
                    ┌──────────┴───────────────┐
                    │   gold.dim_products      │
                    │   ────────────────────   │
                    │   product_key   (PK)     │
                    │   product_id             │
                    │   product_number         │
                    │   product_name           │
                    │   category_id            │
                    │   category               │
                    │   subcategory            │
                    │   maintenance            │
                    │   cost                   │
                    │   product_line           │
                    │   start_date             │
                    └──────────────────────────┘
```

### 🔑 Design Decisions

- ✅ **Surrogate keys** for performance
- ✅ **Business keys** preserved for traceability
- ✅ **Views** instead of tables (no duplication)
- ✅ **Slowly Changing Dimensions** (Type 1)
- ✅ **Optimized for BI** consumption

---

## 📊 Data Model

### Entity Relationship
'''
| Table | Type | Purpose |
|-------|------|---------|
| `gold.dim_customers` | Dimension | Customer demographics & attributes |
| `gold.dim_products` | Dimension | Product catalog & hierarchy |
| `gold.fact_sales` | Fact | Sales transactions (measures) |


-'''--

## 🚀 Getting Started

### Prerequisites

- ✅ Microsoft SQL Server (2019 or later)
- ✅ SQL Server Management Studio (SSMS)
- ✅ Source CSV files in `datasets/` folder

### Setup Steps

**1️⃣ Clone the repository**

```bash
git clone https://github.com/SurakantiMeghana26/sql-data-warehouse-project.git
cd sql-data-warehouse-project
```

**2️⃣ Create the database**

```sql
CREATE DATABASE DataWarehouse;
USE DataWarehouse;
```

**3️⃣ Create schemas**

```sql
CREATE SCHEMA bronze;
CREATE SCHEMA silver;
CREATE SCHEMA gold;
```

**4️⃣ Run DDL scripts** (in order)

```sql
-- Bronze layer tables
:r scripts/bronze/ddl_bronze.sql

-- Silver layer tables
:r scripts/silver/ddl_silver.sql

-- Gold layer views (star schema)
:r scripts/gold/ddl_gold.sql
```

**5️⃣ Load the data**

```sql
EXEC bronze.load_bronze;    -- Loads raw data
EXEC silver.load_silver;    -- Cleans and standardizes
-- Gold layer views are automatically available
```

**6️⃣ Verify the data**

```sql
SELECT TOP 10 * FROM gold.dim_customers;
SELECT TOP 10 * FROM gold.dim_products;
SELECT TOP 10 * FROM gold.fact_sales;
```

---

## 💡 Key Features

### 🎯 Production-Ready Practices

- ✅ **Schema separation** for clear layer boundaries
- ✅ **Stored procedures** for ETL execution
- ✅ **Error handling** with TRY/CATCH blocks
- ✅ **Logging** with PRINT statements
- ✅ **Quality checks** at each layer

### 🔍 Data Quality Validations

- ✅ NULL checks on primary keys
- ✅ Duplicate detection (window functions)
- ✅ Referential integrity validation
- ✅ Data type consistency
- ✅ Business rule validation

### ⚡ Performance Optimizations

- ✅ Bulk loading for raw data (`BULK INSERT`)
- ✅ Indexed primary/foreign keys
- ✅ Surrogate keys for efficient joins
- ✅ Views in gold layer (no data duplication)
- ✅ `TABLOCK` hint for fast loading

---

## 📚 Skills Demonstrated

### 🛠️ Data Engineering
- ✓ ETL pipeline design and implementation
- ✓ Medallion architecture (Bronze/Silver/Gold)
- ✓ Multi-source data integration (CRM + ERP)
- ✓ Data quality validation frameworks
- ✓ Source-to-target mapping
- ✓ Incremental vs full load patterns

### 💻 SQL Server / T-SQL
- ✓ **DDL**: CREATE TABLE, SCHEMA, VIEW
- ✓ **DML**: INSERT, UPDATE, MERGE
- ✓ **Stored Procedures** with parameters
- ✓ **Window functions** (ROW_NUMBER, RANK, OVER)
- ✓ **CTEs** and complex subqueries
- ✓ **BULK INSERT** operations
- ✓ **String functions** (TRIM, UPPER, REPLACE)
- ✓ **Date functions** (CAST, FORMAT, DATEPART)

### 📐 Data Modeling
- ✓ Star schema design (Kimball methodology)
- ✓ Fact and dimension tables
- ✓ Surrogate vs business keys
- ✓ Slowly Changing Dimensions (SCD)
- ✓ Data warehouse best practices

### 🎯 Documentation
- ✓ Data architecture diagrams (Draw.io)
- ✓ Data catalog creation
- ✓ Naming conventions documentation
- ✓ Data flow visualization

---

## 🔗 Related Projects

Explore my other data engineering projects:

- 🚕 [**NYC Taxi ETL Pipeline**](https://github.com/SurakantiMeghana26/nyc-taxi-aws-etl-piplines) — Cloud ETL on AWS
- 🎬 [**Movie Data API Pipeline**](https://github.com/SurakantiMeghana26/movie-data-api-pipeline) — REST API + PySpark
- 🌬️ [**Airflow Orchestration**](https://github.com/SurakantiMeghana26/airflow-aws-orchestration) — Workflow automation
- 🧱 [**Databricks Lakehouse**](https://github.com/SurakantiMeghana26/Databricks_Datalakehouse-_project) — Modern lakehouse pattern

---

## 👤 Author

**Surakanti Meghana** — Aspiring Data Engineer

Passionate about building robust data pipelines and turning raw data into actionable insights.

- 🐙 **GitHub**: [@SurakantiMeghana26](https://github.com/SurakantiMeghana26)
---

## 📜 License

This project is licensed under the **MIT License** — see the [LICENSE](LICENSE) file for details.

---

<div align="center">

### ⭐ If you found this project helpful, please give it a star! ⭐

**Built with 💙 by Surakanti Meghana**

[![GitHub stars](https://img.shields.io/github/stars/SurakantiMeghana26/sql-data-warehouse-project?style=social)](https://github.com/SurakantiMeghana26/sql-data-warehouse-project)
[![GitHub forks](https://img.shields.io/github/forks/SurakantiMeghana26/sql-data-warehouse-project?style=social)](https://github.com/SurakantiMeghana26/sql-data-warehouse-project)

</div>
