
<!--
help_text: ''
key: summary_type_system_&_data_management_e87f9b4e-e6e3-43d0-bd3c-e335b4ddc3b4
modules:
- flytekit.core.type_engine
- flytekit.core.data_persistence
- flytekit.types.directory.types
- flytekit.types.file.file
- flytekit.types.file.image
- flytekit.types.schema.types
- flytekit.types.schema.types_pandas
- flytekit.types.structured.structured_dataset
- flytekit.types.structured.basic_dfs
- flytekit.types.structured.bigquery
- flytekit.types.structured.snowflake
- flytekit.types.numpy.ndarray
- flytekit.types.pickle.pickle
- flytekit.types.error.error
- flytekit.types.iterator.iterator
- flytekit.types.iterator.json_iterator
- flytekit.types.annotation
- flytekit.core.artifact
questions_to_answer: []
type: summary

-->
## Type System & Data Management

The type system and data management capabilities provide a robust and extensible framework for handling data within the platform. This system ensures seamless conversion between Python native types and the platform's internal literal representations, manages data persistence across various storage backends, and enables advanced data lineage and querying.

### Type System Core

The type system's foundation is built around the concept of **Type Transformers**, which define how Python types are converted to and from the platform's internal `LiteralType` and `Literal` representations. The **Type Engine** acts as the central registry and orchestrator for these transformations.

#### Type Transformation Mechanism

The `TypeTransformer` is the base class for defining type conversions. It provides abstract methods that must be implemented for each Python type supported by the platform:

*   `get_literal_type(python_type)`: Converts a Python type hint (e.g., `int`, `str`, `List[int]`) into its corresponding platform `LiteralType` (e.g., `SimpleType.INTEGER`, `CollectionType` of `SimpleType.INTEGER`).
*   `to_literal(ctx, python_val, python_type, expected)`: Transforms a Python value into a platform `Literal`. This is crucial for serializing data from user code into a format consumable by the platform's execution engine.
*   `to_python_value(ctx, lv, expected_python_type)`: Converts a platform `Literal` back into a Python value. This is used when deserializing inputs to tasks or outputs from upstream tasks.
*   `assert_type(t, v)`: Performs runtime type validation, ensuring that a given Python value `v` conforms to the expected Python type `t`.
*   `from_binary_idl(binary_idl_object, expected_python_type)` and `from_generic_idl(generic, expected_python_type)`: Handle deserialization from binary and generic intermediate representations, primarily used for structured data types like dataclasses and dictionaries.

For asynchronous operations, `AsyncTypeTransformer` extends `TypeTransformer`, providing `async_to_literal` and `async_to_python_value` methods. The `TypeEngine` automatically handles the synchronous/asynchronous dispatch.

The `TypeEngine` manages a registry of `TypeTransformer` instances. When a type conversion is requested, it looks up the appropriate transformer. It also supports a recursive search through the Method Resolution Order (MRO) for class hierarchies and handles generic types like `List[T]` and `Dict[K, V]`.

**Key Integration Points:**
*   **Custom Type Registration:** Developers can register their own `TypeTransformer` implementations using `TypeEngine.register()` to enable custom Python types to be used as task inputs/outputs.
*   **Type Assertions:** The `type_assertions_enabled` property on `TypeTransformer` allows transformers to control whether the `TypeEngine` performs type validation before conversion.
*   **`Annotated` Types:** The `TypeEngine` processes `typing.Annotated` types, allowing additional metadata (like `BatchSize` or custom `FlyteAnnotation`s) to be attached to type hints, influencing serialization or runtime behavior.

#### Built-in Type Handling

The type system provides transformers for a wide range of standard Python types:

*   **Primitives and Collections:**
    *   `SimpleTransformer`: Handles basic types like `int`, `float`, `str`, `bool`, `datetime.datetime`, `datetime.timedelta`.
    *   `ListTransformer`: Supports `typing.List[T]`, converting to/from `LiteralCollection`. Only univariate lists are supported.
    *   `DictTransformer`: Handles `dict` types. Typed dictionaries (`Dict[str, T]`) are converted to `LiteralMap`, while untyped dictionaries are serialized to a binary scalar literal using MessagePack.
    *   `UnionTransformer`: Manages `typing.Union` types, including `Optional[T]`. It attempts to convert values to the first matching subtype in the union.
*   **Structured Python Objects:**
    *   `DataclassTransformer`: Provides automatic serialization and deserialization for Python dataclasses. It leverages the `mashumaro` library to convert dataclasses to/from MessagePack bytes, which are then stored as binary literals. It also attempts to extract JSON Schema for dataclasses, which can be useful for UI/CLI introspection.
    *   `ProtobufTransformer`: Handles Google Protobuf `Message` objects, converting them to/from `Struct` literals.
    *   `EnumTransformer`: Supports Python `enum.Enum` types, converting them to `EnumType` literals. Only string-valued enums are currently supported.
*   **I/O Streams:**
    *   `TextIOTransformer`: Handles `typing.TextIO` for text file streams.
    *   `BinaryIOTransformer`: Handles `typing.BinaryIO` for binary file streams.
*   **Specialized Types:**
    *   `RestrictedTypeTransformer`: Used to mark types that are not allowed to be converted to/from literals, preventing their use as task inputs/outputs.
    *   `LiteralsResolver`: A utility for converting `LiteralMap` objects (often results from remote executions) back into Python native values, allowing type hints to guide the deserialization.

### Data Persistence and File Access

The `FileAccessProvider` is the primary utility for interacting with local and remote durable storage. It abstracts away the underlying file system details, providing a consistent interface for data operations.

**Key Capabilities:**

*   **Unified File System Access:** Integrates with `fsspec` to support various protocols like `s3://`, `gs://`, `file://`, `http(s)://`, and others configured via `fsspec` plugins.
*   **Data Transfer:**
    *   `get_data(remote_path, local_path, is_multipart)`: Downloads data from a remote URI to a local path. `is_multipart=True` is used for directories.
    *   `put_data(local_path, remote_path, is_multipart)`: Uploads data from a local path to a remote URI.
    *   `async_put_raw_data(lpath, upload_prefix, file_name, ...)`: A more flexible upload method that accepts file-like objects or paths, writing to the raw output prefix.
*   **Path Management:**
    *   `get_random_local_path()`: Generates a unique temporary local file path within the sandbox directory.
    *   `get_random_remote_path()`: Generates a unique remote URI within the configured raw output prefix.
    *   `join(*args, fs)`: Joins path components using the correct separator for the specified file system.
    *   `generate_new_custom_path(fs, alt, stem)`: Creates new remote paths with custom prefixes or stems.
*   **Execution Context:** The `FileAccessProvider` is accessible via `FlyteContext.current_context().file_access`, ensuring operations are performed within the correct execution environment (e.g., local sandbox, remote output prefix).

**Performance Considerations:**
*   The `FileAccessProvider` leverages `fsspec` for efficient data transfer.
*   Asynchronous methods (`async_get_data`, `async_put_data`) are provided for non-blocking I/O operations, which are crucial in high-throughput scenarios.

### Blob Storage Types: Files and Directories

The platform provides dedicated types for handling single files and directories, abstracting away the complexities of remote storage.

#### Files (`FlyteFile`)

The `FlyteFile` type represents a single file. It implements `os.PathLike`, allowing it to be used directly with file system operations like `open()`.

**Key Features:**

*   **Lazy Downloading:** When a `FlyteFile` object is created from a remote URI (e.g., an input to a task), the actual file content is not downloaded until it's accessed (e.g., via `with open(my_file, "r")`). This optimizes resource usage.
*   **Remote Path Handling:** If a `FlyteFile` is initialized with a remote URI, it will not be re-uploaded to the platform's blob store. Instead, the URI is passed directly.
*   **Local Path Upload:** If initialized with a local path, the file's contents are automatically uploaded to the configured remote blob store when the task completes.
*   **Format Specification:** `FlyteFile["csv"]` allows specifying the file format, which is stored as metadata in the `BlobType`.
*   **Path Creation:**
    *   `FlyteFile.new_remote_file(name, alt)`: Creates a `FlyteFile` object pointing to a new, unique remote location.
    *   `FlyteFile.new(filename)`: Creates a `FlyteFile` object pointing to a new local path within the task's working directory.
    *   `FlyteFile.from_source(source)`: Creates a `FlyteFile` from an existing URI (local or remote).
*   **Streaming Access:** The `open(mode, cache_type, cache_options)` method provides a context manager for streaming file access, supporting various `fsspec` caching options for large files.

The `FlyteFilePathTransformer` handles the conversion between `FlyteFile` (or `os.PathLike`) and the platform's `Literal` representation (a `Single` `Blob`). It includes validation for file types using `libmagic` if installed.

#### Directories (`FlyteDirectory`)

The `FlyteDirectory` type represents a directory. Similar to `FlyteFile`, it implements `os.PathLike` for direct use with directory operations.

**Key Features:**

*   **Lazy Downloading:** Directories are downloaded only when their contents are accessed.
*   **Remote Path Handling:** If initialized with a remote URI, the URI is passed directly without re-uploading.
*   **Local Path Upload:** If initialized with a local path, the directory's contents are automatically uploaded as a `Multipart` `Blob` when the task completes.
*   **Format Specification:** `FlyteDirectory["svg"]` allows specifying the directory format.
*   **Path Creation:**
    *   `FlyteDirectory.new_remote(stem, alt)`: Creates a `FlyteDirectory` object pointing to a new, unique remote location.
    *   `FlyteDirectory.new(dirname)`: Creates a `FlyteDirectory` object pointing to a new local path within the task's working directory.
    *   `FlyteDirectory.new_file(name)`: Creates a new `FlyteFile` object within the current directory.
    *   `FlyteDirectory.new_dir(name)`: Creates a new `FlyteDirectory` object within the current directory.
    *   `FlyteDirectory.from_source(source)`: Creates a `FlyteDirectory` from an existing URI.
*   **Content Listing and Crawling:**
    *   `listdir(directory)`: Lists files and folders within a `FlyteDirectory` *without* downloading their contents, returning `FlyteFile` and `FlyteDirectory` objects that support lazy downloading.
    *   `crawl(maxdepth, topdown)`: Provides a generator to traverse the directory structure, optionally returning detailed file information.
*   **Batching for Large Directories:** The `BatchSize` annotation (`Annotated[FlyteDirectory, BatchSize(10)]`) allows specifying the number of files to download/upload in chunks, improving memory efficiency for very large directories.

The `FlyteDirToMultipartBlobTransformer` handles the conversion between `FlyteDirectory` and the platform's `Literal` representation (a `Multipart` `Blob`).

### Structured Datasets

For tabular data, the `StructuredDataset` type provides a flexible and extensible mechanism, replacing the deprecated `FlyteSchema`. It allows seamless integration with various dataframe libraries and storage formats.

#### `StructuredDataset` Overview

The `StructuredDataset` class is the user-facing type for structured data. It acts as a wrapper around actual dataframe objects (e.g., `pandas.DataFrame`, `pyarrow.Table`) and manages their persistence.

**Key Features:**

*   **Backend Agnostic:** Supports different dataframe libraries through a pluggable encoder/decoder system.
*   **Format Flexibility:** Allows specifying the storage format (e.g., Parquet, CSV).
*   **Column Schema:** Can carry column type information, enabling schema enforcement and projection.
*   **Usage Patterns:**
    *   `sd.open(pd.DataFrame).all()`: Reads the entire structured dataset into a Pandas DataFrame.
    *   `sd.open(pd.DataFrame).iter()`: Provides an iterator over chunks of the structured dataset, useful for large datasets that don't fit in memory.

#### Extensibility with Encoders and Decoders

The `StructuredDatasetTransformerEngine` is the central component for managing structured dataset conversions. It uses `StructuredDatasetEncoder` and `StructuredDatasetDecoder` to handle specific dataframe types and storage protocols.

*   **`StructuredDatasetEncoder`:** Abstract base class for converting a Python dataframe instance into a platform `StructuredDataset` literal. Implementations define how a specific dataframe type (e.g., `pandas.DataFrame`) is written to a given storage protocol (e.g., `s3`) and format (e.g., `parquet`).
*   **`StructuredDatasetDecoder`:** Abstract base class for converting a platform `StructuredDataset` literal into a Python dataframe instance. Implementations define how to read data from a given storage protocol and format into a specific dataframe type.

**Registration:** Custom encoders and decoders are registered with `StructuredDatasetTransformerEngine.register()`. This allows developers to extend support for new dataframe libraries or custom storage formats. The registration process allows specifying default protocols and formats for a given Python dataframe type.

**Column Subsetting:** When a `StructuredDataset` is passed between tasks, the `StructuredDatasetTransformerEngine` intelligently handles column subsetting. If the downstream task's input type specifies a subset of columns, the decoder will attempt to read only those columns, optimizing data transfer and memory usage.

#### Built-in Structured Dataset Handlers

The platform provides out-of-the-box support for common dataframe libraries and storage backends:

*   **Pandas DataFrames:**
    *   `PandasToParquetEncodingHandler`: Encodes `pandas.DataFrame` to Parquet format.
    *   `ParquetToPandasDecodingHandler`: Decodes Parquet to `pandas.DataFrame`.
    *   `PandasToCSVEncodingHandler`: Encodes `pandas.DataFrame` to CSV format.
    *   `CSVToPandasDecodingHandler`: Decodes CSV to `pandas.DataFrame`.
*   **PyArrow Tables:**
    *   `ArrowToParquetEncodingHandler`: Encodes `pyarrow.Table` to Parquet format.
    *   `ParquetToArrowDecodingHandler`: Decodes Parquet to `pyarrow.Table`.
*   **BigQuery Integration:**
    *   `PandasToBQEncodingHandlers`, `ArrowToBQEncodingHandlers`: Encode Pandas DataFrames or PyArrow Tables to BigQuery tables.
    *   `BQToPandasDecodingHandler`, `BQToArrowDecodingHandler`: Decode BigQuery tables to Pandas DataFrames or PyArrow Tables.
*   **Snowflake Integration:**
    *   `PandasToSnowflakeEncodingHandlers`: Encodes Pandas DataFrames to Snowflake tables.
    *   `SnowflakeToPandasDecodingHandler`: Decodes Snowflake tables to Pandas DataFrames.

### Other Specialized Data Types

Beyond general-purpose types and structured datasets, the platform supports several specialized data types:

*   **NumPy Arrays (`numpy.ndarray`)**: The `NumpyArrayTransformer` handles `numpy.ndarray` objects, serializing them to `.npy` files stored as single blobs. It supports `allow_pickle` and `mmap_mode` metadata for advanced usage.
*   **PIL Images (`PIL.Image.Image`)**: The `PILImageTransformer` enables using `PIL.Image.Image` objects directly. Images are saved as PNG files and stored as single blobs. It also provides an HTML renderer for visualization in the UI.
*   **JSON Iterators (`Iterator[JSON]`)**: The `JSONIteratorTransformer` and `JSONIterator` provide an efficient way to handle large streams of JSON objects. It serializes iterators to JSONL (JSON Lines) files stored as single blobs, allowing for memory-efficient processing of large datasets.
*   **Error Type (`FlyteError`)**: The `ErrorTransformer` handles the `FlyteError` type, which is used to represent structured error information (message, failed node ID) within the platform's execution model, particularly for failure handling.
*   **Pickle Fallback (`FlytePickle`)**: The `FlytePickleTransformer` acts as a fallback mechanism. If no specific transformer is found for a given Python type, the object is serialized using `cloudpickle` and stored as a single blob. This ensures that arbitrary Python objects can be passed, though it's generally less efficient and less portable than explicit type handling.

### Data Lineage and Artifacts

The `Artifact` system provides a powerful way to define, track, and query data assets produced by tasks and workflows, enabling robust data lineage.

#### Defining Artifacts (`Artifact`)

The `Artifact` class allows users to declare named, versioned, and optionally partitioned data outputs. This metadata is attached to the data produced by tasks, making it discoverable and traceable.

**Key Attributes:**

*   `name`: A user-defined name for the artifact.
*   `project`, `domain`: Automatically populated from the execution context, but can be specified for querying.
*   `version`: Typically the execution ID, automatically assigned.
*   `time_partitioned`: Boolean indicating if the artifact is time-partitioned.
*   `time_partition_granularity`: Defines the granularity (e.g., `DAY`, `HOUR`) for time partitions.
*   `partition_keys`: A list of string keys for custom partitions (e.g., `["region", "country"]`).

**Usage Patterns:**

*   **Compile-time Binding:** Use `Annotated[Type, Artifact(name="my.artifact")]` in task output signatures. Partitions can be bound at compile time using `Artifact(name="my.artifact")(region=Inputs.region)`. `ArtifactIDSpecification` is an internal helper for this.
*   **Runtime Binding (`create_from`)**: The `create_from(obj, **kwargs)` method allows dynamically setting partition values from within a task's execution. This is useful when partition values are determined at runtime.

    ```python
    from flytekit.core.artifact import Artifact
    from flytekit import task, workflow
    from typing import Annotated
    import pandas as pd
    import datetime

    # Define an artifact with a time partition and a custom partition key
    DailySales = Artifact(name="daily_sales", time_partitioned=True, partition_keys=["region"])

    @task
    def generate_sales_report(region: str) -> Annotated[pd.DataFrame, DailySales]:
        # Simulate generating a sales report
        df = pd.DataFrame({"date": [datetime.date.today()], "sales": [100], "region": [region]})
        
        # Attach artifact metadata at runtime
        return DailySales.create_from(df, region=region, time_partition=datetime.datetime.now())

    @workflow
    def sales_wf(region: str) -> Annotated[pd.DataFrame, DailySales]:
        return generate_sales_report(region=region)
    ```

#### Querying Artifacts (`ArtifactQuery`)

The `ArtifactQuery` class is used to retrieve previously produced artifacts. It allows specifying criteria based on artifact name, project, domain, and partition values.

**Key Features:**

*   **Flexible Querying:** Query by artifact name, project, domain, specific time partitions (`time_partition`), or custom partition key-value pairs (`partitions`).
*   **Dynamic Binding:** Partitions can be bound to task inputs using `Inputs.variable_name`, allowing queries to adapt to workflow inputs.
*   **Dependency Tracking:** The `ArtifactQuery` tracks dependencies on other artifacts if their partitions are used in the query.

**Example:**

```python
from flytekit.core.artifact import Artifact, Inputs
from flytekit import task, workflow
from typing import Annotated
import pandas as pd
import datetime

# Assume DailySales artifact is defined as above

@task
def analyze_sales(sales_data: Annotated[pd.DataFrame, DailySales.query(region=Inputs.region)]) -> float:
    # sales_data will be the DataFrame for the specified region and time partition
    return sales_data["sales"].sum()

@workflow
def analyze_sales_wf(region: str, report_date: datetime.datetime) -> float:
    # Query the DailySales artifact for a specific region and date
    # The 'report_date' input will be used to bind the time_partition
    return analyze_sales(sales_data=DailySales.query(region=region, time_partition=report_date))
```

#### Partitioning Concepts

*   **`Partitions`**: Represents a collection of custom partition key-value pairs.
*   **`Partition`**: A single custom partition, holding a name and a value (static string or input binding).
*   **`TimePartition`**: Represents a time-based partition, supporting `datetime.datetime` values or input bindings. It also allows for time transformations (`__add__`, `__sub__`) for relative time queries.
*   **`InputsBase`**: A helper object (`Inputs`) used to bind partition values to task inputs, enabling dynamic queries.

The `Serializer` and `ArtifactSerializationHandler` are internal components responsible for converting these Python artifact objects into their corresponding platform IDL representations for storage and communication.
<!--
key: summary_type_system_&_data_management_e87f9b4e-e6e3-43d0-bd3c-e335b4ddc3b4
type: summary_end

-->
<!--
code_unit: flytekit.types.file.file
code_unit_type: class
help_text: ''
key: example_51fa0412-48c8-4893-ad1d-beb0e05774b3
type: example

-->
<!--
code_unit: flytekit.types.directory.types
code_unit_type: class
help_text: ''
key: example_cef116e8-228b-4084-b2fc-adc3c692b47b
type: example

-->
<!--
code_unit: flytekit.types.structured.structured_dataset
code_unit_type: class
help_text: ''
key: example_a642d98d-4121-4e95-8c39-81c37b25f1ae
type: example

-->