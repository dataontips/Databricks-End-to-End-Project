# Azure Databricks End-to-End Data Engineering Project

## 📌 Overview

This repository showcases an **end-to-end Data Engineering project built on Azure Databricks**, created as part of my personal portfolio to demonstrate a real-world implementation of the **Medallion Architecture (Bronze, Silver, Gold)** using modern data engineering best practices.

The project covers **full cloud setup**, **streaming ingestion using Auto Loader**, **batch/stream processing**, **job orchestration**, and **analytical fact table creation**, and is intended to be used as a **personal portfolio project** to demonstrate my hands-on experience to recruiters and hiring managers.

---

## 🏗️ Architecture Overview

The solution follows a layered architecture based on the **Medallion Architecture** pattern:

**Source → Bronze → Silver → Gold → Fact Tables → Power BI**

* **Bronze Layer**: Raw ingestion from ADLS Gen2 using Databricks Auto Loader
* **Silver Layer**: Data cleansing, standardization, and business entity separation
* **Gold Layer**: Business-ready dimensions, facts, and SCD handling
* **Power BI**: Reporting and analytics layer (in progress)

### 🔁 Databricks Job Orchestration

The pipeline is orchestrated using **Databricks Jobs**, where each layer depends on the successful completion of the previous one.

![Databricks Pipeline Architecture](https://github.com/dataontips/Databricks-End-to-End-Project/blob/462676906cd612c576f80b4fef07b217c6388e44/Images/databricks_pipeline.png)

---

## 🔧 Technologies Used

* **Cloud Platform**: Microsoft Azure
* **Data Processing**: Azure Databricks (PySpark)
* **Storage**: Azure Data Lake Storage Gen2 (ADLS)
* **Ingestion**: Databricks Auto Loader
* **Architecture Pattern**: Medallion Architecture
* **Orchestration**: Databricks Jobs
* **Metastore**: Unity Catalog (Metastore & External Locations)
* **Version Control**: GitHub

---

## ☁️ Azure Resources Created

All resources were provisioned and configured manually as part of the project:

* Azure Resource Group
* Azure Storage Account (ADLS Gen2)

  * Containers for source, bronze, silver, and gold layers
* Azure Databricks Workspace
* Databricks Access Connector
* Unity Catalog Metastore
* External Locations & Storage Credentials

---

## 📂 Project Structure

> The data files included in this repository are **sample datasets** used for demonstration and portfolio purposes only.

````text
Databricks-ETE-Project/
│
├── data/                  # Sample source data (Parquet)
│   ├── customer_first.parquet
│   ├── customers_second.parquet
│   ├── orders_first.parquet
│   ├── orders_second.parquet
│   ├── products_first.parquet
│   ├── products_second.parquet
│   └── regions.parquet
│
├── notebooks/             # Databricks notebooks (.dbc export)
│   └── Databricks_ETE_Project.dbc
│
├── images/                # Architecture & report screenshots
│   ├── databricks_pipeline.png
│   └── powerbi_report.png
│
├── power_bi/              # Power BI report
│   └── sales_analytics.pbix   # (In Progress)
│
└── README.md

Databricks-ETE-Project/
│
├── bronze_layer/
│   └── bronze_autoloader.dbc
│
├── silver_layer/
│   ├── silver_customers.dbc
│   ├── silver_orders.dbc
│   └── silver_products.dbc
│
├── gold_layer/
│   ├── gold_customers.dbc
│   ├── gold_products_dlt.dbc
│   └── fact_orders.dbc
│
├── power_bi/
│   └── sales_analytics.pbix   # (In Progress)
│
├── jobs/
│   └── job_orchestration_diagram.png
│
└── README.md
````

---

## 🔄 Data Pipeline Flow

---

## 🧬 Slowly Changing Dimensions (SCD)

This project implements **both SCD Type 1 and SCD Type 2** patterns in the **Gold layer**, demonstrating how dimensional data is handled in real-world data warehouses.

### 🔹 SCD Type 1 – Overwrite History (Customers Dimension)

**Use case:** When historical changes are *not required* (e.g., correcting customer attributes).

**Implemented in:** `gold_customers.py`

**Approach:**

* Source data is read from the **Silver Customers** table
* Records are de-duplicated on `customer_id`
* Existing vs new records are identified using a **left join** against the Gold dimension table
* A **surrogate key (`DimCustomerKey`)** is generated for new records
* `create_date` and `update_date` are maintained
* Data is merged into the Gold table using **Delta Lake MERGE**

**Key Logic:**

* Existing records → updated in place
* New records → inserted
* No historical versions are preserved

This logic ensures the **latest version of each customer** is always available in the Gold layer.

---

### 🔹 SCD Type 2 – Preserve History (Products Dimension)

**Use case:** When historical changes *must be tracked* (e.g., product price/category changes).

**Implemented in:** `gold_products.py` using **Delta Live Tables (DLT)**

**Approach:**

* Streaming data is read from the **Silver Products** table
* Data quality rules are enforced using **DLT expectations**
* A staging table and streaming view are created
* Changes are applied using `dlt.apply_changes()` with `stored_as_scd_type = 2`

**Key Features:**

* Maintains full history of product changes
* Automatically manages:

  * `effective_start_date`
  * `effective_end_date`
  * `is_current` flag
* Each change creates a **new version** of the record

This approach demonstrates a **production-grade SCD Type 2 implementation** using Databricks declarative pipelines.

---

## 🔄 Data Pipeline Flow

### 1️⃣ Bronze Layer – Raw Ingestion

* Uses **Databricks Auto Loader** (`cloudFiles`)
* Ingests Parquet files from ADLS source container
* Supports schema evolution
* Checkpointing enabled for fault tolerance
* Parameterized using Databricks widgets

### 2️⃣ Silver Layer – Data Transformation

* Data cleansing and normalization
* Type casting and column standardization
* Business-level entity separation:

  * Customers
  * Orders
  * Products

### 3️⃣ Gold Layer – Business Aggregates

* Curated, analytics-ready tables
* Gold tables created using standard Spark jobs and DLT
* Optimized for downstream consumption

### 4️⃣ Fact Tables

* Creation of `fact_orders` table
* Designed for BI and analytical use cases

---

## 🧩 Orchestration Strategy

* Implemented using **Databricks Jobs**
* Job dependencies reflect Medallion flow
* Separate jobs for Bronze, Silver, and Gold layers
* Supports reruns and failure recovery via checkpoints

---

## 🔐 Governance & Security

* Unity Catalog enabled
* External locations configured for ADLS access
* Centralized metastore for table governance
* Separation of storage and compute

---

## 📊 Power BI Reporting (In Progress)

The Power BI report connects to curated **Gold layer tables** created in Databricks and is intended to support business reporting and analysis use cases.

![Power BI Report](images/powerbi_report.png)

---

## 🚀 Key Highlights

* End-to-end Azure Databricks implementation
* Production-style Medallion Architecture
* Streaming + batch hybrid processing
* Parameterized and reusable pipelines
* Job-based orchestration
* Portfolio-ready project structure

---

## 📈 Possible Enhancements

* Add data quality checks using Delta Live Tables expectations
* CI/CD integration using GitHub Actions
* Monitoring and alerting via Azure Monitor
* Power BI dashboard connected to Gold layer

---

## 👤 Author

**Navjeet Singh**
Data Engineer | Azure | Databricks

This repository is part of my personal portfolio to showcase practical, hands-on experience with Azure Databricks and modern data engineering workflows. If you find it useful, feel free to ⭐ the repository or connect with me.

---

## 📜 License

This project is for learning and portfolio purposes. You are free to fork and adapt it with attribution.
