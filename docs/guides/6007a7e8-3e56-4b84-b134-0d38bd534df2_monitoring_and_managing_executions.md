
<!--
help_text: ''
key: summary_monitoring_and_managing_executions_68e3439a-248b-40be-bf7d-b1c19e065e40
modules:
- flytekit.clients.friendly.SynchronousFlyteClient
- flytekit.remote.remote.FlyteRemote
- flytekit.remote.executions.FlyteWorkflowExecution
- flytekit.remote.executions.FlyteNodeExecution
- flytekit.remote.executions.FlyteTaskExecution
questions_to_answer: []
type: summary

-->
Executions in Flyte represent instances of tasks, workflows, or launch plans running on the platform. Understanding how to monitor their progress, retrieve results, and manage their lifecycle is crucial for building robust data and ML pipelines. Flyte provides a hierarchical view of executions: a **Workflow Execution** can contain multiple **Node Executions**, and each Node Execution can, in turn, contain one or more **Task Executions** (especially in cases like retries or map tasks).

## Monitoring Executions

Monitoring allows you to observe the state, progress, and outcomes of your running and completed processes.

### Retrieving Execution Details

To inspect a specific execution, you can fetch its details using its unique identifier.

The `FlyteRemote` client's `fetch_execution` method retrieves a `FlyteWorkflowExecution` object, which provides comprehensive information about a workflow run.

```python
from flytekit.remote import FlyteRemote
from flytekit.models.core.identifier import WorkflowExecutionIdentifier

# Initialize FlyteRemote (assuming configuration is set up)
remote = FlyteRemote.auto()

# Define the execution identifier
execution_id = WorkflowExecutionIdentifier(
    project="flytesnacks",
    domain="development",
    name="f123456789abcdef" # This is the unique execution name/ID
)

# Fetch the workflow execution
workflow_execution = remote.fetch_execution(
    project=execution_id.project,
    domain=execution_id.domain,
    name=execution_id.name
)

print(f"Execution ID: {workflow_execution.id.name}")
print(f"Current Phase: {workflow_execution.closure.phase}")
```

You can retrieve details for individual node and task executions if you have their respective identifiers. The `SynchronousFlyteClient` offers direct methods for this: `get_node_execution` and `get_task_execution`.

### Listing Executions

To get an overview of multiple executions, you can list them, often with pagination and filtering.

The `FlyteRemote` client provides `recent_executions` to quickly get a list of the most recent workflow runs in a given project and domain.

```python
from flytekit.remote import FlyteRemote
from flytekit.models.admin.common import Sort, SortOrdering
from flytekit.models.filters import Filter
from flytekit.models.core.execution import WorkflowExecutionPhase

remote = FlyteRemote.auto()

# List recent executions, sorted by creation time in descending order
recent_executions = remote.recent_executions(
    project="flytesnacks",
    domain="development",
    limit=10,
)

for exec in recent_executions:
    print(f"Execution: {exec.id.name}, Phase: {exec.closure.phase}")

# You can also apply filters, for example, to list only failed executions
failed_executions = remote.recent_executions(
    project="flytesnacks",
    domain="development",
    limit=5,
    filters=[Filter.from_python_std(f"eq(phase,{WorkflowExecutionPhase.FAILED})")]
)
```

For more granular control, the `SynchronousFlyteClient` offers paginated listing methods:
- `list_executions_paginated`: Lists workflow executions.
- `list_node_executions`: Lists node executions within a specific workflow execution.
- `list_task_executions_paginated`: Lists task executions within a specific node execution.

These methods support `limit`, `token` (for pagination), `filters`, and `sort_by` parameters, enabling efficient querying of large datasets.

### Accessing Inputs and Outputs

Execution inputs and outputs are crucial for debugging and understanding results.

After fetching an execution, its inputs and outputs can be accessed directly from the `FlyteWorkflowExecution`, `FlyteNodeExecution`, or `FlyteTaskExecution` objects.

```python
# Assuming workflow_execution is already fetched
if workflow_execution.inputs:
    print("Execution Inputs:")
    for k, v in workflow_execution.inputs.items():
        print(f"  {k}: {v}")

if workflow_execution.is_successful and workflow_execution.outputs:
    print("Execution Outputs:")
    for k, v in workflow_execution.outputs.items():
        print(f"  {k}: {v}")
```

For large inputs or outputs, Flyte stores them in a blob storage (e.g., S3, GCS). The `SynchronousFlyteClient`'s `get_execution_data`, `get_node_execution_data`, and `get_task_execution_data` methods provide signed URLs to these blobs. The `FlyteRemote` client's `download` method simplifies retrieving these artifacts to your local machine.

```python
# Download all outputs of a successful workflow execution
if workflow_execution.is_successful:
    remote.download(workflow_execution.outputs, download_to="./execution_outputs")
    print("Outputs downloaded to ./execution_outputs")
```

### Checking Execution Status

The `is_done` and `is_successful` properties on execution objects (`FlyteWorkflowExecution`, `FlyteNodeExecution`, `FlyteTaskExecution`) indicate their completion status. If an execution fails, the `error` property provides details about the failure.

```python
if workflow_execution.is_done:
    if workflow_execution.is_successful:
        print("Workflow execution succeeded.")
    else:
        print(f"Workflow execution failed with error: {workflow_execution.error.message}")
else:
    print("Workflow execution is still running.")
```

### Waiting for Completion

For programmatic control flows, you often need to wait for an execution to complete before proceeding. The `wait` method on `FlyteWorkflowExecution` is a blocking call that polls the execution status until it reaches a terminal state.

```python
from datetime import timedelta

# Wait for the execution to complete, with a timeout of 5 minutes
completed_execution = workflow_execution.wait(timeout=timedelta(minutes=5), poll_interval=timedelta(seconds=10))

if completed_execution.is_successful:
    print("Execution finished successfully!")
else:
    print(f"Execution failed: {completed_execution.error.message}")
```

The `sync_execution` method on `FlyteRemote` updates the local execution object with the latest state from the backend, including details of underlying node and task executions if `sync_nodes=True` is specified. This is useful for getting granular progress updates without waiting for the entire workflow to finish.

### Retrieving Execution Metrics

For deeper insights into execution performance and resource utilization, you can retrieve metrics. The `get_execution_metrics` method on `FlyteRemote` provides detailed span information for a workflow execution.

```python
from flytekit.remote import FlyteRemote
from flytekit.models.core.identifier import WorkflowExecutionIdentifier

remote = FlyteRemote.auto()

execution_id = WorkflowExecutionIdentifier(
    project="flytesnacks",
    domain="development",
    name="f123456789abcdef"
)

metrics_span = remote.get_execution_metrics(execution_id, depth=5)
print(f"Execution Span Name: {metrics_span.name}")
print(f"Execution Span Duration: {metrics_span.duration}")
# Further explore metrics_span.spans for detailed node/task metrics
```

### Viewing in the Console

Flyte provides a web-based console for visual monitoring and debugging. The `generate_console_url` method on `FlyteRemote` creates a direct link to an execution or entity in the console.

```python
console_url = remote.generate_console_url(workflow_execution)
print(f"View execution in console: {console_url}")
```

This URL can be opened in a web browser to inspect logs, visualize the workflow graph, and examine inputs/outputs.

## Managing Executions

Beyond monitoring, Flyte offers capabilities to control the lifecycle of your executions.

### Launching Executions

Executions can be launched from registered tasks, workflows, or launch plans. The `execute` method on `FlyteRemote` is the primary entry point for initiating runs. It intelligently handles whether the entity needs to be registered first (for local Python objects) or if it's already a remote entity.

```python
from flytekit import task, workflow
from flytekit.remote import FlyteRemote

remote = FlyteRemote.auto()

@task
def my_task(a: int, b: int) -> int:
    return a + b

# Execute a local task (will be registered if not already)
execution = remote.execute(
    my_task,
    inputs={"a": 10, "b": 20},
    project="flytesnacks",
    domain="development",
    execution_name_prefix="my_task_run"
)
print(f"Launched task execution: {execution.id.name}")

@workflow
def my_workflow(x: int) -> int:
    t1_output = my_task(a=x, b=5)
    return my_task(a=t1_output, b=10)

# Execute a local workflow
wf_execution = remote.execute(
    my_workflow,
    inputs={"x": 100},
    project="flytesnacks",
    domain="development",
    execution_name_prefix="my_workflow_run",
    wait=True # Wait for completion
)
print(f"Launched workflow execution: {wf_execution.id.name}, Status: {wf_execution.closure.phase}")
```

When executing, you can specify:
-   `project` and `domain`: The target project and domain for the execution.
-   `execution_name` or `execution_name_prefix`: To provide a custom name or prefix for the execution, aiding in identification.
-   `options`: To configure execution-specific settings like `overwrite_cache`, `interruptible`, `envs`, `tags`, `cluster_pool`, or `execution_cluster_label`.

### Relaunching and Recovering Executions

Flyte supports restarting executions from their last known state, which is particularly useful for recovering from failures or re-running specific parts of a workflow.

-   `relaunch_execution`: Creates a new execution that re-runs the entire workflow from scratch, using the same inputs as a previous execution.
-   `recover_execution`: Creates a new execution that attempts to resume from the last known failure point of a previous execution, reusing successful outputs where possible.

Both methods are available on the `SynchronousFlyteClient`.

```python
from flytekit.remote import FlyteRemote
from flytekit.models.core.identifier import WorkflowExecutionIdentifier

remote = FlyteRemote.auto()

# Assuming 'failed_execution_id' is the ID of a previously failed execution
failed_execution_id = WorkflowExecutionIdentifier(
    project="flytesnacks",
    domain="development",
    name="failed_run_xyz"
)

# Recover the failed execution
recovered_execution_id = remote.client.recover_execution(
    id=failed_execution_id,
    name="recovered_run_abc"
)
print(f"Recovered execution launched with ID: {recovered_execution_id.name}")

# Relaunch the execution
relaunched_execution_id = remote.client.relaunch_execution(
    id=failed_execution_id,
    name="relaunched_run_def"
)
print(f"Relaunched execution launched with ID: {relaunched_execution_id.name}")
```

### Terminating Executions

You can stop a running workflow execution using the `terminate` method on `FlyteRemote`. This sends a signal to the Flyte backend to halt the execution.

```python
# Assuming 'running_execution' is a FlyteWorkflowExecution object that is currently running
remote.terminate(running_execution, cause="Manual termination for testing.")
print(f"Termination request sent for execution: {running_execution.id.name}")
```

### Interacting with Signals

Flyte's human-in-the-loop capabilities allow workflows to pause and wait for external input or approval via "signals". The `FlyteRemote` client provides methods to interact with these signals.

-   `list_signals`: Retrieves pending signals for a given execution.
-   `approve`: Approves a boolean signal.
-   `reject`: Rejects a boolean signal.
-   `set_input` / `set_signal`: Sets the value for any type of signal.

```python
from flytekit.remote import FlyteRemote

remote = FlyteRemote.auto()

execution_name = "my_workflow_with_signal_run"
signal_id = "user_approval_signal"

# List signals for an execution
signals = remote.list_signals(execution_name=execution_name, project="flytesnacks", domain="development")
for signal in signals:
    print(f"Found signal: {signal.id.signal_id} with type {signal.type.simple}")

# Approve a boolean signal
remote.approve(signal_id=signal_id, execution_name=execution_name, project="flytesnacks", domain="development")
print(f"Signal '{signal_id}' approved for execution '{execution_name}'.")

# Set a custom input for a signal (e.g., a string)
remote.set_input(
    signal_id="data_input_signal",
    execution_name=execution_name,
    value="some_data_from_user",
    python_type=str,
    project="flytesnacks",
    domain="development"
)
print(f"Signal 'data_input_signal' set for execution '{execution_name}'.")
```

### Backfilling Workflows

Backfilling refers to running a workflow for a range of past dates, typically for historical data processing or re-computation. The `launch_backfill` method on `FlyteRemote` automates the creation and execution of a workflow designed for this purpose.

```python
from flytekit.remote import FlyteRemote
from datetime import datetime, timedelta

remote = FlyteRemote.auto()

# Launch a backfill for a specific launch plan over a date range
backfill_execution = remote.launch_backfill(
    project="flytesnacks",
    domain="development",
    launchplan="my_daily_report_lp",
    from_date=datetime(2023, 1, 1),
    to_date=datetime(2023, 1, 5),
    parallel=True, # Run daily instances in parallel
    execute=True,
    execution_name="daily_report_backfill_jan_2023"
)
print(f"Backfill workflow launched: {backfill_execution.id.name}")
```

The `launch_backfill` method can generate a sequential or parallel backfill workflow and allows for dry runs to inspect the generated workflow before execution.

### Downloading Execution Data

The `download` method on `FlyteRemote` facilitates retrieving execution inputs, outputs, or other artifacts (like deck HTML files) to your local file system. This is particularly useful for inspecting large data outputs or debugging.

```python
# Assuming 'workflow_execution' is a completed execution object
if workflow_execution.is_successful:
    # Download all outputs to a local directory
    remote.download(workflow_execution.outputs, download_to="./workflow_outputs", recursive=True)
    print("Workflow outputs downloaded.")
```

The `download` method handles `Literal`, `LiteralMap`, and `LiteralsResolver` objects, recursively downloading file-like objects (e.g., `FlyteFile`, `FlyteDir`, `StructuredDataset`) if `recursive=True` is specified.

## Best Practices and Considerations

-   **Error Handling**: Always include robust error handling when interacting with the Flyte backend. Operations can fail due to network issues, invalid inputs, or backend errors.
-   **Pagination**: When listing entities, be mindful of paginated APIs. Use the `token` parameter to fetch subsequent pages for complete results.
-   **Resource Management**: Terminate long-running or erroneous executions to free up cluster resources.
-   **Idempotency**: When launching executions, consider using `execution_name` for idempotency, especially in automated systems, to prevent duplicate runs.
-   **Security**: Ensure your `FlyteRemote` client is configured with appropriate authentication and SSL settings for production environments. Avoid `insecure=True` unless absolutely necessary for development.
-   **Performance**: For large-scale data transfers, leverage Flyte's native data proxy service and signed URLs rather than attempting to download entire datasets through the client directly. The `download` method handles this efficiently.
-   **Interactive Mode**: The `interactive_mode_enabled` flag in `FlyteRemote` is useful for development in environments like Jupyter notebooks, enabling automatic pickling and registration. For production deployments, explicit registration via `register_task`, `register_workflow`, or `register_launch_plan` is recommended.
-   **Version Management**: When registering entities, specify a meaningful `version` to ensure reproducibility and proper lifecycle management. Auto-generated versions are suitable for interactive development but less so for production.
<!--
key: summary_monitoring_and_managing_executions_68e3439a-248b-40be-bf7d-b1c19e065e40
type: summary_end

-->
<!--
code_unit: flytekit.clients.friendly.SynchronousFlyteClient.get_execution
code_unit_type: class
help_text: ''
key: example_7897073f-37cc-4a68-9f30-5757597a479b
type: example

-->
<!--
code_unit: flytekit.clients.friendly.SynchronousFlyteClient.list_executions_paginated
code_unit_type: class
help_text: ''
key: example_a2f4ea53-9b48-4d19-95b4-768816bb217e
type: example

-->
<!--
code_unit: flytekit.clients.friendly.SynchronousFlyteClient.terminate_execution
code_unit_type: class
help_text: ''
key: example_5e0f1de1-1123-49d9-9775-bda6a562c5f2
type: example

-->
<!--
code_unit: flytekit.clients.friendly.SynchronousFlyteClient.relaunch_execution
code_unit_type: class
help_text: ''
key: example_fe2b13d6-cebc-49ca-b1a9-8348361bc6eb
type: example

-->
<!--
code_unit: flytekit.clients.friendly.SynchronousFlyteClient.recover_execution
code_unit_type: class
help_text: ''
key: example_eb7d2c4d-f9e4-41af-ab9a-b06fbda5e715
type: example

-->
<!--
code_unit: flytekit.clients.friendly.SynchronousFlyteClient.get_node_execution
code_unit_type: class
help_text: ''
key: example_83ed7c4b-627d-4e51-8952-5157215a377b
type: example

-->
<!--
code_unit: flytekit.clients.friendly.SynchronousFlyteClient.list_node_executions
code_unit_type: class
help_text: ''
key: example_2dce4bc2-1f62-4d01-aee1-7233b4a77de8
type: example

-->
<!--
code_unit: flytekit.clients.friendly.SynchronousFlyteClient.get_task_execution
code_unit_type: class
help_text: ''
key: example_3fff4ce1-2676-4517-97fd-22f10ebaac13
type: example

-->
<!--
code_unit: flytekit.clients.friendly.SynchronousFlyteClient.list_task_executions_paginated
code_unit_type: class
help_text: ''
key: example_fca3fbdd-b42e-49f2-8dd8-da5327d1c7d4
type: example

-->
<!--
code_unit: flytekit.remote.remote.FlyteRemote.fetch_execution
code_unit_type: class
help_text: ''
key: example_38a2f3a8-e3ed-4ea8-ba5a-68ade8d06928
type: example

-->
<!--
code_unit: flytekit.remote.remote.FlyteRemote.wait
code_unit_type: class
help_text: ''
key: example_c957460d-8d18-45cf-ae1b-000089733046
type: example

-->
<!--
code_unit: flytekit.remote.remote.FlyteRemote.sync
code_unit_type: class
help_text: ''
key: example_e1116050-d482-46c7-b33c-158b0b6b749a
type: example

-->
<!--
code_unit: flytekit.remote.remote.FlyteRemote.terminate
code_unit_type: class
help_text: ''
key: example_5d1c081c-e26d-4ed0-9caf-2059067eceaf
type: example

-->