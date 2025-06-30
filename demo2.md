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
├── sales_dbt_project/
│   ├── profiles.yml
│   ├── dbt_project.yml
│   └── models/
│       └── sales_summary.sql
└── sales_cosmos_dag.py
```

### 4. Create the dbt Profile Configuration:
**File: `dags/sales_dbt_project/profiles.yml`**

```yaml
config:
  partial_parse: true

sales_dbt_project:
  target: fabric-dev
  outputs:
    fabric-dev:
      type: fabric
      driver: "ODBC Driver 18 for SQL Server"
      server: <YOUR_LAKEHOUSE_SQL_ENDPOINT>  # Replace with your Lakehouse SQL endpoint
      port: 1433
      database: "<YOUR_LAKEHOUSE_NAME>"      # Replace with your Lakehouse name (e.g., SalesDemoLakehouse)
      schema: dbo
      threads: 4
      authentication: ServicePrincipal
      tenant_id: <YOUR_TENANT_ID>            # Replace with your Azure tenant ID
      client_id: <YOUR_CLIENT_ID>            # Replace with your Service Principal client ID
      client_secret: <YOUR_CLIENT_SECRET>    # Replace with your Service Principal client secret
```

**To find your Lakehouse SQL endpoint:**
- Go to your Lakehouse in Fabric
- Click on "SQL analytics endpoint"
- Copy the server name from the connection string

### 5. Create the dbt Project Configuration:
**File: `dags/sales_dbt_project/dbt_project.yml`**

```yaml
name: "sales_dbt_project"
config-version: 2
version: "0.1"
profile: "sales_dbt_project"

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
  sales_dbt_project:
    materialized: table
```


### 6. Create a Summary Model:
**File: `dags/sales_dbt_project/models/sales_summary.sql`**

```sql
-- Daily sales summary based on the fact table
with daily_sales as (
    select * from {{ ref('fct_sales') }}
),

final as (
    SELECT
        sale_date,
        COUNT(*) as total_transactions,
        SUM(total_amount) as daily_revenue,
        AVG(total_amount) as avg_transaction_value,
        COUNT(DISTINCT product) as unique_products_sold
    FROM daily_sales
    GROUP BY sale_date
    ORDER BY sale_date
)

select * from final
```

### 8. Create the Airflow DAG:
**File: `dags/sales_cosmos_dag.py`**

```python
import os
from pathlib import Path
from datetime import datetime

from cosmos import DbtDag, ProjectConfig, ProfileConfig

# Define the path to your dbt project
DEFAULT_DBT_ROOT_PATH = Path(__file__).parent / "sales_dbt_project"
DBT_ROOT_PATH = Path(os.getenv("DBT_ROOT_PATH", DEFAULT_DBT_ROOT_PATH))

# Profile configuration
profile_config = ProfileConfig(
    profile_name="sales_dbt_project",
    target_name="fabric-dev",
    profiles_yml_filepath=DBT_ROOT_PATH / "profiles.yml",
)

# Create the dbt DAG using Cosmos
sales_dbt_dag = DbtDag(
    project_config=ProjectConfig(DBT_ROOT_PATH),
    operator_args={"install_deps": True},
    profile_config=profile_config,
    schedule_interval="@daily",
    start_date=datetime(2024, 1, 1),
    catchup=False,
    dag_id="sales_dbt_dag",
    default_args={
        'owner': 'data-team',
        'depends_on_past': False,
        'email_on_failure': False,
        'email_on_retry': False,
        'retries': 1,
    },
)
```

### 9. Update Configuration Values:
Before running, you must update these placeholder values in `profiles.yml`:

- `<YOUR_LAKEHOUSE_SQL_ENDPOINT>`: Your Lakehouse SQL endpoint (e.g., `abc123-def456.sql.fabric.microsoft.com`)
- `<YOUR_LAKEHOUSE_NAME>`: Your Lakehouse name (e.g., `SalesDemoLakehouse`)
- `<YOUR_TENANT_ID>`: Your Azure tenant ID
- `<YOUR_CLIENT_ID>`: Your Service Principal application ID
- `<YOUR_CLIENT_SECRET>`: Your Service Principal client secret

### 10. Deploy and Run:
1. Save all files in your Airflow job interface
2. Wait for the environment to restart and install dependencies
3. Navigate to the Airflow UI
4. Find your DAG named `sales_dbt_dag`
5. Enable the DAG and trigger a manual run

### 11. Monitor and Verify:
1. Monitor the DAG execution in Airflow UI
2. Once successful, go back to your Fabric Lakehouse
3. Check the Tables section for new tables:
   - `sales_summary` (daily aggregated data)

