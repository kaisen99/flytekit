# pTaskResolver(T

This class specializes in resolving and loading MapTasks, particularly those created with partial subtasks. It addresses the interface interpolation challenges that arise when MapTasks are created using nested partial subtasks, ensuring correct interface reconstruction at runtime. It utilizes a resolver to load the actual task object and reconstructs the interface based on bound variables.

## Constructors
```def pTaskResolver(T()
```
-  Special resolver that is used for MapTasks. This exists because it is possible that MapTasks are created using nested &quot;partial&quot; subtasks. When a maptask is created its interface is interpolated from the interface of the subtask - the interpolation, simply converts every input into a list/collection input.

For example:
  interface - &gt; (i: int, j: str) - &gt; str  = &gt; map_task interface - &gt; (i: List[int], j: List[str]) - &gt; List[str]

But in cases in which `j` is bound to a fixed value by using `functools.partial` we need a way to ensure that the interface is not simply interpolated, but only the unbound inputs are interpolated.

```python
def foo((i: int, j: str) - &gt; str:
    ...

mt = map_task(functools.partial(foo, j=10))

print(mt.interface)
```

output:

        (i: List[int], j: str) - &gt; List[str]

But, at runtime this information is lost. To reconstruct this, we use MapTaskResolver that records the &quot;bound vars&quot; and then at runtime reconstructs the interface with this knowledge



## Methods
```def name()
```
-  Returns the name of the resolver.

- **Return Value**:
**string**
  - The name of the resolver.
def load_task(loader_args: List[str], max_concurrency: int = 0) - > [MapPythonTask](flytekit_core_legacy_map_task_mappythontask)
-  Loads a MapTask given loader arguments. The loader arguments should specify the variables that are bound in the MapTask, the resolver to use for the underlying task, and any arguments for that resolver.
- **Parameters**

  - **loader_args**: List[str]
    - A list of strings representing the loader arguments. Expected format: &#x27;vars &quot;var1,var2,..&quot; resolver &quot;resolver&quot; [resolver_args]&#x27;
  - **max_concurrency**: int
    - The maximum concurrency for the MapTask.

- **Return Value**:
**[MapPythonTask](flytekit_core_legacy_map_task_mappythontask)**
  - A MapPythonTask object.
def loader_args(settings: [SerializationSettings](flytekit_configuration_serializationsettings), t: [MapPythonTask](flytekit_core_legacy_map_task_mappythontask)) - > List[str]
-  Generates the loader arguments for a MapPythonTask, including bound inputs, the resolver location, and its loader arguments.
- **Parameters**

  - **settings**: [SerializationSettings](flytekit_configuration_serializationsettings)
    - Serialization settings.
  - **t**: [MapPythonTask](flytekit_core_legacy_map_task_mappythontask)
    - The MapPythonTask to generate loader arguments for.

- **Return Value**:
**List[str]**
  - A list of strings representing the loader arguments.
```def get_all_tasks()
```
-  This method is not implemented for MapTaskResolver as it cannot return every instance of the map task.

- **Return Value**:
**List[[Task](flytekit_models_task_task)]**
  - A list of Task objects.
