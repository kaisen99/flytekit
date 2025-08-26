
<!--
help_text: ''
key: summary_pandera_data_validation_fa091af1-0c6e-4d6c-93c1-82b235756198
modules:
- flytekitplugins.pandera.config
- flytekitplugins.pandera.pandas_renderer
- flytekitplugins.pandera.pandas_transformer
questions_to_answer: []
type: summary

-->
Pandera Data Validation

Flyte integrates with the Pandera library to provide robust data validation for `pandas.DataFrame` objects. This integration ensures data quality and consistency within your data pipelines by automatically validating DataFrames against predefined schemas at task boundaries.

## Integrating Pandera Schemas

To enable Pandera data validation, define your data schema using Pandera's `DataFrameModel` and annotate your Flyte task inputs or outputs with `pandera.typing.DataFrame`.

### Defining a Schema

Define a Pandera schema using `pandera.DataFrameModel`. This model specifies the expected columns, their data types, and any constraints.

```python
import pandera as pa
from pandera.typing import DataFrame, Series

class MyDataFrameSchema(pa.DataFrameModel):
    column_a: Series[int] = pa.Field(ge=0)
    column_b: Series[str]
    column_c: Series[float] = pa.Field(nullable=True)

    class Config:
        strict = True  # Ensure no extra columns are present
        coerce = True  # Coerce data types if possible
```

### Applying Schemas in Tasks

Annotate task inputs or outputs with `pandera.typing.DataFrame` parameterized by your schema model. The `PanderaPandasTransformer` automatically intercepts these types and applies the validation logic.

```python
import pandas as pd
from flytekit import task, workflow
from pandera.typing import DataFrame

# Assume MyDataFrameSchema is defined as above

@task
def process_data(df: DataFrame[MyDataFrameSchema]) -> DataFrame[MyDataFrameSchema]:
    # Your data processing logic here
    # The input 'df' is guaranteed to conform to MyDataFrameSchema
    # The output DataFrame will also be validated against MyDataFrameSchema
    df["column_a"] = df["column_a"] * 2
    return df

@workflow
def my_workflow():
    initial_data = pd.DataFrame({
        "column_a": [1, 2, 3],
        "column_b": ["x", "y", "z"],
        "column_c": [1.1, 2.2, None],
    })
    process_data(df=initial_data)
```

When `process_data` is executed, the input `initial_data` is validated against `MyDataFrameSchema`. If validation fails, an error is raised (by default), preventing the task from proceeding with invalid data. Similarly, the DataFrame returned by `process_data` is validated before being passed to subsequent tasks or stored as an output.

## Configuring Validation Behavior

The `ValidationConfig` class allows you to control how validation errors are handled.

### `ValidationConfig`

The `ValidationConfig` class provides the `on_error` attribute, which determines the behavior when a validation error occurs:

*   `"raise"` (default): An exception (`pandera.errors.SchemaErrors` or `pandera.errors.SchemaError`) is raised, causing the task to fail.
*   `"warn"`: A warning is logged, but the task continues execution. The invalid DataFrame is passed through.

To specify the `on_error` behavior, use `typing.Annotated` with your `pandera.typing.DataFrame` annotation:

```python
from typing import Annotated
from flytekitplugins.pandera.config import ValidationConfig

@task
def process_data_with_warning(
    df: Annotated[DataFrame[MyDataFrameSchema], ValidationConfig(on_error="warn")]
) -> DataFrame[MyDataFrameSchema]:
    # If validation fails for 'df', a warning is logged, but the task continues.
    # The returned DataFrame will still be validated with the default 'raise' behavior
    # unless explicitly configured.
    return df
```

This configuration is applied by the `PanderaPandasTransformer` during the serialization and deserialization of `pandera.typing.DataFrame` objects.

## Automatic Validation and Reporting

The `PanderaPandasTransformer` automatically performs schema validation and generates detailed HTML reports for every validation attempt, whether successful or failed. These reports are published as Flyte Decks, making it easy to inspect data quality directly from the Flyte UI.

### Validation Process

When a `pandas.DataFrame` is passed as an input or output of a task annotated with a Pandera schema:

1.  The `PanderaPandasTransformer` extracts the `DataFrameSchema` from the type hint.
2.  It attempts to validate the DataFrame against this schema using Pandera's `lazy=True` mode, which collects all validation errors before raising an exception.
3.  Based on the `ValidationConfig.on_error` setting, it either raises an error (failing the task) or logs a warning.
4.  Regardless of success or failure, a validation report is generated.

### Understanding Validation Reports

The `PandasReportRenderer` is responsible for creating comprehensive HTML reports. These reports provide:

*   **Summary**: A high-level overview including the schema name, data shape, and total counts of schema-level and data-level errors.
*   **Data Preview**: A glimpse of the first few rows of the DataFrame.
*   **Schema-level Errors**: Details about errors related to the DataFrame's metadata, such as incorrect column names or data types.
*   **Data-level Errors**: Information about value-level errors within the data, such as null values where not allowed, or values outside specified ranges. This section often includes the number of failures and example failure cases.

These reports are invaluable for debugging data quality issues, allowing developers to quickly identify why a validation failed and pinpoint the problematic data points or schema definitions.

## Best Practices and Considerations

*   **Granular Schemas**: Define specific schemas for different stages of your pipeline. For example, an `InputSchema` for raw data and a `CleanedSchema` for processed data.
*   **Schema Evolution**: As your data evolves, update your Pandera schemas to reflect the changes.
*   **Performance**: For extremely large DataFrames, validation can introduce overhead. Consider the trade-offs between validation strictness and performance. The `lazy=True` validation helps by collecting all errors in one pass rather than stopping at the first error.
*   **Debugging with Reports**: Leverage the automatically generated Flyte Decks to quickly diagnose data quality issues. The detailed error reports provide actionable insights into validation failures.
*   **`on_error="warn"` for Exploration**: Using `on_error="warn"` can be useful during development or for exploratory data analysis where you want to observe data quality issues without immediately failing the task. However, for production pipelines, `on_error="raise"` is generally recommended to ensure data integrity.
<!--
key: summary_pandera_data_validation_fa091af1-0c6e-4d6c-93c1-82b235756198
type: summary_end

-->
<!--
code_unit: flytekitplugins.pandera.examples.pandera_validation_task
code_unit_type: class
help_text: ''
key: example_8dd047c8-6030-4d39-bdd8-57e8ee77b03d
type: example

-->