
<!--
help_text: ''
key: summary_structured_data_management_373e75b5-2e52-4ffc-b331-9e2315d729ce
modules:
- flytekit.types.structured.structured_dataset
- flytekit.types.schema.types
questions_to_answer: []
type: summary

-->
Structured Data Management in Flytekit provides a robust and flexible mechanism for handling structured datasets, such as dataframes, within your workflows. This system ensures type safety, enables efficient data transfer, and supports various storage formats and protocols. It is the recommended approach for managing structured data, superseding the older `FlyteSchema` functionality.

### The StructuredDataset Type

The `StructuredDataset` type is the primary user-facing interface for working with structured data. It acts as a wrapper around underlying dataframe objects (e.g., Pandas DataFrames, Spark DataFrames) and manages their serialization, deserialization, and storage.

**Defining Structured Datasets in Tasks**

To use structured datasets, annotate your task inputs and outputs with `StructuredDataset`. You can optionally define the expected schema (columns and their types) using type annotations.

```python
import pandas as pd
from flytekit.types.structured import StructuredDataset
from typing import OrderedDict

# Define a StructuredDataset with a specific schema
MyDataset = StructuredDataset.columns(OrderedDict([
    ("name", str),
    ("age", int),
    ("city", str),
]))

@task
def process_data(input_data: MyDataset) -> MyDataset:
    # Read the data into a Pandas DataFrame
    df = input_data.open(pd.DataFrame).all()
    
    # Perform operations
    df["age"] = df["age"] + 1
    
    # Return the modified DataFrame wrapped in a StructuredDataset
    return StructuredDataset(dataframe=df)

@task
def analyze_data(data: StructuredDataset.columns(OrderedDict([("age", int)]))):
    # This task only needs the 'age' column, demonstrating column subsetting
    df = data.open(pd.DataFrame).all()
    print(f"Average age: {df['age'].mean()}")
```

**Creating and Accessing Structured Datasets**

A `StructuredDataset` can be instantiated in a few ways:

1.  **From an existing dataframe**:
    ```python
    import pandas as pd
    from flytekit.types.structured import StructuredDataset

    my_df = pd.DataFrame({"col1": [1, 2], "col2": ["a", "b"]})
    sd_output = StructuredDataset(dataframe=my_df)
    ```
    When a task returns a raw dataframe (e.g., `return my_df` where the return type is `StructuredDataset`), the system automatically wraps it in a `StructuredDataset` instance using default handlers.

2.  **From a URI**:
    ```python
    from flytekit.types.structured import StructuredDataset

    # This creates a StructuredDataset object that points to remote data
    # The data is not downloaded until explicitly requested.
    sd_input = StructuredDataset(uri="s3://my-bucket/path/to/data.parquet", file_format="parquet")
    ```

**Reading Data**

To access the underlying dataframe from a `StructuredDataset` object, use the `open()` method followed by `all()` or `iter()`.

*   **`open(dataframe_type)`**: Specifies the desired Python dataframe type (e.g., `pandas.DataFrame`, `pyarrow.Table`) for decoding. This method returns the `StructuredDataset` instance itself, allowing for method chaining.
*   **`all()`**: Reads the entire dataset into memory as the specified `dataframe_type`. This is suitable for smaller datasets.
*   **`iter()`**: Returns a generator that yields chunks of the dataset as the specified `dataframe_type`. This is ideal for large datasets that cannot fit entirely into memory.

```python
import pandas as pd
from flytekit.types.structured import StructuredDataset

@task
def consume_data(sd: StructuredDataset):
    # Read the entire dataset into a Pandas DataFrame
    df_full = sd.open(pd.DataFrame).all()
    print(f"Full DataFrame shape: {df_full.shape}")

    # Iterate over the dataset in chunks
    for chunk in sd.open(pd.DataFrame).iter():
        print(f"Processing chunk of shape: {chunk.shape}")
```

**Schema Definition and Column Subsetting**

The `StructuredDataset.columns()` class method allows you to define the expected schema of your data. This schema is used for type validation and, crucially, for **column subsetting**. If a task's input `StructuredDataset` type annotation specifies a subset of columns, only those columns will be loaded when `open().all()` or `open().iter()` is called.

```python
from typing import OrderedDict
from flytekit.types.structured import StructuredDataset

# Define a dataset type that expects 'id' (int) and 'value' (float)
FullDataset = StructuredDataset.columns(OrderedDict([("id", int), ("value", float), ("timestamp", datetime.datetime)]))

# Define a dataset type that only needs 'id'
IdOnlyDataset = StructuredDataset.columns(OrderedDict([("id", int)]))

@task
def producer_task() -> FullDataset:
    # ... creates a dataframe with id, value, timestamp ...
    return StructuredDataset(dataframe=my_full_df)

@task
def consumer_task(data: IdOnlyDataset):
    # When 'data.open(pd.DataFrame).all()' is called, only the 'id' column will be loaded.
    df = data.open(pd.DataFrame).all()
    print(f"Columns loaded: {df.columns.tolist()}") # Output: ['id']
```

This column subsetting capability is a powerful feature for optimizing data transfer and memory usage in workflows, especially when dealing with wide datasets.

### Extending Structured Data Management with Custom Handlers

The `StructuredDataset` system is highly extensible, allowing you to integrate support for any Python dataframe library or custom data format. This is achieved by implementing and registering custom encoders and decoders.

**The Role of Encoders and Decoders**

*   **`StructuredDatasetEncoder`**: An abstract base class that defines how a Python dataframe object is converted into a Flyte `literals.StructuredDataset` (its literal representation, including URI and metadata). You implement the `encode` method to write your dataframe to a specified storage location and format.
*   **`StructuredDatasetDecoder`**: An abstract base class that defines how a Flyte `literals.StructuredDataset` is converted back into a Python dataframe object. You implement the `decode` method to read data from a URI into your desired dataframe type.

Both `StructuredDatasetEncoder` and `StructuredDatasetDecoder` are initialized with:
*   `python_type`: The specific Python dataframe class they handle (e.g., `pandas.DataFrame`).
*   `protocol`: The storage protocol they support (e.g., `"s3"`, `"gs"`, `"file"`). If `None`, the handler is registered for all protocols that Flytekit's data persistence layer can handle.
*   `supported_format`: The data format they support (e.g., `"parquet"`, `"csv"`). An empty string (`""`) implies support for any format.

**Registering Custom Handlers**

Custom encoders and decoders are registered with the `StructuredDatasetTransformerEngine` using its `register()` class method.

```python
from flytekit.types.structured import StructuredDatasetEncoder, StructuredDatasetDecoder, StructuredDatasetTransformerEngine
from flytekit.core.context_manager import FlyteContext
from flytekit.models import literals
from flytekit.models.types import StructuredDatasetType
from abc import ABC, abstractmethod
from typing import Type, Optional, Union, Iterator, Generic

# Assume 'MyDataFrame' is a custom dataframe type
class MyDataFrame:
    def __init__(self, data):
        self.data = data
    def __repr__(self):
        return f"MyDataFrame({self.data})"

# Example Custom Encoder
class MyDataFrameEncoder(StructuredDatasetEncoder[MyDataFrame]):
    def __init__(self):
        super().__init__(python_type=MyDataFrame, protocol="s3", supported_format="my_format")

    def encode(
        self,
        ctx: FlyteContext,
        structured_dataset: StructuredDataset,
        structured_dataset_type: StructuredDatasetType,
    ) -> literals.StructuredDataset:
        # Implement logic to write structured_dataset.dataframe (MyDataFrame) to S3
        # For example, convert to bytes and upload to ctx.file_access.get_random_remote_directory()
        # Return a literals.StructuredDataset with the URI and metadata
        
        # Placeholder: In a real implementation, you'd write the data
        uri = ctx.file_access.get_random_remote_directory() + ".my_format"
        print(f"Encoding MyDataFrame to {uri}")
        # Simulate upload
        ctx.file_access.put_data("dummy_data".encode(), uri) 
        
        return literals.StructuredDataset(
            uri=uri,
            metadata=literals.StructuredDatasetMetadata(structured_dataset_type=structured_dataset_type)
        )

# Example Custom Decoder
class MyDataFrameDecoder(StructuredDatasetDecoder[MyDataFrame]):
    def __init__(self):
        super().__init__(python_type=MyDataFrame, protocol="s3", supported_format="my_format")

    def decode(
        self,
        ctx: FlyteContext,
        flyte_value: literals.StructuredDataset,
        current_task_metadata: literals.StructuredDatasetMetadata,
    ) -> Union[MyDataFrame, Iterator[MyDataFrame]]:
        # Implement logic to read from flyte_value.uri into MyDataFrame
        # For example, download from ctx.file_access and parse bytes
        
        # Placeholder: In a real implementation, you'd read the data
        print(f"Decoding MyDataFrame from {flyte_value.uri}")
        # Simulate download
        data = ctx.file_access.get_data(flyte_value.uri)
        return MyDataFrame(data.decode())

# Register the handlers
StructuredDatasetTransformerEngine.register(MyDataFrameEncoder())
StructuredDatasetTransformerEngine.register(MyDataFrameDecoder())

# Now you can use MyDataFrame in your tasks
@task
def produce_my_dataframe() -> StructuredDataset:
    return StructuredDataset(dataframe=MyDataFrame("some_data"))

@task
def consume_my_dataframe(sd: StructuredDataset):
    my_df = sd.open(MyDataFrame).all()
    print(f"Consumed: {my_df}")
```

When registering, you can specify `default_for_type=True` to make a handler the default for a given Python type when a raw dataframe is returned from a task. This is useful for common types like Pandas DataFrames.

### The Structured Dataset Transformer Engine

The `StructuredDatasetTransformerEngine` is the core component responsible for orchestrating the conversion between Python dataframe objects and Flyte's internal `Literal` representation of structured datasets. It acts as a central registry and dispatcher for all `StructuredDatasetEncoder` and `StructuredDatasetDecoder` instances.

**Key Responsibilities:**

*   **Handler Discovery**: When a `StructuredDataset` needs to be encoded or decoded, the engine intelligently selects the most appropriate handler based on the Python type, desired storage protocol, and file format. It prioritizes exact matches but can fall back to generic handlers (e.g., those supporting `None` protocol or `""` format).
*   **Type Conversion**: It manages the `async_to_literal` (Python to Flyte) and `async_to_python_value` (Flyte to Python) processes, invoking the registered encoders and decoders.
*   **Metadata Propagation**: Ensures that schema information (columns, format) is correctly propagated between the Python type system and the Flyte IDL `StructuredDatasetType` metadata.
*   **Backward Compatibility**: The engine includes logic to handle conversions from older `FlyteSchema` literals to `StructuredDataset` literals, ensuring smooth transitions for existing workflows.

### Best Practices and Considerations

*   **Prefer `StructuredDataset`**: Always use `StructuredDataset` for structured data inputs and outputs in your task signatures. This provides the most robust and feature-rich experience.
*   **Leverage Type Annotations**: Define your dataset schemas using `StructuredDataset.columns()` to enable type validation and automatic column subsetting, which can significantly improve performance for large datasets.
*   **Lazy Loading for Large Data**: For large datasets, always use `sd.open(YourDataFrameType).iter()` to process data in chunks, preventing out-of-memory errors. Use `sd.open(YourDataFrameType).all()` only when the entire dataset is known to fit comfortably in memory.
*   **Understand Protocol and Format**: When implementing custom handlers, carefully consider the `protocol` (e.g., `s3`, `gs`, `file`) and `supported_format` (e.g., `parquet`, `csv`, `json`) your handler supports. This ensures the engine can correctly match your handler to the data's storage location and format.
*   **Error Handling**: The `DuplicateHandlerError` is raised if you attempt to register a handler that conflicts with an already registered one for the same Python type, protocol, and format. Use the `override=True` flag during registration if you explicitly intend to replace an existing handler.
*   **Performance of Custom Handlers**: The efficiency of your custom `encode` and `decode` implementations directly impacts workflow performance. Optimize I/O operations and data transformations within these methods.
*   **URI Management**: The system automatically manages URIs for data persistence. When returning a `StructuredDataset` from a task, the data is uploaded to a Flyte-managed location. When consuming, the data is downloaded from the specified URI.
<!--
key: summary_structured_data_management_373e75b5-2e52-4ffc-b331-9e2315d729ce
type: summary_end

-->
<!--
code_unit: flytekit.types.structured.structured_dataset.StructuredDataset
code_unit_type: class
help_text: ''
key: example_7d3832be-a33e-4b26-ac70-aba7ee1247d9
type: example

-->
<!--
code_unit: flytekit.types.structured.structured_dataset.StructuredDatasetTransformerEngine
code_unit_type: class
help_text: ''
key: example_f42e926c-1428-4746-a1ae-43ea3750b65e
type: example

-->
<!--
code_unit: flytekit.types.structured.structured_dataset.StructuredDatasetEncoder
code_unit_type: class
help_text: ''
key: example_bdf43b05-76a3-472a-9c28-17b96e3c6bee
type: example

-->
<!--
code_unit: flytekit.types.structured.structured_dataset.StructuredDatasetDecoder
code_unit_type: class
help_text: ''
key: example_dde70665-7d4d-481d-a471-21d3309bbdd3
type: example

-->
<!--
code_unit: flytekit.types.schema.types.FlyteSchema
code_unit_type: class
help_text: ''
key: example_821ffffd-aace-4666-b194-d3a3779d85aa
type: example

-->
<!--
code_unit: flytekit.types.schema.types.FlyteSchemaTransformer
code_unit_type: class
help_text: ''
key: example_2fabc726-fbdd-4724-a9f8-aae8d0e83756
type: example

-->
<!--
code_unit: flytekit.types.schema.types.SchemaEngine
code_unit_type: class
help_text: ''
key: example_9416d70d-f535-40b1-aa70-99cfb093088b
type: example

-->
<!--
code_unit: flytekit.types.schema.types.SchemaHandler
code_unit_type: class
help_text: ''
key: example_e07c8e37-4bb4-4e19-9821-9863f82be270
type: example

-->
<!--
code_unit: flytekit.types.schema.types.SchemaReader
code_unit_type: class
help_text: ''
key: example_212d6798-8b81-4953-aa95-c5c3a77b61ad
type: example

-->
<!--
code_unit: flytekit.types.schema.types.SchemaWriter
code_unit_type: class
help_text: ''
key: example_236e821d-36ef-4596-9f78-d81a81233064
type: example

-->
<!--
code_unit: flytekit.types.schema.types.SchemaFormat
code_unit_type: class
help_text: ''
key: example_0bd0acd0-0b5f-4cb2-ba71-ac6cb7b18e5d
type: example

-->
<!--
code_unit: flytekit.types.schema.types.SchemaOpenMode
code_unit_type: class
help_text: ''
key: example_4dfa8ad9-c269-417e-96c4-f993db54adbb
type: example

-->
<!--
code_unit: flytekit.types.schema.types.LocalIOSchemaReader
code_unit_type: class
help_text: ''
key: example_9dc25151-e38f-4a99-b580-a9f0ca7216d1
type: example

-->
<!--
code_unit: flytekit.types.schema.types.LocalIOSchemaWriter
code_unit_type: class
help_text: ''
key: example_b6d04562-7b82-4813-86bf-a5cddc8ee3e2
type: example

-->