# 🟡 Gold Layer – DLT with Validations

## 🔹 Purpose
Use Delta Live Tables (DLT) to build governed, production-grade datasets with enforced quality checks from the Silver layer.

## 🔹 Code Summary & Explanation
```python
@dlt.table(name = "gold_netflixcategory")
@dlt.expect_or_drop("rule1", "show_id is NOT NULL")
def load_category():
    return spark.readStream.format("delta").load("abfss://silver@<storage>.dfs.core.windows.net/netflix_category")
```
✔ Creates a validated table for category data with only non-null `show_id`.

```python
@dlt.table
def gold_stg_netflixtitles():
    return spark.readStream.format("delta").load("abfss://silver@<storage>.dfs.core.windows.net/netflix_titles")
```
✔ Stages Netflix titles as a streamable Delta table.

```python
@dlt.view
def gold_trns_netflixtitles():
    df = spark.readStream.table("LIVE.gold_stg_netflixtitles")
    return df.withColumn("newflag", lit(1))
```
✔ Adds a dummy transformation (`newflag`) and registers a view.

```python
@dlt.table
@dlt.expect_all_or_drop({"rule1": "newflag is NOT NULL", "rule2": "show_id is NOT NULL"})
def gold_netflixtitles():
    return spark.readStream.table("LIVE.gold_trns_netflixtitles")
```
✔ Final Gold table with multiple enforced quality checks.

## 🔹 What It Does
- Applies schema validation and cleanup via expectations.
- Creates production-ready, governed datasets for analytics or BI use.
