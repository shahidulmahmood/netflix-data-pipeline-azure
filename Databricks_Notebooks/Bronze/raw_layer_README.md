# 🟤 Raw Layer – Azure Databricks Autoloader Pipeline

I will provide a step-by-step guide for setting up a **streaming ingestion pipeline** using **Azure Databricks Autoloader** to continuously load CSV files from an Azure Data Lake Gen2 "raw" container into a "bronze" container in **Delta Lake** format.

---

## Objective

Leverage **Databricks Autoloader** to:
- Detect new CSV files in the `raw` container on ADLS Gen2
- Automatically ingest them as **Delta streaming tables** into the `bronze` layer

---

## Step-by-Step Guide

### Step 1: Ensure HNS Is Enabled

Ensure your Azure Storage Account has **Hierarchical Namespace (HNS)** enabled: This needs to be enabled on your Azure Data Lake Storage Gen2 (ADLS Gen2) account because Databricks Autoloader relies on directory-like behavior and file-level operations, which are only supported when HNS is enabled.

> Azure Portal → Storage Account → Configuration → Hierarchical Namespace = `Enabled`

---

### Step 2: Prepare Containers in ADLS Gen2

Create the following blob containers using **Azure Portal** or **Azure Storage Explorer**:
- `raw` – for input CSV files
- `bronze` – for streaming Delta output
- `silver`, `gold` – for future processing stages

---

### Step 3: Upload Sample CSV Files to `raw`

Manual upload:
1. Navigate to: `Storage Account → Containers → raw`
2. Click **Upload**
3. Select sample `.csv` files (e.g., `netflix_titles.csv`)

You can also automate this using **Azure Data Factory (ADF)** or GitHub actions.

---

### Step 4: Create Autoloader Notebook in Databricks

1. Go to: `Azure Databricks Workspace → Repos or Users`
2. Click **Create > Notebook**
3. Name: `Raw_Autoloader`
4. Language: Python
5. Paste the following code:

```python
checkpoint_location = "abfss://silver@netflixprojectdlshahid.dfs.core.windows.net/checkpoint"

# Read new CSV files using Autoloader
raw_df = spark.readStream   .format("cloudFiles")   .option("cloudFiles.format", "csv")   .option("cloudFiles.schemaLocation", checkpoint_location)   .load("abfss://raw@netflixprojectdlshahid.dfs.core.windows.net")

# Write stream to Bronze container in Delta format
raw_df.writeStream   .format("delta")   .option("checkpointLocation", checkpoint_location)   .trigger(processingTime='10 seconds')   .start("abfss://bronze@netflixprojectdlshahid.dfs.core.windows.net/netflix_titles")
```

6. Click **Run All**

---

## Code Breakdown

| Code | Purpose |
|------|---------|
| `.format("cloudFiles")` | Enables Databricks Autoloader |
| `cloudFiles.format = "csv"` | Specifies input format |
| `schemaLocation` | Stores and versions inferred schema |
| `.load(...)` | Monitors `raw` container for new files |
| `.writeStream.format("delta")` | Outputs to Delta format |
| `checkpointLocation` | Tracks progress to prevent duplicates |
| `.trigger(...)` | Frequency of streaming checks |
| `.start(...)` | Begins streaming to the `bronze` layer |

---

## Benefits of Autoloader

- Auto-detects new files
- Scalable for high-volume ingestion
- Schema inference and evolution supported
- Event-based or directory listing-based processing

---

## Example Use Case

1. Upload `netflix_titles.csv` to the `raw` container
2. Autoloader detects the file within seconds
3. Stream is written to `bronze` as Delta table
4. Progress and schema are automatically tracked

---

## Summary

This setup provides a robust **streaming ingestion pipeline** for the Raw Layer in a modern **data lakehouse** architecture. The use of **Databricks Notebooks** and **Azure Portal** creates a hybrid, low-code approach ideal for scalable ingestion.


---

## Resources

- [Databricks Autoloader Documentation](https://docs.databricks.com/ingestion/auto-loader/index.html)
- [Azure Storage Explorer](https://azure.microsoft.com/en-us/products/storage/storage-explorer/)
- [Delta Lake Documentation](https://docs.delta.io/latest/index.html)
