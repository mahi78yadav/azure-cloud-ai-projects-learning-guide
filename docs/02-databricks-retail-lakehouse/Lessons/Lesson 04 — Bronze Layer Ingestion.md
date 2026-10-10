# Lesson 04 — Bronze Layer Ingestion

## 1. Objective

Understand how the Azure Databricks Retail Lakehouse project loads source files into the Bronze layer.

By the end of this lesson, you should be able to explain:

- Why the Bronze layer exists.
- How source files are read in Databricks.
- How raw data is stored as Delta tables.
- How to validate the ingestion process.
- How Bronze fits into the Medallion Architecture.

## 2. Business requirement

A retail organization receives data from multiple business entities:

- Customers
- Products
- Orders
- Stores

The data may arrive as CSV files in cloud storage. Before the organization can perform reporting or analytics, it needs to ingest this data into a lakehouse.

The Bronze layer provides the initial landing point for structured data inside the lakehouse.

## 3. Architecture

The ingestion flow is:

```text
Source CSV files
       |
       v
Azure Data Lake Storage Gen2
       |
       v
Azure Databricks Notebook
01_Bronze_Ingestion.py
       |
       v
Bronze Delta Tables
       |
       v
Silver Transformation
```

The project uses the following components:

- **ADLS Gen2:** Stores source files.
- **Azure Databricks:** Executes ingestion notebooks.
- **Unity Catalog:** Organizes and governs lakehouse data.
- **Delta Lake:** Stores tables with transactional capabilities.

## 4. Source datasets

The project contains four primary datasets.

| Dataset | Purpose |
|---|---|
| Customers | Customer information |
| Products | Product details |
| Orders | Retail transactions |
| Stores | Store information |

The sample source data contains 10 customers, 10 products, 25 orders, and 7 stores.

These are sample-project counts, not production business volumes.

## 5. Bronze layer design

The Bronze layer stores data close to its source representation.

Typical responsibilities include:

1. Reading source files.
2. Preserving the original business fields.
3. Loading records into Bronze tables.
4. Maintaining traceability to source files where practical.
5. Making data available for downstream transformations.

Bronze should not normally contain complex business calculations. Those belong in later layers.

### Important design consideration

Raw does not necessarily mean completely untyped. CSV files are read using a schema or inferred types, depending on the implementation. For reliable production pipelines, explicitly defining and validating schemas is generally preferable to relying entirely on schema inference.

## 6. Notebook implementation

The project notebook is:

`01_Bronze_Ingestion.py`

Its responsibility is to read the source datasets and create or populate the Bronze layer tables.

The general processing pattern is:

```python
# Illustrative pattern — adapt paths and table names
# to the implementation in your repository.

df = (
    spark.read
    .option("header", "true")
    .option("inferSchema", "true")
    .csv(source_path)
)

display(df)
```

The example demonstrates reading a CSV file. It is not a claim that every line above is identical to the completed project notebook.

A production implementation should also address schema consistency, missing files, malformed records, duplicate records, and repeat execution.

## 7. Writing data to a Bronze Delta table

A typical Delta write pattern is:

```python
# Illustrative pattern
df.write.format("delta").mode("overwrite").saveAsTable(
    "retail.bronze.customers"
)
```

This example uses `overwrite` to demonstrate the syntax. **Do not replace an existing production or completed-project write mode without checking the intended load strategy.** Overwrite replaces existing table data; append adds records and can introduce duplicates if replayed without controls.

The project organizes its data under the `retail` catalog and the `bronze`, `silver`, and `gold` schemas.

## 8. Validation

After ingestion, validate the output rather than assuming that a successful notebook run means the data is correct.

Example checks:

```sql
SELECT COUNT(*) AS customer_count
FROM retail.bronze.customers;
```

```sql
SELECT *
FROM retail.bronze.customers
LIMIT 10;
```

Repeat equivalent checks for the products, orders, and stores tables, adjusting table names if the repository uses different names.

For the sample data, the expected source counts are:

| Dataset | Expected rows |
|---|---:|
| Customers | 10 |
| Products | 10 |
| Orders | 25 |
| Stores | 7 |

Compare source and target counts when the load is intended to preserve all source rows.

## 9. Common issues

**File not found:** Verify the storage path, container, directory, and file name.

**Permission denied:** Check the identity and permissions used by the Databricks workspace.

**Unexpected column types:** Inspect schema inference and consider an explicit schema.

**Duplicate rows:** Check whether the notebook is appending the same input repeatedly.

**Empty table:** Confirm that the input file contains data and that the correct table and schema are being queried.

**Table not found:** Confirm the current catalog, schema, and fully qualified table name.

## 10. Interview explanation

“I implemented a Bronze ingestion layer in Azure Databricks to bring customer, product, order, and store data from ADLS Gen2 into Delta tables. This layer provides a consistent landing point for downstream Silver transformations. I validate ingestion using row counts and sample-record checks before moving data forward.”

## 11. Key takeaways

- Bronze is the initial lakehouse data layer.
- ADLS Gen2 stores the source files.
- Databricks notebooks perform ingestion.
- Delta tables provide a managed tabular representation.
- Successful execution must be followed by data validation.
- The ingestion strategy must be designed for safe reruns.

## 12. Related project artifact

Review `01_Bronze_Ingestion.py` in the Databricks project repository and compare the actual source paths, table names, and write modes with the illustrative patterns in this lesson.
