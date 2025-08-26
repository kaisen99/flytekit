
<!--
help_text: ''
key: summary_core_concepts_2deb94da-ffc5-41fb-be2f-072ac922674b
modules:
- flytekit.core.base_task
- flytekit.core.python_function_task
- flytekit.core.workflow
- flytekit.core.launch_plan
- flytekit.core.node
- flytekit.core.promise
- flytekit.core.interface
- flytekit.core.reference_entity
- flytekit.core.task
- flytekit.core.docstring
- flytekit.core.environment
questions_to_answer: []
type: summary

-->
# Core Concepts

Flytekit provides a robust framework for defining and executing machine learning and data processing workflows. Understanding its core concepts is essential for effectively building and deploying scalable, reproducible, and maintainable data pipelines.

## Tasks

Tasks are the fundamental building blocks of any Flyte workflow. They represent a single, atomic unit of computation.

### Defining Tasks

The base `Task` class represents the Flyte IDL (Interface Definition Language) specification for a task. It captures essential metadata like name, type, and interface.

For Python-native tasks, the `PythonTask` class serves as the foundation. It manages the Python `Interface` (inputs and outputs) and environment variables. Most user-defined tasks inherit from `PythonFunctionTask`, which automatically infers the task's interface from the decorated Python function's signature.

**Example:**

```python
from flytekit import task

@task
def my_data_processing_task(input_data: str, config_param: int) -> float:
    # Your processing logic here
    return float(len(input_data) * config_param)
```

### Task Metadata and Configuration

Tasks can be configured with various metadata to control their behavior during execution. The `TaskMetadata` class encapsulates properties such as:

*   **Caching**: Enables caching of task outputs to avoid re-execution for identical inputs. `cache=True` and `cache_version` are required. `cache_serialize` ensures identical instances run in serial. `cache_ignore_input_vars` allows excluding specific inputs from the cache key calculation.
*   **Retries**: Specifies the number of times a task should be retried on failure.
*   **Timeout**: Sets the maximum duration for a task execution, after which it will be terminated.
*   **Interruptibility**: Indicates if a task can be interrupted, potentially allowing it to run on lower-cost, pre-emptible infrastructure.
*   **Pod Template Name**: References an existing Kubernetes PodTemplate for custom pod configurations.

These metadata properties are typically set via the `@task` decorator arguments.

**Example:**

```python
import datetime
from flytekit import task

@task(retries=3, timeout=datetime.timedelta(minutes=10), cache=True, cache_version="1.0")
def reliable_task(data: str) -> str:
    # Task logic that might fail or take time
    return data.upper()
```

### Task Execution Behavior

`PythonFunctionTask` supports different execution behaviors:

*   **DEFAULT**: Standard task execution.
*   **DYNAMIC**: A dynamic task generates a sub-workflow at runtime. The `execute` method of a dynamic task compiles the Python function into a `DynamicJobSpec`, which defines the nodes and dependencies of the generated workflow. This is powerful for creating flexible, data-dependent workflows.
*   **EAGER**: Eager tasks (implemented by `EagerAsyncPythonFunctionTask`) allow Python code to directly orchestrate Flyte executions. When an eager task calls another Flyte entity (task, workflow, launch plan), it creates a new Flyte execution on the backend, effectively making Python the "propeller" (orchestrator). This enables highly interactive and dynamic workflow construction directly from Python. Eager tasks are inherently asynchronous and are designed for remote execution.

### Special Task Types

*   **`Echo`**: A built-in task type that simply returns its inputs as outputs. This is often used for testing or as a lightweight pass-through operation, leveraging a Flyte backend plugin to avoid spinning up a container.
*   **`IgnoreOutputs`**: An exception that can be raised within a task to signal that its outputs should be safely ignored by the system. This is particularly useful in distributed training or peer-to-peer algorithms where some task instances might not produce meaningful outputs for downstream consumption.

### Extending Tasks with Plugins

The `TaskPlugins` factory allows developers to register custom task types that extend `PythonFunctionTask`. This is the mechanism for integrating Flytekit with external systems or specialized execution environments (e.g., Spark, Kubernetes operators). Each plugin is associated with a `task_config` object, which holds plugin-specific parameters.

### Task Resolution

The `TaskResolverMixin` defines how Flytekit identifies and rehydrates task objects at runtime. When a task is serialized, its container image includes arguments that point to its resolver and specific loader arguments (e.g., module and function name). At execution time, the resolver uses these arguments to load the correct Python task object. This mechanism is crucial for Flyte's ability to execute arbitrary Python code in a distributed environment.

## Workflows

Workflows define the directed acyclic graph (DAG) of tasks and other workflows. They orchestrate the execution flow and manage data dependencies between individual computational steps.

### Defining Workflows

Workflows can be defined in two primary ways:

1.  **Function-based Workflows (`PythonFunctionWorkflow`)**: The most common approach, where a Python function decorated with `@workflow` defines the workflow structure. Flytekit automatically infers inputs, outputs, and node dependencies by analyzing the function's calls to tasks and other workflows.

    **Example:**

    ```python
    from flytekit import workflow, task

    @task
    def greet(name: str) -> str:
        return f"Hello, {name}!"

    @workflow
    def my_greeting_workflow(user_name: str) -> str:
        greeting_message = greet(name=user_name)
        return greeting_message
    ```

2.  **Imperative Workflows (`ImperativeWorkflow`)**: Provides a programmatic way to construct workflows, offering fine-grained control over node creation and input/output binding. This is suitable for dynamic workflow generation or when a functional definition is not flexible enough.

    **Example:**

    ```python
    from flytekit import Workflow, task
    from typing import NamedTuple

    @task
    def add_one(x: int) -> int:
        return x + 1

    Output = NamedTuple("Output", [("result", int)])

    # Create the workflow with a name
    imperative_wf = Workflow(name="my_imperative_workflow")
    # Add a top-level input
    imperative_wf.add_workflow_input("initial_value", int)
    # Add a task node
    node_1 = imperative_wf.add_entity(add_one, x=imperative_wf.inputs["initial_value"])
    # Add a workflow output
    imperative_wf.add_workflow_output("result", node_1.outputs["o0"], python_type=int)
    ```

### Workflow Metadata and Defaults

*   **`WorkflowMetadata`**: Defines properties specific to the workflow execution, such as `on_failure` policy.
*   **`WorkflowMetadataDefaults`**: Specifies default settings that are inherited by tasks within the workflow, such as `interruptible`.

The `WorkflowFailurePolicy` enum dictates how a workflow behaves when a component node fails:
*   `FAIL_IMMEDIATELY`: The entire workflow fails as soon as any node fails (default).
*   `FAIL_AFTER_EXECUTABLE_NODES_COMPLETE`: The workflow continues to run other independent nodes even if one fails, only marking the workflow as failed after all runnable nodes have completed.

Workflows can also define an `on_failure` handler, which is a task or sub-workflow that executes if the main workflow fails. This is useful for cleanup or notification.

## Launch Plans

Launch plans are the primary mechanism for executing workflows on the Flyte platform. They encapsulate all the necessary information to trigger a workflow run, including default input values, fixed input values, schedules, and execution-specific configurations.

A workflow can have multiple launch plans, each with different configurations.

**Key capabilities of launch plans:**

*   **Default and Fixed Inputs**: Specify input values that are pre-filled or immutable for a given launch plan. `default_inputs` can be overridden at execution time, while `fixed_inputs` cannot.
*   **Schedules**: Configure workflows to run automatically at specified intervals using `schedule` or the newer `trigger` mechanism.
*   **Notifications**: Define alerts to be sent on various execution events (e.g., success, failure).
*   **Resource Overrides**: Apply labels, annotations, raw output data configurations, and maximum parallelism settings to executions initiated by the launch plan.
*   **Security Context**: Define the IAM role or Kubernetes service account under which the workflow execution will run.
*   **Cache Overwrite**: Force cache invalidation for executions launched by this plan.
*   **Auto-activation**: Automatically activate the launch plan on registration.

**Creating Launch Plans:**

The `LaunchPlan.get_or_create()` method is the recommended way to instantiate launch plans. It allows creating a default launch plan for a workflow (if no name is provided and no other parameters are set) or a named launch plan with specific configurations.

**Example:**

```python
from flytekit import LaunchPlan, workflow, task
from flytekit.models.schedule import Schedule
import datetime

@task
def my_task(x: int) -> int:
    return x * 2

@workflow
def my_workflow(input_val: int) -> int:
    return my_task(x=input_val)

# Create a launch plan with a daily schedule and a default input
daily_lp = LaunchPlan.get_or_create(
    workflow=my_workflow,
    name="daily_run",
    default_inputs={"input_val": 10},
    schedule=Schedule(cron_expression="0 10 * * *") # Every day at 10 AM UTC
)
```

## Workflow Graph Construction: Nodes and Promises

When a workflow is defined, Flytekit constructs an execution graph. This involves representing each task or sub-workflow call as a `Node` and managing the data flow between them using `Promise` objects.

### Nodes

A `Node` represents a single step within a workflow. It encapsulates the `flyte_entity` (the task, workflow, or launch plan being executed), its `metadata` (e.g., timeout, retries), and `bindings` that connect its inputs to outputs of upstream nodes or workflow inputs.

Nodes also manage `upstream_nodes`, explicitly defining dependencies. The `>>` operator or `runs_before()` method can be used to define explicit ordering between nodes that don't have direct data dependencies.

**Node Overrides:**

The `with_overrides()` method on a `Node` (or a `Promise` that points to a node) allows applying execution-specific configurations to individual nodes within a workflow. This includes:
*   Renaming the node (`node_name`).
*   Setting resource requests and limits (`requests`, `limits`, or `resources`).
*   Overriding task-specific configurations (`task_config`).
*   Specifying a custom container image (`container_image`).
*   Attaching accelerators (`accelerator`).
*   Configuring shared memory (`shared_memory`).
*   Adjusting timeout, retries, interruptibility, and caching for that specific node.

### Promises

`Promise` objects are central to Flytekit's deferred execution model. When a task or workflow is called during the *compilation* phase (e.g., when defining a workflow), it doesn't immediately execute. Instead, it returns `Promise` objects.

A `Promise` represents the *future* output of a node. It holds a reference to the `NodeOutput` (which points to the originating node and the specific output variable) rather than the actual value. This allows Flytekit to build the workflow DAG without executing the underlying code.

During *local execution*, `Promise` objects are "ready" and contain the actual literal values, enabling the workflow to run like a regular Python program.

**Key aspects of Promises:**

*   **Deferred Execution**: Enables building the workflow graph without immediate computation.
*   **Chaining**: Allows calling methods like `with_overrides()` on the output of a task, even before the task has executed.
*   **Type System Integration**: Promises carry type information, allowing Flytekit to perform type checking during compilation.
*   **Attribute Access**: Promises support attribute access (e.g., `my_output.field_name`) and item access (e.g., `my_list_output[0]`) to refer to specific parts of a structured output.

### VoidPromise

For tasks or workflows that do not return any outputs (i.e., their interface has no outputs), a `VoidPromise` is returned. This special promise cannot be used in any operations or comparisons, as it represents the absence of a return value.

### Conditional Logic

Flytekit supports conditional logic within workflows using comparison and conjunction expressions that operate on `Promise` objects.

*   **`ComparisonExpression`**: Represents a comparison between two operands (e.g., `promise_a == promise_b`, `promise_c > 10`).
*   **`ConjunctionExpression`**: Combines comparison or other conjunction expressions using logical AND (`&`) or OR (`|`) operators.

These expressions are used with the `conditional` construct to define branches in a workflow.

**Example:**

```python
from flytekit import workflow, task, conditional

@task
def is_even(x: int) -> bool:
    return x % 2 == 0

@task
def print_even():
    print("Number is even")

@task
def print_odd():
    print("Number is odd")

@workflow
def check_number(num: int):
    even_check = is_even(x=num)
    conditional("even_or_odd").if_(even_check).then(print_even()).else_(print_odd())
```

## Interfaces and Documentation

### Interface

The `Interface` class provides a Python-native representation of a task or workflow's inputs and outputs. It captures variable names, their Python types, and any default values. This information is crucial for Flytekit to:

*   **Type Checking**: Validate input and output types during compilation and execution.
*   **Serialization**: Convert Python types to Flyte's literal types for backend communication.
*   **Docstring Integration**: Associate input/output descriptions from docstrings.

### Docstring

The `Docstring` class extracts structured information from Python function docstrings. This allows Flytekit to automatically populate short descriptions, long descriptions, and input/output parameter descriptions in the generated Flyte entity metadata, improving discoverability and usability in the Flyte UI.

## Referencing Remote Entities

Flytekit allows referencing tasks, workflows, and launch plans that are already registered on a Flyte cluster without needing their source code locally. This is achieved through `ReferenceEntity` and its specialized subclasses: `ReferenceTask`, `ReferenceWorkflow`, and `ReferenceLaunchPlan`.

These reference entities are initialized with the project, domain, name, and version of the remote entity, along with its expected input and output interfaces. They do not initiate network calls during local definition but are resolved at registration or execution time on the Flyte backend.

**Use Cases:**

*   **Cross-Project/Domain Dependencies**: Build workflows that depend on entities defined in other Flyte projects or domains.
*   **Modularization**: Decouple workflow definitions from the implementation details of their constituent tasks or sub-workflows.
*   **Version Control**: Pin dependencies to specific versions of remote entities.

**Example:**

```python
from flytekit import ReferenceTask
from typing import Dict, Type

# Reference a task that exists on the Flyte cluster
remote_add_task = ReferenceTask(
    project="flytesnacks",
    domain="development",
    name="my_project.my_module.add_numbers",
    version="v1",
    inputs={"a": int, "b": int},
    outputs={"sum": int}
)

# This remote task can now be used in a local workflow definition
# @workflow
# def my_remote_workflow(x: int, y: int) -> int:
#     return remote_add_task(a=x, b=y)
```

## Execution Modes and Environment

Flytekit operates in various execution modes, managed by the `FlyteContext` and `ExecutionState`. These modes dictate how tasks and workflows behave, whether they are being compiled, executed locally, or run on a remote Flyte cluster.

*   **`LOCAL_TASK_EXECUTION`**: A task is executed directly in the local Python environment.
*   **`LOCAL_WORKFLOW_EXECUTION`**: A workflow is executed locally, simulating the graph execution without a Flyte backend.
*   **`DYNAMIC_TASK_EXECUTION`**: A dynamic task is compiling its sub-workflow definition.
*   **`EAGER_LOCAL_EXECUTION`**: An eager task is running locally, but its calls to other Flyte entities are still handled by the local Flytekit runtime.
*   **`EAGER_EXECUTION`**: An eager task is running on the Flyte backend, and its calls to other Flyte entities trigger new Flyte executions.
*   **`TASK_EXECUTION`**: A task is running on the Flyte backend as part of a larger workflow execution.

The `Environment` class provides a convenient way to apply common overrides to tasks and dynamic workflows. It acts as a decorator factory, allowing developers to define a set of default configurations (e.g., resource requests, caching settings) that can be inherited and further extended by individual tasks or dynamic workflows. This promotes consistency and reduces boilerplate when applying similar configurations across multiple entities.

**Example:**

```python
from flytekit import Environment, task

# Define an environment with common settings
my_env = Environment(requests={"cpu": "1", "memory": "500Mi"}, cache=True, cache_version="v1")

@my_env.task
def task_in_env(x: int) -> int:
    return x + 1

@my_env.task(retries=1) # Overrides retries from the environment
def another_task_in_env(y: int) -> int:
    return y * 2
```
<!--
key: summary_core_concepts_2deb94da-ffc5-41fb-be2f-072ac922674b
type: summary_end

-->
<!--
code_unit: flytekit.core.workflow
code_unit_type: class
help_text: ''
key: example_f8601196-5f4c-4f41-838c-ad627f95abbe
type: example

-->
<!--
code_unit: flytekit.core.task
code_unit_type: class
help_text: ''
key: example_4846b6dc-8605-4a4c-90da-187a852c1087
type: example

-->
<!--
code_unit: flytekit.core.launch_plan
code_unit_type: class
help_text: ''
key: example_af3dd837-288e-4800-8bce-99a19f709cf6
type: example

-->