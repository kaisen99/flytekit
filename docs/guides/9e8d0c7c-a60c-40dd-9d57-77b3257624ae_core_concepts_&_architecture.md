
<!--
help_text: ''
key: summary_core_concepts_&_architecture_5b61a8d1-5d0a-474c-8e86-8cde61948fbf
modules:
- flytekit.core.base_task
- flytekit.core.python_function_task
- flytekit.core.workflow
- flytekit.core.node
- flytekit.core.promise
- flytekit.core.launch_plan
- flytekit.core.interface
- flytekit.core.context_manager
- flytekit.models.core.identifier
- flytekit.models.core.workflow
- flytekit.models.core.task
- flytekit.models.launch_plan
questions_to_answer: []
type: summary

-->
# Core Concepts & Architecture

Flytekit's core architecture is designed to enable the definition, compilation, and execution of machine learning and data processing workflows. It bridges the gap between Python functions and the Flyte platform's distributed execution engine. This section details the fundamental building blocks and their interrelationships.

## Tasks

Tasks are the atomic units of computation in Flyte. They encapsulate a single, well-defined operation.

### Task Definition and Metadata

The `Task` class serves as the base for all Flyte tasks, representing the platform's `TaskTemplate` definition. It defines the task's name, interface (inputs and outputs), and metadata.

`TaskMetadata` provides crucial configuration for a task's execution behavior. These attributes control aspects like:

*   **Caching (`cache`, `cache_version`, `cache_serialize`, `cache_ignore_input_vars`):** Enables or disables result caching for identical task executions, specifying a version for cache invalidation and controlling serialization behavior.
*   **Retries (`retries`):** Defines the number of times a task will be retried upon failure, enhancing workflow resilience.
*   **Timeout (`timeout`):** Sets a maximum duration for a single task execution, preventing runaway jobs.
*   **Interruptibility (`interruptible`):** Indicates if a task can be preempted, often used for cost optimization on spot instances.
*   **Deck Generation (`generates_deck`):** Controls whether the task produces an HTML "deck" for visualization of execution details.
*   **Eager Execution (`is_eager`):** Marks a task for eager execution, where Python code directly drives the Flyte engine.

**Example: Configuring Task Metadata**

```python
from flytekit.core.base_task import TaskMetadata
import datetime

# Define metadata for a task
my_task_metadata = TaskMetadata(
    cache=True,
    cache_version="v1.0",
    retries=3,
    timeout=datetime.timedelta(minutes=10),
    interruptible=True,
    generates_deck=True,
)

# This metadata would then be associated with a task definition
# @task(metadata=my_task_metadata)
# def my_task(...):
#     ...
```

### Python Task Implementations

`PythonTask` extends the base `Task` to handle Python-native interfaces and execution. It manages the conversion of Python types to Flyte's literal types and vice-versa.

`PythonFunctionTask` is the most common way to define tasks in Flytekit, typically used via the `@task` decorator. It automatically infers the task's interface from the Python function's signature.

**Execution Behaviors:** `PythonFunctionTask` supports different `ExecutionBehavior` modes:

*   **`DEFAULT`**: Standard task execution where the Python function runs within a container.
*   **`DYNAMIC`**: The task function generates a sub-workflow at runtime. This is powerful for creating dynamic graphs based on input data. The `compile_into_workflow` method handles this graph generation.
*   **`EAGER`**: A specialized mode where the Python function directly orchestrates Flyte entities (tasks, workflows) on the backend. This effectively turns Python into the workflow engine ("Python becomes Propeller").

`AsyncPythonFunctionTask` extends `PythonFunctionTask` to support asynchronous Python functions, allowing for non-blocking operations within tasks.

`EagerAsyncPythonFunctionTask` is a specific implementation for eager workflows, leveraging asynchronous execution to interact with the Flyte backend. When an eager task runs, it establishes a connection to the Flyte backend and dispatches subsequent Flyte entity calls (e.g., other tasks or sub-workflows) as remote executions. This enables complex, data-dependent control flow directly from Python.

**Example: Eager Task Execution**

```python
import asyncio
from flytekit import task, workflow
from flytekit.core.python_function_task import EagerAsyncPythonFunctionTask

@task
async def async_sub_task(a: int) -> int:
    await asyncio.sleep(1)
    return a * 2

@task(is_eager=True)
async def eager_workflow_task(x: int) -> int:
    # In eager mode, calling async_sub_task directly dispatches a remote execution
    y = await async_sub_task(a=x)
    z = await async_sub_task(a=y)
    return z

@workflow
def my_eager_workflow(input_val: int) -> int:
    return eager_workflow_task(x=input_val)

# When my_eager_workflow runs, eager_workflow_task's Python code
# will directly interact with the Flyte backend to launch async_sub_task executions.
```

`PythonInstanceTask` is used for tasks that don't have a direct Python function body but rely on a platform-defined `execute` method, often for plugin-specific integrations.

### Task Resolution

`TaskResolverMixin` defines the mechanism by which Flyte locates and rehydrates task objects at execution time. It provides methods like `load_task` (to reconstruct a task from identifiers) and `loader_args` (to generate identifiers for serialization). The `default_task_resolver` handles standard Python function tasks by using their module and function name.

### Output Handling

The `IgnoreOutputs` exception can be raised within a task's `execute` method to signal that the task's outputs should be disregarded by the workflow engine. This is useful in scenarios like distributed training where only side effects (e.g., model checkpoints) are relevant, and the return value is not needed for downstream tasks.

## Workflows

Workflows define the directed acyclic graph (DAG) of tasks and other workflows. They orchestrate the execution flow.

### Workflow Definition

`WorkflowBase` provides the common foundation for all workflow types, managing properties like name, metadata, and the Python interface.

`PythonFunctionWorkflow` represents workflows defined by a Python function, typically decorated with `@workflow`. During compilation, it analyzes the function's body to identify task calls and their dependencies, constructing the workflow graph.

**Example: Function-based Workflow**

```python
from flytekit import task, workflow

@task
def greet(name: str) -> str:
    return f"Hello, {name}!"

@task
def capitalize(text: str) -> str:
    return text.upper()

@workflow
def greeting_workflow(name_input: str) -> str:
    # Task calls within a workflow define nodes and dependencies
    greeting = greet(name=name_input)
    capitalized_greeting = capitalize(text=greeting)
    return capitalized_greeting
```

`ImperativeWorkflow` offers a programmatic way to define workflows, providing explicit control over adding nodes and defining inputs/outputs. This is useful for dynamically constructing workflows or for scenarios where the `@workflow` decorator's implicit graph generation is not sufficient.

**Example: Imperative Workflow**

```python
from flytekit import Workflow
from flytekit.core.node import Node
from flytekit.core.promise import Promise

# Assuming greet and capitalize tasks from above
# wb = Workflow(name="my_imperative_workflow")
# wb.add_workflow_input("name_input", str)
#
# # Add tasks as nodes
# greet_node = wb.add_entity(greet, name=wb.inputs["name_input"])
# capitalize_node = wb.add_entity(capitalize, text=greet_node.outputs["o0"])
#
# # Define workflow outputs
# wb.add_workflow_output("final_greeting", capitalize_node.outputs["o0"])
```

`ReferenceWorkflow` allows referencing an existing workflow on the Flyte platform without redefining its logic. This is crucial for building modular and reusable components across projects or domains.

### Workflow Metadata

`WorkflowMetadata` defines properties for the workflow itself, such as its `on_failure` policy. `WorkflowFailurePolicy` dictates how the workflow behaves when a node fails (e.g., `FAIL_IMMEDIATELY` or `FAIL_AFTER_EXECUTABLE_NODES_COMPLETE`).

`WorkflowMetadataDefaults` specifies default settings that are inherited by tasks within the workflow, such as `interruptible`.

## Nodes

A `Node` represents a single execution unit within a workflow graph. It can encapsulate a `Task`, a `Workflow` (as a sub-workflow), or a `BranchNode` (for conditional logic).

### Node Properties and Overrides

Nodes have an `id` (unique within the workflow), `metadata` (from the encapsulated entity), `inputs` (bindings to upstream outputs or workflow inputs), and `upstream_node_ids` (explicit dependencies).

The `with_overrides` method on a `Node` (or a `Promise` referencing a node) allows fine-grained control over individual node execution parameters, overriding defaults inherited from the task or workflow definition. This includes:

*   **Resource Allocation (`requests`, `limits`, `resources`):** Specifies CPU, memory, and GPU requirements.
*   **Execution Behavior (`timeout`, `retries`, `interruptible`):** Overrides task-level metadata for a specific node.
*   **Container Image (`container_image`):** Forces a specific container image for the node's execution.
*   **Task Configuration (`task_config`):** Allows overriding plugin-specific configurations.
*   **Caching (`cache`):** Enables or disables caching for this specific node, potentially overriding task defaults.

**Example: Node Overrides**

```python
from flytekit import task, workflow, Resources
import datetime

@task
def my_resource_intensive_task(a: int) -> int:
    return a + 1

@workflow
def my_workflow_with_overrides(x: int) -> int:
    # Call the task, then apply overrides to the resulting node
    node = my_resource_intensive_task(a=x).with_overrides(
        node_name="custom_node_name",
        requests=Resources(cpu="1", mem="2Gi"),
        limits=Resources(cpu="2", mem="4Gi"),
        timeout=datetime.timedelta(minutes=5),
        retries=1,
        cache=True,
        cache_version="v2",
    )
    return node.outputs["o0"]
```

### Node Dependencies

The `runs_before` method (`>>` operator) explicitly defines execution order between nodes. If `node_A >> node_B`, `node_B` will only start after `node_A` completes. Implicit dependencies are also formed when a node's inputs depend on another node's outputs.

### Node Outputs

`NodeOutput` objects represent the output variables of a specific node. When a task or workflow is called within a workflow definition, it returns `Promise` objects that internally reference `NodeOutput` instances.

### Conditional Logic

Flytekit supports conditional execution paths within workflows using `BranchNode` and `IfElseBlock`. `ComparisonExpression` and `ConjunctionExpression` are used to build the boolean logic for these conditions.

**Example: Conditional Branching**

```python
from flytekit import workflow, task, conditional

@task
def task_a(x: int) -> str:
    return f"Task A processed {x}"

@task
def task_b(x: int) -> str:
    return f"Task B processed {x}"

@workflow
def conditional_workflow(value: int) -> str:
    return (
        conditional("check_value")
        .if_(value > 10)
        .then(task_a(x=value))
        .elif_(value < 5)
        .then(task_b(x=value))
        .else_()
        .then(task_a(x=0)) # Default path if no conditions met
    )
```

### Gate Nodes

`GateNode` allows a workflow to pause execution until a specific condition is met, such as a user signal (`SignalCondition`), a time duration (`SleepCondition`), or an explicit approval (`ApproveCondition`).

### Array Nodes

`ArrayNode` is used for map tasks, allowing a single task to be executed in parallel over a collection of inputs. It defines parameters like `parallelism`, `min_successes`, and `min_success_ratio`.

## Promises

`Promise` is a fundamental concept in Flytekit that represents a future value. When a task or workflow is called within a workflow definition (during compilation), it doesn't immediately execute; instead, it returns `Promise` objects. These promises act as placeholders for the actual output values that will be produced when the workflow runs on the Flyte platform.

**Key aspects of `Promise`:**

*   **Duality:** `Promise` handles the distinction between local execution (where the actual Python value is returned) and compilation (where a reference to a future node output is returned).
*   **Graph Construction:** Promises enable the construction of the workflow DAG by linking the outputs of upstream nodes to the inputs of downstream nodes.
*   **`is_ready`:** Indicates if the promise holds an immediate value (e.g., a literal constant) or if it's a reference to a future output.
*   **`val`:** If `is_ready` is true, this property holds the actual literal value.
*   **`ref`:** If `is_ready` is false, this property holds a `NodeOutput` object, which points to the specific node and output variable that will produce the value.
*   **`attr_path`:** Tracks attribute access (e.g., `my_promise.some_field` or `my_promise["some_key"]`), allowing complex data structures to be referenced.
*   **`with_overrides`:** As mentioned in the Nodes section, this method allows applying execution overrides to the underlying node that will produce the promise's value.

`VoidPromise` is a special type of `Promise` returned by tasks or workflows that do not produce any outputs. It cannot be used in expressions or comparisons.

## Launch Plans

`LaunchPlan` is a powerful Flyte construct that bundles a workflow with a specific set of configurations, enabling scheduled, parameterized, and managed executions.

### Creating Launch Plans

The `LaunchPlan.get_or_create` and `LaunchPlan.create` methods are used to define launch plans.

*   **Default Launch Plan:** Every workflow implicitly has a default launch plan, which uses the workflow's defined inputs and no additional configurations. This can be retrieved using `LaunchPlan.get_or_create(workflow=my_workflow)`.
*   **Named Launch Plans:** For more complex scenarios, you can create named launch plans with specific configurations.

**Launch Plan Configurations:**

*   **`default_inputs`:** Provides default values for workflow inputs that can be overridden at execution time.
*   **`fixed_inputs`:** Provides fixed values for workflow inputs that *cannot* be overridden at execution time.
*   **`schedule`:** Defines a recurring schedule for the launch plan using Flyte's `Schedule` model.
*   **`notifications`:** Configures email or Slack notifications for various execution states (e.g., success, failure).
*   **`labels` and `annotations`:** Custom Kubernetes labels and annotations to apply to workflow executions launched by this plan.
*   **`raw_output_data_config`:** Specifies the storage location for offloaded data (e.g., S3 buckets).
*   **`max_parallelism`:** Limits the maximum number of concurrent task nodes within a workflow execution.
*   **`security_context`:** Defines the IAM role or Kubernetes service account under which the workflow execution will run.
*   **`overwrite_cache`:** Forces cache invalidation for executions launched by this plan.
*   **`auto_activate`:** (Client-side only) Controls whether the launch plan is automatically activated upon registration.

`ReferenceLaunchPlan` allows referencing an existing launch plan on the Flyte platform, similar to `ReferenceWorkflow`.

**Example: Creating a Scheduled Launch Plan**

```python
from flytekit import LaunchPlan, workflow
from flytekit.models.schedule import Schedule
from datetime import timedelta

@workflow
def my_daily_report_workflow(start_date: str) -> str:
    # ... workflow logic ...
    return "Report generated"

# Create a launch plan to run daily at a specific time
daily_lp = LaunchPlan.create(
    name="daily_report_lp",
    workflow=my_daily_report_workflow,
    default_inputs={"start_date": "2023-01-01"},
    schedule=Schedule(cron_expression="0 10 * * *"), # Every day at 10:00 AM UTC
)
```

## Interfaces

The `Interface` class provides a Python-native representation of a task or workflow's input and output signature. It stores input variable names, their Python types, and optional default values, as well as output variable names and their types. This information is crucial for Flytekit to perform type checking, serialization, and deserialization of data between Python and the Flyte platform's type system.

## Context Management

Flytekit uses a robust context management system to handle the varying environments during compilation, local execution, and remote execution.

### FlyteContext

`FlyteContext` is an internal-facing object that holds the global state for Flytekit operations. It contains:

*   **`file_access`:** Manages local and remote file operations.
*   **`compilation_state`:** Tracks the nodes and other entities created during workflow graph compilation.
*   **`execution_state`:** Defines the current execution mode (e.g., `LOCAL_TASK_EXECUTION`, `TASK_EXECUTION`, `EAGER_EXECUTION`) and manages execution-specific parameters.
*   **`serialization_settings`:** Configuration for how Flyte entities are serialized for the backend.
*   **`output_metadata_tracker`:** Allows users to attach arbitrary metadata to task outputs.
*   **`worker_queue`:** (For eager execution) Manages the queue of remote Flyte entity calls.

Developers typically interact with `FlyteContext` indirectly through context managers or helper functions.

### Execution Parameters

`ExecutionParameters` is the user-facing context object, accessible within any `@task` function via `flytekit.current_context()`. It provides runtime information and utilities:

*   **`stats`:** A handle to a statsd client for emitting metrics.
*   **`logging`:** A logger instance for task-specific logging.
*   **`working_directory`:** A temporary local directory for the task to store files.
*   **`execution_id`:** The unique identifier of the current workflow execution.
*   **`task_id`:** The unique identifier of the current task execution.
*   **`execution_date`:** The timestamp when the workflow execution started.
*   **`secrets`:** A `SecretsManager` instance for securely accessing secrets.
*   **`checkpoint`:** A handle for checkpointing task state.
*   **`decks`:** A list to which HTML decks can be added for visualization.

**Example: Accessing Execution Parameters**

```python
from flytekit import task, current_context

@task
def my_context_aware_task():
    ctx = current_context()
    ctx.logging.info(f"Task {ctx.task_id.name} running in execution {ctx.execution_id.name}")
    ctx.stats.inc("my_task.runs")
    with open(f"{ctx.working_directory}/output.txt", "w") as f:
        f.write("Hello from task!")
```

### Secrets Management

`SecretsManager` provides a secure way to retrieve secrets within tasks. It attempts to resolve secrets first from environment variables (prefixed with `FLYTE_SECRETS_ENV_PREFIX`) and then from files in a configured secrets directory. This ensures sensitive information is not hardcoded.

### Output Metadata Tracking

`OutputMetadataTracker` allows attaching custom metadata to task outputs. This metadata can be used for lineage tracking, data quality annotations, or other custom purposes.

## Identifiers

Flyte uses a hierarchical identification system for all its entities and executions. These identifiers are crucial for uniquely referencing components across projects, domains, and versions.

*   **`Identifier`:** The base class for all Flyte identifiers, comprising `resource_type` (e.g., `TASK`, `WORKFLOW`, `LAUNCH_PLAN`), `project`, `domain`, `name`, and `version`.
*   **`WorkflowExecutionIdentifier`:** Identifies a specific run of a workflow.
*   **`TaskExecutionIdentifier`:** Identifies a specific run of a task within a node execution.
*   **`NodeExecutionIdentifier`:** Identifies a specific run of a node within a workflow execution.
*   **`SignalIdentifier`:** Identifies a user-defined signal for `GateNode`s.

These core concepts and their architectural relationships form the foundation of Flytekit, enabling developers to build robust, scalable, and observable data and ML workflows.
<!--
key: summary_core_concepts_&_architecture_5b61a8d1-5d0a-474c-8e86-8cde61948fbf
type: summary_end

-->
<!--
code_unit: flytekit.core.base_task.Task
code_unit_type: class
help_text: ''
key: example_ac94f3ca-8272-4247-a6e6-f28c2c0a5363
type: example

-->
<!--
code_unit: flytekit.core.python_function_task.PythonFunctionTask
code_unit_type: class
help_text: ''
key: example_92804d98-72e5-4e94-8007-be0c0ee8ed72
type: example

-->
<!--
code_unit: flytekit.core.workflow.WorkflowBase
code_unit_type: class
help_text: ''
key: example_cdba4f97-3212-4e45-aeff-adef71a5db24
type: example

-->
<!--
code_unit: flytekit.core.node.Node
code_unit_type: class
help_text: ''
key: example_21a1b6ae-7b5b-48de-a773-9bf3b24ad2da
type: example

-->
<!--
code_unit: flytekit.core.promise.Promise
code_unit_type: class
help_text: ''
key: example_7569792e-58c5-498e-9536-d7eeecad19de
type: example

-->
<!--
code_unit: flytekit.core.launch_plan.LaunchPlan
code_unit_type: class
help_text: ''
key: example_03182294-9edb-4666-bfb4-0b33fbc926c8
type: example

-->