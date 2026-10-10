# Lesson 07 — Auto Loader and Incremental Ingestion

## 1. Objective

Understand how Databricks Auto Loader can discover and process new files arriving in cloud storage.

By the end of this lesson, you should be able to explain:

- Why recurring file ingestion needs a different approach from a one-time batch.
- What Auto Loader does.
- The role of checkpoints.
- How to validate ingestion.
- Why duplicate prevention and idempotency matter.

## 2. Business requirement

Retail source systems may generate new files throughout the day. A pipeline should be able to process new arrivals without unnecessarily rereading every previously processed file.

For example, a retail order feed may receive additional orders after the initial dataset has been loaded.

Auto Loader is designed to incrementally discover and process new files from supported cloud storage locations.

## 3. Project implementation

The project notebook is:

`04_AutoLoader_Ingestion.py`

The completed project was tested with an orders dataset containing 25 records.

Auto Loader is the ingestion mechanism demonstrated in this notebook. It should not be confused with the separate scheduled Databricks job or with the project's Bronze-to-Silver-to-Gold transformations.

## 4. Architecture

```text
New files in ADLS Gen2
          |
          v
     Auto Loader
          |
          v
   Streaming DataFrame
          |
          v
   Delta destination
          |
          v
   Validation and use
```

In Auto Loader, the `cloudFiles` source format is used to discover new files and read their data.

## 5. Key concepts

### A. `cloudFiles`

Auto Loader uses the `cloudFiles` source format to incrementally process files.

### B. Schema handling

Auto Loader supports schema inference and schema evolution options. The chosen configuration depends on whether new columns or changes in source structure should be accepted.

Schema evolution should be deliberate. Automatically accepting unexpected changes may affect downstream transformations.

### C. Checkpoint

A checkpoint stores streaming progress and state so that the stream can resume after interruption.

Use a stable, dedicated checkpoint location. Do not share one checkpoint directory between unrelated streaming queries.

### D. Trigger and execution

A streaming query can run continuously or process available data and stop, depending on its trigger configuration.

A scheduled Databricks job and a streaming trigger are related operational concepts, but they are not the same thing.

## 6. Illustrative Auto Loader pattern

The following example illustrates the main components. Replace the paths, schema settings, and destination table with the values used by the actual notebook.

```python
from pyspark.sql import functions as F

source_path = "<cloud-storage-source-path>"
schema_location = "<cloud-storage-schema-location>"
checkpoint_location = "<cloud-storage-checkpoint-location>"

orders_stream = (
    spark.readStream
    .format("cloudFiles")
    .option("cloudFiles.format", "csv")
    .option("header", "true")
    .option("cloudFiles.schemaLocation", schema_location)
    .load(source_path)
)

query = (
    orders_stream.writeStream
    .format("delta")
    .option("checkpointLocation", checkpoint_location)
    .toTable("retail.bronze.orders_autoloader")
)
```

This is a conceptual example, not a transcription of the completed project code. Check the actual notebook for its source path, destination, schema options, and whether it uses a trigger or continuous execution.

The `F` import is included for Spark transformations if required; the ingestion example itself does not use it.

## 7. Why checkpoints matter

Imagine a stream processes a set of files and then stops unexpectedly.

With correctly configured checkpointing, a restarted query can recover its recorded progress instead of treating every run as a completely new query.

Checkpointing alone does not guarantee that the entire business pipeline is duplicate-free. Downstream writes, business keys, retries, and the way the stream is restarted must also be considered.

Do not delete a checkpoint casually. Removing it may cause the stream to reprocess data or lose its previous progress, depending on the configuration.

## 8. Validate ingestion

The completed project recorded 25 rows for the Auto Loader orders dataset.

A basic validation query is:

```sql
SELECT COUNT(*) AS order_count
FROM retail.bronze.orders_autoloader;
```

Use the actual destination table name from `04_AutoLoader_Ingestion.py` if it differs from this illustrative name.

Additional checks:

- Inspect a sample of records.
- Confirm that the expected source files were discovered.
- Check streaming progress and error details.
- Verify checkpoint and schema locations.
- Add a test file and confirm the intended ingestion behavior.
- Confirm that reruns do not create unintended duplicates.

## 9. Auto Loader versus batch ingestion

| Topic | Batch file ingestion | Auto Loader |
|---|---|---|
| Typical use | Process a defined set of files | Incrementally discover new files |
| Execution | Often scheduled | Streaming query or triggered micro-batches |
| Progress tracking | Depends on pipeline logic | Uses streaming checkpoints |
| Schema handling | Defined by batch implementation | Supports configurable inference and evolution |
| Operational considerations | Reruns and file selection | Checkpoints, stream recovery, schema changes |

Neither approach is automatically better for every workload. Select the ingestion method based on latency, volume, source behavior, and operational requirements.

## 10. Common issues

**No new records appear:** Confirm the source path, new file location, query status, and checkpoint state.

**Permission error:** Verify the identity and storage permissions.

**Schema mismatch:** Inspect schema settings and source-file structure.

**Duplicates after restart:** Review checkpoint continuity and the destination write strategy.

**Stream stops unexpectedly:** Check the job output, streaming exception, trigger configuration, and resource availability.

## 11. Interview explanation

“I used Databricks Auto Loader to demonstrate incremental file ingestion for the retail orders dataset. Auto Loader uses the `cloudFiles` source to discover new files, while checkpointing maintains streaming progress. I validate the destination data and consider restart behavior, schema changes, and duplicate prevention.”

## 12. Key takeaways

- Auto Loader supports incremental file discovery.
- Checkpoints are important for recovery and progress tracking.
- Schema evolution needs an explicit policy.
- A successful stream does not eliminate the need for data validation.
- The completed project validated 25 orders in its Auto Loader dataset.

## 13. Related project artifact

Review `04_AutoLoader_Ingestion.py` and identify the actual source path, schema location, checkpoint location, destination, and query trigger.
