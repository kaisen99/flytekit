# Task

This class represents a task within a workflow execution. It encapsulates the task&#x27;s unique identifier and the closure containing the underlying workload details. The class provides methods to convert the task to and from Flyte IDL representations, facilitating serialization and deserialization for communication and storage.

## Attributes

- **id**: flytekit.models.core.identifier.Identifier
  - The (project, domain, name, version) identifier for this task.

- **closure**: [TaskClosure](flytekit_models_task_taskclosure)
  - The closure for the underlying workload.

## Constructors
def Task(id: flytekit.models.core.identifier.Identifier, closure: [TaskClosure](flytekit_models_task_taskclosure))
-  Initializes a Task object.
- **Parameters**

  - **id**: flytekit.models.core.identifier.Identifier
    - The (project, domain, name) identifier for this task.
  - **closure**: [TaskClosure](flytekit_models_task_taskclosure)
    - The closure for the underlying workload.

def Task(id: flytekit.models.core.identifier.Identifier, closure: [TaskClosure](flytekit_models_task_taskclosure))
-  Initializes a Task object with an identifier and a closure.
- **Parameters**

  - **id**: flytekit.models.core.identifier.Identifier
    - The (project, domain, name) identifier for this task.
  - **closure**: [TaskClosure](flytekit_models_task_taskclosure)
    - The closure for the underlying workload.



## Methods
```@classmethod
def id()
```
-  The (project, domain, name, version) identifier for this task.

- **Return Value**:
**flytekit.models.core.identifier.Identifier**
  - The (project, domain, name, version) identifier for this task.
```@classmethod
def closure()
```
-  The closure for the underlying workload.

- **Return Value**:
**[TaskClosure](flytekit_models_task_taskclosure)**
  - The closure for the underlying workload.
```@classmethod
def to_flyte_idl()
```
-  Converts the Task object to its Flyte IDL representation.

- **Return Value**:
**flyteidl.admin.task_pb2.Task**
  - The Flyte IDL Task protocol buffer object.
@classmethod
def from_flyte_idl(pb2_object: flyteidl.admin.task_pb2.Task) - > TaskDefinition
-  Creates a Task object from a Flyte IDL Task protocol buffer object.
- **Parameters**

  - **pb2_object**: flyteidl.admin.task_pb2.Task
    - The Flyte IDL Task protocol buffer object.

- **Return Value**:
**TaskDefinition**
  - A TaskDefinition object created from the Flyte IDL.
