
<!--
help_text: ''
key: summary_snowflake_integration_07ff8900-3766-4d96-ae80-dec0e69acfc7
modules:
- flytekitplugins.snowflake.connector
- flytekitplugins.snowflake.task
questions_to_answer: []
type: summary

-->
# Snowflake Integration

The Snowflake integration enables the execution of SQL queries on Snowflake as native tasks within workflows. This allows for seamless orchestration of data transformations and analytical workloads directly on your Snowflake data warehouse.

## Core Concepts

The integration is built around two primary components: the Snowflake Task and the Snowflake Connector.

### Snowflake Task

The `SnowflakeTask` is the user-facing interface for defining Snowflake operations. It extends the base SQL task capabilities, allowing you to specify a SQL query, define inputs, and capture outputs.

When defining a Snowflake task, you provide:

*   **Query Template**: The SQL statement to be executed. This template supports Flyte's Golang templating format, enabling dynamic injection of task inputs into the query.
*   **Snowflake Configuration**: Connection details for your Snowflake instance.
*   **Inputs**: Optional parameters that can be passed to your SQL query.
*   **Output Schema**: An optional `StructuredDataset` type to define the schema of the results if your query produces output.

### Snowflake Configuration

Connection to Snowflake is managed through the `SnowflakeConfig` object. This configuration specifies the necessary details to establish a connection to your Snowflake account.

The `SnowflakeConfig` requires the following parameters:

*   `user`: Your Snowflake username.
*   `account`: Your Snowflake account identifier (e.g., `myorg-myaccount`).
*   `database`: The default database to use.
*   `schema`: The default schema within the database.
*   `warehouse`: The virtual warehouse to use for query execution.

You can retrieve these details from your Snowflake environment using the following SQL query:

```sql
SELECT
    CURRENT_USER() AS "User",
    CONCAT(CURRENT_ORGANIZATION_NAME(), '-', CURRENT_ACCOUNT_NAME()) AS "Account",
    CURRENT_DATABASE() AS "Database",
    CURRENT_SCHEMA() AS "Schema",
    CURRENT_WAREHOUSE() AS "Warehouse";
```

### Snowflake Connector

The Snowflake connector is responsible for the asynchronous execution and management of Snowflake queries. It acts as the bridge between the task definition and the Snowflake service.

Key functionalities of the connector include:

*   **Asynchronous Query Execution**: Queries are submitted to Snowflake using `execute_async`, allowing the task to run in the background without blocking the workflow.
*   **Status Monitoring**: The connector continuously polls Snowflake for the status of the submitted query using the `query_id`.
*   **Result Retrieval**: If the query produces output and the task defines an output schema, the connector constructs a `StructuredDataset` URI pointing to the query results.
*   **Query Cancellation**: The connector supports cancelling a running Snowflake query, which translates to executing `SYSTEM$CANCEL_QUERY` on the Snowflake side.
*   **Authentication**: Authentication to Snowflake is handled using a private key, which must be securely configured in the environment where the connector operates.

The `SnowflakeJobMetadata` object is used internally by the connector to track the state and essential details of a running Snowflake query, including the `query_id` and connection parameters.

## Usage

### Defining a Snowflake Task

To define a Snowflake task, instantiate `SnowflakeTask` with your query, configuration, and optionally, inputs and outputs.

```python
from flytekitplugins.snowflake.task import SnowflakeTask, SnowflakeConfig
from flytekit.types.structured.structured import StructuredDataset

# Configure your Snowflake connection
snowflake_config = SnowflakeConfig(
    user="your_user",
    account="your_account",
    database="your_database",
    schema="your_schema",
    warehouse="your_warehouse",
)

# Define a Snowflake task that does not produce output
no_output_task = SnowflakeTask(
    name="create_table_task",
    query_template="CREATE TABLE my_table (id INT, name VARCHAR);",
    task_config=snowflake_config,
)

# Define a Snowflake task that takes inputs and produces a StructuredDataset output
# The query uses Flyte's Golang templating for inputs
query_with_inputs_and_output = """
SELECT
    id,
    name
FROM
    my_table
WHERE
    id > {{ .min_id }} AND name = '{{ .filter_name }}';
"""

output_schema = StructuredDataset

query_task_with_output = SnowflakeTask(
    name="select_data_task",
    query_template=query_with_inputs_and_output,
    task_config=snowflake_config,
    inputs={"min_id": int, "filter_name": str},
    output_schema_type=output_schema,
)
```

### Passing Inputs to Queries

Inputs defined in the `SnowflakeTask`'s `inputs` dictionary are made available to the `query_template` using Flyte's Golang templating syntax. For example, `{{ .input_name }}` will be replaced with the value of the `input_name` parameter at runtime.

```python
# Example of using inputs in a query template
# The 'min_id' and 'filter_name' are passed as task inputs
# and referenced in the query using {{ .min_id }} and {{ .filter_name }}
@task
def my_workflow_task(min_val: int, name_filter: str) -> StructuredDataset:
    return query_task_with_output(min_id=min_val, filter_name=name_filter)
```

### Handling Query Outputs

If your Snowflake query returns a result set, you can capture it as a `StructuredDataset`. Specify the `output_schema_type` parameter in the `SnowflakeTask` constructor. The connector will then generate a URI pointing to the query results in Snowflake, which can be consumed by downstream tasks.

The `StructuredDataset` output will contain a URI that can be used to access the query results. The format of the `StructuredDataset` is determined by the Snowflake query itself.

## Advanced Considerations

### Asynchronous Execution and Monitoring

Snowflake queries executed via this integration run asynchronously. The connector continuously monitors the query status in Snowflake. This design ensures that long-running queries do not block the workflow execution, and the workflow progresses only when the Snowflake query reaches a terminal state (succeeded, failed, or aborted).

Query status and details can be accessed through the task logs, which include a direct link to the Snowflake query details page.

### Query Cancellation

Tasks can be cancelled during execution. When a task is cancelled, the connector attempts to cancel the corresponding Snowflake query using the `SYSTEM$CANCEL_QUERY` function. This ensures that resources are released and unnecessary computation is stopped.

### Authentication

The Snowflake connector authenticates to Snowflake using a private key. Ensure that the private key is securely configured and accessible in the environment where the Flyte backend is running. This typically involves mounting the private key as a secret or using an environment variable.

### Performance

The asynchronous nature of the integration means that the overhead of managing the Snowflake query lifecycle is handled efficiently. Performance is primarily dictated by the Snowflake query execution time and the network latency between the Flyte backend and Snowflake.

### Best Practices

*   **Parameterize Queries**: Always use inputs and templating for dynamic values in your queries to prevent SQL injection vulnerabilities and improve reusability.
*   **Manage Warehouses**: Ensure your `warehouse` configuration is appropriate for the expected workload size and performance requirements.
*   **Error Handling**: Monitor task logs for Snowflake-specific errors. The connector translates Snowflake errors into task failures.
*   **Output Schema Definition**: Clearly define your `output_schema_type` when expecting results to ensure proper data handling and type checking in downstream tasks.
<!--
key: summary_snowflake_integration_07ff8900-3766-4d96-ae80-dec0e69acfc7
type: summary_end

-->
<!--
code_unit: flytekitplugins.snowflake.examples.simple_snowflake_task
code_unit_type: class
help_text: ''
key: example_8d20bfb1-7355-4545-9752-d03b5502b067
type: example

-->