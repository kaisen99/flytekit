
<!--
help_text: ''
key: summary_managing_flyte_entities_remotely_85748f78-d932-4b2d-b0eb-e5f19e203adb
modules:
- flytekit.remote.remote
- flytekit.remote.entities
- flytekit.remote.lazy_entity
- flytekit.interfaces.cli_identifiers
questions_to_answer: []
type: summary

-->
The `FlyteRemote` class provides the primary interface for programmatically interacting with a remote Flyte backend. This includes fetching, registering, executing, and managing various Flyte entities such as tasks, workflows, launch plans, and executions.

### Initializing the Remote Client

To begin interacting with a Flyte backend, instantiate the `FlyteRemote` class. This class requires configuration details to connect to the Flyte Admin server.

You can initialize `FlyteRemote` in several ways:

*   **Direct Configuration:** Provide a `Config` object, which specifies the Flyte Admin endpoint and other platform settings.
    ```python
    from flytekit.configuration import Config
    from flytekit.remote import FlyteRemote

    # Example: Connect to a specific endpoint
    remote = FlyteRemote(
        config=Config.for_endpoint("grpc://flyte.mycluster.com:80"),
        default_project="flytesnacks",
        default_domain="development",
        data_upload_location="s3://my-data-bucket/flyte-data/",
    )
    ```
*   **Automatic Configuration (`auto`):** Automatically detects configuration from environment variables or a `config.yaml` file.
    ```python
    from flytekit.remote import FlyteRemote

    remote = FlyteRemote.auto(
        default_project="flytesnacks",
        default_domain="development",
    )
    ```
*   **Sandbox Configuration (`for_sandbox`):** Pre-configured for a local Flyte sandbox environment.
    ```python
    from flytekit.remote import FlyteRemote

    remote = FlyteRemote.for_sandbox(
        default_project="flytesnacks",
        default_domain="development",
    )
    ```
*   **Endpoint-Specific Configuration (`for_endpoint`):** A convenience method to configure for a specific endpoint.
    ```python
    from flytekit.remote import FlyteRemote

    remote = FlyteRemote.for_endpoint(
        endpoint="grpc://flyte.mycluster.com:80",
        insecure=True, # Use insecure connection if not using SSL/TLS
        default_project="flytesnacks",
        default_domain="development",
    )
    ```

The `default_project` and `default_domain` parameters set the default context for operations if not explicitly provided in method calls. The `data_upload_location` specifies where non-literal inputs/outputs (e.g., files, directories) are staged.

### Fetching Existing Flyte Entities

The `FlyteRemote` object allows you to retrieve registered Flyte entities from the backend.

*   **Fetching Tasks, Workflows, and Launch Plans:**
    Use `fetch_task`, `fetch_workflow`, and `fetch_launch_plan` to retrieve specific versions of entities. If `version` is omitted, the latest version is fetched.
    ```python
    # Fetch a specific task
    my_task = remote.fetch_task(project="flytesnacks", domain="development", name="my_task_name", version="v1.0.0")

    # Fetch the latest version of a workflow
    my_workflow = remote.fetch_workflow(project="flytesnacks", domain="development", name="my_workflow_name")

    # Fetch a launch plan
    my_launch_plan = remote.fetch_launch_plan(project="flytesnacks", domain="development", name="my_lp_name", version="v1")
    ```
    The `fetch_task_lazy` and `fetch_workflow_lazy` methods return a `LazyEntity` object. This object defers the actual fetching of the entity until it is accessed or called, which can be useful for performance or when dealing with many entities.

*   **Fetching Executions:**
    Retrieve a specific workflow execution using `fetch_execution` by its project, domain, and name.
    ```python
    from flytekit.remote import FlyteWorkflowExecution

    execution: FlyteWorkflowExecution = remote.fetch_execution(
        project="flytesnacks", domain="development", name="my_execution_id"
    )
    ```

*   **Listing Entities:**
    The `FlyteRemote` class provides methods to list various entities:
    *   `list_tasks_by_version`: Lists tasks filtered by version.
    *   `recent_executions`: Retrieves recent workflow executions.
    *   `list_signals`: Lists signals for a given execution.
    *   `list_projects`: Lists all registered projects.
    *   `get_domains`: Lists all registered domains.

*   **Retrieving Data from Flyte Tiny URLs:**
    The `get` method can resolve Flyte tiny URLs, returning outputs (as `LiteralsResolver` or `Literal`), HTML for decks, or raw bytes.
    ```python
    # Assuming 'flyte_uri' is a URL obtained from a Flyte execution output
    data = remote.get(flyte_uri="flyte://.../outputs/my_output")
    if isinstance(data, LiteralsResolver):
        print(data["output_key"])
    ```

### Registering Local Flyte Entities

Before executing a local Python task or workflow on a remote Flyte cluster, it must be registered. Registration involves serializing the Python object into a Flyte-understandable format and uploading it to the Flyte Admin server.

*   **Registering Tasks, Workflows, and Launch Plans:**
    Use `register_task`, `register_workflow`, and `register_launch_plan` to upload your local Python entities.
    ```python
    from flytekit import task, workflow, LaunchPlan
    from flytekit.remote import FlyteRemote

    @task
    def my_local_task(x: int) -> int:
        return x + 1

    @workflow
    def my_local_workflow(x: int) -> int:
        return my_local_task(x=x)

    # Register a task
    remote_task = remote.register_task(my_local_task, version="v1")

    # Register a workflow
    remote_workflow = remote.register_workflow(my_local_workflow, version="v1")

    # Register a launch plan for the workflow
    my_lp = LaunchPlan.get_or_create(my_local_workflow)
    remote_lp = remote.register_launch_plan(my_lp, version="v1")
    ```
    If a `version` is not explicitly provided, `FlyteRemote` will automatically generate one based on a hash of the entity's content and serialization settings. This ensures that identical code is registered with the same version.

    The `RegistrationSkipped` exception is raised if you attempt to register an entity that is already marked as non-registrable (e.g., a `RemoteEntity` that was fetched from the backend).

*   **Script Mode Registration (Fast Serialization):**
    For Python tasks and workflows, `FlyteRemote` supports "script mode" registration, which packages your code into a zip file and uploads it. This is particularly useful for larger codebases or when you want to ensure the exact code environment is deployed.

    Use `register_script` for general script mode registration or `fast_register_workflow` as a specialized version for `PythonFunctionWorkflow` entities.
    ```python
    from flytekit import workflow
    from flytekit.remote import FlyteRemote
    from flytekit.configuration import ImageConfig
    import os

    # Assuming my_workflow is defined in a file within your project root
    # e.g., my_project/my_module/my_workflow.py
    # And the current working directory is my_project/

    @workflow
    def my_workflow(x: int) -> int:
        return x + 1

    # To register my_workflow in script mode:
    # You need to specify the source_path (root of your project)
    # and module_name (the Python module path to your workflow)
    remote_workflow_script_mode = remote.register_script(
        entity=my_workflow,
        project="flytesnacks",
        domain="development",
        version="script-v1",
        source_path=os.getcwd(), # Assuming current directory is project root
        module_name="my_module.my_workflow", # e.g., if my_workflow is in my_module/my_workflow.py
        image_config=ImageConfig.auto(), # Or specify a custom image
    )
    ```
    The `fast_package` method is an internal utility used by script mode registration to package the code. You can also use `upload_file` to upload arbitrary files to the Flyte data plane.

### Executing Flyte Entities

The `execute` method is the primary entry point for running any Flyte entity (tasks, workflows, launch plans, whether local, remote, or referenced) on the remote cluster.

```python
from flytekit.remote import FlyteRemote
from flytekit import task, workflow

# Initialize remote
remote = FlyteRemote.for_sandbox(default_project="flytesnacks", default_domain="development")

@task
def my_local_task(x: int) -> int:
    return x + 1

@workflow
def my_local_workflow(x: int) -> int:
    return my_local_task(x=x)

# 1. Execute a local task (will be registered if not exists)
execution_task = remote.execute(
    my_local_task,
    inputs={"x": 10},
    execution_name_prefix="my-task-exec",
    wait=True, # Wait for completion
)
print(f"Task execution outputs: {execution_task.outputs}")

# 2. Execute a local workflow (will register workflow and default launch plan if not exists)
execution_workflow = remote.execute(
    my_local_workflow,
    inputs={"x": 20},
    execution_name_prefix="my-wf-exec",
    wait=True,
)
print(f"Workflow execution outputs: {execution_workflow.outputs}")

# 3. Execute a fetched remote launch plan
remote_lp = remote.fetch_launch_plan(project="flytesnacks", domain="development", name="my_local_workflow", version="v1")
execution_lp = remote.execute(
    remote_lp,
    inputs={"x": 30},
    execution_name_prefix="my-remote-lp-exec",
    wait=True,
)
print(f"Remote Launch Plan execution outputs: {execution_lp.outputs}")
```

Key parameters for `execute`:

*   `entity`: The task, workflow, or launch plan object to execute.
*   `inputs`: A dictionary of input values. FlyteRemote automatically converts Python native types to Flyte Literals. You can provide `type_hints` for complex types if auto-detection fails.
*   `project`, `domain`: Override the default project/domain for the execution. If a local entity is being executed and not found in the specified project/domain, it will be registered there first.
*   `name`, `version`: For local entities, these can override the entity's default name/version during registration.
*   `execution_name`, `execution_name_prefix`: Control the name of the resulting execution. `execution_name_prefix` appends a unique suffix.
*   `wait`: If `True`, the method blocks until the execution completes.
*   `options`: An `Options` object to configure execution-level settings like notifications, labels, annotations, raw output data configuration, max parallelism, security context, etc.
*   `overwrite_cache`: If `True`, forces re-execution even if cached results are available.
*   `interruptible`: Overrides the entity's default interruptible flag.
*   `envs`, `tags`, `cluster_pool`, `execution_cluster_label`: Additional execution configuration.
*   `serialization_settings`: Provides fine-grained control over how local entities are serialized and registered before execution.

The `execute` method internally dispatches to specialized methods like `execute_remote_task_lp`, `execute_remote_wf`, `execute_reference_task`, `execute_local_task`, etc., based on the type of the `entity` provided.

### Managing Executions

`FlyteRemote` offers capabilities to monitor and control running executions.

*   **Waiting for Completion:**
    The `wait` method blocks until a given `FlyteWorkflowExecution` reaches a terminal state (success, failure, aborted).
    ```python
    from datetime import timedelta

    # Assuming 'execution' is a FlyteWorkflowExecution object
    completed_execution = remote.wait(execution, timeout=timedelta(minutes=5), poll_interval=timedelta(seconds=10))

    if completed_execution.is_successful:
        print("Execution succeeded!")
    else:
        print(f"Execution failed: {completed_execution.error}")
    ```

*   **Syncing Execution State:**
    The `sync_execution` method (also accessible via the `sync` alias) updates a local `FlyteWorkflowExecution` object with the latest state from the remote backend. This includes status, inputs, outputs, and optionally, the state of underlying node and task executions.
    ```python
    # Sync the execution to get the latest status and outputs
    updated_execution = remote.sync_execution(execution, sync_nodes=True)
    ```
    Setting `sync_nodes=True` recursively fetches information about all child node and task executions, providing a comprehensive view of the execution graph.

*   **Terminating Executions:**
    Use `terminate` to stop a running workflow execution.
    ```python
    remote.terminate(execution, cause="User requested termination")
    ```

*   **Interacting with Signals:**
    Flyte workflows can include `wait_for_input` nodes that pause execution until a signal is received. `FlyteRemote` allows you to interact with these signals:
    *   `list_signals`: Discover pending signals for an execution.
    *   `approve`: Approves a boolean signal.
    *   `reject`: Rejects a boolean signal.
    *   `set_input` / `set_signal`: Provides a value for a signal.
    ```python
    # Assuming 'my_signal_id' is the name of a signal in 'my_execution_id'
    remote.set_input(
        signal_id="my_signal_id",
        execution_name="my_execution_id",
        value="some_input_data",
        python_type=str,
    )
    ```

### Utility and Helper Methods

*   **Downloading Data:**
    The `download` method facilitates downloading outputs from `LiteralsResolver` or `LiteralMap` objects to a local path. It can recursively download all file-like objects (e.g., `FlyteFile`, `FlyteDir`, `StructuredDataset`).
    ```python
    # Assuming 'execution_outputs' is a LiteralsResolver from a completed execution
    remote.download(execution_outputs, download_to="/tmp/my_downloads", recursive=True)
    ```

*   **Generating Console URLs:**
    `generate_console_url` creates a direct link to the Flyte UI for a given entity or execution, simplifying navigation to the web console.
    ```python
    console_url = remote.generate_console_url(execution)
    print(f"View execution in console: {console_url}")
    ```

*   **Launching Backfills:**
    The `launch_backfill` method creates and executes a new workflow designed to backfill historical data for a given launch plan. It supports specifying date ranges, parallel execution, and failure policies.
    ```python
    from datetime import datetime, timedelta

    # Launch a backfill for a launch plan
    backfill_execution = remote.launch_backfill(
        project="flytesnacks",
        domain="development",
        from_date=datetime(2023, 1, 1),
        to_date=datetime(2023, 1, 31),
        launchplan="my_daily_lp",
        parallel=True, # Run daily executions in parallel
    )
    ```

*   **Activating Launch Plans:**
    `activate_launchplan` sets a specific version of a launch plan as active, deactivating any previously active versions.
    ```python
    from flytekit.models.core.identifier import Identifier, ResourceType

    lp_id = Identifier(ResourceType.LAUNCH_PLAN, "flytesnacks", "development", "my_lp_name", "v2")
    remote.activate_launchplan(lp_id)
    ```

*   **Retrieving Execution Metrics:**
    `get_execution_metrics` fetches detailed metrics and span information for a workflow execution, useful for performance analysis and debugging.
    ```python
    from flytekit.models.core.identifier import WorkflowExecutionIdentifier

    exec_id = WorkflowExecutionIdentifier("flytesnacks", "development", "my_execution_id")
    metrics = remote.get_execution_metrics(exec_id, depth=5)
    print(metrics.span_id)
    ```

### Important Considerations

*   **Interactive Mode:** The `interactive_mode_enabled` flag in `FlyteRemote`'s constructor (or auto-detected) influences how local entities are handled. In interactive environments (like Jupyter notebooks), `FlyteRemote` can automatically pickle and upload task/workflow definitions, enabling execution without explicit registration steps. This feature is currently in alpha.
*   **Versioning:** When registering entities, explicit versioning is recommended for production environments. If no version is provided, `FlyteRemote` generates a hash-based version, which ensures idempotency for identical code but might not be human-readable.
*   **Data Access:** `FlyteRemote` leverages the configured `FileAccessProvider` for handling data offloading (e.g., to S3, GCS). Ensure your environment has the necessary credentials configured for the specified `data_upload_location`.
*   **Error Handling:** Operations with `FlyteRemote` can raise `FlyteUserException` for user-related errors (e.g., missing arguments, entity not found) and `FlyteTimeout` for operations that exceed a specified time limit.
*   **`ResolvedIdentifiers`:** This internal helper class (`flytekit.remote.remote.ResolvedIdentifiers`) is used to standardize the resolution of project, domain, name, and version for various operations, ensuring consistency across method calls.
<!--
key: summary_managing_flyte_entities_remotely_85748f78-d932-4b2d-b0eb-e5f19e203adb
type: summary_end

-->
<!--
code_unit: flytekit.remote.remote.FlyteRemote.fetch_task
code_unit_type: class
help_text: ''
key: example_83316033-23ff-4ebc-8e43-920c87e51b6d
type: example

-->
<!--
code_unit: flytekit.remote.remote.FlyteRemote.register_workflow
code_unit_type: class
help_text: ''
key: example_723da54e-5cae-4de1-84d6-668b27b40bbb
type: example

-->
<!--
code_unit: flytekit.remote.remote.FlyteRemote.execute
code_unit_type: class
help_text: ''
key: example_a1307ac6-fe98-49f1-b440-c6850c1ebc1b
type: example

-->
<!--
code_unit: flytekit.remote.remote.FlyteRemote.fast_register_workflow
code_unit_type: class
help_text: ''
key: example_2d2fbbde-86ff-415f-a50f-ff5a22730594
type: example

-->
<!--
code_unit: flytekit.remote.remote.FlyteRemote.register_script
code_unit_type: class
help_text: ''
key: example_36b24235-85ff-48d8-8af8-f114f4f63d3f
type: example

-->
<!--
code_unit: flytekit.remote.entities.FlyteTask
code_unit_type: class
help_text: ''
key: example_74de9799-501a-40b4-a4ec-8552ffb6c3bd
type: example

-->
<!--
code_unit: flytekit.remote.entities.FlyteWorkflow
code_unit_type: class
help_text: ''
key: example_c555b1cf-fda4-442d-ac61-5b457e82e63e
type: example

-->
<!--
code_unit: flytekit.remote.entities.FlyteLaunchPlan
code_unit_type: class
help_text: ''
key: example_b588b17c-40df-417d-b65c-5251ac36a2b1
type: example

-->
<!--
code_unit: flytekit.remote.lazy_entity.LazyEntity
code_unit_type: class
help_text: ''
key: example_48367504-47ad-4840-856f-86376560c605
type: example

-->
<!--
code_unit: flytekit.interfaces.cli_identifiers.Identifier
code_unit_type: class
help_text: ''
key: example_53e85914-d3f9-4af3-94d5-eaea8455787f
type: example

-->