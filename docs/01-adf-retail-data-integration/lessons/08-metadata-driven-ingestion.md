# Lesson 8 — Metadata-Driven File Ingestion

## Overview

In the previous lesson, we built a basic Copy Data pipeline to move files from Azure Data Lake Storage Gen2 (ADLS Gen2) `raw` to `processed`.

In this lesson, we will explore how to make the ingestion process reusable so that it can handle multiple files without requiring a separate pipeline or Copy Data activity for every file.

This approach is called **metadata-driven file ingestion**.

Metadata describes information about files and folders, such as file names, paths, sizes, modification times, and child items. Azure Data Factory can use this information to identify files and pass their paths to reusable Copy Data activities.

The existing project contains the pipeline:

`PL_Copy_All_Files`

The implementation uses file and folder discovery, parameterized datasets, and configured copy operations. We will examine the role of each component and explain how they work together.

**Important:** Inspect the existing pipeline before making changes. The exact sequence of activities, expressions, and dependency conditions must match the actual implementation.

## Learning Objectives

By the end of this lesson, you will be able to:

- Explain metadata-driven ingestion.
- Understand the role of Get Metadata and ForEach activities.
- Configure folder metadata retrieval.
- Understand wildcard and recursive file selection.
- Pass dynamic file paths to parameterized datasets.
- Validate multi-file ingestion.
- Troubleshoot common file-discovery and copy errors.

## 1. Why Metadata-Driven Ingestion Is Needed

Consider a retail company receiving customer, product, and order files every day.

A simple pipeline might contain a separate Copy Data activity for each file.

For example:

```text
Copy_Customers
Copy_Products
Copy_Orders
Copy_Stores
```

This approach can work for a small number of fixed files. However, it becomes difficult to maintain when new files or folders are introduced frequently.

A reusable ingestion pipeline can discover matching files and process them dynamically.

### Business benefits

- Less duplicated pipeline logic.
- Easier onboarding of additional files.
- Consistent processing behavior.
- Better maintainability.
- Support for folder-based file ingestion.
- A foundation for metadata-driven configuration.

Metadata-driven ingestion does not automatically guarantee incremental processing or duplicate prevention. Those concerns require additional logic.

## 2. Understand the Architecture

```text
ADLS Gen2: raw
      |
      v
Identify Source Folder
      |
      v
Get Metadata / File Discovery
      |
      v
Select Matching Files
      |
      v
ForEach File
      |
      v
Parameterized Copy Data Activity
      |
      v
ADLS Gen2: processed
      |
      v
Validate Output
```

The diagram shows a common pattern. A pipeline may use either Get Metadata with ForEach or Copy Activity source settings that directly support wildcard and recursive selection.

These are related but different approaches. Use the approach that matches the existing pipeline.

## 3. Understand File and Folder Metadata

Get Metadata retrieves information about a configured file or folder.

Depending on the selected connector and object, supported metadata fields may include:

- Name.
- Type.
- Size.
- Last modified time.
- Child items.
- Other supported properties.

For a folder, the `childItems` field can provide the names and types of immediate child files and directories.

### Important limitation

Get Metadata is not a general-purpose recursive directory crawler. Retrieving a folder's immediate children does not automatically return every file in every nested subfolder.

If recursive traversal is required using Get Metadata, the pipeline must implement the required traversal logic. Alternatively, a Copy Data activity may use supported recursive and wildcard settings directly.

## 4. Inspect the Existing Pipeline

Before implementing anything, inspect the existing pipeline.

1. Open the Azure portal.
2. Open `adf-retail-data-dev-1`.
3. Select **Launch Studio**.
4. Open **Author**.
5. Open `PL_Copy_All_Files`.
6. Review its activities and their dependency arrows.
7. Select each relevant activity and inspect its settings.
8. Record the actual dataset names, parameters, paths, and file-selection configuration.

Look for:

- Get Metadata activities.
- ForEach activities.
- Copy Data activities.
- Dataset parameters.
- Wildcard settings.
- Recursive traversal settings.
- Success and failure dependencies.

Do not add duplicate activities if the existing pipeline already implements the required behavior.

## 5. Configure Get Metadata for a Source Folder

This section explains a common implementation pattern for a fresh development pipeline.

### Step 1 — Add Get Metadata

1. Create a new pipeline or open a test pipeline.
2. Add a **Get Metadata** activity.
3. Give it a meaningful name, such as `Get_Source_Folder_Metadata`.
4. Select the appropriate folder dataset.

For example, the dataset could be `DS_Source_Folder` if that dataset matches the intended folder and is configured appropriately.

### Step 2 — Select the metadata fields

In the activity's field list, select `childItems` when the objective is to obtain the immediate contents of the folder.

Other fields may be selected if required by the implementation.

### Step 3 — Validate the folder path

Confirm that:

- The folder exists.
- The linked service can access it.
- The dataset points to the intended container and directory.
- The selected metadata field is supported for the source.

### Step 4 — Debug the activity

Run a controlled test and inspect the activity output.

A conceptual output for a folder might look like this:

```json
{
  "childItems": [
    {
      "name": "customers.csv",
      "type": "File"
    },
    {
      "name": "products.csv",
      "type": "File"
    },
    {
      "name": "orders",
      "type": "Folder"
    }
  ]
}
```

This is an illustrative example, not output captured from the existing project.

The returned items can be used by downstream activities.

## 6. Use ForEach to Process Multiple Items

A ForEach activity iterates over an array of items.

If a Get Metadata activity returns `childItems`, a typical ForEach expression is:

```text
@activity('Get_Source_Folder_Metadata').output.childItems
```

The activity name in the expression must match the actual name in the pipeline.

### Configure ForEach

1. Add a ForEach activity.
2. Set its **Items** expression to the appropriate array.
3. Open the ForEach activity's inner activities.
4. Add a Copy Data activity inside it.
5. Configure the Copy Data activity to use the current item.

For a simple flat folder containing files, the current item's `name` property can supply the file name.

For example:

```text
@item().name
```

The dataset must have a matching file-name parameter, and the pipeline must pass the expression into that parameter.

### Filter folders from files

If `childItems` includes both files and folders, do not assume every item is a file.

A pipeline should filter or branch appropriately before attempting to copy an item as a file.

A flat-folder example may process only items whose type is `File`. Nested folders require additional traversal logic or an alternative recursive copy design.

## 7. Pass Dynamic File Names to Datasets

The existing project includes reusable datasets:

- `DS_Source_File`
- `DS_Target_File`

These can support dynamic file paths when configured with the required parameters.

Suppose a source dataset defines the parameter `fileName`.

The dataset's file-name property can reference:

```text
@dataset().fileName
```

Inside a ForEach activity, the pipeline can supply:

```text
@item().name
```

These expressions have different purposes:

| Expression | Meaning |
|---|---|
| `@item().name` | Gets the current item's name inside the loop |
| `@dataset().fileName` | Reads the file-name parameter inside the dataset |

The pipeline connects them by passing the current item's name to the dataset parameter.

The parameter names and dataset configuration must match the actual implementation.

## 8. Configure the Copy Data Activity

Inside the loop, configure the Copy Data activity to use the reusable source and destination datasets.

### Source

- Select `DS_Source_File`, if it is the intended dataset.
- Pass the current file name or path through its parameter.
- Confirm the source folder is correct.
- Confirm the file format is correct.

### Sink

- Select `DS_Target_File`, if it is the intended dataset.
- Pass the intended destination file name or relative path.
- Confirm that the destination container is `processed`.
- Ensure multiple source files will not overwrite the same target unintentionally.

### Expected behavior

For a source folder containing several CSV files, the loop processes each eligible item using the same Copy Data activity configuration.

The number of iterations depends on the array returned to ForEach and any filters applied by the pipeline.

## 9. Understand Wildcard and Recursive Copying

Your project also uses a general file-copy design with wildcard and recursive settings.

A wildcard such as:

```text
*.csv
```

selects files matching the CSV file-name pattern within the configured scope.

When recursive traversal is enabled and supported for the configured source, matching files in nested directories can also be included.

For example:

```text
raw/
├── customers/
│   ├── customers_01.csv
│   └── customers_02.csv
├── products/
│   └── products_01.csv
└── orders/
    └── orders_01.csv
```

The combination of source scope, wildcard pattern, and recursive settings determines which files are selected.

### Wildcard versus ForEach

| Approach | Typical use |
|---|---|
| Wildcard and recursive Copy Activity | Copy matching files under a configured source scope |
| Get Metadata plus ForEach | Inspect discovered items and process each item individually |
| Nested traversal with metadata | Recursively discover folders when explicit traversal logic is required |

Choose the approach that fits the business requirement. Avoid adding ForEach and Get Metadata merely because they are common in metadata-driven pipelines.

## 10. Understand File Paths and Folder Preservation

When copying multiple files, the destination design must specify how file names and folder paths are constructed.

For example, if the source is:

```text
raw/customers/customers_01.csv
```

The desired destination might be:

```text
processed/customers/customers_01.csv
```

However, folder preservation depends on the Copy Activity source and sink configuration, dataset paths, and any dynamic path expressions.

Do not assume that a wildcard copy automatically preserves every directory level in the intended form.

Validate the actual output structure with files in multiple folders before relying on it.

## 11. Validate Metadata-Driven Ingestion

Use a controlled set of test files.

### Test 1 — One source file

1. Place a CSV file in the configured source folder.
2. Run the pipeline.
3. Confirm that the expected file is copied.

### Test 2 — Multiple source files

1. Add two or more distinct CSV files.
2. Run the pipeline.
3. Verify that every eligible file is processed.
4. Confirm that the files have distinct, correct destination names.

### Test 3 — Nested folder

1. Place a test file in a nested source folder.
2. Run the pipeline.
3. Confirm whether the configured traversal includes it.
4. Check the destination hierarchy.

### Test 4 — Unsupported item or missing path

Use a controlled test environment to check the behavior when a folder is missing or an unexpected item is encountered.

Review the resulting error and confirm that it can be diagnosed.

### Validation checklist

- [ ] Source folder exists.
- [ ] Metadata activity returns the expected items.
- [ ] ForEach iterates over the intended collection, if used.
- [ ] Folder items are handled correctly.
- [ ] Dynamic file parameters resolve to the expected paths.
- [ ] All eligible files are copied.
- [ ] Destination names are unique where required.
- [ ] Folder hierarchy matches the intended design.
- [ ] Pipeline execution details are available in Monitor.

## 12. Common Problems and Troubleshooting

| Problem | Possible cause | What to check |
|---|---|---|
| `childItems` is missing | Incorrect field selection or wrong source type | Inspect Get Metadata configuration |
| Get Metadata fails | Missing path or insufficient permissions | Verify the dataset and storage access |
| ForEach executes zero times | The returned array is empty or the expression is incorrect | Inspect activity output and the Items expression |
| Folder is treated as a file | Child items include directories | Filter items or implement folder traversal |
| Copy activity reads the wrong file | Incorrect parameter mapping | Inspect the resolved dataset path |
| Nested files are not discovered | Only immediate children were retrieved | Implement recursive traversal or use supported recursive copy settings |
| Destination files overwrite each other | The target name is fixed | Use a unique dynamic destination path |
| Some files are skipped | Incorrect wildcard or source scope | Review file-selection configuration |

## 13. Knowledge Check

### Question 1

What does Get Metadata do?

A. Copies files to the destination.

B. Retrieves supported information about a configured file or folder.

C. Creates a SQL database.

### Question 2

What is the purpose of ForEach?

A. Iterate over a collection of items.

B. Grant storage permissions.

C. Create an Azure subscription.

### Question 3

Does retrieving a folder's `childItems` automatically discover every file in all nested subfolders?

A. Yes, always.

B. No, additional traversal logic or a suitable recursive copy configuration may be required.

C. Only when the folder is named `raw`.

### Question 4

What does `@item().name` represent inside a ForEach loop?

A. The name of the current item being processed.

B. The storage account name.

C. The name of the linked service.

### Answers

1. B
2. A
3. B
4. A

## Summary

In this lesson, we explored metadata-driven ingestion and the components used to process multiple files through reusable pipeline logic.

You learned how to:

- Retrieve file and folder metadata.
- Use ForEach to iterate over discovered items.
- Pass dynamic file names into parameterized datasets.
- Configure Copy Data activities for multiple files.
- Understand wildcard and recursive copy behavior.
- Validate the results and troubleshoot common errors.

The next lesson is **Lesson 9 — Implement Incremental Loading**, where we will examine watermark management, processing boundaries, success-dependent updates, and the limitations of timestamp-based file selection.

**Key principle:** Metadata tells the pipeline what is available; pipeline logic determines what to process and how to process it.
