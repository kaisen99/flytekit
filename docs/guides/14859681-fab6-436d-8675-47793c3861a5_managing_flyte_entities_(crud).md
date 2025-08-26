
<!--
help_text: ''
key: summary_managing_flyte_entities_(crud)_64a16dcb-87d6-4032-aa1a-a2dd9ab49445
modules:
- flytekit.clients.friendly.SynchronousFlyteClient
- flytekit.remote.remote.FlyteRemote
- flytekit.remote.entities.FlyteTask
- flytekit.remote.entities.FlyteWorkflow
- flytekit.remote.entities.FlyteLaunchPlan
questions_to_answer: []
type: summary

-->
# Managing Flyte Entities (CRUD)

Interacting with Flyte entities programmatically involves creating, retrieving, updating, and executing them. The primary interface for developers is the `FlyteRemote` client, which provides a high-level, user-friendly abstraction over the underlying Flyte Admin service. For direct, low-level gRPC interactions, the `SynchronousFlyteClient` is available.

## Connecting to Flyte

To begin managing Flyte entities, instantiate the `FlyteRemote` client. This client requires a configuration that specifies the Flyte Admin endpoint and other platform details.

```python
from flytekit.remote.remote import FlyteRemote
from flytekit.configuration import Config

# Connect to a Flyte sandbox environment
remote = FlyteRemote.for_sandbox(
    default_project="flytesnacks",
    default_domain="development",
    data_upload_location="s3://my-s3-bucket/data" # Adjust for your environment
)

# Or connect to a specific endpoint
# remote = FlyteRemote.for_endpoint(
#     endpoint="your.flyte.admin:port",
#     insecure=True, # Set to False for SSL/TLS
#     default_project="my_project",
#     default_domain="my_domain",
# )
```

The `FlyteRemote` client manages a `SynchronousFlyteClient` internally, which can be accessed for more granular control:

```python
admin_client = remote.client
```

## Managing Tasks

Tasks are the fundamental units of computation in Flyte.

### Registering Tasks

Registering a task makes its definition available to the Flyte Admin service. When registering a local Python task, the `FlyteRemote` client handles the serialization and upload of the task's code.

```python
from flytekit import task
from flytekit.remote.remote import FlyteRemote
from flytekit.configuration import Config

remote = FlyteRemote.for_sandbox(default_project="flytesnacks", default_domain="development")

@task
def my_task(a: int, b: int) -> int:
    return a + b

# Register the task
registered_task = remote.register_task(my_task)
print(f"Registered task: {registered_task.id.name} version {registered_task.id.version}")
```

When registering a workflow that contains tasks, the tasks are automatically registered as part of the workflow registration process.

### Fetching Tasks

Retrieve registered task definitions from the Flyte Admin service using their identifiers (project, domain, name, and version). If the version is not specified, the latest version is fetched.

```python
from flytekit.remote.remote import FlyteRemote

remote = FlyteRemote.for_sandbox(default_project="flytesnacks", default_domain="development")

# Fetch a specific task by name and version
fetched_task = remote.fetch_task(name="my_task", version="your_task_version")
print(f"Fetched task: {fetched_task.id.name} version {fetched_task.id.version}")

# List tasks by version
tasks_in_version = remote.list_tasks_by_version(version="your_task_version")
for task_obj in tasks_in_version:
    print(f"  - {task_obj.id.name}")
```

### Executing Tasks

Execute a task by providing its inputs. The `execute` method handles the creation of a workflow execution.

```python
from flytekit.remote.remote import FlyteRemote

remote = FlyteRemote.for_sandbox(default_project="flytesnacks", default_domain="development")

# Assuming 'registered_task' from the registration example
# Execute a registered task
execution = remote.execute(
    registered_task,
    inputs={"a": 10, "b": 20},
    execution_name_prefix="my_task_exec",
    wait=True # Wait for the execution to complete
)

if execution.is_successful():
    print(f"Task execution successful. Outputs: {execution.outputs}")
else:
    print(f"Task execution failed. Error: {execution.error}")
```

## Managing Workflows

Workflows orchestrate tasks and other workflows.

### Registering Workflows

Registering a workflow makes its definition available to Flyte Admin. This process also registers any tasks or sub-workflows defined within it. By default, a corresponding default launch plan is also created.

```python
from flytekit import task, workflow
from flytekit.remote.remote import FlyteRemote

remote = FlyteRemote.for_sandbox(default_project="flytesnacks", default_domain="development")

@task
def add(a: int, b: int) -> int:
    return a + b

@workflow
def my_workflow(x: int, y: int) -> int:
    sum_val = add(a=x, b=y)
    return sum_val

# Register the workflow
registered_workflow = remote.register_workflow(my_workflow)
print(f"Registered workflow: {registered_workflow.id.name} version {registered_workflow.id.version}")
```

For advanced scenarios like packaging source code for remote execution, use `register_script` or `fast_register_workflow`.

### Fetching Workflows

Retrieve registered workflow definitions.

```python
from flytekit.remote.remote import FlyteRemote

remote = FlyteRemote.for_sandbox(default_project="flytesnacks", default_domain="development")

# Fetch a specific workflow
fetched_workflow = remote.fetch_workflow(name="my_workflow", version="your_workflow_version")
print(f"Fetched workflow: {fetched_workflow.id.name} version {fetched_workflow.id.version}")
```

### Executing Workflows

Execute a workflow by providing its inputs. This implicitly uses a launch plan.

```python
from flytekit.remote.remote import FlyteRemote

remote = FlyteRemote.for_sandbox(default_project="flytesnacks", default_domain="development")

# Assuming 'registered_workflow' from the registration example
# Execute a registered workflow
execution = remote.execute(
    registered_workflow,
    inputs={"x": 5, "y": 7},
    execution_name_prefix="my_workflow_exec",
    wait=True
)

if execution.is_successful():
    print(f"Workflow execution successful. Outputs: {execution.outputs}")
else:
    print(f"Workflow execution failed. Error: {execution.error}")
```

## Managing Launch Plans

Launch plans provide a way to parameterize and schedule workflows.

### Registering Launch Plans

Register a launch plan to make it available for execution or scheduling. If the underlying workflow is not already registered, it will be registered as part of this process.

```python
from flytekit import LaunchPlan
from flytekit.remote.remote import FlyteRemote

remote = FlyteRemote.for_sandbox(default_project="flytesnacks", default_domain="development")

# Assuming 'my_workflow' from the workflow example
my_lp = LaunchPlan.get_or_create(workflow=my_workflow, name="my_workflow_lp")

# Register the launch plan
registered_lp = remote.register_launch_plan(my_lp, version="v1")
print(f"Registered launch plan: {registered_lp.id.name} version {registered_lp.id.version}")
```

### Fetching Launch Plans

Retrieve registered launch plan definitions.

```python
from flytekit.remote.remote import FlyteRemote

remote = FlyteRemote.for_sandbox(default_project="flytesnacks", default_domain="development")

# Fetch a specific launch plan
fetched_lp = remote.fetch_launch_plan(name="my_workflow_lp", version="v1")
print(f"Fetched launch plan: {fetched_lp.id.name} version {fetched_lp.id.version}")

# Fetch the currently active launch plan for a given name
active_lp = remote.fetch_active_launchplan(name="my_workflow_lp")
if active_lp:
    print(f"Active launch plan: {active_lp.id.name} version {active_lp.id.version}")
```

### Updating Launch Plans

The state of a launch plan (e.g., `ACTIVE` or `INACTIVE`) can be updated. Activating a launch plan for a given project, domain, and name will automatically deactivate any other active launch plans with the same identifiers.

```python
from flytekit.remote.remote import FlyteRemote
from flytekit.models.launch_plan import LaunchPlanState

remote = FlyteRemote.for_sandbox(default_project="flytesnacks", default_domain="development")

# Assuming 'registered_lp' from the registration example
# Activate the launch plan
remote.activate_launchplan(registered_lp.id)
print(f"Launch plan {registered_lp.id.name} activated.")

# The low-level client can also be used to update state directly
# remote.client.update_launch_plan(registered_lp.id, LaunchPlanState.INACTIVE)
```

### Executing Launch Plans

Execute a launch plan to initiate a workflow execution.

```python
from flytekit.remote.remote import FlyteRemote

remote = FlyteRemote.for_sandbox(default_project="flytesnacks", default_domain="development")

# Assuming 'fetched_lp' from the fetching example
# Execute the launch plan
execution = remote.execute(
    fetched_lp,
    inputs={"x": 1, "y": 2},
    execution_name_prefix="my_lp_exec",
    wait=True
)

if execution.is_successful():
    print(f"Launch plan execution successful. Outputs: {execution.outputs}")
else:
    print(f"Launch plan execution failed. Error: {execution.error}")
```

## Managing Executions

Executions represent running instances of workflows or tasks.

### Fetching Executions

Retrieve details about a specific workflow execution or list recent executions.

```python
from flytekit.remote.remote import FlyteRemote

remote = FlyteRemote.for_sandbox(default_project="flytesnacks", default_domain="development")

# Assuming 'execution' from a previous example
# Fetch a specific execution by its name
fetched_execution = remote.fetch_execution(name=execution.id.name)
print(f"Fetched execution state: {fetched_execution.current_state}")

# List recent executions
recent_executions = remote.recent_executions(limit=5)
for exec_obj in recent_executions:
    print(f"  - Execution: {exec_obj.id.name}, State: {exec_obj.current_state}")
```

To access execution inputs and outputs, use `get_execution_data` on the low-level client or rely on the `FlyteWorkflowExecution` object's `inputs` and `outputs` properties after syncing.

### Syncing Execution State

The `sync_execution` method updates a `FlyteWorkflowExecution` object with the latest state from the Flyte Admin service, including its inputs, outputs, and the status of its underlying node and task executions.

```python
from flytekit.remote.remote import FlyteRemote

remote = FlyteRemote.for_sandbox(default_project="flytesnacks", default_domain="development")

# Assuming 'execution' is an existing FlyteWorkflowExecution object
synced_execution = remote.sync_execution(execution, sync_nodes=True)
print(f"Synced execution state: {synced_execution.current_state}")

# Access node executions
for node_id, node_exec in synced_execution.node_executions.items():
    print(f"  Node {node_id} state: {node_exec.current_state}")
    # Access task executions within a node
    for task_exec in node_exec.task_executions:
        print(f"    Task {task_exec.id.task_id.name} state: {task_exec.current_state}")
```

Setting `sync_nodes=False` can make the sync operation faster if detailed node and task execution information is not immediately required.

### Terminating Executions

Stop a running workflow execution.

```python
from flytekit.remote.remote import FlyteRemote

remote = FlyteRemote.for_sandbox(default_project="flytesnacks", default_domain="development")

# Assuming 'execution' is a running FlyteWorkflowExecution object
remote.terminate(execution, cause="User requested termination")
print(f"Execution {execution.id.name} terminated.")
```

### Recovering and Relaunching Executions

The low-level client provides methods to recover a failed execution from its last known failure point or relaunch an entirely new execution based on a previous one.

```python
from flytekit.remote.remote import FlyteRemote

remote = FlyteRemote.for_sandbox(default_project="flytesnacks", default_domain="development")

# Assuming 'failed_execution_id' is a WorkflowExecutionIdentifier for a failed execution
# Recover a failed execution
# recovered_exec_id = remote.client.recover_execution(failed_execution_id)
# print(f"Recovered execution: {recovered_exec_id.name}")

# Relaunch an execution
# relaunched_exec_id = remote.client.relaunch_execution(existing_execution_id)
# print(f"Relaunched execution: {relaunched_exec_id.name}")
```

### Managing Signals

Signals allow for external interaction with a running workflow execution, typically for human approval steps or injecting values.

```python
from flytekit.remote.remote import FlyteRemote

remote = FlyteRemote.for_sandbox(default_project="flytesnacks", default_domain="development")

# Assuming 'execution_name' is the name of a running workflow execution
# List signals for an execution
signals = remote.list_signals(execution_name="my_workflow_exec_name")
for signal in signals:
    print(f"Signal ID: {signal.id.signal_id}, Type: {signal.type}")

# Approve a signal (for boolean signals)
# remote.approve(signal_id="my_approval_signal", execution_name="my_workflow_exec_name")

# Set an input for a signal (for data-input signals)
# remote.set_input(
#     signal_id="my_data_signal",
#     execution_name="my_workflow_exec_name",
#     value="some_string_value",
#     python_type=str
# )
```

## Managing Projects and Domains

Projects and domains provide organizational namespaces within Flyte.

### Registering Projects

New projects can be registered with the Flyte Admin service.

```python
from flytekit.remote.remote import FlyteRemote
from flytekit.models.project import Project

remote = FlyteRemote.for_sandbox()

new_project = Project(id="my_new_project", name="My New Project", description="A project for documentation examples")
# remote.client.register_project(new_project)
# print(f"Project {new_project.name} registered.")
```

### Updating Projects

Existing projects can be updated.

```python
from flytekit.remote.remote import FlyteRemote
from flytekit.models.project import Project

remote = FlyteRemote.for_sandbox()

# Assuming 'my_new_project' exists
updated_project = Project(id="my_new_project", name="My Updated Project Name", description="Updated description")
# remote.client.update_project(updated_project)
# print(f"Project {updated_project.id} updated.")
```

### Listing Projects and Domains

Retrieve lists of registered projects and domains.

```python
from flytekit.remote.remote import FlyteRemote

remote = FlyteRemote.for_sandbox()

# List all projects
projects = remote.list_projects()
print("Registered Projects:")
for p in projects:
    print(f"  - {p.name} (ID: {p.id})")

# List all domains
domains = remote.get_domains()
print("Registered Domains:")
for d in domains:
    print(f"  - {d.id}")
```

## Advanced Operations

### Data Proxy (Inputs/Outputs)

The `FlyteRemote` client provides convenient methods to interact with data stored in Flyte's data proxy, such as fetching execution inputs/outputs or uploading files for fast registration.

```python
from flytekit.remote.remote import FlyteRemote
import os

remote = FlyteRemote.for_sandbox(default_project="flytesnacks", default_domain="development")

# Example: Upload a file
# with open("my_local_file.txt", "w") as f:
#     f.write("Hello Flyte!")
# md5_bytes, upload_url = remote.upload_file(pathlib.Path("my_local_file.txt"))
# print(f"Uploaded file to: {upload_url}")

# Example: Download execution outputs
# Assuming 'execution' is a completed FlyteWorkflowExecution object
# remote.download(execution.outputs, download_to="./execution_outputs")
# print("Execution outputs downloaded.")

# Get signed URLs for artifacts (e.g., deck HTML)
# from flytekit.models.literals import ArtifactType
# deck_url_response = remote.client.get_download_artifact_signed_url(
#     node_id="n0", # Example node ID
#     project="flytesnacks",
#     domain="development",
#     name="my_workflow_exec_name", # Execution name
#     artifact_type=ArtifactType.ARTIFACT_TYPE_DECK
# )
# print(f"Deck URL: {deck_url_response.signed_url}")
```

### Named Entity Attributes

Custom attributes can be set and retrieved for projects, project-domain combinations, and workflows. These attributes can influence behavior like resource quotas or execution priorities.

```python
from flytekit.remote.remote import FlyteRemote
from flytekit.models.admin.named_entity import NamedEntityIdentifier, NamedEntityMetadata
from flytekit.models.identifier import ResourceType

remote = FlyteRemote.for_sandbox(default_project="flytesnacks", default_domain="development")

# Update metadata for a named entity (e.g., a workflow)
# named_entity_id = NamedEntityIdentifier(
#     project="flytesnacks",
#     domain="development",
#     name="my_workflow"
# )
# metadata = NamedEntityMetadata(description="My updated workflow description", can_archive=True)
# remote.client.update_named_entity(ResourceType.WORKFLOW, named_entity_id, metadata)

# Get project-domain attributes
# attributes = remote.client.get_project_domain_attributes(
#     project="flytesnacks",
#     domain="development",
#     resource_type=ResourceType.TASK # Example resource type
# )
# print(f"Project-domain attributes: {attributes}")
```

### Fast Registration / Script Mode

Fast registration allows for registering entities by uploading a zipped package of the source code, which Flyte Admin then uses to compile and register the entities. This is particularly useful for large codebases or when working in environments where local compilation is not desired.

```python
from flytekit.remote.remote import FlyteRemote
from flytekit import task, workflow
import os
import pathlib

remote = FlyteRemote.for_sandbox(default_project="flytesnacks", default_domain="development")

# Example: Register a workflow using script mode
# Create a dummy Python file for demonstration
# with open("my_script.py", "w") as f:
#     f.write("""
# from flytekit import task, workflow
# @task
# def script_add(a: int, b: int) -> int:
#     return a + b
# @workflow
# def script_wf(x: int, y: int) -> int:
#     return script_add(a=x, b=y)
# """)

# from my_script import script_wf # Import the workflow from the dummy file

# registered_script_wf = remote.fast_register_workflow(
#     script_wf,
#     version="script_v1",
#     source_path=os.getcwd(), # Path to the directory containing my_script.py
# )
# print(f"Fast registered workflow: {registered_script_wf.id.name} version {registered_script_wf.id.version}")

# os.remove("my_script.py") # Clean up
```

### Backfills

The `launch_backfill` method facilitates creating and launching workflows to re-run historical data for a given launch plan.

```python
from flytekit.remote.remote import FlyteRemote
from datetime import datetime, timedelta

remote = FlyteRemote.for_sandbox(default_project="flytesnacks", default_domain="development")

# Assuming 'my_workflow_lp' is a registered launch plan
# backfill_execution = remote.launch_backfill(
#     project="flytesnacks",
#     domain="development",
#     from_date=datetime.now() - timedelta(days=7),
#     to_date=datetime.now(),
#     launchplan="my_workflow_lp",
#     dry_run=True # Set to False to actually register and execute
# )
# if backfill_execution:
#     print(f"Backfill workflow created (dry run): {backfill_execution.name}")
```

### Console URLs

Generate direct links to the Flyte UI for entities and executions.

```python
from flytekit.remote.remote import FlyteRemote

remote = FlyteRemote.for_sandbox(default_project="flytesnacks", default_domain="development")

# Assuming 'registered_workflow' and 'execution' from previous examples
workflow_console_url = remote.generate_console_url(registered_workflow)
print(f"Workflow Console URL: {workflow_console_url}")

execution_console_url = remote.generate_console_url(execution)
print(f"Execution Console URL: {execution_console_url}")
```

## Low-Level Client Access

The `SynchronousFlyteClient` provides direct access to the Flyte Admin gRPC API. This is useful for operations not directly exposed by `FlyteRemote` or for fine-grained control over API requests.

```python
from flytekit.clients.sync import SynchronousFlyteClient
from flytekit.models.core.identifier import Identifier, ResourceType
from flytekit.models.task import TaskSpec, TaskMetadata
from flytekit.models.interface import TypedInterface, Variable
from flytekit.models.types import LiteralType, SimpleType

# Instantiate the low-level client
# client = SynchronousFlyteClient("your.flyte.admin:port", insecure=True)

# Example: Create a task directly using the low-level client
# task_id = Identifier(
#     resource_type=ResourceType.TASK,
#     project="flytesnacks",
#     domain="development",
#     name="direct_task",
#     version="1.0.0"
# )
# task_spec = TaskSpec(
#     template=TaskTemplate(
#         id=task_id,
#         type="python-task",
#         metadata=TaskMetadata(discoverable=True, timeout=timedelta(minutes=1), retries=0),
#         interface=TypedInterface(
#             inputs={"x": Variable(type=LiteralType(simple=SimpleType.INTEGER), description="")},
#             outputs={"y": Variable(type=LiteralType(simple=SimpleType.INTEGER), description="")}
#         ),
#         custom={}
#     )
# )
# client.create_task(task_id, task_spec)
# print(f"Task 'direct_task' created via low-level client.")
```

## Important Considerations

*   **Versioning**: Flyte entities are versioned. When registering, ensure a consistent versioning strategy. `FlyteRemote` can auto-generate versions based on content hash in interactive mode.
*   **Immutability**: Once registered, task, workflow, and launch plan definitions are immutable. Updates typically involve registering a new version.
*   **Pagination**: Listing methods (e.g., `list_tasks_paginated`, `list_executions_paginated`) are paginated. Always check the returned `token` to fetch subsequent pages.
*   **Error Handling**: API calls can raise `grpc.RpcError` or `FlyteEntityAlreadyExistsException` (when registering an identical entity) and `FlyteEntityNotExistException` (when fetching a non-existent entity). Implement appropriate error handling.
*   **Interactive Mode**: `FlyteRemote` can detect if it's running in an interactive environment (like Jupyter notebooks). In this mode, it can automatically pickle and upload local task/workflow definitions for registration, simplifying the development loop.
*   **Resource Identifiers**: Most operations require a `project`, `domain`, `name`, and `version` to uniquely identify an entity. Ensure these are consistently provided or configured as defaults in `FlyteRemote`.
<!--
key: summary_managing_flyte_entities_(crud)_64a16dcb-87d6-4032-aa1a-a2dd9ab49445
type: summary_end

-->
<!--
code_unit: flytekit.clients.friendly.SynchronousFlyteClient.create_task
code_unit_type: class
help_text: ''
key: example_f517ad85-4a57-4c03-be54-9450560bea83
type: example

-->
<!--
code_unit: flytekit.clients.friendly.SynchronousFlyteClient.list_tasks_paginated
code_unit_type: class
help_text: ''
key: example_1d67f702-1850-4a62-a395-be15ce9b07cb
type: example

-->
<!--
code_unit: flytekit.clients.friendly.SynchronousFlyteClient.get_task
code_unit_type: class
help_text: ''
key: example_9a335f2e-d577-44a4-ba61-d7fcbf988b67
type: example

-->
<!--
code_unit: flytekit.clients.friendly.SynchronousFlyteClient.create_workflow
code_unit_type: class
help_text: ''
key: example_5d557bb4-ce6e-4f7a-ae99-46ae7aa361e5
type: example

-->
<!--
code_unit: flytekit.clients.friendly.SynchronousFlyteClient.list_workflows_paginated
code_unit_type: class
help_text: ''
key: example_418602c3-6ab6-4b8d-ae7c-1973f1926536
type: example

-->
<!--
code_unit: flytekit.clients.friendly.SynchronousFlyteClient.get_workflow
code_unit_type: class
help_text: ''
key: example_e212dd6b-bb46-486a-9072-8bdd62c96402
type: example

-->
<!--
code_unit: flytekit.clients.friendly.SynchronousFlyteClient.create_launch_plan
code_unit_type: class
help_text: ''
key: example_87827bb2-3180-43d8-9e3d-7f26ab5e52b7
type: example

-->
<!--
code_unit: flytekit.clients.friendly.SynchronousFlyteClient.list_launch_plans_paginated
code_unit_type: class
help_text: ''
key: example_b6b1e890-71d5-42f4-8689-f4a185c81b35
type: example

-->
<!--
code_unit: flytekit.clients.friendly.SynchronousFlyteClient.get_launch_plan
code_unit_type: class
help_text: ''
key: example_7c5df9d6-0f60-4246-9e51-8337ddccf977
type: example

-->
<!--
code_unit: flytekit.clients.friendly.SynchronousFlyteClient.update_launch_plan
code_unit_type: class
help_text: ''
key: example_c33a776e-e6de-44ac-9998-d665e0b11282
type: example

-->
<!--
code_unit: flytekit.remote.remote.FlyteRemote.fetch_task
code_unit_type: class
help_text: ''
key: example_e9a6a52b-fc2a-47ba-988b-98494b8b401f
type: example

-->
<!--
code_unit: flytekit.remote.remote.FlyteRemote.fetch_workflow
code_unit_type: class
help_text: ''
key: example_ba88f929-bcbf-4e2f-9e8f-f7f8f378eb1f
type: example

-->
<!--
code_unit: flytekit.remote.remote.FlyteRemote.fetch_launch_plan
code_unit_type: class
help_text: ''
key: example_1f21db1a-d11f-4752-bff9-f9c40ebbc142
type: example

-->
<!--
code_unit: flytekit.remote.remote.FlyteRemote.register_task
code_unit_type: class
help_text: ''
key: example_6f29da91-beff-4f78-8cb8-ebcb88ffb24b
type: example

-->
<!--
code_unit: flytekit.remote.remote.FlyteRemote.register_workflow
code_unit_type: class
help_text: ''
key: example_1eceb089-995d-497f-b6a6-c11c4ef42b70
type: example

-->
<!--
code_unit: flytekit.remote.remote.FlyteRemote.register_launch_plan
code_unit_type: class
help_text: ''
key: example_cb3bd7cf-3561-4815-b9b8-c9e25ef25685
type: example

-->