# Lesson 09 — Jobs, Scheduling, and Orchestration

## 1. Objective

Understand how Azure Databricks jobs automate notebook execution and how scheduling supports repeatable data processing.

By the end of this lesson, you should be able to explain:

- Why a Databricks job is required.
- How notebooks are organized into an automated workflow.
- How schedules trigger job execution.
- How to monitor job runs.
- What to check when a job fails.

## 2. Business requirement

A retail organization needs its lakehouse processing to run regularly without an engineer manually opening and executing each notebook.

The operational workflow should execute the required processing, record the results, and make failures visible for investigation.

## 3. Project implementation

The completed project contains the Databricks job:

| Property | Project value |
|---|---|
| Job name | `JOB_Retail_Lakehouse_ETL` |
| Job ID | `219655900077611` |
| Schedule | Daily at 6:00 AM |
| Time zone | UTC+05:30 |
| Platform | Azure Databricks |

The schedule corresponds to 6:00 AM Indian Standard Time.

## 4. Job architecture

```text
Scheduled Databricks Job
          |
          v
    Notebook execution
          |
          v
  Bronze / Silver / Gold
          |
          v
   Job run status
          |
          v
  Validation and monitoring
```

A Databricks job can execute one or more tasks. Tasks may run sequentially or in parallel, depending on the dependencies and workflow design.

The exact task graph should be confirmed from the configured job rather than inferred from the job name.

## 5. Job versus notebook

A **notebook** contains code and transformations.

A **job** runs notebooks or other supported tasks according to a defined workflow.

A **schedule** determines when the job should start.

A **job run** is one execution of that job.

These are separate concepts. Creating a notebook does not automatically schedule it, and creating a job does not prove that its most recent run succeeded.

## 6. Scheduling considerations

When configuring a schedule, verify:

1. The intended execution frequency.
2. The correct time zone.
3. The task dependencies.
4. The compute configuration.
5. The permissions and identities required to access data.
6. The expected runtime and resource requirements.
7. The handling of failures and overlapping runs.

For this project, the documented schedule is daily at 6:00 AM IST.

Production schedules should also account for source-data arrival. If the source files are late, the workflow may need a readiness check or an appropriate dependency before processing starts.

## 7. Monitoring a job run

In the Databricks workspace:

1. Open **Workflows** or the jobs area.
2. Find `JOB_Retail_Lakehouse_ETL`.
3. Open the job.
4. Review the run history.
5. Open a run to inspect task status and output.
6. Investigate any failed task before rerunning it.

The completed project had multiple successful job runs.

A successful job status is useful operational evidence, but it does not independently prove that every business result is correct.

## 8. Failure and retry considerations

A failed task may be caused by:

- Invalid source paths.
- Missing storage permissions.
- Incorrect table or column names.
- Schema changes.
- Insufficient compute resources.
- Transformation errors.
- Unexpected source data.

A retry is appropriate when the failure is transient or the task is designed to be safely repeated.

Retries are not a substitute for correcting invalid data, incorrect code, or permissions. Before rerunning a task, consider whether it may append duplicate records or overwrite valid output.

## 9. Production improvements

For a production-grade workflow, consider:

- Task dependencies between Bronze, Silver, and Gold.
- Failure notifications.
- Appropriate retry policies.
- Execution time and data-volume monitoring.
- Data-quality checks after transformations.
- Clear ownership for failed runs.
- Controlled deployment between development, test, and production environments.

These are recommended operational practices; verify each one in the job configuration before claiming it is implemented in the project.

## 10. Interview explanation

“I automated the Azure Databricks retail lakehouse workflow using a scheduled job named `JOB_Retail_Lakehouse_ETL`. It is configured for daily execution at 6:00 AM IST. I use job run history to review execution status and investigate failures, while validating the resulting data separately to ensure that a successful run also produces the expected output.”

## 11. Key takeaways

- Notebooks contain processing logic; jobs orchestrate execution.
- Schedules must use the correct time zone.
- Job status and data correctness are separate concerns.
- Retry policies must account for duplicate and overwrite risks.
- Operational monitoring should include both execution and data-quality results.

## 12. Related project artifact

Review the configured job `JOB_Retail_Lakehouse_ETL` and document its actual task list, dependencies, compute configuration, schedule, and failure-handling settings.
