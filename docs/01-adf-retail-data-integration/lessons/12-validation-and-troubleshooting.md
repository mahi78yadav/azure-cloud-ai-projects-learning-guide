# Lesson 12 — End-to-End Validation and Troubleshooting in Azure Data Factory

## 1. Learning Objectives

By the end of this lesson, you should be able to:

- Validate the complete ADF data integration workflow.
- Verify Azure resources and linked services.
- Check pipeline execution and activity results.
- Compare source and target files.
- Validate watermark updates and error handling.
- Troubleshoot common integration failures.
- Determine whether the project is ready for demonstration or handover.

## 2. Business Scenario

A retail business receives customer and other business data in Azure Data Lake Storage Gen2. Azure Data Factory moves the files into a processed area, while Azure SQL Database stores pipeline control information, including watermarks and error logs.

A pipeline showing **Succeeded** is not sufficient evidence that the entire solution is correct. End-to-end validation should confirm that:

1. The expected source data exists.
2. ADF can access the source and target.
3. The pipeline executes as designed.
4. The expected output is created.
5. The watermark changes only after the relevant processing succeeds.
6. Failures are visible and can be investigated.
7. Scheduled execution behaves as expected.

This lesson provides a repeatable validation process for the project.

## 3. Project Resource Reference

| Resource | Project value |
|---|---|
| Resource group | `rg-retail-data-dev` |
| ADLS Gen2 account | `stretaildata202610` |
| ADF instance | `adf-retail-data-dev-1` |
| Azure SQL database | `sqldb-retail-dev` |
| SQL server | `sql-retail-data-dev-2026.database.windows.net` |
| Source container | `raw` |
| Target container | `processed` |
| Watermark table | `dbo.ETL_Watermark` |
| Error log table | `dbo.ETL_ErrorLog` |
| Customer copy pipeline | `PL_Copy_Customers` |
| Main file-copy pipeline | `PL_Copy_All_Files` |
| Schedule trigger | `TR_Daily_Retail_Ingestion` |

Use these names to inspect the resources created for this project. If a resource has been renamed or moved, use its current name in the validation record.

## 4. Validation Strategy

Validate the project in the following order.

| Stage | What to validate |
|---|---|
| 1. Azure resources | Required resources exist and are available |
| 2. Storage | Containers and expected source files exist |
| 3. Connectivity | ADF linked services can connect |
| 4. Datasets | Dataset paths, formats, and parameters are correct |
| 5. Pipeline | Activities and dependencies are configured correctly |
| 6. Execution | Pipeline and activity runs have expected statuses |
| 7. Output | Target files match the expected results |
| 8. Control state | Watermark and error logging behave correctly |
| 9. Automation | The schedule trigger runs the intended pipeline |
| 10. Recovery | A controlled failure can be investigated and resolved |

Complete the checks in sequence so that a failure can be isolated to the relevant layer.

## 5. Stage 1 — Validate Azure Resources

Open the Azure portal and verify that the required resources are available.

### Checklist

- [ ] Resource group `rg-retail-data-dev` exists.
- [ ] Storage account `stretaildata202610` exists.
- [ ] Data Factory `adf-retail-data-dev-1` exists.
- [ ] Azure SQL Database `sqldb-retail-dev` exists.
- [ ] The SQL server endpoint is correct.
- [ ] You have the necessary permissions to inspect the resources.

### What if a resource is missing?

Check the resource group and subscription before creating anything new. A resource may have been created in a different resource group.

Do not create duplicate resources just because they are not visible in the currently selected resource group.

## 6. Stage 2 — Validate Storage

Open the storage account:

`stretaildata202610`

Inspect the containers used by the project.

- `raw` — source files.
- `processed` — copied output files.
- `archive` — available for the project's archival design.
- `config` — available for configuration or control files, as applicable.

The presence of a container does not mean that it is actively used by the pipeline. Validate the actual pipeline paths.

### Source validation

Open the `raw` container and verify:

- Expected source files exist.
- File names and extensions match the pipeline configuration.
- Folder paths match the dataset configuration.
- The source files can be read using the configured identity.

### Target validation

Open the `processed` container and inspect the copied files.

For this project, the main file-copy pipeline uses a recursive wildcard CSV-copy pattern and preserves the folder hierarchy.

Confirm that the expected files appear at the corresponding target paths.

## 7. Stage 3 — Validate Linked Services

The project uses these linked services:

- `LS_ADLS_Retail`
- `LS_AzureSQL_Retail`

A linked service defines connection information and authentication for a data source or destination.

### ADLS linked service

1. Open ADF Studio.
2. Navigate to **Manage**.
3. Open **Linked services**.
4. Select `LS_ADLS_Retail`.
5. Inspect the storage account and authentication configuration.
6. Use **Test connection** if supported by the configured connection.
7. Confirm that the identity has the required storage permissions.

### Azure SQL linked service

1. Select `LS_AzureSQL_Retail`.
2. Verify the SQL server and database.
3. Inspect the authentication configuration.
4. Test the connection.
5. Confirm that the identity has the necessary permissions on the relevant SQL objects.

### Common connection problems

| Symptom | What to check |
|---|---|
| Connection test fails | Endpoint, authentication, firewall, network access, and permissions |
| Storage access denied | Managed identity permissions and storage ACLs where applicable |
| SQL query fails | Database permissions, table names, and SQL syntax |
| Connection works but Copy Activity fails | Dataset configuration, source path, target path, and activity settings |

A successful connection test verifies connectivity, not the correctness of the full pipeline.

## 8. Stage 4 — Validate Datasets

Datasets define the data location and format used by pipeline activities.

The project includes datasets such as:

- `DS_Source_Customers`
- `DS_Target_Customers`
- `DS_Source_File`
- `DS_Target_File`
- `DS_Source_Folder`
- `DS_Source_Root`
- `DS_Target_Root`
- `DS_Watermark`

Inspect the datasets that are actually referenced by each pipeline.

### Checklist

- [ ] Linked service references are correct.
- [ ] Container and directory paths are correct.
- [ ] File names or wildcard patterns match the intended input.
- [ ] File format and delimiter settings are correct.
- [ ] Required parameters are defined.
- [ ] Dataset parameter values are supplied by the pipeline.
- [ ] The watermark dataset points to the expected SQL object.

**Important:** A dataset can be configured correctly but still fail when the source file is missing or permissions are insufficient.

## 9. Stage 5 — Validate Pipeline Configuration

Inspect these pipelines:

- `PL_Copy_Customers`
- `PL_Copy_All_Files`

### PL_Copy_Customers

Verify that the pipeline references the correct source and target datasets and that the Copy Activity uses the expected mapping and file settings.

### PL_Copy_All_Files

Verify the recursive wildcard configuration.

Confirm:

- The source points to the intended `raw` location.
- The wildcard matches the intended CSV files.
- Recursive traversal is configured as intended.
- The target points to the intended `processed` location.
- Folder hierarchy behavior is correct.
- Retry settings match the intended configuration.

### Watermark and error-handling flow

Inspect the project's watermark and failure-handling activities:

`Set_End_Watermark → Get_Watermark → Copy_Raw_To_Processed → Update_Watermark`

Also inspect the `Log_Error` failure-handling path.

Confirm that:

- The watermark update depends on successful processing.
- The error logging activity uses the intended failure dependency.
- The actual activity expressions and SQL statements reference the correct objects.
- A successful logging operation does not hide the original processing failure.

The sequence above reflects the intended flow recorded for the project. Check the current pipeline implementation before changing activity dependencies.

## 10. Stage 6 — Validate Pipeline Execution

Run the relevant pipeline in ADF Studio using Debug or another suitable test method.

After the run starts:

1. Open **Monitor**.
2. Locate the pipeline run.
3. Inspect its status.
4. Open the activity run details.
5. Check each activity's status and duration.
6. Review error messages for any failed activity.
7. Record the run ID for future troubleshooting.

### Expected results

For a successful test:

- The pipeline run reports success.
- The required activities complete successfully.
- The expected output files are created.
- The watermark is updated according to the configured processing logic.

Do not use the pipeline status alone as proof of correctness. The output and control state must also be checked.

## 11. Stage 7 — Validate Source-to-Target Results

A file-copy pipeline should be validated using the actual source and target contents.

### Step 1 — Record expected source files

Inspect the `raw` container and record the files that should be processed.

### Step 2 — Execute the pipeline

Run the intended pipeline and record its run ID.

### Step 3 — Inspect the target

Inspect the `processed` container and verify the output.

### Step 4 — Compare results

| Validation item | Expected result |
|---|---|
| Required source files | Present and readable |
| Target files | Present at the intended locations |
| File hierarchy | Matches configured copy behavior |
| File formats | Remain consistent with the source-copy design |
| Unexpected files | None, unless included intentionally |
| Repeat execution | Matches the documented overwrite or reprocessing behavior |

If the pipeline copies all matching CSV files on every run, repeated executions may copy existing files again. Verify the actual sink behavior and do not assume the watermark automatically filters files.

### What about row counts?

For a file-copy pipeline, matching file names and validating copied contents are often the first checks.

If row counts are a requirement, inspect the file contents or load the data into a suitable queryable destination. Do not report that source and target row counts match unless you have actually measured them.

## 12. Stage 8 — Validate Watermark State

The project uses the SQL table:

`dbo.ETL_Watermark`

Inspect the current records:

```sql
SELECT *
FROM dbo.ETL_Watermark;
```

To inspect the schema:

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

### Successful execution

After successful processing, verify that the relevant watermark was updated as expected.

### Failed execution

For a controlled failure before the watermark update, verify that the previous successful watermark remains unchanged.

### Important distinction

A watermark table records processing state. It does not prove that the Copy Activity filters only new files or rows.

To claim that the implementation is fully incremental, validate the actual selection and filtering logic, including repeat-run behavior.

## 13. Stage 9 — Validate Error Logging

The project uses:

`dbo.ETL_ErrorLog`

Inspect the available records:

```sql
SELECT *
FROM dbo.ETL_ErrorLog;
```

Inspect the actual schema before writing queries that reference specific column names.

### Controlled failure test

In a development environment:

1. Record the current watermark.
2. Use a controlled invalid source path or another safe test failure.
3. Run the pipeline.
4. Inspect the failed activity.
5. Verify whether `Log_Error` executes.
6. Check whether an error record is written.
7. Confirm that the watermark has not advanced past unsuccessful processing.
8. Restore the correct configuration.

A missing error record does not necessarily mean the pipeline did not fail. The logging activity may have failed, its dependency may not have been configured correctly, or the SQL operation may not have completed.

## 14. Stage 10 — Validate Trigger Execution

The project uses:

`TR_Daily_Retail_Ingestion`

The trigger was configured for a 15-minute interval during testing.

To validate it:

1. Open ADF Studio.
2. Navigate to **Manage → Triggers**.
3. Inspect the trigger's schedule, time zone, and status.
4. Confirm that it is published and started when automatic execution is required.
5. Open **Monitor**.
6. Verify that runs are created according to the configured schedule.
7. Inspect the results of those runs.

### Watch for overlapping executions

If a pipeline takes longer than its trigger interval, runs may overlap depending on the configuration.

Inspect the run start and end times. If overlap occurs, review the pipeline's concurrency and repeat-processing behavior.

Do not retain a frequent test schedule in production unless the business requirement justifies it.

## 15. End-to-End Validation Checklist

Use this checklist before demonstrating or handing over the project.

### Azure resources

- [ ] Required resources exist.
- [ ] The intended subscription and resource groups are selected.
- [ ] Storage and SQL resources are accessible.

### Storage and connections

- [ ] Source files exist in `raw`.
- [ ] The target location is correct.
- [ ] `LS_ADLS_Retail` connects successfully.
- [ ] `LS_AzureSQL_Retail` connects successfully.
- [ ] The configured identity has the required permissions.

### Pipeline configuration

- [ ] Source and target datasets are correct.
- [ ] Wildcard and recursive-copy settings are correct.
- [ ] Activity dependencies are correct.
- [ ] Retry settings are verified.
- [ ] Watermark update occurs only after the required processing succeeds.
- [ ] Error logging is connected to the appropriate failure path.

### Execution and output

- [ ] A test pipeline run completes successfully.
- [ ] Individual activity statuses have been inspected.
- [ ] Target files match the expected copy behavior.
- [ ] Repeat-run behavior has been checked.
- [ ] The watermark is consistent with the successful processing state.
- [ ] Failure logging has been tested in a safe environment.

### Automation and operations

- [ ] The trigger configuration has been inspected.
- [ ] Scheduled runs are visible in Monitor.
- [ ] Time zone and recurrence interval are understood.
- [ ] Failure investigation steps are documented.
- [ ] The project README matches the actual implementation.

## 16. Troubleshooting Decision Guide

Use the first relevant symptom to decide where to investigate.

| Symptom | First investigation |
|---|---|
| Pipeline cannot start | Check publication, trigger status, and pipeline configuration |
| Linked service test fails | Check endpoint, authentication, network access, and permissions |
| Dataset reports path not found | Check the container, directory, file name, and wildcard |
| Copy Activity fails | Inspect the activity error output, source access, and sink settings |
| Pipeline succeeds but output is missing | Check copy output, target path, wildcard selection, and source contents |
| Files appear more than once | Inspect repeat-run behavior, overwrite settings, and file tracking |
| Watermark is not updated | Inspect the update activity, dependency condition, SQL permissions, and query |
| Watermark advances after failure | Inspect the activity dependency and update conditions |
| Error record is missing | Inspect the failure path and the SQL logging activity |
| Trigger does not run | Inspect trigger status, publication, recurrence, and time zone |
| Runs overlap | Compare run durations and trigger intervals; review concurrency behavior |

## 17. Project Readiness and Handover

A project should be considered ready for demonstration when the core workflow has been tested and its behavior can be explained.

For this project, prepare to demonstrate:

1. The `raw` and `processed` containers.
2. The ADF linked services and datasets.
3. The `PL_Copy_Customers` and `PL_Copy_All_Files` pipelines.
4. The watermark activity sequence.
5. The `dbo.ETL_Watermark` and `dbo.ETL_ErrorLog` tables.
6. A successful pipeline run and its output.
7. A controlled failure test and its diagnostic evidence.
8. The schedule trigger and monitoring view.
9. The project README and known limitations.

Keep a clear distinction between features that have been tested and enhancements that remain recommendations.

## 18. Interview Questions and Answers

### Q1. How do you validate an ADF pipeline end to end?

Verify resources, connections, datasets, pipeline dependencies, execution results, source-to-target output, control tables, and trigger behavior.

### Q2. Is a successful pipeline status enough?

No. It must be supported by validation of the expected output and relevant processing state.

### Q3. How do you troubleshoot a Copy Activity failure?

Inspect the activity's error output, source and sink paths, permissions, linked services, dataset parameters, and retry behavior.

### Q4. How do you validate incremental loading?

Inspect the actual watermark logic, source filtering, successful watermark updates, failure behavior, and repeat-run results.

### Q5. How do you validate error handling?

Trigger a controlled failure in a development environment and inspect the failed activity, failure dependency, logging result, and watermark state.

### Q6. How do you validate a schedule trigger?

Inspect its published configuration and status, then confirm that expected pipeline runs appear in Monitor.

### Q7. What is the difference between monitoring and validation?

Monitoring shows execution status and diagnostic information. Validation checks whether the output and processing behavior satisfy the expected requirements.

## 19. Practical Knowledge Check

- [ ] I can validate the required Azure resources.
- [ ] I can test the ADLS and SQL linked services.
- [ ] I can inspect dataset paths and pipeline dependencies.
- [ ] I can validate source-to-target file movement.
- [ ] I can explain the watermark update rule.
- [ ] I can inspect SQL error logs.
- [ ] I can troubleshoot a failed Copy Activity.
- [ ] I can verify a scheduled trigger.
- [ ] I can explain the difference between a successful run and a correct result.
- [ ] I can demonstrate the project without claiming unverified functionality.

## 20. Lesson Summary

In this lesson, you learned how to validate an Azure Data Factory solution from its Azure resources and connections through execution, output verification, watermark state, error handling, and scheduled runs.

The most important principle is:

**Validate the result, not just the execution status.**

A reliable data integration solution must produce the expected output, preserve correct processing state, expose failures, and support repeatable operations.
