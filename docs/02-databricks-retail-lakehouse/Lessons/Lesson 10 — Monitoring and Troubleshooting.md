# Lesson 10 — Monitoring and Troubleshooting

## 1. Objective

Learn how to diagnose common Azure Databricks lakehouse failures and determine whether the issue is related to storage, permissions, schemas, transformations, or orchestration.

## 2. Why monitoring matters

A data pipeline may fail before processing begins, during a transformation, or after writing data. It can also complete successfully while producing unexpected results.

Monitoring helps answer four questions:

1. Did the job start?
2. Did every required task finish?
3. Did the output data meet expectations?
4. If something failed, where and why did it happen?

## 3. Project components to monitor

| Component | What to check |
|---|---|
| Databricks job | Run status, duration, task errors |
| Notebooks | Code exceptions and execution output |
| ADLS Gen2 | Source paths, file availability, permissions |
| Unity Catalog | Catalog, schema, table access |
| Delta tables | Row counts, schema, write behavior |
| Auto Loader | Stream progress, checkpoint, schema handling |
| Gold layer | Join results and business totals |

## 4. Troubleshooting workflow

Use this sequence rather than changing multiple settings at once.

```text
Identify the failed or unexpected result
                  |
                  v
Check job and notebook output
                  |
                  v
Locate the failing operation
                  |
                  v
Check path, permissions, schema, and data
                  |
                  v
Correct the root cause
                  |
                  v
Rerun safely and validate the output
```

Start with the earliest failing operation. Later errors may be consequences of an earlier failure.

## 5. Common problem: file not found

**Possible causes**

- Incorrect container or directory.
- Misspelled file name.
- Wrong storage account.
- File not yet uploaded.
- Incorrect path format.

**Investigation**

1. Inspect the source path in the notebook.
2. Confirm the file exists in ADLS Gen2.
3. Verify the storage account and container.
4. Check whether the code expects a file or a directory.
5. Rerun after correcting the actual path.

Avoid changing a path based on guesswork; confirm it against the source storage structure.

## 6. Common problem: permission denied

**Possible causes**

- Incorrect identity.
- Missing storage permissions.
- Missing Unity Catalog grants.
- Access configuration not aligned with the resource.

**Investigation**

1. Identify the identity used by the operation.
2. Confirm the intended storage or catalog resource.
3. Review the relevant role assignments or grants.
4. Grant only the permissions required.
5. Retest the failed operation.

Azure RBAC, storage data permissions, and Unity Catalog privileges serve related but distinct purposes. A permission in one layer does not automatically grant every required permission in another.

## 7. Common problem: unresolved column

A Spark error such as an unresolved column usually means that the query refers to a field that is not available under that name at that point in the transformation.

**Investigation**

```python
df.printSchema()
display(df.limit(5))
```

Then check:

- The spelling and capitalization of the field.
- The input table's actual schema.
- Whether a previous transformation renamed the column.
- Whether a join created duplicate or qualified column names.
- Whether the query is using the expected input DataFrame.

Correct the reference or transformation based on the actual schema. Do not invent a column merely to suppress the error.

## 8. Common problem: unexpected row counts

Suppose a source has 25 orders but the output contains more than 25 rows.

Possible causes include:

- A one-to-many join.
- Duplicate lookup keys.
- Repeated append operations.
- Incorrect source selection.
- Different table grain.

Check the counts at each stage and inspect the join keys. For a one-row-per-order output, validate that the order key remains unique.

If a transformation intentionally filters records, document the reason for the reduction.

## 9. Common problem: Auto Loader does not process new files

Check:

1. The configured source path.
2. Whether the new file has arrived.
3. The streaming query or job status.
4. The checkpoint location.
5. Schema and parsing errors.
6. Whether the file was already processed by that query.

Checkpoint deletion or replacement should not be used as the first troubleshooting step. It can change restart behavior and cause reprocessing.

## 10. Common problem: job succeeds but output is incorrect

A successful execution only indicates that the configured tasks completed according to their execution status.

It does not establish that:

- Every expected source file was included.
- All business rules were applied correctly.
- The table contains no duplicates.
- Revenue totals are correct.
- Data is sufficiently fresh.

Compare the output with expected counts, schema, keys, and business totals. Trace the issue back to the earliest layer where the results differ from expectations.

## 11. Root-cause analysis

When documenting an incident, record:

- **Symptom:** What failed or looked incorrect?
- **Evidence:** What error message or unexpected result was observed?
- **Root cause:** What caused the issue?
- **Resolution:** What change corrected it?
- **Validation:** What evidence confirmed the fix?
- **Prevention:** What change could reduce recurrence?

For example, if a job fails because a column name is incorrect, record the actual column name, the failing transformation, the correction, and a successful validation run.

## 12. Interview explanation

“When troubleshooting Databricks pipelines, I start with job run history and notebook output to locate the first failing operation. I then check source paths, access permissions, table schemas, transformation logic, and row counts. After fixing the root cause, I rerun the affected process safely and validate the resulting data instead of relying only on the job status.”

## 13. Key takeaways

- Diagnose the first failure before investigating downstream symptoms.
- Confirm paths and schemas rather than guessing.
- Review the identity and permission layer involved.
- Investigate unexpected row counts and join cardinality.
- Protect Auto Loader checkpoints.
- Validate business outputs after every recovery.
