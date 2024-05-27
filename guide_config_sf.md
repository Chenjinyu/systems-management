Step-by-Step Guide to Configure Snowflake Connection in Airflow
Step 1: Access Airflow UI
Open your web browser and navigate to the Airflow UI, typically available at http://<your-airflow-server>:8080.
Step 2: Navigate to Connections
Click on the Admin tab in the top menu.
Select Connections from the dropdown menu.
Step 3: Add a New Connection
Click the "+" button to add a new connection.
Step 4: Configure the Snowflake Connection
Conn Id: Enter snowflake_conn (a unique identifier for the connection).

Conn Type: Select Snowflake from the dropdown menu.

Host: Enter your_snowflake_account.snowflakecomputing.com (your Snowflake account URL).

Login: Enter your_snowflake_user (your Snowflake username).

Password: Enter your_snowflake_password (your Snowflake password).

Schema: Enter your_default_schema (your default schema).

Extra: Enter the following JSON, replacing the placeholders with your Snowflake details:

```json
{
  "account": "your_snowflake_account",
  "warehouse": "your_snowflake_warehouse",
  "database": "your_snowflake_database",
  "role": "your_snowflake_role"
}
```
Step 5: Save the Connection
Click the Save button at the bottom of the form to save your connection settings.
Screenshots for Each Step
To assist you visually, here are some general screenshots from the Airflow UI to guide you through the process:

Admin Tab:

Admin Tab

Connections Menu:

Connections Menu

Add Connection:

Add Connection

Connection Configuration:

Connection Configuration

Save Connection:

Save Connection

Example Usage in DAG
Here's how you can use the configured Snowflake connection in an Airflow DAG:

python
Copy code
from airflow import DAG
from airflow.providers.snowflake.operators.snowflake import SnowflakeOperator
from airflow.operators.dummy import DummyOperator
from datetime import datetime

default_args = {
    'owner': 'airflow',
    'start_date': datetime(2023, 1, 1),
    'retries': 1,
}

with DAG('snowflake_example', default_args=default_args, schedule_interval='@daily', catchup=False) as dag:

    start = DummyOperator(task_id='start')

    snowflake_query = """
    SELECT CURRENT_DATE;
    """

    run_snowflake_query = SnowflakeOperator(
        task_id='run_snowflake_query',
        sql=snowflake_query,
        snowflake_conn_id='snowflake_conn'
    )

    end = DummyOperator(task_id='end')

    start >> run_snowflake_query >> end
This example demonstrates how to use the Snowflake connection in an Airflow DAG to execute a simple SQL query.

Summary
By following these steps and using the provided screenshots, you should be able to configure a Snowflake connection in Apache Airflow successfully. This connection can now be used in your DAGs with the SnowflakeOperator to execute queries on Snowflake.