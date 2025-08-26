
<!--
help_text: ''
key: summary_remote_entity_representation_f458ac4d-deec-42b6-8bb6-5d01b47ef7be
modules:
- flytekit.remote.remote_callable.RemoteEntity
- flytekit.remote.entities.FlyteTask
- flytekit.remote.entities.FlyteWorkflow
- flytekit.remote.entities.FlyteLaunchPlan
- flytekit.remote.entities.FlyteNode
- flytekit.remote.entities.FlyteTaskNode
- flytekit.remote.entities.FlyteWorkflowNode
- flytekit.remote.entities.FlyteBranchNode
- flytekit.remote.entities.FlyteArrayNode
- flytekit.remote.entities.FlyteGateNode
- flytekit.remote.executions.RemoteExecutionBase
- flytekit.remote.executions.FlyteWorkflowExecution
- flytekit.remote.executions.FlyteNodeExecution
- flytekit.remote.executions.FlyteTaskExecution
- flytekit.remote.interface.TypedInterface
- flytekit.remote.lazy_entity.LazyEntity
- flytekit.remote.metrics.FlyteExecutionSpan
- flytekit.interfaces.cli_identifiers.Identifier
- flytekit.interfaces.cli_identifiers.WorkflowExecutionIdentifier
- flytekit.interfaces.cli_identifiers.TaskExecutionIdentifier
questions_to_answer: []
type: summary

-->
# Remote Entity Representation

Remote Entity Representation provides a mechanism to interact with Flyte entities (Tasks, Workflows, Launch Plans, and their Executions) that reside on a remote Flyte backend. This allows developers to reference, inspect, and operate on these server-side objects directly from their local Python environment, treating them as first-class Python objects. This capability is fundamental for building applications that interact with deployed Flyte resources, enabling operations like triggering executions, monitoring progress, and retrieving results.

## Core Remote Entities

The foundation of remote entity representation is the `RemoteEntity` abstract base class. It defines the common interface for all entities that can be referenced and interacted with remotely.

### `RemoteEntity`

The `RemoteEntity` class serves as the base for all remote Flyte objects. It provides core properties and behaviors for interacting with entities that are not locally defined Python functions or objects, but rather references to objects managed by the Flyte control plane.

**Key Properties:**

*   `name` (abstract): The human-readable name of the entity.
*   `id` (abstract): A unique `Identifier` for the entity, encompassing its project, domain, name, and version.
*   `python_interface`: An optional `Interface` object, providing type hints for inputs and outputs when the entity's Python interface is known.

**Core Capabilities:**

*   **Node Construction (`construct_node_metadata`)**: Used internally to generate metadata when the remote entity is incorporated as a node within a workflow definition.
*   **Compilation (`compile`)**: When a remote entity is invoked within a workflow definition (i.e., during compilation), this method is called to create and link a node representing the remote entity. This is how a reference to a remote task or workflow becomes a step in a new workflow.
*   **Invocation (`__call__`)**: The `__call__` method enables remote entities to be invoked like regular Python functions. Its behavior depends on the current Flyte context:
    *   **Compilation Mode**: If the current context is in compilation mode (e.g., defining a workflow), calling the remote entity triggers its `compile` method, resulting in the creation of a node within the workflow graph.
    *   **Local Workflow Execution Mode**: When a workflow is being executed locally, calling a remote entity will invoke its `local_execute` method. For remote entities, this typically means they need to be mocked or will raise an assertion error if not handled.
    *   **Plain Execution Mode**: Directly calling `execute` on a `RemoteEntity` that has been fetched from the backend will raise an `AssertionError`. Remote entities are references to server-side objects and cannot be executed locally without explicit mocking.

**Usage Considerations:**

When working with `RemoteEntity` instances, it's crucial to understand that they are pointers to server-side definitions. Direct local execution is not supported. Their primary use is to be referenced within new workflow definitions or to inspect metadata and trigger executions on the remote system.

## Definable Remote Entities

These classes represent the core definable entities within Flyte that can be registered and executed on the platform. They inherit from `RemoteEntity` and provide specific properties and methods relevant to their type.

### `FlyteTask`

Represents a remote Flyte Task. It encapsulates the task's definition as stored on the Flyte backend.

**Key Properties:**

*   `id`: The unique identifier of the task.
*   `type`: The task type (e.g., `python-task`, `container`).
*   `metadata`: Runtime information such as discoverability, timeouts, and retries.
*   `interface`: The strongly typed input and output interface of the task.
*   `container`: If applicable, the container image and command used by the task.
*   `name`: The name of the task.
*   `resource_type`: Always `ResourceType.TASK`.

**Usage:**

A `FlyteTask` object can be fetched from the remote system and then invoked within a local workflow definition to create a node that references this remote task.

```python
# Conceptual example: Assuming 'remote' is an initialized FlyteRemote object
# remote_task = remote.fetch_task(name="my_remote_task", project="flyte-project", domain="development", version="v1")

# In a workflow definition, calling remote_task creates a node:
# @workflow
# def my_workflow():
#     node = remote_task(input_arg="value")
#     # ... further workflow logic
```

### `FlyteWorkflow`

Represents a remote Flyte Workflow. It provides access to the workflow's structure, including its nodes, inputs, and outputs.

**Key Properties:**

*   `id`: The unique identifier of the workflow.
*   `name`: The name of the workflow.
*   `interface`: The strongly typed input and output interface of the workflow.
*   `nodes`: A list of `FlyteNode` objects representing the steps within the workflow.
*   `outputs`: Bindings that define how workflow outputs are constructed from node outputs.
*   `flyte_tasks`: A list of `FlyteTask` objects referenced by this workflow.
*   `flyte_sub_workflows`: A list of `FlyteWorkflow` objects representing sub-workflows.
*   `resource_type`: Always `ResourceType.WORKFLOW`.

**Hydration from Backend:**

*   `promote_from_model`: Used to construct a `FlyteWorkflow` object from its underlying model representation.
*   `promote_from_closure`: A crucial method for reconstructing a `FlyteWorkflow` from a `CompiledWorkflowClosure` received from the control plane. This method recursively promotes sub-workflows and tasks contained within the closure, ensuring a complete local representation of the remote workflow's structure.

**Usage:**

Similar to `FlyteTask`, a `FlyteWorkflow` can be invoked within a local workflow definition to create a node that executes the remote workflow as a sub-workflow.

### `FlyteLaunchPlan`

Represents a remote Flyte Launch Plan. Launch Plans are executable versions of workflows, often configured with default inputs, schedules, and other execution-specific metadata.

**Key Properties:**

*   `id`: The unique identifier of the launch plan.
*   `name`: The name of the launch plan.
*   `workflow_id`: The `Identifier` of the workflow that this launch plan executes.
*   `interface`: The interface of the underlying workflow.
*   `is_scheduled`: Indicates if the launch plan has an active schedule.
*   `resource_type`: Always `ResourceType.LAUNCH_PLAN`.

**Usage:**

`FlyteLaunchPlan` objects are typically used to trigger executions of a workflow with specific configurations or to inspect scheduled executions. They can also be invoked within a workflow definition to create a node that executes the launch plan.

```python
# Conceptual example:
# remote_lp = remote.fetch_launch_plan(name="my_scheduled_lp", project="flyte-project", domain="development", version="v1")

# Check if it's scheduled
# if remote_lp.is_scheduled:
#     print(f"Launch plan {remote_lp.name} is scheduled.")

# Trigger an execution (conceptual, requires FlyteRemote.execute_launch_plan)
# execution = remote.execute_launch_plan(remote_lp, inputs={"param": "value"})
```

## Workflow Structure Nodes

These classes represent the individual components (nodes) within a workflow definition, allowing for the reconstruction and inspection of a remote workflow's graph.

### `FlyteNode`

Represents a node within a remote workflow. A `FlyteNode` acts as a wrapper around the specific type of entity it executes (task, sub-workflow, branch, etc.).

**Key Properties:**

*   `id`: The unique identifier of the node within its parent workflow.
*   `upstream_nodes`: A list of `FlyteNode` objects that this node depends on.
*   `bindings`: Input bindings that connect this node's inputs to outputs of upstream nodes or workflow inputs.
*   `flyte_entity`: A polymorphic property that holds the actual entity executed by this node (e.g., `FlyteTask`, `FlyteWorkflow`, `FlyteLaunchPlan`, `FlyteBranchNode`, `FlyteArrayNode`).

**Node Types:**

`FlyteNode` can encapsulate various types of operations:

*   **`FlyteTaskNode`**: Represents a node that executes a `FlyteTask`. Its `flyte_task` property provides access to the underlying `FlyteTask` object.
*   **`FlyteWorkflowNode`**: Represents a node that executes either a `FlyteWorkflow` (as a sub-workflow) or a `FlyteLaunchPlan`. Its `flyte_workflow` or `flyte_launch_plan` properties provide access to the respective entities.
*   **`FlyteBranchNode`**: Represents conditional logic (`if-else`) within a workflow. It contains an `IfElseBlock` that defines the conditions and the `FlyteNode` to execute for each branch.
*   **`FlyteArrayNode`**: Represents a map task or an array job, allowing a single node to execute multiple instances of an underlying `FlyteNode` in parallel. It wraps a `FlyteNode` and includes properties for parallelism, minimum successes, and success ratio.
*   **`FlyteGateNode`**: Represents a gate in a workflow, used for signaling, sleeping, or approval steps.

**Architectural Significance:**

The `promote_from_model` method in `FlyteNode` is critical for recursively building the entire workflow graph when a `FlyteWorkflow` is promoted from a backend model. It traverses the node definitions, promoting nested tasks, sub-workflows, and branch/array nodes, ensuring that the local `FlyteWorkflow` object accurately reflects the remote definition.

## Remote Executions

These classes provide a way to interact with and monitor the state of running or completed executions on the Flyte backend.

### `RemoteExecutionBase`

The abstract base class for all remote execution objects. It provides common properties for checking execution status and retrieving results.

**Key Properties:**

*   `inputs`: A `LiteralsResolver` object providing access to the execution's input values.
*   `outputs`: A `LiteralsResolver` object providing access to the execution's output values.
    *   **Important**: Outputs are only available when `is_done` is `True` and `error` is `None`. Attempting to access outputs before completion or on a failed execution will raise a `FlyteAssertion` error.
*   `error`: An `ExecutionError` object if the execution failed, otherwise `None`.
*   `is_done`: A boolean indicating whether the execution has reached a terminal state (succeeded, failed, aborted, timed out).

### `FlyteWorkflowExecution`

Represents a remote workflow execution. This object allows for monitoring the overall workflow status and inspecting its constituent node executions.

**Key Properties:**

*   `node_executions`: A dictionary mapping node IDs to `FlyteNodeExecution` objects, providing access to the status and details of individual nodes within the workflow.
*   `flyte_workflow`: The `FlyteWorkflow` object that this execution is an instance of.
*   `execution_url`: A URL to view the execution in the Flyte console.
*   `is_successful`: A boolean indicating if the workflow execution completed successfully.

**Capabilities:**

*   `sync()`: Updates the local `FlyteWorkflowExecution` object with the latest state from the remote backend. This is crucial for monitoring long-running executions.
*   `wait()`: A blocking call that waits for the workflow execution to complete, optionally with a timeout and polling interval. This is useful for scripts that need to wait for results before proceeding.

**Usage:**

```python
# Conceptual example:
# wf_exec = remote.execute_workflow(remote_workflow, inputs={...})
# print(f"Execution URL: {wf_exec.execution_url}")

# Wait for completion
# wf_exec.wait()

# if wf_exec.is_successful:
#     print("Workflow succeeded!")
#     outputs = wf_exec.outputs
#     print(f"Outputs: {outputs.get('output_name')}")
# else:
#     print(f"Workflow failed with error: {wf_exec.error.message}")
```

### `FlyteNodeExecution`

Represents a remote node execution within a workflow execution. It provides details about a specific step in the workflow.

**Key Properties:**

*   `task_executions`: A list of `FlyteTaskExecution` objects if the node executed a task (e.g., for retries or dynamic tasks).
*   `workflow_executions`: A list of `FlyteWorkflowExecution` objects if the node executed a sub-workflow.
*   `subworkflow_node_executions`: A dictionary of `FlyteNodeExecution` objects if the node is a parent of a sub-workflow, allowing traversal into nested workflow structures.
*   `executions`: A convenience property that returns all child executions (tasks or sub-workflows).
*   `interface`: The `TypedInterface` of the task or sub-workflow associated with this node.

**Usage:**

Used to drill down into the details of a workflow execution, inspecting the status, inputs, and outputs of individual nodes.

### `FlyteTaskExecution`

Represents a remote task execution, which is the lowest level of execution detail.

**Key Properties:**

*   `task`: The `FlyteTask` object that this execution is an instance of.

**Usage:**

Provides granular information about a specific task run, including its phase and any errors.

## Utility and Identifier Classes

These classes support the representation and interaction with remote entities.

### `TypedInterface`

Represents the strongly typed input and output interface of a Flyte entity (Task, Workflow, Launch Plan). It provides metadata about the expected types for parameters.

### `LazyEntity`

The `LazyEntity` class provides a mechanism for lazy loading of remote entities. This is a performance optimization that defers the actual fetching of the entity's full definition from the backend until it is explicitly needed (e.g., when one of its properties or methods is accessed).

**Benefits:**

*   **Reduced Network Calls**: Avoids unnecessary API calls when only a reference to an entity is required, but its full details are not immediately used.
*   **Faster Initialization**: Objects can be created quickly without blocking on network I/O.

**Usage:**

A `LazyEntity` is initialized with a `name` and a `getter` callable. The `getter` function is responsible for fetching the actual `RemoteEntity` when `LazyEntity.entity` is accessed or when any attribute/method is called on the `LazyEntity` instance.

```python
# Conceptual example:
# def fetch_my_task():
#     # This function would contain the actual remote call to fetch the task
#     # e.g., return remote.fetch_task(...)
#     pass

# lazy_task = LazyEntity(name="my_task", getter=fetch_my_task)

# At this point, no network call has been made.
# The task is only fetched when an attribute is accessed:
# print(lazy_task.id) # This triggers the fetch_my_task() call
```

### `FlyteExecutionSpan`

Provides a structured representation of execution traces and performance metrics for a Flyte execution. It allows for detailed analysis of the execution's timeline and resource usage.

**Capabilities:**

*   `explain()`: Prints a human-readable summary of the execution span, showing operations, timestamps, and durations.
*   `dump()`: Dumps the aggregated span information in YAML format for programmatic analysis.

### `Identifier`, `WorkflowExecutionIdentifier`, `TaskExecutionIdentifier`

These classes provide standardized, globally unique identifiers for Flyte entities and executions. They are crucial for addressing specific resources on the Flyte backend.

**Structure:**

Identifiers typically include:

*   `project`: The project the entity belongs to.
*   `domain`: The domain within the project.
*   `name`: The name of the entity.
*   `version`: The version of the entity (for definable entities like tasks, workflows, launch plans).

Execution identifiers (`WorkflowExecutionIdentifier`, `TaskExecutionIdentifier`) also include specific details like node IDs and retry attempts to pinpoint a specific run.

**Parsing:**

The `from_python_std` class method allows parsing a string representation of an identifier (e.g., `tsk:project:domain:name:version`) into its structured object form, facilitating easy referencing of remote entities.
<!--
key: summary_remote_entity_representation_f458ac4d-deec-42b6-8bb6-5d01b47ef7be
type: summary_end

-->