# ConnectorBase

This abstract base class provides a foundation for creating connectors. It defines a common interface and initializes a task category for derived connector classes. Subclasses must implement specific connection logic and task handling based on their requirements.

## Attributes

- **name**: string = &quot;Base Connector&quot;
  - Base Connector

## Constructors
def ConnectorBase(task_type_name: str, task_type_version: int = 0, kwargs: dict)
-  Initializes the ConnectorBase with a task type name and version.
- **Parameters**

  - **task_type_name**: str
    - The name of the task type.
  - **task_type_version**: int
    - The version of the task type.
  - **kwargs**: dict
    - Additional keyword arguments.

def ConnectorBase(task_type_name: str, task_type_version: int = 0)
-  Initializes the ConnectorBase with a task type name and version.
- **Parameters**

  - **task_type_name**: str
    - The name of the task type.
  - **task_type_version**: int
    - The version of the task type.



## Methods
```@classmethod
def task_category()
```
-  Returns the task category that the connector supports.

- **Return Value**:
**[TaskCategory](flytekit_extend_backend_base_connector_taskcategory)**
  - The task category.
