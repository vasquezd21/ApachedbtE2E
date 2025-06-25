# Demo 2: Transform Lakehouse Data using dbt within Fabric

**Goal:** This tutorial demonstrates how to set up and use dbt within a Fabric Apache Airflow job's environment to transform data already present in a Fabric Lakehouse. We will define simple transformation logic using dbt models and run them manually from the Airflow IDE.

**Estimated Time:** 30-40 minutes

## Prerequisites:

Before starting this tutorial, you need:

1.  **Completed Demo 1:** You must have successfully completed Demo 1, resulting in a Fabric Lakehouse (e.g., `SalesDemoLakehouse`) containing a Delta table named `raw_sales`.
2.  **A Microsoft Fabric enabled workspace:** Ensure you have access to a Fabric workspace with permissions to create Apache Airflow job items.
3.  **Basic understanding of:** dbt basics (models, `profiles.yml`).
4.  **(Optional but Recommended for local dbt development/testing):**
    *   Python installed on your local machine.
    *   dbt Core installed (`pip install dbt-core`).
    *   The `dbt-fabric` adapter installed (`pip install dbt-fabric`).

## Scenario:

Building upon Demo 1, where we ingested raw sales data into our Lakehouse, we now want to transform this data. We will use dbt to create a derived model that calculates the total amount for each sale (`quantity * price_per_unit`) and selects relevant columns, preparing the data for analysis. We will perform these transformations using dbt running within the managed environment of a Fabric Apache Airflow job.

## Steps:

1.  **Create a New Apache Airflow Job:**
    *   Navigate back to your Fabric workspace main page.
    *   Create a new **Apache Airflow job** item. Give it a descriptive name (e.g., `SalesDemoAirflowJob`). This item provides the computing environment and file storage where your dbt project will live and run.
    *   Once the Airflow job item is created, click on it to open its environment page.
    *   Wait for the Airflow environment to start up. This may take a few minutes. The status will change from "Starting" to "Running".

2.  **Access the Airflow Cloud IDE:**
    *   On the Apache Airflow job environment page (once the status is "Running"), click the **Open Airflow IDE** button. This will open a browser-based code editor connected to the file storage of your Airflow job.

3.  **Initialize a dbt Project:**
    *   In the Airflow IDE, open the terminal (usually found under a `Terminal` menu option, select `New Terminal`).
    *   Navigate into the `dags` folder. While not strictly required, placing your dbt project within or alongside your `dags` folder in the Airflow job's storage is a common practice.

    ```bash
    cd dags
    ```

    *   Run the dbt initialization command. We'll name our project `sales_dbt_transforms`.

    ```bash
    dbt init sales_dbt_transforms
    ```

    *   Follow the prompts:
        *   Enter a name for your project: `sales_dbt_transforms` (or press Enter to accept the default).
        *   Choose a database: Select `fabric` from the list.
    *   dbt will create a new folder named `sales_dbt_transforms` with the standard dbt project structure (`models`, `tests`, `macros`, `dbt_project.yml`, etc.).

4.  **Configure the dbt Profile (`profiles.yml`):**
    *   dbt needs to know how to connect to your Fabric Lakehouse's SQL analytics endpoint to read the `raw_sales` data and write the transformed data. This connection information is stored in a `profiles.yml` file.
    *   In the Airflow IDE, navigate to the root of the `sales_dbt_transforms` folder you just created (`dags/sales_dbt_transforms`).
    *   Create a new file here named `profiles.yml`.
    *   Add the following content to the `profiles.yml` file. **You MUST replace the placeholder values** with your specific Fabric details from Demo 1:

    ```yaml
    sales_dbt_transforms: # This profile name must exactly match the 'profile:' name in dbt_project.yml
      target: dev
      outputs:
        dev:
          type: fabric
          host: <Your_Fabric_Workspace_Id>.sql.fabric.microsoft.com # <-- REPLACE with your Fabric workspace SQL Endpoint host (e.g., ab12345-cd67-ef89-gh01-ij2345kl6789.sql.fabric.microsoft.com)
          database: <Your_Lakehouse_Name> # <-- REPLACE with the name of your Lakehouse from Demo 1 (e.g., SalesDemoLakehouse)
          schema: dbt_sales # You can use a specific schema for dbt output, or 'dbo'
          # Authentication Options (Choose ONE - User Principal is simplest for demo):
          # For this demo, User Principal often works if your user has permissions.
          # For production, Service Principal is recommended for security.

          # Option 1 (Recommended for Demo): User Principal
          authentication: user_principal
          # If your organizational policy requires Multi-Factor Authentication for this user:
          # mfa_enabled: true

          # Option 2 (Recommended for Production/Automation): Service Principal
          # authentication: service_principal
          # client_id: {{ env_var('FABRIC_CLIENT_ID') }} # <-- Requires setting this environment variable on the Airflow Job in Fabric settings
          # client_secret: {{ env_var('FABRIC_CLIENT_SECRET') }} # <-- Requires setting this environment variable on the Airflow Job in Fabric settings
          # tenant_id: {{ env_var('FABRIC_TENANT_ID') }} # <-- Requires setting this environment variable on the Airflow Job in Fabric settings
    ```
    *   **Finding Connection Details:** Go back to your `SalesDemoLakehouse` in the Fabric portal. Click the **SQL analytics endpoint** button. Copy the hostname part of the connection string displayed. Use the exact name of your Lakehouse.
    *   Save the `profiles.yml` file in the Airflow IDE.

5.  **Configure the dbt Project File (`dbt_project.yml`):**
    *   In the Airflow IDE, open the `dbt_project.yml` file located within your `sales_dbt_transforms` folder (`dags/sales_dbt_transforms/dbt_project.yml`).
    *   Ensure the `name:` field at the top is set to `sales_dbt_transforms` (matching your folder and profile name).
    *   Verify that the `profile:` field below it also exactly matches the profile name you used in `profiles.yml` (`sales_dbt_transforms`).
    *   Ensure `model-paths` and `test-paths` are correctly pointing to the default `models` and `tests` folders within your project.
    *   Save the `dbt_project.yml` file.

6.  **Test dbt Connection (Crucial):**
    *   In the Airflow IDE terminal, navigate to the root of your dbt project folder:

    ```bash
    cd dags/sales_dbt_transforms
    ```

    *   Run the dbt debug command:

    ```bash
    dbt debug
    ```

    *   This command is vital. It checks your dbt configuration and attempts to connect to your Fabric Lakehouse using the details in `profiles.yml`.
    *   Look for `Connection test: OK` in the output. If you see this, your dbt project can successfully connect to your Lakehouse.
    *   If the connection test fails, carefully review your `profiles.yml` file for typos in the `host`, `database`, or authentication section. Ensure the Lakehouse name and SQL endpoint host are exactly correct.

7.  **Develop a dbt Model:**
    *   Now, let's create a simple dbt model to transform the data.
    *   In the Airflow IDE, navigate to the `models` folder within your dbt project (`dags/sales_dbt_transforms/models`).
    *   Delete the example `my_first_model.sql` and `my_second_model.sql` files.
    *   Create a new file named `fct_sales.sql`. We'll call this a 'fact' table model.
    *   Add the following SQL code to `fct_sales.sql`:

    ```sql
    -- This model transforms the raw sales data
    SELECT
        sale_id,
        product,
        quantity,
        price_per_unit,
        sale_timestamp,
        -- Calculate the total amount for each sale item
        quantity * price_per_unit AS total_amount
    FROM
        -- Reference the raw_sales table from your Lakehouse
        {{ source('SalesDemoLakehouse', 'raw_sales') }}
    ```
    *   Save the `fct_sales.sql` file.

8.  **Define the Source Table:**
    *   To use `{{ source('SalesDemoLakehouse', 'raw_sales') }}` in your dbt models, you need to tell dbt about your source tables in the Lakehouse using a YAML file.
    *   In the Airflow IDE, navigate to the `models` folder within your dbt project (`dags/sales_dbt_transforms/models`).
    *   Create a new file named `sources.yml`.
    *   Add the following YAML code to `sources.yml`. **Make sure the `database` and `schema` match** where your `raw_sales` table is located in your Lakehouse (it should be in the `dbo` schema within your Lakehouse's SQL endpoint by default, or the schema specified in your `profiles.yml` if you changed it).

    ```yaml
    version: 2

    sources:
      - name: SalesDemoLakehouse # This name is arbitrary, but good practice to match Lakehouse name
        database: <Your_Lakehouse_Name> # <-- REPLACE with your Lakehouse name (e.g., SalesDemoLakehouse)
        schema: dbo # <-- Verify this matches the schema where 'raw_sales' landed (likely 'dbo') or the schema in profiles.yml

        tables:
          - name: raw_sales # This name MUST match the table name in your Lakehouse
            identifier: raw_sales # Use 'identifier' if the actual table name is different from the 'name' you want to use in models (usually they are the same)
    ```
    *   Save the `sources.yml` file.

9.  **Run the dbt Model:**
    *   Now we can run the dbt model to create the transformed table in the Lakehouse.
    *   In the Airflow IDE terminal, ensure you are in the root of your dbt project folder (`dags/sales_dbt_transforms`).
    *   Run the dbt run command:

    ```bash
    dbt run
    ```

    *   dbt will compile your `fct_sales.sql` model, identify the source (`raw_sales`), and execute the compiled SQL against your Fabric Lakehouse.
    *   Look at the output in the terminal. You should see dbt connecting to the database and running the `fct_sales` model. It should report "SUCCESS" when finished.

10. **Verify Transformed Data in the Lakehouse:**
    *   Navigate back to your Fabric workspace portal.
    *   Go to your `SalesDemoLakehouse`.
    *   In the Lakehouse explorer, expand the **Tables** section.
    *   You should now see a **new table** created by dbt. The name will be `fct_sales` (derived from your model file name).
    *   Click on the `fct_sales` table to preview its data. Verify that the new `total_amount` column has been added and calculated correctly for each row.

## Summary of Achievements:

You have successfully:

*   Set up a Fabric Apache Airflow job (as an environment for dbt).
*   Configured a dbt project to connect to your Fabric Lakehouse.
*   Defined a data transformation using a dbt model (`fct_sales`).
*   Defined your source table (`raw_sales`) for dbt to reference.
*   Run the dbt transformation manually using the dbt CLI within the Airflow IDE.
*   Verified the transformed data (`fct_sales`) in the Lakehouse.

---
