# Lesson 3 — Create and Configure Storage and SQL Resources

## Overview

In this lesson, we will prepare the storage and database components required by the Azure Data Factory Retail Data Integration Project.

The solution uses:

- Azure Data Lake Storage Gen2 (ADLS Gen2) to store source and processed files.
- Azure SQL Database to store watermark and error-log information.
- An Azure SQL logical server to host the database.

These resources provide the storage and control layer used by the Data Factory pipelines.

**Important:** The resources in the existing project have already been created. Use the inspection sections to understand the current implementation. Create new resources only when setting up a separate environment.

## Learning Objectives

By the end of this lesson, you will be able to:

- Create an ADLS Gen2 storage account.
- Create the required storage containers.
- Upload and verify sample CSV files.
- Create an Azure SQL logical server and database.
- Create the watermark and error-log tables.
- Understand the network, identity, and permission requirements.
- Validate that the resources are ready for Data Factory configuration.

## 1. Understand the Target Architecture

```text
Azure Resource Group
        |
        +-----------------------------+
        |                             |
        v                             v
ADLS Gen2 Storage Account      Azure SQL Logical Server
        |                             |
        +-- raw                       +-- SQL Database
        +-- processed                       |
        +-- archive                         +-- ETL_Watermark
        +-- config                          +-- ETL_ErrorLog
```

The storage account holds files. Azure SQL holds control information used by the pipeline.

These services have different responsibilities and should be configured independently.

## 2. Identify the Existing Resources

The current project uses the following resources:

| Resource | Existing name |
|---|---|
| Resource group | `rg-retail-data-dev` |
| Storage account | `stretaildata202610` |
| SQL server | `sql-retail-data-dev-2026` |
| SQL database | `sqldb-retail-dev` |

The fully qualified SQL server endpoint is:

`sql-retail-data-dev-2026.database.windows.net`

Azure resource names may be unavailable in a different subscription because some names must be globally unique.

## 3. Create or Verify the Resource Group

A resource group organizes the Azure resources for this project.

### Create a resource group

1. Sign in to the [Azure portal](https://portal.azure.com/).
2. Search for **Resource groups**.
3. Select **Create**.
4. Select your Azure subscription.
5. Enter the resource group name:

   `rg-retail-data-dev`

6. Select a suitable region.
7. Select **Review + create**.
8. Select **Create**.

### Verify the resource group

1. Open **Resource groups**.
2. Select `rg-retail-data-dev`.
3. Review the resources listed inside it.

**Note:** In the existing implementation, some related storage resources may be in a different resource group. Azure services can work together across resource groups, provided connectivity and permissions are configured correctly. Do not move or delete resources just to make the layout look uniform.

## 4. Create the ADLS Gen2 Storage Account

Azure Data Lake Storage Gen2 uses Azure Blob Storage with hierarchical namespace enabled.

### Create the storage account

1. In the Azure portal, search for **Storage accounts**.
2. Select **Create**.
3. Choose the subscription.
4. Select the intended resource group.
5. Enter a globally unique storage account name.

   Existing project name: `stretaildata202610`

6. Select a supported region.
7. Choose an appropriate performance and redundancy configuration for a development workload.
8. Open the **Advanced** tab.
9. Enable **Hierarchical namespace**.
10. Review the remaining configuration.
11. Select **Review**.
12. Select **Create**.

The exact portal labels may change over time.

### Why enable hierarchical namespace?

Hierarchical namespace enables directory-oriented data lake capabilities, including directory-level operations and access-control features needed by many analytics workloads.

### Verify the storage account

1. Open the storage account.
2. Navigate to **Data storage → Containers**.
3. Confirm that the account is accessible.
4. Verify that hierarchical namespace is enabled in the account properties.

If the existing storage account already has hierarchical namespace enabled, do not recreate it.

## 5. Create the Storage Containers

Create these containers if they do not already exist.

| Container | Responsibility |
|---|---|
| `raw` | Incoming source files |
| `processed` | Files copied by Data Factory |
| `archive` | Files retained under an archive policy |
| `config` | Configuration and control files, where required |

### Portal steps

1. Open the storage account.
2. Select **Data storage → Containers**.
3. Select **+ Container**.
4. Enter `raw`.
5. Select the appropriate access level, normally private.
6. Select **Create**.
7. Repeat for `processed`, `archive`, and `config`.

Do not enable anonymous public access for these containers.

### Verify the containers

Confirm that all four names appear in the container list.

**Important:** Creating the containers does not automatically implement archiving or configuration-driven processing. Those behaviors must be configured in the relevant pipeline or application.

## 6. Upload Sample CSV Files

For a fresh implementation, prepare sample files such as:

- `customers.csv`
- `products.csv`
- `orders.csv`

A small customer sample might look like this:

```csv
customer_id,customer_name,city
C001,Ravi,Hyderabad
C002,Sita,Warangal
C003,Arun,Khammam
```

Use synthetic data for learning. Do not upload confidential customer information.

### Upload the file

1. Open the `raw` container.
2. Select **Upload**.
3. Choose `customers.csv`.
4. Upload the file.
5. Repeat for the other sample files.

For folder-based ingestion, organize files in subdirectories as required by the later lessons.

Example:

```text
raw/
├── customers/
│   └── customers.csv
├── products/
│   └── products.csv
└── orders/
    └── orders.csv
```

### Verify the files

Open the `raw` container and confirm that the uploaded files or folders appear in the expected locations.

Do not move source files to `processed` manually when testing the ingestion pipeline; otherwise, you will not be validating the actual copy operation.

## 7. Understand Azure SQL Server and Database

Azure SQL Database runs inside an Azure SQL logical server.

The logical server provides configuration and connectivity endpoints. The database contains the tables used by the project.

Existing resources:

- Server: `sql-retail-data-dev-2026`
- Database: `sqldb-retail-dev`

### Create an Azure SQL logical server and database

For a fresh environment:

1. In the Azure portal, search for **SQL databases**.
2. Select **Create**.
3. Select the appropriate subscription and resource group.
4. Enter the database name:

   `sqldb-retail-dev`

5. Choose to create a new logical server if you do not already have one.
6. Enter an available server name, such as `sql-retail-data-dev-2026`, if it is available.
7. Configure the administrator or supported Microsoft Entra authentication method.
8. Choose a development-appropriate compute and pricing configuration.
9. Review the networking settings.
10. Select **Review + create**.
11. Validate the configuration and select **Create**.

Portal options and pricing choices may vary. Choose a configuration appropriate for your subscription and workload.

### Verify the database

1. Open the SQL database resource.
2. Confirm that its status is available.
3. Record the server endpoint.
4. Confirm that the intended authentication method is configured.

## 8. Configure SQL Connectivity

Azure SQL networking and authentication must permit the intended client to connect.

For development, you may use the Azure portal's query editor if it is supported for your authentication setup, or a supported SQL client such as SQL Server Management Studio.

### Networking considerations

- Confirm that the selected connectivity configuration permits the required connection.
- If using a firewall rule for your own development machine, restrict it to the required client IP where possible.
- Do not open SQL access broadly to the internet.
- For production, prefer a controlled network design and approved identity-based access.
- Verify Data Factory connectivity separately when the linked service is configured.

A successful connection from your personal computer does not prove that Data Factory can connect. They may use different identities and network paths.

## 9. Create the Watermark Table

The watermark table stores the processing boundary used by the incremental-loading pipeline.

Connect to `sqldb-retail-dev` and execute the following SQL if the table does not already exist.

```sql
CREATE TABLE dbo.ETL_Watermark
(
    PipelineName NVARCHAR(200) NOT NULL
        PRIMARY KEY,
    WatermarkValue DATETIME2(7) NOT NULL,
    LastUpdatedAt DATETIME2(7) NOT NULL
        DEFAULT SYSUTCDATETIME()
);
```

### Understand the columns

| Column | Purpose |
|---|---|
| `PipelineName` | Identifies the pipeline or ingestion process |
| `WatermarkValue` | Stores the current processing boundary |
| `LastUpdatedAt` | Records when the row was last updated, if maintained by the implementation |

The `PipelineName` primary key permits one row per pipeline name in this table design.

### Insert an initial watermark

Use a starting value appropriate for your source data and initial-load strategy.

For example:

```sql
INSERT INTO dbo.ETL_Watermark
    (PipelineName, WatermarkValue)
VALUES
    ('PL_Copy_All_Files', '2000-01-01T00:00:00');
```

This is an example initial boundary, not a universal production value. A real implementation must define its initial-load and late-arriving-file strategy.

### Verify the table

```sql
SELECT
    PipelineName,
    WatermarkValue,
    LastUpdatedAt
FROM dbo.ETL_Watermark;
```

Confirm that the expected pipeline row exists.

**Implementation note:** Match the table schema and column names to the actual queries and datasets configured in your existing pipeline. Do not recreate or replace a working table using this example schema without checking compatibility.

## 10. Create the Error-Log Table

The error-log table records information about failures handled by the pipeline.

If the existing table does not already exist, an example design is:

```sql
CREATE TABLE dbo.ETL_ErrorLog
(
    ErrorLogID BIGINT IDENTITY(1,1)
        PRIMARY KEY,
    PipelineName NVARCHAR(200) NULL,
    ActivityName NVARCHAR(200) NULL,
    ErrorMessage NVARCHAR(MAX) NULL,
    LoggedAt DATETIME2(7) NOT NULL
        DEFAULT SYSUTCDATETIME()
);
```

### Understand the columns

| Column | Purpose |
|---|---|
| `ErrorLogID` | Unique identifier for each log record |
| `PipelineName` | Identifies the pipeline associated with the error |
| `ActivityName` | Identifies the activity that failed, when available |
| `ErrorMessage` | Stores the reported error details |
| `LoggedAt` | Records when the error was logged |

### Verify the table

```sql
SELECT TOP (10)
    ErrorLogID,
    PipelineName,
    ActivityName,
    ErrorMessage,
    LoggedAt
FROM dbo.ETL_ErrorLog
ORDER BY ErrorLogID DESC;
```

An empty result is normal before any errors have been logged.

**Implementation note:** This is an example schema. Your existing `Log_Error` activity must map to the actual table columns. Keep the current schema if the pipeline is already working.

## 11. Validate Resource Readiness

Before moving to Data Factory configuration, perform the following checks.

### Storage checks

- [ ] Storage account exists and is accessible.
- [ ] Hierarchical namespace is enabled.
- [ ] `raw` container exists.
- [ ] `processed` container exists.
- [ ] `archive` and `config` containers exist if required.
- [ ] Sample CSV files are present in the expected source paths.

### SQL checks

- [ ] SQL server is available.
- [ ] Database `sqldb-retail-dev` exists.
- [ ] A supported authentication method is configured.
- [ ] `dbo.ETL_Watermark` exists and contains the required initial row.
- [ ] `dbo.ETL_ErrorLog` exists with columns compatible with the error-logging activity.
- [ ] SQL queries can be executed using the intended development access method.

### Security checks

- [ ] Containers remain private.
- [ ] SQL connectivity is restricted appropriately.
- [ ] Credentials are not hardcoded into scripts or pipeline definitions.
- [ ] The identity intended for Data Factory has, or can be granted, the required permissions.

## 12. Troubleshooting

| Problem | Possible cause | What to check |
|---|---|---|
| Storage account name rejected | Name is unavailable or violates naming rules | Choose a valid, unique name |
| Container creation fails | Insufficient permissions | Check resource and storage permissions |
| File is not visible | Incorrect container or directory | Verify the upload path |
| SQL connection fails | Firewall, authentication, or network issue | Check the configured identity and connectivity |
| Watermark query fails | Table or column mismatch | Compare the table schema with the pipeline query |
| Error logging fails | Column mapping or SQL permission issue | Inspect the `Log_Error` activity and SQL access |
| Data Factory cannot access storage | Runtime identity lacks permission | Verify the linked-service identity and storage access |

## 13. Knowledge Check

**Question 1:** What is the purpose of the `raw` container?

A. Store incoming source files  
B. Store only error messages  
C. Schedule pipeline execution  

**Question 2:** Why does the project use Azure SQL Database?

A. To replace ADLS file storage  
B. To maintain watermark and error-log information  
C. To create storage containers automatically  

**Question 3:** Does creating an error-log table automatically log pipeline failures?

A. Yes, no further configuration is required  
B. No, the pipeline must execute a configured error-logging operation  
C. Yes, as long as a trigger is enabled  

**Question 4:** Why should you inspect existing resources before creating new ones?

A. To avoid unnecessary duplication and accidental changes  
B. To make all resources public  
C. To remove the need for validation  

### Answers

1. A
2. B
3. B
4. A

## Summary

In this lesson, we covered the storage and SQL foundation for the retail data integration solution.

You learned how to:

- Create or inspect the ADLS Gen2 storage account.
- Create the required containers.
- Upload sample source files.
- Create or inspect the Azure SQL server and database.
- Understand the watermark and error-log tables.
- Validate networking, authentication, and resource readiness.

The next lesson is **Lesson 4 — Create and Configure Azure Data Factory**. It will cover the Data Factory resource, its configuration, and the prerequisites for connecting it to storage and SQL.

**Key principle:** Create the storage and control layers first, verify that they work, and only then configure the orchestration layer.
