# ralsResolver(col

This class, LiteralsResolver, is designed to facilitate the conversion of Flyte Literal objects into their corresponding Python native representations. It primarily serves to resolve LiteralMaps, especially within the FlyteRemote environment. Key features include caching converted values and utilizing a TypeEngine for type conversion, with the option to specify Python types for elements within the map.

## Attributes

- **native_values**: Dict[str, typing.Any] = {}
  - This will return the native values that have been converted from literals.

- **variable_map**: Optional[Dict[str, _interface_models.Variable]] = None
  - This map should be basically one side (either input or output) of the Flyte TypedInterface model and is used to guess the Python type through the TypeEngine if a Python type is not specified by the user. TypeEngine guessing is flaky though, so calls to get() should specify the as_type parameter when possible.

- **literals**: typing.Dict[str, [Literal](flytekit_models_literals_literal)] = None
  - A Python map of strings to Flyte Literal models.

## Constructors
def ralsResolver(col(literals: typing.Dict[str, [Literal](flytekit_models_literals_literal)], variable_map: Optional[Dict[str, _interface_models.Variable]] = None, ctx: Optional[[FlyteContext](flytekit_core_context_manager_flytecontext)] = None)
-  LiteralsResolver is a helper class meant primarily for use with the FlyteRemote experience or any other situation
    where you might be working with LiteralMaps. This object allows the caller to specify the Python type that should
    correspond to an element of the map.
- **Parameters**

  - **literals**: typing.Dict[str, [Literal](flytekit_models_literals_literal)]
    - A Python map of strings to Flyte Literal models.
  - **variable_map**: Optional[Dict[str, _interface_models.Variable]]
    - This map should be basically one side (either input or output) of the Flyte
  TypedInterface model and is used to guess the Python type through the TypeEngine if a Python type is not
  specified by the user. TypeEngine guessing is flaky though, so calls to get() should specify the as_type
  parameter when possible.
  - **ctx**: Optional[[FlyteContext](flytekit_core_context_manager_flytecontext)]
    - None



