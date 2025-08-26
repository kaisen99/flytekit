# SklearnTypeTransformer

This class transforms scikit-learn models to and from Flyte&#x27;s Literal type. It enables the serialization and deserialization of scikit-learn models, allowing them to be used within Flyte workflows. The class utilizes joblib for model persistence and interacts with Flyte&#x27;s file access for storage and retrieval.



## Methods
@classmethod
def get_literal_type(t: Type[T]) - > [LiteralType](flytekit_models_types_literaltype)
-  Returns the literal type for a given Python type. This method is used to specify how a Python type should be represented as a literal type in the system. It returns a LiteralType object with a BlobType that specifies the format and dimensionality.
- **Parameters**

  - **t**: Type[T]
    - The Python type to get the literal type for.

- **Return Value**:
**[LiteralType](flytekit_models_types_literaltype)**
  - The LiteralType representing the blob type.
@classmethod
def to_literal(ctx: [FlyteContext](flytekit_core_context_manager_flytecontext), python_val: T, python_type: Type[T], expected: [LiteralType](flytekit_models_types_literaltype)) - > [Literal](flytekit_models_literals_literal)
-  Converts a Python value to a Literal. This method is used to serialize a Python object into a format that can be stored or transmitted. It saves the Python object to a local file using joblib and then uploads it to remote storage, returning a Literal object with the blob&#x27;s URI.
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
  - The Literal representation of the Python value.
@classmethod
def to_python_value(ctx: [FlyteContext](flytekit_core_context_manager_flytecontext), lv: [Literal](flytekit_models_literals_literal), expected_python_type: Type[T]) - > T
-  Converts a Literal to a Python value. This method is used to deserialize a Literal object back into a Python object. It downloads the data from the blob&#x27;s URI and loads it using joblib.
- **Parameters**

  - **ctx**: [FlyteContext](flytekit_core_context_manager_flytecontext)
    - The Flyte context.
  - **lv**: [Literal](flytekit_models_literals_literal)
    - The Literal to convert.
  - **expected_python_type**: Type[T]
    - The expected Python type.

- **Return Value**:
**T**
  - The Python value converted from the Literal.
@classmethod
def guess_python_type(literal_type: [LiteralType](flytekit_models_types_literaltype)) - > Type[T]
-  Guesses the Python type from a LiteralType. This method is used to infer the Python type based on the properties of the LiteralType, specifically checking for blob format and dimensionality.
- **Parameters**

  - **literal_type**: [LiteralType](flytekit_models_types_literaltype)
    - The LiteralType to guess the Python type from.

- **Return Value**:
**Type[T]**
  - The guessed Python type.
