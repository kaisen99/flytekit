
<!--
help_text: ''
key: summary_primitive_&_collection_type_handling_342af0e0-71ca-472a-baee-c82ce71be39b
modules:
- flytekit.core.type_engine
questions_to_answer: []
type: summary

-->
Flytekit's type system provides robust mechanisms for handling Python primitive and collection types, ensuring seamless data flow between tasks and workflows. At its core, the system translates native Python objects into Flyte's internal literal representations and vice-versa, enabling execution across diverse environments.

## Core Concepts: The TypeEngine and Transformers

The `TypeEngine` serves as the central registry and dispatcher for all type conversions within Flytekit. It manages a collection of `TypeTransformer` instances, each specialized in converting a particular Python type to and from Flyte's `LiteralType` (type definition) and `Literal` (actual value) models.

A `TypeTransformer` defines the contract for this two-way conversion:
*   `get_literal_type`: Converts a Python type hint (e.g., `int`, `List[str]`, `MyDataclass`) into a Flyte `LiteralType` schema.
*   `to_literal`: Serializes a Python value (e.g., `1`, `["a", "b"]`, `MyDataclass(x=1)`) into a Flyte `Literal` object.
*   `to_python_value`: Deserializes a Flyte `Literal` object back into a Python value.
*   `guess_python_type`: Infers the Python type from a Flyte `LiteralType`.
*   `assert_type`: Performs runtime type validation on Python values.

This modular design allows for easy extension, enabling developers to register custom transformers for their own Python objects.

## Handling Primitive Types

Flytekit automatically handles standard Python primitive types such as `int`, `float`, `str`, `bool`, `datetime.datetime`, `datetime.date`, and `datetime.timedelta`. These conversions are managed by `SimpleTransformer` instances, which leverage efficient MessagePack binary serialization for compact and fast data transfer.

For example, an `int` in Python is converted to a `Literal` with a `Primitive` scalar containing an `integer` value.

### Enumerations

Python `enum.Enum` types are supported through the `EnumTransformer`. These are mapped to Flyte's `EnumType`, which stores a list of allowed string values.

**Considerations:**
*   Only string-valued enums are currently supported.
*   Annotations (e.g., `Annotated[MyEnum, "metadata"]`) are not directly supported on enums.

### Protobuf Messages

The `ProtobufTransformer` enables the use of Google Protobuf `Message` types as inputs and outputs. It converts Protobuf messages to Flyte's `SimpleType.STRUCT` and back. This is particularly useful for integrating with systems that heavily rely on Protobuf for data interchange.

## Handling Collection Types

Flytekit provides robust support for common Python collection types, including lists and dictionaries.

### Lists

The `ListTransformer` handles Python `list` types, converting them to Flyte's `LiteralCollection`. This allows for passing lists of any supported Flyte type between tasks.

**Usage:**
```python
from typing import List
from flytekit import task, workflow

@task
def process_numbers(numbers: List[int]) -> List[int]:
    return [n * 2 for n in numbers]

@workflow
def my_list_workflow(input_numbers: List[int]) -> List[int]:
    return process_numbers(numbers=input_numbers)

# Example execution: my_list_workflow(input_numbers=[1, 2, 3])
```

**Considerations:**
*   Only univariate lists (e.g., `List[int]`, `List[str]`) are supported. Nested lists like `List[List[int]]` are also handled recursively.

### Dictionaries

The `DictTransformer` manages Python `dict` types. Flytekit distinguishes between two primary ways dictionaries are handled:

1.  **Typed Dictionaries (`Dict[str, T]`):** When dictionary keys are explicitly typed as `str` and values are of a consistent type `T`, the dictionary is converted to a Flyte `LiteralMap`. This is the preferred and most efficient way to pass structured dictionaries.

    **Usage:**
    ```python
    from typing import Dict
    from flytekit import task, workflow

    @task
    def process_data(data: Dict[str, float]) -> Dict[str, float]:
        return {k: v * 10 for k, v in data.items()}

    @workflow
    def my_dict_workflow(input_data: Dict[str, float]) -> Dict[str, float]:
        return process_data(data=input_data)

    # Example execution: my_dict_workflow(input_data={"a": 1.0, "b": 2.5})
    ```

2.  **Untyped or Generic Dictionaries:** Dictionaries with non-string keys or mixed value types are serialized as a Flyte `Literal` with `SimpleType.STRUCT`. By default, these are serialized using MessagePack for efficiency.

    **Pickle Fallback:** For complex or untypable dictionaries, a fallback to Python's `pickle` serialization is available. This can be enabled using the `Annotated` type hint:
    ```python
    from typing import Annotated, Dict, Any
    from flytekit import task
    from collections import OrderedDict

    @task
    def process_untyped_dict(data: Annotated[Dict[Any, Any], OrderedDict(allow_pickle=True)]) -> None:
        print(data)

    # This allows dictionaries with non-string keys or complex objects to be passed.
    ```
    **Considerations:** While `pickle` offers flexibility, it is generally less performant and less portable than explicit type handling and MessagePack. Use it only when necessary.

### Unions

The `UnionTransformer` supports Python `typing.Union` types, including `Optional` types (which are syntactic sugar for `Union[T, None]`). When a value is passed for a `Union` type, the transformer attempts to serialize it using the first matching sub-type. During deserialization, it tries to match the received literal's type tag to one of the expected union variants.

**Usage:**
```python
from typing import Union, Optional
from flytekit import task, workflow

@task
def handle_union(value: Union[int, str]) -> Union[int, str]:
    if isinstance(value, int):
        return value + 10
    return value + " World"

@task
def handle_optional(name: Optional[str]) -> str:
    return name if name else "Guest"

# Example execution:
# handle_union(value=5) -> 15
# handle_union(value="Hello") -> "Hello World"
# handle_optional(name="Flyte") -> "Flyte"
# handle_optional(name=None) -> "Guest"
```

**Considerations:**
*   **Ambiguity:** If multiple sub-types in a `Union` are structurally identical or can represent the same literal value, it can lead to ambiguity during serialization or deserialization, resulting in a `TypeError`. Design `Union` types carefully to avoid such overlaps.

## Handling Structured Data (Dataclasses)

Python `dataclasses` are a powerful way to define custom data structures. The `DataclassTransformer` provides comprehensive support for these, converting them to and from Flyte literals using MessagePack for efficient binary serialization.

**Serialization Lifecycle:**
1.  Dataclass attributes are prepared for serialization (e.g., converting string paths to `FlyteFile` objects if annotated).
2.  The `mashumaro` library serializes the dataclass to MessagePack Bytes.
3.  These bytes are wrapped into a Flyte `Literal` with a `Binary` scalar.

**Deserialization Lifecycle:**
1.  MessagePack Bytes are deserialized back into a Python dictionary.
2.  `mashumaro` converts the dictionary into the target dataclass instance.
3.  A post-processing step (`_fix_dataclass_int`) ensures `int` values, which might be upconverted to `double` in Protobuf `Struct` representations, are correctly cast back to `int`.

**JSON Schema Support (Experimental):**
The `DataclassTransformer` can optionally extract a JSON Schema for the dataclass. This schema is embedded in the `LiteralType` metadata and is primarily useful for UI/CLI tools to provide auto-completion and validation hints.

**Nested Flyte Types:**
A unique capability is the automatic handling of Flyte-specific types (like `FlyteFile`, `FlyteDirectory`, `StructuredDataset`) when they are fields within a dataclass. For instance, if a dataclass has a `FlyteFile` field, Flytekit can automatically convert a string path (e.g., `"s3://path/to/file"`) provided by the user into a `FlyteFile` object before serialization.

**Example:**
```python
from dataclasses import dataclass
from flytekit import task, workflow
from flytekit.types.file import FlyteFile

@dataclass
class MyData:
    id: int
    name: str
    data_file: FlyteFile

@task
def process_my_data(data: MyData) -> MyData:
    print(f"Processing ID: {data.id}, Name: {data.name}")
    # data.data_file is a FlyteFile object, can be used directly
    with open(data.data_file.path, "r") as f:
        content = f.read()
        print(f"File content: {content}")
    # Return a new dataclass instance or modify existing
    return MyData(id=data.id + 1, name=data.name + "_processed", data_file=data.data_file)

@workflow
def my_dataclass_workflow(input_data: MyData) -> MyData:
    return process_my_data(data=input_data)

# Example execution:
# my_dataclass_workflow(input_data=MyData(id=1, name="test", data_file="s3://my-bucket/input.txt"))
```

**Considerations:**
*   FlyteAnnotations applied directly to dataclass types are currently skipped.
*   JSON Schema support is experimental and may have limitations with certain Python features (e.g., postponed annotations).
*   The old JSON-based serialization format for dataclasses is deprecated; MessagePack is the recommended approach.

## Handling File-like Objects

Flytekit provides transformers for generic Python file-like objects: `BinaryIOTransformer` for `typing.BinaryIO` and `TextIOTransformer` for `typing.TextIO`. These map to Flyte's `BlobType`.

**Important Limitation:**
These transformers do not implement `to_literal`. This means you cannot directly pass an open `BinaryIO` or `TextIO` object as a task input. Instead, you should pass a path (e.g., `str` or `FlyteFile`/`FlyteDirectory`) which Flytekit can then resolve to a local file-like object within the task. The `to_python_value` method will provide a file-like object opened from a downloaded URI.

**Batching for Directories:**
For `FlyteDirectory` types, the `BatchSize` annotation can be used to optimize download and upload operations. This is particularly useful for directories containing a large number of small files, as it allows Flytekit to process them in chunks, reducing memory footprint.

**Usage:**
```python
from typing import Annotated
from flytekit import task
from flytekit.types.directory import FlyteDirectory
from flytekit.core.type_engine import BatchSize

@task
def process_batched_directory(
    input_dir: Annotated[FlyteDirectory, BatchSize(10)],
) -> Annotated[FlyteDirectory, BatchSize(100)]:
    # Files from input_dir will be downloaded in batches of 10
    # ... process files ...
    # Output directory will be uploaded in batches of 100
    return FlyteDirectory(...)
```

## Advanced Capabilities and Considerations

### Asynchronous Conversions

Many `TypeTransformer` implementations are `AsyncTypeTransformer`s, enabling asynchronous serialization and deserialization. This is crucial for performance, especially when dealing with large collections or remote data, as it allows concurrent I/O operations. The `TypeEngine` automatically manages the asynchronous execution, allowing synchronous calls to `to_literal` and `to_python_value` to seamlessly leverage the underlying async capabilities.

### Type Assertions

`TypeTransformer`s can enforce strict type assertions. When `type_assertions_enabled` is `True` for a transformer, Flytekit performs runtime checks to ensure that the Python value matches the declared Python type, preventing unexpected type mismatches during execution.

### Restricted Types

Certain Python types are explicitly marked as "restricted" using `RestrictedTypeTransformer`. These types are not allowed as direct inputs or outputs of tasks and workflows, and attempting to use them will raise a `RestrictedTypeError`. This mechanism helps enforce best practices and prevent the use of types that are not suitable for Flyte's distributed execution model.

### Literal Resolution

The `LiteralsResolver` is a utility class designed to simplify working with Flyte's `LiteralMap` objects, particularly in scenarios like `FlyteRemote` or when inspecting raw task/workflow outputs. It allows developers to retrieve specific literal values from a map and convert them to their corresponding Python native types, optionally providing type hints for accurate conversion.

**Usage:**
```python
from flytekit.core.type_engine import LiteralsResolver
from flytekit.models.literals import LiteralMap, Literal, Scalar, Primitive
from flytekit.models.types import LiteralType, SimpleType
from flytekit.core.context_manager import FlyteContext

# Assume you have a LiteralMap 'lm' from a Flyte execution
# For demonstration, create a dummy LiteralMap
dummy_lm = LiteralMap(
    literals={
        "my_int": Literal(scalar=Scalar(primitive=Primitive(integer=123))),
        "my_str": Literal(scalar=Scalar(primitive=Primitive(string_value="hello"))),
    }
)

# Assume you have a FlyteContext 'ctx'
ctx = FlyteContext.current_context()

resolver = LiteralsResolver(literals=dummy_lm.literals)

# Get values with explicit type hints
my_int_value = resolver.get("my_int", as_type=int)
my_str_value = resolver.get("my_str", as_type=str)

print(f"Resolved int: {my_int_value} ({type(my_int_value)})")
print(f"Resolved str: {my_str_value} ({type(my_str_value)})")
```

### Custom Type Handling

Developers can extend Flytekit's type system by implementing their own `TypeTransformer` (or `AsyncTypeTransformer`) and registering it with the `TypeEngine`. This allows Flytekit to seamlessly handle custom Python objects as task inputs and outputs, providing a powerful mechanism for integrating domain-specific data structures.

### Performance Considerations

Flytekit prioritizes performance by:
*   **Binary Serialization:** Using MessagePack for many types, which is more compact and faster than JSON.
*   **Asynchronous Operations:** Leveraging `AsyncTypeTransformer` for I/O-bound operations, such as reading/writing large files or collections.
*   **Batching:** Providing `BatchSize` for `FlyteDirectory` to optimize large file transfers.

## Best Practices

*   **Use Explicit Type Hints:** Always provide clear and precise type hints for task inputs and outputs. This enables Flytekit to correctly infer `LiteralType` schemas and apply the appropriate transformers, ensuring reliable data serialization and deserialization.
*   **Prefer Specific Flyte Types for Files:** For file and directory inputs/outputs, use `FlyteFile` and `FlyteDirectory` instead of generic `typing.BinaryIO` or `typing.TextIO`. This allows Flytekit to manage the underlying storage paths and optimize data transfer.
*   **Design Union Types Carefully:** When using `typing.Union`, ensure that the constituent types are distinct enough to avoid ambiguity during serialization and deserialization.
*   **Avoid Pickle Unless Necessary:** While `pickle` offers flexibility for complex objects, it is generally less performant, less portable, and less secure than explicit type handling. Use it as a last resort for untypable data.
*   **Leverage Dataclasses:** For custom structured data, `dataclasses` provide a clean and efficient way to define types that integrate seamlessly with Flytekit's type system.
<!--
key: summary_primitive_&_collection_type_handling_342af0e0-71ca-472a-baee-c82ce71be39b
type: summary_end

-->
<!--
code_unit: flytekit.core.type_engine.SimpleTransformer
code_unit_type: class
help_text: ''
key: example_7503943c-fe20-4eba-8224-56efd4ebffab
type: example

-->
<!--
code_unit: flytekit.core.type_engine.ListTransformer
code_unit_type: class
help_text: ''
key: example_dc1e0bb0-fb5c-46cd-8068-3b4274291051
type: example

-->
<!--
code_unit: flytekit.core.type_engine.DictTransformer
code_unit_type: class
help_text: ''
key: example_24ba3c9c-c9a2-4e62-8bb9-9d408fb848b4
type: example

-->
<!--
code_unit: flytekit.core.type_engine.EnumTransformer
code_unit_type: class
help_text: ''
key: example_be20fb8a-74af-4fe4-8f23-5e6f49f67bb6
type: example

-->
<!--
code_unit: flytekit.core.type_engine.UnionTransformer
code_unit_type: class
help_text: ''
key: example_f0f45064-77f4-4925-a2ff-b2e7520358be
type: example

-->
<!--
code_unit: flytekit.core.type_engine.TextIOTransformer
code_unit_type: class
help_text: ''
key: example_17301605-be40-4bb6-b92b-5e0918c69a17
type: example

-->
<!--
code_unit: flytekit.core.type_engine.BinaryIOTransformer
code_unit_type: class
help_text: ''
key: example_345b37d1-9f50-46f8-be03-342e37a497ec
type: example

-->