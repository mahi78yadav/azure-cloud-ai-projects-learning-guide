# Module 01 — Azure Data Factory Retail Data Integration

## 1. Module Purpose

This module explains how to design, implement, validate, and operate an enterprise-style data integration pipeline using Microsoft Azure.

The objective is not simply to copy files. It is to understand how data moves from source storage into a processing area, how incremental processing avoids unnecessary work, and how failures can be recorded and monitored.

This module documents an already implemented project so that learners can understand it and reproduce it later.

## 2. Business Scenario

A retail business receives customer and other operational data as CSV files. New files arrive over time, and the business needs a reliable process to copy these files into a processed data area.

The solution should:

- Ingest multiple CSV files.
- Process files from nested folders.
- Support incremental processing.
- Track processing progress using a watermark.
- Update the watermark only after successful copying.
- Record failures in an error log.
- Support retries, scheduled execution, and monitoring.

## 3. Architecture

```text
Retail CSV Files
       |
       v
Azure Data Lake Storage Gen2
       |
       | Raw container
       v
Azure Data Factory
       |
       | Get watermark
       | Copy eligible files
       | Update watermark
       | Log failures
       v
Azure Data Lake Storage Gen2
       |
       | Processed container
       v
Validation and Monitoring

Azure SQL Database
       |
       | Watermark table
       | Error log table
       v
Pipeline control and operational history
```

### How the components work together

- **ADLS Gen2:** Stores the source files and copied output files.
- **Azure Data Factory (ADF):** Orchestrates the pipeline activities and their execution order.
- **Azure SQL Database:** Stores pipeline control information, including the last successful watermark and error records.
- **Monitoring:** Shows pipeline runs, activity status, duration, and errors.
- **GitHub:** Stores documentation and version history.

## 4. Azure Resources Used

| Resource | Project resource name | Purpose |
|---|---|---|
| Resource group | `rg-retail-data-dev` | Groups the ADF project resources |
| Storage account | `stretaildata202610` | Stores source and processed files |
| ADF factory | `adf-retail-data-dev-1` | Orchestrates data movement |
| SQL database | `sqldb-retail-dev` | Stores watermarks and error logs |
| SQL logical server | `sql-retail-data-dev-2026` | Hosts the Azure SQL database |
| Integration runtime | `AutoResolveIntegrationRuntime` | Provides the execution environment for data movement |

Resource names are environment-specific. Never include passwords, access tokens, connection secrets, or API keys in GitHub.

## 5. Technical Concepts — What and Why

### 5.1 Resource groups

A resource group is a logical container for Azure resources. It helps organize access, deployment, monitoring, and cost management.

**Why it is used:** Related resources can be identified and managed together. Resources can still communicate across resource groups when permissions and network configuration allow it.

### 5.2 ADLS Gen2

Azure Data Lake Storage Gen2 is built on Azure Blob Storage and supports hierarchical namespaces.

The project uses separate containers:

- `raw`: incoming source files.
- `processed`: copied files ready for downstream use.
- `archive`: reserved for files that need to be retained separately.
- `config`: reserved for configuration and metadata files.

Separating these areas improves organization and helps establish a repeatable ingestion pattern.

### 5.3 Azure Data Factory

ADF is a cloud data integration and orchestration service. A pipeline groups activities into a workflow and defines how they execute.

For example, a pipeline can retrieve a watermark, copy files, and update the watermark only when the copy succeeds.

### 5.4 Linked services

A linked service defines connection information for a data store or compute service.

Project examples:

- `LS_ADLS_Retail`
- `LS_AzureSQL_Retail`

**Why it is used:** Datasets and activities can reuse centrally configured connections rather than repeating connection details.

### 5.5 Datasets

A dataset describes the data that an ADF activity reads or writes. It can refer to a file, folder, table, or other supported data structure.

Project examples:

- `DS_Source_Customers`
- `DS_Target_Customers`
- `DS_Source_File`
- `DS_Target_File`
- `DS_Source_Folder`
- `DS_Source_Root`
- `DS_Target_Root`
- `DS_Watermark`

Parameterized datasets allow a pipeline to reuse a dataset for different paths or files.

### 5.6 Integration runtime

An integration runtime provides the infrastructure required by ADF to move data, dispatch activities, or connect to supported data sources.

The project uses `AutoResolveIntegrationRuntime`, which selects an appropriate Azure integration runtime region based on the service configuration.

## 6. Pipeline Development

### 6.1 Customer copy pipeline

Pipeline: `PL_Copy_Customers`

Purpose:

- Demonstrate a focused source-to-target copy.
- Validate the basic linked service, dataset, and Copy Activity configuration.

### 6.2 Multi-file copy pipeline

Pipeline: `PL_Copy_All_Files`

The main copy configuration reads CSV files from the raw root and writes them to the processed root.

Key settings include:

- Wildcard file selection: `*.csv`
- Recursive traversal of subfolders
- Preservation of folder hierarchy in the destination

**Why wildcard selection is useful:** It allows a pipeline to process multiple matching files without creating a separate Copy Activity for each file.

**Why recursive traversal is useful:** Files can be stored in nested directories and still be included in the ingestion process.

### 6.3 Get Metadata and ForEach

Get Metadata can retrieve information about files and folders, such as child items or file properties.

ForEach iterates over a collection of items and executes the activities inside it.

Together, these activities support dynamic file processing. A production implementation should also define how empty folders, unexpected file types, and individual file failures are handled.

## 7. Incremental Processing and Watermarks

### 7.1 What is a watermark?

A watermark records a processing boundary, such as the last successfully processed modification timestamp.

Instead of copying every eligible file on every run, the pipeline uses the previous watermark and a new upper boundary to determine which files qualify for processing.

A watermark is control metadata. It is not the data itself.

### 7.2 Watermark table

```sql
CREATE TABLE dbo.ETL_Watermark
(
    PipelineName VARCHAR(200) NOT NULL PRIMARY KEY,
    LastWatermark DATETIME2 NOT NULL,
    UpdatedDate DATETIME2 NOT NULL
        DEFAULT SYSUTCDATETIME()
);
```

The initial watermark is inserted with an old timestamp so that eligible existing files can be picked up on the first run.

```sql
INSERT INTO dbo.ETL_Watermark
(
    PipelineName,
    LastWatermark
)
VALUES
(
    'PL_Copy_All_Files',
    '2000-01-01T00:00:00'
);
```

### 7.3 Incremental execution sequence

```text
Set_End_Watermark
        |
        v
Get_Watermark
        |
        v
Copy_Raw_To_Processed
        |
        +---- Success ----> Update_Watermark
        |
        +---- Failure ----> Log_Error
```

The end watermark is captured at the start of the run using the pipeline variable `EndWatermark`.

The Copy Activity uses:

- Start boundary: the previous watermark retrieved from SQL.
- End boundary: the current run's `EndWatermark`.

The watermark update occurs only after the Copy Activity succeeds.

**Why capture an upper boundary?** It defines a consistent processing window for the current run and helps avoid moving the boundary forward before the copy has completed.

**Important limitation:** A modification-time watermark is not a complete change-data-capture system. Files that arrive late, have old timestamps, or change while processing require an appropriate overlap, deduplication, or reconciliation strategy. A production design must also consider retries and partially copied outputs.

### 7.4 Update the watermark

```sql
UPDATE dbo.ETL_Watermark
SET
    LastWatermark = @EndWatermark,
    UpdatedDate = SYSUTCDATETIME()
WHERE PipelineName = 'PL_Copy_All_Files';
```

The pipeline supplies `@EndWatermark` from its variable.

**Why update after success?** Advancing the watermark before copying completes can cause files to be skipped in later runs.

## 8. Error Logging and Retry

### 8.1 Error log table

```sql
CREATE TABLE dbo.ETL_ErrorLog
(
    ErrorLogID INT IDENTITY(1,1) PRIMARY KEY,
    PipelineName VARCHAR(200) NOT NULL,
    ActivityName VARCHAR(200) NULL,
    ErrorMessage NVARCHAR(MAX) NULL,
    ErrorTime DATETIME2 NOT NULL
        DEFAULT SYSUTCDATETIME()
);
```

The failure dependency from the Copy Activity invokes `Log_Error`. The script records the pipeline name, activity name, and error message.

### 8.2 Retry configuration

The Copy Activity was configured with:

- Retry count: 2
- Retry interval: 30 seconds

**Why retry?** Temporary network or service issues may resolve without manual intervention.

Retries should be used with care. Repeating a write operation must not create duplicate or inconsistent output. A robust design also considers idempotency, partial output cleanup, and alerting.

### 8.3 Failure-path validation

The project was tested with an intentionally invalid source path. The pipeline's failure path successfully wrote an error record.

A failed run should not advance the watermark.

## 9. Trigger and Monitoring

The project uses a schedule trigger named `TR_Daily_Retail_Ingestion`, configured during testing to run every 15 minutes in India Standard Time.

The trigger was started, and multiple successful executions were observed.

Monitoring should be used to inspect:

- Pipeline run status
- Activity run status
- Rows and files read or written
- Error messages
- Duration and retry attempts
- Watermark values before and after execution

A 15-minute schedule was a test configuration, not a recommendation for every production workload. Choose a schedule based on the business requirement, expected arrival frequency, and cost.

## 10. Validation Scenarios

| Scenario | Expected outcome |
|---|---|
| Initial eligible files exist | Files are copied to the processed area |
| No new or modified eligible files | No qualifying files are copied |
| A new eligible file arrives | The file is copied |
| Source path is invalid | The Copy Activity fails and the error is logged |
| Copy fails | Watermark is not advanced |
| Copy succeeds | Watermark is updated |
| Scheduled trigger fires | A pipeline run is created and can be monitored |

Observed project results included a successful incremental test with `customers_incremental.csv`, which copied 126 records in one file. A no-new-files test returned zero files read and written.

## 11. Security and Governance

Important principles:

- Apply least-privilege access to storage and SQL.
- Prefer managed identities where supported.
- Keep credentials out of source code and GitHub.
- Use secure secret storage for credentials that cannot be replaced by identity-based authentication.
- Restrict SQL firewall and network access as required.
- Separate development and production resources where appropriate.
- Monitor access and pipeline failures.

Authentication establishes identity; authorization determines what that identity is allowed to do.

## 12. Common Troubleshooting

### Dataset schema import fails

Check the linked service, container, folder path, filename, and whether the source file exists. Schema import requires access to a valid source.

### No files are copied

Check wildcard filters, source path, recursive settings, timestamps, and the previous and current watermark boundaries.

### Watermark does not update

Check the Copy Activity success dependency, SQL linked service, SQL permissions, and the update script parameters.

### Error is not logged

Check that the failure dependency is configured, the logging activity is reachable, the SQL table exists, and the error parameters resolve correctly.

### Trigger does not run

Check whether the trigger is published and started, its schedule and time zone, and the trigger's associated pipeline.

## 13. Reproduction Checklist

Before reproducing the project, confirm that you can explain:

- [ ] Why ADLS Gen2 is used.
- [ ] The difference between a linked service and a dataset.
- [ ] The role of the integration runtime.
- [ ] How wildcard and recursive copying work.
- [ ] Why a watermark is required.
- [ ] How start and end boundaries control incremental processing.
- [ ] Why the watermark updates only after successful copying.
- [ ] How failure dependencies and SQL error logging work.
- [ ] How retry settings help with transient failures.
- [ ] How schedule triggers and monitoring work.
- [ ] How access permissions and secrets are managed.

## 14. Interview Questions

1. What is Azure Data Factory, and when would you use it?
2. What is the difference between a pipeline, activity, linked service, and dataset?
3. What is an integration runtime?
4. How would you design a metadata-driven ingestion pipeline?
5. What is a watermark, and how does it support incremental processing?
6. Why should the watermark be updated only after successful processing?
7. What can go wrong with timestamp-based incremental loading?
8. How would you prevent duplicate processing after a retry?
9. How do you log failures in ADF?
10. How do you monitor scheduled pipelines?
11. How would you secure ADF connections to ADLS and Azure SQL?
12. What additional controls would you add before production deployment?

## 15. Learning Outcome

After studying and reproducing this module, a learner should be able to explain and implement a basic Azure file-ingestion solution with incremental processing, operational error logging, retries, scheduling, and monitoring.

This guide documents the project's implementation and observed tests. It is not, by itself, evidence that every production requirement—such as late-arriving files, concurrent runs, duplicate prevention, alerting, or disaster recovery—has been implemented.
