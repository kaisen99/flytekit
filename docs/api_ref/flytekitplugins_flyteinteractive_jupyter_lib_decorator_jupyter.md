# jupyter

This class serves as a decorator to enable and manage a Jupyter Notebook server within a containerized environment. It launches a Jupyter Notebook server, monitors its activity, and provides options for automatic shutdown based on idle time. The class integrates with a task execution framework, allowing for the execution of user-defined functions alongside the Jupyter Notebook server.

## Attributes

- **task_function**: Optional[Callable] = None
  - The user function to be decorated.

- **max_idle_seconds**: Optional[int] = MAX_IDLE_SECONDS
  - The duration in seconds to live after no activity detected.

- **port**: int = 8080
  - The port to be used by the Jupyter Notebook server.

- **enable**: bool = True
  - Whether to enable the Jupyter decorator.

- **run_task_first**: bool = False
  - Executes the user&#x27;s task first when True. Launches the Jupyter Notebook server only if the user&#x27;s task fails.

- **notebook_dir**: Optional[str] = &quot;/root&quot;
  - Sets the directory where Jupyter Notebook will look for notebooks.

- **pre_execute**: Optional[Callable] = None
  - The function to be executed before the jupyter setup function.

- **post_execute**: Optional[Callable] = None
  - The function to be executed before the jupyter is self-terminated.

## Constructors
def jupyter(task_function: Optional[Callable] = None, max_idle_seconds: Optional[int] = MAX_IDLE_SECONDS, port: int = 8080, enable: bool = True, run_task_first: bool = False, notebook_dir: Optional[str] = &quot;/root&quot;, pre_execute: Optional[Callable] = None, post_execute: Optional[Callable] = None)
-  jupyter decorator modifies a container to run a Jupyter Notebook server:
        1. Launches and monitors the Jupyter Notebook server.
        2. Write Example Jupyter Notebook.
        3. Terminates if the server is idle for a set duration or user shuts down manually.
- **Parameters**

  - **task_function**: Optional[Callable]
    - The user function to be decorated.
  - **max_idle_seconds**: Optional[int]
    - The duration in seconds to live after no activity detected.
  - **port**: int
    - The port to be used by the Jupyter Notebook server.
  - **enable**: bool
    - Whether to enable the Jupyter decorator.
  - **run_task_first**: bool
    - Executes the user&#x27;s task first when True. Launches the Jupyter Notebook server only if the user&#x27;s task fails.
  - **notebook_dir**: Optional[str]
    - The directory where Jupyter Notebook will look for notebooks.
  - **pre_execute**: Optional[Callable]
    - The function to be executed before the jupyter setup function.
  - **post_execute**: Optional[Callable]
    - The function to be executed before the jupyter is self-terminated.



## Methods
@classmethod
def execute(args: tuple, kwargs: dict) - > any
-  Executes the decorated function, optionally launching a Jupyter Notebook server. It handles enabling/disabling the decorator, running the task first, executing pre/post functions, and managing the Jupyter server process with specified configurations like port, idle timeout, and notebook directory.
- **Parameters**

  - **args**: tuple
    - Positional arguments passed to the decorated function.
  - **kwargs**: dict
    - Keyword arguments passed to the decorated function.

- **Return Value**:
**any**
  - The return value of the decorated task function.
```@classmethod
def get_extra_config()
```
-  Returns extra configuration for the Jupyter decorator, including the link type and the port number.

- **Return Value**:
**dict**
  - A dictionary containing &#x27;type&#x27; and &#x27;port&#x27; keys.
