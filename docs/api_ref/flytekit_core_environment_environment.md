# Environment

This class manages and applies configurations, or overrides, to tasks. It allows for updating, extending, and applying these overrides to functions, effectively customizing their behavior. The class supports a callable interface, enabling the application of overrides through function decorators or direct calls, and provides methods for displaying the current configuration.

## Attributes

- **overrides**: dict[str, Any] = {}
  - A dictionary to store overrides.

## Constructors
def Environment(overrides: dict[str, Any])
-  Initializes the Environment with optional overrides.
- **Parameters**

  - **overrides**: dict[str, Any]
    - A dictionary of key-value pairs to override default settings. The key &#x27;_task_function&#x27; is not allowed.



## Methods
@classmethod
def update(overrides: Any) - > None
-  Updates the environment with new overrides. It merges the existing overrides with the new ones, prioritizing the new ones.
- **Parameters**

  - **overrides**: Any
    - A dictionary of key-value pairs to update the environment with.

- **Return Value**:
**None**
  - This method does not return anything.
@classmethod
def extend(overrides: Any) - > [Environment](flytekit_core_environment_environment)
-  Extends the environment with new overrides, returning a new Environment object. It merges the existing overrides with the new ones, prioritizing the new ones.
- **Parameters**

  - **overrides**: Any
    - A dictionary of key-value pairs to extend the environment with.

- **Return Value**:
**[Environment](flytekit_core_environment_environment)**
  - A new Environment object with the merged overrides.
```@classmethod
def show()
```
-  Prints the current overrides of the environment to the console in a formatted panel.

- **Return Value**:
**None**
  - This method does not return anything.
@classmethod
def dynamic(_task_function: Callable = None, overrides: Any) - > Callable
-  Similar to __call__, this method allows the Environment object to be called as a function, specifically for dynamic task functions. It can be used to decorate a dynamic task function with the environment&#x27;s overrides or to create a new dynamic task function with extended overrides.
- **Parameters**

  - **_task_function**: Callable
    - The dynamic task function to be decorated.
  - **overrides**: Any
    - Additional overrides to be applied to the dynamic task function.

- **Return Value**:
**Callable**
  - A callable that wraps the dynamic task function with the environment&#x27;s overrides.
