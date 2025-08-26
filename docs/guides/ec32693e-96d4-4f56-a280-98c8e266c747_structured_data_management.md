
<!--
help_text: ''
key: summary_structured_data_management_b506e566-b1cc-4879-91dd-222ed846ee2c
modules:
- flytekit.core.type_engine.DataclassTransformer
- flytekit.types.schema.types.FlyteSchema
- flytekit.types.schema.types.SchemaEngine
- flytekit.types.schema.types.FlyteSchemaTransformer
- flytekit.types.schema.types.SchemaReader
- flytekit.types.schema.types.SchemaWriter
- flytekit.types.schema.types.LocalIOSchemaReader
- flytekit.types.schema.types.LocalIOSchemaWriter
- flytekit.types.schema.types_pandas.PandasSchemaReader
- flytekit.types.schema.types_pandas.PandasSchemaWriter
- flytekit.types.schema.types_pandas.PandasDataFrameTransformer
- flytekit.types.schema.types_pandas.ParquetIO
- flytekit.types.structured.structured_dataset.StructuredDataset
- flytekit.types.structured.structured_dataset.StructuredDatasetTransformerEngine
- flytekit.types.structured.structured_dataset.StructuredDatasetEncoder
- flytekit.types.structured.structured_dataset.StructuredDatasetDecoder
- flytekit.types.structured.basic_dfs.ArrowToParquetEncodingHandler
- flytekit.types.structured.basic_dfs.PandasToParquetEncodingHandler
- flytekit.types.structured.basic_dfs.ParquetToArrowDecodingHandler
- flytekit.types.structured.basic_dfs.PandasToCSVEncodingHandler
- flytekit.types.structured.basic_dfs.CSVToPandasDecodingHandler
- flytekit.types.structured.basic_dfs.ParquetToPandasDecodingHandler
- flytekit.types.structured.bigquery.PandasToBQEncodingHandlers
- flytekit.types.structured.bigquery.ArrowToBQEncodingHandlers
- flytekit.types.structured.bigquery.BQToArrowDecodingHandler
- flytekit.types.structured.bigquery.BQToPandasDecodingHandler
- flytekit.types.structured.snowflake.SnowflakeToPandasDecodingHandler
- flytekit.types.structured.snowflake.PandasToSnowflakeEncodingHandlers
questions_to_answer: []
type: summary

-->
Structured Data Management

Structured Data Management provides a robust and extensible framework for handling structured datasets, such as dataframes, across various storage systems and data formats. This capability is built around the `StructuredDataset` abstraction, which facilitates seamless data exchange within workflows.

The `FlyteSchema` class is deprecated. All new implementations and existing codebases should migrate to using `StructuredDataset` for managing structured data.

### The `StructuredDataset` Abstraction

The `StructuredDataset` class is the primary user-facing interface for representing structured data. It acts as a wrapper around underlying Python dataframe objects (e.g., `pandas.DataFrame`, `pyarrow.Table`) and manages their remote storage location, format, and associated metadata.

A `StructuredDataset` instance typically holds:
*   `uri`: The Uniform Resource Identifier specifying the location of the data in a remote storage system (e.g., `s3://`, `gs://`, `bq://`, `snowflake://`).
*   `file_format`: The format of the stored data (e.g., `parquet`, `csv`).
*   `dataframe`: An optional reference to the in-memory Python dataframe object.

Data within a `StructuredDataset` is loaded lazily. The actual data is not materialized into memory until explicitly requested, which is crucial for handling large datasets efficiently.

**Key Methods:**

*   `open(dataframe_type)`: Prepares the `StructuredDataset` for reading by specifying the desired Python dataframe type (e.g., `pandas.DataFrame`, `pyarrow.Table`). This method returns the `StructuredDataset` instance itself, allowing for method chaining.
*   `all()`: Loads the entire dataset into memory as the `dataframe_type` specified in `open()`. This is suitable for smaller datasets that fit within available memory.
*   `iter()`: Provides an iterator over chunks of the dataset, allowing for processing large datasets without loading the entire data into memory. This is the recommended approach for memory-intensive operations.

**Example Usage:**

```python
import pandas as pd
from flytekit.types.structured import StructuredDataset
from flytekit import task, workflow
from flytekit.types.structured.structured import kwtypes

@task
def create_dataframe_task() -> StructuredDataset:
    df = pd.DataFrame({"col_a": [1, 2, 3], "col_b": ["x", "y", "z"]})
    # When returning a StructuredDataset, you can optionally specify the URI and format.
    # If not specified, Flyte will determine a default remote path and format (e.g., Parquet).
    return StructuredDataset(dataframe=df)

@task
def process_dataframe_task(input_sd: StructuredDataset) -> StructuredDataset:
    # Open the StructuredDataset as a pandas DataFrame
    df = input_sd.open(pd.DataFrame).all()
    print(f"Processing DataFrame with columns: {df.columns.tolist()}")
    df["col_c"] = df["col_a"] * 2
    return StructuredDataset(dataframe=df)

@task
def analyze_dataframe_task(input_sd: StructuredDataset):
    # For very large datasets, iterate over chunks
    for chunk in input_sd.open(pd.DataFrame).iter():
        print(f"Analyzing chunk with {len(chunk)} rows.")
        # Perform analysis on chunk

@workflow
def my_data_workflow():
    sd1 = create_dataframe_task()
    sd2 = process_dataframe_task(input_sd=sd1)
    analyze_dataframe_task(input_sd=sd2)
```

### Data Transformation Engine

The `StructuredDatasetTransformerEngine` is the central component responsible for managing the conversion between Python dataframe objects and Flyte's internal `Literal` representation of structured datasets. It acts as a registry for various handlers that support different dataframe types, storage protocols, and data formats.

This engine leverages two key abstract classes for extensibility:

*   **`StructuredDatasetEncoder`**: Implementations of this class define how a Python dataframe object (e.g., `pandas.DataFrame`) is converted into a `literals.StructuredDataset` (Flyte's internal representation). This typically involves writing the dataframe to a specified remote storage URI in a particular format.
    *   Example: `PandasToParquetEncodingHandler` converts a `pandas.DataFrame` to a Parquet file.
*   **`StructuredDatasetDecoder`**: Implementations of this class define how a `literals.StructuredDataset` is converted back into a Python dataframe object. This involves reading data from the remote storage URI and materializing it into the desired dataframe type.
    *   Example: `ParquetToPandasDecodingHandler` reads a Parquet file and converts it into a `pandas.DataFrame`.

The engine dynamically selects the appropriate encoder or decoder based on the Python type, the storage protocol (e.g., `s3`, `gs`, `bigquery`, `snowflake`), and the data format (e.g., `parquet`, `csv`).

### Extending Structured Data Support

Developers can extend Structured Data Management to support new dataframe types or integrate with additional data sources by implementing and registering custom `StructuredDatasetEncoder` and `StructuredDatasetDecoder` handlers.

To register a new handler, use the `StructuredDatasetTransformerEngine.register()` class method. This method allows specifying the Python dataframe type, the supported storage protocol, and the data format.

```python
from flytekit.types.structured import StructuredDatasetTransformerEngine, StructuredDatasetEncoder, StructuredDatasetDecoder
from flytekit.models import literals
import typing
from abc import ABC, abstractmethod
from flytekit.core.context_manager import FlyteContext
from flytekit.models.types import StructuredDatasetType

# Assume a hypothetical custom dataframe library and its type
class CustomDataFrame:
    def __init__(self, data):
        self.data = data
    def to_csv(self, path, **kwargs):
        # Simulate writing to CSV
        print(f"Writing CustomDataFrame to {path}")
    @classmethod
    def from_csv(cls, path, **kwargs):
        # Simulate reading from CSV
        print(f"Reading CustomDataFrame from {path}")
        return cls({"col1": [10, 20]})

# Example: A custom encoder for a hypothetical CustomDataFrame to CSV
class CustomDataFrameToCSVEncoder(StructuredDatasetEncoder[CustomDataFrame]):
    def __init__(self):
        super().__init__(CustomDataFrame, protocol=None, supported_format="csv")

    def encode(
        self,
        ctx: FlyteContext,
        structured_dataset: StructuredDataset,
        structured_dataset_type: StructuredDatasetType,
    ) -> literals.StructuredDataset:
        uri = typing.cast(str, structured_dataset.uri) or ctx.file_access.get_random_remote_directory()
        # In a real scenario, you'd use ctx.file_access to manage remote paths
        path = f"{uri}/data.csv" # Simplified path for example
        typing.cast(CustomDataFrame, structured_dataset.dataframe).to_csv(path)
        return literals.StructuredDataset(uri=uri, metadata=literals.StructuredDatasetMetadata(structured_dataset_type))

# Example: A custom decoder for CSV to a hypothetical CustomDataFrame
class CSVToCustomDataFrameDecoder(StructuredDatasetDecoder[CustomDataFrame]):
    def __init__(self):
        super().__init__(CustomDataFrame, protocol=None, supported_format="csv")

    def decode(
        self,
        ctx: FlyteContext,
        flyte_value: literals.StructuredDataset,
        current_task_metadata: literals.StructuredDatasetMetadata,
    ) -> CustomDataFrame:
        uri = flyte_value.uri
        # In a real scenario, you'd use ctx.file_access to manage remote paths
        path = f"{uri}/data.csv" # Simplified path for example
        return CustomDataFrame.from_csv(path)

# Register the custom handlers (typically done in a plugin or initialization code)
# Note: In a real Flytekit environment, these registrations would happen automatically
# if your custom handlers are part of a discoverable plugin.
# For demonstration, we explicitly register them here.
StructuredDatasetTransformerEngine.register(CustomDataFrameToCSVEncoder())
StructuredDatasetTransformerEngine.register(CSVToCustomDataFrameDecoder())
```

### Working with Specific Data Sources

Structured Data Management provides built-in handlers for common data sources and formats:

*   **File-based Storage (S3, GCS, Local)**:
    *   `parquet` format: Handlers like `PandasToParquetEncodingHandler`, `ParquetToPandasDecodingHandler`, `ArrowToParquetEncodingHandler`, and `ParquetToArrowDecodingHandler` enable reading and writing `pandas.DataFrame` and `pyarrow.Table` objects to/from Parquet files.
    *   `csv` format: `PandasToCSVEncodingHandler` and `CSVToPandasDecodingHandler` support `pandas.DataFrame` with CSV files.
    These handlers typically operate on paths that are automatically managed by the underlying file access layer, supporting various protocols (e.g., `s3://`, `gs://`, `file://`).

*   **BigQuery**:
    *   `PandasToBQEncodingHandlers`, `ArrowToBQEncodingHandlers`, `BQToPandasDecodingHandler`, and `BQToArrowDecodingHandler` facilitate direct interaction with BigQuery tables. When using BigQuery, the `uri` for `StructuredDataset` should follow the format `bq://<project_id>.<dataset_id>.<table_id>`.
    *   Example: `StructuredDataset(uri="bq://my-gcp-project.my_dataset.my_table")`

*   **Snowflake**:
    *   `PandasToSnowflakeEncodingHandlers` and `SnowflakeToPandasDecodingHandler` enable reading and writing `pandas.DataFrame` objects to/from Snowflake tables. The `uri` for Snowflake follows a specific connection string format.
    *   Example: `StructuredDataset(uri="snowflake://<account_identifier>/<database>/<schema>/<table>")`

### Advanced Usage and Considerations

#### Column Subsetting

When defining `StructuredDataset` as a task input or output, you can specify expected column names and their types using `kwtypes` (keyword types). This enables automatic column subsetting during deserialization. If a task expects a subset of columns from an incoming `StructuredDataset`, the decoder will attempt to load only those specified columns, optimizing memory usage and data transfer.

```python
from flytekit.types.structured.structured import kwtypes
import pandas as pd
from flytekit import task, workflow
from flytekit.types.structured import StructuredDataset

@task
def producer_task() -> StructuredDataset:
    # Returns a dataset with 'col_a' (int), 'col_b' (str), 'col_c' (float)
    df = pd.DataFrame({"col_a": [1, 2], "col_b": ["x", "y"], "col_c": [1.1, 2.2]})
    return StructuredDataset(dataframe=df)

@task
def consumer_task(input_sd: StructuredDataset[kwtypes(col_b=str, col_c=float)]):
    # Only 'col_b' and 'col_c' will be loaded into the DataFrame
    df = input_sd.open(pd.DataFrame).all()
    print(f"Consumer DataFrame columns: {df.columns.tolist()}") # Output: ['col_b', 'col_c']

@workflow
def subsetting_workflow():
    sd_full = producer_task()
    consumer_task(input_sd=sd_full)
```

#### Performance Considerations

For very large datasets, always prefer using the `iter()` method when reading data from a `StructuredDataset`. This approach processes data in chunks, preventing out-of-memory errors and allowing for scalable data processing. The `all()` method should be reserved for datasets known to fit comfortably within the available memory.

When writing data, the choice of format (e.g., Parquet vs. CSV) can significantly impact performance and storage efficiency. Parquet is generally recommended for its columnar storage, compression, and schema evolution capabilities.

#### Deprecation of `FlyteSchema`

As noted, `FlyteSchema` is deprecated. While the system maintains backward compatibility, new code should exclusively use `StructuredDataset`. The `StructuredDataset` offers a more flexible and extensible design, supporting a wider range of dataframe types, storage protocols, and data formats, and providing a clearer separation of concerns for data handling.
<!--
key: summary_structured_data_management_b506e566-b1cc-4879-91dd-222ed846ee2c
type: summary_end

-->
<!--
code_unit: flytekit.core.type_engine.DataclassTransformer
code_unit_type: class
help_text: ''
key: example_eda89688-e65a-49f7-b97a-5edec337a20b
type: example

-->
<!--
code_unit: flytekit.types.structured.structured_dataset.StructuredDataset
code_unit_type: class
help_text: ''
key: example_1b9001db-1eb0-4c3c-8625-ef2643f5321f
type: example

-->
<!--
code_unit: flytekit.types.schema.types_pandas.PandasDataFrameTransformer
code_unit_type: class
help_text: ''
key: example_78ef872f-67da-4d85-b9da-6e334e4a652f
type: example

-->