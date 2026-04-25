# Azure Data Factory: End-to-End Customer Data Pipeline

## 📝 Project Overview
This project demonstrates a complete Cloud Data Engineering solution using **Azure Data Factory (ADF)**. The pipeline is designed to automate the process of ingesting raw customer data from an external source (GitHub/HTTP), performing complex data transformations, and storing the final processed data into **Azure Data Lake Storage (Gen2)**.

## 🏗️ Architecture Diagram
The following diagram illustrates the logical flow of the data through the pipeline: Ingestion, Transformation, and Loading.

![Architecture Diagram](https://github.com/Ahmed800363/Azure-Data-Factory-Customer-Project/blob/main/Image/Design%20Pipeline%201%20.png)

## 🛠️ Tech Stack & Azure Services
* **Azure Data Factory (ADF):** Used for orchestrating the entire data pipeline and managing data movement.
* **Azure Data Flow:** Used for building graphical data transformations without writing code.
* **Azure Storage Account (ADLS Gen2):** Acts as the final destination (Sink) for the processed CSV files.
* **GitHub/HTTP:** The source of the raw data files.
* **Linked Services & Datasets:** Configured for secure connection and data schema definition.

## 📖 Data Dictionary
To provide a clear understanding of the datasets used in this pipeline, here is a breakdown of the columns from both source tables before they are merged and transformed.

### 1. Customers Source Table (Table_Customer)
This dataset contains the full profile of the customers.

| Column Name | Description | Data Type |
| :--- | :--- | :--- |
| `Cst_ID` | Unique identifier for each customer (Primary Key). | Integer |
| `Firstname` | Customer's first name. | String |
| `LastName` | Customer's last name. | String |
| `Email` | Customer's contact email address. | String |
| `Phone` | Customer's contact phone number. | String |
| `Gender` | Gender of the customer. | String |
| `Country` | Country of residence. | String |

### 2. Jobs Source Table (Table_Jobs) 
This dataset contains the job assignments linked to customers.

| Column Name | Description | Data Type |
| :--- | :--- | :--- |
| `Job_ID` | Unique identifier for the job assignment. | Integer |
| `Cst_ID` | Foreign key used to link with the Customers table. | Integer |
| `Job_Title` | The professional title or role assigned. | String |

## 🔗 Technical Setup: Connection & Dataset Configuration
Before moving the data, I established secure connections between Azure Data Factory and the data sources using **Linked Services** and **Datasets**.

### 1. Linked Services Configuration
I configured an **HTTP Linked Service** to connect to the raw data repository. This involved setting up the base URL and authentication methods to ensure a stable connection.

![Linked Service Setup](https://github.com/Ahmed800363/Azure-Data-Factory-Customer-Project/blob/main/Image/photo%201.png)
![Connection Test](https://github.com/Ahmed800363/Azure-Data-Factory-Customer-Project/blob/main/Image/photo%202.png)

### 2. Dataset Schema Mapping
For each source, I created a dataset where I defined the schema, file format (DelimitedText/CSV), and mapped the column types to ensure Data Factory interprets the data correctly.

![Dataset Configuration](https://github.com/Ahmed800363/Azure-Data-Factory-Customer-Project/blob/main/Image/photo%203.png)
![Source Schema Definition](https://github.com/Ahmed800363/Azure-Data-Factory-Customer-Project/blob/main/Image/photo%204.png)



## 📥 Phase 1: Data Ingestion & Orchestration
The data ingestion process is managed by an ADF Pipeline that connects to the source via HTTP Linked Services.

* **Activity:** I used the `Copy Data` activity to fetch raw CSV files from the source and move them to the landing zone in Azure Data Lake Storage.
* **Orchestration:** The pipeline is designed to trigger the Data Flow automatically upon a successful copy operation.

![Pipeline Overview](https://github.com/Ahmed800363/Azure-Data-Factory-Customer-Project/blob/main/Image/Pipeline%202%20.png)
![Successful Execution](https://github.com/Ahmed800363/Azure-Data-Factory-Customer-Project/blob/main/Image/pipeline%206.png)

> **Execution Proof:** The green checkmarks in the monitoring tab confirm that the pipeline and its underlying activities (Copy & Data Flow) executed successfully.


## ⚙️ Phase 2: Data Transformation Logic (Data Flow)
In this phase, I used **Mapping Data Flows** to build a visual and scalable data transformation logic.

### 1. Data Source Ingestion (Within Data Flow)
The process starts by connecting to the raw datasets (Customers and Cusomers_Details) stored in the Data Lake.

![Data Flow Source Setup_Get Source 1](https://github.com/Ahmed800363/Azure-Data-Factory-Customer-Project/blob/main/Image/source%201.png)
![Get Source 2](https://github.com/Ahmed800363/Azure-Data-Factory-Customer-Project/blob/main/Image/source%202.png)

### 2. Transformation & Cleaning Steps
Once the data is ingested into the flow, the following transformations are applied:

* **Handling Missing Values:** Using a `Derived Column` to replace 'NAS' or Null values with 'Unknown'.
* **Relational Join:** Performing a `Full Outer Join` on `Cst_ID`.
* **Data Formatting:** Normalizing text fields to Uppercase.

**The Transformation Logic (Snippet):**
```sql
iif(isNull(ColumnName) || ColumnName == 'NAS', 'Unknown', ColumnName)
