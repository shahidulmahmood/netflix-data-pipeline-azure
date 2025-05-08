##  Setup Steps

### Create Linked Services
![image](https://github.com/user-attachments/assets/6ef0c810-0b93-4fd1-b279-d2ff55996275)

#### HTTP Linked Service (GitHub)
- Go to **Manage** tab > **Linked Services** > **+ New**
- Choose **HTTP**
- Set:
  - **Name**: `GitHub_LinkedService`
  - **Base URL**: `https://raw.githubusercontent.com/`
  - **Authentication**: Anonymous (for public repos)
- Click **Create**

#### Azure Data Lake Gen2 Linked Service
- Go to **Linked Services** > **+ New**
- Choose **Azure Data Lake Storage Gen2**
- Set:
  - **Name**: `DataLake_LinkedService`
  - Choose your storage account
- Click **Create**

---

### Create Pipeline with Parameters

- Create two parameters in your pipeline:
  - `folder_name`
  - `file_name`

---

### Set Up Datasets

#### HTTP Dataset (GitHub)
- Linked Service: `GitHub_LinkedService`
- Relative URL: dynamic
  - Use:
    ```json
    @concat('<user>/<repo>/main/', pipeline().parameters.folder_name, '/', pipeline().parameters.file_name)
    ```

#### Azure Data Lake Dataset
- Linked Service: `DataLake_LinkedService`
- File path: dynamic
  - Use:
    ```json
    @concat(pipeline().parameters.folder_name, '/', pipeline().parameters.file_name)
    ```

---

### Create a ForEach Activity

- Input: an array of JSON like this:

```json
[
  { "folder_name": "netflix_cast", "file_name": "netflix_cast.csv" },
  { "folder_name": "netflix_category", "file_name": "netflix_category.csv" }
]
```

- Inside the loop:
  - Use **Copy Data** activity
  - Map parameters:
    - `@item().folder_name`
    - `@item().file_name`

---

## Summary

| Step | Purpose |
|------|---------|
| HTTP Linked Service | Connect ADF to GitHub |
| Data Lake Linked Service | Enable write access to ADLS |
| Parameters | Make ingestion dynamic |
| ForEach | Loop through multiple datasets |
| Copy Data | Perform file transfer from GitHub to ADLS |

---

## Outcome

A fully reusable and scalable ADF pipeline that:
- Connects to GitHub without hardcoding file paths
- Dynamically ingests multiple files with one flow
- Cleanly organizes raw files in your Azure Data Lake

---

## Folder Structure

```
adf-github-ingestion/
├── adf/
│   ├── pipelines/
│   ├── datasets/
│   └── linkedServices/
├── README.md
```

---

## ✍️ Author

Built by [Your Name].  
Let me know if you'd like help creating a downloadable JSON template of this pipeline.
