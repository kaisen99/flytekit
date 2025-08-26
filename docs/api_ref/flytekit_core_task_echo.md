# Echo

This class represents an echo task that mirrors its inputs as outputs. It is designed to pass inputs directly to outputs without any processing. The class utilizes the FlytePropeller echo plugin, bypassing pod creation for efficiency.

## Constructors
def Echo(name: string, inputs: Optional[Dict[str, Type]] = None, kwargs: **kwargs)
-  A task that simply echoes the inputs back to the user.
The task&#x27;s inputs and outputs interface are the same.

FlytePropeller uses echo plugin to handle this task, and it won&#x27;t create a pod for this task.
It will simply pass the inputs to the outputs.
https://github.com/flyteorg/flyte/blob/master/flyteplugins/go/tasks/plugins/testing/echo.go

Note: Make sure to enable the echo plugin in the propeller config to use this task.
```
task-plugins:
  enabled-plugins:
    - echo
```
- **Parameters**

  - **name**: string
    - The name of the task.
  - **inputs**: Optional[Dict[str, Type]]
    - Name and type of inputs specified as a dictionary.
e.g. {&quot;a&quot;: int, &quot;b&quot;: str}.
  - **kwargs**: **kwargs
    - All other args required by the parent type - PythonTask.

def Echo(name: str, inputs: Optional[Dict[str, Type]], kwargs: **kwargs)
-  A task that simply echoes the inputs back to the user.
The task&#x27;s inputs and outputs interface are the same.

FlytePropeller uses echo plugin to handle this task, and it won&#x27;t create a pod for this task.
It will simply pass the inputs to the outputs.
https://github.com/flyteorg/flyte/blob/master/flyteplugins/go/tasks/plugins/testing/echo.go

Note: Make sure to enable the echo plugin in the propeller config to use this task.
```
task-plugins:
  enabled-plugins:
    - echo
```

:param name: The name of the task.
:param inputs: Name and type of inputs specified as a dictionary.
    e.g. {&quot;a&quot;: int, &quot;b&quot;: str}.
:param kwargs: All other args required by the parent type - PythonTask.
- **Parameters**

  - **name**: str
    - The name of the task.
  - **inputs**: Optional[Dict[str, Type]]
    - Name and type of inputs specified as a dictionary.
e.g. {&quot;a&quot;: int, &quot;b&quot;: str}.
  - **kwargs**: **kwargs
    - All other args required by the parent type - PythonTask.



## Methods
@classmethod
def execute(kwargs: **kwargs) - > Any
-  This method echoes the inputs back to the user. If there is only one input, it returns that input directly. Otherwise, it returns a tuple of all inputs.
- **Parameters**

  - **kwargs**: **kwargs
    - Arbitrary keyword arguments representing the inputs to the task.

- **Return Value**:
**Any**
  - The echoed input(s).
