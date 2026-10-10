# Lesson 11 — Triggers and Monitoring in Azure Data Factory

## 1. Learning Objectives

By the end of this lesson, you should be able to:

- Explain why Azure Data Factory triggers are used.
- Understand the difference between manual execution and scheduled execution.
- Explain how a schedule trigger works.
- Inspect and manage a published trigger.
- Monitor pipeline runs and activity execution.
- Validate source-to-target data movement.
- Troubleshoot failed or unexpected scheduled executions.

## 2. Business Scenario

A retail business receives new data files regularly. The data integration pipeline should run automatically so that the files are copied and made available for downstream processing.

Running a pipeline manually every time is not practical for a recurring business process.

Azure Data Factory supports triggers that initiate pipeline execution based on defined conditions or schedules.

In this project, a schedule trigger was configured to run the retail ingestion pipeline at 15-minute intervals for testing.

**Important:** A 15-minute interval was used for testing. It should not automatically be treated as the final production schedule.

## 3. Project Resources

| Component | Name |
|---|---|
| Azure Data Factory | `adf-retail-data-dev-1` |
| ADLS Gen2 storage account | `stretaildata202610` |
| Source container | `raw` |
| Target container | `processed` |
| Azure SQL Database | `sqldb-retail-dev` |
| Main ingestion pipeline | `PL_Copy_All_Files` |
| Schedule trigger | `TR_Daily_Retail_Ingestion` |

The trigger name contains the word `Daily`, but its testing configuration was changed to a 15-minute interval. When documenting or reviewing a trigger, rely on its actual published configuration rather than its name.

## 4. Understand Manual and Scheduled Execution

### Manual execution

Manual execution is useful during development and troubleshooting.

You can run a pipeline using Debug or by initiating a pipeline run from the ADF interface.

Typical uses include:

- Testing a newly configured activity.
- Validating source and target connections.
- Checking copy results.
- Investigating a failure.
- Testing a change before enabling a trigger.

### Scheduled execution

Scheduled execution starts a pipeline automatically according to a configured timetable.

Typical uses include:

- Daily ingestion.
- Hourly data processing.
- Periodic file ingestion.
- Recurring data synchronization.

A schedule trigger initiates a pipeline run. It does not, by itself, guarantee that new files exist, that the pipeline succeeds, or that the resulting data is correct.

## 5. Understand Trigger Types

ADF supports several trigger types.

| Trigger type | Purpose |
|---|---|
| Schedule trigger | Starts a pipeline at configured times or intervals |
| Tumbling window trigger | Starts window-based processing with defined intervals and stateful scheduling |
| Event-based trigger | Starts a pipeline when a configured storage event occurs |

For this project, the relevant trigger is a **schedule trigger**.

The other trigger types are useful architectural concepts, but they should not be described as implemented in this project unless they have actually been configured.

## 6. Understand the Schedule Trigger

The project uses:

`TR_Daily_Retail_Ingestion`

The trigger was configured for a 15-minute interval during testing.

The trigger's configuration determines when ADF is allowed to start pipeline runs. The trigger may also have a configured start time, end time, time zone, and enabled or disabled status.

### Why use a 15-minute schedule during testing?

A short interval makes it easier to observe repeated executions and verify that the pipeline can run automatically.

However, a short interval can also cause:

- More pipeline runs and monitoring records.
- Repeated copying of the same files if the pipeline does not filter already processed files.
- Additional cost.
- Overlapping or queued runs if a pipeline takes longer than the trigger interval.

For these reasons, the interval should match the business requirement and the pipeline's ability to handle repeated execution.

## 7. Inspect the Trigger in ADF Studio

Follow these steps to review the existing trigger.

1. Open the Azure portal.
2. Open `adf-retail-data-dev-1`.
3. Select **Launch Studio**.
4. Open the **Manage** section.
5. Select **Triggers**.
6. Find `TR_Daily_Retail_Ingestion`.
7. Open the trigger details.
8. Inspect its schedule, recurrence interval, time zone, start and end times, and status.
9. Confirm which pipeline is associated with the trigger.
10. Compare the configured schedule with the intended business requirement.

The interface may display slightly different labels depending on the current ADF Studio version.

**Do not create a second trigger if the existing trigger already meets the requirement.** Inspect and reuse the working configuration where appropriate.

## 8. Understand Publish and Trigger Activation

Changes made in ADF Studio are not always active until they are published.

A typical workflow is:

1. Make the required change in Author mode.
2. Validate the pipeline and trigger configuration.
3. Select **Publish all** when the changes are ready.
4. Confirm that publishing completes successfully.
5. Check the trigger's status.
6. Start the trigger if it is stopped and scheduled execution is required.
7. Verify that a run occurs according to the schedule.

Publishing and starting a trigger are different operations. A published trigger may still be stopped.

Before starting a trigger, verify that the associated pipeline is ready to execute automatically.

## 9. Understand Time Zones

Schedule configuration must use the intended business time zone.

For example, a business may require ingestion at 6:00 AM India Standard Time (IST).

A schedule expressed in UTC must be converted carefully. IST is UTC+05:30, so 6:00 AM IST corresponds to 12:30 AM UTC.

In ADF Studio, inspect the actual trigger time-zone setting instead of assuming that the displayed time is local.

For this project, the 15-minute test interval is the relevant configuration. The final production execution time should be selected according to the business requirement.

## 10. Monitor Pipeline Runs

After a trigger starts a pipeline, monitoring helps determine whether the run completed successfully.

To inspect runs:

1. Open ADF Studio.
2. Select **Monitor**.
3. Open the pipeline runs view.
4. Find the relevant pipeline run.
5. Review its status, start time, end time, and duration.
6. Open the run to inspect individual activities.
7. Review any failed activity's error message and output.
8. Use the run ID when correlating the execution with SQL error log records.

Common statuses include:

- **Succeeded** — execution completed successfully.
- **Failed** — execution failed.
- **In progress** — execution is still running.
- **Queued** — execution is waiting to run.
- **Cancelled** — execution was cancelled.

A scheduled trigger starting a pipeline does not mean the pipeline succeeded. Always inspect the resulting run.

## 11. Monitor Individual Activities

A pipeline can contain multiple activities, and an individual activity may fail even when other activities succeed.

For this project, pay particular attention to:

- `Set_End_Watermark`
- `Get_Watermark`
- `Copy_Raw_To_Processed`
- `Update_Watermark`
- `Log_Error`

These names describe the project's watermark and failure-handling flow. Inspect the actual activity names and order in ADF Studio if they have changed.

For each run, verify the relevant sequence:

1. The processing boundary is captured.
2. The existing watermark is retrieved.
3. The intended data-copy operation executes.
4. The watermark is updated only after the required processing succeeds.
5. The error-handling path runs when its configured failure condition is met.

A failure in the copy activity should not cause the watermark to advance beyond successfully processed data.

## 12. Validate Source-to-Target Data

Monitoring should be combined with data validation.

### Step 1 — Inspect the source

Open the storage account `stretaildata202610`.

Navigate to the `raw` container and inspect the source files and folder structure.

Record which files you expect the pipeline to process.

### Step 2 — Inspect the target

Navigate to the `processed` container.

Verify that the expected files are present and that their folder hierarchy matches the configured copy behavior.

### Step 3 — Compare source and target

Check:

- Expected files are present.
- Unexpected files have not been copied.
- File sizes are plausible.
- Folder paths are correct.
- Repeat executions behave as intended.

The project uses recursive wildcard CSV copying, preserving folder hierarchy. If the pipeline's selection pattern includes all matching CSV files, repeated scheduled runs may copy existing files again, depending on the configured source and sink behavior.

Do not infer incremental file selection solely from the existence of the watermark table.

### Step 4 — Validate the watermark

In Azure SQL Database, inspect the current watermark:

```sql
SELECT *
FROM dbo.ETL_Watermark;
```

Confirm that the value reflects successful processing according to the pipeline's implemented logic.

### Step 5 — Inspect error records

Run:

```sql
SELECT *
FROM dbo.ETL_ErrorLog;
```

Review any records relevant to the test run, using the actual schema and available run identifiers.

An empty error table does not prove that the pipeline is correct. It may simply mean no error was recorded during the inspected period.

## 13. Understand Overlapping Runs

A schedule trigger can initiate runs repeatedly. If one run takes longer than the schedule interval, another run may start while the previous run is still running, depending on the trigger and pipeline configuration.

For example:

- Trigger interval: 15 minutes.
- Pipeline duration: 20 minutes.

The next scheduled time may arrive before the earlier run has completed.

This can create concurrency risks if two runs attempt to process the same files or update the same watermark.

Recommended checks:

1. Inspect pipeline run start and end times.
2. Determine whether multiple runs overlap.
3. Review how the pipeline behaves when the same files are processed repeatedly.
4. Confirm that watermark updates cannot conflict.
5. Configure concurrency or scheduling controls if required.

Do not assume that every ADF pipeline automatically prevents overlapping executions.

## 14. Troubleshooting

| Problem | What to check |
|---|---|
| Trigger does not start the pipeline | Check whether the trigger is started, published, and within its schedule window |
| Trigger is started but no recent run appears | Inspect start time, time zone, end time, and associated pipeline |
| Pipeline starts but fails | Open the run details and identify the failed activity |
| Pipeline runs more frequently than intended | Inspect recurrence interval and time-zone settings |
| Files are copied repeatedly | Check wildcard settings, target behavior, and whether processed files are tracked |
| Two runs process the same files | Inspect run overlap, concurrency, and idempotency |
| Watermark changes unexpectedly | Review activity dependencies and update logic |
| Error records are missing | Inspect `Log_Error`, its failure dependency, and SQL connectivity |
| Published changes do not appear to take effect | Confirm that publishing completed and the correct trigger is active |
| Run times appear shifted | Check the configured time zone and any UTC-to-local conversion |

## 15. Operational Best Practices

### A. Choose a business-driven schedule

Set the schedule according to how often data arrives and how quickly it must be available.

Do not keep a 15-minute test interval in production unless it is justified by the business requirement.

### B. Monitor failures

Review pipeline failures and configure suitable alerts if operational support requires notifications.

Monitoring a run in ADF is different from automatically notifying someone about the failure.

### C. Validate the output

A successful pipeline status is necessary but not sufficient. Verify the output files and relevant processing state.

### D. Make repeated runs safe

Design for retries, repeated schedules, and recovery from partial failures. Reprocessing should not create incorrect results.

### E. Review the schedule periodically

If the data arrival pattern, processing duration, or business service level changes, reassess the schedule.

### F. Keep a clear operational record

Document the trigger name, pipeline name, intended schedule, time zone, expected output, and recovery steps.

## 16. Interview Questions and Answers

### Q1. What is a trigger in Azure Data Factory?

A trigger initiates pipeline execution based on a schedule, time window, or supported event.

### Q2. What is the difference between Debug and a schedule trigger?

Debug is used to test a pipeline manually. A schedule trigger starts the pipeline automatically according to its configuration.

### Q3. Does a successful trigger mean the pipeline succeeded?

No. The trigger initiates the pipeline. The resulting pipeline run must be inspected to determine whether processing succeeded.

### Q4. Why might a 15-minute schedule be useful during testing?

It allows frequent observation of automated executions. However, it may cause unnecessary runs, repeated file copying, additional costs, or overlapping executions.

### Q5. What should you inspect when a scheduled pipeline fails?

Check the pipeline run, failed activity, error output, linked services, permissions, source and target paths, retry behavior, and error log records.

### Q6. How do you validate a file-copy pipeline?

Compare the expected source files with the target files, inspect the folder hierarchy, review copy activity output, and verify the relevant watermark and error logs.

### Q7. Why should time zones be checked for scheduled triggers?

The configured time zone determines when a pipeline runs. Incorrect assumptions about UTC and local time can shift execution from the intended business time.

### Q8. What is the risk of overlapping runs?

Multiple runs may process the same data or update shared control information at the same time, leading to duplicate processing or inconsistent state if the pipeline is not designed for concurrency.

## 17. Practical Knowledge Check

- [ ] I can explain the difference between manual and scheduled execution.
- [ ] I can locate `TR_Daily_Retail_Ingestion`.
- [ ] I can inspect a trigger's recurrence interval and time zone.
- [ ] I understand the difference between publishing a trigger and starting it.
- [ ] I can locate pipeline runs in ADF Monitor.
- [ ] I can inspect individual activity results.
- [ ] I can validate files in the `raw` and `processed` containers.
- [ ] I can inspect the watermark and error log tables.
- [ ] I understand the risk of overlapping scheduled executions.
- [ ] I can explain why a successful trigger does not guarantee successful data processing.

## 18. Lesson Summary

In this lesson, you learned how Azure Data Factory schedule triggers automate pipeline execution and how monitoring helps validate the results.

The project uses `TR_Daily_Retail_Ingestion`, with a 15-minute interval used during testing. You also learned how to inspect pipeline runs, validate copied files, review watermarks and error logs, and investigate scheduling issues.

The main principle is:

**Automation starts the process; monitoring and data validation establish whether the process worked correctly.**
