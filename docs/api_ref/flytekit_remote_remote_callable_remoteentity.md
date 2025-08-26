# RemoteEntity

This class serves as an abstract base class for remote entities within a distributed execution environment. It defines the fundamental interface for interacting with remote tasks and workflows. Key features include compilation for workflow integration and handling different execution modes, such as local and remote execution.

## Attributes

- **_python_interface**: Optional["Interface"] = None
  - In cases where we make a FlyteTask/Workflow/LaunchPlan from a locally created Python object (i.e. an @task
        or an @workflow decorated function), we actually have the Python interface, so

## Constructors
def RemoteEntity(args: Any, kwargs: Any)
-  In cases where we make a FlyteTask/Workflow/LaunchPlan from a locally created Python object (i.e. an @task
or an @workflow decorated function), we actually have the Python interface, so
self._python_interface: Optional[&quot;Interface&quot;] = None

super().__init__(*args, **kwargs)
- **Parameters**

  - **args**: Any
    - args
  - **kwargs**: Any
    - kwargs



