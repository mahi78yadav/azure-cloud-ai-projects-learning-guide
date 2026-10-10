# Lesson 6 — Create and Configure Datasets

## Overview

In the previous lesson, we configured the linked services that connect Azure Data Factory to Azure Data Lake Storage Gen2 and Azure SQL Database.

In this lesson, we will learn how to create datasets that identify the source files, destination files, folders, and SQL control data used by the pipelines.

A linked service defines the connection. A dataset describes the data location or structure that an activity uses.

We will also learn how parameters make datasets reusable across different files and folders.

**Important:** Your existing project already contains several datasets. Inspect and document their actual configuration before creating new ones.

## Learning Objectives

By the end of this lesson, you will be able to:

- Explain the difference between linked services and datasets.
- Create an ADLS Gen2 dataset for CSV files.
- Create source and destination datasets.
- Configure folder paths and file names.
- Understand dataset parameters and dynamic content.
- Create or inspect datasets used for folder discovery and SQL control operations.
- Test datasets and troubleshoot common configuration errors.

## 1. Understand Datasets

A dataset identifies the data that an Azure Data Factory activity reads or writes.

Examples include:

- A CSV file in ADLS Gen2.
- A folder containing multiple files.
- A SQL table containing watermark values.

A dataset generally references a linked service and defines the relevant data location and format.

### Linked service versus dataset

| Component | Purpose | Example |
|---|---|---|
| Linked service | Defines the connection | `LS_ADLS_Retail` |
| Dataset | Identifies the data location | `DS_Source_Customers` |
| Activity | Performs an operation | Copy Data |
| Pipeline | Coordinates activities | `PL_Copy_Customers` |

Think of the linked service as the connection to a storage account and the dataset as the particular file or folder within that account.

## 2. Identify the Existing Datasets

Your project uses these dataset names:

| Dataset | Intended purpose |
|---|---|
| `DS_Source_Customers` | Source customer data |
| `DS_Target_Customers` | Destination customer data |
| `DS_Source_File` | Reusable source-file definition |
| `DS_Target_File` | Reusable destination-file definition |
| `DS_Source_Folder` | Source-folder operations |
| `DS_Source_Root` | Source root location |
| `DS_Target_Root` | Destination root location |
| `DS_Watermark` | SQL watermark control data |

The exact parameters and configurations should be confirmed from each dataset in Data Factory Studio.

Not every dataset is necessarily used by every pipeline.

## 3. Create a Source CSV Dataset

Follow these steps when creating a new dataset or reproducing the solution in a separate environment.

### Step 1 — Open Data Factory Studio

1. Open the Azure portal.
2. Open `adf-retail-data-dev-1`.
3. Select **Launch Studio**.
4. Open the **Author** area.
5. Locate **Datasets**.
6. Select **New dataset**.

### Step 2 — Select the connector

1. Search for **Azure Data Lake Storage Gen2**.
2. Select the connector.
3. Select **Continue**.
4. Choose the **DelimitedText** format for a CSV file.
5. Select **Continue**.

The exact labels may vary slightly depending on the Studio interface.

### Step 3 — Configure the dataset

Use the following example values:

| Setting | Example |
|---|---|
| Name | `DS_Source_Customers` |
| Linked service | `LS_ADLS_Retail` |
| File system/container | `raw` |
| Directory | `customers` |
| File | `customers.csv` |
| First row as header | Enabled, if the file contains headers |
| Column delimiter | Comma, for a standard CSV |

The directory and file values must match the actual location and format of your source data.

For a fresh environment, create or upload the sample file before attempting schema import.

### Step 4 — Import the schema

If the dataset editor offers schema import:

1. Select the appropriate import option.
2. Choose a valid sample file.
3. Allow Data Factory to infer the column names and data types.
4. Review the resulting schema.
5. Save the dataset.

Schema import may fail if the path is incorrect, the file does not exist, permissions are missing, or the file format is invalid.

A successful schema import is useful, but it does not guarantee that all future files have the same schema.

## 4. Create a Destination CSV Dataset

A destination dataset identifies where copied files should be written.

### Step 1 — Create the dataset

1. Select **New dataset**.
2. Choose **Azure Data Lake Storage Gen2**.
3. Select the **DelimitedText** format.
4. Select **Continue**.

### Step 2 — Configure the destination

Use these example settings:

| Setting | Example |
|---|---|
| Name | `DS_Target_Customers` |
| Linked service | `LS_ADLS_Retail` |
| File system/container | `processed` |
| Directory | `customers` |
| File | `customers.csv` |

Confirm the intended folder and file name before saving.

### Important design consideration

A fixed destination file name is suitable for a fixed-file copy scenario.

For a pipeline that copies multiple files, the destination path and file name may need to be supplied dynamically. Otherwise, multiple source files could overwrite the same destination file, depending on the Copy Activity settings.

## 5. Understand Parameterized Datasets

A parameterized dataset can be reused for different files or folders without creating a separate dataset for every input.

For example, instead of creating individual datasets for every customer CSV, the dataset can accept:

- A folder name.
- A file name.

### Example parameters

Suppose a reusable dataset has the following parameters:

| Parameter | Example value |
|---|---|
| `folderName` | `customers` |
| `fileName` | `customers.csv` |

The dataset's directory and file properties can reference these parameters.

Illustrative expressions:

```text
@dataset().folderName
```

```text
@dataset().fileName
```

The exact expression must match the parameter names defined in the dataset.

### Why use parameters?

Without parameters, you may need many similar datasets.

With parameters, the same dataset can be reused for customer, product, and order files when their connector and format are compatible.

Parameters improve reuse and maintainability, but they do not automatically discover files or perform copying. Those responsibilities belong to the pipeline activities and their configuration.

## 6. Create a Reusable Source-File Dataset

Your project includes:

`DS_Source_File`

This dataset is intended to support reusable file-level operations.

For a parameterized CSV dataset:

1. Create an ADLS Gen2 DelimitedText dataset.
2. Select `LS_ADLS_Retail`.
3. Define the required folder and file parameters.
4. Reference the parameters in the directory and file properties.
5. Configure the correct CSV format.
6. Save the dataset.

If your existing dataset uses different parameter names, preserve those names when documenting the implementation.

### Example runtime values

```text
folderName = customers
fileName   = customers_20261010.csv
```

The pipeline can supply different values for different files.

## 7. Create a Reusable Destination-File Dataset

Your project includes:

`DS_Target_File`

This dataset supports destination paths for file-copy operations.

A corresponding configuration may use:

| Parameter | Example |
|---|---|
| `folderName` | `customers` |
| `fileName` | `customers_20261010.csv` |

The dataset references these parameters in the destination path.

When the pipeline iterates through a list of source files, it can pass the appropriate destination folder and file name to the dataset.

**Important:** Confirm the actual parameter names and the Copy Activity's destination behavior. Parameterization alone does not guarantee that the source directory structure is preserved.

## 8. Understand Folder Datasets

The project also contains:

- `DS_Source_Folder`
- `DS_Source_Root`
- `DS_Target_Root`

These datasets represent folder-oriented and root-path configuration used by the ingestion workflow.

### Source folder dataset

`DS_Source_Folder` is intended for operations involving a source folder, such as retrieving folder metadata or discovering child items.

A folder dataset is different from a file dataset because the activity may need to operate on a collection of files rather than one specific file.

### Root datasets

Root datasets provide reusable references to the top-level source and destination locations.

For example:

```text
raw/
```

and:

```text
processed/
```

The actual dataset configuration determines whether the container, directory, or another path component is stored in the linked service, dataset, or pipeline parameters.

### Recursive processing

When the pipeline is configured for recursive file discovery, it can process matching files in nested folders.

The recursive behavior must be configured in the relevant activity or connector properties. Creating a folder dataset by itself does not enable recursive copying.

## 9. Create or Inspect the SQL Watermark Dataset

Your project includes:

`DS_Watermark`

This dataset is associated with the SQL control data used by the incremental-loading pipeline.

### Configuration

1. Open **Author → Datasets**.
2. Select **New dataset**, if creating it.
3. Choose the Azure SQL Database connector.
4. Select the `LS_AzureSQL_Retail` linked service.
5. Select the configured watermark table or query-based source, depending on the implementation.
6. Save the dataset.

The existing control table is:

`dbo.ETL_Watermark`

The pipeline uses the watermark value to establish its processing boundary.

### Important distinction

The dataset identifies the SQL data source. It does not itself read or update the watermark.

The relevant Lookup, Script, Stored Procedure, or other configured activity performs the database operation.

When documenting the existing implementation, record the actual table, query, and parameter configuration rather than assuming a particular SQL activity is used.

## 10. Validate the Datasets

For each dataset, check the following:

- [ ] Correct linked service is selected.
- [ ] Correct container or database is selected.
- [ ] Directory path is correct.
- [ ] File name is correct where applicable.
- [ ] File format matches the source or destination.
- [ ] Header and delimiter settings are appropriate.
- [ ] Dataset parameters are defined correctly.
- [ ] Dynamic expressions reference valid parameter names.
- [ ] SQL datasets point to the intended control data.
- [ ] The dataset is saved and available to the intended pipeline.

Use **Preview data** or schema import where supported and appropriate. These operations may require additional access and can fail if the source is unavailable.

## 11. Common Dataset Errors

| Error | Possible cause | What to check |
|---|---|---|
| Path not found | Incorrect container, directory, or file name | Verify the actual storage path |
| Schema import fails | Missing file, invalid format, or insufficient access | Check the file and linked service |
| Parameter expression fails | Parameter is missing or misspelled | Compare expressions with dataset parameters |
| Copy reads the wrong file | Incorrect runtime parameter values | Inspect the activity input and resolved path |
| Multiple files overwrite one target | Destination file name is fixed | Use the intended dynamic destination configuration |
| SQL dataset fails | Wrong database, table, or linked service | Validate the SQL connection and object name |
| Preview fails despite correct path | Authentication, network, or format issue | Inspect the detailed error and test connectivity |

## 12. Knowledge Check

### Question 1

What does a dataset identify?

A. The data location or structure used by an activity.

B. The complete pipeline schedule.

C. The permissions of every Azure resource.

### Question 2

Why use a parameterized dataset?

A. To automatically grant access to all files.

B. To reuse a dataset for different files or folders.

C. To replace linked services.

### Question 3

Does creating a folder dataset automatically enable recursive copying?

A. Yes.

B. No, the relevant activity must be configured for the required behavior.

C. Yes, if the folder is named `raw`.

### Question 4

Which component performs the actual file copy?

A. Dataset.

B. Linked service.

C. Copy Data activity.

### Answers

1. A
2. B
3. B
4. C

## Summary

In this lesson, we covered the creation and configuration of datasets for ADLS Gen2 and Azure SQL Database.

You learned how to:

- Create source and destination CSV datasets.
- Configure container, directory, and file properties.
- Use parameters for reusable file datasets.
- Understand folder-oriented and root datasets.
- Configure or inspect the watermark dataset.
- Validate datasets and troubleshoot common errors.

The next lesson is **Lesson 7 — Build the Copy Data Pipeline**, where we will connect datasets through Copy Data activities and validate that files move from `raw` to `processed`.

**Key principle:** Linked services define the connection, datasets define the data location, and activities perform the operation.
