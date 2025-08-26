# ExecutableTemplateShimTask

This class provides an alternative execution pattern for Flyte tasks, utilizing a TaskTemplate and an executor. It facilitates task execution by delegating the computation to a specified executor. The class implements `dispatch_execute` and `execute` methods to mimic a PythonTask for compatibility.

## Attributes

- **_executor_type**: Type[[ShimTaskExecutor](flytekit_core_shim_task_shimtaskexecutor)]
  - The type of the executor to use for this task.

- **_executor**: [ShimTaskExecutor](flytekit_core_shim_task_shimtaskexecutor)
  - The executor instance for this task.

- **_task_template**: _task_model.TaskTemplate
  - The task template associated with this task.

## Constructors
def ExecutableTemplateShimTask(tt: _task_model.TaskTemplate, executor_type: Type[[ShimTaskExecutor](flytekit_core_shim_task_shimtaskexecutor)], args: tuple, kwargs: dict)
-  Initializes the ExecutableTemplateShimTask with a task template and executor type.
- **Parameters**

  - **tt**: _task_model.TaskTemplate
    - The task template defining the task&#x27;s structure and metadata.
  - **executor_type**: Type[[ShimTaskExecutor](flytekit_core_shim_task_shimtaskexecutor)]
    - The type of the executor to be used for running the task.
  - **args**: tuple
    - Positional arguments to pass to the parent class constructor.
  - **kwargs**: dict
    - Keyword arguments to pass to the parent class constructor.



## Methods
```@classmethod
def name()
```
-  Return the name of the underlying task.

- **Return Value**:
**string**
  - the name of the task
```@classmethod
def task_template()
```
-  Returns the task template.

- **Return Value**:
**_task_model.TaskTemplate**
  - The task template
```@classmethod
def executor()
```
-  Returns the executor.

- **Return Value**:
**[ShimTaskExecutor](flytekit_core_shim_task_shimtaskexecutor)**
  - The executor
```@classmethod
def executor_type()
```
-  Returns the executor type.

- **Return Value**:
**Type[[ShimTaskExecutor](flytekit_core_shim_task_shimtaskexecutor)]**
  - The executor type
@classmethod
def execute(kwargs: dict) - > Any
-  Rather than running here, send everything to the executor.
- **Parameters**

  - **kwargs**: dict
    - Keyword arguments for the execution

- **Return Value**:
**Any**
  - The result of the execution
@classmethod
def pre_execute(user_params: Optional[[ExecutionParameters](flytekit_core_context_manager_executionparameters)]) - > Optional[[ExecutionParameters](flytekit_core_context_manager_executionparameters)]
-  This function is a stub, just here to keep dispatch_execute compatibility between this class and PythonTask.
- **Parameters**

  - **user_params**: Optional[[ExecutionParameters](flytekit_core_context_manager_executionparameters)]
    - User parameters

- **Return Value**:
**Optional[[ExecutionParameters](flytekit_core_context_manager_executionparameters)]**
  - The execution parameters
@classmethod
def post_execute(_: Optional[[ExecutionParameters](flytekit_core_context_manager_executionparameters)], rval: Any) - > Any
-  This function is a stub, just here to keep dispatch_execute compatibility between this class and PythonTask.
- **Parameters**

  - **_**: Optional[[ExecutionParameters](flytekit_core_context_manager_executionparameters)]
    - Execution parameters (unused)
  - **rval**: Any
    - The return value of the execution

- **Return Value**:
**Any**
  - The result of the execution
@classmethod
def dispatch_execute(ctx: [FlyteContext](flytekit_core_context_manager_flytecontext), input_literal_map: _literal_models.LiteralMap) - > [Union](flytekit_models_literals_union)[_literal_models.LiteralMap, _dynamic_job.DynamicJobSpec]
-  This function is largely similar to the base PythonTask, with the exception that we have to infer the Python interface before executing. Also, we refer to ``self.task_template`` rather than just ``self`` similar to task classes that derive from the base ``PythonTask``.
- **Parameters**

  - **ctx**: [FlyteContext](flytekit_core_context_manager_flytecontext)
    - The Flyte context
  - **input_literal_map**: _literal_models.LiteralMap
    - The literal map of inputs

- **Return Value**:
**[Union](flytekit_models_literals_union)[_literal_models.LiteralMap, _dynamic_job.DynamicJobSpec]**
  - The literal map or dynamic job spec
