
# Netflix Data Engineering Pipeline on Azure

This project demonstrates how to build an end-to-end data ingestion and processing pipeline using Microsoft Azure services. It ingests Netflix dataset CSVs from GitHub and processes them into a layered Delta Lake architecture using Azure Data Factory, Azure Data Lake Storage Gen2, and Azure Databricks with Autoloader.

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

## Folder Structure

```
netflix-data-pipeline-azure/
├── ADF-pipeline/
│   └── Dynamic_Data_Ingestion.md
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
## Project Overview

![image](https://github.com/user-attachments/assets/03fd84e0-7346-4277-a2ab-d4cd53df5fef)

This pipeline is designed to:

- Dynamically ingest CSV files from a GitHub repository.
- Use Azure Data Factory for dynamic ingestion using parameterized pipelines.
- Transform and stream files using Azure Databricks Autoloader.
- Store raw files in a hierarchical Azure Data Lake (Raw → Bronze → Silver → Gold).
- Structure data using the Delta Lake format for scalable querying.

---

## Architecture Explained

### 1️ Ingestion Layer (Raw / Bronze)
- **GitHub** hosts raw `.csv` files.
- **Azure Data Factory**:
  - Uses **Web Activity** to validate data availability
  - Executes a **ForEach** loop over a list of files
  - Runs **Copy Data** activities to move files to the Data Lake (`raw/` folder)
- **Data Lake Gen2** serves as the storage Zone for the raw data.

---

### 2️ Transformation Layer (Silver)
- Managed via **Databricks Notebooks** or **DLT**
- Reads data from Raw/Bronze and:
  - Cleans data
  - Handles nulls
  - Casts data types
  - Creates new features (e.g., `ShortTitle`, `TypeFlag`)
- Saves "silver quality" data back to Data Lake in Delta format.

---

### 3️ Gold Layer – Serving
- **Delta Live Tables** create **validated** and **enriched** output tables
- Implements quality rules using:
  

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
