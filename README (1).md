
# Netflix Data Engineering Pipeline on Azure

This project demonstrates how to build an end-to-end data ingestion and processing pipeline using Microsoft Azure services. It ingests Netflix dataset CSVs from GitHub and processes them into a layered Delta Lake architecture using Azure Data Factory, Azure Data Lake Storage Gen2, and Azure Databricks with Autoloader.

---

## Project Overview

This pipeline is designed to:

- Automatically ingest CSV files from a GitHub repository.
- Store raw files in a hierarchical Azure Data Lake (Raw → Bronze → Silver → Gold).
- Use Azure Data Factory for dynamic ingestion using parameterized pipelines.
- Transform and stream files using Azure Databricks Autoloader.
- Structure data using the Delta Lake format for scalable querying.

---

## Technologies Used

- Azure Resource Group
- Azure Data Lake Storage Gen2
- Azure Data Factory (ADF)
- Azure Databricks
- Delta Lake
- Autoloader
- GitHub (as source system)

---

## Architecture Overview

```
├── adf/                                  # Azure Data Factory resources
│   ├── pipelines/                        # Pipeline definitions
│   ├── datasets/                         # Dataset definitions
│   └── linkedServices/                   # Linked service configurations
├── databricks/                           # Databricks notebooks
│   ├── bronze_to_silver/                 # Data transformation notebooks
│   ├── silver_to_gold/                   # Data aggregation notebooks 
│   └── utilities/                        # Helper functions and utilities
├── infrastructure/                       # IaC templates
│   ├── arm_templates/                    # ARM templates for Azure resources
│   └── terraform/                        # Terraform scripts (alternative)
├── tests/                                # Test scripts and configurations
├── docs/                                 # Additional documentation
└── README.md                             # Project overview
```

---

## Step-by-Step Setup

### 1. Azure Setup

- Create a **Resource Group**.
- Create a **Storage Account** with **Hierarchical Namespace** enabled.
- Create four containers:
  - `raw`, `bronze`, `silver`, `gold`

### 2. Azure Data Factory (ADF)

- Create two **Linked Services**:
  - HTTP (GitHub): `https://raw.githubusercontent.com/`
  - Azure Data Lake Storage Gen2 (connect to your storage account)

### 3. ADF Pipeline

- Create **Parameters** for folder and file names.
- Add **Copy Data** activity with:
  - Source: HTTP (GitHub)
  - Sink: Azure Data Lake (Raw container)
- Use a **ForEach** activity:
  - Loop through an array of JSON file objects.
  - Example:

```json
[
  { "folder_name": "netflix_cast", "file_name": "netflix_cast.csv" },
  { "folder_name": "netflix_category", "file_name": "netflix_category.csv" },
  ...
]
```

- Add a **Web Activity** (optional) to fetch metadata or validate files.
- Set variables as needed for dynamic behavior.

### 4. Azure Databricks Autoloader

```python
checkpoint_location = "abfss://silver@<your-storage>.dfs.core.windows.net/checkpoint"

df = spark.readStream  .format("cloudFiles")  .option("cloudFiles.format", "csv")  .option("cloudFiles.schemaLocation", checkpoint_location)  .load("abfss://raw@<your-storage>.dfs.core.windows.net")

df.writeStream  .option("checkpointLocation", checkpoint_location)  .trigger(processingTime='10 seconds')  .start("abfss://bronze@<your-storage>.dfs.core.windows.net/netflix_titles")
```

- Autoloader monitors new files and streams them into Bronze.
- Use follow-up notebooks for transformations from Bronze → Silver → Gold using Delta format.

---

## Folder Structure

```
netflix-data-pipeline-azure/
├── adf-pipeline/
│   └── pipeline-definition.json
├── databricks-notebooks/
│   ├── autoloader.py
│   ├── silver_transform.py
│   └── gold_transform.py
├── datasets/
│   └── netflix_cast.csv
├── docs/
│   └── architecture-diagram.png
└── README.md
```

---

## Key Learnings

- Cloud-native data pipelines with ADF and Databricks
- Parameterized and dynamic data ingestion
- Delta Lake architecture for modern data warehousing
- Streaming data ingestion with Autoloader
- Layered data lake design (Raw → Bronze → Silver → Gold)

---

## Final Result

A scalable Azure-based data pipeline capable of automatically ingesting, storing, and transforming data from external sources, following best practices for modern data engineering.

---

## Contact

Built by [Your Name]. Feel free to connect with me on [LinkedIn] or check out more of my work on [GitHub].
