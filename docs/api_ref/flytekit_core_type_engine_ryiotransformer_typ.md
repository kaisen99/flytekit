# ryIOTransformer(Typ

This class serves as a transformer for handling binary input/output streams (BinaryIO). It facilitates the conversion between BinaryIO objects and Flyte&#x27;s Literal representation, specifically BlobType. The class defines methods for obtaining literal types and converting between Python values and literals, though the `to_literal` method is not implemented.

## Constructors
```def ryIOTransformer(Typ()
```
-  Handler for BinaryIO

```def ryIOTransformer(Typ()
```
-  Handler for BinaryIO



## Methods
```def _blob_type()
```
-  Returns a BlobType object representing the binary data.

- **Return Value**:
**_core_types.BlobType**
  - A BlobType object.
def get_literal_type(t: Type[typing.BinaryIO]) - > [LiteralType](flytekit_models_types_literaltype)
-  Returns a LiteralType object for BinaryIO.
- **Parameters**

  - **t**: Type[typing.BinaryIO]
    - The type to get the literal type for.

- **Return Value**:
**[LiteralType](flytekit_models_types_literaltype)**
  - A LiteralType object.
def to_literal(ctx: [FlyteContext](flytekit_core_context_manager_flytecontext), python_val: typing.BinaryIO, python_type: Type[typing.BinaryIO], expected: [LiteralType](flytekit_models_types_literaltype))
-  Converts a Python BinaryIO value to a Literal.
- **Parameters**

  - **ctx**: [FlyteContext](flytekit_core_context_manager_flytecontext)
    - The Flyte context.
  - **python_val**: typing.BinaryIO
    - The Python BinaryIO value.
  - **python_type**: Type[typing.BinaryIO]
    - The Python type.
  - **expected**: [LiteralType](flytekit_models_types_literaltype)
    - The expected LiteralType.

def to_python_value(ctx: [FlyteContext](flytekit_core_context_manager_flytecontext), lv: [Literal](flytekit_models_literals_literal), expected_python_type: Type[typing.BinaryIO]) - > typing.BinaryIO
-  Converts a Literal to a Python BinaryIO value.
- **Parameters**

  - **ctx**: [FlyteContext](flytekit_core_context_manager_flytecontext)
    - The Flyte context.
  - **lv**: [Literal](flytekit_models_literals_literal)
    - The Literal value.
  - **expected_python_type**: Type[typing.BinaryIO]
    - The expected Python type.

- **Return Value**:
**typing.BinaryIO**
  - The Python BinaryIO value.
