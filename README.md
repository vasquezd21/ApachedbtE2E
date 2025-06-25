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

# 1: Prepare Sample Data and Ingest with a Data Pipeline
  - In your Fabric workspace, create a new Lakehouse item if you don't have one already. Give it a name (e.g., NYCTaxiLakehouse). This will be our landing zone for raw and transformed data.
    
# 2. Create a Data Pipeline for Ingestion
   - Go back to your Fabric workspace main page.
   - Create a new Data Pipeline item. Give it a name (e.g., IngestNYCTaxiPipeline).
   - In the pipeline designer, add a Copy data activity.
   - Name the activity something descriptive (e.g. CopyGreenTaxiData).
     
# 3. Configure the Copy Data Activity - Source
   - Select the CopyGreenTaxiData activity
   - Go to the Source tab
   - For connection, choose your ELTDemoLakeouse
   - For Root Folder, choose Files
