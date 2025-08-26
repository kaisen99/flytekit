
<!--
help_text: ''
key: summary_hive_plugin_4bdc109e-a356-444d-b15c-5851ad0174a0
modules:
- flytekitplugins.hive.task
questions_to_answer: []
type: summary

-->
The Hive plugin enables the execution of Hive queries within workflows, allowing for seamless integration with data warehousing operations. It provides components to define, configure, and execute various types of Hive tasks, from arbitrary queries to structured data selections.

### Defining Hive Tasks

The `HiveTask` class serves as the fundamental building block for executing Hive queries. It supports running any valid HiveQL statement, including DDL (Data Definition Language) and DML (Data Manipulation Language) operations.

To define a `HiveTask`, specify the following:

*   `name`: A unique identifier for the task within the project.
*   `query_template`: The HiveQL query to execute. This template supports Flyte's Golang templating syntax, allowing for dynamic injection of inputs.
*   `inputs`: An optional dictionary mapping input names to their Python types. These inputs are made available to the `query_template`.
*   `output_schema_type`: An optional `FlyteSchema` type if the query produces a tabular dataset as output. If provided, the task expects a `results` output matching this schema.
*   `task_config`: An optional `HiveConfig` object to specify static configuration for the task.

**Example: Running a DDL Query**

```python
from flytekitplugins.hive.task import HiveTask

create_table_task = HiveTask(
    name="create_my_table",
    query_template="""
        CREATE TABLE my_database.my_table (
            id INT,
            name STRING
        ) STORED AS ORC;
    """,
)
```

**Example: Running a DML Query with Inputs**

```python
from flytekitplugins.hive.task import HiveTask
from typing import Dict, Type

insert_data_task = HiveTask(
    name="insert_into_my_table",
    query_template="""
        INSERT INTO my_database.my_table VALUES ({{ .inputs.id }}, '{{ .inputs.name }}');
    """,
    inputs={"id": int, "name": str},
)
```

When a `HiveTask` is executed, the `query_template` is rendered with the provided inputs, and the resulting HiveQL is submitted to the configured Hive cluster. The plugin handles the serialization of the query and configuration for remote execution.

### Configuring Hive Tasks

The `HiveConfig` class provides static configuration options for Hive tasks. These configurations are defined at the time the task is constructed and remain constant for all executions of that task.

Available configuration parameters include:

*   `cluster_label`: A string used to identify the target Hive cluster where the query should be executed. This is crucial for routing tasks to the correct environment.
*   `tags`: An optional list of strings that can be associated with the remote execution request, useful for tracking or categorization.

**Example: Applying Hive Configuration**

```python
from flytekitplugins.hive.task import HiveTask, HiveConfig

my_hive_config = HiveConfig(
    cluster_label="production_hive_cluster",
    tags=["data_pipeline", "daily_job"],
)

configured_hive_task = HiveTask(
    name="process_data_with_config",
    query_template="""
        SELECT COUNT(*) FROM my_database.another_table;
    """,
    task_config=my_hive_config,
)
```

Note that `HiveConfig` values are static. Dynamic configuration changes at execution launch are not supported through this mechanism.

### Handling SELECT Queries

The `HiveSelectTask` is a specialized `HiveTask` designed to simplify the execution of `SELECT` queries and the materialization of their results into a `FlyteSchema` output. It automatically generates the necessary HiveQL to stage the query results into a temporary table and then an external table, ensuring the output data is correctly linked to the workflow.

`HiveSelectTask` extends `HiveTask` and automatically formats the provided `select_query` into a sequence of operations:

1.  An optional `stage_query` is executed first. This is useful for setting Hive session properties or enabling specific execution engines (e.g., Apache Tez).
2.  A temporary table is created from the `select_query` results.
3.  An external table is created with the same schema as the temporary table, and its location is set to the workflow's raw output data prefix.
4.  Data is inserted from the temporary table into the external table.
5.  The temporary table is dropped.

To define a `HiveSelectTask`, specify:

*   `name`: A unique identifier for the task.
*   `select_query`: The core `SELECT` statement that produces the desired tabular dataset.
*   `inputs`: Optional inputs for the `select_query`.
*   `output_schema_type`: The `FlyteSchema` type representing the structure of the query's output. This is mandatory for `HiveSelectTask` to correctly materialize results.
*   `config`: An optional `HiveConfig` object.
*   `stage_query`: An optional query to run before the `select_query`.

**Example: Selecting Data and Materializing Output**

```python
from flytekitplugins.hive.task import HiveSelectTask
from flytekit.types.schema import FlyteSchema
from typing import Dict, Type

# Define the schema for the output data
class MyOutputSchema(FlyteSchema):
    id: int
    name: str
    value: float

select_data_task = HiveSelectTask(
    name="select_and_materialize_data",
    select_query="""
        SELECT id, name, value FROM my_database.source_table WHERE value > {{ .inputs.threshold }};
    """,
    inputs={"threshold": float},
    output_schema_type=MyOutputSchema,
    stage_query="SET hive.execution.engine=tez;", # Example: Enable Tez for performance
)
```

The `output_schema_type` ensures that the data produced by the `select_query` is correctly structured and accessible as a `FlyteSchema` object in downstream tasks.

### Key Considerations and Best Practices

*   **Query Templating:** Leverage Golang templating in `query_template` and `select_query` to make tasks dynamic and reusable. Ensure all necessary inputs are declared in the `inputs` dictionary.
*   **Output Schema:** For tasks that produce tabular data, always define an `output_schema_type` (especially with `HiveSelectTask`) to ensure proper data materialization and type safety.
*   **Cluster Labeling:** Consistently use `cluster_label` in `HiveConfig` to direct tasks to the appropriate Hive environment.
*   **Performance Tuning:** For `HiveSelectTask`, utilize the `stage_query` parameter to include Hive session settings that can optimize query performance, such as enabling specific execution engines (e.g., Tez) or adjusting memory configurations.
*   **Error Handling and Retries:** Standard task retry and timeout settings apply to Hive tasks. Configure these at the task definition level to handle transient failures gracefully.
*   **Idempotency:** Design Hive queries to be idempotent where possible, especially for DML operations, to ensure consistent results if a task is retried.
<!--
key: summary_hive_plugin_4bdc109e-a356-444d-b15c-5851ad0174a0
type: summary_end

-->
<!--
code_unit: flytekitplugins.hive.task
code_unit_type: class
help_text: ''
key: example_dc3ba43c-ce3c-4a00-9e19-463c988b62c5
type: example

-->