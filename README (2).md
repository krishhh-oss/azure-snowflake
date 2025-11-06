# Data Loading from Azure Cloud to Snowflake

## Overview
This repository provides a step-by-step guide and automation scripts for loading data from **Azure Cloud (Blob Storage)** into **Snowflake** using the **saanalyst** database. It includes SQL scripts, Python-based automation, and configuration details to streamline the data ingestion process.

## Prerequisites
- **Snowflake Account** with access to the `saanalyst` database, schema, and warehouse
- **Azure Storage Account** with Blob Storage enabled
- **Azure SAS Token or Access Keys** for authentication
- **Storage Integration** set up between Snowflake and Azure
- **Azure CLI** (optional for testing and validation)

## Data Loading Process

### **1. Create a Storage Integration**
```sql
CREATE STORAGE INTEGRATION snow_azure_int
  TYPE = EXTERNAL_STAGE
  STORAGE_PROVIDER = AZURE
  ENABLED = TRUE
  AZURE_TENANT_ID = 'b3745745-b1bb-4e60-9fd9-cf5b2cc9cb0e'
  STORAGE_ALLOWED_LOCATIONS = ('azure://snowazureintg13.blob.core.windows.net/snowflakeaxurefile');
```

### **2. Create Database and Schema**
```sql
CREATE DATABASE IF NOT EXISTS saanalyst;
CREATE SCHEMA IF NOT EXISTS saanalyst.file_formats;
CREATE SCHEMA IF NOT EXISTS saanalyst.external_stages;
```

### **3. Define File Format**
```sql
CREATE OR REPLACE FILE FORMAT saanalyst.file_formats.csv_fileformat
    TYPE = CSV
    FIELD_DELIMITER = '|'
    SKIP_HEADER = 1
    EMPTY_FIELD_AS_NULL = TRUE;
```

### **4. Create an External Stage**
```sql
CREATE OR REPLACE STAGE saanalyst.external_stages.stg_azure_cont
    URL = 'azure://snowazureintg13.blob.core.windows.net/snowflakeaxurefile'
    STORAGE_INTEGRATION = snow_azure_int
    FILE_FORMAT = saanalyst.file_formats.csv_fileformat;
```

### **5. Load Data into Snowflake**
```sql
COPY INTO saanalyst.public.customer_data
    FROM @saanalyst.external_stages.stg_azure_cont
    FILE_FORMAT = (TYPE = 'CSV', SKIP_HEADER = 1);
```

## Verification
- **Check if files are available in the stage:**
```sql
LIST @saanalyst.external_stages.stg_azure_cont;
```
- **Verify if data is loaded correctly:**
```sql
SELECT * FROM saanalyst.public.customer_data LIMIT 10;
```

## Security Considerations
- **Never hardcode credentials** – Use environment variables or secrets management tools.
- **Restrict Snowflake and Azure access** – Apply least privilege access policies.
- **Monitor Data Loads** – Use Snowflake’s query history and Azure logs for troubleshooting.

## Troubleshooting
- If data isn't loading, check Snowflake’s **`COPY_HISTORY`**:
  ```sql
  SELECT * FROM TABLE(INFORMATION_SCHEMA.LOAD_HISTORY())
  ORDER BY LAST_LOAD_TIME DESC;
  ```
- Verify Azure Blob permissions and SAS token validity.
- Check the Snowflake stage URL format and file naming conventions.

## License
This project is licensed under the MIT License. See the `LICENSE` file for details.

## Contributors
Feel free to contribute! Fork this repository, make improvements, and submit a pull request.


