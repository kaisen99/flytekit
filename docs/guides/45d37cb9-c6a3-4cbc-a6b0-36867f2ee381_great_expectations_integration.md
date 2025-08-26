
<!--
help_text: ''
key: summary_great_expectations_integration_dabaa3c1-87a0-40f1-97c3-24e06a09a67e
modules:
- flytekitplugins.great_expectations.schema
- flytekitplugins.great_expectations.task
questions_to_answer: []
type: summary

-->
Great Expectations Integration

This integration enables robust data validation within Flyte workflows using Great Expectations. It offers two primary mechanisms for incorporating data quality checks:
1.  **Type-based Validation**: Automatically validates data as it flows between tasks, leveraging Flyte's type system.
2.  **Task-based Validation**: Provides a dedicated task for explicit data validation steps within a workflow.

Both approaches require a pre-configured Great Expectations project, typically located in a `great_expectations` directory, containing datasources, data connectors, and expectation suites.

### Type-based Data Validation

Type-based validation allows you to define data expectations directly on the inputs or outputs of your Flyte tasks. When data is passed with a Great Expectations type, the validation automatically occurs during the data serialization/deserialization process.

The core components for type-based validation are:

*   **`GreatExpectationsType`**: This is a custom Flyte type that wraps an underlying data type (e.g., `str`, `FlyteFile`, `FlyteSchema`) and associates it with Great Expectations configuration.
    *   To use it, specify the underlying data type and a `GreatExpectationsFlyteConfig` instance. For example, `GreatExpectationsType[str, GreatExpectationsFlyteConfig(...)]`.
    *   The `config` class method defines the default underlying type and configuration.
    *   Supported underlying data types for validation include `str` (for queries or simple paths), `FlyteFile`, and `FlyteSchema`.

*   **`GreatExpectationsFlyteConfig`**: This configuration object specifies how Great Expectations should interact with your data.
    *   `datasource_name`: The name of the datasource configured in your `great_expectations.yml` that tells Great Expectations where your data lives.
    *   `expectation_suite_name`: The name of the expectation suite to apply for validation.
    *   `data_connector_name`: The name of the data connector used to identify data batches.
    *   `context_root_dir`: The path to your Great Expectations project directory (defaults to `./great_expectations`). This directory must be accessible in the execution environment.
    *   `local_file_path`: **Crucial for `FlyteFile` and `FlyteSchema` inputs.** This specifies a local path where the dataset file will be downloaded before validation. Great Expectations often requires a base directory for its file-based datasources.
    *   `data_asset_name`: Required when using a `RuntimeDataConnector` for identifying the data asset.
    *   `checkpoint_params`: Optional parameters for configuring a Great Expectations `SimpleCheckpoint`.
    *   `batch_request_config`: An optional `BatchRequestConfig` object for advanced batch request customization.

*   **`GreatExpectationsTypeTransformer`**: This internal component handles the conversion between Python values and Flyte's literal types for `GreatExpectationsType`. It orchestrates the Great Expectations validation process during `to_python_value` (when data is being loaded into the task).
    *   It determines the appropriate data connector based on your configuration.
    *   It constructs the Great Expectations `BatchRequest` or `RuntimeBatchRequest`.
    *   It executes the configured `checkpoint` with the specified `expectation_suite_name`.
    *   If validation fails, it raises a `great_expectations.exceptions.ValidationError`, causing the Flyte task to fail.

#### Usage Example: Type-based Validation

To use type-based validation, define a task that accepts `GreatExpectationsType` as an input.

```python
import os
from flytekit import task, workflow, FlyteFile, FlyteSchema
from flytekitplugins.great_expectations.schema import GreatExpectationsType, GreatExpectationsFlyteConfig

# Assume 'great_expectations' directory exists with a configured datasource,
# data connector, and expectation suite (e.g., 'my_datasource', 'my_data_connector', 'my_suite').
# For FlyteFile/FlyteSchema, ensure your GE config points to the directory where local_file_path resides.

@task
def validate_data_with_ge_type(
    data_to_validate: GreatExpectationsType[
        FlyteFile,
        GreatExpectationsFlyteConfig(
            datasource_name="my_datasource",
            data_connector_name="my_data_connector",
            expectation_suite_name="my_suite",
            local_file_path="/tmp/data_for_ge/my_dataset.csv", # Must be a path accessible in the container
            context_root_dir="./great_expectations",
        )
    ]
) -> str:
    """
    This task receives a FlyteFile wrapped in GreatExpectationsType.
    The validation happens automatically before this task's logic executes.
    If validation passes, the task proceeds. If it fails, the task will error.
    """
    # If execution reaches here, validation succeeded.
    # 'data_to_validate' will be the original FlyteFile object.
    print(f"Data validation succeeded for: {data_to_validate.path}")
    return "Validation successful!"

@workflow
def ge_type_workflow(input_file: FlyteFile) -> str:
    return validate_data_with_ge_type(data_to_validate=input_file)

# To run this locally, you would need:
# 1. A 'great_expectations' directory in your project root.
# 2. Inside 'great_expectations/great_expectations.yml', configure 'my_datasource' and 'my_data_connector'.
#    For file-based data, ensure your datasource points to the directory containing '/tmp/data_for_ge/'.
# 3. An expectation suite named 'my_suite' in 'great_expectations/expectations/'.
# 4. A sample CSV file to pass as 'input_file'.
```

#### Considerations for Type-based Validation

*   The `context_root_dir` must be available in the container where the task runs. This typically means including your `great_expectations` directory in your Docker image.
*   For `FlyteFile` and `FlyteSchema` inputs, `local_file_path` is mandatory. This path is used to download the remote data to a local file system location that Great Expectations can access. Ensure this path aligns with your Great Expectations datasource configuration.
*   Validation occurs implicitly as part of Flyte's type system. If validation fails, a `ValidationError` is raised, and the task execution stops.

### Task-based Data Validation

Task-based validation provides a dedicated `GreatExpectationsTask` that you can explicitly include in your workflow to perform data quality checks. This is useful when you want validation to be a distinct, observable step in your data pipeline, or when the validation logic is more complex and requires specific task inputs.

*   **`GreatExpectationsTask`**: This is a specialized Flyte task designed to run Great Expectations validations.
    *   It inherits from `PythonInstanceTask` and has a custom `_TASK_TYPE` of "great_expectations".
    *   It takes a single input, which can be `str`, `FlyteFile`, or `FlyteSchema`, representing the dataset to validate.
    *   Its `execute` method performs the Great Expectations validation using the provided configuration.
    *   On successful validation, it returns the validation result as a dictionary. On failure, it raises a `great_expectations.exceptions.ValidationError`.
    *   Parameters like `datasource_name`, `expectation_suite_name`, `data_connector_name`, `context_root_dir`, `local_file_path`, `data_asset_name`, and `checkpoint_params` mirror those in `GreatExpectationsFlyteConfig`.
    *   `task_config`: This parameter accepts a `BatchRequestConfig` object for detailed control over the Great Expectations batch request.

*   **`BatchRequestConfig`**: This configuration object allows fine-grained control over the Great Expectations `BatchRequest` or `RuntimeBatchRequest`.
    *   `data_connector_query`: Used for standard `BatchRequest` to specify how to query the data connector.
    *   `runtime_parameters`: Used for `RuntimeBatchRequest` to pass data or queries directly at runtime.
    *   `batch_identifiers`: Identifiers for the data batch.
    *   `batch_spec_passthrough`: Additional parameters for the data reader method (e.g., if your file lacks an extension).

#### Usage Example: Task-based Validation

Define and instantiate a `GreatExpectationsTask` within your workflow.

```python
import os
from flytekit import task, workflow, FlyteFile, FlyteSchema
from flytekitplugins.great_expectations.task import GreatExpectationsTask, BatchRequestConfig

# Assume 'great_expectations' directory exists with a configured datasource,
# data connector, and expectation suite (e.g., 'my_datasource', 'my_data_connector', 'my_suite').

# Example of a task that produces a dataset
@task
def generate_dataset() -> FlyteFile:
    # In a real scenario, this would generate a dataset and return its FlyteFile handle
    local_path = "/tmp/my_generated_data.csv"
    with open(local_path, "w") as f:
        f.write("col1,col2\n1,a\n2,b\n3,c\n")
    return FlyteFile(local_path)

# Define the Great Expectations validation task
ge_validation_task = GreatExpectationsTask(
    name="my_data_validation_task",
    datasource_name="my_datasource",
    data_connector_name="my_data_connector",
    expectation_suite_name="my_suite",
    inputs={"dataset_to_validate": FlyteFile}, # The input name must match the one used in the workflow
    local_file_path="/tmp/ge_data_cache/input.csv", # Where the dataset will be downloaded for GE
    context_root_dir="./great_expectations",
    # Example of using BatchRequestConfig for a RuntimeBatchRequest with a query
    # task_config=BatchRequestConfig(
    #     runtime_parameters={"query": "SELECT * FROM my_table"},
    #     batch_identifiers={"batch_id": "my_run_123"}
    # )
)

@workflow
def ge_task_workflow() -> dict:
    generated_data = generate_dataset()
    validation_result = ge_validation_task(dataset_to_validate=generated_data)
    return validation_result

# To run this locally, you would need:
# 1. A 'great_expectations' directory in your project root.
# 2. Inside 'great_expectations/great_expectations.yml', configure 'my_datasource' and 'my_data_connector'.
#    For file-based data, ensure your datasource points to the directory containing '/tmp/ge_data_cache/'.
# 3. An expectation suite named 'my_suite' in 'great_expectations/expectations/'.
```

#### Considerations for Task-based Validation

*   The `GreatExpectationsTask` expects exactly one input argument, which represents the dataset to be validated.
*   Similar to type-based validation, the `context_root_dir` must be available in the task's execution environment.
*   `local_file_path` is required for `FlyteFile` and `FlyteSchema` inputs to ensure Great Expectations can access the data locally.
*   The output of `GreatExpectationsTask` is a dictionary containing the detailed validation results from Great Expectations. This allows for programmatic inspection of validation outcomes.

### Common Concepts and Best Practices

*   **Great Expectations Project Setup**: Both integration methods rely heavily on a correctly configured Great Expectations project. This includes `great_expectations.yml`, datasources, data connectors, and expectation suites. Ensure this project structure is available in your Flyte task containers.
*   **`local_file_path`**: This parameter is critical when validating `FlyteFile` or `FlyteSchema` data. It specifies the local path within the container where the remote data will be downloaded for Great Expectations to access. This path must align with how your Great Expectations datasource is configured (e.g., if your datasource expects data in `/data`, then `local_file_path` should point to a file within `/data`).
*   **Runtime vs. Standard Batch Requests**:
    *   **`RuntimeDataConnector`** (used with `runtime_parameters` in `BatchRequestConfig`): Allows you to pass data directly (e.g., a Pandas DataFrame, a Spark DataFrame, or a SQL query string) to Great Expectations for validation without needing to configure a persistent data source for that specific batch. This is powerful for dynamic data.
    *   **Standard `BatchRequest`** (used with `data_connector_query`): Relies on Great Expectations' configured datasources and data connectors to identify and load data batches based on queries or identifiers.
*   **Error Handling**: If Great Expectations validation fails, the integration raises a `great_expectations.exceptions.ValidationError`. This causes the Flyte task to fail, providing clear feedback on data quality issues.
*   **Data Docs**: While the integration performs validation, connecting Great Expectations Data Docs to the Flyte Console for easy viewing of validation reports is a future enhancement. For now, you would typically configure Great Expectations to build Data Docs to a persistent storage location (e.g., S3) and access them separately.
<!--
key: summary_great_expectations_integration_dabaa3c1-87a0-40f1-97c3-24e06a09a67e
type: summary_end

-->
<!--
code_unit: flytekitplugins.great_expectations.examples.data_validation_task
code_unit_type: class
help_text: ''
key: example_6eef9f11-45f9-4251-b2d4-5b5e2dbf4691
type: example

-->