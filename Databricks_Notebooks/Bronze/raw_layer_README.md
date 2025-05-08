# 🟤 Raw Layer – Autoloader Ingestion

## Purpose
Ingest raw CSV files from Azure Data Lake's `raw` container into the `bronze` layer using Databricks Autoloader.

## Code Summary
```python
%sql
CREATE SCHEMA netflix_catalog.net_schema;

checkpoint_location = "abfss://silver@<storage>.dfs.core.windows.net/checkpoint"

df = spark.readStream \
  .format("cloudFiles") \
  .option("cloudFiles.format", "csv") \
  .option("cloudFiles.schemaLocation", checkpoint_location) \
  .load("abfss://raw@<storage>.dfs.core.windows.net")

df.writeStream \
  .option("checkpointLocation", checkpoint_location) \
  .trigger(processingTime='10 seconds') \
  .start("abfss://bronze@<storage>.dfs.core.windows.net/netflix_titles")
```

## What It Does
- Uses Autoloader to detect new files in the `raw` container.
- Writes data incrementally to the `bronze` container.
- Tracks processed files using RocksDB checkpoints.
