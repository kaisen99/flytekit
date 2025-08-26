
<!--
help_text: ''
key: summary_duckdb_plugin_c9559399-52ed-43d5-b3de-f9c5a5fd0a5c
modules:
- flytekitplugins.duckdb.task
questions_to_answer: []
type: summary

-->
The DuckDB Plugin enables executing DuckDB queries as tasks within workflows. It integrates DuckDB's in-process OLAP capabilities directly, allowing for efficient data transformation and analysis.

### `DuckDBQuery` Task

The primary component for defining and executing DuckDB queries is the `DuckDBQuery` task.

**Initialization**

When initializing a `DuckDBQuery` task, configure its behavior using the following arguments:

*   `name`: A unique identifier for the task.
*   `query`: The SQL query string or a list of query strings to execute. If `None`, the query must be provided at runtime during task execution.
*   `inputs`: An optional dictionary mapping input names to their types (e.g., `kwtypes(my_df=pd.DataFrame)`). These inputs are registered as tables or used as parameters within the DuckDB environment.
*   `provider`: Specifies how to connect to DuckDB. This can be a member of the `DuckDBProvider` enum (e.g., `DuckDBProvider.LOCAL`, `DuckDBProvider.MOTHERDUCK`) or a custom callable function that returns a DuckDB connection object. Defaults to `DuckDBProvider.LOCAL`.
*   `secret_requests`: An optional list containing a single `Secret` object. This secret provides authentication tokens or credentials for secure connections, such as to MotherDuck.

**Connection Management**

The task handles connections to DuckDB based on the specified `provider`:

*   `DuckDBProvider.LOCAL`: Connects to a local in-memory or file-based DuckDB instance. This is the default and requires no additional configuration.
*   `DuckDBProvider.MOTHERDUCK`: Connects to MotherDuck. This provider requires a `secret_requests` to be supplied during task initialization, containing the `motherduck_token`.
*   Custom Callable: A user-defined function can be provided as the `provider`. This function should accept an optional `token` argument (a string) if a secret is provided via `secret_requests`, and it must return a DuckDB connection object.

If a non-local `DuckDBProvider` is used without a corresponding `secret_requests`, a `MissingSecretError` is raised.

**Input Handling**

Inputs provided to the `execute` method of the `DuckDBQuery` task are processed and made available within the DuckDB environment:

*   `StructuredDataset`: Registered as a temporary table in DuckDB.
*   `pandas.DataFrame` or `pyarrow.Table`: Registered as a temporary table.
*   `list`: Used as query parameters.
*   `str` (JSON string): Parsed as a JSON list and used as query parameters.
*   The `query` input, if provided at runtime, takes precedence and overrides any `query` specified during task initialization.

**Query Execution**

Queries can contain parameters using `$` or `?` placeholders. When multiple queries are provided as a list to the `query` argument, they are executed sequentially within the same DuckDB connection. Only the result of the last query is returned.

For parameterized queries, inputs provided as a `list` or JSON `str` are used. If multiple queries are parameterized, the `params` list should be a list of lists, where each inner list corresponds to parameters for a specific query.

**Output**

The `DuckDBQuery` task always returns a `StructuredDataset` containing the result of the final executed query. This ensures seamless integration with downstream tasks that expect structured data.

### `DuckDBProvider` Enum

The `DuckDBProvider` enum defines standard DuckDB connection methods:

*   `LOCAL`: Connects to a local DuckDB instance.
*   `MOTHERDUCK`: Connects to MotherDuck. This option requires a secret token for authentication.

Custom connection functions can be passed directly to the `provider` argument of `DuckDBQuery` for more specialized connection scenarios.

### Error Handling

`MissingSecretError`: This error is raised when a `DuckDBProvider` that requires a secret (e.g., `MOTHERDUCK`) is used without specifying a `secret_requests` in the `DuckDBQuery` constructor.

### Usage Examples

**Basic Local Query**

```python
from flytekitplugins.duckdb.task import DuckDBQuery
from flytekit import workflow
import pandas as pd

duckdb_task = DuckDBQuery(
    name="my_first_duckdb_query",
    query="SELECT 1 as col1, 'hello' as col2"
)

@workflow
def my_wf() -> pd.DataFrame:
    return duckdb_task()

# To execute:
# result_df = my_wf()
# print(result_df.open(pd.DataFrame).all())
```

**Querying with DataFrame Input**

```python
from flytekitplugins.duckdb.task import DuckDBQuery
from flytekit import kwtypes, workflow
import pandas as pd

duckdb_transform_task = DuckDBQuery(
    name="transform_data",
    inputs=kwtypes(input_df=pd.DataFrame),
    query="SELECT col1, col2 * 2 as col2_doubled FROM input_df WHERE col1 > 5"
)

@workflow
def data_pipeline(my_data: pd.DataFrame) -> pd.DataFrame:
    return duckdb_transform_task(input_df=my_data)

# To execute:
# df = pd.DataFrame({"col1": [1, 6, 3, 8], "col2": [10, 20, 30, 40]})
# result_df = data_pipeline(my_data=df)
# print(result_df.open(pd.DataFrame).all())
```

**Parameterized Query**

```python
from flytekitplugins.duckdb.task import DuckDBQuery
from flytekit import kwtypes, workflow
import pandas as pd

param_query_task = DuckDBQuery(
    name="filter_by_value",
    inputs=kwtypes(data_df=pd.DataFrame, threshold=int),
    query="SELECT * FROM data_df WHERE value_col > ?"
)

@workflow
def filter_wf(data: pd.DataFrame, threshold_val: int) -> pd.DataFrame:
    return param_query_task(data_df=data, threshold=threshold_val)

# To execute:
# df = pd.DataFrame({"value_col": [10, 20, 5, 30]})
# result_df = filter_wf(data=df, threshold_val=15)
# print(result_df.open(pd.DataFrame).all())
```

**Using MotherDuck with a Secret**

```python
from flytekitplugins.duckdb.task import DuckDBQuery, DuckDBProvider
from flytekit import Secret, workflow
import pandas as pd

# Define a secret for MotherDuck token.
# Ensure this secret (group="motherduck", key="token") is configured in your Flyte environment.
motherduck_secret = Secret(group="motherduck", key="token")

motherduck_query_task = DuckDBQuery(
    name="query_motherduck_data",
    provider=DuckDBProvider.MOTHERDUCK,
    secret_requests=[motherduck_secret],
    query="SELECT * FROM my_motherduck_table LIMIT 10"
)

@workflow
def motherduck_wf() -> pd.DataFrame:
    return motherduck_query_task()

# To execute:
# result_df = motherduck_wf()
# print(result_df.open(pd.DataFrame).all())
```

**Custom Connection Provider**

```python
from flytekitplugins.duckdb.task import DuckDBQuery
from flytekit import Secret, workflow
import duckdb
import pandas as pd

def custom_remote_duckdb_connect(api_key: str):
    """
    A hypothetical custom connection function that uses an API key.
    In a real scenario, this might configure DuckDB to connect to a remote
    data source or a specific file system with authentication.
    """
    con = duckdb.connect(":memory:")
    # Example: Set a custom configuration using the API key
    con.execute(f"SET custom_api_key='{api_key}';")
    return con

# Assume a secret named 'api_key' in group 'my_remote_service' is configured.
remote_api_secret = Secret(group="my_remote_service", key="api_key")

remote_duckdb_query_task = DuckDBQuery(
    name="query_remote_duckdb",
    provider=custom_remote_duckdb_connect,
    secret_requests=[remote_api_secret],
    query="SELECT * FROM some_remote_table LIMIT 5"
)

@workflow
def remote_duckdb_wf() -> pd.DataFrame:
    return remote_duckdb_query_task()

# To execute:
# result_df = remote_duckdb_wf()
# print(result_df.open(pd.DataFrame).all())
```

**Runtime Query Specification**

```python
from flytekitplugins.duckdb.task import DuckDBQuery
from flytekit import kwtypes, workflow
import pandas as pd

# Task defined without a specific query at initialization
dynamic_query_task = DuckDBQuery(
    name="dynamic_duckdb_query",
    inputs=kwtypes(query=str, input_df=pd.DataFrame)
)

@workflow
def dynamic_wf(user_query: str, data: pd.DataFrame) -> pd.DataFrame:
    return dynamic_query_task(query=user_query, input_df=data)

# To execute:
# df = pd.DataFrame({"A": [1, 2], "B": [3, 4]})
# result_df = dynamic_wf(user_query="SELECT A, B*10 as B_scaled FROM input_df", data=df)
# print(result_df.open(pd.DataFrame).all())
```

**Multiple Queries in a Single Task**

```python
from flytekitplugins.duckdb.task import DuckDBQuery
from flytekit import kwtypes, workflow
import pandas as pd

multi_query_task = DuckDBQuery(
    name="multi_step_duckdb_process",
    query=[
        "CREATE TABLE temp_data AS SELECT col1, col2 FROM input_df WHERE col1 > 0",
        "INSERT INTO temp_data VALUES (?, ?)", # Example of parameterized insert
        "SELECT col1, col2 * 10 FROM temp_data"
    ],
    inputs=kwtypes(input_df=pd.DataFrame, insert_params=list)
)

@workflow
def multi_query_wf(data: pd.DataFrame, new_row: list) -> pd.DataFrame:
    return multi_query_task(input_df=data, insert_params=new_row)

# To execute:
# df = pd.DataFrame({"col1": [1, 2, -1], "col2": [10, 20, 30]})
# result_df = multi_query_wf(data=df, new_row=[5, 50])
# print(result_df.open(pd.DataFrame).all())
```

### Considerations and Best Practices

*   **Secret Management:** Always use `Secret` objects for sensitive information like API keys or tokens, especially when using `DuckDBProvider.MOTHERDUCK` or custom providers that require credentials.
*   **Input Data Types:** The plugin efficiently handles `StructuredDataset`, `pandas.DataFrame`, and `pyarrow.Table` by registering them as temporary tables within DuckDB. For very large datasets, `StructuredDataset` is recommended for optimized data transfer.
*   **Query Parameterization:** Use `?` or `$` placeholders for dynamic values in queries. This practice prevents SQL injection vulnerabilities and improves query readability.
*   **Multiple Queries:** When providing a list of queries, only the result of the *last* query is returned as a `StructuredDataset`. Intermediate results are transient within the DuckDB connection and are not directly accessible as task outputs.
*   **Performance:** DuckDB is an in-process OLAP database, making it highly efficient for analytical queries on local data. For datasets that exceed available memory, consider leveraging DuckDB's capabilities to query external data sources directly (e.g., Parquet files on S3) by registering them within your SQL queries.
*   **Error Handling:** Ensure that all necessary secrets are correctly configured and provided for non-local providers to prevent `MissingSecretError`.
*   **Iterative Download (Future):** The current implementation downloads the entire result set of the final query. Future enhancements may enable iterative downloads for extremely large result sets, improving memory efficiency for such cases.
<!--
key: summary_duckdb_plugin_c9559399-52ed-43d5-b3de-f9c5a5fd0a5c
type: summary_end

-->
<!--
code_unit: flytekitplugins.duckdb.task
code_unit_type: class
help_text: ''
key: example_7e268aba-650e-43cd-a666-0e60516ffcdd
type: example

-->