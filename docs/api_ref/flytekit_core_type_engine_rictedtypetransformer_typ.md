# rictedTypeTransformer(Typ

This class, RestrictedTypeTransformer, prevents the conversion of certain types to and from literals, effectively restricting their use as inputs or outputs in tasks and workflows. It inherits from TypeTransformer and implements methods that raise a RestrictedTypeError, ensuring that the transformation to and from literals is not permitted. This class is designed to enforce type restrictions within a specific context.

## Constructors
def rictedTypeTransformer(Typ(name: str, t: Type[T])
-  Initializes the RestrictedTypeTransformer with a name and a type.
- **Parameters**

  - **name**: str
    - The name of the transformer.
  - **t**: Type[T]
    - The type to be transformed.

def rictedTypeTransformer(Typ(name: str, t: Type[T])
-  Initializes the RestrictedTypeTransformer with a name and a Python type.
- **Parameters**

  - **name**: str
    - The name of the transformer.
  - **t**: Type[T]
    - The Python type the transformer is for.



## Methods
def get_literal_type(t: Optional[Type[T]] = None) - > [LiteralType](flytekit_models_types_literaltype)
-  Retrieves the literal type for the restricted type. This method always raises a RestrictedTypeError.
- **Parameters**

  - **t**: Optional[Type[T]]
    - The Python type to get the literal type for. Defaults to None.

- **Return Value**:
**[LiteralType](flytekit_models_types_literaltype)**
  - The literal type representation.
def to_literal(ctx: [FlyteContext](flytekit_core_context_manager_flytecontext), python_val: T, python_type: Type[T], expected: [LiteralType](flytekit_models_types_literaltype)) - > [Literal](flytekit_models_literals_literal)
-  Converts a Python value to a literal. This method always raises a RestrictedTypeError.
- **Parameters**

  - **ctx**: [FlyteContext](flytekit_core_context_manager_flytecontext)
    - The Flyte context.
  - **python_val**: T
    - The Python value to convert.
  - **python_type**: Type[T]
    - The Python type of the value.
  - **expected**: [LiteralType](flytekit_models_types_literaltype)
    - The expected literal type.

- **Return Value**:
**[Literal](flytekit_models_literals_literal)**
  - The literal representation of the Python value.
def to_python_value(ctx: [FlyteContext](flytekit_core_context_manager_flytecontext), lv: [Literal](flytekit_models_literals_literal), expected_python_type: Type[T]) - > T
-  Converts a literal to a Python value. This method always raises a RestrictedTypeError.
- **Parameters**

  - **ctx**: [FlyteContext](flytekit_core_context_manager_flytecontext)
    - The Flyte context.
  - **lv**: [Literal](flytekit_models_literals_literal)
    - The literal value to convert.
  - **expected_python_type**: Type[T]
    - The expected Python type.

- **Return Value**:
**T**
  - The Python value representation of the literal.
