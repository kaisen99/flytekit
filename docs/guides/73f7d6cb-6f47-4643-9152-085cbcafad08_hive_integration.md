
<!--
help_text: ''
key: summary_hive_integration_071ef6b9-12ee-4d21-816a-51a2bb67ba24
modules:
- flytekitplugins.hive.task
questions_to_answer: []
type: summary

-->
Hive Integration enables the execution of Hive queries as managed tasks. This integration provides a structured way to define, configure, and run Hive operations, including data transformations and data loading, within a larger workflow system.

### Configuring Hive Tasks

Use `HiveConfig` to define static configuration parameters for Hive tasks. These configurations are set when a task is defined and are not parameterized during execution.

*   **`cluster_label`**: A string that identifies the specific Hive cluster where the query should execute. This is crucial for routing tasks to the correct environment.
*   **`tags`**: An optional list of strings that can be associated with the remote execution request for organizational or filtering purposes.

**Example:**

```python
from flytekitplugins.hive.task import HiveConfig

hive_config = HiveConfig(
    cluster_label="production_hive_cluster",
    tags=["etl", "daily_job"]
)
```

### Defining General Hive Tasks

The `HiveTask` class provides a flexible way to execute arbitrary Hive queries. It is suitable for any Hive operation, including DDL (Data Definition Language) and DML (Data Manipulation Language) statements that may or may not produce a structured output.

When defining a `HiveTask`:

*   **`name`**: A unique identifier for the task.
*   **`query_template`**: The actual Hive query to be executed. This template supports Flyte's Golang templating syntax, allowing for dynamic query construction based on task inputs.
*   **`inputs`**: An optional dictionary mapping input variable names to their Python types. These inputs can be referenced within the `query_template`.
*   **`output_schema_type`**: An optional `FlyteSchema` type. If the query produces a tabular dataset, specify this type to capture the results. The output will be available under the `results` output variable.
*   **`task_config`**: An optional `HiveConfig` object to apply specific configurations to this task. If not provided, a default `HiveConfig` is used.

**Example: Running a Hive DDL/DML query**

This example demonstrates a `HiveTask` that creates a table and inserts data, without producing a direct output schema.

```python
from flytekitplugins.hive.task import HiveTask
from flytekit.types.schema import FlyteSchema
from typing import Dict, Type

# Define a schema for a potential output, even if not used in this specific task
# class MyOutputSchema(FlyteSchema):
#     id: int
#     name: str

create_and_insert_task = HiveTask(
    name="create_and_insert_data",
    query_template="""
        CREATE TABLE IF NOT EXISTS my_data (
            id INT,
            name STRING
        );
        INSERT INTO TABLE my_data VALUES (1, 'Alice'), (2, 'Bob');
    """,
    task_config=HiveConfig(cluster_label="dev_hive_cluster")
)
```

### Handling Select Queries with HiveSelectTask

For tasks that specifically involve `SELECT` queries and produce a tabular output, use `HiveSelectTask`. This specialized task simplifies the process of capturing query results by automatically generating the necessary boilerplate SQL to stage data into a temporary table and then move it to a managed external table location.

`HiveSelectTask` extends `HiveTask` and includes:

*   **`select_query`**: The core `SELECT` statement. This query's results will be captured.
*   **`stage_query`**: An optional query string that executes *before* the `select_query`. This is useful for setting Hive session properties (e.g., `SET hive.exec.dynamic.partition.mode=nonstrict;`) or configuring the execution engine (e.g., `SET hive.execution.engine=tez;`).
*   **`output_schema_type`**: A required `FlyteSchema` type that defines the structure of the data returned by the `select_query`.

The `HiveSelectTask` automatically formats the provided `select_query` into a sequence of operations:
1.  Executes the `stage_query` (if provided).
2.  Creates a temporary table (`_tmp`) from the `select_query` results.
3.  Creates an external table with the same schema as the temporary table, pointing its location to the task's designated output path.
4.  Inserts data from the temporary table into the external table.
5.  Drops the temporary table.

This automated process ensures that the query results are persisted in a discoverable and managed location.

**Example: Selecting and capturing data**

```python
from flytekitplugins.hive.task import HiveSelectTask, HiveConfig
from flytekit.types.schema import FlyteSchema
from typing import Dict, Type

class UserSchema(FlyteSchema):
    user_id: int
    user_name: str
    email: str

select_users_task = HiveSelectTask(
    name="select_active_users",
    select_query="""
        SELECT id as user_id, name as user_name, email
        FROM users
        WHERE status = 'active'
    """,
    inputs={
        "min_id": int,
        "max_id": int
    },
    output_schema_type=UserSchema,
    config=HiveConfig(cluster_label="reporting_hive_cluster"),
    stage_query="""
        SET hive.cli.print.header=true;
        SET hive.exec.compress.output=true;
    """
)
```

### Managing Task Outputs

When `output_schema_type` is specified for either `HiveTask` or `HiveSelectTask`, the results of the query are captured and made available as a `FlyteSchema` output named `results`. This allows downstream tasks to consume the structured data produced by the Hive query.

### Key Considerations

*   **Query Templating**: Leverage the Golang templating syntax within `query_template` to create dynamic and reusable Hive tasks. Inputs defined for the task are accessible as variables within the template.
*   **Cluster Labeling**: Always specify a `cluster_label` in `HiveConfig` to ensure tasks are routed to the correct Hive environment.
*   **Idempotency**: Design Hive queries, especially for `HiveTask`, to be idempotent where possible. This ensures that re-running a task due to retries or failures does not lead to inconsistent data.
*   **Performance**: For `HiveSelectTask`, the process of creating temporary and external tables involves data movement. Consider the volume of data being processed and the performance characteristics of the underlying Hive cluster.
*   **Retry and Timeout Settings**: While the internal `get_custom` method shows `timeout_sec` and `retry_count`, these are deprecated. Configure task-level timeouts and retries directly on the task definition for proper handling.
*   **Schema Evolution**: When using `FlyteSchema` for outputs, be mindful of schema evolution in your Hive tables. Ensure your `output_schema_type` remains compatible with the actual data produced by the query.
<!--
key: summary_hive_integration_071ef6b9-12ee-4d21-816a-51a2bb67ba24
type: summary_end

-->
<!--
code_unit: flytekitplugins.hive.examples.simple_hive_task
code_unit_type: class
help_text: ''
key: example_17f08aee-93ea-4e22-9124-4ffdd7768dc3
type: example

-->