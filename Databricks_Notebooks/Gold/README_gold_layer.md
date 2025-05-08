
# 🟡 Gold Layer – Delta Live Tables (DLT) for Final Trusted Netflix Data

I will explain how to use **Delta Live Tables (DLT)** in Azure Databricks to build the Gold Layer in a Medallion Architecture. This layer provides fully cleaned, validated, and BI-ready Netflix datasets.

---

## Purpose

The **Gold Layer** serves as the **final presentation-ready layer** of your data pipeline. It ensures:

- Cleaned and validated data
- Removal of nulls or incorrect values
- Readiness for BI tools, dashboards, and reporting

---

## Technology Used

- **Azure Databricks**
- **Delta Lake**
- **Delta Live Tables (DLT)**

---

## What Is Delta Live Tables (DLT)?

Delta Live Tables allows you to define pipelines using Python with special decorators like `@dlt.table`, `@dlt.view`, and `@dlt.expect`.

DLT automatically:

- Reads source data
- Applies transformations
- Performs data quality checks
- Writes validated Delta tables
- Handles dependencies between stages

---

## Step-by-Step Implementation

### Step 1: Ingest and Validate from Silver

```python
@dlt.table(name = "gold_netflixcategory")
@dlt.expect_or_drop("rule1", "show_id IS NOT NULL")
def load_category():
    return spark.readStream.format("delta").load("abfss://silver@netflixprojectdlshahid.dfs.core.windows.net/netflix_category")
```

✔️ Reads from the `netflix_category` table in Silver  
✔️ Applies a validation: drops rows with missing `show_id`  
✔️ Stores data in `gold_netflixcategory` table

---

### Step 2: Stage Main Titles Data

```python
@dlt.table
def gold_stg_netflixtitles():
    return spark.readStream.format("delta").load("abfss://silver@netflixprojectdlshahid.dfs.core.windows.net/netflix_titles")
```

✔️ Stages the main `netflix_titles` from Silver  
✔️ Stores in `gold_stg_netflixtitles`

---

### Step 3: Transform Data

```python
@dlt.view
def gold_trns_netflixtitles():
    df = spark.readStream.table("LIVE.gold_stg_netflixtitles")
    return df.withColumn("newflag", lit(1))
```

✔️ Adds a `newflag` column  
✔️ Prepares a transformed view for downstream use

---

### Step 4: Create Final Gold Table with Validations

```python
@dlt.table
@dlt.expect_all_or_drop({
    "rule1": "newflag IS NOT NULL",
    "rule2": "show_id IS NOT NULL"
})
def gold_netflixtitles():
    return spark.readStream.table("LIVE.gold_trns_netflixtitles")
```

✔️ Reads from the transformed view  
✔️ Drops invalid rows  
✔️ Stores result in `gold_netflixtitles`

---

## Final Outcome

| Table Name            | Description                        |
|----------------------|------------------------------------|
| `gold_netflixcategory` | Validated category table          |
| `gold_netflixtitles`   | Final trusted Netflix titles data |

These tables are clean, validated, and **ready for consumption** by business users and analytics tools.

---

## Benefits

- Data quality ensured
- Supports streaming ingestion
- BI-ready, trusted datasets
- Easy to update and maintain

---

## Future Enhancements

- Add more rules using `@dlt.expect` or `@dlt.expect_all_or_drop`
- Include joins to enrich data
- Add aggregations for KPIs

