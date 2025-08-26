
<!--
help_text: ''
key: summary_client_setup_and_configuration_f71753a1-e469-4be7-9b0f-fbf55836de82
modules:
- flytekit.remote.remote
- flytekit.clients.friendly
- flytekit.clients.raw
questions_to_answer: []
type: summary

-->
## Client Setup and Configuration

The `FlyteRemote` client provides the primary programmatic interface for interacting with a Flyte backend. It abstracts away the complexities of direct gRPC communication, offering a user-friendly way to manage, execute, and monitor Flyte entities such as tasks, workflows, and launch plans.

### Initializing the FlyteRemote Client

To begin interacting with a Flyte backend, instantiate the `FlyteRemote` client. This client requires a `Config` object, which specifies the Flyte Admin endpoint and other platform-specific settings.

You can initialize `FlyteRemote` in several ways:

*   **`FlyteRemote.for_endpoint(endpoint: str, insecure: bool = False, ...)`**: Connects to a specific Flyte Admin endpoint. Use `insecure=True` for deployments without SSL/TLS.
    ```python
    from flytekit.remote import FlyteRemote

    # Connect to a local sandbox environment
    remote = FlyteRemote.for_endpoint(
        endpoint="localhost:30080",
        insecure=True,
        default_project="flytesnacks",
        default_domain="development",
        data_upload_location="s3://my-s3-bucket/data"
    )
    ```

*   **`FlyteRemote.auto(config_file: typing.Union[str, ConfigFile] = None, ...)`**: Automatically discovers configuration from environment variables or a specified configuration file. This is the recommended approach for most production deployments.
    ```python
    from flytekit.remote import FlyteRemote

    # Automatically configure from environment variables or default config file
    remote = FlyteRemote.auto(
        default_project="my_project",
        default_domain="production"
    )
    ```

*   **`FlyteRemote.for_sandbox(default_project: typing.Optional[str] = None, ...)`**: Configured specifically for a local Flyte sandbox environment.
    ```python
    from flytekit.remote import FlyteRemote

    # Connect to a Flyte sandbox
    remote = FlyteRemote.for_sandbox(
        default_project="flytesnacks",
        default_domain="development"
    )
    ```

When initializing, you can specify `default_project` and `default_domain` to avoid repeatedly providing them for operations. The `data_upload_location` parameter defines the default storage path for inputs and outputs that are too large to be passed inline.

The `interactive_mode_enabled` flag (defaulting to `None` for auto-detection) controls whether the `FlyteRemote` client attempts to pickle and upload local Python entities (tasks, workflows) for execution. While convenient for interactive environments like Jupyter notebooks, it's generally recommended to explicitly register entities in production workflows.

### Interacting with Flyte Entities

The `FlyteRemote` client provides comprehensive methods for managing Flyte entities on the backend.

#### Fetching Entities

You can retrieve existing tasks, workflows, launch plans, and executions from the Flyte backend.

*   **Tasks**: Use `fetch_task(project, domain, name, version)` to get a specific task. If `version` is `None`, the latest version is retrieved. `fetch_task_lazy` returns a `LazyEntity` that fetches the task only when accessed.
    ```python
    my_task = remote.fetch_task(project="flytesnacks", domain="development", name="my_task_name", version="v1")
    ```

*   **Workflows**: Use `fetch_workflow(project, domain, name, version)` to retrieve a workflow definition. `fetch_workflow_lazy` provides lazy loading.
    ```python
    my_workflow = remote.fetch_workflow(project="flytesnacks", domain="development", name="my_workflow_name") # Fetches latest
    ```

*   **Launch Plans**: Use `fetch_launch_plan(project, domain, name, version)` to get a launch plan. `fetch_active_launchplan` retrieves the currently active version.
    ```python
    my_lp = remote.fetch_launch_plan(project="flytesnacks", domain="development", name="my_launch_plan_name", version="v1")
    ```

*   **Executions**: Use `fetch_execution(project, domain, name)` to retrieve a specific workflow execution.
    ```python
    my_execution = remote.fetch_execution(project="flytesnacks", domain="development", name="my_execution_id")
    ```

#### Registering Entities

Before executing local Python tasks or workflows on a Flyte backend, they must be registered. Registration involves serializing the Python object into a Flyte-understandable format and uploading it to the Flyte Admin service.

*   **`register_task(entity: PythonTask, serialization_settings: Optional[SerializationSettings] = None, version: Optional[str] = None)`**: Registers a Python task.
*   **`register_workflow(entity: WorkflowBase, serialization_settings: Optional[SerializationSettings] = None, version: Optional[str] = None, default_launch_plan: Optional[bool] = True, options: Optional[Options] = None)`**: Registers a Python workflow. By default, it also creates a default launch plan for the workflow.
*   **`register_launch_plan(entity: LaunchPlan, version: Optional[str] = None, project: Optional[str] = None, domain: Optional[str] = None, options: Optional[Options] = None, serialization_settings: Optional[SerializationSettings] = None)`**: Registers a launch plan. If the underlying workflow is not registered, it will be registered as well.

When `version` is not explicitly provided, especially in interactive mode, the `FlyteRemote` client automatically generates a version based on the hash of the entity's content and serialization settings. This ensures version uniqueness and reproducibility.

For advanced registration scenarios, especially for larger codebases, consider `register_script`.

*   **`register_script(entity: typing.Union[WorkflowBase, PythonTask, LaunchPlan], ...)`**: This method enables "script mode" registration, where the entire Python project or relevant modules are packaged into a zip file and uploaded. This is particularly useful for workflows with many dependencies or complex module structures.
    ```python
    from flytekit import task, workflow
    from flytekit.remote import FlyteRemote

    @task
    def my_simple_task(x: int) -> int:
        return x + 1

    @workflow
    def my_simple_wf(x: int) -> int:
        return my_simple_task(x=x)

    remote = FlyteRemote.for_sandbox(default_project="flytesnacks", default_domain="development")

    # Register the workflow using script mode
    registered_wf = remote.register_script(
        my_simple_wf,
        source_path=".", # Path to your project root
        module_name="your_module_name" # e.g., "my_project.workflows"
    )
    print(f"Registered workflow: {registered_wf.id.name} version {registered_wf.id.version}")
    ```
    The `fast_package_options` parameter in `register_script` allows fine-grained control over which files are included in the uploaded package, enabling efficient registration for large projects.

A `RegistrationSkipped` exception may be raised if you attempt to register an entity that is not designed for remote registration (e.g., a `RemoteEntity` that is already on the backend).

#### Executing Entities

The `execute` method is a versatile entry point for running tasks, workflows, or launch plans on the Flyte backend. It intelligently handles both locally defined Python entities and already registered remote entities.

*   **`execute(entity: typing.Union[FlyteTask, FlyteLaunchPlan, FlyteWorkflow, PythonTask, WorkflowBase, LaunchPlan, ReferenceEntity], inputs: typing.Dict[str, typing.Any], ...)`**:
    ```python
    from flytekit.remote import FlyteRemote
    from flytekit import task, workflow

    @task
    def greet(name: str) -> str:
        return f"Hello, {name}!"

    @workflow
    def greeting_workflow(name: str) -> str:
        return greet(name=name)

    remote = FlyteRemote.for_sandbox(default_project="flytesnacks", default_domain="development")

    # Execute a local workflow (it will be registered if not already present)
    execution = remote.execute(greeting_workflow, inputs={"name": "Flyte User"}, wait=True)

    # Fetch a remote launch plan and execute it
    remote_lp = remote.fetch_launch_plan(project="flytesnacks", domain="development", name="greeting_workflow")
    execution_from_remote = remote.execute(remote_lp, inputs={"name": "Remote User"}, wait=True)

    print(f"Execution status: {execution.current_state}")
    print(f"Execution outputs: {execution.outputs}")
    ```
    Key parameters for `execute`:
    *   `inputs`: A dictionary mapping input names to Python native values. The client automatically converts these to Flyte Literals.
    *   `project`, `domain`: Overrides for the default project/domain for the execution.
    *   `name`, `version`: For local entities, these can override the default name/version used for registration.
    *   `execution_name`, `execution_name_prefix`: Control the name of the resulting execution on the Flyte backend.
    *   `wait`: If `True`, the method blocks until the execution completes and returns the final `FlyteWorkflowExecution` object.
    *   `overwrite_cache`: Forces re-execution even if cached results are available.
    *   `interruptible`: Overrides the entity's default interruptible setting.
    *   `envs`, `tags`, `cluster_pool`, `execution_cluster_label`: Provide additional execution configuration.
    *   `serialization_settings`: Can be provided if the entity needs to be registered before execution.

### Managing Executions

Once an execution is launched, the `FlyteRemote` client offers capabilities to monitor and interact with it.

*   **`wait(execution: FlyteWorkflowExecution, timeout: Optional[Union[timedelta, int]] = None, poll_interval: Optional[Union[timedelta, int]] = None, sync_nodes: bool = True)`**: Blocks until the specified `FlyteWorkflowExecution` completes, or a timeout is reached.
*   **`sync_execution(execution: FlyteWorkflowExecution, sync_nodes: bool = False)`**: Updates the local `FlyteWorkflowExecution` object with the latest state from the Flyte backend, including inputs, outputs, and optionally, underlying node executions.
*   **`get_execution_metrics(id: WorkflowExecutionIdentifier, depth: int = 10)`**: Retrieves detailed metrics and span information for an execution, useful for performance analysis.
*   **`terminate(execution: FlyteWorkflowExecution, cause: str)`**: Stops a running workflow execution.
*   **Signals**:
    *   `list_signals(execution_name, project, domain)`: Lists signals for a given execution.
    *   `approve(signal_id, execution_name, project, domain)`: Approves a signal, typically for human-in-the-loop workflows.
    *   `reject(signal_id, execution_name, project, domain)`: Rejects a signal.
    *   `set_input(signal_id, execution_name, value, ...)`: Sets the input value for a signal.

### Data Management

The `FlyteRemote` client facilitates data transfer to and from the Flyte backend's configured data store.

*   **`upload_file(to_upload: pathlib.Path, project: Optional[str] = None, domain: Optional[str] = None, filename_root: Optional[str] = None)`**: Uploads a local file to the Flyte data store, returning its MD5 hash and the native URL. This is used internally for fast registration and can be used for direct data uploads.
*   **`download(data: typing.Union[LiteralsResolver, Literal, LiteralMap], download_to: str, recursive: bool = True)`**: Downloads data (e.g., outputs of an execution) from the Flyte data store to a local path. If `recursive` is `True` and the data is a `LiteralsResolver` or `LiteralMap`, all file-like objects within it will be downloaded.

### Project and Domain Management

The client provides methods to list available projects and domains on the Flyte backend.

*   **`list_projects(limit: Optional[int] = 100, filters: Optional[typing.List[filter_models.Filter]] = None, sort_by: Optional[admin_common_models.Sort] = None)`**: Retrieves a paginated list of registered projects.
*   **`get_domains()`**: Retrieves a list of all registered domains.

### Advanced Features

*   **`generate_console_url(entity: typing.Union[FlyteWorkflowExecution, ...])`**: Generates a direct link to the Flyte Console UI for a given entity (execution, task, workflow, or launch plan). This is useful for quickly navigating to the UI for detailed inspection.
*   **`launch_backfill(project, domain, from_date, to_date, launchplan, ...)`**: Creates and launches a backfill workflow for a specified launch plan over a date range. This is a powerful utility for re-running historical data. It supports dry runs, registration-only, and full execution modes, along with parallel execution and failure policies.

### Underlying Client

The `FlyteRemote` client internally uses the `SynchronousFlyteClient` (which in turn uses `RawSynchronousFlyteClient`) for direct gRPC communication with the Flyte Admin service. While `FlyteRemote` is recommended for most use cases due to its higher-level abstractions, the `SynchronousFlyteClient` can be accessed via `remote.client` for more granular control over gRPC calls, if needed.

```python
# Accessing the lower-level client
admin_client = remote.client
# Example: List tasks directly using the SynchronousFlyteClient
task_ids, _ = admin_client.list_task_ids_paginated(project="flytesnacks", domain="development")
```
The `SynchronousFlyteClient` provides methods like `create_task`, `get_workflow`, `list_launch_plans_paginated`, and `create_execution`, mirroring the Flyte Admin service API.
<!--
key: summary_client_setup_and_configuration_f71753a1-e469-4be7-9b0f-fbf55836de82
type: summary_end

-->
<!--
code_unit: flytekit.remote.remote.FlyteRemote.for_endpoint
code_unit_type: class
help_text: ''
key: example_f03faa11-e3ad-4750-9661-beedb13c6168
type: example

-->
<!--
code_unit: flytekit.remote.remote.FlyteRemote.auto
code_unit_type: class
help_text: ''
key: example_8eb242df-dbf3-4502-9d76-050e05df8909
type: example

-->
<!--
code_unit: flytekit.remote.remote.FlyteRemote.for_sandbox
code_unit_type: class
help_text: ''
key: example_984c609d-c634-4fdc-99d2-9415d9d0c537
type: example

-->
<!--
code_unit: flytekit.clients.friendly.SynchronousFlyteClient
code_unit_type: class
help_text: ''
key: example_d64aa431-b4ee-429a-902b-c411328401f5
type: example

-->