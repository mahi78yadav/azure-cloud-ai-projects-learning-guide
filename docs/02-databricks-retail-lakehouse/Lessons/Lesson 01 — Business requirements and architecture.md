# Lesson 01 — Business Requirements and Architecture

## 1. Learning Objectives

By the end of this lesson, you should be able to:

- Explain the business problem addressed by the Retail Lakehouse.
- Describe the solution architecture using Azure Databricks and ADLS Gen2.
- Explain the Bronze, Silver, and Gold data layers.
- Identify the main Azure resources and Databricks components.
- Describe the flow of data from source files to business-ready tables.
- Explain the role of Unity Catalog, Delta Lake, Auto Loader, and Databricks Jobs.
- Present the project clearly in an interview or architecture discussion.

## 2. Business Scenario

A retail company collects data from several business areas:

- Customers
- Products
- Orders
- Stores

The data arrives as files and must be transformed into a reliable format for analytics and reporting.

The business needs a data platform that can:

1. Ingest source data into a central storage layer.
2. Preserve the original data for traceability.
3. Clean and standardize the data.
4. Create business-ready tables.
5. Support incremental file ingestion.
6. Apply data quality checks.
7. Provide governed access to data.
8. Run processing jobs on a schedule.
9. Support troubleshooting and recovery.

Azure Databricks provides the data processing engine, while Azure Data Lake Storage Gen2 provides cloud storage.

The solution follows the Lakehouse architecture, combining data lake storage with managed tables, transactional capabilities, governance, and analytical processing.

## 3. Project Objectives

The main objective is to implement an end-to-end retail data engineering workflow using Azure Databricks.

| Objective | Implementation |
|---|---|
| Ingest source files | Databricks notebooks |
| Store raw data | Bronze layer |
| Clean and transform data | Silver layer |
| Create business-ready datasets | Gold layer |
| Support incremental ingestion | Auto Loader notebook |
| Validate data quality | Data quality notebook |
| Manage tables and permissions | Unity Catalog |
| Support reliable table operations | Delta Lake |
| Automate processing | Databricks Job |
| Validate results | SQL queries and notebook checks |

These objectives describe the project's scope. Each capability should be demonstrated using its corresponding notebook, table, or configuration.

## 4. Solution Architecture

<escape>
Source CSV files
       |
       v
Azure Data Lake Storage Gen2
       |
       v
Azure Databricks Workspace
       |
       v
Bronze Layer — Raw Ingestion
       |
       v
Silver Layer — Cleansing and Transformation
       |
       v
Gold Layer — Business-Ready Tables
       |
       v
SQL Analytics and Reporting
</escape>

Cross-cutting capabilities include:

- **Unity Catalog:** Catalog, schema, table, and permission management.
- **Delta Lake:** Transactional table storage, history, and supported recovery operations.
- **Auto Loader:** Incremental file discovery and ingestion.
- **Databricks Jobs:** Scheduled notebook execution.
- **Data quality checks:** Validation of data before it is used downstream.

The architecture separates raw ingestion, transformation, and business consumption so each stage has a clear responsibility.

## 5. Azure Resources

The project uses the following resources.

| Resource | Name |
|---|---|
| Resource group | `rg-databricks-retail-dev` |
| ADLS Gen2 storage account | `stdatabricksretail2026` |
| Databricks workspace | `dbw-retail-lakehouse-dev` |
| Databricks Access Connector | `ac-databricks-retail-dev` |
| Storage credential | `cred_adls_retail` |
| External location | `ext_retail_source` |
| Unity Catalog catalog | `retail` |
| Bronze schema | `bronze` |
| Silver schema | `silver` |
| Gold schema | `gold` |
| Databricks Job | `JOB_Retail_Lakehouse_ETL` |

**Resource-group note:** The storage account is in a separate resource group named `adf-ecom-dev`. This is valid. Azure resources do not need to be in the same resource group to work together, provided that their configuration, network access, and permissions are correct.

The Databricks workspace and the storage account are separate resources. The workspace provides the compute and development environment, while ADLS Gen2 stores the lake data.

## 6. Source Data

The project uses four retail datasets.

### Customers

Contains customer details.

Typical fields include:

- Customer ID
- Customer name
- Email
- City
- State

### Products

Contains product information.

Typical fields include:

- Product ID
- Product name
- Category
- Price

### Orders

Contains transaction information.

Typical fields include:

- Order ID
- Customer ID
- Product ID
- Quantity
- Total amount
- Order date

### Stores

Contains store information.

Typical fields include:

- Store ID
- Store name
- City
- State

The source data is used to demonstrate file ingestion, transformation, data quality, and analytical modelling. Use the actual source CSV schemas when recreating the notebooks rather than assuming the example fields are identical.

## 7. Understand the Bronze, Silver, and Gold Layers

The project follows the Medallion Architecture.

### Bronze Layer — Raw Ingestion

The Bronze layer stores data close to its source representation.

Purpose:

- Ingest source data.
- Preserve source values for traceability.
- Establish a starting point for downstream transformations.
- Support troubleshooting when source data is malformed or unexpected.

In this project, Bronze tables are organized under the `retail.bronze` schema.

### Silver Layer — Cleansing and Transformation

The Silver layer contains cleaned and standardized data.

Typical transformations include:

- Checking for null values.
- Standardizing column names and data types.
- Validating business keys.
- Removing or identifying duplicate records.
- Applying data quality rules.
- Preparing data for downstream joins.

Silver tables are organized under `retail.silver`.

The exact transformations depend on the implemented notebook logic.

### Gold Layer — Business-Ready Data

The Gold layer contains data shaped for business analysis.

Typical uses include:

- Sales analysis.
- Product performance.
- Customer purchase analysis.
- Store performance.
- Aggregated business metrics.

The project includes the `retail.gold.gold_sales` table.

Gold tables are organized under `retail.gold`.

The layers can be summarized as follows:

| Layer | Primary purpose | Main audience |
|---|---|---|
| Bronze | Raw ingestion and traceability | Data engineers |
| Silver | Cleansing and standardization | Data engineers and analysts |
| Gold | Business-ready data | Analysts and reporting consumers |

## 8. Understand Unity Catalog

Unity Catalog is the governance layer used to organize and control access to data assets.

The project uses this hierarchy:

<escape>
Catalog: retail
    |
    +-- Schema: bronze
    |      +-- Bronze tables
    |
    +-- Schema: silver
    |      +-- Silver tables
    |
    +-- Schema: gold
           +-- Gold tables
</escape>

A catalog groups related schemas. A schema groups tables and other supported data objects.

The project also uses a storage credential and an external location to configure governed access to cloud storage.

| Component | Purpose |
|---|---|
| Catalog | Top-level logical organization |
| Schema | Groups related tables |
| Table | Stores structured data |
| Storage credential | Defines the identity used for supported storage access |
| External location | Associates a cloud storage path with a storage credential |
| Permissions | Control which users or groups can access data objects |

The storage credential is named `cred_adls_retail`, and the external location is named `ext_retail_source`.

Do not confuse an external location with a catalog or schema. They serve different purposes.

## 9. Understand Delta Lake

Delta Lake provides transactional table capabilities on top of data lake storage.

Relevant capabilities include:

- Transactional writes.
- Schema enforcement and supported schema evolution.
- Table history.
- Time travel within the available retention and configuration.
- Supported recovery operations.

The project demonstrates Delta Lake history and time travel or restore operations.

For example, a table history query can be run in a Databricks SQL cell:

```sql
DESCRIBE HISTORY retail.gold.gold_sales;
```

The output displays available historical table operations.

Time travel and restore are subject to the table's history, retained data files, permissions, and platform configuration. They should not be treated as unlimited backups.

## 10. Understand Auto Loader

Auto Loader incrementally discovers and ingests new files into Databricks.

The project includes the notebook:

`04_AutoLoader_Ingestion.py`

Auto Loader is useful when source files arrive continuously or periodically and the pipeline needs to process newly discovered files without repeatedly listing and reprocessing the entire source.

Important concepts include:

- Source directory.
- File format.
- Schema inference or schema location.
- Checkpoint location.
- Streaming query.
- Incremental file discovery.

A checkpoint tracks streaming progress and state. It is different from the business watermark used in some batch pipelines.

The presence of an Auto Loader notebook does not mean that every Bronze, Silver, and Gold notebook uses Auto Loader. In this project, Auto Loader is demonstrated in a separate notebook.

## 11. Understand the Notebooks

The project contains five exported notebooks.

| Notebook | Purpose |
|---|---|
| `01_Bronze_Ingestion.py` | Ingest source data into Bronze |
| `02_Silver_Transformation.py` | Clean and transform data into Silver |
| `03_Gold_Business_Layer.py` | Build business-ready Gold tables |
| `04_AutoLoader_Ingestion.py` | Demonstrate incremental file ingestion |
| `05_Data_Quality.py` | Demonstrate data quality validation |

The intended workflow is to ingest data, transform it, build business-ready tables, and validate the result.

Inspect the notebook code when reproducing the project. The table names, column mappings, transformation rules, and dependencies should match the actual implementation.

## 12. Understand Databricks Jobs

The project uses the job:

`JOB_Retail_Lakehouse_ETL`

The job automates notebook execution.

Its recorded job ID is:

`219655900077611`

The job was configured with a daily schedule at 6:00 AM in the UTC+05:30 time zone, and successful runs were observed.

A job can orchestrate one or more tasks, depending on its configuration. To understand the actual workflow, inspect the job's task definitions, task dependencies, compute settings, and schedule.

A scheduled job is useful because it reduces the need for manual notebook execution and creates an execution history for monitoring.

## 13. End-to-End Data Flow

The high-level data flow is:

1. Source CSV files are made available in the configured storage location.
2. Databricks reads the source data.
3. The Bronze notebook ingests raw data into Bronze tables.
4. The Silver notebook cleans and transforms the data.
5. The Gold notebook creates business-ready tables.
6. The data quality notebook validates selected data quality rules.
7. The Auto Loader notebook demonstrates incremental file ingestion.
8. The Databricks Job automates its configured tasks.
9. SQL queries and table history are used to validate the results.

**Implementation note:** These steps describe the project capabilities. The exact task order executed by `JOB_Retail_Lakehouse_ETL` must be verified in the job configuration; do not assume that every notebook is included in the scheduled job.

## 14. Validation Evidence

The project has recorded the following implementation results:

| Validation item | Recorded result |
|---|---|
| Bronze, Silver, and Gold layers | Implemented |
| Source datasets | Customers, products, orders, stores |
| Auto Loader orders ingestion | 25 records observed |
| Gold sales table | `retail.gold.gold_sales` |
| Gold sales records | 25 records observed |
| Delta Lake history and recovery | Demonstrated |
| Databricks Job | `JOB_Retail_Lakehouse_ETL` |
| Job execution | Successful runs observed |

These are recorded project results, not a substitute for rerunning the checks after changes to the environment or code.

For example, the Gold table can be inspected using:

```sql
SELECT *
FROM retail.gold.gold_sales;
```

To validate its current row count:

```sql
SELECT COUNT(*) AS row_count
FROM retail.gold.gold_sales;
```

The expected count of 25 applies to the previously validated project state. If the source data or transformation logic has changed, the current result may differ.

## 15. Security and Governance Considerations

The architecture should follow least-privilege access principles.

Recommended checks include:

- Verify the permissions assigned to the Databricks Access Connector.
- Confirm that the storage credential references the intended identity.
- Review the external location's URL and permissions.
- Grant catalog and schema permissions only to appropriate users or groups.
- Avoid embedding storage credentials or secrets in notebooks.
- Use managed identities and supported identity-based access where appropriate.
- Review job permissions and access to output tables.

These are validation and design considerations. Document individual controls as implemented only after verifying the current configuration.

## 16. Scope and Limitations

This project demonstrates an Azure Databricks Lakehouse workflow. It is separate from the Azure Data Factory project and the Microsoft Foundry AI Assistant project.

The three projects are not one fully integrated production pipeline.

The Databricks project demonstrates:

- A Medallion architecture.
- Governed table organization using Unity Catalog.
- Data transformations using Spark and PySpark.
- Delta Lake capabilities.
- Incremental file ingestion with Auto Loader.
- Data quality validation.
- Scheduled job execution.

A production implementation may additionally require formal deployment pipelines, automated testing, alerting, cost controls, data lineage review, and service-level monitoring.

These enhancements should be treated as future work unless they have been implemented and validated.

## 17. Interview Questions and Answers

### Q1. What problem does the Retail Lakehouse solve?

It organizes retail data ingestion, cleansing, transformation, and business consumption in a governed cloud data platform.

### Q2. Why use Azure Databricks?

Azure Databricks provides a managed environment for distributed Spark processing, data transformation, Delta Lake operations, and data engineering workflows.

### Q3. What is the Medallion Architecture?

It is a layered data architecture in which Bronze stores raw data, Silver contains cleaned and standardized data, and Gold contains business-ready datasets.

### Q4. What is Unity Catalog?

Unity Catalog provides centralized governance for supported data assets, including catalogs, schemas, tables, permissions, storage credentials, and external locations.

### Q5. What is Delta Lake?

Delta Lake provides transactional table capabilities on data lake storage, including table history and supported recovery features.

### Q6. What is Auto Loader?

Auto Loader is a Databricks capability for incrementally discovering and ingesting files from supported cloud storage sources.

### Q7. What is the difference between Auto Loader and a watermark?

Auto Loader tracks file-ingestion progress using its streaming and checkpoint mechanisms. A watermark typically represents a processing boundary, such as a timestamp or sequence number, used by an ingestion process to select a data range.

### Q8. Why use a Databricks Job?

A job automates configured notebook or task execution and provides execution history for operational monitoring.

### Q9. Why separate Bronze, Silver, and Gold?

The layers separate raw ingestion, data cleansing, and business modelling. This improves maintainability and makes each stage easier to understand and validate.

### Q10. Is the project a production enterprise platform?

It is an implemented learning and demonstration project that covers key Lakehouse capabilities. Production readiness depends on additional requirements such as operational monitoring, automated deployment, formal testing, recovery objectives, and security review.

## 18. Practical Knowledge Check

- [ ] I can explain the retail business problem.
- [ ] I can draw the source-to-Gold architecture.
- [ ] I can explain the purpose of Bronze, Silver, and Gold.
- [ ] I can identify the Azure resources used by the project.
- [ ] I can explain the Unity Catalog hierarchy.
- [ ] I can explain the roles of a storage credential and external location.
- [ ] I can explain Delta Lake history and time travel.
- [ ] I can describe how Auto Loader tracks incremental ingestion.
- [ ] I can identify the five exported notebooks.
- [ ] I can explain the Databricks Job and its schedule.
- [ ] I can validate the Gold sales table.
- [ ] I can distinguish implemented features from production recommendations.

## 19. Lesson Summary

The Azure Databricks Retail Lakehouse demonstrates how retail source data can be ingested, transformed, governed, and prepared for analytics.

The architecture uses ADLS Gen2 for storage, Azure Databricks for Spark processing, Unity Catalog for governance, Delta Lake for transactional table capabilities, Auto Loader for incremental file ingestion, and Databricks Jobs for automation.

The key principle is:

**Separate raw ingestion, data transformation, and business consumption into clear, governed layers, then validate each layer before relying on the resulting data.**
