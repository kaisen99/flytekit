# FlyteSchemaTransformer

This class serves as a transformer for FlyteSchema objects, enabling the conversion between Python objects and Flyte&#x27;s schema representation. It supports various data types and facilitates the handling of schema-based data within the Flyte ecosystem. The class implements the AsyncTypeTransformer interface, providing asynchronous methods for literal conversion and type inference.

## Constructors
```def FlyteSchemaTransformer()
```
-  Initializes the FlyteSchemaTransformer with a name and the FlyteSchema type.



## Methods
@classmethod
def assert_type(t: Type[[FlyteSchema](flytekit_types_schema_types_flyteschema)], v: typing.Any) - > None
-  Asserts that the given value is compatible with the expected FlyteSchema type. It checks if automatic conversion is possible.
- **Parameters**

  - **t**: Type[[FlyteSchema](flytekit_types_schema_types_flyteschema)]
    - The expected FlyteSchema type.
  - **v**: typing.Any
    - The value to check.

- **Return Value**:
**None**
  - This method does not return any value.
@classmethod
def get_literal_type(t: Type[[FlyteSchema](flytekit_types_schema_types_flyteschema)]) - > [LiteralType](flytekit_models_types_literaltype)
-  Returns the LiteralType for a given FlyteSchema type.
- **Parameters**

  - **t**: Type[[FlyteSchema](flytekit_types_schema_types_flyteschema)]
    - The FlyteSchema type.

- **Return Value**:
**[LiteralType](flytekit_models_types_literaltype)**
  - The LiteralType representation of the FlyteSchema.
@classmethod
def async_to_literal(ctx: [FlyteContext](flytekit_core_context_manager_flytecontext), python_val: [FlyteSchema](flytekit_types_schema_types_flyteschema), python_type: Type[[FlyteSchema](flytekit_types_schema_types_flyteschema)], expected: [LiteralType](flytekit_models_types_literaltype)) - > [Literal](flytekit_models_literals_literal)
-  Asynchronously converts a Python value (FlyteSchema or compatible) into a Flyte Literal.
- **Parameters**

  - **ctx**: [FlyteContext](flytekit_core_context_manager_flytecontext)
    - The Flyte context.
  - **python_val**: [FlyteSchema](flytekit_types_schema_types_flyteschema)
    - The Python value to convert.
  - **python_type**: Type[[FlyteSchema](flytekit_types_schema_types_flyteschema)]
    - The expected Python type, which is FlyteSchema.
  - **expected**: [LiteralType](flytekit_models_types_literaltype)
    - The expected LiteralType.

- **Return Value**:
**[Literal](flytekit_models_literals_literal)**
  - The resulting Flyte Literal.
@classmethod
def dict_to_flyte_schema(dict_obj: typing.Dict[str, str], expected_python_type: Type[[FlyteSchema](flytekit_types_schema_types_flyteschema)]) - > [FlyteSchema](flytekit_types_schema_types_flyteschema)
-  Converts a dictionary representation of a FlyteSchema into a FlyteSchema object.
- **Parameters**

  - **dict_obj**: typing.Dict[str, str]
    - The dictionary containing the FlyteSchema data, including &#x27;remote_path&#x27;.
  - **expected_python_type**: Type[[FlyteSchema](flytekit_types_schema_types_flyteschema)]
    - The expected FlyteSchema type to instantiate.

- **Return Value**:
**[FlyteSchema](flytekit_types_schema_types_flyteschema)**
  - The FlyteSchema object created from the dictionary.
@classmethod
def from_binary_idl(binary_idl_object: [Binary](flytekit_models_literals_binary), expected_python_type: Type[[FlyteSchema](flytekit_types_schema_types_flyteschema)]) - > [FlyteSchema](flytekit_types_schema_types_flyteschema)
-  Deserializes a FlyteSchema from a binary IDL object (e.g., MessagePack).
- **Parameters**

  - **binary_idl_object**: [Binary](flytekit_models_literals_binary)
    - The binary IDL object containing the serialized FlyteSchema.
  - **expected_python_type**: Type[[FlyteSchema](flytekit_types_schema_types_flyteschema)]
    - The expected FlyteSchema type for deserialization.

- **Return Value**:
**[FlyteSchema](flytekit_types_schema_types_flyteschema)**
  - The deserialized FlyteSchema object.
@classmethod
def from_generic_idl(generic: Struct, expected_python_type: Type[[FlyteSchema](flytekit_types_schema_types_flyteschema)]) - > [FlyteSchema](flytekit_types_schema_types_flyteschema)
-  Deserializes a FlyteSchema from a generic IDL object (e.g., Protobuf Struct from Flyte Console).
- **Parameters**

  - **generic**: Struct
    - The generic IDL object (protobuf Struct) containing the serialized FlyteSchema.
  - **expected_python_type**: Type[[FlyteSchema](flytekit_types_schema_types_flyteschema)]
    - The expected FlyteSchema type for deserialization.

- **Return Value**:
**[FlyteSchema](flytekit_types_schema_types_flyteschema)**
  - The deserialized FlyteSchema object.
@classmethod
def async_to_python_value(ctx: [FlyteContext](flytekit_core_context_manager_flytecontext), lv: [Literal](flytekit_models_literals_literal), expected_python_type: Type[[FlyteSchema](flytekit_types_schema_types_flyteschema)]) - > [FlyteSchema](flytekit_types_schema_types_flyteschema)
-  Asynchronously converts a Flyte Literal into a Python FlyteSchema object.
- **Parameters**

  - **ctx**: [FlyteContext](flytekit_core_context_manager_flytecontext)
    - The Flyte context.
  - **lv**: [Literal](flytekit_models_literals_literal)
    - The Flyte Literal to convert.
  - **expected_python_type**: Type[[FlyteSchema](flytekit_types_schema_types_flyteschema)]
    - The expected Python type, which is FlyteSchema.

- **Return Value**:
**[FlyteSchema](flytekit_types_schema_types_flyteschema)**
  - The Python FlyteSchema object.
@classmethod
def guess_python_type(literal_type: [LiteralType](flytekit_models_types_literaltype)) - > Type[[FlyteSchema](flytekit_types_schema_types_flyteschema)]
-  Infers the Python type (specifically, a parameterized FlyteSchema) from a LiteralType&#x27;s schema definition.
- **Parameters**

  - **literal_type**: [LiteralType](flytekit_models_types_literaltype)
    - The LiteralType object containing schema information.

- **Return Value**:
**Type[[FlyteSchema](flytekit_types_schema_types_flyteschema)]**
  - The inferred Python type, which is a FlyteSchema with specific column types.
