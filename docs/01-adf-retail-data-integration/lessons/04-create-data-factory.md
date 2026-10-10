# Lesson 4 — Create and Configure Azure Data Factory

## Overview

In the previous lessons, we identified the business requirements, reviewed the architecture, and prepared the storage and SQL resources.

In this lesson, we will create or inspect the Azure Data Factory resource that orchestrates the retail data integration workflow.

Azure Data Factory coordinates data movement and control activities. It uses linked services to connect to external services, datasets to describe data locations, pipelines to define workflows, and triggers to start those workflows.

**Important:** Your existing Data Factory is `adf-retail-data-dev-1`. If it is already working, inspect its configuration rather than creating another factory.

## Learning Objectives

By the end of this lesson, you will be able to:

- Explain the purpose of Azure Data Factory.
- Create a Data Factory resource in the Azure portal.
- Identify the main components of the Data Factory Studio interface.
- Understand the relationship between linked services, datasets, pipelines, and triggers.
- Review Git integration and publishing concepts.
- Validate that Data Factory is ready for pipeline development.

## 1. Understand the Role of Azure Data Factory

Azure Data Factory is a cloud data integration and orchestration service.

In this project, it coordinates the movement of CSV files from the ADLS Gen2 `raw` container to the `processed` container.

It also orchestrates SQL operations associated with incremental processing and error logging.

### Responsibilities in this project

- Connect to ADLS Gen2.
- Connect to Azure SQL Database.
- Discover source files.
- Execute Copy Data activities.
- Coordinate watermark-related activities.
- Handle configured failure paths.
- Start pipelines through scheduled triggers.
- Provide execution monitoring.

### What Azure Data Factory does not do by itself

- It does not automatically discover every file in every storage account.
- It does not automatically configure permissions for connected services.
- It does not automatically implement incremental loading.
- It does not guarantee that a failed pipeline can recover without intervention.

These capabilities depend on the pipeline design, configuration, and access permissions.

## 2. Identify the Existing Data Factory

The existing project uses the following resource:

| Property | Value |
|---|---|
| Resource group | `rg-retail-data-dev` |
| Data Factory name | `adf-retail-data-dev-1` |
| Service | Azure Data Factory |
| Environment | Development |

The Data Factory may use resources in other resource groups. This is supported as long as the required connectivity and permissions are configured.

## 3. Create Azure Data Factory

Follow these steps only when preparing a fresh environment.

### Step 1 — Open the Azure portal

1. Navigate to the [Azure portal](https://portal.azure.com/).
2. Sign in using your Azure account.
3. Search for **Data factories**.
4. Select **Create**.

### Step 2 — Configure the Basics tab

Enter the following settings:

| Field | Example value |
|---|---|
| Subscription | Your Azure subscription |
| Resource group | `rg-retail-data-dev` |
| Region | A supported region appropriate for your environment |
| Name | `adf-retail-data-dev-1` |
| Version | V2, where shown |

The Data Factory name must be available and satisfy Azure naming rules.

### Step 3 — Configure Git integration

Depending on the portal version, the creation wizard may provide a Git configuration tab.

For a learning project, Git integration is useful because it allows pipeline definitions and related Data Factory artifacts to be version controlled.

If configuring Git during resource creation:

1. Select the supported repository provider.
2. Select your organization or account.
3. Select the repository.
4. Select the collaboration branch.
5. Configure the root folder and publishing branch as required.

If you do not configure Git during creation, you can configure it later through Data Factory Studio.

**Important:** Git integration and publishing are different operations. Committing changes to a collaboration branch does not automatically deploy them to the live Data Factory service.

### Step 4 — Configure networking

Review the networking options presented by the creation wizard.

For a development environment, use an appropriate supported configuration that allows the intended services and users to connect.

For production, networking may require private endpoints, managed virtual networks, or other approved controls.

Do not expose storage or SQL publicly simply to make the initial setup easier.

### Step 5 — Review and create

1. Select **Review + create**.
2. Resolve any validation errors.
3. Select **Create**.
4. Wait for deployment to finish.
5. Select **Go to resource**.

The exact tabs and labels can change as Azure portal features evolve.

## 4. Open Azure Data Factory Studio

Data Factory Studio is the authoring and monitoring interface.

### Steps

1. Open the Data Factory resource.
2. Select **Launch Studio** or **Open Azure Data Factory Studio**, depending on the portal interface.
3. Wait for the Studio interface to load.
4. Review the navigation pane.

### Important areas

| Area | Purpose |
|---|---|
| Author | Create and edit pipelines, datasets, linked services, and related artifacts |
| Monitor | Inspect pipeline runs, activity runs, and failures |
| Manage | Configure linked-service administration, triggers, integration runtimes, and other management settings |
| Git and publish controls | Manage collaboration changes and publish artifacts, when Git integration is configured |

The available labels and features depend on the current Studio interface and repository configuration.

## 5. Understand the Data Factory Authoring Components

Before building pipelines, understand the following components.

### Linked service

A linked service describes how Data Factory connects to an external resource.

Your project uses:

- `LS_ADLS_Retail`
- `LS_AzureSQL_Retail`

The ADLS linked service connects to the storage account. The Azure SQL linked service connects to the database.

### Dataset

A dataset identifies the data location used by an activity.

Examples include:

- `DS_Source_Customers`
- `DS_Target_Customers`
- `DS_Source_File`
- `DS_Target_File`
- `DS_Source_Folder`
- `DS_Source_Root`
- `DS_Target_Root`
- `DS_Watermark`

Datasets may use parameters to support reusable configurations.

### Pipeline

A pipeline groups activities into an executable workflow.

The existing project includes:

- `PL_Copy_Customers`
- `PL_Copy_All_Files`

### Activity

An activity performs an operation within a pipeline.

Examples include Copy Data, Get Metadata, ForEach, Lookup, and Set Variable.

### Trigger

A trigger starts a pipeline according to a schedule or another supported triggering condition.

The existing project includes:

`TR_Daily_Retail_Ingestion`

## 6. Review the Existing Factory

Because the project is already implemented, use this section to verify the current resource.

1. Open `adf-retail-data-dev-1` in the Azure portal.
2. Launch Data Factory Studio.
3. Open the **Author** area.
4. Locate `PL_Copy_Customers`.
5. Locate `PL_Copy_All_Files`.
6. Locate the linked services and datasets listed in this handbook.
7. Open **Monitor** and review the available pipeline runs.
8. Open the trigger management area and locate `TR_Daily_Retail_Ingestion`.

Do not recreate existing artifacts if they are already present.

### Expected result

You should be able to locate the main pipelines, supporting datasets, linked services, and trigger.

If an artifact is missing, verify that you are viewing the correct Data Factory and the correct Git or live authoring context before changing anything.

## 7. Understand Git Mode and Live Mode

When Git integration is configured, Data Factory provides a collaboration workflow.

### Git mode

In Git mode, you work with the artifacts represented in the configured repository branch.

Changes can be reviewed, committed, and merged using the team's source-control process.

### Live mode

Live mode represents the artifacts currently published to the Data Factory service.

The authoring experience may expose different options depending on whether Git integration is configured.

### Publish workflow

A typical workflow is:

```text
Edit Data Factory Artifacts
          |
          v
Validate Changes
          |
          v
Commit to Collaboration Branch
          |
          v
Review and Merge Changes
          |
          v
Publish Approved Artifacts
          |
          v
Monitor Live Pipeline Runs
```

The exact workflow depends on the repository configuration and deployment process.

**Important:** A successful Git commit or merge alone does not prove that the live Data Factory contains the latest changes. Validate the published artifacts and test the intended execution.

## 8. Understand Integration Runtime

An integration runtime provides the compute and connectivity infrastructure used by Data Factory activities.

Depending on the activity and connectivity requirements, Data Factory can use supported integration runtime types such as:

- Azure integration runtime.
- Self-hosted integration runtime.
- Azure-SSIS integration runtime.

For this project, review the integration runtime used by the existing linked services and activities.

An Azure integration runtime is commonly used for cloud-to-cloud data movement. A self-hosted integration runtime may be needed for resources accessible only through private networks or local infrastructure.

Do not create another integration runtime without first checking the existing configuration and the connectivity requirements.

## 9. Validate Data Factory Readiness

Use this checklist before proceeding to linked-service configuration.

### Resource checks

- [ ] Data Factory resource exists.
- [ ] Data Factory Studio opens successfully.
- [ ] Correct subscription and resource group are selected.
- [ ] The expected factory name is confirmed.

### Authoring checks

- [ ] Existing pipelines can be located.
- [ ] Existing linked services can be located.
- [ ] Existing datasets can be located.
- [ ] Trigger configuration can be inspected.
- [ ] The current Git or live authoring context is understood.

### Operational checks

- [ ] Monitor view is accessible.
- [ ] Existing execution history can be inspected.
- [ ] The configured integration runtime is understood.
- [ ] Changes will not overwrite or disrupt a working implementation.

## 10. Troubleshooting

| Problem | Possible cause | What to check |
|---|---|---|
| Data Factory creation fails | Invalid name, missing permissions, or deployment validation error | Review deployment details |
| Studio does not open | Browser, authorization, or service configuration issue | Verify access and retry |
| Expected pipeline is missing | Wrong factory, branch, or authoring context | Confirm the resource and repository context |
| Linked service test fails | Incorrect endpoint, credentials, network, or permissions | Review the linked service configuration |
| Changes are visible in Git but not live | Changes have not been published or deployed | Check the publishing workflow |
| Pipeline fails during execution | Connection, dataset, source, destination, or activity error | Inspect activity-level monitoring details |

## 11. Knowledge Check

### Question 1

What is the primary role of Azure Data Factory?

A. Store all CSV files permanently.

B. Orchestrate data integration workflows.

C. Replace the SQL database.

### Question 2

What is the purpose of a linked service?

A. Define connection information for an external service.

B. Store every row of source data.

C. Schedule a pipeline.

### Question 3

What is the difference between committing and publishing?

A. They are always the same operation.

B. Committing records source-control changes; publishing makes supported artifacts available in the live service.

C. Publishing only changes the GitHub README.

### Question 4

Why should the existing integration runtime be inspected before creating another one?

A. To avoid unnecessary configuration and ensure the correct connectivity model is used.

B. To make all services public.

C. To eliminate the need for linked services.

### Answers

1. B
2. A
3. B
4. A

## Summary

In this lesson, we covered the creation and configuration of Azure Data Factory, the Studio interface, authoring components, integration runtime, Git integration, publishing, and validation.

The existing factory `adf-retail-data-dev-1` is the orchestration component for the retail data integration project.

The next lesson is **Lesson 5 — Configure Linked Services and Authentication**, where we will examine how Data Factory connects securely to ADLS Gen2 and Azure SQL Database.

**Key principle:** A pipeline is only as reliable as its connections, permissions, dependencies, and deployment process.
