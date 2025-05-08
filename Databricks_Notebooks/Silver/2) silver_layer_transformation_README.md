# ⚪ Silver Layer – Azure Databricks ETL for Cleaned Netflix Data

This repository documents the Silver Layer in a modern data lakehouse architecture using **Azure Databricks**. It contains a PySpark pipeline that transforms and enriches raw Netflix data from the Bronze Layer and stores it as query-optimized Delta tables in the Silver Layer.

---

## Objective

Transform bronze-layer Netflix data by:
- Cleaning missing values
- Casting data types
- Creating derived columns
- Ranking by duration
- Writing refined data to the **Silver container** in Delta format

---

## Step-by-Step Guide

### Step 1: Open Databricks Notebook

1. Go to Azure Databricks Workspace → Repos or Workspace
2. Click **Create > Notebook**
3. Name it: `Silver_Netflix_Transform`
4. Select language: Python

---

### Step 2: Read Data from Bronze Layer

```python
from pyspark.sql.functions import *
from pyspark.sql.types import *
from pyspark.sql.window import Window

bronze_path = "abfss://bronze@netflixprojectdlshahid.dfs.core.windows.net/netflix_titles"
df = spark.read.format("delta").load(bronze_path)
df.printSchema()
df.display()
```

This loads raw bronze Delta data for processing.

---

### Step 3: Clean and Normalize the Data

```python
df = df.fillna({"duration_minutes": 0, "duration_seasons": 1})
df = df.withColumn("duration_minutes", col("duration_minutes").cast(IntegerType()))
df = df.withColumn("duration_seasons", col("duration_seasons").cast(IntegerType()))
```

Handles nulls and ensures numeric fields are cast correctly.

---

### Step 4: Create New Derived Columns

```python
df = df.withColumn("Shorttitle", split(col("title"), ":")[0])
df = df.withColumn("rating", split(col("rating"), "-")[0])
df = df.withColumn("type_flag", when(col("type") == "Movie", 1)
                   .when(col("type") == "TV Show", 2)
                   .otherwise(0))
```

Adds fields that are useful for analytics and filtering.

---

### Step 5: Add Ranking Analytics

```python
df = df.withColumn("duration_ranking", dense_rank()
                   .over(Window.orderBy(col("duration_minutes").desc())))
```

Creates a ranking column for analyzing top-duration titles.

---

### Step 6: Write Transformed Data to Silver Layer

```python
silver_path = "abfss://silver@netflixprojectdlshahid.dfs.core.windows.net/netflix_titles"
df.write.format("delta") \
  .mode("overwrite") \
  .option("overwriteSchema", "true") \
  .option("path", silver_path) \
  .save()
```

Saves final data to the **Silver zone** in Delta format for analytics.

---

## Code Function Summary

| Transformation | Description |
|----------------|-------------|
| `fillna()` | Fills missing durations with defaults |
| `cast(IntegerType())` | Converts string durations to numeric |
| `split(...)[0]` | Extracts short title or base rating |
| `when(...).otherwise(...)` | Adds numeric flags for type filtering |
| `dense_rank()` | Enables top-N and trend analysis |
| `.write.format("delta")` | Saves cleaned data to Silver layer |

---

## Example Output

After running the notebook:
- Output path: `silver/netflix_titles`
- Format: Delta
- Ready for use in BI tools or Gold Layer ETL

---

## Summary

This Silver Layer cleans and structures raw Netflix data to prepare it for advanced querying, analytics, or reporting in the Gold Layer. It’s the heart of any data refinement process in a medallion architecture.

---

## 📎 Resources

- [Databricks Delta Lake](https://docs.delta.io/latest/delta-intro.html)
- [Azure Data Lake Storage Gen2](https://learn.microsoft.com/en-us/azure/storage/blobs/data-lake-storage-introduction)
- [PySpark API Docs](https://spark.apache.org/docs/latest/api/python/)
