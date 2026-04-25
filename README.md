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



### 2. Data Transformation & Standardization (Derived Columns)
In this stage, I used the **Expression Builder** within the Derived Column transformation to clean and standardize categorical data.

#### A. Standardizing Marital Status
The raw dataset used single-letter codes (`S`, `M`). I mapped these to full descriptive strings using the following logic:

**Transformation Logic:**
```sql
iif(upper(trim(Marital_Status)) == 'S', 'Single', 
    iif(upper(trim(Marital_Status)) == 'M', 'Married', 
        upper(trim(Marital_Status))
    )
)
```
![tran 1](https://github.com/Ahmed800363/Azure-Data-Factory-Customer-Project/blob/main/Image/tran%201.png)

#### B. Standardizing Gender
Similarly, the Gender column was normalized from codes (F, M) to full titles to ensure data consistency:

**Transformation Logic:**
```sql
iif(upper(trim(Gender)) == 'F', 'Female', 
    iif(upper(trim(Gender)) == 'M', 'Male', 
        upper(trim(Gender))
    )
)
```
![tran 2](https://github.com/Ahmed800363/Azure-Data-Factory-Customer-Project/blob/main/Image/tran%202.png)


## 🧹 Data Cleaning (Handling "NAS" Values)
Before merging the datasets, I had to clean the primary key column used for the join.

**Why this was done:**
The primary key column in the first table contained `"NAS"` values, while the corresponding column in the second table did not. If left uncleaned, the **Join** operation would fail to match the records properly. I replaced `"NAS"` to ensure a clean, error-free matching process between the two tables.

**Transformation Logic:**
```sql
iif(isNull(Cst_ID) || Cst_ID == 'NAS', '', Cst_ID)
```
![cleaning](https://github.com/Ahmed800363/Azure-Data-Factory-Customer-Project/blob/main/Image/Delete%20NAS.png)


### 🔗 Data Integration (Full Outer Join)
With the datasets cleaned and standardized, the next crucial step was merging them into a unified schema.

**The Process:**
I used a **Full Outer Join** transformation to combine the primary customer dataset with the secondary details table. The join condition was based on the `Cst_ID` column, which was previously cleaned to guarantee accurate matching.

**Why Full Outer Join?**
This ensures zero data loss. Unmatched records from both tables are retained, providing a complete, 360-degree view of all customers and their associated details, even if some attributes are missing.

![Full Outer Join Configuration](https://github.com/Ahmed800363/Azure-Data-Factory-Customer-Project/raw/main/Image/join%201.png)

### 🚀 Final Data Flow Architecture
This is the complete visual representation of the transformation pipeline. It shows the end-to-end journey of the data, from the two initial sources, through the cleaning and standardization steps, to the final Join and Sink.


**Key Features of this Flow:**
* **Scalability:** The flow is designed to handle large datasets efficiently within the Azure environment.
* **Visibility:** Each step is modular, making it easy to debug and maintain.
* **Success Status:** The final execution confirms that all business rules were applied correctly across the entire pipeline.

![End-to-End Data Flow](https://github.com/Ahmed800363/Azure-Data-Factory-Customer-Project/raw/main/Image/last%20Data%20flow.png)


## 📊 Pipeline Execution & Monitoring
To ensure the end-to-end automation of the project, I orchestrated the entire process through an **Azure Data Factory Pipeline**. 

**Execution Details:**
* **Orchestration:** The pipeline first triggers the data ingestion and then automatically starts the transformation logic.
* **Validation:** Monitoring the execution shows that all activities were completed successfully without errors.

![Pipeline Monitoring Success](https://github.com/Ahmed800363/Azure-Data-Factory-Customer-Project/blob/main/Image/pipeline%206.png)

> **Success Indicator:** The green status icons in the monitoring dashboard confirm that both the data movement and the complex transformations were executed as planned.


## 🏁 Final Result: Data Landing (Sink)
The transformation process is now complete, and the cleaned, unified dataset has been successfully loaded into the destination.

**Final Destination:**
The output is stored in **Azure Data Lake Storage Gen2** as a partitioned CSV file. This file is now optimized and ready for consumption by downstream applications like **Power BI** for visualization or **Azure Synapse Analytics** for further processing.

**Key Deliverables:**
- [x] Zero data loss during the Full Outer Join.
- [x] All "NAS" and Null values handled for the Join key.
- [x] Fully standardized columns (Gender & Marital Status).

![Final Output in Container](https://github.com/Ahmed800363/Azure-Data-Factory-Customer-Project/raw/main/Image/test%20data%20in%20container.png)

---
### 🚀 Conclusion
This project demonstrates a complete Cloud Data Engineering life cycle, from establishing raw HTTP connections to orchestrating complex transformations in a serverless environment.

### 📊 View Processed Data
You can explore the final cleaned and integrated dataset directly on GitHub:

👉 **[Download / View Customer Data CSV](https://github.com/Ahmed800363/Azure-Data-Factory-Customer-Project/blob/main/customer_data.csv)**




