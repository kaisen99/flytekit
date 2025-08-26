# JSONIteratorTransformer

This class transforms between an iterator of JSON objects and a JSONL file format. It enables the serialization and deserialization of JSON data streams for storage and retrieval. The class implements the AsyncTypeTransformer interface, facilitating asynchronous data transformations.

## Attributes

- **JSON_ITERATOR_FORMAT**: string = &quot;jsonl&quot;
  - The format of the JSON iterator.

- **JSON_ITERATOR_METADATA**: string = &quot;json iterator&quot;
  - The metadata for the JSON iterator.

## Constructors
```def JSONIteratorTransformer()
```
-  Initializes the JSONIteratorTransformer with a name and a type.



## Methods
@classmethod
def get_literal_type(t: Type[Iterator[JSON]]) - > [LiteralType](flytekit_models_types_literaltype)
-  This method returns the literal type for an iterator of JSON objects, specifying the format as JSONL and a single blob dimensionality.
- **Parameters**

  - **t**: Type[Iterator[JSON]]
    - The type of the iterator of JSON objects.

- **Return Value**:
**[LiteralType](flytekit_models_types_literaltype)**
  - A LiteralType object representing the JSON iterator.
@classmethod
def async_to_literal(ctx: [FlyteContext](flytekit_core_context_manager_flytecontext), python_val: Iterator[JSON], python_type: Type[Iterator[JSON]], expected: [LiteralType](flytekit_models_types_literaltype)) - > [Literal](flytekit_models_literals_literal)
-  Converts an iterator of JSON objects into a literal blob in JSONL format. It writes the JSON objects to a temporary file and then uploads it.
- **Parameters**

  - **ctx**: [FlyteContext](flytekit_core_context_manager_flytecontext)
    - The Flyte context.
  - **python_val**: Iterator[JSON]
    - The iterator of JSON objects to convert.
  - **python_type**: Type[Iterator[JSON]]
    - The expected Python type, which is an iterator of JSON objects.
  - **expected**: [LiteralType](flytekit_models_types_literaltype)
    - The expected literal type.

- **Return Value**:
**[Literal](flytekit_models_literals_literal)**
  - A Literal object representing the JSON iterator as a blob.
@classmethod
def async_to_python_value(ctx: [FlyteContext](flytekit_core_context_manager_flytecontext), lv: [Literal](flytekit_models_literals_literal), expected_python_type: Type[Iterator[JSON]]) - > [JSONIterator](flytekit_types_iterator_json_iterator_jsoniterator)
-  Converts a literal blob (assumed to be in JSONL format) back into a JSON iterator.
- **Parameters**

  - **ctx**: [FlyteContext](flytekit_core_context_manager_flytecontext)
    - The Flyte context.
  - **lv**: [Literal](flytekit_models_literals_literal)
    - The literal value to convert.
  - **expected_python_type**: Type[Iterator[JSON]]
    - The expected Python type, which is an iterator of JSON objects.

- **Return Value**:
**[JSONIterator](flytekit_types_iterator_json_iterator_jsoniterator)**
  - A JSONIterator object that reads from the blob.
@classmethod
def guess_python_type(literal_type: [LiteralType](flytekit_models_types_literaltype)) - > Type[Iterator[JSON]]
-  Guesses the Python type (Iterator[JSON]) from a given LiteralType if it matches the JSONL format and metadata.
- **Parameters**

  - **literal_type**: [LiteralType](flytekit_models_types_literaltype)
    - The literal type to inspect.

- **Return Value**:
**Type[Iterator[JSON]]**
  - The Python type Iterator[JSON] if the literal type matches.
