# TaskTemplateResolver

This class resolves tasks at execution time using only the TaskTemplate, suitable for tasks with all necessary information within the template. It loads tasks as ExecutableTemplateShimTask instances, irrespective of their original type, ensuring only template-serialized data is accessible. It utilizes a specific loader_args format and always returns an empty list when get_all_tasks is called.

## Constructors
```def TaskTemplateResolver()
```
-  Initializes the TaskTemplateResolver with default values.



## Methods
```@classmethod
def name()
```
-  Returns the name of the resolver.

- **Return Value**:
**string**
  - The name of the resolver.
@classmethod
def load_task(loader_args: List[str]) - > [ExecutableTemplateShimTask](flytekit_core_shim_task_executabletemplateshimtask)
-  Loads a task from the provided loader arguments. It always returns an ExecutableTemplateShimTask.
- **Parameters**

  - **loader_args**: List[str]
    - A list of strings containing the path to the task template and the executor class.

- **Return Value**:
**[ExecutableTemplateShimTask](flytekit_core_shim_task_executabletemplateshimtask)**
  - An ExecutableTemplateShimTask instance.
@classmethod
def loader_args(settings: [SerializationSettings](flytekit_configuration_serializationsettings), t: [PythonCustomizedContainerTask](flytekit_core_python_customized_container_task_pythoncustomizedcontainertask)) - > List[str]
-  Generates the loader arguments for a given task.
- **Parameters**

  - **settings**: [SerializationSettings](flytekit_configuration_serializationsettings)
    - Serialization settings.
  - **t**: [PythonCustomizedContainerTask](flytekit_core_python_customized_container_task_pythoncustomizedcontainertask)
    - The task to generate loader arguments for.

- **Return Value**:
**List[str]**
  - A list of strings containing the path to the task template and the executor class.
```@classmethod
def get_all_tasks()
```
-  Returns an empty list of tasks.

- **Return Value**:
**List[[Task](flytekit_models_task_task)]**
  - An empty list.
