# Lesson 7 — Build the Copy Data Pipeline

## Overview

In the previous lessons, we prepared the Azure resources, configured linked services, and defined datasets.

In this lesson, we will learn how to build a pipeline that copies retail CSV files from Azure Data Lake Storage Gen2 (ADLS Gen2) into the processed storage area.

Azure Data Factory uses the **Copy Data activity** to move data between supported sources and destinations.

The existing project contains two relevant pipelines:

- `PL_Copy_Customers`
- `PL_Copy_All_Files`

The first demonstrates a focused customer-copy scenario. The second supports reusable file ingestion based on its configured source paths, wildcard rules, and recursive behavior.

**Important:** Your existing pipelines are already implemented. Inspect them first. Follow the creation steps in a separate development environment if you want to reproduce the solution from scratch.

## Learning Objectives

By the end of this lesson, you will be able to:

- Explain the purpose of a Copy Data activity.
- Create a basic file-copy pipeline.
- Configure source and destination datasets.
- Understand source paths, destination paths, and file formats.
- Configure recursive file selection and wildcard filtering.
- Validate the output files.
- Troubleshoot common pipeline execution failures.

## 1. Understand the Copy Pipeline Architecture

The objective is to copy CSV files from the `raw` container to the `processed` container.

```text id="1j4ph9"
ADLS Gen2
  |
  v
raw/customers/customers.csv
  |
  v
Azure Data Factory
  |
  v
Copy Data Activity
  |
  v
ADLS Gen2
  |
  v
processed/customers/customers.csv
```

The exact destination path depends on the dataset and Copy Activity configuration.

The Copy Data activity reads from the configured source and writes to the configured destination. It does not automatically discover every file in the storage account; the source settings determine which files are selected.

## 2. Identify the Existing Pipeline and Datasets

The existing project uses these artifacts:

| Artifact | Name |
|---|---|
| Customer copy pipeline | `PL_Copy_Customers` |
| General file ingestion pipeline | `PL_Copy_All_Files` |
| Customer source dataset | `DS_Source_Customers` |
| Customer target dataset | `DS_Target_Customers` |
| Reusable source file dataset | `DS_Source_File` |
| Reusable target file dataset | `DS_Target_File` |

Before changing anything, open the existing pipeline and verify the dataset names actually referenced by its Copy Data activity.

## 3. Create a Basic Copy Pipeline

Follow these steps only if creating a new pipeline or reproducing the solution in a separate environment.

### Step 1 — Open Data Factory Studio

1. Open the Azure portal.
2. Open `adf-retail-data-dev-1`.
3. Select **Launch Studio**.
4. Open the **Author** area.
5. Select the option to create a new pipeline.

### Step 2 — Name the pipeline

Set the pipeline name to:

`PL_Copy_Customers`

If this pipeline already exists, do not create a duplicate. Open the existing pipeline instead.

### Step 3 — Add a Copy Data activity

1. Locate **Move and transform** in the activity pane.
2. Drag **Copy Data** onto the pipeline canvas.
3. Select the activity.
4. Give it a meaningful name, such as `Copy_Customers`.

The activity name is for readability and monitoring. It does not change the source or destination by itself.

## 4. Configure the Source

Select the Copy Data activity and open its **Source** tab.

Configure the source dataset:

`DS_Source_Customers`

Verify that the dataset points to the intended source file in the `raw` container.

For example:

```text id="x6ku5v"
Container: raw
Directory: customers
File: customers.csv
```

The actual path must match the file uploaded to ADLS Gen2.

### Source settings to review

- The correct source dataset is selected.
- The correct container and directory are configured.
- The file name matches the source file.
- The source format is configured as CSV.
- Header and delimiter settings match the file.
- The linked service can access the source path.

### Why this matters

A pipeline can be configured correctly at the activity level but still fail if the dataset points to a nonexistent file or the identity cannot read it.

## 5. Configure the Destination

Open the Copy Data activity's **Sink** tab.

Select:

`DS_Target_Customers`

Verify that the destination points to the intended location in the `processed` container.

For example:

```text id="d9w4yk"
Container: processed
Directory: customers
File: customers.csv
```

The destination configuration must be compatible with the source and the desired output format.

### Destination settings to review

- The correct target dataset is selected.
- The target container is `processed`.
- The destination path is correct.
- The destination format is appropriate.
- The identity has permission to write to the target location.
- Existing files will be handled according to the configured overwrite or file-writing behavior.

**Important:** A fixed destination file name may be appropriate for a single-file pipeline. For multiple source files, use a suitable dynamic destination design to prevent unintended overwrites.

## 6. Validate and Debug the Pipeline

Before running the pipeline, validate its configuration.

### Step 1 — Validate

1. Select **Validate** in the authoring interface.
2. Review any reported errors.
3. Correct missing datasets, invalid expressions, or configuration problems.

Validation checks the pipeline definition. It does not prove that every source file exists or that runtime permissions are sufficient.

### Step 2 — Debug

1. Select **Debug**.
2. Wait for the pipeline run to complete.
3. Review the activity result.
4. If it fails, inspect the activity's detailed error information.

Debug runs are useful during development. For scheduled production execution, ensure the intended artifacts have been published and the trigger is configured correctly.

## 7. Verify the Output Files

After a successful run:

1. Open the Azure portal.
2. Navigate to storage account `stretaildata202610`.
3. Open the `processed` container.
4. Navigate to the configured destination directory.
5. Confirm that the expected file exists.
6. Inspect its contents or compare its row count with the source, where appropriate.

For a simple file copy, compare:

- Source and destination file names.
- File sizes, allowing for any expected format or encoding behavior.
- Row counts and column headers.
- Sample records.

A successful pipeline status is necessary, but output validation confirms that the expected data was actually written.

## 8. Understand the General File-Copy Pipeline

The existing project also includes:

`PL_Copy_All_Files`

This pipeline supports the reusable file-ingestion scenario.

Unlike a pipeline configured for one fixed customer file, a general ingestion pipeline can use folder discovery, file filtering, and parameterized datasets.

Its behavior depends on the configured activities and source settings.

### Typical conceptual flow

```text id="jj41j7"
Start Pipeline
      |
      v
Identify Source Scope
      |
      v
Select Matching CSV Files
      |
      v
Copy Files to Processed
      |
      v
Review Run Results
```

The actual implementation may use Get Metadata, ForEach, or other activities to discover and process files. These activities should be documented in detail in the next lesson.

## 9. Configure Recursive File Selection

A recursive source configuration allows the file-copy operation to include matching files in nested directories, where supported by the selected connector and activity settings.

For example:

```text id="3f4s6p"
raw/
├── customers/
│   ├── customers_01.csv
│   └── customers_02.csv
├── products/
│   └── products_01.csv
└── orders/
    └── orders_01.csv
```

A source configuration using the wildcard `*.csv` and recursive selection can include matching CSV files throughout the configured directory hierarchy.

### Important configuration considerations

- Confirm the root source path.
- Configure the intended wildcard pattern.
- Enable recursive traversal where required.
- Check how the destination path is constructed.
- Confirm that duplicate or repeated source files will not overwrite unrelated destination files.
- Validate the result with files in nested folders.

Recursive traversal and wildcard filtering are separate concepts: recursion determines whether nested folders are traversed, while the wildcard determines which file names match.

## 10. Understand Copy Activity Retry Settings

Your project uses a retry configuration of two retries with a 30-second interval.

Retries can help recover from temporary failures, such as a brief service interruption.

However, retries cannot fix persistent configuration problems such as:

- A permanently invalid source path.
- Missing permissions.
- An incorrect destination configuration.
- An incompatible file format.

The activity's final status and error details should still be reviewed after execution.

The precise retry behavior depends on the activity settings and error condition.

## 11. Common Copy Pipeline Errors

| Problem | Possible cause | What to check |
|---|---|---|
| Source file not found | Incorrect source path or file name | Verify the dataset path and source file |
| Access denied | Insufficient storage permissions | Check the linked-service identity and access controls |
| Destination file missing | Incorrect sink path or failed activity | Inspect the sink settings and activity result |
| Files overwrite each other | Fixed destination name used for multiple inputs | Review the destination naming strategy |
| Pipeline validates but fails at runtime | Missing file, permission issue, or runtime dependency | Inspect the detailed activity error |
| Some files are not copied | Wildcard, recursion, or source scope excludes them | Review source selection settings |
| Unexpected output structure | Destination hierarchy settings differ from expectations | Compare the source and sink configurations |

## 12. Knowledge Check

### Question 1

What is the purpose of the Copy Data activity?

A. Create a resource group.

B. Transfer data from a configured source to a destination.

C. Schedule a trigger automatically.

### Question 2

What does the source dataset identify?

A. The source data location and format.

B. The SQL database administrator.

C. The pipeline's execution history.

### Question 3

What does recursive file selection allow?

A. It automatically creates all missing containers.

B. It can include matching files in nested directories under the configured source scope.

C. It skips all nested folders.

### Question 4

Why must destination naming be reviewed when copying multiple files?

A. To avoid unintended overwrites or incorrect output paths.

B. To disable authentication.

C. To prevent Data Factory from running.

### Answers

1. B
2. A
3. B
4. A

## 13. Practical Exercise

Using the existing project or a separate development environment:

- [ ] Locate `PL_Copy_Customers`.
- [ ] Inspect its source and sink datasets.
- [ ] Verify the source file exists in `raw`.
- [ ] Validate the pipeline.
- [ ] Run a development test if appropriate.
- [ ] Confirm the output in `processed`.
- [ ] Compare the source and destination contents.
- [ ] Record the outcome and any troubleshooting steps.

Do not rerun a working production workflow simply to complete this exercise. Use a controlled test scenario when needed.

## Summary

In this lesson, we covered the basic Copy Data pipeline and the configuration required to move files from ADLS Gen2 `raw` to `processed`.

You learned how to:

- Create or inspect a Copy Data activity.
- Configure source and destination datasets.
- Validate and debug a pipeline.
- Verify copied output.
- Understand recursive file selection and wildcard filtering.
- Review retries and troubleshoot copy failures.

The next lesson is **Lesson 8 — Metadata-Driven File Ingestion**, where we will examine file discovery, Get Metadata, ForEach, parameterized file processing, and the general file-ingestion pipeline.

**Key principle:** A successful copy requires the correct source, the correct destination, sufficient permissions, and verification of the resulting data.
