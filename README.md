## Overview

This project demonstrates how to build a **modular**, **scalable**, and **dynamic data ingestion pipeline** using **Azure Data Factory (ADF)** to retrieve datasets from **GitHub** and store them in **Azure Data Lake Storage Gen2**. It lays the foundation for a complete **ETL pipeline** aligned with the **Medallion Architecture**, enabling clean separation of raw, refined, and curated data layers (**Raw → Bronze → Silver → Gold**).

The pipeline leverages **parameterization**, **Web Activity-based validation**, and **ForEach control flow** to dynamically ingest multiple files without requiring manual updates to the code or pipeline.

---

## This project will cover the following 

 1) ### Leverage Azure Data Factory to:
      - Create Azure Data Lake – configured with hierarchical namespace
      - External data ingestion using HTTP connection with GitHub
      - Dynamically ingests data from GitHub using 'ForEach' logic
      - Validates file existence
      - Writes files to structured folders in Azure Data LakeAzure Data Factory
- Pipeline control using Web Activity and Set Variable
- Modular, scalable ETL architecture based on Medallion principles (Raw → Bronze → Silver → Gold)

---

## 1) Azure Data Factory: Dynamic GitHub to Data Lake Ingestion Pipeline

### Goal:
Build a reusable pipeline that:
- Validates file existence
- Dynamically ingests data from GitHub
- Writes files to structured folders in Azure Data Lake
![image](https://github.com/user-attachments/assets/ee6d4fa9-f49f-4f3c-86a8-cf95c8b6740e)
---

## Flow Diagram Components (Explained)

### Web Activity – Metadata
- **Purpose**: Sends an HTTP **GET request** to GitHub to validate that a file exists before processing.
- **Example URL**:
### Step-by-Step Flow

1. **Web Activity (GET Method)**  
   Makes a request to GitHub's API (or file URL) to check file availability or retrieve metadata.

2. **Set Variable**  
   Stores response data from Web Activity, such as `statusCode`.

3. **Validation**  
   Checks if the base file exists. Pipeline only continues if validation passes (`statusCode == 200`).

4. **ForEach**  
   Iterates over a JSON array of files with `folder_name` and `file_name` values.

5. **Copy Data Activity**  
   Inside ForEach, the following happens:
   
   - **Source**: GitHub over HTTP  
     Dynamically constructs a file URL using parameterized values.
   
   - **Sink**: Azure Data Lake Gen2  
     Writes data to a structured folder path: `raw/{folder_name}/{file_name}`

---

## Setup Guide

### 1️ Create Azure Resources

- **Resource Group**
- **Storage Account**
  - Enable `Hierarchical Namespace` (required for Data Lake Gen2)
  - Create containers: `raw`, `bronze`, `silver`, `gold`

### 2️ Create Azure Data Factory (ADF)

- Go to ADF > Manage > **Linked Services**:
  - `GitHub_HTTP`: Base URL → `https://raw.githubusercontent.com/`
  - `AzureDataLakeGen2`: Link to the storage account

---

## JSON Parameter Config (Used in ForEach)

```json
[
  {
    "folder_name": "netflix_cast",
    "file_name": "netflix_cast.csv"
  },
  {
    "folder_name": "netflix_category",
    "file_name": "netflix_category.csv"
  },
  {
    "folder_name": "netflix_countries",
    "file_name": "netflix_countries.csv"
  },
  {
    "folder_name": "netflix_directors",
    "file_name": "netflix_directors.csv"
  }
]
