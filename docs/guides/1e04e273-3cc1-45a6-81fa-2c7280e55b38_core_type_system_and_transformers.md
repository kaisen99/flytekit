
<!--
help_text: ''
key: summary_core_type_system_and_transformers_37d0c91c-7677-4fca-9831-91442712bd0e
modules:
- flytekit.core.type_engine
questions_to_answer: []
type: summary

-->
The Core Type System and Transformers provide the foundational mechanism for converting Python native types to Flyte's internal literal representation and vice-versa. This system ensures that data can be seamlessly passed between tasks, workflows, and the Flyte platform, regardless of its original Python type.

## Type Transformers

At the heart of the type system are **Type Transformers**. These components define how specific Python types are converted to and from Flyte's `Literal` format. Each transformer is responsible for a particular Python type, handling its serialization to a `Literal` and deserialization back to a Python value.

### Base Type Transformers

The `TypeTransformer` is the abstract base class for all type transformers. It defines the core interface for type conversions:

*   `get_literal_type(python_type)`: Converts a Python type hint (e.g., `int`, `typing.List[str]`) into a Flyte `LiteralType` schema.
*   `to_literal(ctx, python_val, python_type, expected_literal_type)`: Serializes a Python value into a Flyte `Literal`.
*   `to_python_value(ctx, literal_value, expected_python_type)`: Deserializes a Flyte `Literal` into a Python value.
*   `assert_type(python_type, python_val)`: Performs type assertions to ensure the Python value matches the expected type.
*   `from_binary_idl(binary_idl_object, expected_python_type)`: Handles deserialization from binary representations, commonly MessagePack.

For operations that might involve I/O or network calls, the `AsyncTypeTransformer` extends `TypeTransformer` to support asynchronous conversions using `async_to_literal` and `async_to_python_value`. This is crucial for performance when dealing with large data transfers.

### Built-in Type Transformers

The system includes several pre-built transformers for common Python types:

*   **List Transformer (`ListTransformer`)**: Handles `typing.List[T]` types. It converts Python lists into Flyte `LiteralCollection`s and vice-versa. It supports only univariate lists (e.g., `List[int]`, not `List[Union[int, str]]`).
    *   **Usage**: Automatically applied when `list` or `typing.List` is used in type hints.
    *   **Example**: `typing.List[int]` will be transformed.

*   **Dictionary Transformer (`DictTransformer`)**: Manages `dict` types. It can transform typed dictionaries (e.g., `Dict[str, int]`) into Flyte `LiteralMap`s, where keys must be strings. Untyped dictionaries (`dict`) or dictionaries with non-string keys are serialized into a binary scalar literal using MessagePack.
    *   **Usage**: `Dict[str, float]` or `dict`.
    *   **Considerations**: For untyped dictionaries, the `allow_pickle` annotation can be used as a fallback for serialization if MessagePack fails, though MessagePack is preferred for performance and interoperability.

*   **Simple Transformer (`SimpleTransformer`)**: Provides a straightforward way to define transformers using lambda functions for `to_literal` and `from_literal` conversions. This reduces boilerplate for simple type mappings.
    *   **Usage**: Ideal for custom primitive types or simple wrappers.

*   **Protobuf Transformer (`ProtobufTransformer`)**: Converts `google.protobuf.message.Message` types to and from Flyte's `Literal` representation, typically using a `STRUCT` simple type. It includes metadata to store the original Protobuf type tag, enabling accurate deserialization.
    *   **Usage**: For tasks that exchange Protobuf messages.

*   **IO Transformers (`TextIOTransformer`, `BinaryIOTransformer`)**: Handle `typing.TextIO` and `typing.BinaryIO` respectively. These transformers convert file-like objects to and from Flyte `Blob` literals, managing the underlying file access (downloading/uploading data to/from storage).
    *   **Usage**: For reading/writing text or binary files within tasks.

*   **Union Transformer (`UnionTransformer`)**: Supports `typing.Union[T1, T2, ...]`. It attempts to convert a Python value to one of the union's constituent types. During deserialization, it uses a stored type tag to identify the correct variant.
    *   **Considerations**: Ambiguity can arise if a value can be validly converted to multiple types within the union (e.g., `Union[int, float]` for `1`). The system logs debug messages for such cases and raises an error if ambiguity cannot be resolved. It also handles `Optional[T]` as a special case of `Union[T, None]`.

*   **Dataclass Transformer (`DataclassTransformer`)**: Provides robust serialization and deserialization for Python dataclasses. It converts dataclasses to and from MessagePack bytes, transported as binary IDL objects. It also attempts to extract JSON Schema for the dataclass, which can be useful for UI/CLI auto-completion.
    *   **Serialization Lifecycle**: Dataclass attributes are made serializable (e.g., converting string paths to `FlyteFile` objects), then serialized to MessagePack bytes, and finally wrapped in a Flyte `Literal`.
    *   **Deserialization Lifecycle**: MessagePack bytes are converted back to a dataclass.
    *   **Performance**: Uses `mashumaro` for efficient MessagePack encoding/decoding.
    *   **Limitations**: Experimental schema support. Requires careful handling of nested Flyte types within dataclasses. An environment variable `FLYTE_USE_OLD_DC_FORMAT` can be used to revert to an older JSON-based format, but MessagePack is the recommended approach.

*   **Enum Transformer (`EnumTransformer`)**: Enables conversion of `enum.Enum` types. It supports string-valued enums, converting them to `LiteralType.EnumType` and back.
    *   **Usage**: For defining fixed sets of string values.

### Restricted Type Transformer

The `RestrictedTypeTransformer` is a special transformer that prevents certain Python types from being used as task inputs or outputs. Any attempt to convert a restricted type to or from a `Literal` will raise a `RestrictedTypeError`. This is used for internal types or types that are not meant for direct data passing.

## The Type Engine

The `TypeEngine` is the central orchestrator for all type transformations. It acts as a registry for `TypeTransformer` instances and provides the public API for converting between Python types and Flyte `Literal`s.

### Transformer Registration

Type transformers are registered with the `TypeEngine` using the `register` method. This associates a Python type with its corresponding transformer. The engine also supports lazy loading of transformers for external libraries (e.g., TensorFlow, PyTorch, Pandas) to avoid unnecessary dependencies.

### Type Conversion Flow

The `TypeEngine` provides methods for both directions of conversion:

*   **Python Type to Flyte Literal Type (`to_literal_type`)**: Given a Python type hint, the `TypeEngine` finds the appropriate transformer and uses it to generate the corresponding Flyte `LiteralType` schema. This schema is crucial for defining task and workflow interfaces. It also incorporates metadata from `FlyteAnnotation`s if present in the type hint.

*   **Python Value to Flyte Literal (`to_literal`, `async_to_literal`)**: When a Python value needs to be passed as an input or output, the `TypeEngine` converts it into a Flyte `Literal`.
    1.  It identifies the correct transformer for the Python type.
    2.  It performs type assertions to ensure the value matches the expected type.
    3.  It invokes the transformer's `to_literal` or `async_to_literal` method.
    4.  It can also calculate a hash for the literal if a `HashMethod` annotation is present, enabling content-addressable caching.
    5.  For large literals, the system can offload the actual data to remote storage, storing only a URI in the `Literal` and handling the download transparently during deserialization.

*   **Flyte Literal to Python Value (`to_python_value`, `async_to_python_value`)**: When a Flyte `Literal` is received (e.g., as a task input), the `TypeEngine` converts it back to a Python value.
    1.  It first checks for offloaded literals and downloads the actual data if necessary.
    2.  It determines the expected Python type (either explicitly provided or guessed from the `LiteralType`).
    3.  It invokes the transformer's `to_python_value` or `async_to_python_value` method.

### Handling Collections

The `TypeEngine` also facilitates conversions for collections of values:

*   **`literal_map_to_kwargs`**: Converts a `LiteralMap` (a dictionary of Flyte `Literal`s) into a Python dictionary of keyword arguments, suitable for passing to a Python function. This process involves asynchronously converting each `Literal` in the map to its corresponding Python value.
*   **`dict_to_literal_map`**: Converts a Python dictionary into a `LiteralMap`. This is used when Python dictionary outputs need to be represented as Flyte `LiteralMap`s.

### Type Resolution and Guessing

The `TypeEngine` includes `guess_python_type` and `guess_python_types` methods. These attempt to infer the original Python type from a Flyte `LiteralType` schema. While useful for dynamic scenarios (e.g., inspecting remote task outputs), explicit type hints are always recommended for robustness and clarity.

### Error Handling

The `TypeTransformerFailedError` is a specific exception raised when a type transformation fails, providing detailed information about the mismatch or conversion issue.

## Literals Resolver

The `LiteralsResolver` is a utility class designed to simplify working with `LiteralMap`s, especially in interactive or remote environments. It allows developers to access values from a `LiteralMap` and convert them to Python native types on demand.

*   **Dynamic Type Resolution**: When retrieving a value, `LiteralsResolver` can use a provided Python type hint or attempt to guess the type from the associated `Variable` map (if available).
*   **Caching**: Once a `Literal` is converted to a Python value, it is cached, preventing redundant conversions.
*   **Convenience**: Provides dictionary-like access (`resolver[key]`) and a `get(key, as_type)` method for explicit type specification.

## Performance Considerations

*   **Asynchronous Operations**: The use of `AsyncTypeTransformer` and `asyncio.gather` for batching conversions (e.g., in `ListTransformer`, `DictTransformer`, `literal_map_to_kwargs`) significantly improves performance by allowing concurrent I/O operations.
*   **Binary Serialization**: MessagePack is the preferred serialization format for complex types like dataclasses and untyped dictionaries due to its efficiency and compactness compared to JSON.
*   **`BatchSize` Annotation**: For `FlyteDirectory` types (and potentially other collection types), the `BatchSize` annotation can be used to control the number of files downloaded or uploaded concurrently, optimizing resource usage and preventing out-of-memory errors for very large directories.

## Best Practices

*   **Explicit Type Hints**: Always use clear and explicit Python type hints in your task and workflow definitions. This enables the `TypeEngine` to perform accurate and efficient type conversions.
*   **Dataclasses for Complex Structures**: For user-defined complex data structures, leverage Python dataclasses. The `DataclassTransformer` provides robust and performant serialization.
*   **Avoid Ambiguous Unions**: While `Union` types are supported, be mindful of potential ambiguities where a value could match multiple types within the union. Design your types to minimize such overlaps.
*   **`LiteralsResolver` for Dynamic Outputs**: When dealing with outputs from remote executions or dynamic literal maps, use `LiteralsResolver` to safely and efficiently convert `Literal`s to Python values, especially when the exact output types might not be known upfront.
<!--
key: summary_core_type_system_and_transformers_37d0c91c-7677-4fca-9831-91442712bd0e
type: summary_end

-->