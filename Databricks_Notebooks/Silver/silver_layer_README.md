# ⚪ Silver Layer – Transformation and Cleansing

## Purpose
Transform data from the Bronze layer to apply cleaning, enrichments, and data type conversions before storing in the Silver layer.

## Code Summary
```python
from pyspark.sql.functions import * 
from pyspark.sql.types import *

df = spark.read.format("delta").load("abfss://bronze@<storage>.dfs.core.windows.net/netflix_titles")

df = df.fillna({"duration_minutes": 0, "duration_seasons": 1})
df = df.withColumn("duration_minutes", col('duration_minutes').cast(IntegerType()))
df = df.withColumn("duration_seasons", col('duration_seasons').cast(IntegerType()))

df = df.withColumn("Shorttitle", split(col('title'), ':')[0])
df = df.withColumn("rating", split(col('rating'), '-')[0])
df = df.withColumn("type_flag", when(col('type')=='Movie',1).when(col('type')=='TV Show',2).otherwise(0))

df = df.withColumn("duration_ranking", dense_rank().over(Window.orderBy(col('duration_minutes').desc())))

df.write.format("delta") \
  .mode("overwrite") \
  .option("path", "abfss://silver@<storage>.dfs.core.windows.net/netflix_titles") \
  .save()
```

## What It Does
- Cleans and casts data types.
- Adds flags and derived attributes.
- Writes clean, queryable data to the Silver layer in Delta format.
