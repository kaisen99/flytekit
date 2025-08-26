# TaskResolverMixin

This class provides a mixin for resolving Flytekit tasks within the Flyte platform. It facilitates the uploading and execution of tasks by defining interfaces for task loading and argument handling. Key features include methods for identifying, loading, and retrieving tasks, enabling custom task resolution strategies.

## Attributes

- **location**: str
  - The location of the task&#x27;s task resolver.

- **name**: str
  - The name of the task.



## Methods
```@classmethod
def location()
```
-  Flytekit tasks interact with the Flyte platform very, very broadly in two steps. They need to be uploaded to Admin, and then they are run by the user upon request (either as a single task execution or as part of a workflow). In any case, at execution time, for most tasks (that is those that generate a container target) the container image containing the task needs to be spun up again at which point the container needs to know which task it&#x27;s supposed to run and how to rehydrate the task object.

- **Return Value**:
**string**
  - Flytekit tasks interact with the Flyte platform very, very broadly in two steps. They need to be uploaded to Admin, and then they are run by the user upon request (either as a single task execution or as part of a workflow). In any case, at execution time, for most tasks (that is those that generate a container target) the container image containing the task needs to be spun up again at which point the container needs to know which task it&#x27;s supposed to run and how to rehydrate the task object.
```@classmethod
def name()
```
-  Flytekit tasks interact with the Flyte platform very, very broadly in two steps. They need to be uploaded to Admin, and then they are run by the user upon request (either as a single task execution or as part of a workflow). In any case, at execution time, for most tasks (that is those that generate a container target) the container image containing the task needs to be spun up again at which point the container needs to know which task it&#x27;s supposed to run and how to rehydrate the task object.

- **Return Value**:
**string**
  - Flytekit tasks interact with the Flyte platform very, very broadly in two steps. They need to be uploaded to Admin, and then they are run by the user upon request (either as a single task execution or as part of a workflow). In any case, at execution time, for most tasks (that is those that generate a container target) the container image containing the task needs to be spun up again at which point the container needs to know which task it&#x27;s supposed to run and how to rehydrate the task object.
@classmethod
def load_task(loader_args: List[str]) - > [Task](flytekit_models_task_task)
-  Given the set of identifier keys, should return one Python Task or raise an error if not found
- **Parameters**

  - **loader_args**: List[str]
    - Given the set of identifier keys, should return one Python Task or raise an error if not found

- **Return Value**:
**[Task](flytekit_models_task_task)**
  - Given the set of identifier keys, should return one Python Task or raise an error if not found
@classmethod
def loader_args(settings: [SerializationSettings](flytekit_configuration_serializationsettings), t: [Task](flytekit_models_task_task)) - > List[str]
-  Return a list of strings that can help identify the parameter Task
- **Parameters**

  - **settings**: [SerializationSettings](flytekit_configuration_serializationsettings)
    - Return a list of strings that can help identify the parameter Task
  - **t**: [Task](flytekit_models_task_task)
    - Return a list of strings that can help identify the parameter Task

- **Return Value**:
**List[str]**
  - Return a list of strings that can help identify the parameter Task
```@classmethod
def get_all_tasks()
```
-  Future proof method. Just making it easy to access all tasks (Not required today as we auto register them)

- **Return Value**:
**List[[Task](flytekit_models_task_task)]**
  - Future proof method. Just making it easy to access all tasks (Not required today as we auto register them)
@classmethod
def task_name(t: [Task](flytekit_models_task_task)) - > Optional[str]
-  Overridable function that can optionally return a custom name for a given task
- **Parameters**

  - **t**: [Task](flytekit_models_task_task)
    - Overridable function that can optionally return a custom name for a given task

- **Return Value**:
**Optional[str]**
  - Overridable function that can optionally return a custom name for a given task
