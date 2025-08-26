
<!--
help_text: ''
key: summary_type_system_&_data_management_12190404-f0d2-4bc3-86f0-f605a26e5e23
modules:
- flytekit.core.type_engine
- flytekit.types.directory.types
- flytekit.types.error.error
- flytekit.types.file.file
- flytekit.types.file.image
- flytekit.types.iterator.iterator
- flytekit.types.iterator.json_iterator
- flytekit.types.numpy.ndarray
- flytekit.types.pickle.pickle
- flytekit.types.schema.types
- flytekit.types.schema.types_pandas
- flytekit.types.structured.basic_dfs
- flytekit.types.structured.bigquery
- flytekit.types.structured.snowflake
- flytekit.types.structured.structured_dataset
- flytekit.core.data_persistence
- flytekit.models.literals
- flytekit.models.types
questions_to_answer: []
type: summary

-->
## Type System & Data Management

Flytekit's type system provides a robust mechanism for seamlessly converting Python native types to Flyte's internal Literal types and vice-versa. This enables tasks and workflows to operate on familiar Python objects while Flyte handles the underlying data serialization, deserialization, and persistence.

### Core Type Conversion with TypeEngine

The central component for type conversion is the `TypeEngine`. It acts as a registry for `TypeTransformer` instances, which are responsible for defining how specific Python types map to Flyte Literal types and how values are converted between these representations.

`TypeEngine` offers the following key capabilities:

*   **`register`**: Adds a `TypeTransformer` for a given Python type, allowing Flytekit to recognize and handle it.
*   **`get_transformer`**: Retrieves the appropriate `TypeTransformer` for a specified Python type, supporting inheritance and annotated types.
*   **`to_literal_type`**: Converts a Python type hint (e.g., `int`, `typing.List[str]`) into its corresponding Flyte `LiteralType` definition. This is crucial for defining task and workflow interfaces.
*   **`to_literal` / `async_to_literal`**: Transforms a Python value into a Flyte `Literal` object. This is used when passing data from a task's output to Flyte's execution engine.
*   **`to_python_value` / `async_to_python_value`**: Converts a Flyte `Literal` object back into a Python native value. This occurs when a task receives inputs from the Flyte engine.
*   **`guess_python_type`**: Infers the Python type from a Flyte `LiteralType`, useful for dynamic type resolution, especially in remote execution contexts.
*   **`literal_map_to_kwargs` / `_literal_map_to_kwargs`**: Converts a `LiteralMap` (Flyte's representation of a dictionary of inputs/outputs) into Python keyword arguments for task execution.
*   **`dict_to_literal_map` / `_dict_to_literal_map`**: Converts a Python dictionary into a `LiteralMap`.

When a conversion fails, a `TypeTransformerFailedError` is raised, indicating a mismatch between the expected and actual types or values. For types explicitly marked as unsupported for literal conversion, a `RestrictedTypeError` is raised.

### Data Persistence with FileAccessProvider

Data management, especially for large files and directories, is handled by the `FileAccessProvider`. This utility provides an abstraction over various storage backends (e.g., S3, GCS, local filesystem) and manages the transfer of data between the local sandbox and remote durable storage.

`FileAccessProvider` methods include:

*   **`local_sandbox_dir`**: Provides a temporary local directory for data operations.
*   **`raw_output_prefix`**: The base URI for storing task outputs in remote storage.
*   **`get_filesystem` / `get_filesystem_for_path`**: Retrieves an `fsspec` filesystem object for a given protocol (e.g., "s3", "file").
*   **`is_remote`**: Checks if a given path points to a remote storage location.
*   **`get_data` / `async_get_data`**: Downloads data from a remote URI to a local path.
*   **`put_data` / `async_put_data`**: Uploads data from a local path to a remote URI.
*   **`put_raw_data` / `async_put_raw_data`**: A flexible method for uploading file-like objects or paths to the raw output prefix.
*   **`get_random_local_path` / `get_random_local_directory`**: Generates unique paths within the local sandbox.
*   **`get_random_remote_path` / `get_random_remote_directory`**: Generates unique paths in the remote storage.

Transformers for file and directory types heavily rely on `FileAccessProvider` to manage the actual data movement, abstracting away the complexities of cloud storage.

### Built-in Type Transformers

Flytekit provides a comprehensive set of built-in `TypeTransformer` implementations for common Python types:

#### Primitive Types

The `SimpleTransformer` handles basic Python primitives like `int`, `float`, `str`, `bool`, `datetime.datetime`, and `datetime.timedelta`. It converts these directly to their corresponding Flyte `Primitive` literals.

#### Collection Types

*   **`ListTransformer`**: Supports `typing.List[T]`. It converts Python lists to Flyte `LiteralCollection` objects, recursively transforming each element. Only univariate lists (lists of a single type) are supported.
*   **`DictTransformer`**: Handles `typing.Dict[str, T]`.
    *   For typed dictionaries (e.g., `Dict[str, int]`), it converts them to Flyte `LiteralMap` objects, where keys are strings and values are recursively transformed.
    *   For untyped dictionaries (`dict`), it serializes them to binary literals using MessagePack, or falls back to Pickle if allowed. This is represented as a `SimpleType.STRUCT` in Flyte.
*   **`UnionTransformer`**: Manages `typing.Union[T1, T2, ...]`. It attempts to convert a Python value to each variant in the union until a successful conversion is found. It handles `Optional[T]` as a special case of `Union[T, None]`. Ambiguous conversions (where a value could match multiple union variants) result in a `TypeError`.
*   **`IteratorTransformer`**: Converts `typing.Iterator[T]` to a `LiteralCollection`. The `FlyteIterator` class provides a lazy way to consume the collection elements as an iterator.
*   **`JSONIteratorTransformer`**: Specifically designed for `typing.Iterator[JSON]` (where `JSON` is a type alias for `typing.Dict[str, typing.Any]`). It serializes the iterator's contents into a JSONL (JSON Lines) file, which is then uploaded as a single blob. The `JSONIterator` provides a streaming interface for reading JSONL files.

#### File and Directory Types

Flytekit provides dedicated types for handling single files and directories, which abstract away the underlying storage details.

*   **`FlyteFile` and `FlyteFilePathTransformer`**:
    *   `FlyteFile` represents a single file. It implements `os.PathLike`, allowing it to be used directly with functions like `open()`.
    *   When a `FlyteFile` is returned from a task, its contents are uploaded to Flyte's blob store. If the `FlyteFile` instance points to an already remote URI (e.g., `s3://...`), no upload occurs; the URI is simply passed through.
    *   `FlyteFile["format"]` syntax (e.g., `FlyteFile["csv"]`) allows specifying the file format, which is stored as metadata.
    *   `new_remote_file()` and `new()` facilitate creating new `FlyteFile` instances for writing.
    *   The `open()` method provides a streaming file handle, optionally with caching.
    *   `from_source()` creates a `FlyteFile` from a given URI, setting it as the remote source.
    *   **Usage Example**:
        ```python
        from flytekit.types.file import FlyteFile
        import os

        def process_file(input_file: FlyteFile["txt"]) -> FlyteFile:
            with open(input_file, "r") as f:
                content = f.read()
            output_file = FlyteFile.new_remote_file(name="output.txt")
            with open(output_file, "w") as f:
                f.write(content.upper())
            return output_file
        ```

*   **`FlyteDirectory` and `FlyteDirToMultipartBlobTransformer`**:
    *   `FlyteDirectory` represents a collection of files within a directory. Similar to `FlyteFile`, it implements `os.PathLike`.
    *   When a `FlyteDirectory` is returned, its entire contents are uploaded as a multipart blob.
    *   `FlyteDirectory["format"]` (e.g., `FlyteDirectory["images"]`) specifies the directory format.
    *   `new_remote()` and `new()` create new `FlyteDirectory` instances.
    *   `new_file()` and `new_dir()` allow creating nested files and directories within a `FlyteDirectory` object.
    *   `listdir()` provides a way to list contents of a `FlyteDirectory` without downloading them, returning `FlyteFile` or `FlyteDirectory` objects that can be lazily downloaded.
    *   `crawl()` offers a generator to traverse the directory structure.
    *   The `BatchSize` annotation can be used with `FlyteDirectory` to control the batching of file downloads/uploads, which can be beneficial for very large directories.
    *   **Usage Example**:
        ```python
        from flytekit.types.directory import FlyteDirectory, BatchSize
        from annotated_types import Annotated

        def process_directory(input_dir: Annotated[FlyteDirectory, BatchSize(10)]) -> FlyteDirectory:
            # List files in the directory
            for entry in FlyteDirectory.listdir(input_dir):
                print(f"Found: {entry.path}")
            
            output_dir = FlyteDirectory.new_remote()
            # Create a new file within the output directory
            new_file = output_dir.new_file("summary.txt")
            with open(new_file, "w") as f:
                f.write("Processed directory contents.")
            return output_dir
        ```

#### Structured Data Types

Flytekit provides robust support for structured data, with `StructuredDataset` being the recommended approach.

*   **`FlyteSchema` and `FlyteSchemaTransformer` (Deprecated)**:
    *   `FlyteSchema` was an earlier mechanism for handling tabular data. It is now deprecated in favor of `StructuredDataset`.
    *   It allowed defining column names and types (`FlyteSchema[{"col1": int, "col2": str}]`).
    *   `SchemaEngine` was used to register handlers for specific dataframe formats (e.g., Pandas).
    *   `open()` provided `SchemaReader` and `SchemaWriter` interfaces for reading/writing data.

*   **`StructuredDataset` and `StructuredDatasetTransformerEngine`**:
    *   `StructuredDataset` is the modern, extensible type for structured data (e.g., DataFrames, Tables). It provides a flexible way to represent and transfer data across tasks, supporting various dataframe libraries and storage protocols.
    *   It allows specifying column types, format, and external schema information.
    *   **Extensibility**: The `StructuredDatasetTransformerEngine` uses `StructuredDatasetEncoder` and `StructuredDatasetDecoder` to support different dataframe types (e.g., Pandas DataFrame, PyArrow Table) and storage protocols (e.g., Parquet, CSV, BigQuery, Snowflake).
        *   Custom encoders/decoders can be registered using `StructuredDatasetTransformerEngine.register()`.
        *   `default_for_type` and `default_format_for_type` arguments in `register` allow setting default behaviors for specific dataframe types.
    *   **Usage**:
        *   A `StructuredDataset` can be instantiated with an in-memory dataframe or a URI pointing to data.
        *   The `open()` method, followed by `all()` or `iter()`, allows accessing the underlying dataframe in the desired format.
        *   Column subsetting is automatically handled: if a task input specifies a subset of columns, only those columns are loaded.
    *   **Usage Example**:
        ```python
        import pandas as pd
        from flytekit.types.structured import StructuredDataset
        from annotated_types import Annotated
        from collections import OrderedDict

        # Define a structured dataset with specific columns
        MyDataset = Annotated[StructuredDataset, OrderedDict([("col_a", int), ("col_b", str)])]

        def create_dataset() -> MyDataset:
            df = pd.DataFrame({"col_a": [1, 2], "col_b": ["x", "y"]})
            return StructuredDataset(dataframe=df)

        def process_dataset(input_sd: MyDataset) -> pd.DataFrame:
            # input_sd will automatically be converted to a Pandas DataFrame
            df = input_sd.open(pd.DataFrame).all()
            print(df)
            return df
        ```

#### Other Specific Types

*   **`ProtobufTransformer`**: Handles `google.protobuf.Message` types, converting them to/from Flyte `Struct` literals.
*   **`EnumTransformer`**: Supports Python `enum.Enum` types, converting them to Flyte `EnumType` literals. Only string-valued enums are supported.
*   **`NumpyArrayTransformer`**: Converts `numpy.ndarray` objects to/from single blob literals. It uses `.npy` format for serialization.
*   **`PILImageTransformer`**: Handles `PIL.Image.Image` objects, converting them to/from single blob literals, typically in PNG format. It also provides an `to_html` method for rendering images in Flyte UI.
*   **`FlyteError` and `ErrorTransformer`**: `FlyteError` is a special type used to represent errors within the Flyte system, allowing error messages and failed node IDs to be passed as literals.
*   **`FlytePickle` and `FlytePickleTransformer`**: This is a fallback mechanism. If Flytekit does not have a specific `TypeTransformer` registered for a given Python type, it defaults to pickling the object using `cloudpickle` and storing it as a blob. This ensures that any Python object can be passed through Flyte, though it may have performance implications and is less portable than explicitly supported types.

### Advanced Usage and Best Practices

*   **Annotations for Enhanced Type Information**: The `Annotated` type from `annotated_types` (or `typing.Annotated` in Python 3.9+) is widely used in Flytekit to attach additional metadata or behavior to types without changing their core Python representation. Examples include `BatchSize` for `FlyteDirectory`, column definitions for `StructuredDataset`, and `HashMethod` for custom hashing.
*   **Custom Type Extensions**: For custom Python objects or specific data formats not natively supported, implement a custom `TypeTransformer` (or `AsyncTypeTransformer` for asynchronous operations). For structured data, extend `StructuredDatasetEncoder` and `StructuredDatasetDecoder` and register them with `StructuredDatasetTransformerEngine`.
*   **Performance Considerations**:
    *   Leverage `AsyncTypeTransformer` for I/O-bound operations to improve performance.
    *   Utilize `BatchSize` annotation with `FlyteDirectory` for efficient large directory transfers.
    *   For `StructuredDataset`, choose appropriate protocols and formats (e.g., Parquet for columnar storage) and ensure your custom handlers are optimized for data access.
*   **`LiteralsResolver`**: When working with `LiteralMap` objects directly (e.g., in remote execution scenarios), `LiteralsResolver` provides a convenient way to access and convert literal values to Python types, optionally providing type hints for accurate conversion.
<!--
key: summary_type_system_&_data_management_12190404-f0d2-4bc3-86f0-f605a26e5e23
type: summary_end

-->
<!--
code_unit: flytekit.types.file.file.FlyteFile
code_unit_type: class
help_text: ''
key: example_084609a4-c16b-45e8-b536-2d0a28d852f3
type: example

-->
<!--
code_unit: flytekit.types.directory.types.FlyteDirectory
code_unit_type: class
help_text: ''
key: example_0868e33b-5852-4919-8544-43b9fdba7e2d
type: example

-->
<!--
code_unit: flytekit.types.schema.types.FlyteSchema
code_unit_type: class
help_text: ''
key: example_29ba5e4e-a93d-4959-91da-99edf552fbdd
type: example

-->
<!--
code_unit: flytekit.types.structured.structured_dataset.StructuredDataset
code_unit_type: class
help_text: ''
key: example_3d50a285-9c65-4bbb-ab0c-8431b8b0d728
type: example

-->
<!--
code_unit: flytekit.types.error.error.FlyteError
code_unit_type: class
help_text: ''
key: example_16c86991-1210-41aa-8df6-06d4cb0d6e8e
type: example

-->