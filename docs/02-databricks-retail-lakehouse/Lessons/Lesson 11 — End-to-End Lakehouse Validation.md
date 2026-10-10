# Lesson 11 — End-to-End Lakehouse Validation

## 1. Objective

Validate the completed retail lakehouse from source ingestion through Gold business output.

By the end of this lesson, you should be able to explain:

- How to validate the Bronze, Silver, and Gold layers.
- How to reconcile row counts.
- How to validate joins and business measures.
- How to confirm that Auto Loader and scheduled execution work as expected.
- How to distinguish technical success from business correctness.

## 2. Validation scope

The completed project uses:

- Azure Data Lake Storage Gen2 for source files.
- Azure Databricks notebooks for processing.
- Unity Catalog to organize and govern tables.
- Delta tables for lakehouse storage.
- Auto Loader for incremental file ingestion.
- A scheduled Databricks job for automated execution.

Validation should cover both the data itself and the processes that produce it.

## 3. End-to-end flow

```text
Source files in ADLS Gen2
          |
          v
   Bronze ingestion
          |
          v
   Silver transformation
          |
          v
    Gold business data
          |
          v
 Quality and reconciliation
          |
          v
    Reporting readiness
```

Auto Loader and the scheduled job are additional ingestion and orchestration mechanisms. Their execution should be validated separately from the layer-by-layer data checks.

## 4. Step 1 — Validate source data

The project uses four sample datasets:

| Dataset | Expected source rows |
|---|---:|
| Customers | 10 |
| Products | 10 |
| Orders | 25 |
| Stores | 7 |

Confirm that the source files are available in the expected storage locations and that the data is readable.

These counts apply to the supplied sample datasets. They are not fixed requirements for future files or production data.

## 5. Step 2 — Validate Bronze

Check that the expected Bronze tables exist and contain the intended records.

Example:

```sql
SELECT COUNT(*) AS order_count
FROM retail.bronze.orders;
```

Repeat for the customer, product, and store tables, using the actual table names in the repository.

Verify:

- Table exists.
- Schema is as expected.
- Records are readable.
- Row counts align with the source or are explainably different.
- Repeated executions do not introduce unintended duplicates.

## 6. Step 3 — Validate Silver

Review the output of `02_Silver_Transformation.py`.

Check:

- Required fields.
- Data types.
- Duplicate business keys.
- Invalid records.
- Customer and product references.
- Row-count differences from Bronze.

Silver counts do not have to match Bronze exactly if the documented transformations remove invalid records or consolidate duplicates. Every material difference should have a clear explanation.

## 7. Step 4 — Validate Gold

The completed project includes:

`retail.gold.gold_sales`

The recorded result contains 25 rows.

Validate the output:

```sql
SELECT COUNT(*) AS sales_count
FROM retail.gold.gold_sales;
```

Inspect records:

```sql
SELECT *
FROM retail.gold.gold_sales
LIMIT 10;
```

Then check the table's intended grain, uniqueness, joins, and business measures.

If the table is intended to contain one row per order, check that the order key is unique. If its grain is order-line level, use the appropriate composite key instead.

## 8. Step 5 — Validate Auto Loader

The Auto Loader orders dataset was validated with 25 records in the completed project.

Check the actual destination table configured in `04_AutoLoader_Ingestion.py`.

Confirm:

- The ingestion query or job completed.
- Expected records are present.
- The schema is correct.
- The checkpoint is configured.
- A subsequent run behaves as intended.
- No unexpected duplicates were introduced.

A table count of 25 validates the recorded sample output, but it does not independently prove that all future files will be processed correctly.

## 9. Step 6 — Validate scheduled execution

Open `JOB_Retail_Lakehouse_ETL` and review its run history.

Verify:

- The job is configured for the intended daily schedule.
- The time zone is UTC+05:30.
- The relevant task runs completed.
- There are no unexplained failures.
- Output tables remain valid after scheduled execution.

A job can succeed while data-quality problems remain, so combine run-status checks with table validation.

## 10. Step 7 — Validate Delta behavior

The project demonstrated Delta history, time travel, and restore.

When reviewing these capabilities, distinguish between:

- **History:** Reviewing recorded table operations.
- **Time travel:** Reading an available earlier table version.
- **Restore:** Reverting a table to an earlier version.

These capabilities depend on the table's retained history and underlying data files remaining available. Do not assume an older version can be accessed indefinitely.

Example history query:

```sql
DESCRIBE HISTORY retail.gold.gold_sales;
```

Use time travel or restore only in an appropriate test environment unless the recovery has been approved for the target data.

## 11. End-to-end acceptance checklist

- [ ] Source files are available.
- [ ] Bronze tables are readable.
- [ ] Silver transformations complete.
- [ ] Gold sales table contains the expected sample output.
- [ ] Data-quality checks have been reviewed.
- [ ] Auto Loader output has been validated.
- [ ] Scheduled job history has been checked.
- [ ] Delta history and recovery concepts have been demonstrated.
- [ ] Unexplained count differences have been investigated.
- [ ] Repository documentation matches the actual implementation.

## 12. Interview explanation

“I validated the retail lakehouse end to end, from source files in ADLS Gen2 through Bronze and Silver processing to the Gold sales table. I checked table availability, sample row counts, schemas, data relationships, Auto Loader output, and scheduled job history. I also explored Delta history, time travel, and restore to understand data recovery capabilities.”

## 13. Key takeaways

- Validate each layer independently and then validate the overall flow.
- Counts must be interpreted alongside business rules.
- A successful job is not sufficient evidence of correct data.
- Auto Loader requires separate validation of its incremental behavior.
- Delta recovery capabilities depend on retained history and data availability.
