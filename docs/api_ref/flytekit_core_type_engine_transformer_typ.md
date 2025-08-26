# Transformer(Typ

This class, EnumTransformer, facilitates the conversion of Python&#x27;s enum.Enum type to Flyte&#x27;s LiteralType.EnumType. It provides methods for transforming enums to and from literals, handling string-valued enums, and inferring Python types from literal types. The class also includes type assertions to ensure values conform to the expected enum type.

## Constructors
```def Transformer(Typ()
```
-  Enables converting a python type enum.Enum to LiteralType.EnumType

```def Transformer(Typ()
```
-  Enables converting a python type enum.Enum to LiteralType.EnumType



## Methods
def get_literal_type(t: Type[T]) - > [LiteralType](flytekit_models_types_literaltype)
-  Converts a Python Enum type to a Flyte LiteralType.EnumType.
- **Parameters**

  - **t**: Type[T]
    - The Python Enum type to convert.

- **Return Value**:
**[LiteralType](flytekit_models_types_literaltype)**
  - A LiteralType representing the Enum.
def to_literal(ctx: [FlyteContext](flytekit_core_context_manager_flytecontext), python_val: enum.Enum, python_type: Type[T], expected: [LiteralType](flytekit_models_types_literaltype)) - > [Literal](flytekit_models_literals_literal)
-  Converts a Python Enum value to a Flyte Literal.
- **Parameters**

  - **ctx**: [FlyteContext](flytekit_core_context_manager_flytecontext)
    - The Flyte context.
  - **python_val**: enum.Enum
    - The Python Enum value to convert.
  - **python_type**: Type[T]
    - The Python type of the Enum.
  - **expected**: [LiteralType](flytekit_models_types_literaltype)
    - The expected Flyte LiteralType.

- **Return Value**:
**[Literal](flytekit_models_literals_literal)**
  - A Flyte Literal representing the Enum value.
def to_python_value(ctx: [FlyteContext](flytekit_core_context_manager_flytecontext), lv: [Literal](flytekit_models_literals_literal), expected_python_type: Type[T]) - > T
-  Converts a Flyte Literal to a Python Enum value.
- **Parameters**

  - **ctx**: [FlyteContext](flytekit_core_context_manager_flytecontext)
    - The Flyte context.
  - **lv**: [Literal](flytekit_models_literals_literal)
    - The Flyte Literal to convert.
  - **expected_python_type**: Type[T]
    - The expected Python type of the Enum.

- **Return Value**:
**T**
  - The Python Enum value.
def guess_python_type(literal_type: [LiteralType](flytekit_models_types_literaltype)) - > Type[enum.Enum]
-  Guesses the Python type from a Flyte LiteralType.
- **Parameters**

  - **literal_type**: [LiteralType](flytekit_models_types_literaltype)
    - The Flyte LiteralType to guess from.

- **Return Value**:
**Type[enum.Enum]**
  - The guessed Python Enum type.
def assert_type(t: Type[enum.Enum], v: T)
-  Asserts that a value is of the expected Enum type.
- **Parameters**

  - **t**: Type[enum.Enum]
    - The expected Python Enum type.
  - **v**: T
    - The value to assert.

