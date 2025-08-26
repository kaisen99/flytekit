
<!--
help_text: ''
key: summary_interacting_with_signals_1dad247a-c818-45c1-b4f9-750606467db1
modules:
- flytekit.clients.friendly.SynchronousFlyteClient
- flytekit.clients.raw.RawSynchronousFlyteClient
- flytekit.remote.remote.FlyteRemote
questions_to_answer: []
type: summary

-->
Interacting with Signals

Signals provide a mechanism for external systems or human users to interact with and influence the execution of a running workflow. They enable workflows to pause, await external input, or receive approval/rejection decisions before proceeding. This capability is crucial for building interactive, human-in-the-loop, or event-driven workflows.

### Accessing Signal Capabilities

The primary interface for interacting with signals is the `FlyteRemote` object. This object provides high-level methods to list, approve, reject, or set the value of signals associated with a workflow execution. Underneath, `FlyteRemote` leverages the `SynchronousFlyteClient` to make direct gRPC calls to the Flyte Admin service.

To begin, initialize a `FlyteRemote` instance:

```python
from flytekit.remote import FlyteRemote, Config

# Configure FlyteRemote with your Flyte deployment details
# For a local sandbox, you might use:
remote = FlyteRemote.for_sandbox(
    default_project="flytesnacks",
    default_domain="development",
)

# For a remote deployment:
# remote = FlyteRemote.for_endpoint(
#     endpoint="your.flyte.admin.endpoint:port",
#     insecure=True, # Set to False for SSL/TLS enabled deployments
#     default_project="your_project",
#     default_domain="your_domain",
# )
```

All signal operations require specifying the `execution_name` of the target workflow execution, along with its `project` and `domain`. If `default_project` and `default_domain` are set during `FlyteRemote` initialization, these can be omitted from method calls.

### Listing Signals

To inspect the signals currently pending for a workflow execution, use the `list_signals` method. This is useful for understanding which parts of a workflow are awaiting external input.

```python
from flytekit.models.filters import Filter
from flytekit.models.common import Sort, SortOrdering

# Assuming 'my_workflow_execution_name' is the name of your running execution
execution_name = "my_workflow_execution_name"
project = "flytesnacks" # Or use remote.default_project
domain = "development" # Or use remote.default_domain

# List all signals for a specific execution
signals = remote.list_signals(
    execution_name=execution_name,
    project=project,
    domain=domain,
    limit=10, # Optional: limit the number of results
    # filters=[Filter.from_python_std("eq(state,PENDING)")], # Optional: filter by state
)

for signal in signals:
    print(f"Signal ID: {signal.id.signal_id}, Type: {signal.type.simple}, State: {signal.state}")
```

The `list_signals` method returns a list of `Signal` objects, each containing details like its unique identifier (`signal_id`), expected type, and current state.

### Approving a Signal

For boolean signals, typically used for human approval steps, the `approve` method sets the signal's value to `True`. This allows the workflow to proceed past the approval gate.

```python
signal_id_to_approve = "my_approval_signal"
execution_name = "my_workflow_execution_name"
project = "flytesnacks"
domain = "development"

try:
    remote.approve(
        signal_id=signal_id_to_approve,
        execution_name=execution_name,
        project=project,
        domain=domain,
    )
    print(f"Signal '{signal_id_to_approve}' for execution '{execution_name}' approved successfully.")
except Exception as e:
    print(f"Failed to approve signal: {e}")
```

### Rejecting a Signal

Conversely, to reject a boolean signal, setting its value to `False`, use the `reject` method. This typically causes the workflow to take an alternative path or fail, depending on its definition.

```python
signal_id_to_reject = "my_approval_signal"
execution_name = "my_workflow_execution_name"
project = "flytesnacks"
domain = "development"

try:
    remote.reject(
        signal_id=signal_id_to_reject,
        execution_name=execution_name,
        project=project,
        domain=domain,
    )
    print(f"Signal '{signal_id_to_reject}' for execution '{execution_name}' rejected successfully.")
except Exception as e:
    print(f"Failed to reject signal: {e}")
```

### Setting a Signal with Custom Input

Workflows can also wait for arbitrary data inputs. The `set_input` method (which is an alias for `set_signal`) allows you to provide a value of any supported Flyte type to a signal.

When providing the `value`, you can either pass a `Literal` object directly or a Python native value. If a Python native value is provided, the system attempts to convert it to a `Literal` based on the `python_type` or `literal_type` hints. It is recommended to provide `python_type` for complex types (e.g., `typing.List[int]`, `typing.Dict[str, str]`) to ensure correct conversion.

```python
from flytekit.types.file import FlyteFile
from flytekit.types.structured.structured_dataset import StructuredDataset

# Example 1: Setting a signal with an integer value
signal_id_int = "user_count_input"
execution_name = "my_workflow_execution_name"
project = "flytesnacks"
domain = "development"
int_value = 123

try:
    remote.set_input(
        signal_id=signal_id_int,
        execution_name=execution_name,
        project=project,
        domain=domain,
        value=int_value,
        python_type=int, # Explicitly provide the Python type
    )
    print(f"Signal '{signal_id_int}' set with value {int_value}.")
except Exception as e:
    print(f"Failed to set signal with int value: {e}")

# Example 2: Setting a signal with a string value
signal_id_str = "config_path"
str_value = "s3://my-bucket/configs/prod.yaml"

try:
    remote.set_input(
        signal_id=signal_id_str,
        execution_name=execution_name,
        project=project,
        domain=domain,
        value=str_value,
        python_type=str,
    )
    print(f"Signal '{signal_id_str}' set with value '{str_value}'.")
except Exception as e:
    print(f"Failed to set signal with string value: {e}")

# Example 3: Setting a signal with a FlyteFile (requires local file to upload)
signal_id_file = "data_file"
local_file_path = "/tmp/my_data.csv"
# Create a dummy file for demonstration
with open(local_file_path, "w") as f:
    f.write("col1,col2\n1,a\n2,b\n")

# When providing a FlyteFile, the remote client handles the upload
file_value = FlyteFile(local_file_path)

try:
    remote.set_input(
        signal_id=signal_id_file,
        execution_name=execution_name,
        project=project,
        domain=domain,
        value=file_value,
        python_type=FlyteFile,
    )
    print(f"Signal '{signal_id_file}' set with file '{local_file_path}'.")
except Exception as e:
    print(f"Failed to set signal with file value: {e}")
```

### Common Use Cases

*   **Human Approval Gates:** Workflows can pause at critical junctures (e.g., before deploying to production) and wait for a human to `approve` or `reject` the continuation.
*   **Dynamic Configuration Injection:** A running workflow might require configuration parameters that are only known at runtime or provided by an external system. Signals allow these parameters to be injected.
*   **External Event Triggers:** While Flyte has dedicated event triggers, simple external events can also be modeled as signals, allowing a workflow to proceed only after a specific external condition is met.
*   **Interactive Data Input:** For workflows that require user-provided data during execution, signals offer a way to supply this data programmatically.

### Important Considerations

*   **Signal ID Matching:** The `signal_id` provided when setting a signal must exactly match the ID that the workflow is waiting for. This ID is typically defined within the workflow's `wait_for_input` or `approve` node.
*   **Execution Context:** Signals are strictly tied to a specific workflow execution. Ensure you provide the correct `project`, `domain`, and `execution_name` to target the intended running instance.
*   **Type Consistency:** The type of the `value` provided to `set_input` must be compatible with the type expected by the signal in the workflow definition. Mismatches will result in runtime errors.
*   **Idempotency:** Setting a signal multiple times with the same value is generally idempotent, meaning subsequent identical calls will not change the state or cause errors once the signal has been fulfilled.
*   **Error Handling:** Operations on signals can fail due to network issues, incorrect identifiers, or invalid input types. Implement appropriate error handling (e.g., `try-except` blocks) to manage these scenarios.
*   **Workflow Definition:** The ability to interact with signals depends on the workflow being designed to explicitly `wait_for_input` or `approve` at specific nodes. Without these nodes, there are no signals to interact with.
<!--
key: summary_interacting_with_signals_1dad247a-c818-45c1-b4f9-750606467db1
type: summary_end

-->
<!--
code_unit: flytekit.clients.friendly.SynchronousFlyteClient.set_signal
code_unit_type: class
help_text: ''
key: example_6c6060b0-df9d-4ff0-abe9-ddc639ae1728
type: example

-->
<!--
code_unit: flytekit.clients.friendly.SynchronousFlyteClient.list_signals
code_unit_type: class
help_text: ''
key: example_6fb9376a-004b-4491-9dff-3088a78a4011
type: example

-->
<!--
code_unit: flytekit.remote.remote.FlyteRemote.list_signals
code_unit_type: class
help_text: ''
key: example_37bbcf34-b3b3-41fe-b12f-48ac51818ab2
type: example

-->
<!--
code_unit: flytekit.remote.remote.FlyteRemote.approve
code_unit_type: class
help_text: ''
key: example_bda5272b-c0d0-4e24-90c4-1b5909fa07c8
type: example

-->
<!--
code_unit: flytekit.remote.remote.FlyteRemote.reject
code_unit_type: class
help_text: ''
key: example_821b16cb-f226-4520-83cc-d743fd20c8d4
type: example

-->
<!--
code_unit: flytekit.remote.remote.FlyteRemote.set_input
code_unit_type: class
help_text: ''
key: example_5c9d63bc-cf92-4d96-a10f-1e5f61650abb
type: example

-->
<!--
code_unit: flytekit.remote.remote.FlyteRemote.set_signal
code_unit_type: class
help_text: ''
key: example_a0a1b821-94a2-4617-a899-14c7ae7a7dca
type: example

-->