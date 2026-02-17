# Global Sales ELT Pipeline (Snowflake + Snowpark + GitHub Actions)

## Project Overview

This project demonstrates an **end-to-end ELT data pipeline** built on **Snowflake** using **Snowpark (Python)** for transformations and **GitHub Actions** for simple CI/CD-style orchestration.

The goal of the project is to show how heterogeneous sales datasets from multiple countries can be ingested, standardized, transformed, and curated into analytics-ready tables — while also demonstrating how automation fits into a real-world data engineering workflow.

This is intentionally designed as a **production-style pipeline**, not just a one-time script execution.

---

## Architecture

**Data Sources**

* India sales data (CSV)
* USA sales data (Parquet)
* UK sales data (CSV)

**Storage**

* Amazon S3 (external stage)

**Processing & Warehousing**

* Snowflake
* Snowpark (Python)

**Orchestration / CI-CD**

* GitHub Actions

---

## Data Warehouse Layering

The pipeline follows a standard **layered warehouse design**:

1. **STAGGING** (Landing layer)

   * Raw data loaded from S3
   * Minimal validation
   * Schema matches source systems

2. **RAW** (Cleaned & typed layer)

   * Data type normalization
   * Basic cleaning
   * Insert timestamps added

3. **TRANSFORMED** (Business-standardized layer)

   * Country-level standardization
   * Schema alignment across regions
   * Unified global sales table

4. **CURATED** (Analytics-ready layer)

   * Aggregated and business-facing tables
   * Optimized for reporting and dashboards

---

## Repository Structure

```
Global-Sales-ELT-Pipeline/
├── snowpark/
│   ├── raw_load.py
│   ├── transformed_load.py
│   └── curated_load.py
├── sql/
│   └── setup.sql
├── utils/
│   └── run_sql.py
├── .github/
│   └── workflows/
│       └── snowpark_pipeline.yml
└── README.md
```

---

## Core Components

### 1. setup.sql

Creates all required Snowflake objects:

* Database and schemas
* Staging tables
* External stage (S3)

This script is designed to be **idempotent** and can be safely re-run.

---

### 2. raw_load.py

Responsible for:

* Truncating staging tables
* Loading raw files from S3 using `COPY INTO`
* Cleaning and typing data
* Writing cleaned results into the **RAW** schema

This script represents the **Extract and Load** portion of the pipeline.

---

### 3. transformed_load.py

Responsible for:

* Standardizing schemas across countries
* Joining India order and order-detail datasets
* Unioning India, USA, and UK data
* Creating a single **GLOBAL_SALES_ORDER** table

This script represents the **Transform** portion of the pipeline.

---

### 4. curated_load.py

Creates analytics-ready tables including:

* Sales by country
* Category performance
* Monthly sales trends
* India sales vs targets
* Top products by revenue

These tables are designed for direct consumption by BI tools.

---

### 5. verify_pipeline.sql

Read-only validation script used to:

* Confirm row counts at each layer
* Inspect curated tables
* Validate pipeline correctness

This script is intentionally **not automated** and is meant for human inspection.

---

## GitHub Actions Orchestration

The pipeline is orchestrated using **GitHub Actions** to provide a lightweight CI/CD experience without requiring Airflow or other schedulers.

### Workflow Behavior

* Triggered on push to `main` or manual dispatch
* Executes steps in order:

  1. setup.sql
  2. raw_load.py
  3. transformed_load.py
  4. curated_load.py

### Purpose of Automation

* Ensures reproducibility
* Validates pipeline execution without manual intervention
* Provides execution logs and failure visibility
* Demonstrates production-readiness

Automation is **not used to display data**, only to confirm successful execution.

---

## How to Run the Project

### Option 1: Manual (Development / Debugging)

Run in Snowflake:

1. `setup.sql`
2. `raw_load.py`
3. `transformed_load.py`
4. `curated_load.py`
5. `verify_pipeline.sql`

---

### Option 2: Automated (CI/CD)

1. Push changes to the `main` branch
2. GitHub Actions automatically executes the pipeline
3. Verify results in Snowflake using `verify_pipeline.sql`

---

## How to Verify the Pipeline

Verification is intentionally separated from execution.

Example checks:

```sql
SELECT COUNT(*) FROM SNOWPARK_DB.RAW.INDIA_ORDERS;
SELECT COUNT(*) FROM SNOWPARK_DB.TRANSFORMED.GLOBAL_SALES_ORDER;
SELECT * FROM SNOWPARK_DB.CURATED.SALES_BY_COUNTRY;
```

---

## Key Takeaways

* Demonstrates ELT best practices
* Uses Snowpark for scalable transformations
* Applies layered warehouse design
* Shows practical CI/CD without Airflow
* Separates execution from verification

---

## Author Notes

This project is intentionally scoped to balance **clarity, correctness, and production realism**, making it suitable for interviews, demos, and learning modern data engineering workflows.
