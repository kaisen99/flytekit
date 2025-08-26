
<!--
help_text: ''
key: summary_sqlalchemy_integration_f3800d13-141a-439b-8611-931a7750b8fd
modules:
- flytekitplugins.sqlalchemy.task
questions_to_answer: []
type: summary

-->
SQLAlchemy Integration

The SQLAlchemy integration enables running client-side SQLAlchemy queries directly within Flyte tasks. This allows for seamless interaction with various relational databases, supporting both data retrieval (SELECT) and data manipulation/definition (INSERT, UPDATE, DELETE, DDL) operations.

### Defining SQLAlchemy Tasks

To define a task that executes a SQLAlchemy query, use the `SQLAlchemyTask` construct. This task type is designed to encapsulate database interactions, allowing you to specify the query, database connection details, and expected output schema.

A `SQLAlchemyTask` requires a `name`, a `query_template`, and a `task_config` (an instance of `SQLAlchemyConfig`).

```python
from flytekitplugins.sqlalchemy.task import SQLAlchemyTask, SQLAlchemyConfig
from flytekit.types.schema import FlyteSchema
from flytekit import kwtypes
import pandas as pd

# Define a schema for the output if the query returns data
# This schema will be used to validate and serialize the DataFrame output
MyOutputSchema = FlyteSchema[kwtypes(id=int, name=str, value=float)]

# Configure the database connection
db_config = SQLAlchemyConfig(
    uri="postgresql+psycopg2://user:password@host:port/database_name"
)

# Define a task that selects data
select_task = SQLAlchemyTask(
    name="fetch_data_from_db",
    query_template="SELECT id, name, value FROM my_table WHERE id > {{ .id_threshold }}",
    task_config=db_config,
    inputs={"id_threshold": int},
    output_schema_type=MyOutputSchema,
)

# Define a task that performs an update (no output expected)
update_task = SQLAlchemyTask(
    name="update_data_in_db",
    query_template="UPDATE my_table SET value = {{ .new_value }} WHERE id = {{ .record_id }}",
    task_config=db_config,
    inputs={"new_value": float, "record_id": int},
    output_schema_type=None, # No output expected for DML operations
)
```

The `query_template` supports templating using Jinja2-like syntax. Input variables defined in the `inputs` argument of the task become available within the template for interpolation.

When a query is expected to return data (e.g., a `SELECT` statement), specify an `output_schema_type` using `FlyteSchema`. The results are automatically converted into a Pandas DataFrame and then serialized according to the defined `FlyteSchema`. If no output is expected (e.g., for `INSERT`, `UPDATE`, `DELETE`, or DDL statements), set `output_schema_type=None`.

### Configuring Database Connections

Database connection details are managed through the `SQLAlchemyConfig` object. This configuration allows specifying the database URI and additional connection arguments, including secure handling of credentials.

```python
from flytekitplugins.sqlalchemy.task import SQLAlchemyConfig
from flytekit.models.security import Secret
from flytekit import Secret as FlyteSecret

# Standard URI and connect arguments
config_basic = SQLAlchemyConfig(
    uri="sqlite:///my_database.db",
    connect_args={"timeout": 30}
)

# Using Flyte secrets for sensitive information
# Assume 'my-db-secrets' is a secret group and 'db-password' is a key within it
config_with_secrets = SQLAlchemyConfig(
    uri="postgresql+psycopg2://user@host:port/database_name",
    secret_connect_args={
        "password": FlyteSecret(group_name="my-db-secrets", key="db-password")
    }
)
```

*   **`uri`**: This is the standard SQLAlchemy database URL, specifying the dialect, driver, username, password, host, port, and database name. Refer to the [SQLAlchemy documentation on database URLs](https://docs.sqlalchemy.org/en/14/core/engines.html#database-urls) for detailed format.
*   **`connect_args`**: A dictionary of keyword arguments passed directly to the SQLAlchemy `create_engine` function. Use this for non-sensitive connection parameters like `host`, `port`, `timeout`, etc., if not already part of the URI.
*   **`secret_connect_args`**: This crucial feature allows injecting Flyte secrets directly into the SQLAlchemy connection arguments. Provide a dictionary where keys are the argument names (e.g., "password") and values are `flytekit.Secret` objects. At runtime, the task executor retrieves these secrets from the Flyte secrets store and passes them to the SQLAlchemy engine, ensuring sensitive data is not exposed in task definitions or logs.

### Executing Queries and Handling Results

When a `SQLAlchemyTask` executes, the `SQLAlchemyTaskExecutor` performs the following steps:

1.  **Secret Resolution**: If `secret_connect_args` are provided in the `SQLAlchemyConfig`, the executor fetches the corresponding secret values from the Flyte execution environment. These values are then merged into the `connect_args`.
2.  **Engine Creation**: A SQLAlchemy `Engine` is created using the configured `uri` and resolved `connect_args`.
3.  **Query Interpolation**: The `query_template` is interpolated with any provided task inputs.
4.  **Query Execution**:
    *   If an `output_schema_type` is defined, the interpolated query is executed using `pandas.read_sql_query`. The results are loaded into a Pandas DataFrame, which is then returned and serialized by Flyte according to the `FlyteSchema`.
    *   If `output_schema_type` is `None`, the query is executed directly without expecting a return value. This is suitable for DDL (Data Definition Language) or DML (Data Manipulation Language) operations like `CREATE TABLE`, `INSERT`, `UPDATE`, or `DELETE`.

All query execution happens within the task's container. This implies that the container must have network access to the database specified in the `uri`.

### Container Images

SQLAlchemy tasks run within a container environment that includes the necessary Python packages (SQLAlchemy, Pandas, and database drivers).

The `SQLAlchemyTask` defaults to using pre-built container images provided by `SQLAlchemyDefaultImages`. These images are tagged per Python version and include the required dependencies.

```python
from flytekitplugins.sqlalchemy.task import SQLAlchemyDefaultImages

# Example of how default images are structured
# print(SQLAlchemyDefaultImages.default_image()) # e.g., cr.flyte.org/flyteorg/flytekit:py3.10-sqlalchemy-latest
```

For specialized requirements or custom dependencies, you can specify a custom `container_image` when defining the `SQLAlchemyTask`. Ensure your custom image includes SQLAlchemy, Pandas, and the appropriate database driver (e.g., `psycopg2` for PostgreSQL, `mysqlclient` for MySQL).

### Best Practices and Considerations

*   **Network Connectivity**: Ensure the Flyte task's execution environment (e.g., Kubernetes pod) has network connectivity to your database instance. This often involves configuring VPC peering, security groups, or network policies.
*   **Resource Management**: For queries returning very large datasets, consider the memory implications within the task container, as `pandas.read_sql_query` loads the entire result set into memory. For extremely large datasets, consider using Flyte's `SQLTask` (which leverages a remote SQL engine) or implementing a streaming approach.
*   **Security**: Always use `secret_connect_args` for sensitive credentials like database passwords. Avoid hardcoding them in task definitions or passing them as plain text inputs.
*   **Idempotency for DML/DDL**: When performing DML (e.g., `INSERT`, `UPDATE`, `DELETE`) or DDL (e.g., `CREATE TABLE`, `ALTER TABLE`) operations, design your queries to be idempotent where possible. This prevents unintended side effects if a task is retried.
*   **Error Handling**: Database connection errors or query execution failures (e.g., syntax errors, constraint violations) result in standard Python exceptions within the task, which Flyte captures and reports as task failures. Implement appropriate error handling within your task logic if specific recovery or logging is needed.
*   **Database Drivers**: The `uri` string specifies the database dialect and optionally the driver (e.g., `postgresql+psycopg2`). Ensure the chosen driver is installed in the container image used by the task. The default images include common drivers.
<!--
key: summary_sqlalchemy_integration_f3800d13-141a-439b-8611-931a7750b8fd
type: summary_end

-->
<!--
code_unit: flytekitplugins.sqlalchemy.examples.sqlalchemy_query_task
code_unit_type: class
help_text: ''
key: example_82b6335e-5c31-4441-8427-edc035e0774d
type: example

-->