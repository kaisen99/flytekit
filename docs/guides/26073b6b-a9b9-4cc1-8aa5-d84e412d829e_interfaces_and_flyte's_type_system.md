
<!--
help_text: ''
key: summary_interfaces_and_flyte's_type_system_0d70fcc0-5a24-4825-920d-b69189cd4f49
modules:
- flytekit.core.interface
- flytekit.core.docstring
questions_to_answer: []
type: summary

-->
Interfaces and Flyte's Type System

Flyte's robust type system is fundamental to its ability to orchestrate and execute complex data and ML workflows reliably. At the core of this system is the concept of an `Interface`, which provides a canonical representation of a task or workflow's inputs and outputs. This abstraction allows Flyte to understand, validate, and serialize data flowing between different components, ensuring type safety and enabling powerful features like automatic data marshaling and UI generation.

### The Interface Class

The `Interface` class serves as a simplified, Python-native object akin to `inspect.signature`, specifically tailored for Flyte's needs. It captures the essential contract of any callable entity (like a task or workflow) by defining its expected inputs and outputs, including their types and any default values.

**Defining Inputs and Outputs**

An `Interface` object is initialized with dictionaries for `inputs` and `outputs`.

*   **Inputs**: Represented as a dictionary where keys are input variable names (which must be valid Python identifiers) and values are either:
    *   A Python type (e.g., `str`, `int`, `typing.List[int]`).
    *   A tuple `(type, default_value)`, indicating the Python type and an optional default value for that input.
    Flyte automatically infers these from the type hints and default arguments of your Python functions decorated with `@task` or `@workflow`.

    The `inputs` property provides a dictionary of input names to their types, while `inputs_with_defaults` exposes the full `(type, default_value)` tuples. Default values can also be accessed as a keyword argument dictionary via `default_inputs_as_kwargs`.

*   **Outputs**: Represented as a dictionary where keys are output variable names and values are their Python types.

**Example of Interface Representation:**

Consider a Python function:

```python
from typing import NamedTuple, List

class MyOutput(NamedTuple):
    result: int
    message: str

def my_task(x: int, y: str = "default") -> MyOutput:
    # ... task logic ...
    return MyOutput(result=x, message=y)
```

Flyte would infer an `Interface` for `my_task` that conceptually looks like:

*   `inputs`: `{"x": int, "y": str}`
*   `inputs_with_defaults`: `{"x": (int, None), "y": (str, "default")}`
*   `outputs`: `{"result": int, "message": str}`
*   `output_tuple_name`: `"MyOutput"`

**Handling Multivariate Returns and NamedTuples**

When a task or workflow returns multiple values, Flyte encourages the use of `typing.NamedTuple` for clarity and type safety. The `Interface` class supports this through the `output_tuple_name` property, which stores the name of the `NamedTuple` class used for the return type.

The `output_tuple` property provides a dynamically created `collections.namedtuple` subclass. This internal `Output` class is crucial for:
*   Rewrapping multivariate outputs during local execution, allowing them to be dereferenced.
*   Enabling features like `with_overrides` on task outputs, which is useful for advanced control flow.

**Programmatic Interface Manipulation**

While Flyte typically infers interfaces automatically, the `Interface` class provides methods to programmatically modify input and output specifications. This is particularly useful for library developers or in scenarios where implicit inputs/outputs are added by the Flytekit framework itself.

*   `remove_inputs(vars: Optional[List[str]]) -> Interface`: Creates a new `Interface` instance with specified input variables removed. This is useful for inputs that are implicitly supplied by the Flyte runtime (e.g., `spark_session` for Spark tasks) and should not be exposed as user-defined inputs in the Flyte backend.
*   `with_inputs(extra_inputs: Dict[str, Type]) -> Interface`: Creates a new `Interface` instance with additional inputs. This allows for injecting implicit inputs that are not part of the user's function signature but are required for execution. An error is raised if an `extra_input` key already exists.
*   `with_outputs(extra_outputs: Dict[str, Type]) -> Interface`: Creates a new `Interface` instance with additional outputs. This is used when a task or workflow is expected to produce outputs beyond those explicitly defined in its return signature, often for internal Flyte mechanisms. An error is raised if an `extra_output` key already exists.

These methods return new `Interface` instances, ensuring immutability of the original object.

### Integrating with Docstrings for Rich Metadata

The `Interface` class integrates with the `Docstring` utility to extract and store rich metadata from Python docstrings. This metadata is crucial for enhancing the user experience in the Flyte console and for making workflows more self-documenting.

**The Docstring Utility**

The `Docstring` utility parses standard Python docstrings (including those following common conventions like Sphinx or Google style) to extract:
*   `short_description`: The first line of the docstring.
*   `long_description`: The main body of the docstring.
*   `input_descriptions`: A dictionary mapping input parameter names to their descriptions (e.g., from `@param` or `:param` tags).
*   `output_descriptions`: A dictionary mapping output parameter names to their descriptions (e.g., from `@returns` or `:returns` tags).

Flyte automatically populates the `docstring` property of an `Interface` object by parsing the docstring of the decorated `@task` or `@workflow` function. This allows the Flyte UI to display comprehensive information about each input and output, improving discoverability and understanding of your workflow components.

**Best Practices for Docstrings:**

To leverage this feature fully, provide detailed docstrings for your tasks and workflows, including descriptions for all inputs and outputs.

```python
from flytekit import task
from typing import NamedTuple

class TaskOutput(NamedTuple):
    sum_val: int
    product_val: int

@task
def calculate_metrics(
    x: int,
    y: int = 10,
) -> TaskOutput:
    """
    Calculates the sum and product of two integers.

    This task demonstrates how to use docstrings to provide rich metadata
    for inputs and outputs, which is visible in the Flyte UI.

    :param x: The first integer value.
    :param y: The second integer value. Defaults to 10.
    :returns: A NamedTuple containing the sum and product.
    :rtype: TaskOutput
    """
    return TaskOutput(sum_val=x + y, product_val=x * y)
```

### Practical Considerations

*   **Automatic Inference:** For most users, Flyte automatically infers the `Interface` from your Python function signatures and type hints. You typically do not need to instantiate `Interface` objects directly.
*   **Input Name Validation:** Input names must be valid Python identifiers. The `Interface` class enforces this during initialization, raising a `ValueError` for invalid names.
*   **Extensibility:** The `Interface` class provides a stable and consistent way for Flyte to interact with user code, enabling powerful extensions and integrations without requiring users to manually define complex schemas.
<!--
key: summary_interfaces_and_flyte's_type_system_0d70fcc0-5a24-4825-920d-b69189cd4f49
type: summary_end

-->
<!--
code_unit: flytekit.core.interface.Interface
code_unit_type: class
help_text: ''
key: example_c9a28e5f-a572-455c-b280-4bbefd5d8d3e
type: example

-->
<!--
code_unit: flytekit.core.docstring.Docstring
code_unit_type: class
help_text: ''
key: example_6d1af66c-add3-42d3-959d-d499e7f10f19
type: example

-->