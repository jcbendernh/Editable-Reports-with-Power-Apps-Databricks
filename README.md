# Editable-Reports-with-Power-Apps-Databricks
This is an instructional repository of how to enable the ability to edit data in Azure Databricks Delta tables via an embedded Power App via a Power Automate Cloud Flow, all from within a Power BI report that uses Direct Query against a Databricks Serverless SQL compute engine.  

## Introduction
A few years ago I documented the step of how to do this with an Azure SQL database at [Editable-Reports-with-Power-Apps](https://github.com/jcbendernh/Editable-Reports-with-Power-Apps/edit/main/README.md).  Given the new functionality released within the Power Platform with reagrds to Azure Databricks Connectors, you can now do the same with an Azure Databricks environment.

For this example, I am using the golddb.Products delta table within an Azure Databricks Unity Catalog.  

## src Folder Contents.
- **products.csv** 
    - This contains the data we will use to upload to an Azure Databricks Volume within Unity Catalog and then create a delta table from that volume.
- **Databricks Flow and Apps Power Automate Solution file**
    - **Products - Databricks Canvas App** - This is used for exploratory purposes to understand how the Cloud Flows operate using the [Execute SQL Commands](https://learn.microsoft.com/en-us/connectors/databricks/#execute-a-sql-statement). This is not used in the report and I left it in the solution in case you want to explore this functionality further.
    - **ReadDatabricksGoldProducts - Cloud Flow** - This utilizes the Execute SQL Command to read data from an Azure Databricks Delta table via a serverless SQL Warehouse compute.This is not used in the report. This is not used in the report and I left it in the solution in case you want to explore this functionality further.
    - **UpdateDatabricksGoldProducts - Cloud Flow** - This utilizes the Execute SQL Command to update data to back into the Azure Databricks Delta table from the values captured on the form.<BR>**This is utilized in the report**
    - Databricks Connection
- **Editable-Products-Databricks.pbix**
    - This is the Power BI Report we will publish to the service and add the Power App to.

## Architectural Overview
![Architecture diagram](img/architecture.png)









