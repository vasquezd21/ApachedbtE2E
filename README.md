# Demo 1: Ingest Simple Data into Fabric Lakehouse (using Data Pipeline)

Goal: This tutorial demonstrates how to use a Fabric Data Pipeline to ingest sample data from a flat file into a managed Delta table within a Fabric Lakehouse. This is a common first step in building data pipelines.

# Prerequisites:
Before getting started, you will need
1. A Microsoft Fabric enabled workspace: Ensure you have access to a Fabric workspace with the necessary permissions to create Lakehouse and Data pipeline items.
2. Basic understanding of: Microsoft Fabric concepts, specifically Lakehouse and Data pipelines.

# Scenario:
We have a small set of sample sales data in a CSV file. We need to load this data into our Fabric Lakehouse as a structured table so it can be easily queried and used for subsequent transformations.


# 1:  Prepare Sample Data 
  - Open a text editor and create a file named sales.csv with the following content. This data is simple but represents typical raw data.
    ```
    sale_id,product,quantity,price_per_unit,sale_timestamp
    1,Laptop,1,1200.00,2023-10-15 10:00:00
    2,Mouse,2,25.00,2023-10-15 10:05:00
    3,Keyboard,1,75.00,2023-10-15 10:10:00
    4,Monitor,1,300.00,2023-10-16 11:00:00
    5,Laptop,1,1200.00,2023-10-16 11:05:00
    ```
   - Save this file on your local machine
    
# 2. Create a Fabric Lakehouse:
   - In your Fabric workspace, create a new Lakehouse item if you don't have one already.
   - Give your Lakehouse a descriptive name (e.g., SalesDemoLakehouse). This will serve as the storage layer for our raw and transformed data.
     
# 3. Upload Data to Lakehouse Files:
   - Once the SalesDemoLakehouse is created, navigate to it.
   - In the Lakehouse explorer view on the left, select the Files section.
   - Create a new folder within Files (e.g., uploads). This keeps raw uploaded files separate from managed tables.
   - Upload the sales.csv file you created from your local machine into the uploads folder within your Lakehouse's Files.

# 4. Create a Data Pipeline for Loading into Tables:
   - Go back to your Fabric workspace main page.
   - Create a new Data pipeline item. Give it a name (e.g., LoadSalesToTablePipeline).
   - In the pipeline designer, add a Copy data activity by dragging it from the Activities pane or clicking "Add pipeline activity".
   - Name the activity something descriptive like CopyCsvToSalesTable.

# 5. Configure the Copy Data Activity - Source:
   - Select the CopyCsvToSalesTable activity.
   - Go to the Source tab in the properties pane below.
   - For "Data store type", select Workspace.
   - For "Workspace data store", select Lakehouse.
   - Choose your SalesDemoLakehouse.
   - For "File path type", select File path in Lakehouse.
   - Browse and select the uploads/sales.csv file you uploaded in Step 3.
   - Under "File format settings", ensure Binary format is unchecked and the "File format" is set to Delimited text.
   - Review the "File format settings" for Delimited text. Make sure "First row as header" is checked and the delimiter (comma) is correct. You can click "Preview data" to verify it looks correct.
     
# 6. Configure the Copy Data Activity - Destination
   - Go to the Destination tab.
   - For "Data store type", select Workspace.
   - For "Workspace data store", select Lakehouse.
   - Choose your SalesDemoLakehouse.
   - For "Root folder", select Tables (this is where we will load the data as a managed table).
   - For "Table setting", select Auto create table.
   - For "Table name", enter raw_sales. This will be the name of the Delta table created in your Lakehouse.
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
