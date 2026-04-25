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
