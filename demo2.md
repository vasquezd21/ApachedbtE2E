# Demo 2: Transform Lakehouse Data using dbt within Fabric

Goal: This tutorial demonstrates how to set up and use dbt within a Fabric Apache Airflow job's environment to transform data already present in a Fabric Lakehouse. We will define simple transformation logic using dbt models and run them manually from the Airflow IDE.

# Prerequisites:
Before starting this tutorial, you need:
1. Completed Demo 1: You must have successfully completed Demo 1, resulting in a Fabric Lakehouse (e.g., SalesDemoLakehouse) containing a Delta table named raw_sales.
2. A Microsoft Fabric enabled workspace: Ensure you have access to a Fabric workspace with permissions to create Apache Airflow job items.
3. Basic understanding of: dbt basics (models, profiles.yml).
4. (Optional but Recommended for local dbt development/testing):
  - Python installed on your local machine.
  - dbt Core installed (pip install dbt-core).
  - The dbt-fabric adapter installed (pip install dbt-fabric).

# Scenario:
Building upon Demo 1, where we ingested raw sales data into our Lakehouse, we now want to transform this data. We will use dbt to create a derived model that calculates the total amount for each sale (quantity * price_per_unit) and selects relevant columns, preparing the data for analysis. We will perform these transformations using dbt running within the managed environment of a Fabric Apache Airflow job.
