# Interface

This class represents a Python-native interface object, similar to inspect.signature but simplified. It defines input and output types for functions or tasks, including default values for inputs. The class supports operations like removing or adding inputs and outputs, and provides access to input/output metadata.

## Attributes

- **_inputs**: Dict[str, Tuple[Type, Any]] = {}
  - Input variables and their types as a dictionary

- **_outputs**: Dict[str, Type] = {}
  - Output variables and their types as a dictionary

- **_output_tuple_name**: Optional[str] = None
  - The name of a typing.NamedTuple when the task or workflow returns one.

- **_output_tuple_class**: Type[collections.namedtuple] = None
  - This class can be used in two different places. For multivariate-return entities this class is used to rewrap the outputs so that our with_overrides function can work. For manual node creation, it&#x27;s used during local execution as something that can be dereferenced.

- **_docstring**: Optional[[Docstring](flytekit_core_docstring_docstring)] = None
  - Docstring of the annotated @task or @workflow from which the interface derives from.

## Constructors
def Interface(inputs: [Union](flytekit_models_literals_union)[Optional[Dict[str, Type]], Optional[Dict[str, Tuple[Type, Any]]]] = None, outputs: [Union](flytekit_models_literals_union)[Optional[Dict[str, Type]], Optional[Dict[str, Optional[Type]]]] = None, output_tuple_name: Optional[str] = None, docstring: Optional[[Docstring](flytekit_core_docstring_docstring)] = None)
-  Initializes the Interface object.

:param outputs: Output variables and their types as a dictionary
:param inputs: Map of input name to either a tuple where the first element is the python type, and the second
    value is the default, or just a single value which is the python type. The latter case is used by tasks
    for which perhaps a default value does not make sense. For consistency, we turn it into a tuple.
:param output_tuple_name: This is used to store the name of a typing.NamedTuple when the task or workflow
    returns one. This is also used as a proxy for better or for worse for the presence of a tuple return type,
    primarily used when handling one-element NamedTuples.
:param docstring: Docstring of the annotated @task or @workflow from which the interface derives from.
- **Parameters**

  - **inputs**: [Union](flytekit_models_literals_union)[Optional[Dict[str, Type]], Optional[Dict[str, Tuple[Type, Any]]]]
    - Map of input name to either a tuple where the first element is the python type, and the second
    value is the default, or just a single value which is the python type. The latter case is used by tasks
    for which perhaps a default value does not make sense. For consistency, we turn it into a tuple.
  - **outputs**: [Union](flytekit_models_literals_union)[Optional[Dict[str, Type]], Optional[Dict[str, Optional[Type]]]]
    - Output variables and their types as a dictionary
  - **output_tuple_name**: Optional[str]
    - This is used to store the name of a typing.NamedTuple when the task or workflow
    returns one. This is also used as a proxy for better or for worse for the presence of a tuple return type,
    primarily used when handling one-element NamedTuples.
  - **docstring**: Optional[[Docstring](flytekit_core_docstring_docstring)]
    - Docstring of the annotated @task or @workflow from which the interface derives from.

def Interface(inputs: [Union](flytekit_models_literals_union)[Optional[Dict[str, Type]], Optional[Dict[str, Tuple[Type, Any]]]] = None, outputs: [Union](flytekit_models_literals_union)[Optional[Dict[str, Type]], Optional[Dict[str, Optional[Type]]]] = None, output_tuple_name: Optional[str] = None, docstring: Optional[[Docstring](flytekit_core_docstring_docstring)] = None)
-  Initialize the Interface object.

        :param outputs: Output variables and their types as a dictionary
        :param inputs: Map of input name to either a tuple where the first element is the python type, and the second
            value is the default, or just a single value which is the python type. The latter case is used by tasks
            for which perhaps a default value does not make sense. For consistency, we turn it into a tuple.
        :param output_tuple_name: This is used to store the name of a typing.NamedTuple when the task or workflow
            returns one. This is also used as a proxy for better or for worse for the presence of a tuple return type,
            primarily used when handling one-element NamedTuples.
        :param docstring: Docstring of the annotated @task or @workflow from which the interface derives from.
- **Parameters**

  - **inputs**: [Union](flytekit_models_literals_union)[Optional[Dict[str, Type]], Optional[Dict[str, Tuple[Type, Any]]]]
    - Map of input name to either a tuple where the first element is the python type, and the second
            value is the default, or just a single value which is the python type. The latter case is used by tasks
            for which perhaps a default value does not make sense. For consistency, we turn it into a tuple.
  - **outputs**: [Union](flytekit_models_literals_union)[Optional[Dict[str, Type]], Optional[Dict[str, Optional[Type]]]]
    - Output variables and their types as a dictionary
  - **output_tuple_name**: Optional[str]
    - This is used to store the name of a typing.NamedTuple when the task or workflow
            returns one. This is also used as a proxy for better or for worse for the presence of a tuple return type,
            primarily used when handling one-element NamedTuples.
  - **docstring**: Optional[[Docstring](flytekit_core_docstring_docstring)]
    - Docstring of the annotated @task or @workflow from which the interface derives from.



## Methods
```@classmethod
def output_tuple()
```
-  Returns the namedtuple class for the outputs.

- **Return Value**:
**Type[collections.namedtuple]**
  - The namedtuple class for the outputs.
```@classmethod
def output_tuple_name()
```
-  Returns the name of the output tuple.

- **Return Value**:
**Optional[str]**
  - The name of the output tuple.
```@classmethod
def inputs()
```
-  Returns a dictionary of input names and their types.

- **Return Value**:
**Dict[str, type]**
  - A dictionary of input names and their types.
```@classmethod
def output_names()
```
-  Returns a list of output names.

- **Return Value**:
**Optional[List[str]]**
  - A list of output names.
```@classmethod
def inputs_with_defaults()
```
-  Returns a dictionary of input names and their types along with default values.

- **Return Value**:
**Dict[str, Tuple[Type, Any]]**
  - A dictionary of input names and their types along with default values.
```@classmethod
def default_inputs_as_kwargs()
```
-  Returns a dictionary of default input values as keyword arguments.

- **Return Value**:
**Dict[str, Any]**
  - A dictionary of default input values as keyword arguments.
```@classmethod
def outputs()
```
-  Returns a dictionary of output names and their types.

- **Return Value**:
**typing.Dict[str, type]**
  - A dictionary of output names and their types.
```@classmethod
def docstring()
```
-  Returns the docstring of the interface.

- **Return Value**:
**Optional[[Docstring](flytekit_core_docstring_docstring)]**
  - The docstring of the interface.
@classmethod
def remove_inputs(vars: Optional[List[str]] = None) - > [Interface](flytekit_core_interface_interface)
-  This method is useful in removing some variables from the Flyte backend inputs specification, as these are
        implicit local only inputs or will be supplied by the library at runtime. For example, spark-session etc
    It creates a new instance of interface with the requested variables removed
- **Parameters**

  - **vars**: Optional[List[str]]
    - A list of input variable names to remove.

- **Return Value**:
**[Interface](flytekit_core_interface_interface)**
  - A new Interface instance with the specified inputs removed.
@classmethod
def with_inputs(extra_inputs: Dict[str, Type]) - > [Interface](flytekit_core_interface_interface)
-  Use this to add additional inputs to the interface. This is useful for adding additional implicit inputs that
        are added without the user requesting for them
- **Parameters**

  - **extra_inputs**: Dict[str, Type]
    - A dictionary of additional inputs to add.

- **Return Value**:
**[Interface](flytekit_core_interface_interface)**
  - A new Interface instance with the additional inputs.
@classmethod
def with_outputs(extra_outputs: Dict[str, Type]) - > [Interface](flytekit_core_interface_interface)
-  This method allows addition of extra outputs are expected from a task specification
- **Parameters**

  - **extra_outputs**: Dict[str, Type]
    - A dictionary of additional outputs to add.

- **Return Value**:
**[Interface](flytekit_core_interface_interface)**
  - A new Interface instance with the additional outputs.
