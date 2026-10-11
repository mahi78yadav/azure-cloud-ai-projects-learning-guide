# PySpark Foundation for the Azure Databricks Retail Lakehouse

## Purpose

This study material covers the base PySpark knowledge needed to
understand the project's five notebooks:

-   `01_Bronze_Ingestion.py`
-   `02_Silver_Transformation.py`
-   `03_Gold_Business_Layer.py`
-   `04_AutoLoader_Ingestion.py`
-   `05_Data_Quality.py`

Learn these topics in order. Examples are illustrative: use the actual
paths, columns, table names, and write strategies from your notebooks.

## 1. PySpark basics

Apache Spark is a distributed data-processing engine. PySpark is its
Python API.

  Term             Meaning
  ---------------- --------------------------------------------------------
  SparkSession     Entry point for Spark
  DataFrame        Data represented in rows and named columns
  Schema           Column names and data types
  Transformation   Defines a change to a DataFrame
  Action           Triggers execution or returns a result
  Delta Lake       Table format supporting transactions and table history

Databricks notebooks usually provide the `spark` SparkSession.

``` python
from pyspark.sql import functions as F
```

Spark uses lazy evaluation: transformations generally define a plan, and
actions such as `show()`, `count()`, and writes trigger execution.

## 2. Read and inspect CSV files

``` python
customers_df = (
    spark.read
    .option("header", True)
    .option("inferSchema", True)
    .csv("<path-to-customers.csv>")
)

customers_df.show(5)
customers_df.printSchema()
customers_df.count()
print(customers_df.columns)
customers_df.describe().show()
```

  Method            Purpose
  ----------------- ---------------------------
  `show(5)`         Display sample rows
  `printSchema()`   Display columns and types
  `count()`         Count rows
  `columns`         Return column names
  `describe()`      Show basic statistics

`inferSchema=True` is convenient for learning. Production pipelines
often define schemas explicitly and validate incoming data.

## 3. Select, filter, and sort

``` python
customers_df.select("customer_id", "customer_name").show()

customers_df.filter(
    F.col("city") == "Hyderabad"
).show()

customers_df.filter(
    (F.col("age") >= 18) &
    (F.col("city") == "Hyderabad")
).show()

customers_df.orderBy(F.col("customer_id").desc()).show()
```

Remember: `&` means AND, `|` means OR, and `~` means NOT. Put
parentheses around each condition.

## 4. Create and convert columns

``` python
customers_df = customers_df.withColumn(
    "customer_name",
    F.trim(F.col("customer_name"))
)

orders_df = orders_df.withColumn(
    "order_amount",
    F.col("order_amount").cast("decimal(18,2)")
)
```

Useful functions:

  Function                Purpose
  ----------------------- -----------------------------------
  `withColumn()`          Add or replace a column
  `withColumnRenamed()`   Rename a column
  `cast()`                Convert data type
  `trim()`                Remove surrounding spaces
  `upper()` / `lower()`   Change text case
  `when().otherwise()`    Conditional logic
  `coalesce()`            First non-null expression
  `to_date()`             Convert supported values to dates

Validate cast results: malformed values may become null or produce
errors, depending on the operation and settings.

## 5. Nulls and duplicates

``` python
# Find rows missing a required identifier
customers_df.filter(
    F.col("customer_id").isNull()
).show()

# Keep only rows with the required identifier
valid_customers_df = customers_df.filter(
    F.col("customer_id").isNotNull()
)

# Remove identical rows
deduplicated_df = customers_df.dropDuplicates()

# Deduplicate by a business key
deduplicated_customers_df = customers_df.dropDuplicates(
    ["customer_id"]
)
```

Only remove nulls when the business rule requires the field.
`dropDuplicates(["customer_id"])` does not guarantee that the latest
record is retained. If duplicate keys have different values, define a
deterministic rule, such as retaining the latest timestamp.

## 6. Aggregations

``` python
sales_summary = (
    orders_df
    .groupBy("product_id")
    .agg(
        F.sum("order_amount").alias("total_sales"),
        F.countDistinct("order_id").alias("order_count"),
        F.avg("order_amount").alias("average_order_amount")
    )
)

sales_summary.show()
```

Know `sum`, `count`, `countDistinct`, `avg`, `min`, and `max`. Check
actual column names and table grain before calculating business
measures.

## 7. Joins

``` python
sales_df = (
    orders_df
    .join(customers_df, on="customer_id", how="left")
    .join(products_df, on="product_id", how="left")
)
```

  Join          Meaning
  ------------- ----------------------------------------
  `inner`       Only matching rows
  `left`        All left rows plus matching right data
  `right`       All right rows plus matching left data
  `full`        Rows from both sides
  `left_anti`   Left rows with no right-side match

Find orders without a matching customer:

``` python
unmatched_orders_df = orders_df.join(
    customers_df,
    on="customer_id",
    how="left_anti"
)
unmatched_orders_df.show()
```

**Important:** If the lookup side has duplicate join keys, the join can
multiply rows. Validate key uniqueness and row counts. Define the
table's grain---the business meaning of one row---before joining.

## 8. Medallion architecture

### Bronze --- `01_Bronze_Ingestion.py`

-   Read source CSV files.
-   Inspect schema and sample records.
-   Write to Bronze tables.
-   Validate initial counts.

### Silver --- `02_Silver_Transformation.py`

-   Standardize data.
-   Validate required fields.
-   Convert types.
-   Handle duplicates according to business rules.
-   Check relationships between entities.

### Gold --- `03_Gold_Business_Layer.py`

-   Join relevant datasets.
-   Create business-ready tables.
-   Validate table grain, joins, and business measures.

The documented Gold output is `retail.gold.gold_sales`. The completed
sample project recorded 25 rows; this is a sample result, not a
permanent production expectation.

## 9. Delta Lake basics

Read a table:

``` python
customers_df = spark.table("retail.silver.customers")
```

Illustrative full-refresh write:

``` python
df.write.format("delta").mode("overwrite").saveAsTable(
    "retail.silver.customers"
)
```

Write modes:

-   `overwrite`: replaces existing table data.
-   `append`: adds rows.
-   `errorifexists`: fails if the target already exists.
-   `ignore`: skips the write if the target already exists.

Choose the mode based on the load design. Do not replace the completed
project's write strategy without checking it.

Inspect history:

``` sql
DESCRIBE HISTORY retail.gold.gold_sales;
```

Time travel example:

``` sql
SELECT *
FROM retail.gold.gold_sales VERSION AS OF 0;
```

The version must exist and its data files must remain available. Time
travel is not an unlimited backup strategy. Learn Delta `MERGE INTO`
after mastering basic reads, transformations, and writes.

## 10. Auto Loader basics

The project notebook is `04_AutoLoader_Ingestion.py`. Auto Loader uses
`cloudFiles` to discover and process files incrementally.

Illustrative read:

``` python
stream_df = (
    spark.readStream
    .format("cloudFiles")
    .option("cloudFiles.format", "csv")
    .option("header", "true")
    .option("cloudFiles.schemaLocation", schema_location)
    .load(source_path)
)
```

Illustrative write:

``` python
query = (
    stream_df.writeStream
    .format("delta")
    .option("checkpointLocation", checkpoint_location)
    .toTable("retail.bronze.orders_autoloader")
)
```

Define the source, schema, and checkpoint paths. Confirm the actual
destination and configuration in the project notebook.

-   `readStream`: creates a streaming DataFrame.
-   `cloudFiles`: Auto Loader source.
-   `schemaLocation`: stores schema information.
-   `checkpointLocation`: stores progress and streaming state.
-   `writeStream`: writes streaming output.
-   Trigger: controls when processing runs.

Use a stable checkpoint for each stream. Do not casually delete or reuse
checkpoints, because this can change recovery and reprocessing behavior.

## 11. Data-quality checks

The project notebook is `05_Data_Quality.py`.

Count records:

``` python
print("Orders:", orders_df.count())
```

Find missing order IDs:

``` python
orders_df.filter(F.col("order_id").isNull()).show()
```

Find duplicate order IDs:

``` python
(
    orders_df.groupBy("order_id")
    .count()
    .filter(F.col("count") > 1)
    .show()
)
```

Use the duplicate check only if `order_id` is expected to be unique at
the table's grain.

Find orders without a matching customer:

``` python
orders_df.join(
    customers_df,
    on="customer_id",
    how="left_anti"
).show()
```

Validate Gold output:

``` sql
SELECT COUNT(*) AS sales_count
FROM retail.gold.gold_sales;
```

A successful job does not guarantee correct data. Validate schemas,
counts, keys, relationships, and business totals.

## 12. Common troubleshooting

### Unresolved column

Run `df.printSchema()` and inspect `df.columns`. Check spelling,
aliases, and earlier renames.

### File not found

Check storage account, container, directory, filename, and permissions.

### Unexpected row count

Check filters, joins, duplicate keys, source selection, and append
behavior.

### Null values after casting

Inspect malformed source values and define whether to reject, correct,
or quarantine them.

### Permission denied

Identify the identity used. Azure storage permissions and Unity Catalog
privileges are separate access-control layers.

### Auto Loader does not process a new file

Check the source path, query status, schema settings, checkpoint, and
whether the file was already processed.

## 13. Practice exercises

Use `customers.csv`, `products.csv`, `orders.csv`, and `stores.csv`.
Replace paths and column names with the actual values in your
environment.

1.  Read `customers.csv`; show five rows, print the schema, count rows,
    and list columns.
2.  Trim customer names; find null customer IDs and duplicate keys.
3.  Read orders; calculate total amount and sales by product.
4.  Join orders with customers; compare row counts before and after the
    join.
5.  Find orders with no matching customer.
6.  Write a test DataFrame to a separate Delta table; compare append and
    overwrite safely.
7.  Inspect Delta history.
8.  Identify the source, schema, checkpoint, and destination settings in
    the Auto Loader notebook.
9.  Validate counts, mandatory fields, duplicates, and relationships
    across the lakehouse.

## 14. Interview questions

1.  What is PySpark?
2.  What is a DataFrame?
3.  What is the difference between a transformation and an action?
4.  What is lazy evaluation?
5.  How do you read a CSV file?
6.  How do you filter rows and create columns?
7.  How do you handle nulls and duplicates?
8.  What is the difference between row deduplication and business-key
    deduplication?
9.  What are common join types?
10. How can a join increase row counts?
11. What are the Bronze, Silver, and Gold layers?
12. What is Delta Lake?
13. What is the difference between append and overwrite?
14. What is Auto Loader?
15. Why do schema and checkpoint locations matter?
16. How do you validate a pipeline?
17. How do you troubleshoot an unresolved column?
18. How do you verify that Gold business data is correct?

## 15. Recommended study order

1.  DataFrames, schemas, CSV reads, `show()`, and `count()`.
2.  `select`, `filter`, `withColumn`, `cast`, and text functions.
3.  Null handling and duplicate detection.
4.  Joins, `groupBy`, and aggregations.
5.  Delta tables and write modes.
6.  Bronze, Silver, and Gold notebook walkthrough.
7.  Auto Loader, streaming, and checkpoints.
8.  Data-quality checks and troubleshooting.

You can defer advanced RDD programming, complex Spark internals, custom
UDFs, and advanced performance tuning until needed.

## 16. Official references

-   PySpark API:
    https://spark.apache.org/docs/latest/api/python/index.html
-   PySpark on Databricks: https://docs.databricks.com/aws/en/pyspark/
-   Auto Loader:
    https://docs.databricks.com/aws/en/ingestion/cloud-object-storage/auto-loader/

The Databricks links above are in the AWS documentation section; consult
corresponding Azure Databricks documentation for cloud-specific
configuration.

## Completion target

You are ready to explain the PySpark foundation when you can read each
of the five project notebooks and describe what data is read, what
transformations occur, which tables are written, why the write strategy
is appropriate, how errors are handled, and how the output is validated.
