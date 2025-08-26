
<!--
help_text: ''
key: summary_fetching_flyte_entities_916c64ac-a992-4d4f-a426-3ea341306d87
modules:
- flytekit.remote.remote
- flytekit.remote.entities
- flytekit.remote.lazy_entity
questions_to_answer: []
type: summary

-->
# Fetching Flyte Entities

The `FlyteRemote` object serves as the primary interface for interacting with a Flyte backend, enabling programmatic access to tasks, workflows, launch plans, and executions. It allows developers to fetch, register, and execute these entities.

## Initializing the Remote Connection

To begin, initialize a `FlyteRemote` object, providing configuration details for your Flyte backend. You can specify default `project` and `domain` values, which will be used if not explicitly provided in subsequent fetching or execution calls.

```python
from flytekit.remote.remote import FlyteRemote
from flytekit.configuration import Config

# Connect to a Flyte backend
# Example: connecting to a local sandbox
remote = FlyteRemote(
    config=Config.for_sandbox(),
    default_project="flytesnacks",
    default_domain="development",
)

# Alternatively, connect to a specific endpoint
# remote = FlyteRemote.for_endpoint(
#     endpoint="grpc.my-flyte-cluster.com:443",
#     insecure=False,
#     default_project="my_project",
#     default_domain="production",
# )
```

## Fetching Individual Entities

The `FlyteRemote` object provides dedicated methods to retrieve specific types of Flyte entities from the backend. Each method typically requires the `project`, `domain`, `name`, and optionally the `version` of the entity. If `project` or `domain` are omitted, the defaults set during `FlyteRemote` initialization are used. If `version` is omitted, the latest version of the entity is retrieved.

### Fetching Tasks

Use `fetch_task` to retrieve a `FlyteTask` object, which is a remote representation of a task registered on the Flyte backend.

```python
from flytekit.remote.remote import FlyteRemote

# Assuming 'remote' is an initialized FlyteRemote object
task = remote.fetch_task(name="my_task_name", version="abcdef1234")

# If project and domain are set as defaults during initialization:
# task = remote.fetch_task(name="my_task_name")

print(f"Fetched Task: {task.name} (Version: {task.id.version})")
```

### Fetching Workflows

The `fetch_workflow` method retrieves a `FlyteWorkflow` object. This object encapsulates the workflow definition, including its nodes, interfaces, and references to any sub-workflows or tasks.

```python
from flytekit.remote.remote import FlyteRemote

# Assuming 'remote' is an initialized FlyteRemote object
workflow = remote.fetch_workflow(name="my_workflow_name", version="v1.0.0")

print(f"Fetched Workflow: {workflow.name} (Version: {workflow.id.version})")
# Access workflow interface
print(f"Workflow Inputs: {workflow.interface.inputs}")
```

### Fetching Launch Plans

Use `fetch_launch_plan` to retrieve a `FlyteLaunchPlan` object. Launch plans are executable versions of workflows, often configured with default inputs or schedules.

```python
from flytekit.remote.remote import FlyteRemote

# Assuming 'remote' is an initialized FlyteRemote object
launch_plan = remote.fetch_launch_plan(name="my_launch_plan_name", version="v1.0.0")

print(f"Fetched Launch Plan: {launch_plan.name} (Version: {launch_plan.id.version})")
print(f"Associated Workflow ID: {launch_plan.workflow_id}")
```

To retrieve the currently active version of a launch plan, use `fetch_active_launchplan`. This is useful for scheduled launch plans or those frequently updated.

```python
from flytekit.remote.remote import FlyteRemote

# Assuming 'remote' is an initialized FlyteRemote object
active_lp = remote.fetch_active_launchplan(name="my_scheduled_lp")

if active_lp:
    print(f"Fetched Active Launch Plan: {active_lp.name} (Version: {active_lp.id.version})")
else:
    print("No active launch plan found.")
```

### Fetching Workflow Executions

The `fetch_execution` method retrieves a `FlyteWorkflowExecution` object, representing a specific run of a workflow. This object provides access to the execution's status, inputs, outputs, and details of its constituent node and task executions.

```python
from flytekit.remote.remote import FlyteRemote

# Assuming 'remote' is an initialized FlyteRemote object
# 'execution_name' is the unique identifier for a specific execution run
execution = remote.fetch_execution(name="my_workflow_execution_id")

print(f"Fetched Execution: {execution.id.name} (Status: {execution.current_state})")

# To access outputs after completion
if execution.is_successful:
    print(f"Execution Outputs: {execution.outputs}")
```

## Lazy Fetching

For scenarios where an entity might not be immediately needed, or to defer network calls, `FlyteRemote` offers lazy fetching capabilities. Methods like `fetch_task_lazy` and `fetch_workflow_lazy` return a `LazyEntity` object. The actual fetching from the Flyte backend only occurs when an attribute or method of the `LazyEntity` is accessed for the first time.

```python
from flytekit.remote.remote import FlyteRemote

# Assuming 'remote' is an initialized FlyteRemote object
lazy_task = remote.fetch_task_lazy(name="my_task_name", version="abcdef1234")

print(f"Lazy entity created for: {lazy_task.name}") # 'name' is directly accessible without fetching

# The actual network call to fetch the task happens here, on first access of 'id'
print(f"Task ID: {lazy_task.id}")

# Subsequent accesses do not trigger re-fetching
print(f"Task Type: {lazy_task.type}")
```

This approach can improve application startup performance by only fetching necessary entities when they are actively used.

## Listing Entities

Beyond fetching individual entities, `FlyteRemote` provides methods to list collections of entities, often with filtering and pagination options.

### Listing Recent Executions

Use `recent_executions` to retrieve a list of `FlyteWorkflowExecution` objects, ordered by the most recent first.

```python
from flytekit.remote.remote import FlyteRemote

# Assuming 'remote' is an initialized FlyteRemote object
recent_execs = remote.recent_executions(limit=5)

print("Recent Executions:")
for exec_obj in recent_execs:
    print(f"- {exec_obj.id.name} (Status: {exec_obj.current_state})")
```

### Listing Tasks by Version

The `list_tasks_by_version` method allows you to retrieve all tasks that share a specific version string within a given project and domain.

```python
from flytekit.remote.remote import FlyteRemote

# Assuming 'remote' is an initialized FlyteRemote object
tasks_v1 = remote.list_tasks_by_version(version="v1.0.0")

print("Tasks with version v1.0.0:")
for task_obj in tasks_v1:
    print(f"- {task_obj.name}")
```

### Listing Signals

Signals are a mechanism for human-in-the-loop interactions within workflows. `list_signals` allows you to retrieve pending signals for a given workflow execution.

```python
from flytekit.remote.remote import FlyteRemote

# Assuming 'remote' is an initialized FlyteRemote object
execution_name = "my_workflow_execution_id"
signals = remote.list_signals(execution_name=execution_name)

print(f"Signals for execution {execution_name}:")
for signal in signals:
    print(f"- Signal ID: {signal.id.signal_id}, Type: {signal.type}")
```

## Important Considerations

*   **Project and Domain Defaults:** Always be mindful of the `default_project` and `default_domain` set during `FlyteRemote` initialization. Explicitly providing these parameters in fetching methods overrides the defaults.
*   **Versioning:** Flyte entities are versioned. When fetching, specifying a `version` ensures you retrieve a precise entity. Omitting it typically fetches the latest registered version.
*   **Remote Entity Representation:** The objects returned by fetching methods (`FlyteTask`, `FlyteWorkflow`, `FlyteLaunchPlan`, `FlyteWorkflowExecution`) are remote representations. They contain metadata and references to the backend state. To interact with their inputs/outputs or underlying components, further synchronization or access methods might be required (e.g., `execution.outputs`).
*   **Error Handling:** Fetching operations can raise exceptions like `FlyteEntityNotExistException` if the requested entity is not found, or `FlyteAssertion` for invalid arguments. Implement appropriate error handling in your applications.
*   **Performance:** For operations involving many entities or frequent fetching, consider the implications of network latency. Lazy fetching can help manage this by deferring data retrieval until necessary.
<!--
key: summary_fetching_flyte_entities_916c64ac-a992-4d4f-a426-3ea341306d87
type: summary_end

-->