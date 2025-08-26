
<!--
help_text: ''
key: summary_duckdb_integration_7b4bc7b0-8923-4335-bd21-40fc281ce47b
modules:
- flytekitplugins.duckdb.task
questions_to_answer: []
type: summary

-->
The DuckDB Integration provides a powerful Flyte task for executing DuckDB queries directly within your workflows. This integration allows you to leverage DuckDB's analytical capabilities for in-memory or file-based data processing, seamlessly integrating with Flyte's data passing mechanisms and secret management.

### Capabilities

The DuckDB Integration offers the following key capabilities:

*   **SQL Execution**: Execute single or multiple DuckDB SQL queries.
*   **Dynamic Queries**: Define queries at task creation or provide them dynamically at runtime as task inputs.
*   **Data Ingestion**: Register Flyte `StructuredDataset` objects, Pandas DataFrames, or PyArrow Tables as temporary tables within the DuckDB session, making them directly queryable.
*   **Parameterized Queries**: Support for parameterized SQL queries, allowing for flexible and secure query construction.
*   **Flexible Connectivity**: Connect to local DuckDB instances (in-memory or file-based) or remote services like MotherDuck. The integration also supports custom connection functions for specialized DuckDB deployments.
*   **Secure Credentials**: Integrate with Flyte's secret management system to securely handle database connection tokens for remote providers.
*   **Structured Output**: All query results are returned as a `StructuredDataset`, enabling seamless consumption by downstream Flyte tasks.

### Connecting to DuckDB

The `DuckDBQuery` task manages connections to DuckDB instances. You specify the connection method using the `provider` argument during task initialization.

#### Standard Providers

The `DuckDBProvider` enum defines common connection types:

*   **`DuckDBProvider.LOCAL`**: Connects to a local DuckDB instance. This is the default and typically creates an in-memory database.
    ```python
    from flytekitplugins.duckdb.task import DuckDBQuery, DuckDBProvider
    import pandas as pd
    from flytekit import workflow, kwtypes

    # Define a task for local DuckDB operations
    local_duckdb_task = DuckDBQuery(
        name="my_local_query",
        query="SELECT * FROM my_data WHERE value > 10",
        inputs=kwtypes(my_data=pd.DataFrame),
        provider=DuckDBProvider.LOCAL,
    )

    @workflow
    def local_wf(input_df: pd.DataFrame) -> pd.DataFrame:
        return local_duckdb_task(my_data=input_df)
    ```

*   **`DuckDBProvider.MOTHERDUCK`**: Connects to MotherDuck, a cloud-based DuckDB service. This provider requires a connection token, which must be supplied via Flyte secrets.

    To use `MOTHERDUCK`, you must configure a Flyte secret and pass it to the `DuckDBQuery` task using the `secret_requests` argument. Only one secret can be specified for a `DuckDBQuery` task.

    ```python
    from flytekitplugins.duckdb.task import DuckDBQuery, DuckDBProvider, MissingSecretError
    from flytekit import Secret, workflow, kwtypes
    import pandas as pd

    # Assume 'duckdb-secrets' is your secret group and 'motherduck_token' is the key
    # This secret must be configured in your Flyte environment.
    motherduck_secret = Secret(group="duckdb-secrets", key="motherduck_token")

    try:
        motherduck_task = DuckDBQuery(
            name="my_motherduck_query",
            query="SELECT * FROM my_cloud_table",
            provider=DuckDBProvider.MOTHERDUCK,
            secret_requests=[motherduck_secret],
        )

        @workflow
        def motherduck_wf() -> pd.DataFrame:
            return motherduck_task()

    except MissingSecretError as e:
        print(f"Error initializing MotherDuck task: {e}")
        # Handle the error, e.g., by ensuring the secret is configured.
    ```

    If a non-local provider like `MOTHERDUCK` is chosen without a corresponding `secret_requests` argument, a `MissingSecretError` is raised during task initialization.

#### Custom Connection Functions

For more advanced or specific connection requirements, you can provide a custom callable as the `provider`. This callable should accept an optional connection token (if a secret is provided) and return a DuckDB connection object.

```python
import duckdb
from flytekitplugins.duckdb.task import DuckDBQuery
from flytekit import Secret, workflow, kwtypes
import pandas as pd

def custom_connect_s3(token: str = None):
    """
    Custom function to connect to DuckDB with S3 credentials.
    The 'token' here could be an AWS access key, or a more complex object.
    """
    con = duckdb.connect(":memory:")
    # Example: Configure S3 access using the token
    if token:
        con.execute(f"SET s3_access_key_id='{token.split(':')[0]}'")
        con.execute(f"SET s3_secret_access_key='{token.split(':')[1]}'")
        con.execute("INSTALL httpfs; LOAD httpfs;")
    return con

# Define a secret for S3 credentials (e.g., "access_key:secret_key")
s3_secret = Secret(group="s3-credentials", key="aws_token")

s3_duckdb_task = DuckDBQuery(
    name="my_s3_duckdb_query",
    query="SELECT * FROM 's3://my-bucket/data.parquet'",
    provider=custom_connect_s3,
    secret_requests=[s3_secret],
)

@workflow
def s3_wf() -> pd.DataFrame:
    return s3_duckdb_task()
```

### Defining and Executing DuckDB Queries

The `DuckDBQuery` task is initialized with a name, an optional query, and optional inputs.

#### Specifying Queries

You can provide the SQL query in two ways:

1.  **At Task Definition**: Pass a single query string or a list of query strings to the `query` argument during `DuckDBQuery` initialization.
    ```python
    from flytekitplugins.duckdb.task import DuckDBQuery
    from flytekit import workflow
    import pandas as pd

    # Single query
    single_query_task = DuckDBQuery(
        name="single_query_example",
        query="SELECT 1 + 1 AS result",
    )

    # Multiple queries executed sequentially
    multi_query_task = DuckDBQuery(
        name="multi_query_example",
        query=[
            "CREATE TABLE my_temp_table (id INTEGER, name VARCHAR)",
            "INSERT INTO my_temp_table VALUES (1, 'Alice'), (2, 'Bob')",
            "SELECT * FROM my_temp_table WHERE id = 1",
        ],
    )

    @workflow
    def query_definition_wf() -> (pd.DataFrame, pd.DataFrame):
        res1 = single_query_task()
        res2 = multi_query_task()
        return res1, res2
    ```

2.  **At Task Execution**: If `query=None` is provided during task definition, the query can be passed as an input argument named `query` when the task is called within a workflow. This allows for dynamic query generation.
    ```python
    from flytekitplugins.duckdb.task import DuckDBQuery
    from flytekit import workflow, kwtypes
    import pandas as pd

    dynamic_query_task = DuckDBQuery(
        name="dynamic_query_example",
        inputs=kwtypes(user_query=str), # Define 'user_query' as an input
    )

    @workflow
    def dynamic_query_wf(user_query: str) -> pd.DataFrame:
        # The 'user_query' input will be mapped to the task's internal 'query'
        return dynamic_query_task(user_query=user_query)

    # Example usage: dynamic_query_wf(user_query="SELECT 'Hello, DuckDB!' AS message")
    ```

#### Passing Input Data

The `DuckDBQuery` task can accept various data types as inputs, which are then registered as temporary tables or used as parameters within the DuckDB session.

*   **`StructuredDataset`**: Flyte `StructuredDataset` objects are automatically registered as tables in DuckDB. The table name will correspond to the input argument name.
    ```python
    from flytekitplugins.duckdb.task import DuckDBQuery
    from flytekit import workflow, kwtypes, StructuredDataset
    import pandas as pd

    data_processing_task = DuckDBQuery(
        name="process_structured_dataset",
        query="SELECT category, COUNT(*) AS count FROM input_data GROUP BY category",
        inputs=kwtypes(input_data=StructuredDataset), # 'input_data' will be available as a table
    )

    @workflow
    def process_data_wf(my_sd: StructuredDataset) -> pd.DataFrame:
        return data_processing_task(input_data=my_sd)
    ```

*   **Pandas DataFrames and PyArrow Tables**: These are also automatically registered as tables.
    ```python
    from flytekitplugins.duckdb.task import DuckDBQuery
    from flytekit import workflow, kwtypes
    import pandas as pd
    import pyarrow as pa

    df_processing_task = DuckDBQuery(
        name="process_dataframe",
        query="SELECT col1, col2 FROM my_dataframe WHERE col1 > 5",
        inputs=kwtypes(my_dataframe=pd.DataFrame), # 'my_dataframe' will be available as a table
    )

    arrow_processing_task = DuckDBQuery(
        name="process_arrow_table",
        query="SELECT * FROM my_arrow_table LIMIT 10",
        inputs=kwtypes(my_arrow_table=pa.Table), # 'my_arrow_table' will be available as a table
    )

    @workflow
    def process_df_arrow_wf(df: pd.DataFrame, arrow_table: pa.Table) -> (pd.DataFrame, pd.DataFrame):
        res1 = df_processing_task(my_dataframe=df)
        res2 = arrow_processing_task(my_arrow_table=arrow_table)
        return res1, res2
    ```

*   **Parameterized Queries**: For queries requiring parameters (e.g., `SELECT * FROM users WHERE id = ?`), you can pass a `list` of parameters or a JSON-encoded `str` representing a list.
    ```python
    from flytekitplugins.duckdb.task import DuckDBQuery
    from flytekit import workflow, kwtypes
    import pandas as pd
    import json

    param_query_task = DuckDBQuery(
        name="parameterized_query",
        query="SELECT * FROM my_table WHERE id = ?",
        inputs=kwtypes(params=list), # 'params' will be used for query parameters
    )

    multi_param_query_task = DuckDBQuery(
        name="multi_parameterized_query",
        query="INSERT INTO my_table VALUES (?, ?)",
        inputs=kwtypes(params=str), # JSON string for multiple sets of parameters
    )

    @workflow
    def param_wf(user_id: int, new_data: list) -> (pd.DataFrame, pd.DataFrame):
        # Single parameter
        res1 = param_query_task(params=[user_id])
        # Multiple sets of parameters for INSERT (e.g., [[1, 'A'], [2, 'B']])
        res2 = multi_param_query_task(params=json.dumps(new_data))
        return res1, res2
    ```

#### Returning Results

The `DuckDBQuery` task always returns its result as a `StructuredDataset`. This allows the output to be easily consumed by other Flyte tasks, stored, or converted to other formats like Pandas DataFrames.

```python
from flytekitplugins.duckdb.task import DuckDBQuery
from flytekit import workflow, StructuredDataset
import pandas as pd

# Task that returns a StructuredDataset
my_duckdb_task = DuckDBQuery(
    name="my_reporting_query",
    query="SELECT region, SUM(sales) AS total_sales FROM sales_data GROUP BY region",
    inputs={"sales_data": StructuredDataset},
)

@workflow
def reporting_wf(sales_data_sd: StructuredDataset) -> pd.DataFrame:
    # The task returns a StructuredDataset
    result_sd = my_duckdb_task(sales_data=sales_data_sd)
    # Convert the StructuredDataset to a Pandas DataFrame for further processing or return
    return result_sd.open(pd.DataFrame).all()
```

### Data Handling and Performance

The DuckDB Integration is designed to efficiently handle data within the workflow context.

*   **In-Memory Processing**: DuckDB is an in-process OLAP database, meaning it performs queries directly on data loaded into memory. When you register `StructuredDataset`s, Pandas DataFrames, or PyArrow Tables, their contents are made available to the DuckDB engine for querying.
*   **Output Materialization**: The `execute` method of the `DuckDBQuery` task fetches the entire result of the final query into memory as an Arrow table before wrapping it in a `StructuredDataset`. For extremely large result sets, consider optimizing your queries to return aggregated or paginated data to manage memory consumption. Future enhancements may include iterative download capabilities for very large outputs.

### Best Practices and Considerations

*   **Security**: Always use Flyte secrets for sensitive connection information (e.g., MotherDuck tokens, custom cloud credentials). Avoid hardcoding credentials in your code.
*   **Query Optimization**: While DuckDB is highly performant, standard SQL optimization practices still apply. Write efficient queries, use appropriate indexes (if applicable for your DuckDB setup), and filter data early.
*   **Error Handling**: Be prepared to handle `MissingSecretError` if you are using a non-local DuckDB provider without correctly configured secrets.
*   **Input Data Types**: Ensure your task inputs match the expected types (`StructuredDataset`, `pd.DataFrame`, `pa.Table`, `list`, `str` for parameters) to avoid `ValueError` during execution.
*   **Multi-Query Execution**: When providing a list of queries, the integration executes them sequentially. Only the result of the *last* query is returned. This pattern is useful for setting up temporary tables or performing intermediate DDL/DML operations before a final `SELECT` statement.
<!--
key: summary_duckdb_integration_7b4bc7b0-8923-4335-bd21-40fc281ce47b
type: summary_end

-->
<!--
code_unit: flytekitplugins.duckdb.examples.duckdb_query_task
code_unit_type: class
help_text: ''
key: example_d514810d-874d-4011-a8b4-8006cfca61dd
type: example

-->