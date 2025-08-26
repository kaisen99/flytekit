# Promise

This class serves as a wrapper for task outputs within a workflow, handling the distinction between compilation and local execution. It enables the use of methods like `with_overrides` on task results and facilitates conditional logic. The `Promise` class also manages attribute access and pathing for accessing nested data structures within workflows.

## Attributes

- **_var**: string
  - Name of the variable bound with this promise

- **_promise_ready**: bool = True
  - Indicates whether the promise is ready (i.e., not a reference and the value is available).

- **_val**: [Union](flytekit_models_literals_union)[[NodeOutput](flytekit_core_promise_nodeoutput), _literals_models.Literal]
  - If the promise is ready, this holds the actual evaluated value in Flyte&#x27;s type system.

- **_ref**: [NodeOutput](flytekit_core_promise_nodeoutput)
  - If the promise is NOT READY / Incomplete, then it maps to the origin node that owns the promise.

- **_attr_path**: List[[Union](flytekit_models_literals_union)[str, int]] = []
  - The attribute path the promise will be resolved with.

- **_type**: typing.Optional[_type_models.LiteralType]
  - The Flyte type of the promise.

## Constructors
def Promise(var: string, val: [Union](flytekit_models_literals_union)[[NodeOutput](flytekit_core_promise_nodeoutput), _literals_models.Literal], type: typing.Optional[_type_models.LiteralType] = None)
-  This object is a wrapper and exists for three main reasons. Let&#x27;s assume we&#x27;re dealing with a task like ::

        @task
        def t1() - &gt; (int, str): ...

       #. Handling the duality between compilation and local execution - when the task function is run in a local execution
          mode inside a workflow function, a Python integer and string are produced. When the task is being compiled as
          part of the workflow, the task call creates a Node instead, and the task returns two Promise objects that
          point to that Node.
       #. One needs to be able to call ::

             x = t1().with_overrides(...)

          If the task returns an integer or a ``(int, str)`` tuple like ``t1`` above, calling ``with_overrides`` on the
          result would throw an error. This Promise object adds that.
       #. Assorted handling for conditionals.
- **Parameters**

  - **var**: string
    - Name of the variable bound with this promise
  - **val**: [Union](flytekit_models_literals_union)[[NodeOutput](flytekit_core_promise_nodeoutput), _literals_models.Literal]
    - The value of the promise, which can be a NodeOutput or a Literal.
  - **type**: typing.Optional[_type_models.LiteralType]
    - The type of the promise, which can be a LiteralType or None.



## Methods
@classmethod
def with_var(new_var: str) - > [Promise](flytekit_core_promise_promise)
-  Creates a new Promise with a different variable name.
- **Parameters**

  - **new_var**: str
    - The new variable name for the promise.

- **Return Value**:
**[Promise](flytekit_core_promise_promise)**
  - A new Promise instance with the updated variable name.
```@classmethod
def is_ready()
```
-  Returns if the Promise is READY (is not a reference and the val is actually ready).

- **Return Value**:
**bool**
  - True if the promise is ready, False otherwise.
```@classmethod
def val()
```
-  If the promise is ready then this holds the actual evaluate value in Flyte&#x27;s type system.

- **Return Value**:
**_literals_models.Literal**
  - The evaluated value of the promise.
```@classmethod
def ref()
```
-  If the promise is NOT READY / Incomplete, then it maps to the origin node that owns the promise.

- **Return Value**:
**[NodeOutput](flytekit_core_promise_nodeoutput)**
  - The NodeOutput object representing the reference.
```@classmethod
def var()
```
-  Name of the variable bound with this promise.

- **Return Value**:
**str**
  - The variable name.
```@classmethod
def attr_path()
```
-  The attribute path the promise will be resolved with.

- **Return Value**:
**List[[Union](flytekit_models_literals_union)[str, int]]**
  - The list of attribute keys forming the path.
```@classmethod
def eval()
```
-  Evaluates the promise to its primitive value. Only applicable for promises that are ready and hold primitive types.

- **Return Value**:
**Any**
  - The primitive value of the promise.
@classmethod
def is_(v: bool) - > [ComparisonExpression](flytekit_models_core_condition_comparisonexpression)
-  Creates a comparison expression for equality with a given boolean value.
- **Parameters**

  - **v**: bool
    - The boolean value to compare against.

- **Return Value**:
**[ComparisonExpression](flytekit_models_core_condition_comparisonexpression)**
  - A ComparisonExpression object.
```@classmethod
def is_false()
```
-  Creates a comparison expression to check if the promise is equal to False.

- **Return Value**:
**[ComparisonExpression](flytekit_models_core_condition_comparisonexpression)**
  - A ComparisonExpression object for checking inequality with False.
```@classmethod
def is_true()
```
-  Creates a comparison expression to check if the promise is equal to True.

- **Return Value**:
**[ComparisonExpression](flytekit_models_core_condition_comparisonexpression)**
  - A ComparisonExpression object for checking equality with True.
```@classmethod
def is_none()
```
-  Creates a comparison expression to check if the promise is equal to None.

- **Return Value**:
**[ComparisonExpression](flytekit_models_core_condition_comparisonexpression)**
  - A ComparisonExpression object for checking equality with None.
@classmethod
def with_overrides(node_name: Optional[str] = None, aliases: Optional[Dict[str, str]] = None, requests: Optional[[Resources](flytekit_models_task_resources)] = None, limits: Optional[[Resources](flytekit_models_task_resources)] = None, timeout: Optional[[Union](flytekit_models_literals_union)[int, datetime.timedelta, object]] = Node.TIMEOUT_OVERRIDE_SENTINEL, retries: Optional[int] = None, interruptible: Optional[bool] = None, name: Optional[str] = None, task_config: Optional[Any] = None, container_image: Optional[str] = None, accelerator: Optional[[BaseAccelerator](flytekit_extras_accelerators_baseaccelerator)] = None, cache: Optional[[Union](flytekit_models_literals_union)[bool, [Cache](flytekit_core_cache_cache)]] = None) - > [Promise](flytekit_core_promise_promise)
-  Applies overrides to the underlying node if the promise is not ready.
- **Parameters**

  - **node_name**: Optional[str]
    - Optional name for the node.
  - **aliases**: Optional[Dict[str, str]]
    - Optional dictionary of aliases.
  - **requests**: Optional[[Resources](flytekit_models_task_resources)]
    - Optional resource requests.
  - **limits**: Optional[[Resources](flytekit_models_task_resources)]
    - Optional resource limits.
  - **timeout**: Optional[[Union](flytekit_models_literals_union)[int, datetime.timedelta, object]]
    - Optional timeout value.
  - **retries**: Optional[int]
    - Optional number of retries.
  - **interruptible**: Optional[bool]
    - Optional flag for interruptibility.
  - **name**: Optional[str]
    - Optional name for the task.
  - **task_config**: Optional[Any]
    - Optional task configuration.
  - **container_image**: Optional[str]
    - Optional container image to use.
  - **accelerator**: Optional[[BaseAccelerator](flytekit_extras_accelerators_baseaccelerator)]
    - Optional accelerator configuration.
  - **cache**: Optional[[Union](flytekit_models_literals_union)[bool, [Cache](flytekit_core_cache_cache)]]
    - Optional cache configuration.

- **Return Value**:
**[Promise](flytekit_core_promise_promise)**
  - The promise object itself, after applying overrides.
```@classmethod
def deepcopy()
```
-  Creates a deep copy of the promise object.

- **Return Value**:
**[Promise](flytekit_core_promise_promise)**
  - A deep copy of the promise.
