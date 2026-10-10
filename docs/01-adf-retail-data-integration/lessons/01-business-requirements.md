
# Lesson 1 — Understand the Business Requirements

## Overview

Before implementing an Azure Data Factory solution, a data engineer must understand the business problem, the source data, the expected output, and the operational requirements.

In this lesson, we will understand the business scenario behind the **Azure Data Factory Retail Data Integration Project**.

This lesson focuses on requirements and architecture. Azure resource creation and pipeline implementation will be covered in subsequent lessons.

## Learning Objectives

By the end of this lesson, you will be able to:

- Explain the business problem this project solves.
- Identify the source data, destination, and Azure services.
- Understand the difference between data ingestion and data transformation.
- Explain why incremental loading is required.
- Describe the purpose of watermark tracking, error logging, retries, and monitoring.
- Explain the end-to-end architecture to another engineer.

## 1. Business Scenario

Consider a retail company that receives data files from different business systems.

These files contain information about:

- Customers
- Products
- Orders
- Stores and other retail entities

The source systems generate CSV files periodically. The company needs to move these files into a centralized data lake so that downstream systems can access the data consistently.

Initially, files might be copied manually. However, manual processing becomes difficult as the number of files and frequency of delivery increase.

The company therefore needs an automated data integration solution.

### Business Problem

The company faces the following challenges:

1. New files arrive regularly and must be processed reliably.
2. Files may be organized in multiple folders.
3. Reprocessing every file on every run wastes time and resources.
4. Pipeline failures must be detected and investigated.
5. The system must record its processing state.
6. Operations teams need to monitor pipeline execution.
7. The ingestion process should be reusable when new files are introduced.

## 2. Proposed Solution

We will use Azure Data Factory to orchestrate file ingestion and Azure Data Lake Storage Gen2 to store source and processed files.

Azure SQL Database will maintain control information, including the watermark and error-log records used by the pipeline.

The solution will provide:

- Automated file copying.
- Reusable and parameterized datasets.
- Recursive file discovery where configured.
- Watermark-based incremental processing.
- Failure handling and error logging.
- Configurable retry behavior.
- Scheduled execution and monitoring.

## 3. Business Requirements

### 3.1 Functional Requirements

| ID | Requirement | Implementation approach |
|---|---|---|
| FR-01 | Store incoming source files centrally. | ADLS Gen2 `raw` container |
| FR-02 | Copy files to a processed location. | Azure Data Factory Copy Activity |
| FR-03 | Process files across nested folders where configured. | Recursive file selection and wildcard settings |
| FR-04 | Reuse pipeline logic for different files. | Parameterized datasets and pipeline parameters |
| FR-05 | Track the processing boundary. | SQL watermark table |
| FR-06 | Advance the watermark only after successful processing. | Success-dependent pipeline activity |
| FR-07 | Record processing failures. | SQL error-log table and failure path |
| FR-08 | Retry eligible failures. | Copy Activity retry configuration |
| FR-09 | Run ingestion automatically. | Schedule trigger |
| FR-10 | Inspect pipeline execution results. | Data Factory Monitor |

### 3.2 Non-Functional Requirements

| Requirement | Explanation |
|---|---|
| Reliability | Failures must be visible and recoverable. |
| Maintainability | Pipeline logic should be reusable rather than duplicated. |
| Scalability | The design should support additional files and folders. |
| Security | Services must have only the permissions they require. |
| Observability | Pipeline runs and failures must be inspectable. |
| Cost awareness | Resources and trigger frequency should suit the workload. |
| Data integrity | Files must be validated and processing state must remain consistent. |

## 4. Understand the Source and Destination

The project uses Azure Data Lake Storage Gen2.

The storage account is:

`stretaildata202610`

The containers are:

| Container | Intended responsibility |
|---|---|
| `raw` | Incoming source files |
| `processed` | Files copied by the ingestion pipeline |
| `archive` | Retention of files according to an archive policy |
| `config` | Configuration and control files, where required |

Example source structure:

```text
raw/
├── customers/
│   ├── customers_20261009.csv
│   └── customers_20261010.csv
├── products/
│   └── products_20261010.csv
└── orders/
    ├── orders_20261009.csv
    └── orders_20261010.csv
```

The pipeline is designed to copy matching files to the processed area while preserving the folder hierarchy according to its configuration.

**Important:** The existence of an `archive` or `config` container does not mean every pipeline automatically uses it. Each container's behavior depends on the implementation.

## 5. Understand the Azure Services

### Azure Data Factory

Azure Data Factory is the orchestration and data movement service.

In this project, it coordinates activities such as file discovery, copying, watermark management, and error logging.

### Azure Data Lake Storage Gen2

ADLS Gen2 provides scalable storage for source and processed files.

It separates storage from the pipeline logic, allowing files to be retained independently of the orchestration service.

### Azure SQL Database

Azure SQL Database stores pipeline control information.

The project uses these tables:

- `dbo.ETL_Watermark` — records the processing boundary.
- `dbo.ETL_ErrorLog` — records failures handled by the configured error-logging path.

These tables are control and operational records; they are not the source CSV files themselves.

## 6. High-Level Architecture

```text
Retail Source Systems
        |
        v
CSV Files
        |
        v
ADLS Gen2: raw
        |
        v
Azure Data Factory
        |
        +--> Discover and select files
        |
        +--> Apply incremental-processing rules
        |
        +--> Copy eligible files
        |
        +--> Handle failures and retries
        |
        +--> Update watermark after success
        |
        v
ADLS Gen2: processed

Azure SQL Database
        |
        +--> ETL_Watermark
        |
        +--> ETL_ErrorLog
```

The diagram represents the logical flow. The exact execution sequence depends on the activities and dependency conditions configured in the pipeline.

## 7. Why Is Incremental Loading Important?

Suppose a source folder contains 10,000 files, but only 20 new files arrive today.

A full-copy design might examine or copy all 10,000 files on every run, depending on its configuration.

An incremental design identifies the files that qualify for processing and avoids unnecessary repeat work.

A watermark records the processing boundary used by this selection process.

For example:

| Run | Processing boundary | Expected behavior |
|---|---|---|
| First run | Initial configured boundary | Process files within the initial scope |
| Second run | Previous successful boundary | Process qualifying new or changed files |
| Failed run | Boundary must not advance prematurely | Allow eligible work to be retried |

A watermark alone does not guarantee exactly-once processing. A robust implementation must also consider duplicate files, partial copies, timestamp ties, late-arriving files, and safe reruns.

## 8. Reliability and Failure Handling

The pipeline must not assume every activity succeeds.

Possible failures include:

- A source file is missing.
- Access permissions are insufficient.
- A storage or SQL service is temporarily unavailable.
- A file is locked or malformed.
- A destination path is invalid.

The project configures retries and an error-logging path.

A retry is useful for transient errors. Persistent errors require investigation.

The watermark must be updated only when the required copy operation succeeds. Otherwise, the pipeline could record a boundary that moves beyond files that were not successfully processed.

## 9. Project Resources

The principal resources used by the existing implementation are:

| Resource | Name |
|---|---|
| Resource group | `rg-retail-data-dev` |
| Storage account | `stretaildata202610` |
| Data Factory | `adf-retail-data-dev-1` |
| Azure SQL Database | `sqldb-retail-dev` |
| SQL server | `sql-retail-data-dev-2026.database.windows.net` |
| ADLS linked service | `LS_ADLS_Retail` |
| Azure SQL linked service | `LS_AzureSQL_Retail` |
| Customer copy pipeline | `PL_Copy_Customers` |
| File ingestion pipeline | `PL_Copy_All_Files` |
| Scheduled trigger | `TR_Daily_Retail_Ingestion` |

These names document the existing project. If you reproduce the solution in another subscription, choose names appropriate to that environment.

## 10. Scope and Limitations

This project demonstrates data integration, orchestration, incremental-processing concepts, error logging, and monitoring.

It should not automatically be described as a complete enterprise data platform.

Additional design work may be required for production use, including:

- Idempotent processing and duplicate prevention.
- File-level manifests or checkpoints.
- Schema validation and data-quality enforcement.
- Late-arriving file handling.
- Retention and archival policies.
- Alerting and operational ownership.
- Disaster recovery and deployment automation.
- Production access reviews and cost controls.

The purpose of identifying these items is to distinguish the implemented project from future enhancements.

## 11. Knowledge Check

**Question 1:** What is the primary responsibility of Azure Data Factory in this project?

A. Permanently store all source files  
B. Orchestrate and execute the ingestion workflow  
C. Replace Azure SQL Database  

**Question 2:** What is the purpose of `dbo.ETL_Watermark`?

A. Store every source CSV file  
B. Store pipeline processing-boundary information  
C. Schedule Azure resources  

**Question 3:** Why should the watermark not advance after a failed copy?

A. To avoid incorrectly treating unprocessed files as already handled  
B. To reduce the size of the storage account  
C. To disable all retries  

**Question 4:** What does the `processed` container represent?

A. The configured destination for copied files  
B. The SQL error table  
C. The Data Factory trigger  

### Answers

1. B
2. B
3. A
4. A

## 12. Practical Exercise

Before moving to the next lesson, verify that you can:

- [ ] Identify the purpose of each Azure resource.
- [ ] Explain the difference between raw and processed data.
- [ ] Explain why Data Factory uses linked services and datasets.
- [ ] Describe the purpose of the watermark and error-log tables.
- [ ] Explain the difference between retry handling and error logging.
- [ ] Draw the high-level architecture without referring to this document.

## Summary

In this lesson, we established the business problem, functional and non-functional requirements, Azure services, high-level architecture, and reliability expectations for the retail data integration solution.

The next lesson is **Lesson 2 — Architecture and Prerequisites**, where we will examine the components, dependencies, access requirements, and design decisions before implementing the Azure resources.

**Key principle:** Understand the business requirement first, design the solution second, and implement and validate it step by step.
