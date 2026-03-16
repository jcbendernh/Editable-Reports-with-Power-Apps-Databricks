# Editable-Reports-with-Power-Apps-Databricks
This is an instructional repository of how to enable the ability to edit data in Azure Databricks Delta tables via an embedded Power App via a Power Automate Cloud Flow, all from within a Power BI report that uses Direct Query against a Databricks Serverless SQL compute engine.  

## Introduction
A few years ago I documented the step of how to do this with an Azure SQL database at [Editable-Reports-with-Power-Apps](https://github.com/jcbendernh/Editable-Reports-with-Power-Apps/edit/main/README.md).  Given the new functionality released within the Power Platform with reagrds to Azure Databricks Connectors, you can now do the same with an Azure Databricks environment.

For this example, I am using the golddb.Products delta table within an Azure Databricks Unity Catalog.  

## src folder
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

### Databricks
The Product data resides in a Delta table that is then served to both Power BI and the Power App and is updated via the Power Automate Cloud Flow.

### Power BI
- This reads the Product Delta table from Databricks via Direct Query
- A Power App is embedded in the report and we use the **PowerBIIntegration** function to filter the records within the Power App.

### Power App
- It is embedded within the Power BI Report via the web service / browser
- This reads the Product Delta table from Databricks via the Azure Databricks Connection Power Platform connector.
- Data is filtered to the selected row highlighted in the Poweer BI Report via the **PowerBIIntegration** function.
- The Product Delta table in Databricks is updated via a Power Automate Cloud Flow.

### Power Automate Cloud Flow
- Activated from the Power App
- Updates the Product Delta table in Databricks.


## Instructions

### Databricks

**IMPORTANT:** Please use/setup a serverless SQL Warehouse so that the startup time is in seconds and not minutes.  If you use a traditional SQL Warehouse, the Power Automate Cloud Flow will time out if the SQL warehouse is not started.  To create one, follow these instructions: [Create a SQL warehouse](https://learn.microsoft.com/en-us/azure/databricks/compute/sql-warehouse/create?source=recommendations)

1. Upload the [`products.csv`](./src/products.csv) to a volume within your Databricks Unity Catalog.  For instructions on how to do so, check out [Upload files to a Unity Catalog volume](https://learn.microsoft.com/en-us/azure/databricks/ingestion/file-upload/upload-to-volume).
2. Create a Python based notebook in your Databricks workspace and paste the following cells to read the products.csv and write the data to a new delta table in your Databricks catalog. <BR>
    ```python
    df = spark.read.option("header", "true").option("inferSchema", "true").csv("/Volumes/<catalog>/<schema>/products/products.csv")
    display(df)
    ```
    Replace <catalog> and <schema> with you catalog and schema.<br>

    ```python
    df.write.mode("overwrite").option("overwriteSchema", "true").saveAsTable("<catalog>.<schema>.products")
    ```
3. We need to make the ProductID field a non nullable primary key for the integration to work in this scenario.  To do so, paste the following cells into the notebook and run them.
    ```python
    %sql
    ALTER TABLE <catalog>.<schema>.products 
    ALTER COLUMN ProductID SET NOT NULL
    ```
    &nbsp;
    ```python
    %sql
    ALTER TABLE <catalog>.<schema>.products 
    ADD CONSTRAINT products_pk PRIMARY KEY (ProductID)
    ```

**NOTE:** Please remember your values for <catalog>.<schema>.products as we will use these later on for Power BI, Power Apps and Power Automate.

### Power Automate Setup

4. Import the [`DatabricksFlowsandApp`](src/DatabricksFlowsandApp_1_0_0_2.zip) solution file into your Power Platform environment via Solutions in Power Automate.  For instructions on how to do so, check out [Import a solution](https://learn.microsoft.com/en-us/power-automate/import-flow-solution).

5. During the import process, you will need to update the Databricks Connection Reference under **+ New Connection**.  On the Connect to Azure Databricks screen, set the following properties and click Create.

    |Field | Value |  
    |----------|----------|
    | Authentication Type: | OAuth Connection |
    | Server Hostname: | **Server hostname** value on **Connection details** tab of the Databricks SQL Warehouse.| 
    | HTTP Path: | **HTTP path** value on **Connection details** tab of the Databricks SQL Warehouse| 

6.  Once the solution is imported, we will need to change a few values on both the 
**ReadDatabricksGoldProducts** and 
**UpdateDatabricksGoldProducts** Cloud Flows.  Open each Cloud Flow within Power Automate and click **Edit** in the toolbar and change the following values in the Execute a SQL Statement item on the canvas.<BR>
    a. Body/warehouse_id - change the SQL Warehouse ID to reflect your enviroment.  This can be found on the **Overview** tab of the Databricks SQL Warehouse.| <BR>
    b. Body/statement - change the Databricks catalog and schema in the update statement to reflect your enviroment.<BR>
    c. Body/catalog - change the Databricks catalog value to reflect your enviroment.<BR>
    d. Body/schema - change the Databricks schema value to reflect your enviroment.<BR>

    ![Power Automate-Execute SQL](./img/PowerAutomate-ExecuteSQL.png)
    

### Power BI
7.  Download the [`Editable-Products-Databricks.pbix`](src/Editable-Products-Databricks.pbix) file to your local machine and open it with the Power BI Desktop.

8.  Once the report opens, click on **Transform Data** in the toolbar to open the Power Query Editor.  

9.  Within the Power Query editor, double click on the **Source** option under **Applied Steps** to modify our Databricks connection values. 

10.  Change the following values to match your environment.<BR>
    a. Server Hostname - **Server hostname** value on **Connection details** tab of the Databricks SQL Warehouse.<BR>
    b. HTTP Path - **HTTP path** value on **Connection details** tab of the Databricks SQL Warehouse.<BR>
    c. Default Catalog (Optional)<BR>
    d. Native query **schema** and **catalog** values in the select statement. <BR>

11. When finished click **OK** and you should see the Product Data in the data preview.  Next, click **Close & Apply** to save your changes in Power Query and return to the report.  When finished, the report should look like the following:
    ![PowerBIDesktop](./img/PowerBIDesktop.png)

