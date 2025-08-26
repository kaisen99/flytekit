
<!--
help_text: ''
key: summary_monitoring_and_managing_executions_808e28cb-6220-4d71-8d66-47ddb9337002
modules:
- flytekit.remote.remote
- flytekit.remote.executions
- flytekit.remote.metrics
questions_to_answer: []
type: summary

-->
# Monitoring and Managing Executions

Flyte provides robust capabilities for monitoring and managing your workflow, task, and node executions on a remote backend. The `FlyteRemote` object serves as the primary interface for these operations, allowing you to programmatically interact with the lifecycle and state of your running entities.

## Accessing Executions

To monitor and manage executions, first obtain a reference to the execution itself. You can either fetch a specific execution by its identifier or list recent executions.

### Fetching a Specific Execution

Retrieve a `FlyteWorkflowExecution` object using its unique identifier, which includes the project, domain, and name.

```python
from flytekit.remote.remote import FlyteRemote

# Initialize FlyteRemote
remote = FlyteRemote.auto() # Or use .for_endpoint(), .for_sandbox()

# Fetch a specific execution
# Replace with your execution's project, domain, and name
execution = remote.fetch_execution(
    project="flytesnacks",
    domain="development",
    name="f8a7b6c5d4e3f2a1b9c8d7e6f5a4b3c2d1e0f9a8"
)
print(f"Fetched execution: {execution.id.name}")
```

### Listing Recent Executions

List a collection of recent workflow executions, optionally applying filters or limits.

```python
from flytekit.remote.remote import FlyteRemote
from flytekit.models.common import Sort, SortOrdering
from flytekit.models.filters import Filter

remote = FlyteRemote.auto()

# List the 5 most recent executions in a specific project and domain
recent_executions = remote.recent_executions(
    project="flytesnacks",
    domain="development",
    limit=5,
    sort_by=Sort("created_at", SortOrdering.DESCENDING)
)

for exec_obj in recent_executions:
    print(f"Execution: {exec_obj.id.name}, Phase: {exec_obj.closure.phase}")

# Example with a filter for a specific launch plan
# from flytekit.models.filters import Filter
# from flytekit.models.common import NamedEntityIdentifier
#
# lp_id = NamedEntityIdentifier(project="flytesnacks", domain="development", name="my_launch_plan_name")
# filter_by_lp = Filter.from_python_std(f"eq(launch_plan.name,{lp_id.name})")
# filtered_executions = remote.recent_executions(
#     project="flytesnacks",
#     domain="development",
#     filters=[filter_by_lp]
# )
```

## Understanding Execution State

Execution objects (`FlyteWorkflowExecution`, `FlyteNodeExecution`, `FlyteTaskExecution`) provide properties to inspect their current state, inputs, and outputs.

*   **`is_done`**: A boolean indicating if the execution has reached a terminal state (succeeded, failed, aborted, skipped, timed out).
*   **`is_successful`**: (Workflow executions only) A boolean indicating if the workflow execution completed successfully.
*   **`error`**: If the execution failed, this property returns an `ExecutionError` object containing details about the failure. Accessing this property on an in-progress execution raises a `FlyteAssertion` error.
*   **`inputs`**: A `LiteralsResolver` object providing access to the execution's input values.
*   **`outputs`**: A `LiteralsResolver` object providing access to the execution's output values. This property is only available once the execution is `is_done` and `is_successful`. Accessing it otherwise raises a `FlyteAssertion` error.

```python
# Assuming 'execution' is a FlyteWorkflowExecution object
if execution.is_done:
    print(f"Execution {execution.id.name} is complete.")
    if execution.is_successful:
        print("Execution succeeded!")
        # Access outputs
        try:
            outputs = execution.outputs
            print(f"Outputs: {outputs.to_dict()}")
        except Exception as e:
            print(f"Could not retrieve outputs: {e}")
    else:
        print("Execution failed or was aborted.")
        if execution.error:
            print(f"Error message: {execution.error.message}")
            print(f"Error code: {execution.error.code}")
else:
    print(f"Execution {execution.id.name} is still running (Phase: {execution.closure.phase}).")

# Access inputs (available regardless of execution phase)
inputs = execution.inputs
print(f"Inputs: {inputs.to_dict()}")
```

## Synchronizing Execution State

Execution objects maintain a local representation of their state. To get the most up-to-date information from the Flyte backend, synchronize the object.

The `sync()` method on `FlyteWorkflowExecution` updates its internal state by fetching the latest information from the remote backend.

```python
# Assuming 'execution' is a FlyteWorkflowExecution object
print(f"Current phase: {execution.closure.phase}")

# Sync the execution to get the latest state
updated_execution = execution.sync()
print(f"Updated phase: {updated_execution.closure.phase}")

# To also fetch details about all underlying node executions (recursively), set sync_nodes=True.
# This can be more network-intensive for large workflows.
updated_execution_with_nodes = execution.sync(sync_nodes=True)
print(f"Number of node executions: {len(updated_execution_with_nodes.node_executions)}")
```

## Waiting for Execution Completion

For scenarios where you need to block execution until a workflow completes, use the `wait()` method. This method polls the Flyte backend at a specified interval until the execution reaches a terminal state or a timeout occurs.

```python
from datetime import timedelta
from flytekit.remote.remote import FlyteRemote

remote = FlyteRemote.auto()

# Assume 'my_launch_plan' is a registered LaunchPlan object
# execution = remote.execute(my_launch_plan, inputs={"x": 10}) # Example of starting an execution

# Wait for the execution to complete, with a 5-minute timeout and 10-second poll interval
try:
    completed_execution = execution.wait(timeout=timedelta(minutes=5), poll_interval=timedelta(seconds=10))
    if completed_execution.is_successful:
        print(f"Execution {completed_execution.id.name} completed successfully.")
        print(f"Outputs: {completed_execution.outputs.to_dict()}")
    else:
        print(f"Execution {completed_execution.id.name} failed: {completed_execution.error.message}")
except Exception as e:
    print(f"Waiting for execution failed: {e}")
```

## Controlling Executions

Flyte provides mechanisms to terminate running executions and interact with human-in-the-loop signals.

### Terminating Executions

You can terminate a running workflow execution by providing its identifier and a cause for termination.

```python
from flytekit.remote.remote import FlyteRemote

remote = FlyteRemote.auto()

# Assume 'running_execution' is a FlyteWorkflowExecution object
# running_execution = remote.fetch_execution(project="...", domain="...", name="...")

if not running_execution.is_done:
    print(f"Terminating execution {running_execution.id.name}...")
    remote.terminate(running_execution, cause="User requested termination")
    print("Termination request sent.")
else:
    print(f"Execution {running_execution.id.name} is already in a terminal state.")
```

### Interacting with Signals

Flyte supports human-in-the-loop workflows using "signals," which allow a workflow to pause and wait for external input or approval.

*   **`list_signals`**: Retrieves a list of pending signals for a given execution.
*   **`approve` / `reject`**: Used to respond to boolean signals (e.g., approval gates).
*   **`set_input` / `set_signal`**: Used to provide arbitrary input for `wait_for_input` signals.

```python
from flytekit.remote.remote import FlyteRemote
from flytekit.models.filters import Filter
from flytekit.models.common import SignalListRequest

remote = FlyteRemote.auto()

# Assume 'execution_name' refers to an execution with pending signals
execution_name = "my_workflow_execution_with_signals"
project = "flytesnacks"
domain = "development"

# List all pending signals for the execution
signals = remote.list_signals(execution_name=execution_name, project=project, domain=domain)
if signals:
    print(f"Found {len(signals)} pending signals:")
    for signal in signals:
        print(f"  Signal ID: {signal.id.signal_id}, Type: {signal.type.simple_type}")

    # Example: Approve a signal named "user_approval"
    signal_to_approve = "user_approval"
    if any(s.id.signal_id == signal_to_approve for s in signals):
        print(f"Approving signal '{signal_to_approve}'...")
        remote.approve(signal_id=signal_to_approve, execution_name=execution_name, project=project, domain=domain)
        print(f"Signal '{signal_to_approve}' approved.")
    else:
        print(f"Signal '{signal_to_approve}' not found or not pending.")

    # Example: Set an input for a signal named "user_data_input"
    signal_to_set_input = "user_data_input"
    if any(s.id.signal_id == signal_to_set_input for s in signals):
        print(f"Setting input for signal '{signal_to_set_input}'...")
        remote.set_input(
            signal_id=signal_to_set_input,
            execution_name=execution_name,
            value="some_user_provided_string",
            python_type=str,
            project=project,
            domain=domain
        )
        print(f"Input set for signal '{signal_to_set_input}'.")
    else:
        print(f"Signal '{signal_to_set_input}' not found or not pending.")
else:
    print("No pending signals found for this execution.")
```

## Navigating Execution Details

Flyte executions are hierarchical. A workflow execution consists of node executions, which in turn can contain task executions, sub-workflow executions, or even other node executions (for dynamic workflows or array nodes).

*   **`FlyteWorkflowExecution.node_executions`**: A dictionary mapping node IDs to `FlyteNodeExecution` objects.
*   **`FlyteNodeExecution.task_executions`**: A list of `FlyteTaskExecution` objects if the node executed a task.
*   **`FlyteNodeExecution.workflow_executions`**: A list of `FlyteWorkflowExecution` objects if the node launched a sub-workflow via a launch plan.
*   **`FlyteNodeExecution.subworkflow_node_executions`**: A dictionary mapping node IDs to `FlyteNodeExecution` objects if the node represents a static or dynamic sub-workflow.

To access these nested details, ensure you call `sync(sync_nodes=True)` on the `FlyteWorkflowExecution` object.

```python
# Assuming 'execution' is a FlyteWorkflowExecution object, synced with sync_nodes=True
# execution = remote.fetch_execution(project="...", domain="...", name="...").sync(sync_nodes=True)

print(f"Workflow Execution: {execution.id.name}")

for node_id, node_exec in execution.node_executions.items():
    print(f"  Node Execution: {node_id}, Phase: {node_exec.closure.phase}")

    if node_exec.task_executions:
        for task_exec in node_exec.task_executions:
            print(f"    Task Execution: {task_exec.id.retry_attempt}, Phase: {task_exec.closure.phase}")
            if task_exec.is_done and task_exec.is_successful:
                print(f"      Task Outputs: {task_exec.outputs.to_dict()}")

    if node_exec.workflow_executions:
        for wf_exec in node_exec.workflow_executions:
            print(f"    Sub-Workflow Execution: {wf_exec.id.name}, Phase: {wf_exec.closure.phase}")

    if node_exec.subworkflow_node_executions:
        print(f"    Sub-workflow nodes within {node_id}:")
        for sub_node_id, sub_node_exec in node_exec.subworkflow_node_executions.items():
            print(f"      Sub-Node Execution: {sub_node_id}, Phase: {sub_node_exec.closure.phase}")
```

## Visualizing and Analyzing Executions

Flyte provides tools to visualize executions in the Flyte UI and retrieve detailed metrics.

### Generating Console URLs

Generate direct links to the Flyte UI for quick visualization of executions or registered entities.

```python
from flytekit.remote.remote import FlyteRemote

remote = FlyteRemote.auto()

# Assuming 'execution' is a FlyteWorkflowExecution object
# execution = remote.fetch_execution(project="...", domain="...", name="...")

console_url = remote.generate_console_url(execution)
print(f"View execution in Flyte UI: {console_url}")

# You can also generate URLs for registered entities
# task = remote.fetch_task(project="flytesnacks", domain="development", name="my_task", version="abcdef123")
# task_url = remote.generate_console_url(task)
# print(f"View task in Flyte UI: {task_url}")
```

### Retrieving Execution Metrics

Access detailed operational metrics for an execution, including timing information for all its components. The `FlyteExecutionSpan` object provides methods to `explain()` the hierarchy of operations or `dump()` the raw data.

```python
from flytekit.remote.remote import FlyteRemote

remote = FlyteRemote.auto()

# Assuming 'execution' is a FlyteWorkflowExecution object
# execution = remote.fetch_execution(project="...", domain="...", name="...")

metrics_span = remote.get_execution_metrics(execution.id)

print("Execution Metrics (Explanation):")
metrics_span.explain()

print("\nExecution Metrics (Raw Dump):")
metrics_span.dump()
```

## Advanced Management: Backfills

The `launch_backfill` method allows you to create and execute a new workflow that reruns a specified launch plan for a range of historical dates. This is useful for data reprocessing or historical analysis.

```python
from flytekit.remote.remote import FlyteRemote
from datetime import datetime, timedelta
from flytekit.models.core.workflow import WorkflowFailurePolicy

remote = FlyteRemote.auto()

project = "flytesnacks"
domain = "development"
launchplan_name = "my_daily_report_lp"

# Define the date range for the backfill
from_date = datetime(2023, 1, 1)
to_date = datetime(2023, 1, 5)

# Perform a dry run to see the generated workflow without registering or executing
print("Performing dry run for backfill...")
backfill_workflow_dry_run = remote.launch_backfill(
    project=project,
    domain=domain,
    from_date=from_date,
    to_date=to_date,
    launchplan=launchplan_name,
    dry_run=True,
    parallel=True, # Run each daily instance in parallel
    failure_policy=WorkflowFailurePolicy.FAIL_FAST # Stop immediately on any failure
)
print(f"Dry run generated workflow: {backfill_workflow_dry_run.name}")

# Register and execute the backfill workflow
print("\nLaunching backfill execution...")
backfill_execution = remote.launch_backfill(
    project=project,
    domain=domain,
    from_date=from_date,
    to_date=to_date,
    launchplan=launchplan_name,
    execute=True, # Default, but explicit for clarity
    parallel=True,
    failure_policy=WorkflowFailurePolicy.FAIL_FAST,
    execution_name=f"backfill-{launchplan_name}-{from_date.strftime('%Y%m%d')}-{to_date.strftime('%Y%m%d')}"
)
print(f"Backfill execution launched: {backfill_execution.id.name}")
print(f"Console URL: {remote.generate_console_url(backfill_execution)}")

# Optionally wait for the backfill to complete
# backfill_execution.wait()
```

## Important Considerations

*   **`FlyteRemote` Initialization**: Always initialize `FlyteRemote` with appropriate configuration, including `default_project` and `default_domain`, to simplify subsequent calls.
*   **Interactive Mode**: When running in an interactive environment (like Jupyter notebooks), `FlyteRemote` can automatically detect and enable interactive mode, which affects how tasks and workflows are versioned and registered (e.g., by pickling the entity).
*   **Error Handling**: Be prepared to handle exceptions such as `FlyteAssertion` (for invalid state access), `FlyteTimeout` (when waiting for executions), and `FlyteEntityNotExistException` (when fetching non-existent entities).
*   **Data Access**: The `download` method on `FlyteRemote` allows you to download outputs (files, directories, structured datasets) from completed executions to a local path.
    ```python
    # Assuming 'completed_execution' is a FlyteWorkflowExecution object with outputs
    # completed_execution = remote.fetch_execution(...).wait()
    if completed_execution.is_successful:
        remote.download(completed_execution.outputs, download_to="/tmp/my_execution_outputs", recursive=True)
        print("Outputs downloaded to /tmp/my_execution_outputs")
    ```
<!--
key: summary_monitoring_and_managing_executions_808e28cb-6220-4d71-8d66-47ddb9337002
type: summary_end

-->