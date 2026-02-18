Global Sales ELT Pipeline (Snowflake + Snowpark + GitHub Actions + Power BI)
Project Overview

This project demonstrates an end-to-end ELT data pipeline built on Snowflake using Snowpark (Python) for transformations, GitHub Actions for CI/CD-style orchestration, and Power BI for business intelligence and analytics.

The pipeline ingests heterogeneous sales datasets from multiple countries, standardizes and transforms them into analytics-ready models, and exposes curated datasets directly to Power BI dashboards for decision-making.

This project intentionally mirrors a production-style data platform, covering ingestion, transformation, automation, and visualization — rather than a one-time script execution.

Architecture

Data Sources

India sales data (CSV)

USA sales data (Parquet)

UK sales data (CSV)

Storage

Amazon S3 (external stage)

Processing & Warehousing

Snowflake

Snowpark (Python)

Orchestration / CI-CD

GitHub Actions

Analytics & Visualization

Power BI (connected directly to curated Snowflake tables)

Data Warehouse Layering

The pipeline follows a standard layered warehouse design:

STAGGING (Landing layer)

Raw data loaded from S3

Minimal validation

Schema matches source systems

RAW (Cleaned & typed layer)

Data type normalization

Basic cleaning

Insert timestamps added

TRANSFORMED (Business-standardized layer)

Country-level standardization

Schema alignment across regions

Unified global sales table

CURATED (Analytics-ready layer)

Aggregated and business-facing tables

Optimized for reporting and dashboards

Directly consumed by Power BI

Repository Structure
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

Core Components
1. setup.sql

Creates all required Snowflake objects:

Database and schemas

Staging tables

External stage (S3)

This script is designed to be idempotent and can be safely re-run.

2. raw_load.py

Responsible for:

Truncating staging tables

Loading raw files from S3 using COPY INTO

Cleaning and typing data

Writing cleaned results into the RAW schema

This script represents the Extract and Load portion of the pipeline.

3. transformed_load.py

Responsible for:

Standardizing schemas across countries

Joining India order and order-detail datasets

Unioning India, USA, and UK data

Creating a single GLOBAL_SALES_ORDER table

This script represents the Transform portion of the pipeline.

4. curated_load.py

Creates analytics-ready tables including:

Sales by country

Category performance

Monthly sales trends

India sales vs targets

Top products by revenue

These tables are designed for direct consumption by BI tools.

5. verify_pipeline.sql

Read-only validation script used to:

Confirm row counts at each layer

Inspect curated tables

Validate pipeline correctness

This script is intentionally not automated and is meant for human inspection.

Power BI Integration

Power BI is used as the analytics and reporting layer, connected directly to the CURATED schema in Snowflake.

Power BI Data Model

Power BI connects to the following curated tables:

SALES_BY_COUNTRY

MONTHLY_SALES_TREND

CATEGORY_PERFORMANCE

TOP_PRODUCTS_BY_REVENUE

INDIA_SALES_VS_TARGET

These tables are modeled as read-only semantic layers, ensuring that all business logic remains inside Snowflake.

Power BI Dashboards

The Power BI report includes:

Global Sales KPIs

Total Sales

Total Quantity

Profit Margin

Sales Performance Analysis

Sales by Country

Monthly Sales Trend by Country

Sales by Category

Target vs Actual Analysis (India)

Actual vs Target Sales Line Chart

Variance Heatmap by Category

KPI indicators for goal tracking

Distribution & Variability Analysis

Box Plot showing variance distribution across categories

These dashboards validate that the ELT pipeline produces business-consumable outputs, not just transformed tables.

Power BI Refresh Strategy

GitHub Actions runs the ELT pipeline

Snowflake tables are updated

Power BI refresh pulls latest curated data from Snowflake

This mirrors a real-world warehouse → BI workflow.

GitHub Actions Orchestration

The pipeline is orchestrated using GitHub Actions to provide a lightweight CI/CD experience without requiring Airflow or other schedulers.

Workflow Behavior

Triggered on push to main or manual dispatch

Executes steps in order:

setup.sql

raw_load.py

transformed_load.py

curated_load.py

Purpose of Automation

Ensures reproducibility

Validates pipeline execution without manual intervention

Provides execution logs and failure visibility

Demonstrates production-readiness

Automation is not used to display data, only to confirm successful execution.

How to Run the Project
Option 1: Manual (Development / Debugging)

Run in Snowflake:

setup.sql

raw_load.py

transformed_load.py

curated_load.py

verify_pipeline.sql

Then refresh Power BI to view updated dashboards.

Option 2: Automated (CI/CD)

Push changes to the main branch

GitHub Actions automatically executes the pipeline

Verify results in Snowflake using verify_pipeline.sql

Refresh Power BI dashboards

How to Verify the Pipeline

Verification is intentionally separated from execution.

Example checks:

SELECT COUNT(*) FROM SNOWPARK_DB.RAW.INDIA_ORDERS;
SELECT COUNT(*) FROM SNOWPARK_DB.TRANSFORMED.GLOBAL_SALES_ORDER;
SELECT * FROM SNOWPARK_DB.CURATED.SALES_BY_COUNTRY;


Visual verification is done via Power BI dashboards.

Key Takeaways

Demonstrates ELT best practices

Uses Snowpark for scalable transformations

Applies layered warehouse design

Shows practical CI/CD without Airflow

Integrates Snowflake directly with Power BI

Separates execution, validation, and visualization concerns

Author Notes

This project is intentionally scoped to balance clarity, correctness, and production realism, making it suitable for interviews, demos, and learning modern data engineering workflows.
