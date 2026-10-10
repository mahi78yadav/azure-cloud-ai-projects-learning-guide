# Lesson 09 — Implement Incremental Loading in Azure Data Factory

## 1. Learning Objectives

By the end of this lesson, you should be able to:

- Explain incremental loading and why it is required.
- Understand the purpose of a watermark.
- Explain the role of the `dbo.ETL_Watermark` table.
- Understand the activity sequence used in the ADF pipeline.
- Explain why the watermark must be updated only after successful processing.
- Validate pipeline execution and troubleshoot common problems.

## 2. Business Scenario

Assume a retail business receives new files regularly in its Azure Data Lake Storage Gen2 account.

The source files are stored in the `raw` container. Processed files are written to the `processed` container.

Processing every historical file repeatedly can waste time, increase processing costs, and create duplicate outputs if the pipeline is not designed carefully.

Incremental loading helps the pipeline track its processing progress using a watermark.

A watermark is a stored value that represents a processing boundary, such as a timestamp or sequence number.

**Important:** A watermark does not automatically make a pipeline incremental. The pipeline must use that value to determine what should be processed, and the watermark must be updated correctly after successful processing.

## 3. Architecture

The relevant resources in this project are:

| Component | Name |
|---|---|
| Azure Data Factory | `adf-retail-data-dev-1` |
| Azure Data Lake Storage Gen2 | `stretaildata202610` |
| Source container | `raw` |
| Target container | `processed` |
| Azure SQL database | `sqldb-retail-dev` |
| Watermark table | `dbo.ETL_Watermark` |
| Main ingestion pipeline | `PL_Copy_All_Files` |

The Azure SQL server is:

`sql-retail-data-dev-2026.database.windows.net`

The watermark table is stored in Azure SQL Database and is accessed by ADF through the SQL linked service.

## 4. Understand the Watermark Mechanism

Consider a pipeline that tracks the last successfully processed timestamp.

For example:

| Run | Previous watermark | New processing boundary |
|---|---|---|
| Run 1 | 2026-10-09 00:00:00 | 2026-10-10 00:00:00 |
| Run 2 | 2026-10-10 00:00:00 | 2026-10-11 00:00:00 |

These timestamps are illustrative examples, not the actual values from the project.

The pipeline records the boundary for the next run and updates the stored watermark only after the processing step succeeds.

A common logical approach is:

1. Read the previously stored watermark.
2. Capture the current processing boundary.
3. Select or identify data within the intended processing window.
4. Process that data.
5. Update the watermark after successful processing.

The exact filtering expression depends on the source and the watermark column.

### Why use a watermark?

Without a watermark, a pipeline may repeatedly scan or process the same data.

With a correctly implemented watermark, the pipeline can track its processing progress and identify the next range of data to process.

## 5. Understand the Watermark Table

The project uses the SQL table:

`dbo.ETL_Watermark`

The table stores the watermark information used by the pipeline.

A typical watermark table may contain columns such as:

| Column | Purpose |
|---|---|
| `TableName` or `PipelineName` | Identifies the process being tracked |
| `WatermarkValue` | Stores the last successful processing boundary |
| `LastUpdated` | Records when the watermark was updated |

These are illustrative column names. **Inspect the actual table schema before using or changing any column names.** The implementation may use different names or additional fields.

To inspect the table in SQL Server Management Studio or the Azure portal Query editor, run:

```sql
SELECT *
FROM dbo.ETL_Watermark;
```

To inspect the table's column definitions, run:

```sql
SELECT
    COLUMN_NAME,
    DATA_TYPE,
    IS_NULLABLE
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_SCHEMA = 'dbo'
  AND TABLE_NAME = 'ETL_Watermark'
ORDER BY ORDINAL_POSITION;
```

Do not recreate the table if it already exists and is being used by the pipeline.

## 6. Understand the ADF Pipeline Sequence

The project's incremental-loading flow uses these activities:

`Set_End_Watermark → Get_Watermark → Copy_Raw_To_Processed → Update_Watermark`

Each activity has a specific responsibility.

### Activity 1 — Set_End_Watermark

Purpose:

- Capture the end boundary for the current run.
- Keep the processing boundary consistent throughout the run.
- Provide a value that can be stored after successful processing.

For example, the pipeline might capture the current timestamp at the beginning of a run.

Capturing the boundary once is preferable to recalculating it independently at different stages, because different timestamps could otherwise produce inconsistent processing windows.

The actual expression and storage mechanism depend on the pipeline configuration.

### Activity 2 — Get_Watermark

Purpose:

- Retrieve the existing watermark from `dbo.ETL_Watermark`.
- Make the stored processing boundary available to the pipeline.

The previous watermark represents the point from which the next processing window is determined.

Verify the following:

- The SQL linked service works.
- The activity references the correct watermark table.
- The query or lookup returns the expected value.
- The returned value is available to downstream activities.

### Activity 3 — Copy_Raw_To_Processed

Purpose:

- Copy the intended source data from the `raw` container to the `processed` container.
- Use the configured source and sink settings.
- Respect the intended processing window if the source and pipeline configuration support timestamp-based filtering.

The project also has a recursive wildcard CSV-copy pattern. That pattern is useful for copying files across folders, but **recursive wildcard copying alone is not watermark-based incremental loading**.

For a file-based solution, the pipeline must have a clear rule for identifying files that belong to the current processing window. That rule could use file metadata, a manifest, a control table, or another supported mechanism.

If the current Copy Activity simply copies every matching CSV file, do not claim that it filters files using the watermark unless you have verified that configuration.

### Activity 4 — Update_Watermark

Purpose:

- Persist the new processing boundary after the copy activity succeeds.
- Ensure that the next run starts from the latest successfully recorded boundary.

The update must depend on the successful completion of the processing step.

Do not update the watermark unconditionally or before processing succeeds. Doing so can cause data to be skipped if the copy operation fails.

## 7. Configure Activity Dependencies Correctly

The intended logical dependency is:

1. Capture the end boundary.
2. Read the stored watermark.
3. Process the intended source data.
4. Update the stored watermark only after successful processing.

In ADF Studio:

1. Open the relevant pipeline.
2. Select the watermark and copy activities.
3. Inspect the dependency arrows between activities.
4. Confirm that the watermark update depends on the successful completion of `Copy_Raw_To_Processed`.
5. Confirm that the update does not run when the copy activity fails.
6. Save and validate the pipeline.

**Implementation note:** The sequence above describes the intended flow recorded for this project. Inspect the existing dependency arrows and expressions before making changes. Do not rebuild a working pipeline simply to match example screenshots or expressions from this handbook.

## 8. Understand Failure Scenarios

### Scenario A — Copy succeeds

Expected behavior:

- The copy operation completes successfully.
- `Update_Watermark` runs.
- The new watermark is stored.
- The next run can use the updated processing boundary.

### Scenario B — Copy fails

Expected behavior:

- The copy operation reports failure.
- `Update_Watermark` does not run.
- The previous successful watermark remains available for a subsequent run.
- The failure is investigated and the pipeline is rerun after the problem is corrected.

This avoids advancing the watermark past data that was not successfully processed.

### Scenario C — Copy succeeds, but the watermark update fails

This scenario requires attention.

The data may have been copied successfully, but the watermark may remain unchanged. A later run could process the same data again.

For this reason, a production-grade pipeline should consider idempotency: rerunning a process should not create incorrect duplicate outputs or corrupt the target.

For file copying, possible safeguards include a clear file-selection rule, predictable target paths, and appropriate overwrite or deduplication behavior. The right approach depends on the business requirements.

### Scenario D — A late-arriving file appears

A file may arrive after the watermark has advanced, even though the file belongs to an earlier business period.

If the pipeline relies only on a strict timestamp boundary, the file could be missed.

Possible design improvements include:

- Reprocessing a small overlapping time window.
- Tracking processed file names or unique file identifiers.
- Using a source manifest or control table.
- Reconciliation to identify missing files.

These are design options to evaluate; do not assume they are implemented in the current project.

## 9. Important Difference: File-Level and Row-Level Incremental Loading

There are different ways to implement incremental processing.

### File-level incremental loading

The pipeline determines which files have already been processed and which files are new or changed.

Possible tracking attributes include:

- File path or file name.
- Last-modified timestamp.
- File size.
- A unique file identifier.
- A processing status stored in a control table.

### Row-level incremental loading

The pipeline filters records based on a column in the source data, such as:

- `ModifiedDate`
- `UpdatedTimestamp`
- A monotonically increasing identifier.

For example, a database extraction may select records where the source modification timestamp falls within a defined processing window.

**These two approaches are not interchangeable.** A timestamp in a control table does not, by itself, establish that individual files or rows are being filtered correctly.

For this project, distinguish the implemented watermark-control flow from any additional file-level or row-level filtering that has not been verified.

## 10. Test the Incremental-Loading Flow

Perform these checks in a development environment.

### Test 1 — Inspect the initial watermark

Run:

```sql
SELECT *
FROM dbo.ETL_Watermark;
```

Record the current value before testing.

### Test 2 — Execute the pipeline

1. Open ADF Studio.
2. Open the pipeline containing the watermark activities.
3. Select **Debug** or use the appropriate published trigger for the test.
4. Open the pipeline run details.
5. Inspect the status and output of each activity.

Confirm that the copy activity processes the expected files.

### Test 3 — Verify the updated watermark

After a successful run, query the table again:

```sql
SELECT *
FROM dbo.ETL_Watermark;
```

Confirm that the relevant watermark value has been updated as expected.

### Test 4 — Verify failure behavior

In a safe development test, use a controlled failure scenario, such as an invalid test source path, only if it can be done without disrupting a shared or scheduled pipeline.

Verify that:

- The copy activity fails.
- The watermark update does not execute.
- The previous watermark remains unchanged.

Restore the valid configuration after testing.

### Test 5 — Verify repeat execution

Run the pipeline again and inspect the output.

Check whether the same files are copied again and whether that behavior matches the design.

A successful pipeline status alone does not prove that incremental selection is correct. Validate the source files, target files, watermark values, and repeat-run behavior together.

## 11. Common Problems and Troubleshooting

| Problem | What to inspect |
|---|---|
| Watermark query returns no rows | Confirm the table, schema, initial record, and query |
| Watermark value is null | Check initialization and null handling |
| Copy processes old files repeatedly | Verify whether file selection actually uses the watermark |
| New files are skipped | Review boundary comparisons, late-arriving files, and watermark timing |
| Watermark advances after failure | Inspect activity dependency conditions |
| Copy succeeds but watermark stays unchanged | Check the update activity, SQL permissions, and activity output |
| Repeated runs create duplicate outputs | Review target paths, overwrite behavior, and idempotency |
| Timestamp comparisons behave unexpectedly | Check time zones, timestamp precision, and inclusive/exclusive boundaries |

## 12. Security and Operational Practices

- Give the ADF managed identity only the permissions it requires.
- Restrict SQL permissions to the relevant objects and operations.
- Avoid placing credentials directly in pipeline expressions or source control.
- Review pipeline run history and activity output when troubleshooting.
- Use controlled test data before modifying production-like configurations.
- Consider logging the run identifier, start boundary, end boundary, outcome, and processed-file count for operational traceability.

These are recommended practices; verify which controls are already present before documenting them as implemented.

## 13. Interview Questions and Answers

### Q1. What is a watermark in Azure Data Factory?

A watermark is a stored processing boundary, often a timestamp or sequence number, that helps a pipeline determine what data should be processed next.

### Q2. Why do we update the watermark only after successful processing?

If the watermark advances before processing succeeds, a later run may skip data that was never processed successfully.

### Q3. What happens if the copy succeeds but the watermark update fails?

The watermark may remain unchanged, causing the next run to process the same data again. The pipeline should be designed to handle retries safely.

### Q4. Is copying only new files the same as using a watermark?

Not necessarily. New-file detection and watermark tracking are related but different mechanisms. The pipeline must use a reliable rule to identify files or records that belong to the current processing window.

### Q5. What is idempotency?

Idempotency means that repeating an operation produces the intended result without introducing incorrect additional effects, such as duplicate records or inconsistent files.

### Q6. Can a watermark alone guarantee that no data is missed?

No. Late-arriving data, boundary conditions, failed updates, and source changes can create gaps. A robust design needs appropriate tracking, retry handling, and reconciliation.

### Q7. What is the difference between file-level and row-level incremental loading?

File-level processing tracks which files should be processed. Row-level processing filters records based on a source attribute such as a modification timestamp. The source design determines which approach is appropriate.

## 14. Practical Knowledge Check

Before marking this lesson complete, confirm that you can explain each item:

- [ ] I can explain the purpose of `dbo.ETL_Watermark`.
- [ ] I can describe the four activities in the incremental-loading flow.
- [ ] I understand why the watermark update must follow successful processing.
- [ ] I can inspect the actual watermark table schema.
- [ ] I can validate the watermark before and after a pipeline run.
- [ ] I can explain what happens when the copy operation fails.
- [ ] I understand the difference between recursive file copying and watermark-based filtering.
- [ ] I can distinguish file-level incremental processing from row-level incremental processing.
- [ ] I can explain late-arriving data and idempotency.

## 15. Lesson Summary

In this lesson, you learned how a watermark can help track processing progress in Azure Data Factory.

The project uses a watermark table in Azure SQL Database and a pipeline flow that captures a processing boundary, reads the existing watermark, copies data, and updates the watermark after successful processing.

The key engineering principle is:

**Never advance the recorded processing boundary beyond data that has been successfully processed.**

Also remember that the existence of a watermark table does not automatically prove that the Copy Activity selects only new files or records. Verify the actual filtering logic and repeat-run behavior before describing the pipeline as fully incremental.
