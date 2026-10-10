# Module 02 — Azure Databricks Retail Lakehouse

## 1. Module Purpose

This module explains how to build a retail data lakehouse using Azure Databricks, Azure Data Lake Storage Gen2, Apache Spark, PySpark, Delta Lake, and Unity Catalog.

The objective is to understand not only how to execute notebooks, but also why each technology is used and how the complete data engineering architecture works.

This guide documents the existing implementation and provides a structured path for learning, revision, interviews, and teaching.

## 2. Business Scenario

A retail company maintains customer, product, order, and store information in CSV files.

The company wants to transform these raw files into clean, reliable, business-ready datasets for reporting and analytics.

The solution should:

- Ingest source files from cloud storage.
- Maintain a raw data layer.
- Clean and standardize data.
- Handle incremental arrivals.
- Validate data quality.
- Create business-level datasets.
- Control access to data.
- Schedule and monitor processing.

## 3. Architecture

```text id="8p9k8v"
Retail CSV Files
       |
       v
Azure Data Lake Storage Gen2
       |
       v
Bronze Layer — Raw Ingested Data
       |
       v
Silver Layer — Cleaned and Standardized Data
       |
       v
Gold Layer — Business-Ready Data
       |
       v
Databricks SQL / Analytics
```

Supporting services:

- Azure Databricks runs PySpark transformations.
- Unity Catalog manages data objects, access, and governance.
- Delta Lake provides transactional table storage and history.
- Databricks Jobs orchestrate notebooks.
- Monitoring helps investigate failures and performance.

## 4. Azure Resources Used

| Resource | Name | Purpose |
|---|---|---|
| Resource group | `rg-databricks-retail-dev` | Organizes the Databricks project resources |
| ADLS Gen2 storage account | `stdatabricksretail2026` | Stores source files and lakehouse data |
| Databricks workspace | `dbw-retail-lakehouse-dev` | Provides the environment for notebooks and SQL |
| Access Connector | `ac-databricks-retail-dev` | Provides a managed identity for storage access |
| Storage credential | `cred_adls_retail` | Represents the identity Databricks uses to access storage |
| External location | `ext_retail_source` | Associates a storage path with a Unity Catalog credential |
| Catalog | `retail` | Organizes project data objects |
| Schemas | `bronze`, `silver`, `gold` | Separate data objects by processing layer |

The storage account was placed in a different resource group from the Databricks workspace. This is allowed in Azure; resources can communicate across resource groups when access and configuration are correct.

## 5. Technical Concepts — What and Why

### 5.1 Azure Databricks

Azure Databricks is a managed analytics platform built around Apache Spark.

It provides an environment for processing large datasets, running distributed transformations, working with Delta tables, and building data engineering pipelines.

**Why it is used:** PySpark transformations can process datasets at scale without requiring developers to manage an entire Spark cluster themselves.

### 5.2 Apache Spark

Apache Spark is a distributed data processing engine.

Instead of relying only on one machine, Spark can distribute processing across multiple workers.

Important concepts:

- **Driver:** Coordinates a Spark application and its execution.
- **Executors:** Run tasks and process data.
- **Transformations:** Define operations such as filtering and joining.
- **Actions:** Trigger execution and return results or write output.

Spark uses lazy evaluation for many transformations. Operations such as `filter()` may build a plan without immediately processing all data. An action or write can trigger execution.

### 5.3 PySpark

PySpark is the Python API for Apache Spark.

Examples:

```python
df = spark.read.option("header", True).csv(source_path)

clean_df = df.dropDuplicates(["customer_id"])

clean_df.display()
```

The first statement reads CSV data into a Spark DataFrame. The second removes duplicate customer IDs according to the selected key. The third displays results in the Databricks notebook environment.

**Why it is used:** It lets data engineers express distributed data transformations using Python.

### 5.4 ADLS Gen2

Azure Data Lake Storage Gen2 provides cloud storage for files and analytics data.

The project uses separate containers:

- `source`: original CSV files.
- `bronze`: ingested Delta data.
- `silver`: cleaned Delta data.
- `gold`: business-ready Delta data.

This separation makes it easier to understand data lineage and control how data moves through the platform.

### 5.5 Delta Lake

Delta Lake is a storage layer that adds transaction support and table-management capabilities to data stored in supported file systems.

Key features include:

- ACID transactions.
- Schema enforcement.
- Controlled schema evolution.
- Table history.
- Time travel and supported restore operations.

**Why it is used:** Plain Parquet files do not independently provide all the transaction and table-history capabilities required for reliable concurrent table operations.

## 6. Medallion Architecture

### 6.1 Bronze layer

**Purpose:** Preserve ingested source data with minimal transformation.

The Bronze notebook is `01_Bronze_Ingestion`.

It reads the source CSV files and writes Delta data for:

- Customers
- Products
- Orders
- Stores

Example:

```python
df = (
    spark.read
         .option("header", True)
         .option("inferSchema", True)
         .csv(source_path)
)

df.write.format("delta").mode("overwrite").save(target_path)
```

This is an illustrative batch-ingestion pattern. The actual notebook configuration should be consulted before reproducing the project.

**Why Bronze matters:** It provides a traceable ingestion layer that can be used to investigate source issues and rebuild downstream transformations.

### 6.2 Silver layer

**Purpose:** Clean and standardize the ingested data.

Notebook: `02_Silver_Transformation`

Typical operations:

- Trim text fields.
- Standardize selected values.
- Cast columns to appropriate data types.
- Remove duplicates according to business keys.
- Write cleaned Delta tables.

Example:

```python
from pyspark.sql.functions import col, trim, upper

clean_customers = (
    bronze_customers
    .withColumn("customer_id", trim(col("customer_id")))
    .withColumn(
        "customer_segment",
        upper(trim(col("customer_segment")))
    )
    .dropDuplicates(["customer_id"])
)
```

**Why Silver matters:** Downstream reports should not need to repeatedly handle inconsistent text, duplicate business keys, or incorrectly typed values.

Data cleaning must be based on business rules. For example, dropping duplicates is safe only when the chosen key and duplicate-selection behavior are appropriate.

### 6.3 Gold layer

**Purpose:** Produce business-ready datasets for analytics.

Notebook: `03_Gold_Business_Layer`

The project created:

`retail.gold.gold_sales`

The table combines order, customer, product, and store information into a dataset suitable for sales analysis.

Example query:

```sql
SELECT COUNT(*) AS row_count
FROM retail.gold.gold_sales;
```

The project validated a count of 25 rows during testing.

**Why Gold matters:** Business users can work with meaningful, integrated datasets instead of repeatedly joining and cleaning raw source tables.

## 7. Unity Catalog and Data Governance

Unity Catalog provides centralized governance for supported Databricks data objects and access patterns.

### 7.1 Catalog, schema, and table

The project uses this hierarchy:

```text id="a2a4s8"
retail
├── bronze
│   ├── customers
│   ├── products
│   ├── orders
│   └── stores
├── silver
│   ├── customers
│   ├── products
│   ├── orders
│   └── stores
└── gold
    └── gold_sales
```

- **Catalog:** Top-level logical container.
- **Schema:** Groups related tables and views.
- **Table:** A named data object containing structured records.

A fully qualified table name follows this pattern:

`catalog.schema.table`

Example:

`retail.gold.gold_sales`

### 7.2 Managed and external tables

A managed table has its storage lifecycle managed by Databricks and Unity Catalog within the configured managed storage arrangement.

An external table references data stored at an externally managed location. Dropping the table generally removes its catalog registration but does not automatically delete the underlying external files.

Always check the table type and storage location before dropping a table.

### 7.3 Access Connector and managed identity

An Access Connector provides a managed identity that Databricks can use to access Azure resources.

The storage credential associates the identity with Unity Catalog. The external location associates the credential with a particular storage URL.

Access must be granted through the appropriate Azure role assignments and Unity Catalog permissions.

**Why this matters:** Managed identity avoids placing storage account keys directly in notebook code. It does not automatically grant every required permission; the identity still needs appropriate authorization.

## 8. Incremental Ingestion with Auto Loader

Notebook: `04_AutoLoader_Ingestion`

Auto Loader incrementally discovers and processes new files arriving in supported cloud storage locations.

It is built around Spark Structured Streaming infrastructure and can maintain state about files already processed.

Example pattern:

```python
stream_df = (
    spark.readStream
         .format("cloudFiles")
         .option("cloudFiles.format", "parquet")
         .load(source_path)
)
```

The exact source format, schema configuration, checkpoint path, and write mode must match the actual implementation.

**Why Auto Loader is useful:**

- Handles continuing arrivals of files.
- Supports scalable file discovery.
- Can use checkpoint state to track progress.
- Avoids repeatedly treating every historical file as new.

In this project, the incremental orders data was processed and the Bronze Auto Loader table was validated at 25 rows.

**Important:** A streaming query must be configured with appropriate checkpointing and a suitable trigger or continuous execution strategy. Checkpoint state and output behavior must be considered when restarting or recovering a job.

## 9. Data Quality

Notebook: `05_Data_Quality`

Data quality checks help establish whether datasets satisfy expected rules.

Examples:

- Required keys are not null.
- Business keys meet uniqueness expectations.
- Numeric fields have valid data types.
- Orders reference valid customers and products.
- Quantities and prices meet business constraints.
- Record counts are checked after transformations.

Example:

```python
from pyspark.sql.functions import col

invalid_orders = silver_orders.filter(
    col("order_id").isNull()
    | col("customer_id").isNull()
    | col("quantity").isNull()
)
```

This example identifies records with missing required fields. A complete quality framework should also define whether invalid records are rejected, quarantined, corrected, or reported.

A filtered validation count should not be confused with the total number of rows in the source table.

## 10. Delta Lake History and Restore

The project explored table history and restoration.

Inspect history:

```sql
DESCRIBE HISTORY retail.gold.gold_sales;
```

Read a historical version:

```sql
SELECT *
FROM retail.gold.gold_sales VERSION AS OF 4;
```

Restore a table:

```sql
RESTORE TABLE retail.gold.gold_sales
TO VERSION AS OF 4;
```

A restore changes the current table state to a previous version. It should be used carefully because downstream consumers may depend on the current version.

**Why these features matter:** History helps investigate changes and diagnose unexpected results. Time travel supports controlled historical reads, subject to retention and file availability.

## 11. Jobs, Scheduling, and Monitoring

The project created a scheduled Databricks Job named:

`JOB_Retail_Lakehouse_ETL`

The job includes notebook tasks for Bronze ingestion, Auto Loader ingestion, Silver transformation, Gold business processing, and data quality.

A job provides orchestration around notebook execution.

Monitoring should include:

- Run status and duration.
- Task dependencies and execution order.
- Error messages and failed tasks.
- Input and output counts.
- Checkpoint and retry behavior.
- Cost and compute utilization.

The configured schedule is environment-specific. Before running a scheduled job, verify that the tasks are in the intended order and that downstream tasks do not start before required upstream data is ready.

## 12. End-to-End Data Flow

```text id="z6e6e2"
1. Read source files
       |
       v
2. Bronze ingestion
       |
       v
3. Incremental orders ingestion
       |
       v
4. Silver transformations
       |
       v
5. Data quality validation
       |
       v
6. Gold business dataset
       |
       v
7. SQL validation and analytics
       |
       v
8. Job monitoring and history
```

This diagram describes the logical processing stages. In an actual scheduled job, task dependencies must be configured to enforce the intended execution order.

## 13. Validation Results from the Project

The project observed the following counts during validation:

| Dataset | Validated count |
|---|---:|
| Customers | 10 |
| Products | 10 |
| Orders in the Auto Loader Bronze dataset | 25 |
| Stores | 7 |
| Gold sales table | 25 |

These counts describe the tested sample dataset, not general production expectations.

Additional validation included:

- Delta table reads and writes.
- Silver transformation checks.
- Data quality checks.
- Gold row-count validation.
- Table history inspection.
- A deliberate table update followed by a restore.
- Successful scheduled job runs.

## 14. Common Troubleshooting

### Cannot read files from ADLS

Check the container and path, storage credential, external location, managed identity role assignments, and Unity Catalog permissions.

### Unity Catalog table cannot be found

Verify the catalog, schema, table name, current workspace, and the permissions of the user or service principal.

### Schema mismatch during a write

Inspect the source schema and existing Delta table schema. Use schema evolution or overwrite-schema options only when the change is intentional and understood.

### Auto Loader does not pick up files

Check the source path, file format, checkpoint location, schema configuration, trigger configuration, and whether the file has already been processed.

### Duplicate records appear

Check the business key, source duplication, streaming checkpoint, write semantics, and whether the pipeline is safe to retry.

### Job fails

Inspect the failed task's error details, permissions, source availability, compute configuration, and notebook parameters.

## 15. Security and Cost Awareness

- Grant least-privilege permissions to users and service identities.
- Prefer managed identity over embedded storage keys.
- Avoid storing credentials in notebooks or GitHub.
- Keep source and output paths explicit.
- Use suitable compute for the workload.
- Review job schedules and stop unnecessary interactive compute.
- Check cloud resource costs and clean up resources that are no longer required.
- Confirm retention and recovery requirements before deleting or restoring data.

## 16. Reproduction Checklist

- [ ] Explain the difference between a data lake and a lakehouse.
- [ ] Explain Spark, PySpark, DataFrames, transformations, and actions.
- [ ] Read source CSV files from ADLS Gen2.
- [ ] Write and validate Bronze Delta data.
- [ ] Explain Auto Loader and checkpointing.
- [ ] Clean and standardize Silver data.
- [ ] Apply data quality rules.
- [ ] Build a Gold business dataset.
- [ ] Explain catalogs, schemas, tables, credentials, and external locations.
- [ ] Inspect Delta history and perform a safe restore exercise.
- [ ] Schedule a Databricks Job and monitor its runs.
- [ ] Explain permissions, data lineage, and cost controls.

## 17. Interview Questions

1. What is Azure Databricks, and why is Apache Spark used?
2. What is the difference between a DataFrame transformation and an action?
3. What is Delta Lake, and how does it differ from plain Parquet storage?
4. Explain the Bronze, Silver, and Gold layers.
5. What is Unity Catalog?
6. Explain the difference between a catalog, schema, and table.
7. What are a storage credential and an external location?
8. How does managed identity work with ADLS Gen2?
9. How does Auto Loader discover and process new files?
10. Why are checkpoints important in incremental ingestion?
11. How do you design data quality checks?
12. What is Delta Lake time travel?
13. What is the difference between a managed table and an external table?
14. How do you orchestrate and monitor notebook tasks?
15. How would you make a pipeline safe to retry?

## 18. Teaching Exercise for BTech Freshers

Use the four source files: customers, products, orders, and stores.

Ask learners to:

1. Draw the architecture.
2. Explain why raw data is retained.
3. Read one CSV file with PySpark.
4. Identify duplicate and null keys.
5. Write a cleaned Silver table.
6. Join orders with customers and products.
7. Create a simple Gold sales dataset.
8. Explain how Auto Loader handles new files.
9. Describe how Unity Catalog permissions protect data.
10. Present the solution and explain its limitations.

## 19. Learning Outcome

After studying and reproducing this module, learners should be able to explain and implement a foundational cloud lakehouse pipeline using Azure Databricks, PySpark, Delta Lake, Unity Catalog, incremental ingestion, data quality checks, and scheduled jobs.

This guide records the project's demonstrated capabilities. Before calling the solution production-ready, additionally verify recovery behavior, data-quality enforcement, idempotency, concurrency, observability, access controls, and cost management.
