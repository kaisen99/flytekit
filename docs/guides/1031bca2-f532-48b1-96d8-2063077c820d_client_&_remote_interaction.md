
<!--
help_text: ''
key: summary_client_&_remote_interaction_99932f74-598d-46c3-9b21-007fed534a67
modules:
- flytekit.clients.friendly
- flytekit.clients.raw
- flytekit.clients.grpc_utils.auth_interceptor
- flytekit.clients.grpc_utils.default_metadata_interceptor
- flytekit.clients.grpc_utils.wrap_exception_interceptor
- flytekit.remote.entities
- flytekit.remote.executions
- flytekit.remote.interface
- flytekit.remote.lazy_entity
- flytekit.remote.metrics
- flytekit.remote.remote
- flytekit.remote.remote_callable
- flytekit.remote.remote_fs
- flytekit.interfaces.cli_identifiers
questions_to_answer: []
type: summary

-->
## Client & Remote Interaction

Interacting with the Flyte platform programmatically involves connecting to the Flyte Admin service to manage and execute Flyte entities such as tasks, workflows, launch plans, and their executions. This section details the core components and capabilities for client and remote interaction.

### Connecting to the Flyte Control Plane

The primary way to interact with the Flyte Admin service is through a client.

The `SynchronousFlyteClient` provides a user-friendly interface for making direct gRPC service calls to the control plane. It simplifies interactions by handling the underlying protocol buffer conversions and providing Pythonic methods for common operations.

To create a client:

```python
from flytekit.clients.friendly import SynchronousFlyteClient
from flytekit.configuration import Config, PlatformConfig

# For a local sandbox environment
client = SynchronousFlyteClient(PlatformConfig("localhost:30080", insecure=True))

# For a production deployment (assuming SSL enabled)
# client = SynchronousFlyteClient(PlatformConfig("your.domain:port", insecure=False))
```

This client is built on top of a lower-level `RawSynchronousFlyteClient`, which is a thin wrapper around the auto-generated gRPC stubs. The `SynchronousFlyteClient` enhances this raw client with:

*   **Authentication**: Automatically handles authentication metadata and token refreshing for gRPC calls.
*   **Default Metadata**: Injects standard gRPC metadata, such as `accept: application/grpc`.
*   **Exception Wrapping**: Converts raw gRPC errors into more specific and user-friendly Flyte exceptions (e.g., `FlyteEntityAlreadyExistsException`, `FlyteEntityNotExistException`, `FlyteInvalidInputException`, `FlyteSystemUnavailableException`, `FlyteTimeout`). This simplifies error handling for developers.
*   **Retries**: Implements retry logic for transient errors, improving robustness.

### High-Level Remote Operations with `FlyteRemote`

For most common use cases, the `FlyteRemote` class offers a higher-level, more convenient abstraction over the `SynchronousFlyteClient`. It streamlines workflows for fetching, registering, and executing Flyte entities, and provides integrated file access.

#### Initializing `FlyteRemote`

You can initialize `FlyteRemote` in several ways, depending on your environment:

*   **Automatic Configuration**: Infers configuration from environment variables or a Flyte configuration file.

    ```python
    from flytekit.remote import FlyteRemote
    remote = FlyteRemote.auto()
    ```

*   **Specific Endpoint**: Connects to a specified Flyte Admin endpoint.

    ```python
    remote = FlyteRemote.for_endpoint("your.domain:port", insecure=True)
    ```

*   **Sandbox Environment**: Pre-configured for a local Flyte sandbox.

    ```python
    remote = FlyteRemote.for_sandbox()
    ```

When initializing, you can specify `default_project`, `default_domain`, and `data_upload_location` to simplify subsequent operations. The `interactive_mode_enabled` flag (automatically detected in Jupyter notebooks) enables features like pickling local entities for registration.

#### Fetching Entities

Retrieve existing tasks, workflows, and launch plans from the Flyte Admin service.

*   **`fetch_task(project, domain, name, version)`**: Fetches a specific task. If `version` is `None`, it retrieves the latest version.
*   **`fetch_workflow(project, domain, name, version)`**: Fetches a specific workflow.
*   **`fetch_launch_plan(project, domain, name, version)`**: Fetches a specific launch plan.
*   **`fetch_active_launchplan(project, domain, name)`**: Retrieves the currently active version of a launch plan.

```python
# Fetch the latest version of a task
my_remote_task = remote.fetch_task(project="flytesnacks", domain="development", name="my_task")

# Fetch a specific version of a workflow
my_remote_workflow = remote.fetch_workflow(project="flytesnacks", domain="development", name="my_wf", version="abcdef123")

# Fetch an active launch plan
my_active_lp = remote.fetch_active_launchplan(project="flytesnacks", domain="development", name="my_lp")
```

For scenarios where you need to defer fetching an entity until it's actually accessed, use the `fetch_task_lazy` or `fetch_workflow_lazy` methods. These return a `LazyEntity` object, which acts as a promise to fetch the actual entity only when its attributes are accessed or it is called.

```python
lazy_task = remote.fetch_task_lazy(project="flytesnacks", domain="development", name="my_task")
# The task is not fetched yet
print(lazy_task.name) # Task is fetched here
```

#### Registering Entities

Register local Flyte entities (Python tasks, workflows, and launch plans) with the Flyte Admin service. Registration makes your code available for execution on the Flyte platform.

*   **`register_task(entity, serialization_settings, version)`**: Registers a Python task.
*   **`register_workflow(entity, serialization_settings, version, default_launch_plan, options)`**: Registers a Python workflow. By default, it also creates a default launch plan for the workflow.
*   **`register_launch_plan(entity, version, project, domain, options, serialization_settings)`**: Registers a launch plan. If the underlying workflow is not already registered, it will be registered as well.

```python
from flytekit import task, workflow
from flytekit.remote import FlyteRemote
from flytekit.configuration import SerializationSettings, ImageConfig

@task
def my_local_task(x: int) -> int:
    return x + 1

@workflow
def my_local_workflow(x: int) -> int:
    return my_local_task(x=x)

remote = FlyteRemote.for_sandbox(default_project="flytesnacks", default_domain="development")

# Register a task
registered_task = remote.register_task(my_local_task, version="1.0.0")

# Register a workflow (and its default launch plan)
registered_workflow = remote.register_workflow(my_local_workflow, version="1.0.0")
```

**Fast Registration and Script Mode**:
For efficient registration of Python code, especially in interactive environments or when dealing with large codebases, use `fast_register_workflow` or `register_script`. These methods package your code (e.g., into a zip file) and upload it to remote storage, then register the entity with a reference to this location.

*   **`fast_package(root, output, deref_symlinks, options)`**: Packages local files into an installable zip and returns its MD5 hash and uploaded URL.
*   **`upload_file(to_upload, project, domain, filename_root)`**: Uploads a single file to the configured data proxy service, returning its MD5 hash and native URL.

```python
# Example using register_script for a workflow
# Assuming my_local_workflow is defined in a file within your project root
registered_workflow_script = remote.register_script(
    my_local_workflow,
    version="1.0.0",
    source_path="/path/to/your/project/root",
    module_name="your_module.your_file", # e.g., "my_project.workflows.my_workflow_file"
)
```

#### Executing Entities

The `execute` method provides a unified way to run tasks, workflows, or launch plans, whether they are locally defined Python objects, fetched remote entities, or reference entities.

```python
from flytekit import task, workflow
from flytekit.remote import FlyteRemote

@task
def add_one(x: int) -> int:
    return x + 1

@workflow
def my_wf(x: int) -> int:
    return add_one(x=x)

remote = FlyteRemote.for_sandbox(default_project="flytesnacks", default_domain="development")

# Execute a local workflow (will register if not exists)
execution = remote.execute(my_wf, inputs={"x": 5}, execution_name_prefix="my_first_exec")

# Execute a fetched remote task
remote_task = remote.fetch_task(project="flytesnacks", domain="development", name="my_task", version="1.0.0")
execution_from_remote_task = remote.execute(remote_task, inputs={"x": 10})

# Wait for an execution to complete
execution.wait()
print(f"Execution {execution.id.name} finished with outputs: {execution.outputs.get('o0')}")
```

The `execute` method supports various options:

*   **`inputs`**: A dictionary mapping input names to Python native values. The system automatically converts these to Flyte Literals.
*   **`execution_name` / `execution_name_prefix`**: Control the name of the created execution for idempotency or organization.
*   **`wait`**: A boolean flag to block execution until the remote execution completes.
*   **`options`**: An `Options` object to configure execution-level settings like `overwrite_cache`, `interruptible`, `max_parallelism`, `labels`, `annotations`, `notifications`, `raw_output_data_config`, `security_context`.
*   **`envs`**: Environment variables to set for the execution.
*   **`tags`**: Tags to set for the execution.
*   **`cluster_pool` / `execution_cluster_label`**: Specify where the execution should be placed.
*   **`type_hints`**: Optional type hints for input conversion, especially useful for complex types or when automatic type guessing is insufficient.

#### Managing Executions

Once an execution is launched, you can monitor and interact with it using `FlyteWorkflowExecution`, `FlyteNodeExecution`, and `FlyteTaskExecution` objects.

*   **`sync(execution, sync_nodes=True)`**: Updates the local execution object with the latest state from the remote. Setting `sync_nodes=True` recursively fetches information about all underlying node and task executions.
*   **`wait(execution, timeout, poll_interval, sync_nodes)`**: Blocks until the execution reaches a terminal state (succeeded, failed, aborted, timed out).
*   **`terminate(execution, cause)`**: Stops a running workflow execution.
*   **`relaunch_execution(id, name)`**: Re-runs a completed execution from scratch.
*   **`recover_execution(id, name)`**: Re-runs a failed execution from the last known failure point.
*   **`get_execution_metrics(id, depth)`**: Retrieves detailed performance metrics for a workflow execution, represented by a `FlyteExecutionSpan` object.

```python
# Sync an execution
execution.sync()
print(f"Current phase: {execution.closure.phase}")

# Access execution outputs (only available if successful)
if execution.is_successful:
    print(f"Outputs: {execution.outputs.to_dict()}")

# Access execution errors (if failed)
if execution.error:
    print(f"Error: {execution.error.message}")

# Get node executions
for node_id, node_exec in execution.node_executions.items():
    print(f"Node {node_id} phase: {node_exec.closure.phase}")
    if node_exec.task_executions:
        for task_exec in node_exec.task_executions:
            print(f"  Task {task_exec.id.retry_attempt} phase: {task_exec.closure.phase}")

# Terminate an execution
# remote.terminate(execution, "Stopping due to manual intervention")

# Get execution metrics
metrics_span = remote.get_execution_metrics(execution.id)
metrics_span.explain() # Prints a human-readable tree of execution times
```

#### Interacting with Signals

Signals enable human-in-the-loop workflows by allowing external input to unblock a workflow execution.

*   **`list_signals(execution_name, project, domain, limit, filters)`**: Lists pending signals for an execution.
*   **`approve(signal_id, execution_name, project, domain)`**: Approves a boolean signal.
*   **`reject(signal_id, execution_name, project, domain)`**: Rejects a boolean signal.
*   **`set_input(signal_id, execution_name, value, project, domain, python_type, literal_type)`**: Sets the value for a signal that expects a specific input type.

```python
# Assuming a workflow is paused waiting for a signal 'user_approval'
# remote.approve(signal_id="user_approval", execution_name="my_paused_execution")

# Setting a complex input for a signal
# remote.set_input(
#     signal_id="data_input",
#     execution_name="my_data_workflow",
#     value={"key": "value", "number": 123},
#     python_type=typing.Dict[str, typing.Union[str, int]]
# )
```

#### File System Integration (`flyte://`)

The `FlyteRemote` object integrates with `fsspec` to provide a seamless file system interface for Flyte-specific URIs (`flyte://`). This allows you to interact with remote data locations as if they were local files, especially useful for `FlyteFile`, `FlyteDir`, and `StructuredDataset` types.

*   **`download(data, download_to, recursive)`**: Downloads data (LiteralsResolver, Literal, LiteralMap) to a local path. If `recursive` is `True` and the data contains file-like objects (e.g., `FlyteFile`, `FlyteDir`), it downloads all referenced files.
*   **`get(flyte_uri)`**: A general-purpose method to resolve Flyte tiny URLs. It can return `LiteralsResolver` for output data, `Literal` for single values, or HTML/bytes for deck links.

```python
import os
from flytekit import FlyteFile

# Assuming 'my_file_output' is a FlyteFile output from an execution
# remote.download(execution.outputs.get("my_file_output"), "/tmp/downloaded_file.txt")

# Download all outputs from an execution
# remote.download(execution.outputs, "/tmp/all_outputs", recursive=True)

# Accessing a deck link
# deck_html = remote.get("flyte://console/projects/flytesnacks/domains/development/executions/my_exec/deck")
# if isinstance(deck_html, HTML):
#     display(deck_html) # In a Jupyter environment
```

#### Project and Domain Management

Manage projects and domains registered with the Flyte Admin service.

*   **`list_projects(limit, filters, sort_by)`**: Lists all registered projects.
*   **`get_domains()`**: Lists all registered domains.
*   **`register_project(project)`**: Registers a new project.
*   **`update_project(project)`**: Updates an existing project.

#### Backfill Workflows

The `launch_backfill` method allows you to create and execute workflows that re-run a launch plan for a specified date range. This is useful for re-processing historical data or recovering from past failures.

```python
from datetime import datetime, timedelta

# Launch a backfill for a launch plan
# backfill_exec = remote.launch_backfill(
#     project="flytesnacks",
#     domain="development",
#     from_date=datetime(2023, 1, 1),
#     to_date=datetime(2023, 1, 3),
#     launchplan="my_daily_lp",
#     parallel=True, # Run each daily execution in parallel
# )
# backfill_exec.wait()
```

#### Console Integration

Generate direct links to the Flyte Console UI for quick inspection of entities and executions.

*   **`generate_console_url(entity)`**: Returns a URL to the Flyte Console for the given entity (task, workflow, launch plan, or execution).

```python
console_url = remote.generate_console_url(execution)
print(f"View execution in console: {console_url}")
```

### Identifiers

Flyte uses a consistent identification scheme for all entities and executions. These identifiers are crucial for fetching, registering, and interacting with objects on the platform.

*   **`Identifier`**: Used for tasks, workflows, and launch plans. It includes `resource_type` (e.g., `ResourceType.TASK`), `project`, `domain`, `name`, and `version`.
*   **`WorkflowExecutionIdentifier`**: Used for workflow executions, comprising `project`, `domain`, and `name`.
*   **`TaskExecutionIdentifier`**: Used for task executions, including `task_id`, `node_execution_id`, and `retry_attempt`.

These identifier objects can be constructed from their components or parsed from a standard string format (e.g., `tsk:project:domain:name:version`).

### Error Handling

The client automatically wraps gRPC errors into specific Flyte exceptions, making it easier to handle different failure scenarios:

*   **`FlyteEntityAlreadyExistsException`**: Raised when attempting to create an entity that already exists with the same identifier.
*   **`FlyteEntityNotExistException`**: Raised when attempting to fetch or interact with an entity that does not exist.
*   **`FlyteInvalidInputException`**: Indicates an issue with the provided input arguments.
*   **`FlyteSystemUnavailableException`**: Signifies that the Flyte Admin service is unreachable or experiencing issues.
*   **`FlyteTimeout`**: Raised when a blocking operation (like `wait`) exceeds its specified timeout.

Developers should catch these specific exceptions to implement robust error recovery or user feedback mechanisms.

### Best Practices and Considerations

*   **Version Management**: Always consider versioning when registering entities. While `FlyteRemote` can auto-generate versions in interactive mode, explicitly providing a version (e.g., using Git SHAs or semantic versions) is recommended for production deployments.
*   **Paginated APIs**: Many listing methods (e.g., `list_tasks_paginated`, `list_executions_paginated`) are paginated. They return a list of entities and a `token` for fetching the next page. Implement pagination logic when retrieving large numbers of entities.
*   **Data Offloading**: For large inputs or outputs, Flyte automatically offloads data to configured blob storage. The `flyte://` file system integration simplifies interacting with these offloaded data.
*   **Security**: When connecting to a Flyte deployment, ensure you configure SSL appropriately (`insecure=False`) and provide necessary credentials for secure communication.
*   **Interactive Mode**: While convenient for development, be aware that pickling local code for registration can sometimes lead to unexpected behavior if the environment where the code is pickled differs significantly from the execution environment. For production, prefer `register_script` or pre-compiled packages.
<!--
key: summary_client_&_remote_interaction_99932f74-598d-46c3-9b21-007fed534a67
type: summary_end

-->
<!--
code_unit: flytekit.clients.friendly.SynchronousFlyteClient
code_unit_type: class
help_text: ''
key: example_fd0acb36-dfc4-4ea5-9c31-f5ca54522966
type: example

-->
<!--
code_unit: flytekit.remote.remote.FlyteRemote
code_unit_type: class
help_text: ''
key: example_a2d5a2e5-7ee4-4eac-9ad1-95f271a54f1a
type: example

-->
<!--
code_unit: flytekit.remote.executions.FlyteWorkflowExecution
code_unit_type: class
help_text: ''
key: example_2a23c549-a705-47b4-8e6e-423044d18317
type: example

-->