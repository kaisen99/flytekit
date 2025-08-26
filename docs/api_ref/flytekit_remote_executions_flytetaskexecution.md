# FlyteTaskExecution

This class represents a task execution within a Flyte remote backend environment. It extends RemoteExecutionBase and admin_task_execution_models.TaskExecution to manage the lifecycle of a task. Key features include checking execution status, accessing task-related information, and retrieving error details upon completion. It relies on the admin_task_execution_models.TaskExecution model for its core functionality.

## Attributes

- **task**: Optional[[FlyteTask](flytekit_remote_entities_flytetask)] = None
  - The FlyteTask object associated with this execution.

- **is_done**: bool = False
  - Whether or not the execution is complete.

- **error**: Optional[core_execution_models.ExecutionError] = None
  - If execution is in progress, raise an exception. Otherwise, return None if no error was present upon reaching completion.

## Constructors
```def FlyteTaskExecution()
```
-  Initializes a FlyteTaskExecution instance.



## Methods
```@classmethod
def task()
```
-  Returns the FlyteTask object associated with this execution.

- **Return Value**:
**Optional[[FlyteTask](flytekit_remote_entities_flytetask)]**
  - The FlyteTask object, or None if not set.
```@classmethod
def is_done()
```
-  Whether or not the execution is complete.

- **Return Value**:
**bool**
  - True if the execution is complete (succeeded, failed, or aborted), False otherwise.
```@classmethod
def error()
```
-  If execution is in progress, raise an exception. Otherwise, return None if no error was present upon reaching completion.

- **Return Value**:
**Optional[core_execution_models.ExecutionError]**
  - The execution error, or None if no error occurred.
@classmethod
def promote_from_model(base_model: admin_task_execution_models.TaskExecution) - > [FlyteTaskExecution](flytekit_remote_executions_flytetaskexecution)
-  Promotes a base model TaskExecution to a FlyteTaskExecution.
- **Parameters**

  - **base_model**: admin_task_execution_models.TaskExecution
    - The base TaskExecution model to promote.

- **Return Value**:
**[FlyteTaskExecution](flytekit_remote_executions_flytetaskexecution)**
  - A new FlyteTaskExecution instance created from the base model.
