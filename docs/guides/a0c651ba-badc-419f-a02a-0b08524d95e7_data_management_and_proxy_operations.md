
<!--
help_text: ''
key: summary_data_management_and_proxy_operations_73a9b1de-ec6a-44d5-b794-b8c89028f7ed
modules:
- flytekit.remote.remote
- flytekit.remote.remote_fs
questions_to_answer: []
type: summary

-->
Data Management and Proxy Operations

The `FlyteRemote` class serves as the primary interface for interacting programmatically with a Flyte backend. It enables developers to manage, register, execute, and monitor Flyte entities such as tasks, workflows, and launch plans, as well as handle data transfer operations.

### Connecting to Flyte

To begin interacting with a Flyte backend, initialize a `FlyteRemote` object. This object requires configuration details to establish a connection to the Flyte Admin server.

**Initialization:**

The `FlyteRemote` class offers several convenient class methods for initialization:

*   **`FlyteRemote.for_endpoint(endpoint: str, ...)`**: Connects to a specific Flyte Admin endpoint.
    ```python
    from flytekit.remote import FlyteRemote, Config

    # Connect to a local sandbox environment
    remote = FlyteRemote.for_endpoint(
        endpoint="localhost:30080",
        insecure=True, # Use True for local sandbox, False for production with SSL
        default_project="flytesnacks",
        default_domain="development",
        data_upload_location="s3://my-s3-bucket/data" # Override for non-sandbox environments
    )
    ```
*   **`FlyteRemote.auto(...)`**: Automatically detects configuration from environment variables or a specified configuration file.
    ```python
    remote = FlyteRemote.auto(
        default_project="my_project",
        default_domain="production",
    )
    ```
*   **`FlyteRemote.for_sandbox(...)`**: Pre-configured for a local Flyte sandbox environment.
    ```python
    remote = FlyteRemote.for_sandbox(
        default_project="flytesnacks",
        default_domain="development",
    )
    ```

During initialization, you can specify `default_project` and `default_domain` to set the context for subsequent operations. The `data_upload_location` parameter is critical, defining the default storage location for data inputs and outputs when interacting with the Flyte backend. For non-sandbox environments, ensure this is configured to a suitable cloud storage bucket (e.g., S3, GCS, Azure Blob Storage).

### Managing Flyte Entities

The `FlyteRemote` object provides comprehensive capabilities for managing tasks, workflows, and launch plans on the Flyte platform.

#### Fetching Entities

Retrieve existing Flyte entities from the backend using their identifiers (project, domain, name, and optionally version).

*   **`fetch_task(project, domain, name, version)`**: Fetches a specific task.
*   **`fetch_workflow(project, domain, name, version)`**: Fetches a specific workflow.
*   **`fetch_launch_plan(project, domain, name, version)`**: Fetches a specific launch plan.
*   **`fetch_active_launchplan(project, domain, name)`**: Retrieves the currently active version of a launch plan.
*   **`fetch_execution(project, domain, name)`**: Fetches a specific workflow execution.

For example, to fetch the latest version of a workflow:

```python
from flytekit.remote import FlyteRemote

remote = FlyteRemote.for_sandbox(default_project="flytesnacks", default_domain="development")

# Fetch the latest version of a workflow
my_workflow = remote.fetch_workflow(name="my_example_workflow")
print(f"Fetched workflow: {my_workflow.id.name} v{my_workflow.id.version}")

# Fetch a specific task version
my_task = remote.fetch_task(name="my_example_task", version="abcdef123")
```

The `fetch_task_lazy` and `fetch_workflow_lazy` methods return `LazyEntity` objects, which defer the actual fetching until the entity is accessed, useful for scenarios where the entity might not be immediately needed.

#### Registering Entities

Registering entities makes your local Python tasks, workflows, and launch plans available on the Flyte backend for execution.

*   **`register_task(entity, serialization_settings, version)`**: Registers a Python task.
*   **`register_workflow(entity, serialization_settings, version, default_launch_plan, options)`**: Registers a Python workflow. By default, this also creates a default launch plan for the workflow.
*   **`register_launch_plan(entity, version, project, domain, options, serialization_settings)`**: Registers a launch plan. If the underlying workflow is not registered, it will be registered as well.

**Serialization Settings and Versioning:**

Registration requires `SerializationSettings`, which define how your code is packaged and deployed. This includes the Docker image to use (`image_config`), project/domain context, and the version. If a `version` is not explicitly provided, `FlyteRemote` can automatically generate one based on the content hash of your code, especially when `interactive_mode_enabled` is set to `True`.

```python
from flytekit import task, workflow
from flytekit.remote import FlyteRemote, SerializationSettings, ImageConfig

remote = FlyteRemote.for_sandbox(default_project="flytesnacks", default_domain="development")

@task
def my_task(a: int, b: int) -> int:
    return a + b

@workflow
def my_workflow(x: int, y: int) -> int:
    return my_task(a=x, b=y)

# Register the task and workflow
# A default image will be used if not specified in SerializationSettings
task_remote = remote.register_task(my_task)
workflow_remote = remote.register_workflow(my_workflow)

print(f"Registered task: {task_remote.id.name} v{task_remote.id.version}")
print(f"Registered workflow: {workflow_remote.id.name} v{workflow_remote.id.version}")
```

**Script Mode Registration (`register_script`, `fast_register_workflow`):**

For more efficient and robust deployments, especially for larger codebases, use script mode registration. This packages your Python code into a zip file and uploads it to the Flyte backend, allowing the Flyte engine to execute your code directly from storage.

*   **`register_script(...)`**: A versatile method for registering tasks, workflows, or launch plans by packaging the source code.
*   **`fast_register_workflow(...)`**: A specialized version of `register_script` optimized for Python function workflows, enabling faster packaging.

```python
# Example using register_script (assuming my_workflow is defined in a file)
# This will package the source code and register the workflow.
# source_path should be the root directory of your project
# module_name should be the Python module path to your workflow (e.g., "my_project.workflows.my_workflow_file")

# remote.register_script(
#     entity=my_workflow,
#     project="flytesnacks",
#     domain="development",
#     source_path="/path/to/my/project/root",
#     module_name="my_module.my_workflow_file",
#     version="unique_version_string"
# )
```

#### Listing Entities

Query and list various entities registered on the Flyte platform.

*   **`recent_executions(project, domain, limit, filters)`**: Lists recent workflow executions.
*   **`list_tasks_by_version(version, project, domain, limit)`**: Lists tasks matching a specific version.
*   **`list_projects(limit, filters, sort_by)`**: Retrieves a list of registered projects.
*   **`get_domains()`**: Retrieves a list of registered domains.

```python
# List recent executions in the default project/domain
recent_execs = remote.recent_executions(limit=5)
for exec_obj in recent_execs:
    print(f"Execution: {exec_obj.id.name} Status: {exec_obj.closure.phase.name}")

# List all projects
projects = remote.list_projects()
for p in projects:
    print(f"Project: {p.id}")
```

### Executing Workflows and Tasks

The `execute` method is the primary entry point for launching tasks, workflows, or launch plans on the Flyte backend. It supports both locally defined Python entities and entities already registered on the remote Flyte platform.

*   **`execute(entity, inputs, project, domain, name, version, execution_name, ...)`**: Executes the specified entity with the given inputs.

When executing a local Python entity (e.g., a `@task` or `@workflow` decorated function), `execute` first attempts to find a matching registered entity. If not found, it automatically compiles and registers the entity before initiating the execution.

**Input Handling:**

Inputs are provided as a dictionary mapping argument names to Python native values. The `TypeEngine` automatically converts these Python values into Flyte Literals for the execution. For complex types or when automatic type guessing is insufficient, provide `type_hints` to guide the conversion.

**Execution Options:**

The `execute` method supports various options to customize the execution behavior:

*   `execution_name` / `execution_name_prefix`: Assign a custom name or prefix to the execution for easier identification.
*   `wait`: If `True`, the method blocks until the execution completes, returning the final `FlyteWorkflowExecution` object.
*   `overwrite_cache`: Forces re-execution of all tasks, ignoring cached results.
*   `interruptible`: Controls whether the execution can be interrupted.
*   `envs`: Environment variables to set for the execution.
*   `tags`: Arbitrary tags for the execution.
*   `cluster_pool`, `execution_cluster_label`: Directs the execution to specific compute resources.

```python
# Execute a registered workflow
execution = remote.execute(
    workflow_remote,
    inputs={"x": 10, "y": 20},
    execution_name="my_first_remote_execution",
    wait=True, # Wait for the execution to complete
)

if execution.is_successful():
    print(f"Execution {execution.id.name} succeeded!")
    # Retrieve outputs
    outputs = execution.outputs
    print(f"Outputs: {outputs}")
else:
    print(f"Execution {execution.id.name} failed with error: {execution.error}")
```

### Monitoring and Controlling Executions

`FlyteRemote` provides tools to monitor the status of running executions and interact with them.

#### Syncing Execution State

*   **`sync_execution(execution, sync_nodes)`**: Updates a `FlyteWorkflowExecution` object with the latest state from the Flyte Admin backend, including its phase, inputs, and outputs. Setting `sync_nodes=True` recursively fetches the state of all underlying node and task executions.
*   **`sync_node_execution(execution, node_mapping)`**: Updates a `FlyteNodeExecution` object.
*   **`sync_task_execution(execution, entity_interface)`**: Updates a `FlyteTaskExecution` object.

```python
# Assuming 'execution' is a FlyteWorkflowExecution object
updated_execution = remote.sync_execution(execution, sync_nodes=True)

# Access updated status and node details
print(f"Current status: {updated_execution.closure.phase.name}")
for node_id, node_exec in updated_execution.node_executions.items():
    print(f"  Node {node_id} status: {node_exec.closure.phase.name}")
```

#### Waiting for Completion

*   **`wait(execution, timeout, poll_interval, sync_nodes)`**: Blocks execution until the specified `FlyteWorkflowExecution` completes or a timeout is reached.

#### Terminating Executions

*   **`terminate(execution, cause)`**: Stops a running workflow execution.

```python
# Terminate a running execution
# remote.terminate(execution, "User requested termination")
```

#### Human-in-the-Loop Operations (Signals)

Flyte supports human-in-the-loop workflows using signals. `FlyteRemote` allows you to interact with these signals.

*   **`list_signals(execution_name, project, domain, limit, filters)`**: Lists pending signals for an execution.
*   **`approve(signal_id, execution_name, project, domain)`**: Approves a boolean signal.
*   **`reject(signal_id, execution_name, project, domain)`**: Rejects a boolean signal.
*   **`set_input(signal_id, execution_name, value, project, domain, python_type, literal_type)`**: Sets the value for a signal that expects an input.

```python
# Example: Approving a signal
# remote.approve("my_approval_signal", "my_execution_name")

# Example: Setting an input signal
# remote.set_input("user_input_signal", "my_execution_name", "some_string_value", python_type=str)
```

#### Retrieving Execution Metrics

*   **`get_execution_metrics(id, depth)`**: Fetches detailed execution metrics, providing insights into performance and resource usage.

```python
# metrics = remote.get_execution_metrics(execution.id)
# print(metrics)
```

### Data Proxy and File System Operations

Flyte leverages a data proxy service for efficient transfer of large inputs and outputs, as well as for packaging code. The `flyte://` protocol, implemented by `FlyteFS`, integrates with `fsspec` to provide a unified file system interface for Flyte's data operations.

#### The `flyte://` Protocol

The `FlyteFS` class is an `fsspec` implementation that understands `flyte://` URIs. When you interact with `flyte://` paths, `FlyteFS` translates these requests into calls to the Flyte Admin's data proxy service, which in turn interacts with the underlying cloud storage (e.g., S3, GCS). The `FlytePathResolver` maintains a mapping between `flyte://` URIs and their actual remote storage locations.

#### Uploading Data

*   **`upload_file(to_upload, project, domain, filename_root)`**: Uploads a single local file to the Flyte data proxy. It computes an MD5 hash of the file and uses signed URLs for secure transfer.

    ```python
    import pathlib
    # Create a dummy file
    with open("my_local_file.txt", "w") as f:
        f.write("Hello Flyte!")

    md5_bytes, uploaded_url = remote.upload_file(pathlib.Path("my_local_file.txt"))
    print(f"Uploaded file to: {uploaded_url}")
    ```

*   **`fast_package(root, deref_symlinks, output, options)`**: Packages a local directory into an installable zip file. This is primarily used internally by `register_script` and `fast_register_workflow` for efficient code deployment.

#### Downloading Data

*   **`download(data, download_to, recursive)`**: Downloads data represented by `Literal` or `LiteralMap` objects to a specified local path. If `recursive` is `True` and the data contains file-like objects (e.g., `FlyteFile`, `FlyteDirectory`, `StructuredDataset`), it will download all nested files.

    ```python
    # Assuming 'execution' is a completed FlyteWorkflowExecution object
    # and it has an output named 'my_output_file' which is a FlyteFile
    if execution.is_successful():
        outputs = execution.outputs
        if "my_output_file" in outputs:
            remote.download(outputs["my_output_file"], "downloaded_output.txt")
            print("Output file downloaded to downloaded_output.txt")
    ```

**Important Considerations for Data Operations:**

*   **`data_upload_location`**: Ensure this is correctly configured in your `FlyteRemote` initialization, as it dictates where data is staged.
*   **Credentials**: For non-sandbox environments, your local environment must have appropriate cloud provider credentials configured to allow `FlyteRemote` to upload and download data to/from the specified `data_upload_location`.
*   **`fsspec` Integration**: The `flyte://` protocol allows `fsspec` to seamlessly interact with Flyte's data proxy, enabling operations like `fsspec.open("flyte://path/to/data", "rb")` (though direct reads are not yet supported for `flyte://` in `FlyteFS`).

### Advanced Operations

#### Launching Backfills

*   **`launch_backfill(project, domain, from_date, to_date, launchplan, launchplan_version, ...)`**: Automates the creation and execution of a backfill workflow for a given launch plan over a specified date range. This is useful for re-processing historical data.

    ```python
    from datetime import datetime, timedelta

    # Launch a backfill for a launch plan
    # backfill_exec = remote.launch_backfill(
    #     project="flytesnacks",
    #     domain="development",
    #     from_date=datetime.now() - timedelta(days=7),
    #     to_date=datetime.now(),
    #     launchplan="my_daily_report_lp",
    #     execute=True,
    #     parallel=False, # Run backfill executions sequentially
    # )
    # print(f"Backfill execution launched: {backfill_exec.id.name}")
    ```

#### Activating Launch Plans

*   **`activate_launchplan(ident)`**: Sets a specific version of a launch plan as active, deactivating any previously active versions for that named entity.

#### Generating Console URLs

*   **`generate_console_url(entity)`**: Creates a direct URL to the Flyte UI (console) for a given entity (task, workflow, launch plan) or execution, allowing for easy visual inspection.

    ```python
    console_url = remote.generate_console_url(execution)
    print(f"View execution in console: {console_url}")
    ```

### Limitations and Best Practices

*   **Interactive Mode (Jupyter/IPython)**: When `interactive_mode_enabled` is `True`, `FlyteRemote` attempts to pickle your Python entities and upload them for execution. Be mindful of the size of objects being pickled, as very large objects can lead to performance issues or exceed limits (e.g., 150MB for pickled tasks).
*   **Error Handling**: Be prepared to handle exceptions like `RegistrationSkipped` (when an entity is not meant for registration) or `FlyteEntityNotExistException` (when fetching a non-existent entity).
*   **Security**: For production environments, always use secure connections (i.e., `insecure=False` and configure `ca_cert_file_path` if needed) and manage your cloud credentials securely.
*   **Version Management**: Leverage Flyte's versioning capabilities. While `FlyteRemote` can auto-generate versions, explicitly providing meaningful versions for your entities is a best practice for reproducibility and deployment management.
<!--
key: summary_data_management_and_proxy_operations_73a9b1de-ec6a-44d5-b794-b8c89028f7ed
type: summary_end

-->
<!--
code_unit: flytekit.remote.remote.FlyteRemote.upload_file
code_unit_type: class
help_text: ''
key: example_e8ccaf02-8ee4-4236-82b9-f420f33e6628
type: example

-->
<!--
code_unit: flytekit.remote.remote.FlyteRemote.download
code_unit_type: class
help_text: ''
key: example_34b0b4e3-c9b7-4f10-af11-0b02af89ab72
type: example

-->
<!--
code_unit: flytekit.remote.remote_fs.FlyteFS
code_unit_type: class
help_text: ''
key: example_d198745e-9830-4664-96b3-b808edbd4d50
type: example

-->
<!--
code_unit: flytekit.remote.remote_fs.HttpFileWriter
code_unit_type: class
help_text: ''
key: example_067ee469-75e6-4667-9125-caf3f3e59861
type: example

-->