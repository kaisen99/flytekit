
<!--
help_text: ''
key: summary_structured_datasets_a3a29d80-8722-43f8-9ef6-468d7fc83d6e
modules:
- flytekit.types.structured.structured_dataset
- flytekit.types.structured.basic_dfs
- flytekit.types.structured.bigquery
- flytekit.types.structured.snowflake
questions_to_answer: []
type: summary

-->
Structured Datasets provide a unified way to handle structured data, such as dataframes, within workflows. This abstraction allows tasks to operate on data without needing to know the underlying storage system (e.g., S3, Google Cloud Storage, BigQuery, Snowflake) or the specific file format (e.g., Parquet, CSV). This enables seamless interoperability and data portability across different environments and data sources.

### The `StructuredDataset` Type

The `StructuredDataset` class is the primary user-facing type for representing structured data. It acts as a wrapper around your in-memory dataframe or a reference to data stored externally.

Key properties of a `StructuredDataset` instance include:

*   **`uri`**: An optional string representing the location of the structured data (e.g., `s3://my-bucket/data.parquet`, `bq://project.dataset.table`). When a `StructuredDataset` is returned from a task, Flytekit automatically uploads the data to a generated URI if not explicitly provided.
*   **`file_format`**: An optional string indicating the format of the data (e.g., `"parquet"`, `"csv"`). This helps the system determine the correct handler for serialization and deserialization.
*   **`dataframe`**: An optional reference to the actual in-memory dataframe object (e.g., a Pandas DataFrame or PyArrow Table).

**Instantiating a `StructuredDataset`**

You can create a `StructuredDataset` in two primary ways:

1.  **From an in-memory dataframe**:
    ```python
    import pandas as pd
    from flytekit.types.structured import StructuredDataset

    my_df = pd.DataFrame({"col1": [1, 2], "col2": ["a", "b"]})
    sd = StructuredDataset(dataframe=my_df)
    ```
    When this `sd` object is returned from a task, the `dataframe` will be automatically serialized and uploaded to a remote URI.

2.  **From an existing URI**:
    ```python
    from flytekit.types.structured import StructuredDataset

    sd = StructuredDataset(uri="s3://my-bucket/path/to/data.parquet", file_format="parquet")
    ```
    This is useful when you want to reference data that already exists in a remote location.

**Accessing Data from a `StructuredDataset`**

To work with the data contained within a `StructuredDataset` object, use the `open()` method followed by `all()` or `iter()`. The `open()` method specifies the desired local dataframe type (e.g., `pandas.DataFrame`, `pyarrow.Table`) for deserialization.

*   **`open(dataframe_type).all()`**: Downloads and loads the entire structured dataset into memory as the specified `dataframe_type`. This is suitable for smaller datasets that fit into memory.

    ```python
    import pandas as pd
    from flytekit import task
    from flytekit.types.structured import StructuredDataset

    @task
    def process_data(input_sd: StructuredDataset) -> StructuredDataset:
        # Load the entire dataset into a Pandas DataFrame
        df = input_sd.open(pd.DataFrame).all()
        print(f"DataFrame loaded with {len(df)} rows.")
        # Perform operations on df
        processed_df = df * 2 # Example operation
        return StructuredDataset(dataframe=processed_df)
    ```

*   **`open(dataframe_type).iter()`**: Returns an iterator that yields chunks of the structured dataset as the specified `dataframe_type`. This is ideal for large datasets that may not fit entirely into memory, allowing for processing in a streaming fashion.

    ```python
    import pyarrow as pa
    from flytekit import task
    from flytekit.types.structured import StructuredDataset

    @task
    def process_large_data(input_sd: StructuredDataset):
        # Iterate over chunks of the dataset as PyArrow Tables
        for chunk in input_sd.open(pa.Table).iter():
            print(f"Processing chunk with {len(chunk)} rows.")
            # Process each chunk
    ```

### Data Transformation with the `StructuredDatasetTransformerEngine`

The `StructuredDatasetTransformerEngine` is the core component responsible for converting `StructuredDataset` objects between their Python representations (e.g., Pandas DataFrames, PyArrow Tables) and Flyte's internal literal format, which includes managing data persistence to various storage backends. It achieves this through a system of registered **Encoders** and **Decoders**.

**Encoders**

An `StructuredDatasetEncoder` is responsible for converting a Python dataframe object into a Flyte `StructuredDataset` literal, typically involving writing the data to a specified URI in a particular format.

*   **`encode(ctx, structured_dataset, structured_dataset_type)`**: This abstract method must be implemented by concrete encoders. It takes the `FlyteContext` (for file access), the `StructuredDataset` wrapper containing the Python dataframe, and the `StructuredDatasetType` (which includes schema and format information) as input. It returns a Flyte `literals.StructuredDataset` object, which contains the URI where the data was written and its metadata.

**Decoders**

An `StructuredDatasetDecoder` is responsible for converting a Flyte `StructuredDataset` literal (which points to data at a URI) back into a Python dataframe object.

*   **`decode(ctx, flyte_value, current_task_metadata)`**: This abstract method must be implemented by concrete decoders. It takes the `FlyteContext`, the Flyte `literals.StructuredDataset` literal, and `current_task_metadata` (which can specify desired columns for subsetting) as input. It returns either a Python dataframe instance or an iterator of dataframe instances.

**Handler Registration**

The `StructuredDatasetTransformerEngine` maintains a registry of available encoders and decoders. When a `StructuredDataset` needs to be serialized or deserialized, the engine selects the appropriate handler based on:

1.  The Python dataframe type (e.g., `pandas.DataFrame`, `pyarrow.Table`).
2.  The storage protocol (e.g., `"s3"`, `"gs"`, `"bq"`, `"snowflake"`, or `"fsspec"` for generic file system access).
3.  The data format (e.g., `"parquet"`, `"csv"`, or a generic empty string `""` if the handler supports any format).

Handlers are registered using the `StructuredDatasetTransformerEngine.register()` class method. This method allows you to specify if a handler should be the default for a given Python type or format.

### Supported Data Formats and Storage Backends

Flytekit provides built-in handlers for common dataframe types and storage systems:

*   **Pandas DataFrames**:
    *   **Parquet**: `PandasToParquetEncodingHandler` and `ParquetToPandasDecodingHandler` enable reading and writing Pandas DataFrames to/from Parquet files on any supported file system (e.g., S3, GCS, local disk).
    *   **CSV**: `PandasToCSVEncodingHandler` and `CSVToPandasDecodingHandler` enable reading and writing Pandas DataFrames to/from CSV files.
    *   **BigQuery**: `PandasToBQEncodingHandlers` and `BQToPandasDecodingHandler` facilitate direct interaction with BigQuery tables.
    *   **Snowflake**: `PandasToSnowflakeEncodingHandlers` and `SnowflakeToPandasDecodingHandler` enable direct interaction with Snowflake tables.

*   **PyArrow Tables**:
    *   **Parquet**: `ArrowToParquetEncodingHandler` and `ParquetToArrowDecodingHandler` support reading and writing PyArrow Tables to/from Parquet files.
    *   **BigQuery**: `ArrowToBQEncodingHandlers` and `BQToArrowDecodingHandler` support direct interaction with BigQuery tables.

### Common Use Cases and Best Practices

**Type Hinting for Structured Datasets**

You can use `StructuredDataset` directly as a type hint in task signatures. Flytekit automatically handles the conversion.

```python
import pandas as pd
from flytekit import task
from flytekit.types.structured import StructuredDataset

@task
def create_dataframe() -> StructuredDataset:
    df = pd.DataFrame({"a": [1, 2], "b": [3, 4]})
    return StructuredDataset(dataframe=df)

@task
def consume_dataframe(sd: StructuredDataset):
    df = sd.open(pd.DataFrame).all()
    print(df)
```

**Specifying Schema and Format with `Annotated`**

For more control over the expected schema or file format, use `typing.Annotated`.

*   **Column Schema**: Define the expected columns and their types using a dictionary. This enables column subsetting during deserialization.

    ```python
    from typing import Annotated
    from flytekit import task
    from flytekit.types.structured import StructuredDataset
    import pandas as pd

    @task
    def process_specific_columns(
        input_sd: Annotated[StructuredDataset, {"id": int, "name": str}]
    ) -> Annotated[StructuredDataset, {"id": int, "name": str, "age": float}]:
        df = input_sd.open(pd.DataFrame).all()
        # df will only contain 'id' and 'name' columns if available in the source
        df["age"] = df["id"] * 10.0
        return StructuredDataset(dataframe=df)
    ```
    When `process_specific_columns` receives `input_sd`, the `decode` method of the chosen handler will receive the `current_task_metadata` containing the `{ "id": int, "name": str }` schema. The handler is then responsible for subsetting the data to these columns.

*   **File Format**: Specify the desired file format for serialization or deserialization.

    ```python
    from typing import Annotated
    from flytekit import task
    from flytekit.types.structured import StructuredDataset
    import pandas as pd

    @task
    def convert_to_csv(input_sd: StructuredDataset) -> Annotated[StructuredDataset, {}, "csv"]:
        df = input_sd.open(pd.DataFrame).all()
        # The output will be stored as a CSV file
        return StructuredDataset(dataframe=df)
    ```
    The format specified in `Annotated` takes precedence over the `file_format` attribute set on the `StructuredDataset` instance if both are provided.

**Rendering in the UI**

The `to_html` method of the `StructuredDatasetTransformerEngine` allows for rendering structured data in the Flyte UI. If a dataframe is present, it attempts to use a registered renderer for that dataframe type. Otherwise, it renders the column information.

### Extensibility: Custom Handlers

Developers can extend Structured Datasets to support new dataframe types, storage protocols, or file formats by implementing custom encoders and decoders.

1.  **Create an Encoder**: Inherit from `StructuredDatasetEncoder`.
    *   Implement the `__init__` method, specifying the `python_type` (e.g., `my_custom_dataframe.DataFrame`), `protocol` (e.g., `"s3"`, `"my_db_protocol"`), and `supported_format` (e.g., `"jsonl"`).
    *   Implement the `encode` method to write your `structured_dataset.dataframe` to the `uri` provided by the `ctx.file_access` in the specified format.

    ```python
    from flytekit.types.structured import StructuredDatasetEncoder, StructuredDataset, StructuredDatasetType
    from flytekit.core.context_manager import FlyteContext
    from flytekit.models import literals
    import typing

    class MyCustomDataFrameEncoder(StructuredDatasetEncoder):
        def __init__(self):
            super().__init__(python_type=MyCustomDataFrame, protocol="s3", supported_format="jsonl")

        def encode(
            self,
            ctx: FlyteContext,
            structured_dataset: StructuredDataset,
            structured_dataset_type: StructuredDatasetType,
        ) -> literals.StructuredDataset:
            # Assume MyCustomDataFrame has a to_jsonl method
            df = typing.cast(MyCustomDataFrame, structured_dataset.dataframe)
            uri = typing.cast(str, structured_dataset.uri) or ctx.file_access.get_random_remote_directory()
            output_path = ctx.file_access.join(uri, "data.jsonl")
            df.to_jsonl(output_path) # Custom method to write to JSONL
            return literals.StructuredDataset(uri=uri, metadata=literals.StructuredDatasetMetadata(structured_dataset_type))
    ```

2.  **Create a Decoder**: Inherit from `StructuredDatasetDecoder`.
    *   Implement the `__init__` method, specifying the `python_type`, `protocol`, and `supported_format`.
    *   Implement the `decode` method to read data from the `flyte_value.uri` into your `python_type`. Pay attention to `current_task_metadata` for column subsetting.

    ```python
    from flytekit.types.structured import StructuredDatasetDecoder, StructuredDatasetMetadata
    from flytekit.core.context_manager import FlyteContext
    from flytekit.models import literals
    import typing

    class MyCustomDataFrameDecoder(StructuredDatasetDecoder):
        def __init__(self):
            super().__init__(python_type=MyCustomDataFrame, protocol="s3", supported_format="jsonl")

        def decode(
            self,
            ctx: FlyteContext,
            flyte_value: literals.StructuredDataset,
            current_task_metadata: StructuredDatasetMetadata,
        ) -> MyCustomDataFrame:
            # Assume MyCustomDataFrame has a from_jsonl method
            uri = flyte_value.uri
            # Handle column subsetting if current_task_metadata.structured_dataset_type.columns is set
            columns = [c.name for c in current_task_metadata.structured_dataset_type.columns] if current_task_metadata.structured_dataset_type and current_task_metadata.structured_dataset_type.columns else None
            return MyCustomDataFrame.from_jsonl(uri, columns=columns) # Custom method to read from JSONL
    ```

3.  **Register the Handlers**: Use `StructuredDatasetTransformerEngine.register()` to make your custom handlers available to Flytekit.

    ```python
    from flytekit.types.structured import StructuredDatasetTransformerEngine

    StructuredDatasetTransformerEngine.register(MyCustomDataFrameEncoder())
    StructuredDatasetTransformerEngine.register(MyCustomDataFrameDecoder())
    ```

### Important Considerations

*   **Column Subsetting**: When a `StructuredDataset` is passed as an input to a task with a specific column schema defined via `Annotated`, the `StructuredDatasetDecoder` receives this schema in `current_task_metadata`. Decoders should implement logic to subset the incoming data to only the requested columns, optimizing data transfer and memory usage.
*   **Lazy vs. Eager Loading**: The `open().all()` method eagerly loads the entire dataset into memory. For very large datasets, `open().iter()` provides a lazy, iterative approach, which is more memory-efficient.
*   **URI Management**: The `FlyteContext.file_access` object is crucial for managing data paths, especially when dealing with remote storage. It handles the underlying file system operations (e.g., uploading, downloading, joining paths).
*   **`file_format` Precedence**: If a `StructuredDataset` instance is created with a `file_format` (e.g., `StructuredDataset(..., file_format="csv")`) and the task signature also specifies a format via `Annotated` (e.g., `Annotated[StructuredDataset, {}, "parquet"]`), the format specified in the `Annotated` type hint will take precedence during serialization.
*   **Direct URI Initialization**: When a `StructuredDataset` is initialized with only a `uri` (and no `dataframe`), Flytekit treats it as a reference to existing data. No data is uploaded in this case. The `_set_literal` method is used internally to ensure the `StructuredDataset` literal representation is correctly populated for downstream use.
<!--
key: summary_structured_datasets_a3a29d80-8722-43f8-9ef6-468d7fc83d6e
type: summary_end

-->