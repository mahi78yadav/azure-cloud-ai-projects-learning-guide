# Lesson 06 — Gold Business Layer

## 1. Objective

Understand how the Gold layer presents curated retail data for reporting and business analysis.

By the end of this lesson, you should be able to explain:

- The purpose of the Gold layer.
- How multiple datasets are joined for analysis.
- How a business-ready sales table is created.
- How to validate the resulting dataset.
- How Gold tables support BI and downstream applications.

## 2. Business requirement

Retail business stakeholders want to answer questions such as:

- How much revenue did the business generate?
- Which products generate the most sales?
- Which customers place the most orders?
- How do sales vary across stores?
- How many orders are recorded in the lakehouse?

The Gold layer organizes trusted data into structures that are easier for analysts, dashboards, and downstream applications to consume.

## 3. Architecture

```text
Silver Customers ──┐
Silver Products ───┤
Silver Orders ─────┼──> Gold Business Layer
Silver Stores ─────┘            |
                                v
                         gold_sales
                                |
                                v
                      Reporting and analytics
```

The project notebook for this stage is `03_Gold_Business_Layer.py`.

The completed project includes the table `retail.gold.gold_sales`, which was validated with 25 rows.

## 4. What belongs in Gold?

Gold datasets are designed around business consumption rather than simply copying source data.

Typical examples include:

- Sales fact tables.
- Customer and product dimensions.
- Store-level performance datasets.
- Aggregated sales metrics.
- Reporting-ready datasets.

Not every project needs all these objects. Implement only the tables required by the business use cases.

## 5. Joining business datasets

A sales dataset may combine order information with customer, product, and store details.

The conceptual relationship is:

```text
Orders
  |
  +-- Customer ID --> Customers
  |
  +-- Product ID ---> Products
  |
  +-- Store ID -----> Stores
```

An illustrative Spark SQL query is:

```sql
SELECT
    o.order_id,
    o.customer_id,
    c.customer_name,
    o.product_id,
    p.product_name,
    o.store_id,
    o.quantity,
    o.order_amount
FROM retail.silver.orders o
LEFT JOIN retail.silver.customers c
    ON o.customer_id = c.customer_id
LEFT JOIN retail.silver.products p
    ON o.product_id = p.product_id;
```

This is an illustrative query. Confirm the actual column names and joins against the completed project schemas. Include the store join when the relevant keys and attributes are available.

## 6. Important join considerations

### Choose the correct join type

- **INNER JOIN:** Keeps records that match on both sides.
- **LEFT JOIN:** Keeps every row from the left dataset, even when the lookup has no match.
- **LEFT ANTI JOIN:** Returns left-side records without a matching right-side record.

The correct choice depends on the business requirement.

### Prevent row multiplication

If one order matches multiple customer or product records, the join may produce extra rows.

Before joining, validate that lookup keys are unique when the model expects one record per key.

### Protect business measures

If the Gold table contains sales amounts, verify that joining dimensions does not duplicate those amounts. Reconcile totals before and after the join.

## 7. Create or refresh a Gold table

A typical Delta table write is:

```python
# Illustrative pattern — confirm the project's actual implementation
gold_df.write.format("delta").mode("overwrite").saveAsTable(
    "retail.gold.gold_sales"
)
```

Overwrite is suitable for some full-refresh workloads but not automatically suitable for every production pipeline. Incremental workloads may require a merge/upsert strategy or another controlled update approach.

## 8. Validate the Gold output

Check the table row count:

```sql
SELECT COUNT(*) AS sales_count
FROM retail.gold.gold_sales;
```

The completed sample project returned **25 rows** for `retail.gold.gold_sales`.

Inspect sample records:

```sql
SELECT *
FROM retail.gold.gold_sales
LIMIT 10;
```

Check order uniqueness if one row per order is the intended grain:

```sql
SELECT order_id, COUNT(*) AS row_count
FROM retail.gold.gold_sales
GROUP BY order_id
HAVING COUNT(*) > 1;
```

Run this uniqueness check only if `order_id` is expected to be unique in the Gold table. If the table is designed at order-line grain, the correct key may be a combination of order ID and line ID.

## 9. Data grain and modeling

**Grain** means what one row represents.

Examples:

- One row per order.
- One row per order line.
- One row per customer.
- One row per store and day.

The grain should be decided before calculating metrics. Otherwise, joins and aggregations can create misleading totals.

## 10. Common issues

**Unresolved column error:** Check the schema of each input table and use the correct column name.

**Gold row count exceeds expectations:** Inspect join cardinality and duplicate lookup keys.

**Missing dimension attributes:** Check for unmatched foreign keys and determine whether missing values are acceptable.

**Incorrect revenue total:** Confirm the table grain, join behavior, and measure definition.

**Table is not visible:** Confirm that it was written to the expected catalog and schema.

## 11. Interview explanation

“I implemented a Gold business layer in Databricks to create a curated sales dataset from the retail lakehouse. The purpose is to make data easier to consume for reporting and analytics. I validate row counts, table grain, join behavior, and business totals to ensure the curated output remains trustworthy.”

## 12. Key takeaways

- Gold is designed for business consumption.
- Table grain must be explicit.
- Joins must not unintentionally multiply records.
- Business metrics need reconciliation.
- The completed project validated `retail.gold.gold_sales` with 25 rows.

## 13. Related project artifact

Review `03_Gold_Business_Layer.py` and confirm the actual join logic, Gold table schema, and validation performed in the project.
