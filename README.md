Goal: This tutorial demonstrates how to build a complete ELT pipeline in Microsoft Fabric. We will use a Data Pipeline to ingest raw data into a Fabric Lakehouse, then use a Fabric Apache Airflow job to orchestrate a dbt project to transform the data, making it ready for consumption.

# Requirements
Before getting started, you will need
1. A Microsoft Fabric enabled workspace: Ensure you have access to a Fabric workspace with the necessary permissions to create Data Factory
2. Basic understanind of: Microsoft Fabric, dbt, and Apache Airflow
3. (Optional but recommended for local dbt devleopment):
   - Python installed on your local machine
   - dbt Core installed (pip install dbt-core)
   - The dbt-fabric adapter installed (pip install dbt-fabric)

# Scenario:
We will work with the NYC Green Taxi Trip Records dataset to build a pipeline that ingests rwa trip data and transforms it into a structured format suitable for analysis, such as calculating trip durations or aggregating data by location

# 1:  Prepare Sample Data and Ingest into Lakehouse Table with a Data Pipeline
  - First, we need some sample data in our Lakehouse that dbt can read and transform.
  - Open a text editor and create a file named sales.csv with the following content. This data is simple but allows for basic transformations (like calculating total sale amount).
    ```
    sale_id,product,quantity,price_per_unit,sale_timestamp
    1,Laptop,1,1200.00,2023-10-15 10:00:00
    2,Mouse,2,25.00,2023-10-15 10:05:00
    3,Keyboard,1,75.00,2023-10-15 10:10:00
    4,Monitor,1,300.00,2023-10-16 11:00:00
    5,Laptop,1,1200.00,2023-10-16 11:05:00
    ```
   - Save this file
    
# 2. Create a Fabric Lakehouse:
   - In your Fabric workspace, create a new Lakehouse item if you don't have one already. Give it a name (e.g., SalesDemoLakehouse). This will be our landing zone.
     
# 3. Upload Data to Lakehouse Files:
   - Once the SalesDemoLakehouse is created, navigate to the Files section in the Lakehouse explorer view on the left.
   - Create a new folder (e.g., uploads).
   - Upload the sales.csv file you created into the uploads folder.

# 4. Create a Data Pipeline for Loading into Tables:
   - Go back to your Fabric workspace main page.
   - Create a new Data pipeline item. Give it a name (e.g., LoadSalesToTablePipeline).
   - In the pipeline designer, add a Copy data activity.
   - Name the activity something descriptive like CopyCsvToSalesTable.

# 5. Configure the Copy Data Activity - Source:
   - Select the CopyCsvToSalesTable activity.
   - Go to the Source tab.
   - For "Data store type", select Workspace.
   - For "Workspace data store", select Lakehouse.
   - Choose your SalesDemoLakehouse.
   - For "File path type", select File path in Lakehouse.
   - Browse and select the uploads/sales.csv file you uploaded earlier.
   - Under "File format settings", ensure Binary format is unchecked and the "File format" is set to Delimited text.
   - Review the "File format settings" for Delimited text. Make sure "First row as header" is checked and the delimiter (comma) is correct. You can click "Preview data" to verify it looks correct.
     
# 6. Configure the Copy Data Activity - Destination
   - Go to the Destination tab.
   - For "Data store type", select Workspace.
   - For "Workspace data store", select Lakehouse.
   - Choose your SalesDemoLakehouse.
   - For "Root folder", select Tables (this is where we will load the data as a managed table).
   - For "Table setting", select Auto create table.
   - For "Table name", enter raw_sales.
   - For "File format", select Delta.
   - For "Copy behavior", select None (since it's a small static file, no need for upsert/merge).

# 7. Run the Data Pipeline:
   - Click the Run tab at the top of the pipeline designer.
   - Click Run.
   - Monitor the pipeline run in the "Output" pane. It should succeed quickly with this small file.

# 8. Verify Data Ingestion:
   - Navigate back to your SalesDemoLakehouse.
   - n the Lakehouse explorer, expand the Tables section. You should see a new table named raw_sales.
   - Click on the raw_sales table to preview its data. Ensure the data from your CSV file has been loaded correctly into a Delta table.
