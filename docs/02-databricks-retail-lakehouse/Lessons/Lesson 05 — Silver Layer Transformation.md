# Lesson 05 — Silver Layer Transformation

## 1. Objective

Understand how the Silver layer converts ingested data into cleaner, more consistent datasets for analytics.

By the end of this lesson, you should be able to explain:

- Why Bronze data needs transformation.
- How Spark DataFrames support data cleansing.
- How to standardize and validate records.
- How to handle duplicates and missing values.
- How Silver tables support the Gold layer.

## 2. Business requirement

Source data is often suitable for ingestion but not immediately suitable for reporting.

Examples of potential issues include:

- Missing customer or product identifiers.
- Duplicate records.
- Inconsistent text values.
- Dates stored in inconsistent formats.
- Numeric values represented as strings.
- Orders referencing missing customers or products.

The Silver layer addresses data-quality and consistency requirements before data is used for business analytics.

## 3. Architecture

```text
Bronze Delta Tables
        |
        v
02_Silver_Transformation.py
        |
        v
Cleansing and standardization
        |
        v
Silver Delta Tables
        |
        v
Gold business layer
```

In the project, `02_Silver_Transformation.py` is the notebook associated with this processing stage.

## 4. Bronze versus Silver

| Area | Bronze | Silver |
|---|---|---|
| Primary purpose | Initial ingestion | Clean and standardize |
| Data representation | Close to source | Consistent and validated |
| Transformations | Usually limited | Cleansing and business-rule preparation |
| Quality checks | Basic ingestion checks | Stronger validity and consistency checks |
| Consumers | Transformation processes | Gold layer and other analytical workloads |

## 5. Common Silver transformations

### A. Inspect the schema

```python
df.printSchema()
display(df.limit(10))
```

This helps identify column names, data types, missing values, and unexpected source structures.

### B. Remove records that lack required identifiers

Illustrative example:

```python
from pyspark.sql import functions as F

clean_df = df.filter(F.col("customer_id").isNotNull())
```

Apply this only when `customer_id` is a required field for the particular dataset. Do not remove valid records simply because an optional field is null.

### C. Remove duplicates

```python
deduplicated_df = clean_df.dropDuplicates()
```

This removes identical rows. It does not necessarily identify duplicate business entities when other columns differ.

For business-key deduplication, define the correct key and decide which record should be retained. For example, a customer table may require one current record per customer ID.

### D. Standardize text

```python
standardized_df = deduplicated_df.withColumn(
    "customer_name",
    F.trim(F.col("customer_name"))
)
```

Standardization should follow the data contract. Avoid changing case or formatting when those changes would affect business meaning.

### E. Convert data types

```python
typed_df = standardized_df.withColumn(
    "order_amount",
    F.col("order_amount").cast("decimal(18,2)")
)
```

Validate conversion failures. Depending on the input and Spark settings, invalid values may become null or cause errors.

## 6. Write the Silver table

A typical Delta write pattern is:

```python
# Illustrative pattern — confirm the project's write strategy
typed_df.write.format("delta").mode("overwrite").saveAsTable(
    "retail.silver.customers"
)
```

The example shows the general write syntax, not a guarantee that every project table uses overwrite mode.

For incremental pipelines, use an appropriate strategy such as append, merge/upsert, or a controlled replacement, depending on the data and business requirements.

## 7. Referential integrity checks

Retail datasets often have relationships:

- Orders reference customers.
- Orders reference products.
- Stores are associated with orders or sales.

An example check for orders with no matching customer is:

```sql
SELECT o.*
FROM retail.silver.orders o
LEFT ANTI JOIN retail.silver.customers c
  ON o.customer_id = c.customer_id;
```

Replace the example key names if the project schema differs.

An empty result may indicate that all orders have matching customers. A non-empty result requires investigation; the records should not automatically be deleted without a business rule.

## 8. Validation checklist

- Confirm that all expected Silver tables exist.
- Compare Bronze and Silver row counts.
- Investigate unexpected count reductions.
- Check mandatory fields for nulls.
- Check duplicate business keys.
- Validate data types and date fields.
- Check relationships between customers, products, and orders.
- Review sample records after transformation.

Silver counts may be lower than Bronze counts when invalid records are deliberately rejected. The reason should be documented.

## 9. Common issues

**Column not found:** Check the actual source schema and spelling of the column.

**Cast produces nulls:** Identify malformed values before conversion.

**Unexpected row-count decrease:** Review filters, deduplication rules, and joins.

**Duplicate business entities remain:** `dropDuplicates()` across all columns may not be sufficient.

**Join creates too many rows:** Check whether the lookup side has duplicate keys.

## 10. Interview explanation

“I used the Silver layer to standardize and prepare ingested retail datasets for downstream analytics. Typical responsibilities include schema validation, cleansing, deduplication where appropriate, type conversion, and checking relationships between business entities. The transformations are governed by data-quality rules rather than indiscriminately removing records.”

## 11. Key takeaways

- Silver improves consistency and usability.
- Data cleansing must follow business rules.
- Duplicate rows and duplicate business keys are different problems.
- Join quality and referential integrity matter.
- Validation should explain changes in row counts.
- Incremental workloads require a deliberate write strategy.

## 12. Related project artifact

Review `02_Silver_Transformation.py` and identify the actual transformations, input tables, output tables, and write modes used in the completed project.
