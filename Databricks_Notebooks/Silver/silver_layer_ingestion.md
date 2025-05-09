
# ⚪ Silver Layer – Parameterized Ingestion from Bronze to Delta

This Databricks notebook enables **dynamic, reusable ingestion** of Netflix-related datasets from the **Bronze** layer (CSV) to the **Silver** layer (Delta Lake format) using `dbutils.widgets`.

## Objective

Transform semi-structured CSVs in the Bronze layer into **optimized Delta format** tables in the Silver layer, using **parameterized paths** for maximum reusability.

---
##  Prerequisite

- Azure Databricks Workspace
- Delta Lake Enabled
- Access to ADLS Gen2 with proper `abfss://` paths
- Permissions to read from Bronze and write to Silver

---

## Notebook Workflow Summary

### Step 1: Define Input Parameters (Widgets)
```python
dbutils.widgets.text("sourcefolder", "netflix_directors")
dbutils.widgets.text("targetfolder", "netflix_directors")
```
Creates dynamic inputs for dataset names (source and target folders).

---

### Step 2: Retrieve Widget Values
```python
var_src_folder = dbutils.widgets.get("sourcefolder")
var_trg_folder = dbutils.widgets.get("targetfolder")
```
Fetches the values entered in widgets for use in path-building.

---

### Step 3: Read from Bronze (CSV)
```python
df = spark.read.format("csv") \
    .option("header", True) \
    .option("inferSchema", True) \
    .load(f"abfss://bronze@bronze_layer.dfs.core.windows.net/{var_src_folder}")
```
Reads CSV data from a dynamically defined Bronze path.

---

### Step 4: Write to Silver (Delta)
```python
df.write.format("delta") \
    .mode("append") \
    .option("path", f"abfss://silver@silver_layer.dfs.core.windows.net/{var_trg_folder}") \
    .save()
```
Writes the data in Delta format to the Silver container.

---

## Why Parameterize?

This notebook is built with **widgets** to make it fully **reusable across datasets**. Just provide different folder names without changing any code.

### Example Widget Inputs

| `sourcefolder`     | `targetfolder`       |
|--------------------|----------------------|
| `netflix_cast`     | `netflix_cast`       |
| `netflix_directors`| `netflix_directors`  |
| `netflix_category` | `netflix_category`   |

---

## Optional: Orchestrate with Azure Data Factory (ADF)

Use Azure Data Factory to run this notebook in a loop using a **ForEach** activity.

### ADF ForEach Input Example
```json
[
  { "sourcefolder": "netflix_cast", "targetfolder": "netflix_cast" },
  { "sourcefolder": "netflix_directors", "targetfolder": "netflix_directors" },
  { "sourcefolder": "netflix_category", "targetfolder": "netflix_category" }
]
```

### Databricks Notebook Activity (inside ADF ForEach)

| Parameter Name | Value                       |
|----------------|-----------------------------|
| sourcefolder   | `@item().sourcefolder`      |
| targetfolder   | `@item().targetfolder`      |

---

## Benefits

| Feature         | Benefit                                         |
|----------------|--------------------------------------------------|
| Widgets         | Reusable for any dataset                        |
| Dynamic Paths   | No hardcoded folder names                       |
| CSV to Delta    | Optimized for analytics                         |
| Scalable Logic  | Works with ForEach loops or automated triggers  |

---

## Example Output Paths

| Input Value        | Output Path                                  |
|--------------------|-----------------------------------------------|
| `netflix_cast`     | `abfss://silver/.../netflix_cast/`            |
| `netflix_directors`| `abfss://silver/.../netflix_directors/`       |



