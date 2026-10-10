# Lesson 10 — Error Handling, Retries, and Monitoring in Azure Data Factory

## 1. Learning Objectives

By the end of this lesson, you should be able to:

- Explain why error handling is important in ETL pipelines.
- Understand retry policies in Azure Data Factory (ADF).
- Explain how pipeline failures are logged.
- Understand success and failure dependency conditions.
- Troubleshoot failed pipeline activities.
- Explain the difference between retrying an activity and logging an error.
- Describe how to monitor pipeline executions.

## 2. Business Scenario

Consider a retail data integration platform that copies files from Azure Data Lake Storage Gen2 into a processed area.

A pipeline can fail for several reasons:

- A source file is missing.
- A storage path is incorrect.
- The storage account is temporarily unavailable.
- The Azure SQL Database connection fails.
- The pipeline's managed identity lacks the required permissions.
- A SQL query or stored procedure fails.
- A file has an unexpected format.

Without appropriate error handling, a failure may be difficult to diagnose. A pipeline might also stop before recording enough information for support teams to investigate the problem.

The goal is to make failures visible, support safe retries, and prevent unsuccessful processing from being reported as successful.

## 3. Project Resources

The relevant resources in this project are:

| Component | Name |
|---|---|
| Azure Data Factory | `adf-retail-data-dev-1` |
| ADLS Gen2 storage account | `stretaildata202610` |
| Source container | `raw` |
| Target container | `processed` |
| Azure SQL Database | `sqldb-retail-dev` |
| Error log table | `dbo.ETL_ErrorLog` |
| Main ingestion pipeline | `PL_Copy_All_Files` |

The project uses an Azure SQL error log table and a failure-handling path. It also has a configured retry policy for relevant activities.

Before changing an existing pipeline, inspect its current activities, dependencies, and settings. Do not recreate a working configuration just to match an example.

## 4. Understand the Types of Error Handling

Error handling in ADF involves several related controls.

| Control | Purpose |
|---|---|
| Retry policy | Reattempts an activity after a failure |
| Failure dependency | Runs a downstream activity when an upstream activity fails |
| Error logging | Stores details about a failure for investigation |
| Pipeline monitoring | Shows activity status, duration, and execution details |
| Alerting | Notifies a support team when a defined condition occurs |
| Recovery and rerun | Restores processing after the underlying problem is corrected |

These controls solve different problems. A retry does not replace error logging, and logging an error does not automatically recover the pipeline.

## 5. Understand Retry Policies

A retry policy tells ADF whether to attempt a failed activity again.

The project uses the following retry settings for the configured activity:

| Setting | Value |
|---|---:|
| Retry count | 2 |
| Retry interval | 30 seconds |

A retry count of 2 means ADF can make up to two retries after the initial failed attempt. For an activity that fails on every attempt, this can result in up to three attempts in total.

### Example

Suppose a Copy Activity temporarily cannot access the storage service.

1. ADF starts the Copy Activity.
2. The first attempt fails.
3. ADF waits for the configured interval.
4. ADF retries the activity.
5. If the retry fails, ADF can make the second retry.
6. If all attempts fail, the activity ultimately reports failure.

The exact retry behavior and activity status should be confirmed in the pipeline run details.

### When are retries useful?

Retries are useful for temporary problems, such as:

- Transient network issues.
- Temporary service unavailability.
- Intermittent throttling.

Retries generally do not solve persistent problems, such as:

- An invalid file path.
- A permanently missing file.
- Incorrect SQL syntax.
- Missing permissions.
- An incompatible source schema.

If an error is caused by a persistent configuration problem, investigate and correct the root cause rather than relying on repeated retries.

## 6. Configure or Inspect a Retry Policy

To inspect the retry configuration in ADF Studio:

1. Open the Azure portal.
2. Open `adf-retail-data-dev-1`.
3. Select **Launch Studio**.
4. Open the relevant pipeline.
5. Select the activity you want to inspect.
6. Open its **General** or activity settings tab, depending on the activity type and Studio interface.
7. Locate the retry and retry interval settings.
8. Confirm the configured values.
9. Save and publish changes only if changes are required.

For this project, verify the existing retry count of `2` and retry interval of `30` seconds where configured.

**Important:** Retry settings can be configured per activity. Do not assume that every activity in every pipeline has the same retry policy.

## 7. Understand Success and Failure Dependencies

ADF activities can be connected using dependency conditions.

The common dependency conditions are:

- **Succeeded** — run the downstream activity when the upstream activity succeeds.
- **Failed** — run the downstream activity when the upstream activity fails.
- **Completed** — run the downstream activity when the upstream activity finishes, regardless of success or failure.
- **Skipped** — run the downstream activity when the upstream activity is skipped.

The condition selected on the dependency determines when the downstream activity runs.

### Example: Copy and error logging

The intended logical flow is:

```text
Copy_Raw_To_Processed
        |
        |-- Succeeded --> Continue normal processing
        |
        |-- Failed ----> Log_Error
```

In this example, `Log_Error` is configured to run when the copy activity fails.

The error path should record the failure and preserve the correct overall outcome. Successfully writing an error record must not make the original data-processing failure appear successful.

### Inspect the actual dependency

1. Open the pipeline in ADF Studio.
2. Select the Copy Activity.
3. Inspect the outgoing dependency arrows.
4. Confirm that the error logging activity is connected using the intended failure condition.
5. Check whether the error logging activity itself can fail.
6. Verify the final pipeline status after a controlled test.

The project contains a `Log_Error` failure-handling path. Inspect its actual dependencies before changing the implementation.

## 8. Understand the Error Log Table

The project uses the SQL table:

`dbo.ETL_ErrorLog`

The table is intended to retain failure information for troubleshooting.

A typical ETL error log may contain the following fields:

| Field | Purpose |
|---|---|
| Pipeline name | Identifies the pipeline |
| Activity name | Identifies the activity that failed |
| Run ID | Helps locate the relevant ADF execution |
| Error message | Describes the failure |
| Error time | Records when the error was logged |
| Source path | Identifies the affected source, where applicable |

These are common design fields, not a guarantee of the actual schema in your project.

### Inspect the existing table

Run this query in your Azure SQL Database query editor or SQL Server Management Studio:

```sql
SELECT *
FROM dbo.ETL_ErrorLog;
```

To inspect the column definitions:

```sql
SELECT
    COLUMN_NAME,
    DATA_TYPE,
    IS_NULLABLE
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_SCHEMA = 'dbo'
  AND TABLE_NAME = 'ETL_ErrorLog'
ORDER BY ORDINAL_POSITION;
```

Use the actual column names when investigating records or modifying the logging activity.

Do not drop or recreate an existing error log table simply to match a sample schema.

## 9. Understand What Should Be Logged

A useful error record should help answer four questions:

1. **What failed?** — pipeline and activity.
2. **When did it fail?** — timestamp.
3. **Which execution failed?** — run ID.
4. **Why did it fail?** — error details.

Where available and appropriate, logging can also include a source file path or business process identifier.

Avoid recording passwords, access tokens, connection strings containing secrets, or unnecessary sensitive data in error messages.

### Why is the run ID important?

ADF run history contains the execution details. The run ID provides a way to connect a log record with the corresponding pipeline execution.

When investigating a failure, use the run ID to locate the relevant execution and inspect the activity's error output.

## 10. Test Error Handling Safely

A controlled test is more useful than assuming the failure path works because the pipeline has been published.

Perform the test in a development environment.

### Test 1 — Run the pipeline successfully

1. Open the pipeline in ADF Studio.
2. Run it using Debug or the appropriate test mechanism.
3. Open the pipeline run details.
4. Confirm that the expected activity succeeds.
5. Check the target output.
6. Confirm that the watermark is updated only when the corresponding processing operation succeeds.

### Test 2 — Trigger a controlled failure

Use an isolated test configuration to make a source path invalid. Do not change a shared production configuration or disrupt a scheduled workload.

1. Record the current watermark value.
2. Configure the test to use an invalid source path.
3. Run the pipeline.
4. Open the activity details.
5. Confirm that the Copy Activity reports failure after its configured retry behavior.
6. Check whether the `Log_Error` activity executes.
7. Inspect the error log table for a corresponding record.
8. Verify that the watermark has not advanced past unsuccessful processing.
9. Restore the valid source configuration.

### Test 3 — Verify the pipeline outcome

Confirm that:

- The original failed activity is visible in the run details.
- The error logging path runs according to its dependency configuration.
- The log contains useful diagnostic information, where logging succeeds.
- The overall execution does not incorrectly represent failed data processing as successful.
- The watermark remains consistent with successful processing.

**Important:** Error logging can fail too. If the database is unavailable, a SQL permission is missing, or the logging query is incorrect, the failure record may not be written. Inspect both the original activity and the logging activity.

## 11. Monitor Pipeline Runs

ADF monitoring helps you understand whether data processing completed as expected.

To inspect pipeline runs:

1. Open ADF Studio.
2. Select **Monitor**.
3. Open the pipeline runs view.
4. Find the relevant pipeline execution.
5. Open its activity run details.
6. Inspect the activity status, duration, error message, and output.
7. Use the run ID to connect the execution with the error log, if applicable.

Common statuses include:

- **Succeeded** — the activity or pipeline completed successfully.
- **Failed** — an activity or pipeline failed.
- **In progress** — execution is still running.
- **Cancelled** — execution was cancelled.
- **Queued** — execution is waiting to run.

The exact statuses displayed depend on the activity and the monitoring view.

### What should you monitor?

| Monitoring item | Why it matters |
|---|---|
| Pipeline status | Identifies successful and failed runs |
| Activity status | Helps isolate the failing step |
| Run duration | Helps detect unusually slow processing |
| Error details | Helps identify the root cause |
| Copy activity output | Helps verify copy behavior and reported counts |
| Watermark values | Helps confirm processing progress |
| Error log records | Helps retain failure information |

## 12. Common Problems and Troubleshooting

| Problem | Investigation |
|---|---|
| Pipeline fails after all retries | Inspect the original error and correct the underlying issue |
| Error logging activity is skipped | Check its dependency condition and upstream activity |
| Error log table has no record | Check whether the logging activity ran, SQL connectivity, permissions, and the insert operation |
| Logging activity fails | Inspect its SQL error separately from the original pipeline error |
| Watermark advances after a failed copy | Inspect the update dependency and activity conditions |
| Pipeline shows success even though processing failed | Review failure-path logic and how the final pipeline status is determined |
| Retries do not help | Determine whether the problem is persistent rather than transient |
| Failure is difficult to correlate with monitoring | Record and use the pipeline run ID |
| The pipeline is slow | Inspect activity duration, file sizes, source availability, and concurrency |

## 13. Production Recommendations

The following practices can improve reliability as the project evolves.

### A. Make retries appropriate for the operation

Use retries for transient errors, not as a substitute for fixing invalid configuration or missing permissions.

### B. Preserve failure visibility

Keep the original failure visible in ADF monitoring even when a separate activity successfully records the error.

### C. Design safe reruns

Consider what happens if a pipeline is rerun after a partial failure. Repeated execution should not produce unintended duplicate or inconsistent output.

### D. Protect the watermark

Update the watermark only after the processing step it represents has succeeded. If a process has several dependent stages, determine exactly which stage must complete before progress can be committed.

### E. Consider alerts

For an operational deployment, consider configuring alerts for repeated failures or other defined conditions. An error log table does not automatically send notifications.

### F. Avoid exposing secrets

Do not store credentials or access tokens in logs, pipeline parameters, source control, or troubleshooting documents.

These recommendations should be recorded as implemented controls only after they have actually been configured and tested.

## 14. Interview Questions and Answers

### Q1. What is error handling in Azure Data Factory?

Error handling is the design of retries, failure dependencies, logging, monitoring, and recovery procedures to identify and manage pipeline failures.

### Q2. What is the difference between a retry and an error log?

A retry attempts an activity again. An error log records details about a failure for investigation. One does not replace the other.

### Q3. What does a retry count of 2 mean?

It permits up to two retries after the initial attempt. If every attempt fails, there can be three total attempts.

### Q4. How can you run a logging activity only when a Copy Activity fails?

Configure the dependency from the Copy Activity to the logging activity using the **Failed** condition.

### Q5. What if the logging activity also fails?

Inspect both failures. A separate logging mechanism or monitoring alert may be needed if the SQL database used for logging is unavailable.

### Q6. Why should the watermark not be updated after a failed copy?

Advancing the watermark may cause the next run to skip data that was not processed successfully.

### Q7. Does a successful error logging activity mean the pipeline succeeded?

No. The logging activity may succeed while the original data-processing activity remains failed. The overall pipeline outcome must accurately reflect the processing result.

### Q8. How do you troubleshoot an ADF pipeline failure?

Inspect the pipeline run, identify the failed activity, review its error output, check dependencies and retry settings, validate source and target connectivity, and correlate the execution with error logs using the run ID.

## 15. Practical Knowledge Check

- [ ] I can explain the purpose of retry policies.
- [ ] I know that a retry count of 2 permits two retries after the initial attempt.
- [ ] I can explain the difference between a success dependency and a failure dependency.
- [ ] I can inspect the `dbo.ETL_ErrorLog` schema.
- [ ] I can find a failed activity in ADF Monitor.
- [ ] I can connect a pipeline run ID with diagnostic records.
- [ ] I can test a failure path safely in a development environment.
- [ ] I understand why a successful logging operation must not hide a data-processing failure.
- [ ] I can explain why the watermark must not advance after unsuccessful processing.
- [ ] I can distinguish implemented controls from recommended production enhancements.

## 16. Lesson Summary

In this lesson, you learned how to handle failures in Azure Data Factory through retry policies, failure dependencies, SQL error logging, and pipeline monitoring.

Your project uses retry settings of two retries with a 30-second interval for the configured activity, an error log table named `dbo.ETL_ErrorLog`, and a `Log_Error` failure-handling path.

The most important principle is:

**A reliable pipeline must make failures visible, record useful diagnostic information, and preserve correct processing state.**

Retries can help with temporary failures, while monitoring and error logs help identify the root cause. Safe recovery also requires careful handling of watermarks and repeated execution.
