
# Azure Data Factory: Dynamic File Ingestion from GitHub Using Parameters

This project demonstrates how to use **Azure Data Factory (ADF)** to dynamically ingest multiple CSV files from a public GitHub repository into Azure Data Lake Gen2. The pipeline is parameterized and uses a `ForEach` loop to scale the ingestion process with minimal effort. I will do a step by step guide on how this can be done.

---

## Tools & Services

- Azure Data Factory
- Azure Data Lake Storage Gen2
- GitHub (public repo)
- Azure Portal & ADF Studio UI

---

## Architecture Overview

1. Define file info in a JSON array
2. Use ADF pipeline with parameters for file name and folder name
3. GitHub (HTTP) as Source → Azure Data Lake Gen2 as Sink
4. Scalable via ForEach loop

---

## Folder Structure

```
adf-github-ingestion/
├── adf/
│   ├── pipelines/
│   ├── datasets/
│   └── linkedServices/
├── notebooks/
│   └── set_array_parameter.py
├── docs/
│   └── data-flow-diagram.png
├── README.md
```

---

## Azure Data Factory UI – Step-by-Step Setup

### 1. Launch ADF Studio

1. Go to [portal.azure.com](https://portal.azure.com)
2. Open your **Data Factory** resource
3. Click **"Launch Studio"** to open ADF Studio

**Summary**: This opens the development environment where you'll build the pipeline.

---

### 2. Create Linked Services

#### GitHub HTTP Linked Service
1. Click **Manage** (gear icon in left panel)
2. Click **Linked Services** → **+ New**
3. Select **HTTP** from connector options
4. Name: `GitHub_LinkedService`
5. Base URL: `https://raw.githubusercontent.com/`
6. Authentication Type: `Anonymous`
7. Click **Create**

#### Azure Data Lake Gen2 Linked Service
1. Click **+ New** again in Linked Services
2. Choose **Azure Data Lake Storage Gen2**
3. Name: `DataLake_LinkedService`
4. Choose your Storage Account
5. Use default or Managed Identity authentication
6. Click **Create**

**Summary**: These connections allow ADF to read from GitHub and write to Azure Data Lake.

---

### 3. Define Pipeline Parameters

1. Go to **Author** tab
2. Create a **New Pipeline**
3. In the pipeline's **Parameters** section (bottom panel), add:
   - `file_name` (String)
   - `targetfolder` (String)

**Summary**: These parameters make your pipeline reusable for any file path.

---

### 4. Create Datasets

#### HTTP Dataset (GitHub)
1. Under **Author > Datasets**, click **+ New Dataset**
2. Choose `HTTP`, then `DelimitedText`
3. Name: `GitHubSource`
4. Select `GitHub_LinkedService`
5. For **Relative URL**, click the dynamic content icon and use:
   ```
   @concat('shahidulmahmood/netflix-data-pipeline-azure/main/ADF Pipeline/', pipeline().parameters.file_name)
   ```
6. Click **OK**

#### ADLS Dataset (Sink)
1. Click **+ New Dataset**, choose `DelimitedText` with `Azure Data Lake Gen2`
2. Name: `DataLakeSink`
3. Linked Service: `DataLake_LinkedService`
4. File Path (dynamic):
   ```
   @concat(pipeline().parameters.targetfolder, '/', pipeline().parameters.file_name)
   ```
5. Click **OK**

**Summary**: These datasets configure your source (GitHub file) and destination (Data Lake path).

---

### 🔹 5. Add ForEach + Copy Activities

1. Drag a **ForEach** activity into your pipeline
2. In **Items**, paste:
---

```json
[
  { "file_name": "netflix_cast.csv", "targetfolder": "netflix_cast" },
  { "file_name": "netflix_directors.csv", "targetfolder": "netflix_directors" },
  { "file_name": "netflix_countries.csv", "targetfolder": "netflix_countries" },
  { "file_name": "netflix_category.csv", "targetfolder": "netflix_category" }
]
```

This defines a list of files to ingest, specifying their source and target folders. The array can be passed into the ADF pipeline using `dbutils.jobs.taskValues.set()`.


3. Inside ForEach → drag a **Copy Data** activity
4. Set **Source Dataset**: `GitHubSource`
   - File name param: `@item().file_name`
5. Set **Sink Dataset**: `DataLakeSink`
   - Target folder param: `@item().targetfolder`

**Summary**: ForEach reads from the array and performs a file copy operation per entry.

---

### 🔹 6. Validate and Trigger Pipeline

1. Click **Validate All** on the toolbar
2. Click **Debug** to test locally OR **Add Trigger > Trigger Now**

**Summary**: This runs the pipeline and copies files from GitHub to Data Lake.

---

## Benefits

| Feature        | Benefit                                           |
|----------------|---------------------------------------------------|
| Parameterized  | One pipeline handles many datasets                |
| ForEach Loop   | Easily scale to 10s or 100s of files              |
| GitHub + ADLS  | Seamless integration with public datasets         |
| UI Driven      | No need to write raw code in ADF                  |

---

## Final Outcome

A dynamic, reusable ADF pipeline that connects GitHub to Azure Data Lake and ingests multiple files using a parameterized loop—all configured via Azure's UI.
