# ClassStorageTaskResolver

This class stores and resolves tasks using a class variable. It provides methods for adding, retrieving, and loading tasks based on provided arguments. The class utilizes a mapping to store tasks and relies on the `TrackedInstance` and `TaskResolverMixin` interfaces.

## Attributes

- **mapping**: List[[PythonAutoContainerTask](flytekit_core_python_auto_container_pythonautocontainertask)] = []
  - Stores tasks inside a class variable.

## Constructors
def ClassStorageTaskResolver(args: Any, kwargs: Any)
-  Stores tasks inside a class variable. The class must be inherited from at the point of usage because the task loading process basically relies on the same sequence of things happening.
- **Parameters**

  - **args**: Any
    - Positional arguments to pass to the parent class constructor.
  - **kwargs**: Any
    - Keyword arguments to pass to the parent class constructor.



## Methods
```@classmethod
def name()
```
-  Returns the name of the ClassStorageTaskResolver.

- **Return Value**:
**string**
  - The name of the resolver.
```@classmethod
def get_all_tasks()
```
-  Retrieves all tasks stored within the resolver.

- **Return Value**:
**List[[PythonAutoContainerTask](flytekit_core_python_auto_container_pythonautocontainertask)]**
  - A list of all tasks.
@classmethod
def add(t: [PythonAutoContainerTask](flytekit_core_python_auto_container_pythonautocontainertask))
-  Adds a task to the storage.
- **Parameters**

  - **t**: [PythonAutoContainerTask](flytekit_core_python_auto_container_pythonautocontainertask)
    - The task to add.

@classmethod
def load_task(loader_args: List[str]) - > [PythonAutoContainerTask](flytekit_core_python_auto_container_pythonautocontainertask)
-  Loads a task based on the provided loader arguments.
- **Parameters**

  - **loader_args**: List[str]
    - A list containing a single string which is the index of the task to load.

- **Return Value**:
**[PythonAutoContainerTask](flytekit_core_python_auto_container_pythonautocontainertask)**
  - The loaded task.
@classmethod
def loader_args(settings: [SerializationSettings](flytekit_configuration_serializationsettings), t: [PythonAutoContainerTask](flytekit_core_python_auto_container_pythonautocontainertask)) - > List[str]
-  Generates loader arguments for a given task.
- **Parameters**

  - **settings**: [SerializationSettings](flytekit_configuration_serializationsettings)
    - Serialization settings.
  - **t**: [PythonAutoContainerTask](flytekit_core_python_auto_container_pythonautocontainertask)
    - The task for which to generate loader arguments.

- **Return Value**:
**List[str]**
  - A list containing the index of the task as a string.
