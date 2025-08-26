
<!--
help_text: ''
key: summary_data_management_897e22d7-0984-4f49-a2fb-50d0090f4d4d
modules:
- flytekit.clients.friendly.SynchronousFlyteClient
- flytekit.remote.remote.FlyteRemote
- flytekit.remote.remote_fs.FlyteFS
- flytekit.remote.remote_fs.HttpFileWriter
- flytekit.remote.remote_fs.FlytePathResolver
questions_to_answer: []
type: summary

-->
## Data Management

Data management within the platform encompasses the lifecycle of defining, registering, retrieving, executing, and observing data-driven entities and their associated runtime data. This includes the blueprints of your computations (tasks, workflows, launch plans) and the actual data produced and consumed during their execution.

### Entity Definition and Metadata Management

The platform provides robust capabilities for managing the metadata of your computational entities. These entities, such as tasks, workflows, and launch plans, are versioned and stored in the Admin database, ensuring reproducibility and traceability.

#### Registering Entities

Entities are registered with the Admin service to make them available for execution and discovery. The `SynchronousFlyteClient` offers low-level methods like `create_task`, `create_workflow`, and `create_launch_plan` for direct registration.

For a higher-level, more convenient approach, the `FlyteRemote` object simplifies the registration process, automatically handling serialization and dependencies.

To register a local Python task:

```python
from flytekit import task
from flytekit.remote import FlyteRemote, Config

remote = FlyteRemote(config=Config.for_sandbox(), default_project="flytesnacks", default_domain="development")

@task
def my_task(a: int, b: int) -> int:
    return a + b

registered_task = remote.register_task(my_task, version="1.0.0")
print(f"Registered task: {registered_task.id}")
```

Similarly, workflows and launch plans can be registered:

```python
from flytekit import workflow, LaunchPlan

@workflow
def my_workflow(x: int, y: int) -> int:
    return my_task(a=x, b=y)

registered_workflow = remote.register_workflow(my_workflow, version="1.0.0")
print(f"Registered workflow: {registered_workflow.id}")

# A default launch plan is often created automatically with workflow registration.
# To explicitly register a launch plan:
lp = LaunchPlan.get_default_launch_plan(remote.context, my_workflow)
registered_lp = remote.register_launch_plan(lp, version="1.0.0")
print(f"Registered launch plan: {registered_lp.id}")
```

When registering, if an identical version of an entity already exists, the operation will succeed without overwriting, ensuring data integrity.

#### Fast Registration (Script Mode)

For Python-based entities, the platform supports "fast registration" or "script mode," which packages your code and uploads it to remote storage. This allows the Admin service to execute your code directly without requiring a pre-built Docker image. This is particularly useful for rapid iteration and development.

The `FlyteRemote.register_script` method facilitates this:

```python
from flytekit import task, workflow
from flytekit.remote import FlyteRemote, Config
import os

remote = FlyteRemote(config=Config.for_sandbox(), default_project="flytesnacks", default_domain="development")

# Assume my_module.py contains:
# @task
# def my_fast_task(a: int) -> int:
#     return a * 2
#
# @workflow
# def my_fast_workflow(x: int) -> int:
#     return my_fast_task(a=x)

# To register my_fast_workflow from my_module.py:
# You would typically import the workflow object directly from your module
# from my_module import my_fast_workflow
# registered_fast_wf = remote.register_script(
#     my_fast_workflow,
#     source_path=os.getcwd(), # Root directory of your project
#     module_name="my_module",
#     version="fast_reg_1.0.0"
# )
# print(f"Fast registered workflow: {registered_fast_wf.id}")
```

Fast registration automatically computes a version hash based on the code content and serialization settings, ensuring that changes to your code result in a new version.

#### Retrieving Entities

Registered entities can be retrieved by their unique identifiers (project, domain, name, version).

The `SynchronousFlyteClient` provides granular `get_task`, `get_workflow`, and `get_launch_plan` methods. For listing entities, paginated methods like `list_tasks_paginated`, `list_workflows_paginated`, and `list_launch_plans_paginated` are available, allowing efficient retrieval of large numbers of entities.

The `FlyteRemote` object offers simplified `fetch_task`, `fetch_workflow`, and `fetch_launch_plan` methods:

```python
# Fetch a previously registered task
fetched_task = remote.fetch_task(name="my_task", version="1.0.0")
print(f"Fetched task: {fetched_task.id}")

# Fetch the latest active launch plan for a given name
active_lp = remote.fetch_active_launchplan(name="my_workflow")
if active_lp:
    print(f"Fetched active launch plan: {active_lp.id}")
```

For performance-sensitive applications, `fetch_task_lazy` and `fetch_workflow_lazy` return `LazyEntity` objects, which defer the actual fetching until the entity's properties are accessed.

#### Updating Entity Metadata

The platform allows updating certain metadata associated with named entities (tasks, workflows, launch plans identified by project, domain, and name across all versions). The `SynchronousFlyteClient.update_named_entity` method facilitates this.

Launch plans also support state updates (e.g., `ACTIVE` or `INACTIVE`) via `SynchronousFlyteClient.update_launch_plan`. Activating a launch plan automatically deactivates any other active launch plans with the same project, domain, and name.

### Execution Data Management

Execution data refers to the runtime information of tasks and workflows, including their inputs, outputs, status, and logs.

#### Launching Executions

Executions are instances of registered launch plans, tasks, or workflows. The `SynchronousFlyteClient.create_execution` method initiates a new execution.

The `FlyteRemote.execute` method provides a unified interface for launching executions from both local Python objects and remote entities:

```python
# Execute a registered workflow
execution = remote.execute(
    registered_workflow,
    inputs={"x": 5, "y": 10},
    execution_name_prefix="my_wf_run",
    wait=True # Wait for the execution to complete
)
print(f"Execution status: {execution.current_state}")
```

The `execute` method automatically handles the registration of local entities if they are not found on the remote system. It also supports various execution options, such as cache overwrites, interruptibility, environment variables, and tags.

#### Monitoring and Retrieving Execution Data

Execution status and data can be retrieved at various levels: workflow execution, node execution, and task execution.

The `SynchronousFlyteClient` offers methods like `get_execution`, `get_node_execution`, and `get_task_execution` to retrieve detailed information about specific executions. `get_execution_data`, `get_node_execution_data`, and `get_task_execution_data` provide access to execution inputs and outputs, often as signed URLs for large data blobs.

The `FlyteRemote.sync_execution`, `sync_node_execution`, and `sync_task_execution` methods update local execution objects with the latest remote state, including inputs, outputs, and the status of child executions.

```python
# Sync an execution to get its latest state and outputs
synced_execution = remote.sync_execution(execution)
if synced_execution.is_successful:
    print(f"Workflow outputs: {synced_execution.outputs.get('o0')}") # Assuming 'o0' is an output key
```

For human-in-the-loop workflows, `FlyteRemote` provides `list_signals`, `approve`, `reject`, and `set_input` methods to interact with signal nodes, allowing external input to influence workflow progression.

#### Recovering and Relaunching Executions

The platform supports recovering failed workflow executions from their last known failure point using `SynchronousFlyteClient.recover_execution`. This is useful for resuming long-running processes without re-running completed steps.

`SynchronousFlyteClient.relaunch_execution` creates a new execution based on a previous one, effectively re-running the entire workflow.

#### Backfilling Data

The `FlyteRemote.launch_backfill` method provides a specialized capability for re-running a launch plan over a historical date range. This generates a new workflow that orchestrates multiple executions of the specified launch plan for each interval within the range.

```python
from datetime import datetime, timedelta

# Launch a backfill for a launch plan
# backfill_execution = remote.launch_backfill(
#     project="flytesnacks",
#     domain="development",
#     from_date=datetime(2023, 1, 1),
#     to_date=datetime(2023, 1, 3),
#     launchplan="my_workflow",
#     launchplan_version="1.0.0",
#     execution_name="my_backfill_run",
#     parallel=True # Run each daily execution in parallel
# )
# print(f"Backfill execution launched: {backfill_execution.id}")
```

### Data I/O and Storage Integration

The platform integrates seamlessly with various remote storage solutions (e.g., S3, GCS, Azure Blob Storage) to manage large data inputs and outputs.

#### The `flyte://` Protocol

The platform extends the `fsspec` library to introduce a `flyte://` protocol. This allows developers to interact with remote data locations using familiar file system operations, abstracting away the underlying storage details. When you use `flyte://` paths, the system automatically handles signed URL generation and data transfer via the Admin service's data proxy.

#### Uploading Data

The `FlyteRemote.upload_file` method facilitates uploading local files to the remote data store. It leverages `SynchronousFlyteClient.get_upload_signed_url` to obtain a secure, temporary URL for direct upload.

```python
import pathlib
import os

# Create a dummy file
with open("my_local_file.txt", "w") as f:
    f.write("Hello, Flyte!")

# Upload the file
md5_bytes, remote_url = remote.upload_file(pathlib.Path("my_local_file.txt"))
print(f"Uploaded file to: {remote_url}")
os.remove("my_local_file.txt")
```

When uploading directories or multiple files, the system computes a consistent filename root based on the content hashes, ensuring that identical content results in the same remote location.

#### Downloading Data

The `FlyteRemote.download` method allows retrieving data from remote locations to your local file system. It supports downloading individual literals or recursively downloading all file-like objects within a `LiteralMap` or `LiteralsResolver`.

```python
# Assuming 'execution' is a FlyteWorkflowExecution object with outputs
# execution.outputs.download("local_output_dir")
# print("Outputs downloaded to local_output_dir")
```

The `SynchronousFlyteClient.get_data` method is a low-level way to retrieve data referenced by a Flyte URI, which can resolve to `LiteralMap`, `Literal`, or pre-signed URLs for larger data.

### Project and Domain Organization

Data and entities are organized hierarchically within projects and domains. Projects represent top-level organizational units, while domains provide further isolation within a project (e.g., `development`, `staging`, `production`).

The `SynchronousFlyteClient` provides `register_project`, `update_project`, `list_projects_paginated`, and `get_domains` to manage this organizational structure.

### Advanced Data Management Patterns

#### Interactive Mode

When `FlyteRemote` is initialized with `interactive_mode_enabled=True` (automatically detected in Jupyter notebooks), it enables "interactive task support." This allows tasks and workflows to be pickled and uploaded directly to the Admin service for execution, bypassing the need for explicit image builds and registrations. This is an alpha feature designed for rapid prototyping.

#### Performance Considerations

-   **Pagination**: Listing methods (e.g., `list_tasks_paginated`, `list_executions_paginated`) are paginated. Users must manage the `token` parameter to retrieve subsequent pages of results. Be aware that entries added between requests for different pages might lead to duplicate entries on subsequent pages.
-   **Caching**: The `get_task` and `get_launch_plan` methods in `SynchronousFlyteClient` use `lru_cache` for performance, reducing redundant network calls for frequently accessed entities.
-   **Execution Caching**: When launching executions, the `overwrite_cache` option can be used to force re-computation even if cached results are available, which is useful for testing or specific data reprocessing scenarios.

#### Error Handling

The client methods raise `grpc.RpcError` for gRPC-related issues and `FlyteEntityAlreadyExistsException` when attempting to create an entity that already exists with the same identifier. `FlyteEntityNotExistException` is raised when a requested entity is not found. Implement appropriate error handling in your applications.
<!--
key: summary_data_management_897e22d7-0984-4f49-a2fb-50d0090f4d4d
type: summary_end

-->
<!--
code_unit: flytekit.clients.friendly.SynchronousFlyteClient.get_execution_data
code_unit_type: class
help_text: ''
key: example_27e72e98-543e-4c3d-8c1a-b2e97f666db4
type: example

-->
<!--
code_unit: flytekit.clients.friendly.SynchronousFlyteClient.get_node_execution_data
code_unit_type: class
help_text: ''
key: example_d99bfb6c-4786-4b56-8cc4-25f5c76dc72e
type: example

-->
<!--
code_unit: flytekit.clients.friendly.SynchronousFlyteClient.get_task_execution_data
code_unit_type: class
help_text: ''
key: example_321b613d-0c1e-44dd-9eed-58221b868422
type: example

-->
<!--
code_unit: flytekit.clients.friendly.SynchronousFlyteClient.get_upload_signed_url
code_unit_type: class
help_text: ''
key: example_7e26a12e-84a9-43a5-84c5-3cf78b234f1d
type: example

-->
<!--
code_unit: flytekit.clients.friendly.SynchronousFlyteClient.get_download_signed_url
code_unit_type: class
help_text: ''
key: example_269b5f1d-6f8e-4ece-ab21-f9ab245711cb
type: example

-->
<!--
code_unit: flytekit.clients.friendly.SynchronousFlyteClient.get_data
code_unit_type: class
help_text: ''
key: example_ecc1d6a4-1815-4493-9e02-3b9a56e54789
type: example

-->
<!--
code_unit: flytekit.clients.friendly.SynchronousFlyteClient.get_download_artifact_signed_url
code_unit_type: class
help_text: ''
key: example_619baab7-5e20-434d-ad50-d9df85532465
type: example

-->
<!--
code_unit: flytekit.remote.remote.FlyteRemote.get
code_unit_type: class
help_text: ''
key: example_a0b3e195-e448-4623-87b2-1fbed660904e
type: example

-->
<!--
code_unit: flytekit.remote.remote.FlyteRemote.upload_file
code_unit_type: class
help_text: ''
key: example_32946843-da38-4e99-aeb4-7ac47464ac24
type: example

-->
<!--
code_unit: flytekit.remote.remote.FlyteRemote.download
code_unit_type: class
help_text: ''
key: example_2774b168-b8ae-4412-919a-1f8880e187e2
type: example

-->