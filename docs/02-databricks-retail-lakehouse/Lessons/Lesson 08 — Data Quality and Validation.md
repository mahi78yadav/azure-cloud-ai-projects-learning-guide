# Lesson 08 — Data Quality and Validation

## 1. Objective

Understand how to validate data across the Bronze, Silver, and Gold layers and detect issues before they affect business reporting.

By the end of this lesson, you should be able to explain:

- Why data quality is important in a lakehouse.
- How to validate row counts and mandatory fields.
- How to detect duplicates and invalid relationships.
- How to validate Gold business outputs.
- How to operationalize data-quality checks.

## 2. Business requirement

A pipeline can complete successfully and still produce incorrect data.

For example:

- Orders may contain invalid customer IDs.
- Product prices may be missing.
- Duplicate records may inflate sales totals.
- Incorrect joins may multiply rows.
- A source file may contain fewer records than expected.

Data-quality checks help detect these problems before data reaches dashboards or business users.

## 3. Project implementation

The project includes the notebook:

`05_Data_Quality.py`

The lakehouse uses the `retail` catalog and the `bronze`, `silver`, and `gold` schemas.

The completed project also validated the Gold sales table and the Auto Loader orders dataset, each with 25 rows in the recorded sample results.

This lesson explains the validation approach. The example SQL statements are reusable patterns and should not be mistaken for proof that every listed check is already implemented in the notebook.

## 4. Data-quality dimensions

| Dimension | Meaning | Retail example |
|---|---|---|
| Completeness | Required information is present | Customer ID is not null |
| Uniqueness | Records are not duplicated at the intended grain | One row per order |
| Validity | Values conform to expected rules | Quantity is positive |
| Consistency | Related datasets agree | Order customer exists |
| Accuracy | Data reflects the intended business fact | Sales amount is correct |
| Timeliness | Data arrives within the required period | Daily orders are available on time |

## 5. Check row counts

Start by checking the number of records in each table.

```sql
SELECT COUNT(*) AS customer_count
FROM retail.silver.customers;
```

```sql
SELECT COUNT(*) AS product_count
FROM retail.silver.products;
```

```sql
SELECT COUNT(*) AS order_count
FROM retail.silver.orders;
```

```sql
SELECT COUNT(*) AS sales_count
FROM retail.gold.gold_sales;
```

Use the actual table names from the project if they differ.

Expected source counts for the sample datasets are:

| Dataset | Expected source rows |
|---|---:|
| Customers | 10 |
| Products | 10 |
| Orders | 25 |
| Stores | 7 |

Counts may differ between layers when transformation rules intentionally filter or consolidate records. Investigate unexplained differences.

## 6. Check mandatory fields

For example, identify orders without an order ID:

```sql
SELECT *
FROM retail.silver.orders
WHERE order_id IS NULL;
```

Check for missing customer identifiers:

```sql
SELECT *
FROM retail.silver.orders
WHERE customer_id IS NULL;
```

These are examples. Whether a field is mandatory must be defined by the business data contract.

## 7. Detect duplicates

If one row per order is expected:

```sql
SELECT
    order_id,
    COUNT(*) AS duplicate_count
FROM retail.silver.orders
GROUP BY order_id
HAVING COUNT(*) > 1;
```

If the business grain is one row per order line, validate the appropriate composite key instead.

Do not automatically delete every duplicate result. First determine whether it is a genuine duplicate, a valid repeated transaction, or an expected consequence of the table grain.

## 8. Validate business rules

For a retail order dataset, possible rules include:

- Quantity must be greater than zero.
- Order amount must not be negative unless the business supports adjustments or refunds.
- Required dates must be valid.
- Product and customer references must be resolvable.

Example:

```sql
SELECT *
FROM retail.silver.orders
WHERE quantity <= 0;
```

Apply this query only if `quantity` is the correct field and zero or negative quantities are invalid under the business rules.

## 9. Check referential integrity

Find orders that reference a customer missing from the Silver customer table:

```sql
SELECT o.*
FROM retail.silver.orders o
LEFT ANTI JOIN retail.silver.customers c
    ON o.customer_id = c.customer_id;
```

Repeat the pattern for products using the actual product key.

Investigate unmatched records. They may indicate missing master data, an ingestion issue, a key-format mismatch, or an expected business exception.

## 10. Validate Gold business totals

A Gold sales table should preserve the intended business grain and metrics.

Start with a count:

```sql
SELECT COUNT(*) AS sales_count
FROM retail.gold.gold_sales;
```

If the Gold table contains an order amount field, an illustrative aggregation is:

```sql
SELECT SUM(order_amount) AS total_order_amount
FROM retail.gold.gold_sales;
```

Before interpreting the total, confirm:

- The actual measure column name.
- Whether the table contains one row per order or order line.
- Whether the join logic can multiply records.
- Whether refunds, cancellations, and adjustments are included.
- Whether the result reconciles with the expected source data.

A total that differs from the source is not necessarily wrong; it must be explained by the transformation and business rules.

## 11. Make quality checks operational

A mature data-quality process should do more than display query results.

A typical pattern is:

```text
Pipeline execution
       |
       v
Run quality checks
       |
       v
Evaluate thresholds
       |
       +---- Pass ----> Publish or continue
       |
       +---- Fail ----> Log, alert, or stop
```

For production implementation, define:

1. The check to execute.
2. The expected result or threshold.
3. The action when the check fails.
4. The location for recording results.
5. The owner responsible for investigation.

For example, a pipeline may fail when a mandatory key is missing or record a warning when an expected daily volume changes beyond an approved threshold.

This workflow is a recommended design pattern. Confirm the actual failure and alerting behavior of the completed notebook before describing it as implemented.

## 12. Common issues

**Row count differs from source:** Check filters, duplicate handling, and join cardinality.

**Unexpected nulls:** Review source data, schema conversion, and transformation rules.

**Duplicate keys:** Confirm the intended grain and deduplication strategy.

**Quality check passes but reporting is wrong:** Validate business definitions, joins, and metric calculations.

**Checks run but failures are ignored:** Add explicit thresholds and a documented response to failed checks.

## 13. Interview explanation

“I added a data-quality validation stage to the Databricks retail lakehouse. The validation approach covers row counts, mandatory fields, duplicate detection, relationships between business entities, and Gold output checks. The objective is to identify data issues early and ensure that downstream analytics uses consistent, trustworthy data.”

## 14. Key takeaways

- Successful execution is not the same as correct data.
- Validation should cover every layer.
- Expected row counts are useful but insufficient on their own.
- Checks must reflect the business definition of valid data.
- Gold metrics require reconciliation and join validation.
- Production quality checks should have explicit pass/fail actions.

## 15. Related project artifact

Review `05_Data_Quality.py` and document which checks are actually implemented, what each check validates, and how failures are handled.
