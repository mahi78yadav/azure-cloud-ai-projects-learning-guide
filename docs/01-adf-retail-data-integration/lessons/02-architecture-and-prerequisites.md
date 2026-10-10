# Lesson 2 — Architecture and Prerequisites

## Overview

In Lesson 1, we identified the business requirements for the Azure Data Factory Retail Data Integration Project.

In this lesson, we will examine the solution architecture, understand how the Azure services interact, identify dependencies, and review the prerequisites needed to implement and reproduce the project.

We will focus on the architecture of the existing implementation rather than recreating resources that are already configured.

## Learning Objectives

By the end of this lesson, you will be able to:

- Explain the logical architecture of the solution.
- Identify the purpose of each Azure resource.
- Understand the dependencies between Azure services.
- Explain the role of linked services, datasets, pipelines, and triggers.
- Identify the permissions required for data movement.
- Prepare an implementation checklist.
- Understand the difference between a development environment and a production environment.

## 1. Solution Architecture

The solution uses Azure Data Factory to orchestrate file ingestion, ADLS Gen2 to store files, and Azure SQL Database to maintain pipeline control information.

### Architecture Diagram

```text
                 RETAIL SOURCE DATA
              Customer / Product / Order
                         |
                         v
              ADLS GEN2 STORAGE ACCOUNT
                   raw container
                         |
                         v
                 AZURE DATA FACTORY
                         |
              +----------+----------+
              |                     |
              v                     v
        File Discovery         SQL Control Data
        and Copy Activity      Watermark / Errors
              |                     |
              v                     v
        processed container   Azure SQL Database
              |
              v
       Downstream Consumption
```

The diagram shows the logical responsibilities. SQL control operations are performed by the configured pipeline activities, not automatically by ADLS.

## 2. Azure Resources and Their Responsibilities

The following resources belong to the existing development implementation.

| Resource | Name | Responsibility |
|---|---|---|
| Resource group | `rg-retail-data-dev` | Groups the project's Azure resources |
| ADLS Gen2 storage account | `stretaildata202610` | Stores raw and processed files |
| Data Factory | `adf-retail-data-dev-1` | Orchestrates ingestion activities |
| Azure SQL Database | `sqldb-retail-dev` | Stores watermark and error-log records |
| SQL server | `sql-retail-data-dev-2026.database.windows.net` | Hosts the Azure SQL Database |

### Resource Group

A resource group provides a logical boundary for organizing and managing related Azure resources.

It simplifies resource discovery, access management, and lifecycle administration.

### Storage Account

The storage account provides the data lake storage used by the pipeline.

Its containers have distinct responsibilities:

- `raw`: incoming files.
- `processed`: copied files.
- `archive`: intended for retained files under an archive policy.
- `config`: intended for configuration and control files where required.

The exact behavior of each container depends on the pipeline and operational procedures configured for it.

### Azure Data Factory

Data Factory is responsible for orchestrating the workflow. It does not permanently store the source CSV files.

It uses connections, datasets, activities, and triggers to locate, move, and manage data-processing operations.

### Azure SQL Database

Azure SQL Database stores control information used by the workflow.

The project uses:

- `dbo.ETL_Watermark`
- `dbo.ETL_ErrorLog`

The watermark table tracks processing state. The error-log table records failures handled by the configured error path.

## 3. Understand the Main Data Factory Components

### 3.1 Linked Services

A linked service defines connection information for an external service.

The existing project uses:

- `LS_ADLS_Retail`
- `LS_AzureSQL_Retail`

The ADLS linked service connects Data Factory to the storage account. The Azure SQL linked service connects it to the SQL database.

Authentication details and permissions must be configured appropriately for each connection.

### 3.2 Datasets

A dataset identifies the file, folder, table, or other data location used by an activity.

Examples from the project include:

- `DS_Source_Customers`
- `DS_Target_Customers`
- `DS_Source_File`
- `DS_Target_File`
- `DS_Source_Folder`
- `DS_Source_Root`
- `DS_Target_Root`
- `DS_Watermark`

Some datasets use parameters so that a pipeline can reuse the same configuration for different files or folders.

### 3.3 Pipelines

A pipeline groups activities into an executable workflow.

The project includes:

- `PL_Copy_Customers`
- `PL_Copy_All_Files`

The customer pipeline supports a focused copy scenario. The all-files pipeline supports the configured reusable file-ingestion scenario.

### 3.4 Activities

Activities are the individual operations within a pipeline.

Relevant activity types include:

| Activity | Purpose |
|---|---|
| Copy Data | Copies data between source and destination |
| Get Metadata | Retrieves metadata about a configured file or folder |
| ForEach | Iterates over a collection of items |
| Lookup | Reads data from a configured source, such as a SQL control table |
| Stored Procedure or Script | Executes supported SQL operations when configured |
| Set Variable | Assigns a value to a pipeline variable |
| Wait | Pauses execution for a configured duration |

Not every activity listed above is necessarily used in every pipeline. The actual pipeline definition determines which activities execute.

### 3.5 Triggers

A trigger starts a pipeline according to a defined condition or schedule.

The existing project includes:

`TR_Daily_Retail_Ingestion`

The schedule used during testing was set to run every 15 minutes. Before production use, the schedule should be reviewed against the business requirement.

## 4. Understand the Dependencies

The components depend on one another.

```text
Azure Subscription
       |
       v
Resource Group
       |
       +--------------------+
       |                    |
       v                    v
ADLS Gen2              Azure SQL Database
       |                    |
       +----------+---------+
                  |
                  v
          Azure Data Factory
                  |
                  v
      Linked Services and Datasets
                  |
                  v
        Pipelines and Activities
                  |
                  v
         Trigger and Monitoring
```

For a pipeline to run successfully:

1. The required Azure resources must exist.
2. Data Factory must be able to authenticate to the source and destination.
3. The identity or credentials used must have the necessary permissions.
4. Linked services must point to the correct endpoints.
5. Datasets must reference valid paths or tables.
6. Pipeline parameters and activity dependencies must be configured correctly.
7. The trigger must be enabled and configured appropriately when scheduled execution is required.

## 5. Identity and Access Prerequisites

Azure resource creation and runtime access are different concerns.

### Resource Management Access

Permissions are needed to create and configure Azure resources.

For example, creating a Data Factory or storage account requires appropriate permissions on the subscription or resource group.

### Runtime Data Access

The identity used by Data Factory must have access to the services it operates against.

Depending on the authentication configuration:

- ADLS access may use a managed identity or another supported credential.
- SQL access may use a configured database authentication method.
- Azure RBAC and storage data-plane permissions may both be relevant to ADLS access.

The required permissions depend on the chosen authentication method and operation. Grant only the minimum access needed.

### Security Checklist

- [ ] Identify the identity used by each linked service.
- [ ] Confirm that the identity can read the required source paths.
- [ ] Confirm that the identity can write to the destination paths.
- [ ] Confirm SQL access for watermark and error-log operations.
- [ ] Avoid hardcoding credentials in pipeline definitions.
- [ ] Verify access using the actual configured identity.
- [ ] Review permissions before using production data.

## 6. Technical Prerequisites

Before following the implementation lessons, ensure that you understand the following concepts.

| Concept | Required understanding |
|---|---|
| Azure subscription | Where Azure resources are created and billed |
| Resource group | Logical organization for Azure resources |
| ADLS Gen2 | Containers, directories, files, and access permissions |
| Azure Data Factory | Linked services, datasets, pipelines, activities, and triggers |
| Azure SQL Database | Tables, queries, and database connectivity |
| CSV files | File paths, delimiters, headers, and schemas |
| SQL | Basic `SELECT`, `INSERT`, and `UPDATE` statements |
| Incremental processing | Tracking which data qualifies for processing |
| Monitoring | Reviewing pipeline and activity execution results |
| Git and GitHub | Versioning documentation and implementation code |

You do not need to master every topic before proceeding. Learn each concept in greater depth when the relevant lesson introduces it.

## 7. Development Environment and Naming Conventions

The existing project uses development-oriented resource names, such as `rg-retail-data-dev`.

When reproducing the solution, use a consistent naming convention that distinguishes environments and resource types.

For example:

```text
rg-<project>-<environment>
<storage-account-name>
adf-<project>-<environment>
sqldb-<project>-<environment>
LS_<Service>_<Project>
DS_<SourceOrTarget>_<Entity>
PL_<Operation>
TR_<Schedule>
```

Storage account names have additional naming restrictions, including lowercase letters and numbers with a permitted length. Always validate the naming rules when creating a new account.

The names in this handbook document the existing implementation. They may not be available in another Azure subscription or region.

## 8. Cost and Environment Considerations

Azure resources may incur charges while running, even when they are used for learning.

Before starting a new implementation:

- Review the pricing model of the selected resources.
- Use development-sized workloads and test files.
- Avoid unnecessary high-frequency triggers.
- Disable or remove resources that are no longer needed, where appropriate.
- Confirm whether any resource is shared with another project before modifying or deleting it.
- Review Azure cost information before and after testing.

Do not delete the existing project resources merely to repeat a lesson. Use the current resources to understand the implementation and create a separate environment only when a fresh reproduction is necessary.

## 9. Pre-Implementation Checklist

Use this checklist before beginning the next lesson.

- [ ] Understand the business requirements from Lesson 1.
- [ ] Identify the existing storage account and containers.
- [ ] Identify the Data Factory and SQL database.
- [ ] Understand the linked services and datasets.
- [ ] Know where the watermark and error-log tables are stored.
- [ ] Confirm the authentication method used by each linked service.
- [ ] Understand the difference between development and production configuration.
- [ ] Review the cost implications of running Azure resources.
- [ ] Ensure that you have permission to view and inspect the required resources.

## 10. Knowledge Check

### Question 1

What is the primary responsibility of Azure Data Factory?

A. Store all source files permanently.

B. Orchestrate data integration workflows.

C. Replace the Azure storage account.

### Question 2

What is the difference between a linked service and a dataset?

A. A linked service defines the connection, while a dataset identifies the data location.

B. A dataset schedules the pipeline, while a linked service monitors it.

C. They always represent the same configuration.

### Question 3

Why must runtime access permissions be checked?

A. To make all resources publicly accessible.

B. To ensure that the configured identity can perform the required operations.

C. To eliminate the need for authentication.

### Question 4

What should be reviewed before changing the schedule trigger?

A. Only the storage account name.

B. Business data-freshness requirements, execution frequency, and operational cost.

C. The GitHub repository name.

### Answers

1. B
2. A
3. B
4. B

## Summary

In this lesson, we examined the solution architecture, Azure resources, Data Factory components, dependencies, access requirements, and implementation prerequisites.

The architecture separates storage, orchestration, and control information. This separation makes the solution easier to understand, maintain, troubleshoot, and extend.

The next lesson is **Lesson 3 — Create and Configure Storage and SQL Resources**. It will explain the resource setup and validation steps needed for a fresh environment, while distinguishing them from resources that already exist in the current implementation.

**Key principle:** Before implementing a pipeline, understand the services it depends on, how they communicate, and which identities are allowed to access them.
