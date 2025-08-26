
<!--
help_text: ''
key: summary_custom_python_object_serialization_69126556-c398-41af-b74d-beb1378702ab
modules:
- flytekit.core.type_engine
- flytekit.types.pickle.pickle
- flytekit.types.error.error
questions_to_answer: []
type: summary

-->
## Custom Python Object Serialization

Custom Python object serialization is essential for bridging the gap between arbitrary Python types and Flyte's internal type system. This process enables developers to define how their custom data structures are converted into a format Flyte understands for execution, storage, and inter-task communication, and then back into Python objects.

The core of this capability lies within the `TypeEngine` and `TypeTransformer` components.

### The TypeEngine: Orchestrating Type Conversions

The `TypeEngine` serves as the central registry and orchestrator for all type conversions within the system. It manages a collection of `TypeTransformer` instances, each responsible for handling specific Python types. When a Python object needs to be serialized for a Flyte task input or output, or deserialized back into a Python object, the `TypeEngine` determines the appropriate `TypeTransformer` to use.

**Key Capabilities:**

*   **Type Registration:** Developers can extend the system by registering their own `TypeTransformer` implementations using `TypeEngine.register`. This allows for custom serialization logic for any Python type.
*   **LiteralType Conversion:** The `TypeEngine.to_literal_type` method converts a Python type hint (e.g., `MyDataclass`, `List[int]`) into a Flyte `LiteralType`. This `LiteralType` defines the schema and structure of the data within Flyte's type system.
*   **Value Serialization:** The `TypeEngine.to_literal` method takes a Python object and its corresponding Python type, and converts it into a Flyte `Literal`. This `Literal` is the concrete representation of the data that Flyte understands.
*   **Value Deserialization:** Conversely, `TypeEngine.to_python_value` converts a Flyte `Literal` back into a Python object, based on an expected Python type.
*   **Type Guessing:** `TypeEngine.guess_python_type` attempts to infer the original Python type from a Flyte `LiteralType`. This is particularly useful when the Python type is not explicitly known, such as when inspecting outputs from an unknown task.
*   **Annotation Handling:** The `TypeEngine` recognizes and processes `Annotated` types (e.g., `Annotated[MyType, MyAnnotation]`). It can extract custom annotations, including those that specify a `TypeTransformer` directly or provide metadata for serialization.
*   **Asynchronous Operations:** The `TypeEngine` supports asynchronous serialization and deserialization, leveraging `AsyncTypeTransformer` implementations and managing concurrent operations efficiently.

**Integration Points:**

The `TypeEngine` is implicitly used whenever Python objects are passed as inputs to or returned as outputs from tasks and workflows. Developers interact with it primarily by defining type hints for their task signatures, and optionally by registering custom `TypeTransformer` instances.

### Implementing Custom Serialization with TypeTransformers

`TypeTransformer` is the abstract base class that defines the interface for converting between Python types and Flyte's `Literal` representation. To implement custom serialization for a Python type, developers subclass `TypeTransformer` (or `AsyncTypeTransformer` for asynchronous operations) and override key methods.

**Core Methods of a `TypeTransformer`:**

*   `get_literal_type(self, t: Type[T]) -> LiteralType`: This method defines how a given Python type `t` maps to a Flyte `LiteralType`. For example, a custom object might map to a `LiteralType` with `SimpleType.STRUCT` and custom metadata.
*   `to_literal(self, ctx: FlyteContext, python_val: T, python_type: Type[T], expected: LiteralType) -> Literal`: This method converts a Python object `python_val` of `python_type` into a Flyte `Literal`. The `FlyteContext` provides access to utilities like the file system for data staging.
*   `to_python_value(self, ctx: FlyteContext, lv: Literal, expected_python_type: Type[T]) -> Optional[T]`: This method converts a Flyte `Literal` `lv` back into a Python object of `expected_python_type`.
*   `guess_python_type(self, literal_type: LiteralType) -> Type[T]`: This method attempts to reverse the `get_literal_type` operation, inferring the Python type from a Flyte `LiteralType`.
*   `assert_type(self, t: Type[T], v: T)`: This method performs runtime type validation, ensuring that a given value `v` conforms to the expected Python type `t`.
*   `from_binary_idl(self, binary_idl_object: Binary, expected_python_type: Type[T]) -> Optional[T]`: A utility method for deserializing from binary IDL formats, primarily MessagePack. This is crucial for efficient data transfer of complex objects.
*   `from_generic_idl(self, generic: Struct, expected_python_type: Type[T]) -> Optional[T]`: A utility method for deserializing from generic IDL (Protobuf Struct).

**Asynchronous Transformers:**

For operations that involve I/O (e.g., reading/writing to remote storage), `AsyncTypeTransformer` provides `async_to_literal` and `async_to_python_value` methods, allowing for non-blocking execution. The `TypeEngine` automatically handles the synchronization of these asynchronous calls when invoked from synchronous contexts.

### Built-in Custom Object Transformers

The `type_engine` package provides several pre-built `TypeTransformer` implementations for common Python data structures, demonstrating practical custom serialization patterns.

#### DataclassTransformer

The `DataclassTransformer` handles Python dataclasses, providing efficient serialization and deserialization.

*   **Serialization Mechanism:** Dataclasses are converted to and from MessagePack bytes using the `mashumaro` library. This binary representation is then embedded within a Flyte `Literal` as a `Binary` scalar.
*   **JSON Schema Integration:** The transformer attempts to extract a JSON Schema for the dataclass, which can be used for metadata, validation, and UI rendering. This is an experimental feature.
*   **Nested Types:** It recursively handles nested dataclasses, lists, and dictionaries defined within a dataclass.
*   **Integer Conversion:** Addresses a known issue where Protobuf's `Struct` type converts integers to doubles, ensuring that deserialized integers retain their original type.
*   **`DataClassJSONMixin` Support:** If a dataclass inherits from `DataClassJSONMixin`, the transformer leverages its `to_json` and `from_json` methods for serialization, offering more control over the process.

**Example:**

```python
from dataclasses import dataclass
from typing import List
# Assuming FlyteFile is imported from flytekit.types.file
from flytekit.types.file import FlyteFile

@dataclass
class MyCustomData:
    an_int: int
    a_string: str
    a_list_of_floats: List[float]
    a_file: FlyteFile # FlyteFile is also a dataclass internally

@task
def process_data(data: MyCustomData) -> MyCustomData:
    # 'data' is automatically deserialized into MyCustomData object
    print(f"Processing int: {data.an_int}, string: {data.a_string}")
    print(f"List of floats: {data.a_list_of_floats}")
    print(f"File path: {data.a_file.path}")
    # Perform operations
    data.an_int += 1
    return data # 'data' is automatically serialized back to a Literal
```

#### DictTransformer

The `DictTransformer` manages the serialization of Python dictionaries.

*   **Typed Dictionaries (`Dict[str, T]`):** For dictionaries with string keys and a consistent value type (e.g., `Dict[str, int]`), the transformer converts them into a Flyte `LiteralMap`. This is the preferred and most efficient way to represent structured dictionaries.
*   **Untyped Dictionaries (`dict`):** For generic dictionaries without specific type hints for values, or dictionaries with non-string keys, they are serialized into MessagePack bytes and stored as a `Binary` scalar within a `Literal` with `SimpleType.STRUCT`.
*   **Pickle Fallback:** The transformer can fall back to `FlytePickle` for serialization if MessagePack encoding fails and the `allow_pickle` annotation is explicitly set on the dictionary type. This is generally less efficient and should be used with caution.

**Example:**

```python
from typing import Dict, Any, Annotated

@task
def process_typed_dict(data: Dict[str, int]) -> Dict[str, int]:
    data["new_key"] = 100
    return data

@task
def process_untyped_dict(data: Dict[str, Any]) -> Dict[str, Any]:
    # This will be serialized as a STRUCT with MessagePack
    data["status"] = "processed"
    return data

# Example with allow_pickle (less common, for complex untyped dicts)
# from flytekit.core.annotation import FlyteAnnotation
# from collections import OrderedDict
# @task
# def process_pickled_dict(data: Annotated[Dict[str, Any], FlyteAnnotation(OrderedDict(allow_pickle=True))]) -> ...:
#     ...
```

#### ListTransformer

The `ListTransformer` handles Python lists with a single generic type (e.g., `List[int]`). It converts them into a Flyte `LiteralCollection`.

**Example:**

```python
from typing import List

@task
def process_list(numbers: List[int]) -> List[int]:
    return [n * 2 for n in numbers]
```

#### UnionTransformer

The `UnionTransformer` supports Python `Union` types, including `Optional` (which is a `Union` with `None`).

*   **Variant Matching:** During serialization, it attempts to match the Python value to one of the types in the `Union` and uses the corresponding transformer.
*   **Type Tagging:** To ensure correct deserialization, the `UnionTransformer` stores a "tag" (the name of the transformer used) within the `Literal`'s `Union` scalar. This tag guides the deserialization process.
*   **Ambiguity Resolution:** If a value could be validly serialized by multiple variants in a `Union` (e.g., `Union[int, float]` for an integer value), the transformer raises an error to prevent ambiguous conversions.

**Example:**

```python
from typing import Union, Optional

@task
def process_union(value: Union[int, str]) -> Union[int, str]:
    if isinstance(value, int):
        return value + 1
    return value + " processed"

@task
def process_optional(data: Optional[str]) -> Optional[str]:
    if data:
        return data.upper()
    return None
```

#### EnumTransformer

The `EnumTransformer` enables the use of Python `enum.Enum` types as task inputs and outputs. It specifically supports enums where values are strings.

**Example:**

```python
import enum

class Status(enum.Enum):
    PENDING = "pending"
    COMPLETED = "completed"
    FAILED = "failed"

@task
def update_status(current_status: Status) -> Status:
    if current_status == Status.PENDING:
        return Status.COMPLETED
    return current_status
```

#### ProtobufTransformer

The `ProtobufTransformer` handles `google.protobuf.Message` types, converting them to and from Flyte `LiteralType.STRUCT`. This is useful for direct integration with Protobuf messages.

#### FlytePickleTransformer: The Fallback Mechanism

The `FlytePickleTransformer` acts as a generic fallback for any Python type that does not have a specific `TypeTransformer` registered or cannot be handled by other built-in transformers.

*   **Mechanism:** It uses `cloudpickle` to serialize the Python object into a binary stream, which is then uploaded to a blob storage URI (e.g., S3, GCS). The `Literal` then stores a reference to this URI. During deserialization, the binary data is downloaded and unpickled.
*   **Limitations:**
    *   **Performance:** Pickling can be less efficient than structured binary formats like MessagePack, especially for large objects, due to higher serialization/deserialization overhead and network transfer.
    *   **Portability:** Pickled objects are Python-version specific and can break across different Python versions or environments.
    *   **Security:** Unpickling data from untrusted sources can pose security risks.
*   **Best Practice:** While convenient, relying on `FlytePickleTransformer` should be a last resort. For custom data structures, defining a dedicated `DataclassTransformer` (by using Python dataclasses) or a custom `TypeTransformer` is strongly recommended for better performance, portability, and type safety.

### Advanced Considerations and Best Practices

*   **Performance:** For large or frequently transferred custom objects, prioritize `DataclassTransformer` or `DictTransformer` (for untyped dicts) which leverage MessagePack for efficient binary serialization. Avoid `FlytePickleTransformer` unless absolutely necessary.
*   **Type Assertions:** `TypeTransformer.assert_type` provides runtime validation. Ensure your custom transformers implement this method robustly to catch type mismatches early.
*   **Error Handling:** `TypeTransformerFailedError` should be raised by transformers when a conversion cannot be performed due to type incompatibility or other issues.
*   **Debugging:** When encountering type conversion issues, inspect the `LiteralType` and `Literal` representations to understand how Flyte is interpreting your Python objects. The `TypeEngine.guess_python_type` can also help in understanding the inferred Python type from a Flyte `LiteralType`.
*   **When to Create a Custom Transformer:**
    *   When integrating with external libraries that have their own complex object models (e.g., custom ORM objects, specialized data structures) that cannot be easily represented as dataclasses or simple types.
    *   When specific serialization/deserialization logic is required that goes beyond what `mashumaro` or standard Python serialization offers.
    *   When fine-grained control over the `LiteralType` metadata or structure is needed.
<!--
key: summary_custom_python_object_serialization_69126556-c398-41af-b74d-beb1378702ab
type: summary_end

-->
<!--
code_unit: flytekit.core.type_engine.DataclassTransformer
code_unit_type: class
help_text: ''
key: example_df05b79b-6df2-4339-80b1-da6841ee0853
type: example

-->
<!--
code_unit: flytekit.core.type_engine.ProtobufTransformer
code_unit_type: class
help_text: ''
key: example_224bc528-09c2-4918-9baf-cfbcdbef4146
type: example

-->
<!--
code_unit: flytekit.types.pickle.pickle.FlytePickleTransformer
code_unit_type: class
help_text: ''
key: example_93a984b4-d1c7-487a-9958-07409732ca94
type: example

-->
<!--
code_unit: flytekit.types.error.error.ErrorTransformer
code_unit_type: class
help_text: ''
key: example_865cae54-c182-4eb3-bb99-3791c0c6ced3
type: example

-->