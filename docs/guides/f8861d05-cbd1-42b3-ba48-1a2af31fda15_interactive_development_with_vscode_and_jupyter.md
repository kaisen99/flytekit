
<!--
help_text: ''
key: summary_interactive_development_with_vscode_and_jupyter_c79774de-f7a9-477d-9b48-7bd71fdad82f
modules:
- flytekit.interactive.vscode_lib.decorator
- flytekit.interactive.vscode_lib.config
- flytekitplugins.flyteinteractive.jupyter_lib.decorator
questions_to_answer: []
type: summary

-->
Interactive Development with VSCode and Jupyter

Interactive development within a task provides a powerful way to debug, explore, and iterate on code directly within the execution environment. This capability allows developers to launch a VSCode server or a Jupyter Notebook server as part of their task, enabling real-time interaction with the task's state and resources.

### Interactive VSCode Development

The `vscode` decorator enables launching a VSCode server within a task's container, facilitating remote development and debugging.

#### Purpose and Capabilities

When applied to a task, the `vscode` decorator transforms the task's execution to:
*   **Provision a VSCode Server**: Automatically downloads and launches a VSCode server instance within the task's container.
*   **Enable Remote Debugging**: Prepares the environment for Python debugging, including generating a `launch.json` file and a dedicated Python script for interactive debugging.
*   **Support Task Resumption**: Provides mechanisms to resume the original task execution after an interactive session, allowing for seamless transition from debugging to full task completion.
*   **Monitor and Terminate**: Monitors the VSCode server for activity and automatically terminates the session if it remains idle for a specified duration, or upon user-triggered task resumption.

#### Using the `vscode` Decorator

Apply the `vscode` decorator to any task function to enable an interactive VSCode session.

```python
from flytekit.interactive.vscode_lib.decorator import vscode
from flytekit.interactive.vscode_lib.config import VscodeConfig
from flytekit import task

@vscode(max_idle_seconds=3600, port=8080)
@task
def my_debuggable_task(a: int, b: int) -> int:
    """
    This task will launch a VSCode server for interactive debugging.
    """
    result = a + b
    print(f"The sum is: {result}")
    # You can set a breakpoint here and debug remotely
    return result

# Example with pre/post execution hooks and run_task_first
def pre_vscode_setup():
    print("Setting up environment before VSCode launch...")

def post_vscode_cleanup():
    print("Cleaning up after VSCode session...")

@vscode(
    max_idle_seconds=7200,
    port=8080,
    run_task_first=True, # Run the task first, launch VSCode only if it fails
    pre_execute=pre_vscode_setup,
    post_execute=post_vscode_cleanup
)
@task
def resilient_task(data: list) -> int:
    """
    This task attempts to run normally. If it fails, a VSCode server is launched.
    """
    try:
        # Simulate a potential error
        if not data:
            raise ValueError("Input data cannot be empty")
        total = sum(data)
        return total
    except Exception as e:
        print(f"Task failed: {e}. VSCode will be launched for debugging.")
        raise # Re-raise to trigger VSCode launch if run_task_first is True
```

**Decorator Arguments:**

*   `task_function` (Callable, optional): The task function being decorated. This is typically inferred.
*   `max_idle_seconds` (int, optional): The maximum duration (in seconds) the VSCode server will remain active without user interaction. Defaults to a system-defined value. Set to `0` for no idle timeout.
*   `port` (int, optional): The port on which the VSCode server will listen. Defaults to `8080`.
*   `enable` (bool, optional): A flag to enable or disable the VSCode decorator. When `False`, the task executes normally without launching VSCode. Defaults to `True`.
*   `run_task_first` (bool, optional): If `True`, the original task function executes first. The VSCode server is launched only if the task fails. This is useful for "fail-over to interactive" debugging. Defaults to `False`.
*   `pre_execute` (Callable, optional): A function to execute immediately before the VSCode server setup begins. Useful for custom environment preparation.
*   `post_execute` (Callable, optional): A function to execute just before the VSCode server terminates. Useful for cleanup or final logging.
*   `config` (VscodeConfig, optional): An instance of `VscodeConfig` to customize VSCode server and extension paths.

#### Configuring VSCode with `VscodeConfig`

The `VscodeConfig` class allows for advanced customization of the VSCode server and its extensions.

```python
from flytekit.interactive.vscode_lib.config import VscodeConfig

# Create a custom VSCode configuration
custom_vscode_config = VscodeConfig(
    code_server_remote_paths={
        "linux-x64": "https://example.com/my-custom-code-server.tar.gz"
    },
    extension_remote_paths=[
        "https://open-vsx.org/api/ms-python/python/2023.18.0/file/ms-python.python-2023.18.0.vsix",
        "https://open-vsx.org/api/ms-toolsai/jupyter/2023.10.1003070119/file/ms-toolsai.jupyter-2023.10.1003070119.vsix"
    ]
)

# Add more extensions dynamically
custom_vscode_config.add_extensions("https://open-vsx.org/api/some-publisher/some-extension/1.0.0/file/some-publisher.some-extension-1.0.0.vsix")

@vscode(config=custom_vscode_config)
@task
def task_with_custom_vscode_setup():
    print("VSCode launched with custom server and extensions.")
```

**`VscodeConfig` Attributes:**

*   `code_server_remote_paths` (Dict[str, str], optional): A dictionary mapping platform (e.g., "linux-x64") to the URL of the code-server tarball.
*   `code_server_dir_names` (Dict[str, str], optional): A dictionary mapping platform to the expected directory name after extracting the code-server tarball.
*   `extension_remote_paths` (List[str], optional): A list of URLs to VSCode extension `.vsix` files. These extensions will be downloaded and installed in the VSCode server.

Use the `add_extensions` method to append additional extension URLs to the configuration.

#### Execution Flow and Debugging

When a task decorated with `vscode` is executed remotely:
1.  The `pre_execute` function (if provided) runs.
2.  The VSCode server and specified extensions are downloaded into the container.
3.  A VSCode server process is launched, listening on the configured `port`.
4.  An interactive Python debugging script and a `launch.json` configuration are generated within the task's working directory. These files enable connecting a remote VSCode client to debug the original task function.
5.  The task enters a monitoring state, waiting for user interaction or idle timeout.

To debug, connect your local VSCode client to the remote VSCode server running in the task's container using port forwarding. The generated `launch.json` will allow you to attach to the running Python process or launch a new one for debugging.

#### Task Resumption

The interactive VSCode session is designed to be temporary. Users can trigger task resumption from within the VSCode environment. This action signals the task to terminate the VSCode server and proceed with the original task function's execution, or to exit if the task function has already completed or failed.

### Interactive Jupyter Development

The `jupyter` decorator enables launching a Jupyter Notebook server within a task's container, providing an interactive notebook environment.

#### Purpose and Capabilities

When applied to a task, the `jupyter` decorator transforms the task's execution to:
*   **Launch Jupyter Server**: Starts a Jupyter Notebook server within the task's container.
*   **Provide Notebook Access**: Makes the Jupyter environment accessible via a specified port, allowing users to connect and interact with notebooks.
*   **Generate Example Notebook**: Creates an example Jupyter Notebook file (`example.ipynb`) in the specified notebook directory, demonstrating how to interact with the task's context and data.
*   **Monitor and Terminate**: Monitors the Jupyter server for activity and automatically terminates the session if it remains idle for a specified duration, or if the user manually shuts down the server.

#### Using the `jupyter` Decorator

Apply the `jupyter` decorator to any task function to enable an interactive Jupyter session.

```python
from flytekitplugins.flyteinteractive.jupyter_lib.decorator import jupyter
from flytekit import task

@jupyter(max_idle_seconds=3600, port=8888, notebook_dir="/root/notebooks")
@task
def my_jupyter_task(data: list) -> float:
    """
    This task will launch a Jupyter Notebook server.
    """
    print(f"Received data: {data}")
    # You can interact with this data in the Jupyter Notebook
    return sum(data) / len(data)

# Example with run_task_first
@jupyter(run_task_first=True)
@task
def data_analysis_task(dataset_path: str) -> dict:
    """
    Attempts to analyze data. If it fails, a Jupyter server is launched for interactive analysis.
    """
    try:
        # Simulate data loading and analysis
        if not dataset_path:
            raise FileNotFoundError("Dataset path is empty.")
        print(f"Analyzing data from: {dataset_path}")
        # Perform analysis
        return {"status": "success", "summary": "data analyzed"}
    except Exception as e:
        print(f"Analysis failed: {e}. Launching Jupyter for manual inspection.")
        raise # Re-raise to trigger Jupyter launch if run_task_first is True
```

**Decorator Arguments:**

*   `task_function` (Callable, optional): The task function being decorated. This is typically inferred.
*   `max_idle_seconds` (int, optional): The maximum duration (in seconds) the Jupyter server will remain active without user interaction. Defaults to a system-defined value. Set to `0` for no idle timeout.
*   `port` (int, optional): The port on which the Jupyter Notebook server will listen. Defaults to `8888`.
*   `enable` (bool, optional): A flag to enable or disable the Jupyter decorator. When `False`, the task executes normally without launching Jupyter. Defaults to `True`.
*   `run_task_first` (bool, optional): If `True`, the original task function executes first. The Jupyter server is launched only if the task fails. This is useful for "fail-over to interactive" debugging. Defaults to `False`.
*   `notebook_dir` (str, optional): The directory where Jupyter Notebook will look for notebooks and where the example notebook will be written. Defaults to `/root`.
*   `pre_execute` (Callable, optional): A function to execute immediately before the Jupyter server setup begins.
*   `post_execute` (Callable, optional): A function to execute just before the Jupyter server terminates.

#### Execution Flow and Notebook Access

When a task decorated with `jupyter` is executed remotely:
1.  The `pre_execute` function (if provided) runs.
2.  A Jupyter Notebook server process is launched, listening on the configured `port`. The server is configured to listen on all interfaces (`--ip='*'`) and disables token-based authentication for easier remote access (`--NotebookApp.token=''`).
3.  An example Jupyter Notebook (`example.ipynb`) is written to the specified `notebook_dir`. This notebook provides a starting point for interacting with the task's environment.
4.  The task enters a monitoring state, waiting for user interaction or idle timeout.

To access the Jupyter Notebook, connect your local browser to the remote Jupyter server running in the task's container using port forwarding.

### Common Considerations and Best Practices

#### Remote Execution Requirement

Both the `vscode` and `jupyter` decorators are designed for **remote execution** of tasks. When tasks decorated with these are run locally (e.g., using `pyflyte run` or direct Python execution), the interactive server components are automatically disabled, and the task function executes normally. This ensures consistent behavior across local development and remote deployment.

#### Managing Idle Sessions

The `max_idle_seconds` argument is crucial for efficient resource management. Setting an appropriate timeout ensures that interactive sessions do not consume cluster resources indefinitely if left unattended. For long-running debugging or exploration, set a higher value or `0` to disable the timeout, but be mindful of resource consumption.

#### Customizing Execution with Hooks

The `pre_execute` and `post_execute` arguments provide powerful hooks for custom setup and teardown logic. Use `pre_execute` to prepare the environment (e.g., download additional data, set environment variables) before the interactive server starts. Use `post_execute` for cleanup, logging, or saving results after the interactive session concludes.

#### Resource Management

Launching interactive servers consumes additional CPU, memory, and potentially network resources within the task's container. Ensure that the task's resource requests are sufficient to accommodate both the task's workload and the interactive server.
<!--
key: summary_interactive_development_with_vscode_and_jupyter_c79774de-f7a9-477d-9b48-7bd71fdad82f
type: summary_end

-->
<!--
code_unit: flytekit.interactive.vscode_lib.decorator.vscode
code_unit_type: class
help_text: ''
key: example_2d0e513f-e952-414e-b6bf-e107fbc47c25
type: example

-->
<!--
code_unit: flytekitplugins.flyteinteractive.jupyter_lib.decorator.jupyter
code_unit_type: class
help_text: ''
key: example_e78585b2-d62d-46a1-aa9a-54c4071b1a3c
type: example

-->