
<!--
help_text: ''
key: summary_executing_workflows_and_tasks_3543ff3e-b39b-454e-aa69-a1e88bc05937
modules:
- flytekit.clients.friendly.SynchronousFlyteClient
- flytekit.remote.remote.FlyteRemote
- flytekit.remote.executions.FlyteWorkflowExecution
questions_to_answer: []
type: summary

-->
# Executing Workflows and Tasks

The Flyte platform enables the execution of defined tasks and workflows on a remote backend. This documentation describes how to interact with the Flyte control plane to register, launch, monitor, and manage these executions programmatically.

## Connecting to the Flyte Platform

Interaction with the Flyte platform begins by establishing a connection to the Flyte Admin service. The `SynchronousFlyteClient` provides a low-level interface for direct gRPC service calls, while the `FlyteRemote` client offers a more user-friendly, higher-level abstraction for common operations. For most use cases, the `FlyteRemote` client is recommended.

To initialize a `FlyteRemote` client:

```python
from flytekit.remote.remote import FlyteRemote, Config

# Connect to a Flyte sandbox environment
remote = FlyteRemote.for_sandbox(
    default_project="flytesnacks",
    default_domain="development",
    data_upload_location="s3://my-s3-bucket/data" # Override for non-sandbox
)

# Alternatively, connect to a specific endpoint
remote = FlyteRemote.for_endpoint(
    endpoint="your.domain:port",
    insecure=True, # Set to True if SSL is not enabled
    default_project="my_project",
    default_domain="production",
)
```

The `FlyteRemote` client manages configuration, file access, and context, simplifying interactions with the remote Flyte backend. It also provides access to the underlying `SynchronousFlyteClient` via the `client` property for more granular control.

## Registering Entities for Execution

Before a local task or workflow can be executed on the Flyte platform, it must be registered with the Flyte Admin service. Registration makes the entity definition available to the control plane.

The `FlyteRemote` client provides methods to register Python tasks (`PythonTask`), workflows (`WorkflowBase`), and launch plans (`LaunchPlan`).

### Standard Registration

When registering an entity, a unique identifier (project, domain, name, and version) is assigned. If a version is not explicitly provided, the system can automatically generate one based on a hash of the entity's content and serialization settings, especially when interactive mode is enabled.

```python
from flytekit import task, workflow
from flytekit.remote.remote import FlyteRemote
from flytekit.configuration import ImageConfig, SerializationSettings

remote = FlyteRemote.for_sandbox(default_project="flytesnacks", default_domain="development")

@task
def my_task(x: int) -> int:
    return x + 1

@workflow
def my_workflow(x: int) -> int:
    return my_task(x=x)

# Register a task
registered_task = remote.register_task(my_task, version="v1.0.0")
print(f"Registered task: {registered_task.id}")

# Register a workflow (a default launch plan is also created by default)
registered_workflow = remote.register_workflow(my_workflow, version="v1.0.0")
print(f"Registered workflow: {registered_workflow.id}")

# Register a specific launch plan
from flytekit import LaunchPlan
my_lp = LaunchPlan.get_or_create(my_workflow, name="my_workflow_lp")
registered_lp = remote.register_launch_plan(my_lp, version="v1.0.0")
print(f"Registered launch plan: {registered_lp.id}")
```

If an entity with the same identifier already exists, the registration process will typically succeed without overwriting, provided the definitions are identical.

### Script Mode (Fast Registration)

For Python-based tasks and workflows, script mode (also known as fast registration) allows the Flyte platform to execute code directly from a compressed archive uploaded to a data store. This is particularly useful for iterative development or when dealing with large codebases, as it avoids the need for container image rebuilds for every code change.

When using script mode, the `FlyteRemote` client packages the necessary Python files, uploads them, and registers the entity with a reference to the uploaded archive.

```python
import pathlib
import os
from flytekit import task, workflow
from flytekit.remote.remote import FlyteRemote
from flytekit.configuration import FastPackageOptions, CopyFileDetection

remote = FlyteRemote.for_sandbox(default_project="flytesnacks", default_domain="development")

# Assume my_module.py contains the task and workflow definitions
# For demonstration, let's create a dummy file
current_dir = pathlib.Path(__file__).parent.absolute()
module_file = current_dir / "my_module.py"
with open(module_file, "w") as f:
    f.write("""
from flytekit import task, workflow

@task
def script_task(a: int, b: int) -> int:
    return a + b

@workflow
def script_workflow(a: int, b: int) -> int:
    return script_task(a=a, b=b)
""")

# Import the entities from the dummy module
from my_module import script_task, script_workflow

# Register the workflow using script mode
# source_path should be the root directory of your project
# module_name should be the Python module path to your workflow/task
registered_script_wf = remote.register_script(
    entity=script_workflow,
    source_path=str(current_dir),
    module_name="my_module",
    version="script-v1",
    # Optional: Customize packaging behavior
    fast_package_options=FastPackageOptions(
        top_level_packages=["my_module"], # Only package this module
        copy_style=CopyFileDetection.TOP_LEVEL_ONLY # Copy only top-level files
    )
)
print(f"Registered script workflow: {registered_script_wf.id}")

# Clean up the dummy file
os.remove(module_file)
```

The `fast_package_options` parameter allows fine-grained control over which files are included in the uploaded archive, optimizing upload size and time.

## Launching Executions

Once tasks or workflows are registered, they can be launched as executions on the Flyte platform. An execution represents a single run of a task or workflow.

The `execute` method on the `FlyteRemote` client is the primary entry point for launching executions. It supports both already-registered remote entities and local Python entities, handling registration automatically if needed.

### Executing Registered Entities

To execute an entity that is already registered on the Flyte platform, first fetch its definition using `fetch_task`, `fetch_workflow`, or `fetch_launch_plan`, then pass the fetched object to `execute`.

```python
from flytekit.remote.remote import FlyteRemote
from flytekit.models.core.identifier import WorkflowExecutionIdentifier

remote = FlyteRemote.for_sandbox(default_project="flytesnacks", default_domain="development")

# Assuming 'my_workflow' and 'my_workflow_lp' (default launch plan) are registered as 'v1.0.0'
# Fetch the launch plan
fetched_lp = remote.fetch_launch_plan(name="my_workflow_lp", version="v1.0.0")

# Launch an execution using the fetched launch plan
execution_name = f"my_first_execution_{remote.default_project}_{remote.default_domain}"
execution = remote.execute(
    fetched_lp,
    inputs={"x": 10},
    execution_name=execution_name,
    wait=True # Wait for the execution to complete
)

print(f"Execution launched: {execution.id.name}")
print(f"Execution status: {execution.closure.phase}")
```

### Executing Local Entities

The `execute` method can also directly accept local Python task or workflow objects. If the entity is not already registered with the specified project, domain, and version, the `FlyteRemote` client will automatically register it before launching the execution. This simplifies the development loop.

```python
from flytekit import task, workflow
from flytekit.remote.remote import FlyteRemote

remote = FlyteRemote.for_sandbox(default_project="flytesnacks", default_domain="development")

@task
def local_task(y: str) -> str:
    return f"Hello, {y}!"

@workflow
def local_workflow(y: str) -> str:
    return local_task(y=y)

# Execute a local workflow directly. It will be registered if not already present.
# A version will be auto-generated if not provided.
execution = remote.execute(
    local_workflow,
    inputs={"y": "Flyte"},
    project="flytesnacks",
    domain="development",
    execution_name_prefix="local-wf-run", # A unique name will be generated
    wait=True
)

print(f"Local workflow execution status: {execution.closure.phase}")
```

### Providing Inputs

Inputs to tasks and workflows are provided as a Python dictionary mapping input names to their values. The `FlyteRemote` client's `TypeEngine` automatically converts these Python native values into Flyte Literals, handling complex types like `FlyteFile`, `FlyteDirectory`, and `StructuredDataset`.

For large inputs (e.g., files), the client automatically uploads them to the configured data upload location (`data_upload_location` in `FlyteRemote` initialization) and passes references to the Flyte backend.

```python
from flytekit import task, workflow, FlyteFile
from flytekit.remote.remote import FlyteRemote
import os

remote = FlyteRemote.for_sandbox(default_project="flytesnacks", default_domain="development")

@task
def process_file(input_file: FlyteFile) -> str:
    with open(input_file, "r") as f:
        content = f.read()
    return f"Processed: {content}"

@workflow
def file_workflow(input_file: FlyteFile) -> str:
    return process_file(input_file=input_file)

# Create a dummy local file
with open("my_data.txt", "w") as f:
    f.write("This is some test data.")

# Execute the workflow with a local file input
execution = remote.execute(
    file_workflow,
    inputs={"input_file": "my_data.txt"}, # The client handles uploading this file
    execution_name_prefix="file-wf-run",
    wait=True
)

print(f"File workflow execution status: {execution.closure.phase}")
os.remove("my_data.txt")
```

### Execution Options

The `execute` method accepts various optional parameters to customize execution behavior:

*   **`execution_name` / `execution_name_prefix`**: Assign a specific name or a prefix for the execution. If only a prefix is provided, a unique suffix is appended.
*   **`wait`**: A boolean flag to block execution until the remote workflow completes.
*   **`options`**: An `Options` object to configure advanced settings like notifications, labels, annotations, raw output data configuration, maximum parallelism, and security context.
*   **`overwrite_cache`**: If `True`, forces all tasks within the execution to re-run, ignoring cached results.
*   **`interruptible`**: Overrides the default interruptibility setting for the executed entity.
*   **`envs`**: A dictionary of environment variables to set for the execution.
*   **`tags`**: A list of tags to associate with the execution for easier filtering and organization.
*   **`cluster_pool`**: Specifies a cluster pool for the execution.
*   **`execution_cluster_label`**: Specifies a label for the cluster(s) where the execution should be placed.

```python
from flytekit import task, workflow
from flytekit.remote.remote import FlyteRemote, Options
from flytekit.models.core.execution import WorkflowExecutionPhase

remote = FlyteRemote.for_sandbox(default_project="flytesnacks", default_domain="development")

@task
def simple_task(x: int) -> int:
    return x * 2

@workflow
def simple_workflow(x: int) -> int:
    return simple_task(x=x)

# Example with custom options
custom_options = Options(
    labels={"team": "data-science"},
    annotations={"priority": "high"},
    disable_notifications=True,
    max_parallelism=5,
)

execution = remote.execute(
    simple_workflow,
    inputs={"x": 5},
    execution_name="my-custom-execution",
    options=custom_options,
    overwrite_cache=True,
    interruptible=True,
    envs={"MY_ENV_VAR": "value"},
    tags=["test", "production"],
    wait=True
)

print(f"Execution '{execution.id.name}' completed with phase: {WorkflowExecutionPhase.enum_to_string(execution.closure.phase)}")
```

## Monitoring and Managing Executions

After launching an execution, developers can monitor its progress, retrieve results, and manage its lifecycle.

### Retrieving Execution Details

The `fetch_execution` method retrieves a `FlyteWorkflowExecution` object by its project, domain, and name. This object provides access to the execution's current state, metadata, and associated data.

```python
from flytekit.remote.remote import FlyteRemote
from flytekit.models.core.execution import WorkflowExecutionPhase

remote = FlyteRemote.for_sandbox(default_project="flytesnacks", default_domain="development")

# Assuming an execution named 'my-custom-execution' exists
execution_name = "my-custom-execution"
fetched_execution = remote.fetch_execution(
    project="flytesnacks",
    domain="development",
    name=execution_name
)

print(f"Fetched execution '{fetched_execution.id.name}' status: {WorkflowExecutionPhase.enum_to_string(fetched_execution.closure.phase)}")
print(f"Execution URL: {fetched_execution.execution_url}")
```

The `sync` method on a `FlyteWorkflowExecution` object updates its local state with the latest information from the Flyte Admin service.

```python
# Sync the execution to get the latest status
fetched_execution.sync()
print(f"Synced execution status: {WorkflowExecutionPhase.enum_to_string(fetched_execution.closure.phase)}")
```

### Accessing Inputs and Outputs

The `inputs` and `outputs` properties of a `FlyteWorkflowExecution` (or `FlyteNodeExecution`, `FlyteTaskExecution`) provide access to the literal maps of data passed into and out of the execution. These are resolved into Python native types by the `LiteralsResolver`.

```python
# Assuming 'execution' is a completed FlyteWorkflowExecution object
if execution.is_successful:
    print(f"Execution inputs: {execution.inputs.to_dict()}")
    print(f"Execution outputs: {execution.outputs.to_dict()}")

    # Access specific outputs
    output_value = execution.outputs["o"] # 'o' is the output variable name
    print(f"Specific output 'o': {output_value}")
else:
    print(f"Execution failed or is not complete. Error: {execution.error.message if execution.error else 'N/A'}")
```

For large inputs/outputs stored remotely, the `download` method on `FlyteRemote` can be used to retrieve them to a local path.

```python
import os

# Download all outputs to a local directory
download_path = "./execution_outputs"
os.makedirs(download_path, exist_ok=True)
remote.download(execution.outputs, download_path, recursive=True)
print(f"Outputs downloaded to: {download_path}")
```

### Inspecting Node and Task Executions

Workflows are composed of nodes, and nodes can execute tasks or sub-workflows. The `node_executions` property of a `FlyteWorkflowExecution` provides a dictionary of `FlyteNodeExecution` objects, allowing inspection of individual steps within the workflow. Each `FlyteNodeExecution` can, in turn, expose its `task_executions` or `workflow_executions` for deeper inspection.

To retrieve node execution details, ensure `sync_nodes=True` is passed to `sync` or `wait`.

```python
# Sync with node details
execution.sync(sync_nodes=True)

for node_id, node_exec in execution.node_executions.items():
    print(f"Node '{node_id}' status: {node_exec.closure.phase}")
    if node_exec.task_executions:
        for task_exec in node_exec.task_executions.values():
            print(f"  Task '{task_exec.id.task_id.name}' status: {task_exec.closure.phase}")
            if task_exec.is_successful:
                print(f"  Task outputs: {task_exec.outputs.to_dict()}")
```

### Waiting for Completion

The `wait` method on `FlyteWorkflowExecution` is a blocking call that pauses execution until the remote workflow completes, fails, or times out.

```python
from datetime import timedelta

# Wait for up to 5 minutes, polling every 10 seconds
execution.wait(timeout=timedelta(minutes=5), poll_interval=timedelta(seconds=10))

if execution.is_successful:
    print("Execution completed successfully!")
else:
    print(f"Execution failed with error: {execution.error.message}")
```

### Terminating and Relaunching

Executions can be terminated prematurely or relaunched from a specific point.

*   **`terminate_execution(id, cause)`**: Stops a running workflow execution.
*   **`relaunch_execution(id, name)`**: Relaunches a completed or failed execution. This creates a new execution that re-runs the entire workflow.
*   **`recover_execution(id, name)`**: Recovers a previously failed execution, restarting from the last known failure point.

```python
# Terminate an execution
# remote.terminate_execution(execution.id, "User requested termination")

# Relaunch a completed execution
# relaunched_execution = remote.relaunch_execution(execution.id, name="my-relaunch-run")
# print(f"Relaunched execution: {relaunched_execution.id.name}")

# Recover a failed execution
# recovered_execution = remote.recover_execution(failed_execution.id, name="my-recovery-run")
# print(f"Recovered execution: {recovered_execution.id.name}")
```

### Handling Signals (Human-in-the-loop)

Flyte supports human-in-the-loop workflows using `Signal` nodes. The `FlyteRemote` client provides methods to list, approve, reject, or set inputs for these signals.

*   **`list_signals(execution_name, project, domain)`**: Retrieves pending signals for an execution.
*   **`approve(signal_id, execution_name, project, domain)`**: Approves a boolean signal.
*   **`reject(signal_id, execution_name, project, domain)`**: Rejects a boolean signal.
*   **`set_input(signal_id, execution_name, value, ...)`**: Sets the value for a signal that expects input.

```python
# Assuming a workflow with a signal node is running
# signals = remote.list_signals(execution_name="my-signal-workflow-run")
# for signal in signals:
#     print(f"Found signal: {signal.id.signal_id} with type {signal.type}")
#     if signal.type.simple == flytekit.models.core.types.SimpleType.BOOLEAN:
#         remote.approve(signal.id.signal_id, "my-signal-workflow-run")
#     elif signal.type.simple == flytekit.models.core.types.SimpleType.INTEGER:
#         remote.set_input(signal.id.signal_id, "my-signal-workflow-run", 123, python_type=int)
```

## Advanced Execution Scenarios

### Launching Backfills

The `launch_backfill` method simplifies the process of re-running a scheduled workflow for a range of past dates. It generates a new workflow that orchestrates individual launch plan executions for each specified interval.

```python
from datetime import datetime, timedelta
from flytekit.remote.remote import FlyteRemote

remote = FlyteRemote.for_sandbox(default_project="flytesnacks", default_domain="development")

# Assuming 'my_workflow_lp' is a registered launch plan
backfill_execution = remote.launch_backfill(
    project="flytesnacks",
    domain="development",
    launchplan="my_workflow_lp",
    from_date=datetime(2023, 1, 1),
    to_date=datetime(2023, 1, 3),
    parallel=True, # Run daily executions in parallel
    execute=True, # Register and execute the backfill workflow
    execution_name="daily-backfill-jan-2023",
    overwrite_cache=True,
    # dry_run=True # Set to True to inspect the generated workflow without executing
)

if backfill_execution:
    print(f"Backfill execution launched: {backfill_execution.id.name}")
    backfill_execution.wait()
    print(f"Backfill execution status: {backfill_execution.closure.phase}")
```

### Retrieving Execution Metrics

The `get_execution_metrics` method on `FlyteRemote` allows fetching detailed execution metrics, including span information, which can be useful for performance analysis and debugging.

```python
# Assuming 'execution' is a FlyteWorkflowExecution object
metrics_span = remote.get_execution_metrics(execution.id, depth=5)
print(f"Execution metrics root span: {metrics_span.name}")
# Further process metrics_span for detailed timing and resource usage
```

## Best Practices and Considerations

*   **Version Management**: Always use meaningful versions for your tasks, workflows, and launch plans. While auto-versioning is convenient for development, explicit versions are crucial for reproducibility and deployment.
*   **Error Handling**: Implement robust error handling around API calls to the Flyte Admin service, as network issues or invalid requests can lead to `grpc.RpcError` or `FlyteEntityAlreadyExistsException`/`FlyteEntityNotExistException`.
*   **Paginated APIs**: When listing entities (tasks, workflows, executions), be aware that many `SynchronousFlyteClient` methods are paginated. Use the `limit` and `token` parameters to retrieve all results. The `FlyteRemote` client's `list_*` methods simplify this by handling pagination internally for common use cases.
*   **Data Transfer**: For large inputs/outputs, ensure your `data_upload_location` is configured correctly and has appropriate permissions. The `FlyteRemote` client handles the underlying data proxy interactions for signed URLs.
*   **Interactive Mode**: While `FlyteRemote` supports interactive environments like Jupyter notebooks, be mindful that pickling tasks/workflows for execution can sometimes lead to serialization issues if the code has complex dependencies or non-serializable objects.
*   **Console Integration**: Leverage the `generate_console_url` method to quickly access the Flyte UI for visual monitoring and debugging of executions.
<!--
key: summary_executing_workflows_and_tasks_3543ff3e-b39b-454e-aa69-a1e88bc05937
type: summary_end

-->
<!--
code_unit: flytekit.clients.friendly.SynchronousFlyteClient.create_execution
code_unit_type: class
help_text: ''
key: example_a0a7d085-66b6-4913-8bdf-133365ad24f9
type: example

-->
<!--
code_unit: flytekit.remote.remote.FlyteRemote.execute
code_unit_type: class
help_text: ''
key: example_0f947e81-cad4-4851-a82c-cf5038222225
type: example

-->
<!--
code_unit: flytekit.remote.remote.FlyteRemote.execute_remote_task_lp
code_unit_type: class
help_text: ''
key: example_091dec2f-0969-41b0-8e41-440f68b1d67c
type: example

-->
<!--
code_unit: flytekit.remote.remote.FlyteRemote.execute_remote_wf
code_unit_type: class
help_text: ''
key: example_881e97af-088c-4c00-9517-63d732bae75f
type: example

-->
<!--
code_unit: flytekit.remote.remote.FlyteRemote.execute_local_task
code_unit_type: class
help_text: ''
key: example_58832323-32aa-415a-ad40-be52ef29d595
type: example

-->
<!--
code_unit: flytekit.remote.remote.FlyteRemote.execute_local_workflow
code_unit_type: class
help_text: ''
key: example_00e11785-e654-4462-aa35-131f5f9a6f53
type: example

-->
<!--
code_unit: flytekit.remote.remote.FlyteRemote.execute_local_launch_plan
code_unit_type: class
help_text: ''
key: example_251b29d9-dbf4-4e15-b34d-c2b7a5152925
type: example

-->