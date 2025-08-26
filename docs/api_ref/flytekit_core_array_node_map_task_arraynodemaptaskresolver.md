# ArrayNodeMapTaskResolver

This class specializes in resolving ArrayNodeMapTasks, particularly those created with nested partial subtasks. It reconstructs the task interface at runtime, accounting for bound variables to ensure correct input interpolation. It leverages a resolver to load the actual task object and stores bound inputs for interface reconstruction.



## Methods
```@classmethod
def name()
```
-  Returns the fully qualified name of this resolver.

- **Return Value**:
**string**
  - The fully qualified name of this resolver.
@classmethod
def load_task(loader_args: List[str], max_concurrency: int = 0) - > [ArrayNodeMapTask](flytekit_core_array_node_map_task_arraynodemaptask)
-  Loads a task given the loader arguments. The loader arguments should be of the form: vars &quot;var1,var2,..&quot; resolver &quot;resolver&quot; [resolver_args]. This method reconstructs the ArrayNodeMapTask by using the provided resolver to load the actual task definition and then binding the specified variables.
- **Parameters**

  - **loader_args**: List[str]
    - A list of strings representing the arguments for loading the task. Expected format: &#x27;vars &quot;var1,var2,..&quot; resolver &quot;resolver&quot; [resolver_args]&#x27;.
  - **max_concurrency**: int
    - The maximum concurrency for the map task.

- **Return Value**:
**[ArrayNodeMapTask](flytekit_core_array_node_map_task_arraynodemaptask)**
  - An instance of ArrayNodeMapTask.
@classmethod
def loader_args(settings: [SerializationSettings](flytekit_configuration_serializationsettings), t: [ArrayNodeMapTask](flytekit_core_array_node_map_task_arraynodemaptask)) - > List[str]
-  Returns the loader arguments for the ArrayNodeMapTask. This includes the bound inputs, the resolver&#x27;s location, and any additional loader arguments required by the resolver.
- **Parameters**

  - **settings**: [SerializationSettings](flytekit_configuration_serializationsettings)
    - The serialization settings.
  - **t**: [ArrayNodeMapTask](flytekit_core_array_node_map_task_arraynodemaptask)
    - The ArrayNodeMapTask instance.

- **Return Value**:
**List[str]**
  - A list of strings representing the loader arguments.
```@classmethod
def get_all_tasks()
```
-  This method is not implemented for ArrayNodeMapTaskResolver as it cannot return every instance of the map task.

- **Return Value**:
**List[[Task](flytekit_models_task_task)]**
  - A list of Task objects.
