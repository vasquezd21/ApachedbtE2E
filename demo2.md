# Demo 2: Transform Lakehouse Data using dbt within Fabric

**Goal:** This tutorial demonstrates how to set up and use dbt within a Fabric Apache Airflow job to transform data already present in a Fabric Lakehouse. We will follow Microsoft's official approach using Service Principal authentication and Cosmos for orchestration.

**Estimated Time:** 30-40 minutes

## Prerequisites:

Before starting this tutorial, you need:

1. **Completed Demo 1:** You must have successfully completed Demo 1, resulting in a Fabric Lakehouse (e.g., `SalesDemoLakehouse`) containing a Delta table named `raw_sales`.
2. **A Microsoft Fabric enabled workspace:** Ensure you have access to a Fabric workspace with permissions to create Apache Airflow job items.
3. **Service Principal Setup:** You need to create a Service Principal and add it as a Contributor to your workspace. This is required for dbt authentication.
4. **Basic understanding of:** dbt basics and Apache Airflow concepts.

## Important: Service Principal Setup

Before proceeding, you must set up a Service Principal:

1. **Create Service Principal** in Azure Portal:
   - Go to Azure Portal → Azure Active Directory → App registrations
   - Create a new application registration
   - Note down the **Application (client) ID**, **Directory (tenant) ID**
   - Create a **Client Secret** and note it down

2. **Add Service Principal to Fabric Workspace**:
   - Go to your Fabric workspace → Settings → Manage access
   - Add the Service Principal as **Contributor**

## Scenario:

Building upon Demo 1, we'll transform the raw sales data using dbt. We'll create models that calculate total amounts and create summary tables, following Microsoft's official dbt-Fabric integration pattern.

## Steps:

### 1. Create a New Apache Airflow Job:
- Navigate to your Fabric workspace main page.
- Create a new **Apache Airflow job** item (e.g., `SalesDemoAirflowJob`).
- Wait for the Airflow environment to start up.

### 2. Set Up Requirements:
Create a file named `requirements.txt` in the `dags` folder:

```txt
astronomer-cosmos==1.0.3
dbt-fabric==1.5.0
```

### 3. Create dbt Project Structure:
Following Microsoft's official pattern, create this folder structure in your `dags` folder:

```
dags/
├── requirements.txt
├── sales/
│   ├── profiles.yml
│   ├── dbt_project.yml
│   └── models/
│       └── sales_summary.sql
└── sales_cosmos_dag.py
```

### 4. Create the dbt Profile Configuration:
**File: `dags/sales/profiles.yml`**

```yaml
sales:
  target: dev
  outputs:
    dev:
      type: fabricspark
      method: livy
      authentication: ServicePrincipal
      endpoint: https://api.fabric.microsoft.com/v1
      workspaceid: <Workspace ID>
      lakehouseid: <Lakhouse ID>
      lakehouse: <Lakehouse name>
      schema: <Schema name>
      spark_config:
        name: sales_demo
        # optional
        archives:
          - "example-archive.zip"
        conf:
            spark.executor.memory: "2g"
            spark.executor.cores: "2"
        tags:
          project: salesDemo
          user: v-davasquez@microsoft.com
          driverMemory: "2g"
          driverCores: 2
          executorMemory: "4g"
          executorCores: 4
          numExecutors: 3
      # optional
      connect_retries: 0
      connect_timeout: 10
      retry_all: true
```

**To find your Lakehouse SQL endpoint:**
- Go to your Lakehouse in Fabric
- Click on "SQL analytics endpoint"
- Copy the server name from the connection string

### 5. Create the dbt Project Configuration:
**File: `dags/sales/dbt_project.yml`**

```yaml
name: "sales"

config-version: 2
version: "0.1"

profile: "sales"

model-paths: ["models"]
seed-paths: ["seeds"]
test-paths: ["tests"]
analysis-paths: ["analysis"]
macro-paths: ["macros"]

target-path: "target"
clean-targets:
  - "target"
  - "dbt_modules"
  - "logs"

require-dbt-version: [">=1.0.0", "<2.0.0"]

models:
  sales:
    materialized: table
```


### 6. Create a Summary Model:
**File: `dags/sales/models/sales_summary.sql`**

```sql
-- Daily sales summary based on the fact table

SELECT
    sale_date,
    COUNT(*) as total_transactions,
    SUM(total_amount) as daily_revenue,
    AVG(total_amount) as avg_transaction_value,
    MIN(total_amount) as min_transaction_value,
    MAX(total_amount) as max_transaction_value,
    COUNT(DISTINCT product) as unique_products_sold
FROM {{ ref('fct_sales') }}
GROUP BY sale_date
ORDER BY sale_date
```

### 7. Create the Airflow DAG:
**File: `dags/sales_cosmos_dag.py`**

```python
import os
from pathlib import Path
from datetime import datetime
from airflow.utils.dates import days_ago
from cosmos import DbtDag, ProjectConfig, ProfileConfig
from airflow import DAG

DEFAULT_DBT_ROOT_PATH = Path(__file__).parent.parent / "dags" / "sales"
DBT_ROOT_PATH = Path(os.getenv("DBT_ROOT_PATH", DEFAULT_DBT_ROOT_PATH))

profile_config = ProfileConfig(
    profile_name="sales",
    target_name="fabric-dev",
    profiles_yml_filepath=DBT_ROOT_PATH / "profiles.yml",
)

with DAG(
    dag_id="dbt_fabric_dag",
    schedule_interval="@daily",
    start_date=days_ago(1),
    catchup=False,
) as dag:
    dbt_fabric_dag = DbtDag(
        project_config=ProjectConfig(DBT_ROOT_PATH),
        operator_args={"install_deps": True},
        profile_config=profile_config,
    )

```

### 8. Update Configuration Values:
Before running, you must update these placeholder values in `profiles.yml`:

- `<YOUR_LAKEHOUSE_SQL_ENDPOINT>`: Your Lakehouse SQL endpoint (e.g., `abc123-def456.sql.fabric.microsoft.com`)
- `<YOUR_LAKEHOUSE_NAME>`: Your Lakehouse name (e.g., `SalesDemoLakehouse`)
- `<YOUR_TENANT_ID>`: Your Azure tenant ID
- `<YOUR_CLIENT_ID>`: Your Service Principal application ID
- `<YOUR_CLIENT_SECRET>`: Your Service Principal client secret

### 9. Deploy and Run:
1. Save all files in your Airflow job interface
2. Wait for the environment to restart and install dependencies
3. Navigate to the Airflow UI
4. Find your DAG named `sales_dbt_dag`
5. Enable the DAG and trigger a manual run

### 10. Monitor and Verify:
1. Monitor the DAG execution in Airflow UI
2. Once successful, go back to your Fabric Lakehouse
3. Check the Tables section for new tables:
   - `sales_summary` (daily aggregated data)

