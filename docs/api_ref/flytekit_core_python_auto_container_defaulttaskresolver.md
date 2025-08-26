# DefaultTaskResolver

This class serves as the default implementation for resolving tasks. It provides the functionality to load and retrieve tasks, leveraging a mixin for core behavior. The class utilizes a specific method to extract task-related information and relies on external modules for task definitions.



## Methods
```@classmethod
def name()
```
-  Returns the name of the task resolver.

- **Return Value**:
**string**
  - The name of the task resolver.
@classmethod
def load_task(loader_args: List[str]) - > [PythonAutoContainerTask](flytekit_core_python_auto_container_pythonautocontainertask)
-  Loads a task based on the provided loader arguments.
- **Parameters**

  - **loader_args**: List[str]
    - A list of strings representing the arguments for loading the task.

- **Return Value**:
**[PythonAutoContainerTask](flytekit_core_python_auto_container_pythonautocontainertask)**
  - The loaded task.
@classmethod
def loader_args(settings: [SerializationSettings](flytekit_configuration_serializationsettings), task: [PythonAutoContainerTask](flytekit_core_python_auto_container_pythonautocontainertask)) - > List[str]
-  Generates the loader arguments for a given task.
- **Parameters**

  - **settings**: [SerializationSettings](flytekit_configuration_serializationsettings)
    - The serialization settings.
  - **task**: [PythonAutoContainerTask](flytekit_core_python_auto_container_pythonautocontainertask)
    - The task for which to generate loader arguments.

- **Return Value**:
**List[str]**
  - A list of strings representing the loader arguments.
```@classmethod
def get_all_tasks()
```
-  Retrieves all available tasks. This method is intended to be implemented by subclasses.

- **Return Value**:
**List[[PythonAutoContainerTask](flytekit_core_python_auto_container_pythonautocontainertask)]**
  - A list of all available tasks.
