
<!--
help_text: ''
key: summary_executing_flyte_entities_b574b0ec-e61e-4ec4-bdb6-777a6470dbd0
modules:
- flytekit.remote.remote
- flytekit.remote.remote_callable
- flytekit.remote.executions
questions_to_answer: []
type: summary

-->
## Executing Flyte Entities

Interacting with a Flyte backend to execute tasks, workflows, and launch plans is primarily managed through the `FlyteRemote` class. This class serves as the main entry point for programmatically accessing and controlling your Flyte deployments. It provides capabilities for fetching existing entities, registering new ones, and initiating executions.

### Connecting to Flyte

To begin, instantiate the `FlyteRemote` class. You can configure it to connect to a specific Flyte endpoint, automatically detect settings, or connect to a local sandbox environment.

```python
from flytekit.remote import FlyteRemote, Config

# Connect to a specific endpoint
remote = FlyteRemote.for_endpoint(
    endpoint="grpc.my-flyte-cluster.com:443",
    insecure=False,
    default_project="flytesnacks",
    default_domain="development",
)

# Auto-detect configuration from environment variables or config files
remote_auto = FlyteRemote.auto()

# Connect to a local Flyte sandbox
remote_sandbox = FlyteRemote.for_sandbox(
    default_project="flytesnacks",
    default_domain="development",
)
```

When initializing `FlyteRemote`, you can specify `default_project` and `default_domain` to avoid repeatedly providing them for subsequent operations. The `data_upload_location` parameter determines where non-literal inputs (like files or directories) are staged before execution. `interactive_mode_enabled` is useful for Jupyter notebooks, enabling automatic pickling and uploading of local code.

### Executing Entities

The `FlyteRemote.execute` method is the primary interface for initiating a Flyte execution. It is highly versatile, capable of executing:

*   **Remote Entities:** Instances of `FlyteTask`, `FlyteWorkflow`, or `FlyteLaunchPlan` that have been fetched from the Flyte backend.
*   **Reference Entities:** `ReferenceTask`, `ReferenceWorkflow`, or `ReferenceLaunchPlan` objects that point to existing entities on the backend by their identifier.
*   **Local Entities:** Python objects like `@task`-decorated functions (`PythonTask`), `@workflow`-decorated functions (`WorkflowBase`), or `LaunchPlan` objects defined in your local code.

When executing local entities, the `execute` method automatically handles the compilation and registration of these entities with the Flyte backend if they are not already present. This streamlines the development workflow, allowing you to run local code directly on the remote cluster.

#### Common Execution Parameters

The `execute` method accepts several parameters to control the execution behavior:

*   **`entity`**: The Flyte entity to execute (required).
*   **`inputs`**: A dictionary mapping input names to their Python native values (required). These values are automatically converted to Flyte Literals using the `TypeEngine`. For complex types, providing `type_hints` can assist in correct conversion.
*   **`project`**, **`domain`**: The project and domain for the execution. If not provided, the `default_project` and `default_domain` from the `FlyteRemote` instance are used.
*   **`name`**, **`version`**: For local entities, these specify the name and version for registration if the entity doesn't exist. For remote/reference entities, these are derived from the entity's identifier.
*   **`execution_name`**: A specific name for the execution. If not provided, a unique name is auto-generated.
*   **`execution_name_prefix`**: A prefix for the auto-generated execution name, followed by a random suffix.
*   **`wait`**: If `True`, the method blocks until the execution completes and returns the final `FlyteWorkflowExecution` object.
*   **`options`**: An `Options` object to configure advanced execution settings like labels, annotations, raw output data configuration, maximum parallelism, and security context.
*   **`overwrite_cache`**: If `True`, forces all cached results for the workflow and its tasks to be recomputed.
*   **`interruptible`**: Overrides the default interruptible flag for the executed entity.
*   **`envs`**: A dictionary of environment variables to set for the execution.
*   **`tags`**: A list of tags to associate with the execution.
*   **`cluster_pool`**: Specifies a cluster pool for the execution.
*   **`execution_cluster_label`**: Specifies a label for the cluster(s) where the execution should be placed.
*   **`serialization_settings`**: Provides fine-grained control over how local entities are serialized and registered.

#### Example: Executing a Local Workflow

```python
from flytekit import task, workflow
from flytekit.remote import FlyteRemote

# Assume 'remote' is an initialized FlyteRemote object
remote = FlyteRemote.for_sandbox(default_project="flytesnacks", default_domain="development")

@task
def my_task(a: int, b: str) -> str:
    return f"Task received {a} and {b}"

@workflow
def my_workflow(x: int, y: str) -> str:
    return my_task(a=x, b=y)

# Execute the local workflow. It will be automatically registered if not found.
execution = remote.execute(
    my_workflow,
    inputs={"x": 10, "y": "hello"},
    execution_name_prefix="my-first-workflow-run",
    wait=True, # Wait for the execution to complete
)

print(f"Execution status: {execution.closure.phase}")
print(f"Execution outputs: {execution.outputs.get('o0')}") # 'o0' is the default output name
```

#### Example: Executing a Remote Task

First, fetch the task, then execute it.

```python
from flytekit.remote import FlyteRemote

# Assume 'remote' is an initialized FlyteRemote object
remote = FlyteRemote.for_sandbox(default_project="flytesnacks", default_domain="development")

# Fetch an existing task by name and version
# Replace 'my_remote_task_name' and 'my_task_version' with actual values
remote_task = remote.fetch_task(name="my_remote_task_name", version="my_task_version")

# Execute the fetched task
execution = remote.execute(
    remote_task,
    inputs={"input_param_1": "value1", "input_param_2": 123},
    execution_name_prefix="remote-task-run",
    wait=True,
)

print(f"Remote task execution status: {execution.closure.phase}")
```

### Monitoring Executions

After initiating an execution, you receive a `FlyteWorkflowExecution` object. This object provides methods and properties to monitor the execution's progress and retrieve its results.

*   **`is_done`**: A boolean indicating if the execution has completed (succeeded, failed, aborted, or timed out).
*   **`is_successful`**: A boolean indicating if the execution completed successfully.
*   **`error`**: If the execution failed, this property returns an `ExecutionError` object with details. Raises an exception if the execution is still in progress.
*   **`outputs`**: A `LiteralsResolver` object providing access to the execution's outputs. Raises an exception if the execution is not done or failed.
*   **`execution_url`**: Generates a URL to view the execution in the Flyte Console.

#### Syncing and Waiting for Executions

Execution objects are snapshots of the remote state at the time they were created or last synced. To get the latest status:

*   **`FlyteWorkflowExecution.sync()`**: Updates the local `FlyteWorkflowExecution` object with the latest state from the Flyte backend.
*   **`FlyteWorkflowExecution.wait()`**: A blocking call that repeatedly syncs the execution until it completes or a timeout is reached.

```python
# Assuming 'execution' is a FlyteWorkflowExecution object
execution.sync(sync_nodes=True) # Syncs the workflow and its underlying node executions

if execution.is_done:
    if execution.is_successful:
        print(f"Execution succeeded. Outputs: {execution.outputs.to_dict()}")
    else:
        print(f"Execution failed. Error: {execution.error.message}")
else:
    print("Execution is still running.")

# Wait for completion with a timeout
try:
    execution.wait(timeout=300) # Wait for 5 minutes
    print("Execution completed.")
except Exception as e:
    print(f"Execution did not complete in time: {e}")

# Get the console URL
print(f"View execution in console: {execution.execution_url}")
```

#### Accessing Node and Task Executions

For detailed insights into a workflow's progress, you can inspect its constituent node and task executions:

*   **`FlyteWorkflowExecution.node_executions`**: A dictionary mapping node IDs to `FlyteNodeExecution` objects.
*   **`FlyteNodeExecution.task_executions`**: A list of `FlyteTaskExecution` objects for a task node.
*   **`FlyteNodeExecution.subworkflow_node_executions`**: For parent nodes (e.g., dynamic workflows or subworkflows), this provides access to their child node executions.

```python
# Assuming 'execution' is a synced FlyteWorkflowExecution object
for node_id, node_exec in execution.node_executions.items():
    print(f"Node '{node_id}' status: {node_exec.closure.phase}")

    if node_exec.task_executions:
        for task_exec in node_exec.task_executions:
            print(f"  Task '{task_exec.id.retry_attempt}' status: {task_exec.closure.phase}")
            if task_exec.is_done and task_exec.is_successful:
                print(f"    Task outputs: {task_exec.outputs.to_dict()}")
```

### Advanced Operations

The `FlyteRemote` class also offers capabilities for managing signals, launching backfills, and handling file uploads.

#### Managing Signals

Signals allow for human intervention or external event handling within a workflow.

*   **`list_signals(execution_name, project, domain)`**: Retrieves active signals for a given execution.
*   **`approve(signal_id, execution_name, project, domain)`**: Approves a boolean signal, setting its value to `True`.
*   **`reject(signal_id, execution_name, project, domain)`**: Rejects a boolean signal, setting its value to `False`.
*   **`set_input(signal_id, execution_name, value, ...)`**: Sets the value for a signal that expects a specific input type.

```python
# Assuming 'remote' and 'execution' are initialized
signals = remote.list_signals(execution_name=execution.id.name)
for signal in signals:
    print(f"Found signal: {signal.id.signal_id} with type {signal.type.simple_type}")

# Example: Approving a signal
# remote.approve("my_approval_signal", execution.id.name)

# Example: Setting an input signal
# remote.set_input("user_input_data", execution.id.name, {"key": "value"}, python_type=dict)
```

#### Launching Backfills

The `launch_backfill` method creates and executes a new workflow designed to re-run a specific launch plan for a range of historical dates. This is useful for reprocessing data or recovering from past failures.

```python
from datetime import datetime, timedelta

# Assuming 'remote' is initialized
backfill_execution = remote.launch_backfill(
    project="flytesnacks",
    domain="development",
    from_date=datetime(2023, 1, 1),
    to_date=datetime(2023, 1, 3),
    launchplan="my_daily_lp", # Name of the launch plan to backfill
    parallel=True, # Run daily instances in parallel
    execute=True, # Register and execute the backfill workflow
    execution_name="my-backfill-run-2023-01-01-to-03",
)
print(f"Backfill execution started: {backfill_execution.execution_url}")
```

#### Uploading and Downloading Files

`FlyteRemote` provides utilities for managing files, especially for fast serialization (script mode) or direct data transfer.

*   **`upload_file(to_upload, ...)`**: Uploads a local file to the Flyte data proxy service, returning its MD5 hash and the native URL.
*   **`fast_package(root, ...)`**: Packages a directory into an installable zip file and uploads it. This is used internally by `register_script` for efficient code distribution.
*   **`download(data, download_to, recursive)`**: Downloads data (e.g., outputs from an execution) to a specified local path. If `data` is a `LiteralsResolver` or `LiteralMap`, `recursive=True` downloads all contained file-like objects.

```python
import pathlib

# Assuming 'remote' is initialized
# Upload a local file
file_path = pathlib.Path("./my_local_data.txt")
file_path.write_text("This is some data.")
md5, url = remote.upload_file(file_path)
print(f"Uploaded {file_path} to {url} with MD5 {md5.hex()}")

# Download execution outputs
# Assuming 'execution' is a completed FlyteWorkflowExecution object
output_dir = "./downloaded_outputs"
remote.download(execution.outputs, download_to=output_dir, recursive=True)
print(f"Outputs downloaded to {output_dir}")
```

### Best Practices and Considerations

*   **Interactive Mode (`interactive_mode_enabled`)**: When enabled, `FlyteRemote` attempts to pickle and upload your local Python code for execution. This is convenient for rapid iteration in environments like Jupyter. However, be aware that pickling can sometimes fail for complex objects or large closures. For production deployments, consider using `register_script` or standard `pyflyte` build and register commands.
*   **Versioning**: When registering local entities, `FlyteRemote` can automatically generate a version based on the entity's hash and serialization settings if `version` is not explicitly provided. This ensures that identical code is not re-registered.
*   **Error Handling**: Always check the `is_done` and `error` properties of `FlyteWorkflowExecution` (and `FlyteNodeExecution`, `FlyteTaskExecution`) to understand the outcome of an execution.
*   **`sync_nodes`**: When calling `sync()` or `wait()` on a `FlyteWorkflowExecution`, setting `sync_nodes=True` fetches the status and details of all underlying node and task executions. While useful for debugging, setting it to `False` can improve performance for simple status checks.
*   **`RegistrationSkipped`**: This exception is raised internally when `FlyteRemote` attempts to register an entity that is explicitly marked as not registrable (e.g., certain remote entities). It indicates that the operation was intentionally skipped.
*   **`FlyteEntityAlreadyExistsException`**: This exception indicates that an entity with the given identifier (project, domain, name, version) already exists on the Flyte backend. `FlyteRemote` often handles this gracefully by fetching the existing entity instead of attempting to re-register.
<!--
key: summary_executing_flyte_entities_b574b0ec-e61e-4ec4-bdb6-777a6470dbd0
type: summary_end

-->