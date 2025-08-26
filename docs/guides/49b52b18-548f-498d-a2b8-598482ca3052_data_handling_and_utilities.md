
<!--
help_text: ''
key: summary_data_handling_and_utilities_f4fb28b6-80f9-4340-8670-433552fe3566
modules:
- flytekit.remote.remote
- flytekit.remote.remote_fs
- flytekit.remote.interface
questions_to_answer: []
type: summary

-->
The `FlyteRemote` object provides a comprehensive interface for interacting with a Flyte backend, enabling developers to manage, execute, and monitor Flyte tasks, workflows, and launch plans. It centralizes data handling, entity registration, and execution management, abstracting away the complexities of direct API interactions and cloud storage.

### Connecting to a Flyte Backend

To begin interacting with a Flyte backend, initialize a `FlyteRemote` object. This object serves as the primary entry point for all subsequent operations.

```python
from flytekit.remote.remote import FlyteRemote, Config

# Connect to a specific endpoint
remote = FlyteRemote.for_endpoint(
    endpoint="your-flyte-admin-endpoint:80",
    insecure=True, # Use False for HTTPS
    default_project="flytesnacks",
    default_domain="development",
    data_upload_location="s3://your-bucket/flyte-data/"
)

# Auto-discover configuration (e.g., from ~/.flyte/config.yaml)
remote_auto = FlyteRemote.auto()

# Connect to a local Flyte sandbox environment
remote_sandbox = FlyteRemote.for_sandbox()
```

The `data_upload_location` parameter is crucial for specifying where non-literal inputs and outputs (e.g., files, directories, structured datasets) will be staged. For non-sandbox environments, ensure this is configured to a suitable cloud storage bucket.

### Managing Data with FlyteRemote

The remote interface provides robust capabilities for handling data, from automatic type conversion to seamless file system integration.

#### Input and Output Handling

When executing Flyte entities, Python native data types are automatically converted to Flyte Literals for inputs and resolved back to Python types for outputs. This conversion is managed by the underlying type engine.

The `_assign_inputs_and_outputs` helper method within `FlyteRemote` demonstrates how input and output `LiteralMap` objects are retrieved from execution data and then resolved into accessible Python objects using `LiteralsResolver`. For large inputs/outputs, data is offloaded to the `data_upload_location` and accessed via signed URLs.

#### File and Directory Management

The `flyte://` protocol provides a unified way to interact with remote data storage. The `FlyteFS` class, an `fsspec` implementation, enables `flyte://` URIs to behave like standard file paths, abstracting the underlying cloud storage.

-   **Uploading Files**: The `upload_file` method facilitates uploading local files to the configured `data_upload_location`. It handles hashing and obtaining signed URLs from the Flyte Admin service.

    ```python
    import pathlib
    # Assuming 'remote' is an initialized FlyteRemote object
    local_file = pathlib.Path("my_data.csv")
    # Create a dummy file for demonstration
    local_file.write_text("col1,col2\n1,a\n2,b")

    md5_bytes, remote_url = remote.upload_file(local_file)
    print(f"Uploaded to: {remote_url}")
    ```

    The `FlyteFS._put` method, used internally by `fsspec` operations (e.g., `fs.put`), leverages `upload_file` to handle recursive directory uploads, ensuring all files are correctly staged.

-   **Downloading Data**: The `download` method allows retrieving data from remote storage to a local path. It supports `LiteralsResolver`, `Literal`, or `LiteralMap` objects. When `recursive` is `True`, it downloads all file-like objects within a `LiteralMap`.

    ```python
    # Assuming 'execution' is a completed FlyteWorkflowExecution object
    # and 'outputs' is a LiteralsResolver from that execution
    # For example: outputs = execution.outputs
    # remote.download(outputs, "/tmp/downloaded_data", recursive=True)
    ```

    The `FlytePathResolver` class maintains a thread-safe mapping between `flyte://` URIs and their actual remote storage paths, ensuring consistent resolution during data operations.

#### Fast Serialization and Script Mode

For efficient code deployment, especially in interactive environments, the remote interface supports "fast serialization" or "script mode." This involves packaging the Python code and its dependencies into a zip archive and uploading it to the `data_upload_location`.

-   `fast_package`: This method creates an installable zip archive of a given root path, including options for dereferencing symlinks and customizing copy behavior (`FastPackageOptions`). It returns the MD5 hash of the package and its uploaded URL.
-   `register_script`: This method registers a task, workflow, or launch plan by packaging its source code. It automatically determines the version based on the code's hash and serialization settings, ensuring idempotency and efficient updates.

    ```python
    from flytekit import task, workflow
    from flytekit.remote.remote import FlyteRemote

    @task
    def my_remote_task(x: int) -> int:
        return x + 1

    @workflow
    def my_remote_workflow(x: int) -> int:
        return my_remote_task(x=x)

    # Assuming 'remote' is an initialized FlyteRemote object
    # Registered task using script mode
    # registered_task = remote.register_script(my_remote_task, project="flytesnacks", domain="development")
    # print(f"Registered task: {registered_task.id.name}:{registered_task.id.version}")

    # Registered workflow using script mode
    # registered_workflow = remote.register_script(my_remote_workflow, project="flytesnacks", domain="development")
    # print(f"Registered workflow: {registered_workflow.id.name}:{registered_workflow.id.version}")
    ```

    The `_pickle_and_upload_entity` method, used internally, handles the pickling of Python entities and their upload, especially when interactive mode is enabled.

### Interacting with Flyte Entities

The `FlyteRemote` object provides comprehensive methods for managing Flyte entities (tasks, workflows, launch plans, and executions) on the remote backend.

#### Fetching Entities

Entities can be retrieved from the Flyte Admin service using their project, domain, name, and optionally, version. If no version is specified, the latest version is typically fetched.

-   `fetch_task(project, domain, name, version)`: Retrieves a `FlyteTask` object.
-   `fetch_workflow(project, domain, name, version)`: Retrieves a `FlyteWorkflow` object.
-   `fetch_launch_plan(project, domain, name, version)`: Retrieves a `FlyteLaunchPlan` object.
-   `fetch_execution(project, domain, name)`: Retrieves a `FlyteWorkflowExecution` object.
-   `fetch_task_lazy`, `fetch_workflow_lazy`: Return `LazyEntity` objects that defer fetching until the entity is accessed, useful for large graphs.

#### Registering Entities

Local Python tasks, workflows, and launch plans can be registered with the Flyte backend, making them available for execution. Registration involves serializing the entity into a format understood by Flyte Admin.

-   `register_task(entity, serialization_settings, version)`: Registers a `PythonTask`.
-   `register_workflow(entity, serialization_settings, version, default_launch_plan, options)`: Registers a `WorkflowBase` (e.g., a `@workflow` decorated function). It can optionally create a default launch plan.
-   `register_launch_plan(entity, version, project, domain, options, serialization_settings)`: Registers a `LaunchPlan`. If the underlying workflow is not registered, it will be registered as well.

The `_resolve_version` method automatically generates a version string based on the entity's hash and serialization settings when not explicitly provided, ensuring unique and reproducible registrations. The `raw_register` method handles the low-level registration of serialized control plane entities.

#### Executing Entities

The `execute` method is a versatile entry point for running any Flyte entity, whether it's a locally defined Python object or a fetched remote entity. It handles automatic registration if the local entity is not found on the backend.

```python
from flytekit import task, workflow
from flytekit.remote.remote import FlyteRemote

@task
def add_one(x: int) -> int:
    return x + 1

@workflow
def my_wf(x: int) -> int:
    return add_one(x=x)

remote = FlyteRemote.for_sandbox()

# Execute a local workflow
execution = remote.execute(my_wf, inputs={"x": 5}, wait=True)
print(f"Execution status: {execution.current_state}")
print(f"Execution outputs: {execution.outputs.get('o0')}")

# Execute a remote task (assuming it's already registered or will be registered by execute)
# remote_task_execution = remote.execute(
#     remote.fetch_task(project="flytesnacks", domain="development", name="my_remote_task"),
#     inputs={"x": 10},
#     wait=True
# )
```

The `execute` method supports various options for customizing the execution, including:
-   `project`, `domain`, `name`, `version`: For specifying the target entity.
-   `execution_name`, `execution_name_prefix`: For controlling the execution identifier.
-   `inputs`: A dictionary of Python native values, automatically converted to Flyte Literals.
-   `wait`: If `True`, the method blocks until the execution completes.
-   `overwrite_cache`, `interruptible`, `envs`, `tags`, `cluster_pool`, `execution_cluster_label`: For advanced execution configuration.

Specialized execution methods like `execute_local_task`, `execute_remote_wf`, `execute_reference_launch_plan` provide tailored interfaces for different entity types.

#### Monitoring and Controlling Executions

-   `wait(execution, timeout, poll_interval, sync_nodes)`: Blocks until a given `FlyteWorkflowExecution` completes, with configurable timeout and polling.
-   `sync_execution(execution, sync_nodes)`: Updates the local `FlyteWorkflowExecution` object with the latest state from the remote backend, including inputs, outputs, and (optionally) child node and task executions.
-   `terminate(execution, cause)`: Stops a running workflow execution.
-   `list_signals(execution_name, project, domain, limit, filters)`: Retrieves signals for a given execution, useful for interactive workflows.
-   `approve(signal_id, execution_name, project, domain)`: Approves a boolean signal, setting its value to `True`.
-   `reject(signal_id, execution_name, project, domain)`: Rejects a boolean signal, setting its value to `False`.
-   `set_input(signal_id, execution_name, value, project, domain, python_type, literal_type)`: Sets the value of a signal, supporting various Python types and Flyte Literals.

### Advanced Utilities

#### Backfilling Workflows

The `launch_backfill` method simplifies the process of creating and executing a workflow that runs a given launch plan for a specified date range. This is particularly useful for re-processing historical data.

```python
from datetime import datetime, timedelta
# Assuming 'remote' is an initialized FlyteRemote object

# backfill_execution = remote.launch_backfill(
#     project="flytesnacks",
#     domain="development",
#     from_date=datetime.now() - timedelta(days=7),
#     to_date=datetime.now(),
#     launchplan="my_daily_lp", # Name of an existing launch plan
#     parallel=True, # Run backfill instances in parallel
#     execute=True,
#     overwrite_cache=True,
# )
# print(f"Backfill execution launched: {backfill_execution.id.name}")
```

This utility can generate sequential or parallel backfill workflows and offers options for dry runs and controlling failure policies.

#### Console Integration

The `generate_console_url` method provides a convenient way to obtain a direct link to the Flyte UI for any given entity (execution, task, workflow, or launch plan).

```python
# Assuming 'execution' is a FlyteWorkflowExecution object
# console_url = remote.generate_console_url(execution)
# print(f"View execution in console: {console_url}")
```

#### Project and Domain Management

The remote interface allows listing available projects and domains registered in the Flyte backend.

-   `list_projects(limit, filters, sort_by)`: Retrieves a list of registered projects.
-   `get_domains()`: Retrieves a list of registered domains.

### Considerations and Best Practices

-   **Default Project and Domain**: Configure `default_project` and `default_domain` during `FlyteRemote` initialization to reduce verbosity in subsequent calls.
-   **Data Upload Location**: Always ensure `data_upload_location` is correctly set for your environment, especially for production deployments, to avoid data staging issues.
-   **Versioning**: Leverage automatic versioning for local entities by omitting the `version` argument during registration, or explicitly provide a version for precise control.
-   **Interactive Mode**: Be aware that interactive mode (e.g., in Jupyter notebooks) enables pickling of entities for registration. While convenient, it can lead to larger package sizes.
-   **Error Handling**: Implement robust error handling for operations that interact with the remote backend, such as `FlyteEntityNotExistException` and `RegistrationSkipped`.
-   **Performance**: For large-scale data transfers, ensure your `data_upload_location` is optimized for performance (e.g., using a high-throughput S3 bucket). The `upload_file` method includes progress indicators for large uploads.
-   **Security**: When connecting to a Flyte backend, carefully manage `insecure` flag and `ca_cert_file_path` for secure communication. Ensure appropriate IAM roles or credentials are configured for data access.
<!--
key: summary_data_handling_and_utilities_f4fb28b6-80f9-4340-8670-433552fe3566
type: summary_end

-->