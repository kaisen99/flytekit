
<!--
help_text: ''
key: summary_legacy_schema_types_8e71733d-1971-4324-a506-5fe49257a904
modules:
- flytekit.types.schema.types
- flytekit.types.schema.types_pandas
questions_to_answer: []
type: summary

-->
Legacy Schema Types

Legacy Schema Types provide a mechanism for defining and handling structured datasets, such as dataframes, within Flyte workflows. While still functional, this type system is deprecated. For new development, it is strongly recommended to use the more flexible and modern Structured Dataset type.

### Overview of Legacy Schema Types

A legacy schema type represents a structured collection of data, similar to a table or dataframe, with explicitly defined column names and their corresponding data types. The primary storage format supported is Parquet.

The core component is the `FlyteSchema` class. It allows users to define the structure of their data and provides methods for interacting with the underlying storage, abstracting away the complexities of local and remote file operations.

### Defining a Legacy Schema

To define a legacy schema with specific column types, use the `FlyteSchema` class with a dictionary specifying column names and their Python types.

**Example:**

```python
from flytekit.types.schema import FlyteSchema
import datetime

# Define a schema for a dataset with specific columns and types
MyTypedSchema = FlyteSchema[
    {
        "id": int,
        "name": str,
        "value": float,
        "timestamp": datetime.datetime,
        "duration": datetime.timedelta,
        "is_active": bool,
    }
]

# A schema without explicit column types can also be used,
# but it provides less type safety.
UntypedSchema = FlyteSchema
```

The `format()` method on a `FlyteSchema` class indicates the storage format, which is currently limited to `SchemaFormat.PARQUET`.

### Reading and Writing Legacy Schema Data

Legacy schema types manage data transfer between local execution environments and remote storage (e.g., S3, GCS). When a `FlyteSchema` object is created or passed between tasks, Flyte handles the serialization and deserialization, including the necessary data uploads and downloads.

The `FlyteSchema` object provides an `open()` method to obtain a reader or writer object, depending on the intended operation.

*   **`SchemaOpenMode.WRITE`**: Used when a task produces a `FlyteSchema` output. Data is written to a local path, which Flyte then uploads to the remote `remote_path`.
*   **`SchemaOpenMode.READ`**: Used when a task consumes a `FlyteSchema` input. Flyte downloads the data from the `remote_path` to a `local_path`, allowing the task to read it.

The `open()` method returns an instance of `SchemaReader` or `SchemaWriter`, which are abstract base classes. Concrete implementations, such as `PandasSchemaReader` and `PandasSchemaWriter`, handle the actual reading and writing of data for specific dataframe types (e.g., `pandas.DataFrame`).

**Example: Writing a Pandas DataFrame to a Legacy Schema**

```python
import pandas as pd
from flytekit.types.schema import FlyteSchema, SchemaOpenMode
from flytekit.core.context_manager import FlyteContextManager, FlyteContext
from flytekit.remote.remote import FlyteRemote

# Assume a FlyteContext is available for local execution or testing
# In a real Flyte task, the context is managed automatically.
ctx = FlyteContextManager.current_context()

# Define a typed schema
MySchema = FlyteSchema[{"col1": int, "col2": str}]

# Create a FlyteSchema instance in write mode
# local_path will be a temporary directory managed by Flyte
schema_obj = MySchema(supported_mode=SchemaOpenMode.WRITE)

# Create a Pandas DataFrame
df = pd.DataFrame({"col1": [1, 2, 3], "col2": ["A", "B", "C"]})

# Open the schema object in write mode and get a writer
writer = schema_obj.open(dataframe_fmt=pd.DataFrame)

# Write the DataFrame
writer.write(df)

# At this point, the data is written to the local_path.
# When the task completes, Flyte will upload this data to the remote_path.
print(f"Data written to local path: {schema_obj.local_path}")
print(f"Will be uploaded to remote path: {schema_obj.remote_path}")
```

**Example: Reading a Pandas DataFrame from a Legacy Schema**

```python
import pandas as pd
from flytekit.types.schema import FlyteSchema, SchemaOpenMode
from flytekit.core.context_manager import FlyteContextManager, FlyteContext

# Assume a FlyteContext is available
ctx = FlyteContextManager.current_context()

# In a real task, 'remote_path' would be provided by Flyte.
# For demonstration, let's assume a remote path from the previous write example.
# In a real scenario, you would receive a FlyteSchema object as an input.
# For this example, we simulate receiving a FlyteSchema object that points to remote data.
# The 'downloader' callable is crucial for Flyte to fetch the data.
def mock_downloader(remote_path, local_path):
    # In a real scenario, this would involve downloading from S3/GCS etc.
    # For this example, let's create a dummy file to simulate downloaded data.
    print(f"Simulating download from {remote_path} to {local_path}")
    import os
    os.makedirs(local_path, exist_ok=True)
    pd.DataFrame({"col1": [10, 20], "col2": ["X", "Y"]}).to_parquet(os.path.join(local_path, "data.parquet"))

# Create a FlyteSchema instance in read mode, pointing to a remote path
# The local_path will be where Flyte downloads the data.
schema_obj_read = FlyteSchema(
    remote_path="s3://my-bucket/my-schema-data/some-uuid", # This would be the actual remote path
    supported_mode=SchemaOpenMode.READ,
    downloader=mock_downloader # Flyte provides this callable
)

# Open the schema object in read mode and get a reader
reader = schema_obj_read.open(dataframe_fmt=pd.DataFrame)

# Read all data into a Pandas DataFrame
read_df = reader.all()
print("Read DataFrame:")
print(read_df)

# Alternatively, iterate over chunks (if multiple files exist)
# for chunk_df in reader.iter():
#     print("Read chunk:")
#     print(chunk_df)
```

The `as_readonly()` method allows converting a `FlyteSchema` object (potentially created in write mode) into a read-only version, which can be useful for subsequent read operations on the same data within a task.

### Schema Handling Components

The legacy schema system relies on several internal components to manage type conversions and I/O operations:

*   **`SchemaReader` and `SchemaWriter`**: These are generic abstract base classes defining the interface for reading and writing structured data.
    *   `LocalIOSchemaReader` and `LocalIOSchemaWriter` are specialized base classes for handlers that operate on local file paths, with Flyte managing the remote data transfer.
*   **`SchemaFormat`**: An enumeration defining the supported storage formats. Currently, only `PARQUET` is supported.
*   **`SchemaEngine`**: This central registry manages `SchemaHandler` implementations. A `SchemaHandler` connects a specific Python object type (e.g., `pandas.DataFrame`) to its corresponding `SchemaReader` and `SchemaWriter` implementations. This allows Flyte to automatically determine how to handle different dataframe types.
*   **`FlyteSchemaTransformer`**: This component is responsible for the serialization and deserialization of `FlyteSchema` objects to and from Flyte's internal `Literal` representation. It maps Python types (like `int`, `str`, `float`, `datetime.datetime`, `datetime.timedelta`, `bool`) to Flyte's `SchemaColumnType` for schema definition. It also orchestrates the asynchronous data transfer (upload/download) between local and remote storage.

### Limitations and Deprecation

As noted, `FlyteSchema` is deprecated. Key limitations include:

*   **Limited Format Support**: Currently, only Parquet is supported as a storage format.
*   **Complexity**: The underlying implementation involves managing local and remote paths, and explicit `open()` calls, which can be less intuitive than newer type systems.
*   **Future Development**: All new features and improvements for structured data handling will focus on Structured Dataset.

For new projects and when refactoring existing code, migrate to using Structured Dataset. It offers a more streamlined and flexible approach to handling structured data in Flyte.
<!--
key: summary_legacy_schema_types_8e71733d-1971-4324-a506-5fe49257a904
type: summary_end

-->