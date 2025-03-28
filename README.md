---

# Building a Lakehouse in Microsoft Fabric with Azure CLI

## Overview
This project demonstrates the creation of a fully functional lakehouse in Microsoft Fabric using Azure CLI and other tools. A lakehouse combines the flexibility of a data lake with the analytical capabilities of a data warehouse, leveraging the Delta Lake format for structured and unstructured data management. This project showcases key Azure Data Engineer skills, including data storage design, ingestion, transformation with Spark, and querying with SQL endpoints.

### Objectives
- **Design and Implement Data Storage**: Set up a lakehouse in Microsoft Fabric to manage structured and unstructured data efficiently.
- **Data Ingestion**: Ingest data from an Azure SQL Database and CSV files into the lakehouse.
- **Data Organization**: Use partitioning to optimize storage and query performance.
- **Data Transformation**: Transform data using Spark notebooks for scalable processing.
- **Data Querying**: Query the lakehouse using SQL endpoints for analytics.
- **Outcome**: A production-ready lakehouse demonstrating end-to-end data engineering capabilities.

### Skills Showcased
- Data storage architecture design
- Data ingestion and integration with Azure services
- Apache Spark transformations in Microsoft Fabric
- SQL querying for analytics
- Proficiency with Azure CLI and Fabric integration

---

## Prerequisites
Before starting, ensure you have the following:
- **Azure Subscription**: An active subscription with contributor access.
- **Azure CLI**: Installed locally (version 2.56.0 or later). [Install Guide](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli)
- **Microsoft Fabric Access**: A Fabric-enabled workspace (requires a Fabric license or trial).
- **Git**: Installed for version control. [Install Git](https://git-scm.com/downloads)
- **Text Editor/IDE**: VS Code or similar for editing scripts and notebooks.
- **Sample Data**: Access to an Azure SQL Database and a CSV file (e.g., sample sales data).

---

## Step-by-Step Guide

### Step 1: Set Up Your Azure Environment
**Why**: Configuring your Azure environment ensures you’re authenticated and targeting the correct subscription. This step establishes the foundation for all subsequent CLI commands.

1. **Open a Terminal**: Use your preferred terminal (e.g., PowerShell, Bash).
2. **Log in to Azure CLI**:
   ```bash
   az login
   ```
   - This opens a browser window to authenticate your Azure account.
   - *Why*: Authentication is required to interact with Azure resources securely.

3. **Set Your Subscription**:
   ```bash
   az account set --subscription "<subscription-id>"
   ```
   - Replace `<subscription-id>` with your Azure subscription ID (find it via `az account list`).
   - *Why*: Ensures commands target the correct subscription if you have multiple.

**Screenshot**: Capture the terminal output showing successful login and subscription set.  
*Suggested Caption*: "Azure CLI login and subscription selection."

---

### Step 2: Create a Resource Group
**Why**: A resource group organizes all Azure resources for this project, making management and cleanup easier. It’s a best practice for structuring cloud projects.

1. **Run the Command**:
   ```bash
   az group create --name "LakehouseRG" --location "eastus"
   ```
   - `LakehouseRG`: Resource group name (customizable).
   - `eastus`: Region (choose based on proximity or requirements).
   - *Why*: Centralizes resources in a single region for performance and compliance.

**Screenshot**: Show the terminal output confirming the resource group creation.  
*Suggested Caption*: "Resource group created for lakehouse project."

---

### Step 3: Provision Microsoft Fabric Capacity
**Why**: Fabric requires a capacity (compute resource) to run workloads like lakehouses. Provisioning this via CLI ensures scalability and automation potential.

1. **Create Fabric Capacity**:
   ```bash
   az resource create --resource-group "LakehouseRG" --name "LakehouseCapacity" --resource-type "Microsoft.Fabric/capacities" --location "eastus" --properties '{"sku": {"name": "F2", "tier": "Fabric"}}'
   ```
   - `F2`: Smallest Fabric SKU for testing (adjust based on needs).
   - *Why*: Defines the compute power available for Fabric operations like Spark and SQL processing.

**Screenshot**: Display the CLI output confirming capacity creation.  
*Suggested Caption*: "Fabric capacity provisioned in Azure."

**Note**: Fabric CLI support is limited; some steps may require the Fabric portal.

---

### Step 4: Create a Fabric Workspace
**Why**: A workspace in Fabric is a collaborative environment for data engineers to manage lakehouses, pipelines, and notebooks. It’s the entry point for this project.

1. **Log in to Fabric Portal**: Navigate to [fabric.microsoft.com](https://fabric.microsoft.com).
2. **Create Workspace**:
   - Click "Workspaces" > "New Workspace".
   - Name: `LakehouseWorkspace`.
   - Assign to `LakehouseCapacity`.
   - *Why*: Links the workspace to the provisioned capacity for compute resources.

3. **CLI Alternative (Preview)**: Check for Fabric CLI updates; currently, workspace creation is portal-based.

**Screenshot**: Show the Fabric portal with the new workspace listed.  
*Suggested Caption*: "LakehouseWorkspace created in Microsoft Fabric."

---

### Step 5: Create a Lakehouse
**Why**: The lakehouse is the core storage solution, unifying data lake and warehouse features. It uses Delta Lake for reliability and performance.

1. **In Fabric Workspace**:
   - Open `LakehouseWorkspace`.
   - Click "New" > "Lakehouse".
   - Name: `SalesLakehouse`.
   - *Why*: A dedicated lakehouse isolates this project’s data for clarity and scalability.

**Screenshot**: Capture the lakehouse creation confirmation in the portal.  
*Suggested Caption*: "SalesLakehouse created in Fabric workspace."

---

### Step 6: Ingest Data from Azure SQL Database
**Why**: Ingesting data from a relational source like Azure SQL Database demonstrates integration skills and prepares data for transformation.

1. **Set Up Azure SQL Database** (if not existing):
   ```bash
   az sql server create --name "sql-lakehouse-server" --resource-group "LakehouseRG" --location "eastus" --admin-user "<admin-user>" --admin-password "<admin-password>"
   az sql db create --resource-group "LakehouseRG" --server "sql-lakehouse-server" --name "SalesDB" --service-objective "S0"
   ```
   - *Why*: Creates a sample database for ingestion; adjust credentials securely.

2. **Create a Data Pipeline in Fabric**:
   - In `LakehouseWorkspace`, click "New" > "Data Pipeline".
   - Name: `SQLtoLakehouse`.
   - Use "Copy Data" activity.
   - Source: Azure SQL Database (configure connection with server and DB details).
   - Sink: `SalesLakehouse` (Files section).
   - *Why*: Pipelines automate ingestion, a critical data engineering task.

**Screenshot**: Show the pipeline configuration in Fabric.  
*Suggested Caption*: "Data pipeline configured to ingest from Azure SQL to lakehouse."

---

### Step 7: Ingest Data from a CSV File
**Why**: Handling unstructured data like CSV files showcases versatility in data ingestion, a common real-world scenario.

1. **Upload CSV to Azure Blob Storage**:
   ```bash
   az storage account create --name "lakehousestorage" --resource-group "LakehouseRG" --location "eastus" --sku "Standard_LRS"
   az storage container create --account-name "lakehousestorage" --name "csvdata"
   az storage blob upload --account-name "lakehousestorage" --container-name "csvdata" --name "sales.csv" --file "path/to/sales.csv"
   ```
   - *Why*: Blob storage is a scalable source for unstructured data.

2. **Ingest into Lakehouse**:
   - Create a new pipeline: `CSVtoLakehouse`.
   - Source: Azure Blob Storage (link to `csvdata/sales.csv`).
   - Sink: `SalesLakehouse` (Files section).
   - *Why*: Demonstrates multi-source ingestion capability.

**Screenshot**: Display the CSV pipeline setup in Fabric.  
*Suggested Caption*: "CSV ingestion pipeline from Blob Storage to lakehouse."

---

### Step 8: Organize Data with Partitioning
**Why**: Partitioning improves query performance by dividing data into manageable chunks, a key optimization technique.

1. **Partition Data**:
   - In `SalesLakehouse`, navigate to the Files section.
   - Manually create folders (e.g., `year=2023`, `year=2024`) and move ingested files accordingly.
   - *Why*: Partitioning by year aligns with common analytical patterns.

**Screenshot**: Show the partitioned folder structure in the lakehouse explorer.  
*Suggested Caption*: "Partitioned data structure in SalesLakehouse."

---

### Step 9: Transform Data with Spark Notebooks
**Why**: Spark notebooks enable scalable transformations, a cornerstone of big data processing in Fabric.

1. **Create a Notebook**:
   - In `LakehouseWorkspace`, click "New" > "Notebook".
   - Name: `TransformSalesData`.

2. **Sample Code**:
   ```python
   # Load data from lakehouse
   df = spark.read.format("delta").load("abfss://SalesLakehouse@onelake.dfs.fabric.microsoft.com/Files")
   # Transform: Aggregate sales by year
   df_transformed = df.groupBy("year").sum("sales")
   # Save as Delta table
   df_transformed.write.format("delta").mode("overwrite").save("abfss://SalesLakehouse@onelake.dfs.fabric.microsoft.com/Tables/SalesSummary")
   ```
   - *Why*: Aggregates data for reporting, showcasing Spark’s power.

**Screenshot**: Capture the notebook with code and output.  
*Suggested Caption*: "Spark notebook transforming sales data."

---

### Step 10: Query Data with SQL Endpoints
**Why**: SQL endpoints allow analysts to query lakehouse data easily, bridging engineering and analytics.

1. **Switch to SQL Mode**:
   - In `SalesLakehouse`, switch to "SQL analytics endpoint".
2. **Run Query**:
   ```sql
   SELECT year, SUM(sales) as total_sales
   FROM SalesSummary
   GROUP BY year;
   ```
   - *Why*: Validates transformation and demonstrates SQL integration.

**Screenshot**: Show the SQL query results in Fabric.  
*Suggested Caption*: "SQL query results from lakehouse endpoint."

---

## Repository Structure
```
LakehouseProject/
├── scripts/
│   ├── setup_resource_group.sh       # CLI script for resource group
│   ├── provision_fabric_capacity.sh  # CLI script for Fabric capacity
│   ├── sql_setup.sh                  # CLI script for SQL database
│   └── blob_upload.sh                # CLI script for CSV upload
├── notebooks/
│   └── TransformSalesData.ipynb      # Spark notebook for transformation
├── sql/
│   └── sales_query.sql               # SQL query for endpoint
├── screenshots/                      # Folder for all screenshots
└── README.md                         # This file
```

---

## Conclusion
This project demonstrates a complete lakehouse implementation in Microsoft Fabric, from setup to querying. It highlights my ability to:
- Design scalable data storage solutions.
- Integrate diverse data sources.
- Perform complex transformations with Spark.
- Enable analytics with SQL.

Feel free to explore the code, replicate the setup, or reach out for collaboration!

---

## Next Steps for You
1. **Create the GitHub Repo**:
   - Initialize a repo named `LakehouseProject` and push this README and scripts.
2. **Take Screenshots**:
   - Follow the placeholders and capture each step as you execute it.
3. **Enhance with Comments**:
   - Add detailed comments in scripts and notebooks for clarity.
4. **Test Everything**:
   - Ensure all steps work end-to-end before finalizing.

This README is now polished, detailed, and ready to impress potential employers!
