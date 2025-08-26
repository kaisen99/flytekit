# DefaultNotebookTaskResolver

This class resolves tasks defined within a notebook environment. It facilitates loading tasks from pickled data, ensuring compatibility with the Python version used during the pickling process. It leverages the `TaskResolverMixin` interface and relies on cloudpickle for object serialization and deserialization.



## Methods
```@classmethod
def name()
```
-  Returns the name of the resolver.

- **Return Value**:
**string**
  - The name of the resolver.
@classmethod
def load_task(loader_args: List[str]) - > [PythonAutoContainerTask](flytekit_core_python_auto_container_pythonautocontainertask)
-  Loads a task from a pickle file based on the provided loader arguments.
- **Parameters**

  - **loader_args**: List[str]
    - A list of strings containing arguments for loading the task. The first argument is expected to be the entity name.

- **Return Value**:
**[PythonAutoContainerTask](flytekit_core_python_auto_container_pythonautocontainertask)**
  - The loaded task.
@classmethod
def loader_args(settings: [SerializationSettings](flytekit_configuration_serializationsettings), task: [PythonAutoContainerTask](flytekit_core_python_auto_container_pythonautocontainertask)) - > List[str]
-  Generates the loader arguments required to load a task.
- **Parameters**

  - **settings**: [SerializationSettings](flytekit_configuration_serializationsettings)
    - Serialization settings.
  - **task**: [PythonAutoContainerTask](flytekit_core_python_auto_container_pythonautocontainertask)
    - The task for which to generate loader arguments.

- **Return Value**:
**List[str]**
  - A list of strings representing the loader arguments.
```@classmethod
def get_all_tasks()
```
-  Retrieves all available tasks. This method is not implemented for this resolver.

- **Return Value**:
**List[[PythonAutoContainerTask](flytekit_core_python_auto_container_pythonautocontainertask)]**
  - A list of all available tasks.
