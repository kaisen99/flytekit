# peTransformer(t

This class serves as a base class for transforming Python native types to Flyte&#x27;s internal representation and vice versa. It defines the interface for converting between Python types and Flyte LiteralTypes, including methods for serialization and deserialization. Concrete implementations of this class are responsible for handling specific Python types and their corresponding Flyte representations.

## Attributes

- **name**: string
  - This returns the name

- **python_type**: Type[T]
  - This returns the python type

- **is_async**: bool = False
  - This returns whether the transformer is async

- **type_assertions_enabled**: bool
  - Indicates if the transformer wants type assertions to be enabled at the core type engine layer

## Constructors
def peTransformer(t(name: str, t: Type[T], enable_type_assertions: bool = True)
-  Base transformer type that should be implemented for every python native type that can be handled by flytekit
- **Parameters**

  - **name**: str
    - The name of the transformer.
  - **t**: Type[T]
    - The Python type that this transformer handles.
  - **enable_type_assertions**: bool
    - Whether to enable type assertions for this transformer.



