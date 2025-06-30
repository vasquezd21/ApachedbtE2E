# Microsoft Fabric ELT Demo Series: Data Ingestion & dbt Transformation

This repository contains a series of interconnected demos showcasing how to build key components of an ELT (Extract, Load, Transform) pipeline within Microsoft Fabric. The series utilizes:

*   **Fabric Data Pipelines** for data ingestion.
*   **dbt (Data Build Tool)** for data transformation.

This demo series directly addresses the request to set up demos combining Apache Airflow job + dbt within Microsoft Fabric, by breaking down the end-to-end process into manageable and understandable steps.

## Demo Breakdown & Status

To make the process clear and easy to follow, the end-to-end ELT pipeline is broken down into smaller, focused demos:

1.  **Demo 1: Data Ingestion into Fabric Lakehouse** (`demo1.md`)
    *   **Goal:** Demonstrates using a Fabric Data Pipeline's Copy activity to load simple sample data (`sales.csv`) from a file upload into a managed Delta table within a Fabric Lakehouse (`raw_sales`).
    *   **Status:** **Complete**. The step-by-step guide is available in `demo1.md`.

2.  **Demo 2: dbt Transformation within Fabric Airflow Job Environment** (`demo2.md`)
    *   **Goal:** Shows how to set up a dbt project within the file storage of a Fabric Apache Airflow job, configure it to connect to the Lakehouse data from Demo 1, define a simple dbt model, and run the dbt transformation manually from the Airflow IDE terminal to validate the logic.
    *   **Status:** **Complete**. The step-by-step guide is available in `demo2.md`.



## How to View and Run the Demos

Each demo is documented in its respective markdown file (`demo1.md`, `demo2.md`). You can click on these files directly in this GitHub repository to read the step-by-step instructions.

To actually *run* these demos in your own Fabric environment, please refer to the "Prerequisites" section within each `.md` file, which lists the necessary Fabric items and software installations.
