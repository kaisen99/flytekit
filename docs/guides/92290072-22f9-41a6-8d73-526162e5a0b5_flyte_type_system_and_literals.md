
<!--
help_text: ''
key: summary_flyte_type_system_and_literals_18202ad1-5dc1-4013-b4c7-ab9b6f4e7c6e
modules:
- flytekit.models.literals
- flytekit.models.core.types
- flytekit.models.interface
questions_to_answer: []
type: summary

-->
Flyte Type System and Literals

Flyte's type system and literal representation are fundamental to defining and exchanging data within workflows. The type system ensures data consistency and enables robust serialization and deserialization across different programming languages and execution environments. Literals are the concrete, serialized representations of data values that flow through Flyte tasks and workflows.

### The Flyte Type System

The Flyte type system defines the structure and nature of data. Every input and output in a Flyte task or workflow is associated with a specific type, enabling the platform to validate data, manage storage, and facilitate interoperability.

At its core, the type system is built around a flexible `LiteralType` concept, which can represent various data structures.

#### Primitive Types

Primitive types represent basic, atomic data values. These are encapsulated within the `Primitive` class:

*   **Integers**: Whole numbers.
*   **Floats**: Floating-point numbers.
*   **Strings**: Textual data.
*   **Booleans**: True or false values.
*   **Datetimes**: Specific points in time, including date and time components.
*   **Durations**: Time spans.

#### Structured Data Types

Flyte supports structured data, which is common in data science and machine learning workflows.

*   **Schema**: Represents strongly typed tabular data, similar to a dataframe. A `Schema` literal includes a URI pointing to the data's location and a `SchemaType` that defines the columns and their types. This allows Flyte to understand the structure of the data without needing to load its entire contents.
*   **StructuredDataset**: Similar to `Schema`, `StructuredDataset` also represents structured data, often used for datasets that might have more complex metadata or storage characteristics. It includes a URI and `StructuredDatasetMetadata`, which contains a `StructuredDatasetType`.

#### Binary and File-like Data Types

Flyte handles binary data and files efficiently, often by offloading them to external storage.

*   **Blob**: Represents binary data stored externally, identified by a URI. A `Blob` literal includes `BlobMetadata`, which specifies the `BlobType`.
    *   **`BlobType`**: Defines the format (e.g., "csv", "parquet") and dimensionality (`SINGLE` for a single file, `MULTIPART` for a directory or collection of files) of the blob.
*   **Binary**: Represents raw binary data directly embedded within the literal, along with an optional `tag` for identification. This is suitable for smaller binary payloads.

#### Collection and Map Types

Flyte supports standard collection types for organizing data.

*   **Collection Type**: Represents a list of values, where all elements in the list must be of the same `LiteralType`.
*   **Map Type**: Represents a dictionary where keys are strings and values are of a consistent `LiteralType`.

#### Advanced Types

*   **Union**: Represents a tagged union, allowing a variable to hold values of different, predefined `LiteralType`s. The `Union` literal stores the actual `value` (a `Literal`) and its `stored_type`.
*   **Enum**: Defines a type with a fixed set of allowed string values. The `EnumType` class lists these permissible values.
*   **Void**: Represents the absence of a value, analogous to `None` in Python.

### Flyte Literals: Representing Data Values

Literals are the concrete, serialized data values that flow through Flyte workflows. They are the runtime representation of the data defined by the type system.

#### The `Literal` Class

The `Literal` class is the primary container for any data value in Flyte. It is a "oneof" type, meaning a `Literal` instance can hold exactly one of the following:

*   **`scalar`**: A single, atomic value.
*   **`collection`**: A list of `Literal` objects.
*   **`map`**: A dictionary mapping string keys to `Literal` objects.

Additionally, a `Literal` can carry:

*   **`hash`**: An optional hash for caching purposes.
*   **`metadata`**: A dictionary of string-to-string metadata.
*   **`offloaded_metadata`**: Information about literals that are too large to be embedded directly and are stored externally.

#### The `Scalar` Class

The `Scalar` class represents a single, atomic data value. Like `Literal`, it is a "oneof" type, encapsulating various specific scalar types:

*   **`primitive`**: An instance of the `Primitive` class (integer, float, string, boolean, datetime, duration).
*   **`blob`**: An instance of the `Blob` class (offloaded binary data).
*   **`binary`**: An instance of the `Binary` class (embedded binary data).
*   **`schema`**: An instance of the `Schema` class (structured tabular data).
*   **`union`**: An instance of the `Union` class (tagged union value).
*   **`none_type`**: An instance of the `Void` class (representing `None`).
*   **`error`**: Represents an error literal.
*   **`generic`**: A generic structured value, typically a Protobuf `Struct`.
*   **`structured_dataset`**: An instance of the `StructuredDataset` class.

#### `LiteralCollection` and `LiteralMap`

*   **`LiteralCollection`**: Holds a list of `Literal` objects. This is used when the `Literal` itself represents a collection.
*   **`LiteralMap`**: Holds a dictionary where keys are strings and values are `Literal` objects. This is used when the `Literal` represents a map.

#### `LiteralOffloadedMetadata`

For very large literals, Flyte can offload the actual data to external storage (e.g., S3, GCS). `LiteralOffloadedMetadata` provides information about this offloaded data, including:

*   **`uri`**: The URI where the offloaded literal data is stored.
*   **`size_bytes`**: The size of the offloaded data in bytes.
*   **`inferred_type`**: The `LiteralType` of the offloaded data, which can be inferred during the offloading process.

This mechanism is crucial for performance and scalability when dealing with large datasets that would be impractical to embed directly in workflow definitions or execution messages.

### Data Binding and Flow

Data flows through Flyte workflows via bindings, which connect variables (inputs/outputs) to specific values or to outputs from other nodes.

#### `Binding` and `BindingData`

*   **`Binding`**: Represents a connection between a variable name (`var`) and its corresponding data.
*   **`BindingData`**: Specifies the actual data to be used for a binding. It is a "oneof" type and can be:
    *   **`scalar`**: A direct `Scalar` value.
    *   **`collection`**: A `BindingDataCollection`, allowing for nested lists of bindings.
    *   **`map`**: A `BindingDataMap`, allowing for nested maps of bindings.
    *   **`promise`**: A reference to an output from another node in the workflow. This is a key mechanism for connecting tasks and propagating data dependencies.

The `BindingData` class includes a `to_literal_model()` method, which converts the binding data into a `Literal`. However, this conversion will raise an error if the `BindingData` contains a `promise`, as promises represent future values that are not yet concrete literals.

#### `BindingDataCollection` and `BindingDataMap`

These classes allow for complex, nested data structures to be bound:

*   **`BindingDataCollection`**: A list of `BindingData` items.
*   **`BindingDataMap`**: A dictionary mapping string keys to `BindingData` items.

These recursive structures enable flexible data flow definitions within workflows.

#### `Variable` and `TypedInterface`

*   **`Variable`**: Defines a named input or output within a task or workflow. It specifies the `type` (a `LiteralType`) and a `description`. Variables can also include `artifact_partial_id` and `artifact_tag` for managing data artifacts.
*   **`TypedInterface`**: Defines the complete input and output signature for a task or workflow. It contains two maps: `inputs` and `outputs`, where each map consists of variable names mapped to `Variable` objects. This interface is critical for type checking and ensuring compatibility between connected components.

#### `Parameter` and `ParameterMap`

*   **`Parameter`**: A specialized `Variable` used for inputs to a launch plan. Parameters can have a `default` `Literal` value, or they can be marked as `required`. This allows for flexible execution, where some inputs can be optional with predefined defaults. Parameters can also bind to `artifact_query` or `artifact_id` for dynamic or static artifact resolution.
*   **`ParameterMap`**: A collection of `Parameter` objects, defining all the configurable inputs for a launch plan.

### Practical Considerations

*   **Type Safety**: Flyte's strong type system ensures that data passed between tasks and workflows conforms to expected schemas, preventing common runtime errors related to data mismatches.
*   **Serialization**: The `to_flyte_idl()` and `from_flyte_idl()` methods on all these classes facilitate seamless serialization to and from Flyte's internal Protobuf representation, enabling cross-language compatibility and efficient data transfer.
*   **Large Data Handling**: The `Blob`, `Schema`, `StructuredDataset`, and `LiteralOffloadedMetadata` mechanisms are designed to handle large datasets efficiently by storing them externally and passing only references (URIs) within the workflow graph.
*   **Immutability**: Once a `Literal` is created, its value is generally considered immutable. This simplifies reasoning about data flow and enables caching.
*   **Caching**: The `hash` property on the `Literal` class is used by Flyte's caching system to determine if a task's output can be reused, based on its inputs.
<!--
key: summary_flyte_type_system_and_literals_18202ad1-5dc1-4013-b4c7-ab9b6f4e7c6e
type: summary_end

-->
<!--
code_unit: flytekit.models.literals.Literal
code_unit_type: class
help_text: ''
key: example_809de791-2a7a-4c55-b092-bff1c7fcabb7
type: example

-->
<!--
code_unit: flytekit.models.literals.Scalar
code_unit_type: class
help_text: ''
key: example_2c3bd058-fde4-4f65-af6a-f4c14ebb5050
type: example

-->
<!--
code_unit: flytekit.models.interface.TypedInterface
code_unit_type: class
help_text: ''
key: example_38c09e21-ec80-4efc-ad4f-c4eb539f0426
type: example

-->
<!--
code_unit: flytekit.models.interface.Variable
code_unit_type: class
help_text: ''
key: example_90ba41a4-5df3-40ac-a2ec-40456a06f7ba
type: example

-->