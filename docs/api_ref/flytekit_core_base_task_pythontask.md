# PythonTask

This class serves as the base for all Python tasks within the system, providing a foundation for task definition and execution. It defines the interface for Python-native tasks, handling input/output type conversions and execution context. Key features include environment variable support, deck output control, and methods for pre- and post-execution hooks.

## Attributes

- **task_type**: str
  - defines a unique task-type for every new extension. If a backend plugin is required then
                this has to be done in-concert with the backend plugin identifier

- **name**: str
  - A unique name for the task instantiation. This is unique for every instance of task.

- **task_config**: Optional[T]
  - Configuration for the task. This is used to configure the specific plugin that handles this
                task

- **interface**: Optional[[Interface](flytekit_core_interface_interface)]
  - A python native typed interface ``(inputs) - &gt; outputs`` that declares the
                signature of the task

- **environment**: Optional[Dict[str, str]]
  - Any environment variables that should be supplied during the
                execution of the task. Supplied as a dictionary of key/value pairs

- **disable_deck**: Optional[bool]
  - (deprecated) If true, this task will not output deck html file

- **enable_deck**: Optional[bool]
  - If true, this task will output deck html file

- **deck_fields**: Optional[Tuple[[DeckField](flytekit_deck_deck_deckfield), ...]] = (
            DeckField.SOURCE_CODE,
            DeckField.DEPENDENCIES,
            DeckField.TIMELINE,
            DeckField.INPUT,
            DeckField.OUTPUT,
        )
  - Tuple of decks to be
                generated for this task. Valid values can be selected from fields of ``flytekit.deck.DeckField`` enum

- **python_interface**: [Interface](flytekit_core_interface_interface)
  - Returns this task&#x27;s python interface.

- **task_config**: Optional[T]
  - Returns the user-specified task config which is used for plugin-specific handling of the task.

- **_outputs_interface**: Dict[Any, [Variable](flytekit_models_interface_variable)]
  - Returns the names and python types as a dictionary for the outputs of this task.

- **environment**: Dict[str, str]
  - Any environment variables that supplied during the execution of the task.

- **disable_deck**: bool
  - If true, this task will not output deck html file

- **enable_deck**: bool
  - If true, this task will output deck html file

- **deck_fields**: List[[DeckField](flytekit_deck_deck_deckfield)]
  - If not empty, this task will output deck html file for the specified decks

## Constructors
def PythonTask(task_type: str, name: str, task_config: Optional[T] = None, interface: Optional[[Interface](flytekit_core_interface_interface)] = None, environment: Optional[Dict[str, str]] = None, disable_deck: Optional[bool] = None, enable_deck: Optional[bool] = None, deck_fields: Optional[Tuple[[DeckField](flytekit_deck_deck_deckfield)]] = (
            DeckField.SOURCE_CODE,
            DeckField.DEPENDENCIES,
            DeckField.TIMELINE,
            DeckField.INPUT,
            DeckField.OUTPUT,
        ))
-  Args:
            task_type (str): defines a unique task-type for every new extension. If a backend plugin is required then
                this has to be done in-concert with the backend plugin identifier
            name (str): A unique name for the task instantiation. This is unique for every instance of task.
            task_config (T): Configuration for the task. This is used to configure the specific plugin that handles this
                task
            interface (Optional[Interface]): A python native typed interface ``(inputs) - &gt; outputs`` that declares the
                signature of the task
            environment (Optional[Dict[str, str]]): Any environment variables that should be supplied during the
                execution of the task. Supplied as a dictionary of key/value pairs
            disable_deck (bool): (deprecated) If true, this task will not output deck html file
            enable_deck (bool): If true, this task will output deck html file
            deck_fields (Tuple[DeckField]): Tuple of decks to be
                generated for this task. Valid values can be selected from fields of ``flytekit.deck.DeckField`` enum
- **Parameters**

  - **task_type**: str
    - defines a unique task-type for every new extension. If a backend plugin is required then
                this has to be done in-concert with the backend plugin identifier
  - **name**: str
    - A unique name for the task instantiation. This is unique for every instance of task.
  - **task_config**: Optional[T]
    - Configuration for the task. This is used to configure the specific plugin that handles this
                task
  - **interface**: Optional[[Interface](flytekit_core_interface_interface)]
    - A python native typed interface ``(inputs) - &gt; outputs`` that declares the
                signature of the task
  - **environment**: Optional[Dict[str, str]]
    - Any environment variables that should be supplied during the
                execution of the task. Supplied as a dictionary of key/value pairs
  - **disable_deck**: Optional[bool]
    - (deprecated) If true, this task will not output deck html file
  - **enable_deck**: Optional[bool]
    - If true, this task will output deck html file
  - **deck_fields**: Optional[Tuple[[DeckField](flytekit_deck_deck_deckfield)]]
    - Tuple of decks to be
                generated for this task. Valid values can be selected from fields of ``flytekit.deck.DeckField`` enum

def PythonTask(task_type: str, name: str, task_config: Optional[T], interface: Optional[[Interface](flytekit_core_interface_interface)], environment: Optional[Dict[str, str]], disable_deck: Optional[bool], enable_deck: Optional[bool], deck_fields: Optional[Tuple[[DeckField](flytekit_deck_deck_deckfield), ...]] = (
            DeckField.SOURCE_CODE,
            DeckField.DEPENDENCIES,
            DeckField.TIMELINE,
            DeckField.INPUT,
            DeckField.OUTPUT,
        ))
-  Args:
            task_type (str): defines a unique task-type for every new extension. If a backend plugin is required then
                this has to be done in-concert with the backend plugin identifier
            name (str): A unique name for the task instantiation. This is unique for every instance of task.
            task_config (T): Configuration for the task. This is used to configure the specific plugin that handles this
                task
            interface (Optional[Interface]): A python native typed interface ``(inputs) - &gt; outputs`` that declares the
                signature of the task
            environment (Optional[Dict[str, str]]): Any environment variables that should be supplied during the
                execution of the task. Supplied as a dictionary of key/value pairs
            disable_deck (bool): (deprecated) If true, this task will not output deck html file
            enable_deck (bool): If true, this task will output deck html file
            deck_fields (Tuple[DeckField]): Tuple of decks to be
                generated for this task. Valid values can be selected from fields of ``flytekit.deck.DeckField`` enum
- **Parameters**

  - **task_type**: str
    - defines a unique task-type for every new extension. If a backend plugin is required then
                this has to be done in-concert with the backend plugin identifier
  - **name**: str
    - A unique name for the task instantiation. This is unique for every instance of task.
  - **task_config**: Optional[T]
    - Configuration for the task. This is used to configure the specific plugin that handles this
                task
  - **interface**: Optional[[Interface](flytekit_core_interface_interface)]
    - A python native typed interface ``(inputs) - &gt; outputs`` that declares the
                signature of the task
  - **environment**: Optional[Dict[str, str]]
    - Any environment variables that should be supplied during the
                execution of the task. Supplied as a dictionary of key/value pairs
  - **disable_deck**: Optional[bool]
    - (deprecated) If true, this task will not output deck html file
  - **enable_deck**: Optional[bool]
    - If true, this task will output deck html file
  - **deck_fields**: Optional[Tuple[[DeckField](flytekit_deck_deck_deckfield), ...]]
    - Tuple of decks to be
                generated for this task. Valid values can be selected from fields of ``flytekit.deck.DeckField`` enum



## Methods
```@classmethod
def python_interface()
```
-  Returns this task&#x27;s python interface.

- **Return Value**:
**[Interface](flytekit_core_interface_interface)**
  - this task&#x27;s python interface.
```@classmethod
def task_config()
```
-  Returns the user-specified task config which is used for plugin-specific handling of the task.

- **Return Value**:
**Optional[T]**
  - the user-specified task config which is used for plugin-specific handling of the task.
@classmethod
def get_type_for_input_var(k: str, v: Any) - > Type[Any]
-  Returns the python type for an input variable by name.
- **Parameters**

  - **k**: str
    - input variable name
  - **v**: Any
    - input variable value

- **Return Value**:
**Type[Any]**
  - the python type for an input variable by name.
@classmethod
def get_type_for_output_var(k: str, v: Any) - > Type[Any]
-  Returns the python type for the specified output variable by name.
- **Parameters**

  - **k**: str
    - output variable name
  - **v**: Any
    - output variable value

- **Return Value**:
**Type[Any]**
  - the python type for the specified output variable by name.
```@classmethod
def get_input_types()
```
-  Returns the names and python types as a dictionary for the inputs of this task.

- **Return Value**:
**Dict[str, type]**
  - the names and python types as a dictionary for the inputs of this task.
```@classmethod
def construct_node_metadata()
```
-  Used when constructing the node that encapsulates this task as part of a broader workflow definition.

- **Return Value**:
**_workflow_model.NodeMetadata**
  - NodeMetadata for the task
@classmethod
def compile(ctx: [FlyteContext](flytekit_core_context_manager_flytecontext)) - > Optional[[Union](flytekit_models_literals_union)[Tuple[[Promise](flytekit_core_promise_promise)], [Promise](flytekit_core_promise_promise), [VoidPromise](flytekit_core_promise_voidpromise)]]
-  Generates a node that encapsulates this task in a workflow definition.
- **Parameters**

  - **ctx**: [FlyteContext](flytekit_core_context_manager_flytecontext)
    - Flyte context

- **Return Value**:
**Optional[[Union](flytekit_models_literals_union)[Tuple[[Promise](flytekit_core_promise_promise)], [Promise](flytekit_core_promise_promise), [VoidPromise](flytekit_core_promise_voidpromise)]]**
  - A node that encapsulates this task in a workflow definition.
@classmethod
def dispatch_execute(ctx: [FlyteContext](flytekit_core_context_manager_flytecontext), input_literal_map: _literal_models.LiteralMap) - > [Union](flytekit_models_literals_union)[_literal_models.LiteralMap, _dynamic_job.DynamicJobSpec, Coroutine]
-  This method translates Flyte&#x27;s Type system based input values and invokes the actual call to the executor
        This method is also invoked during runtime.

        * ``VoidPromise`` is returned in the case when the task itself declares no outputs.
        * ``Literal Map`` is returned when the task returns either one more outputs in the declaration. Individual outputs
          may be none
        * ``DynamicJobSpec`` is returned when a dynamic workflow is executed
- **Parameters**

  - **ctx**: [FlyteContext](flytekit_core_context_manager_flytecontext)
    - Flyte context
  - **input_literal_map**: _literal_models.LiteralMap
    - Input LiteralMap for the task

- **Return Value**:
**[Union](flytekit_models_literals_union)[_literal_models.LiteralMap, _dynamic_job.DynamicJobSpec, Coroutine]**
  - The result of the task execution, which can be a LiteralMap, DynamicJobSpec, or a Coroutine.
@classmethod
def pre_execute(user_params: Optional[[ExecutionParameters](flytekit_core_context_manager_executionparameters)]) - > Optional[[ExecutionParameters](flytekit_core_context_manager_executionparameters)]
-  This is the method that will be invoked directly before executing the task method and before all the inputs
        are converted. One particular case where this is useful is if the context is to be modified for the user process
        to get some user space parameters. This also ensures that things like SparkSession are already correctly
        setup before the type transformers are called

        This should return either the same context of the mutated context
- **Parameters**

  - **user_params**: Optional[[ExecutionParameters](flytekit_core_context_manager_executionparameters)]
    - User parameters for the execution.

- **Return Value**:
**Optional[[ExecutionParameters](flytekit_core_context_manager_executionparameters)]**
  - The potentially modified execution parameters.
```@classmethod
def execute()
```
-  This method will be invoked to execute the task.

- **Return Value**:
**Any**
  - The result of the task execution.
@classmethod
def post_execute(user_params: Optional[[ExecutionParameters](flytekit_core_context_manager_executionparameters)], rval: Any) - > Any
-  Post execute is called after the execution has completed, with the user_params and can be used to clean-up,
        or alter the outputs to match the intended tasks outputs. If not overridden, then this function is a No-op

        Args:
            rval is returned value from call to execute
            user_params: are the modified user params as created during the pre_execute step
- **Parameters**

  - **user_params**: Optional[[ExecutionParameters](flytekit_core_context_manager_executionparameters)]
    - User parameters used during pre-execution.
  - **rval**: Any
    - The return value from the execute method.

- **Return Value**:
**Any**
  - The potentially modified return value.
```@classmethod
def environment()
```
-  Any environment variables that supplied during the execution of the task.

- **Return Value**:
**Dict[str, str]**
  - Environment variables for the task.
```@classmethod
def disable_deck()
```
-  If true, this task will not output deck html file

- **Return Value**:
**bool**
  - True if deck output is disabled, False otherwise.
```@classmethod
def enable_deck()
```
-  If true, this task will output deck html file

- **Return Value**:
**bool**
  - True if deck output is enabled, False otherwise.
```@classmethod
def deck_fields()
```
-  If not empty, this task will output deck html file for the specified decks

- **Return Value**:
**List[[DeckField](flytekit_deck_deck_deckfield)]**
  - List of deck fields to be generated.
