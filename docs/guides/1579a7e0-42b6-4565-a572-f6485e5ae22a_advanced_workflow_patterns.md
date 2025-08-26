
<!--
help_text: ''
key: summary_advanced_workflow_patterns_ac84e1d8-a8f1-4b15-b62c-c82794596970
modules:
- flytekit.core.dynamic_job
- flytekit.core.array_node
- flytekit.core.array_node_map_task
- flytekit.core.condition
- flytekit.core.gate
- flytekit.core.resources
- flytekit.core.options
- flytekit.core.pod_template
- flytekit.core.legacy_map_task
questions_to_answer: []
type: summary

-->
## Advanced Workflow Patterns

Workflows often require dynamic behavior, parallel execution, or specific resource configurations to handle complex data pipelines. Flytekit provides several advanced patterns to address these needs, enabling robust and efficient orchestration of tasks and workflows.

### Parallel Execution with Map Tasks

Map tasks allow you to execute a single task over a collection of inputs in parallel. This is highly efficient for processing large datasets where each item can be processed independently.

#### ArrayNode for Dynamic Mapping

The `ArrayNode` construct facilitates mapping over existing Flyte entities like `LaunchPlan` or `ReferenceTask`. This is particularly useful when you need to execute a sub-workflow or a referenced task multiple times, once for each item in an input collection.

When using an `ArrayNode`, the target entity (task or launch plan) must have a single output. The `ArrayNode` automatically transforms the interface of the target entity to accept list inputs and produce a list output.

Key capabilities of `ArrayNode` include:
*   **Concurrency Control**: Limit the number of parallel executions using the `concurrency` parameter. A value of `0` indicates unbounded concurrency.
*   **Success Criteria**: Define the minimum number or ratio of successful executions required for the map task to be considered successful using `min_successes` or `min_success_ratio`. If the success criteria are not met, the map task will fail.
*   **Input Binding**: Inputs to the mapped entity can be either mapped over (each item in the list passed to a separate instance) or bound (the same scalar value passed to all instances).

Example of mapping a launch plan:

```python
from flytekit import workflow, task, LaunchPlan
from flytekit.core.array_node import ArrayNode

@task
def process_item(item: int) -> int:
    return item * 2

@workflow
def my_sub_workflow(x: int) -> int:
    return process_item(item=x)

my_sub_lp = LaunchPlan.create("my_sub_lp", my_sub_workflow)

@workflow
def map_over_workflow(input_list: list[int]) -> list[int]:
    # Map the launch plan over the input list
    # The ArrayNode automatically handles the list transformation
    mapped_results = ArrayNode(target=my_sub_lp, concurrency=5)(input_list=input_list)
    return mapped_results
```

#### ArrayNodeMapTask for Python Functions

For mapping over Python functions decorated as Flyte tasks, the `ArrayNodeMapTask` provides a specialized implementation. It extends the `PythonTask` and handles the transformation of the task's interface to accept list inputs and produce a list output.

The `ArrayNodeMapTask` also supports `concurrency`, `min_successes`, and `min_success_ratio` parameters, similar to `ArrayNode`.

Example of mapping a Python task:

```python
from flytekit import task, workflow, map_task
import functools

@task
def process_data(data: str, prefix: str) -> str:
    return f"{prefix}-{data}"

@workflow
def my_map_workflow(data_list: list[str], common_prefix: str) -> list[str]:
    # Map the process_data task over data_list.
    # The 'prefix' input is bound to a single value for all mapped instances.
    mapped_task = map_task(functools.partial(process_data, prefix=common_prefix))
    results = mapped_task(data=data_list)
    return results
```

#### Handling Bound Inputs

When a task is mapped, its inputs are typically expected to be lists, with each element of the list corresponding to an individual execution of the mapped task. However, some inputs might be scalar values that should be passed identically to all mapped instances. These are referred to as "bound inputs."

The `ArrayNodeMapTask` automatically detects bound inputs when `functools.partial` is used to pre-fill arguments. Alternatively, you can explicitly specify `bound_inputs` when constructing the `ArrayNodeMapTask` or `MapPythonTask` (a similar, older concept for mapping Python tasks).

The `ArrayNodeMapTaskResolver` (and its legacy counterpart `MapTaskResolver`) plays a crucial role in reconstructing the task's interface at runtime, ensuring that only the unbound inputs are treated as collections to be mapped over. This resolver serializes and deserializes the information about bound variables, allowing the Flyte engine to correctly execute individual instances of the mapped task.

### Conditional Logic

Conditional logic allows workflows to execute different branches of tasks based on runtime conditions. This enables dynamic workflow paths, error handling, and A/B testing scenarios.

#### Defining Conditional Branches

The `conditional` construct provides a fluent API for defining `if`, `elif`, and `else` branches within a workflow. Each branch can contain a sequence of tasks or sub-workflows.

*   **`if_(expr)`**: Starts the first conditional branch. `expr` must be a comparison or conjunction expression involving workflow inputs or outputs of upstream nodes.
*   **`elif_(expr)`**: Adds an "else if" branch.
*   **`else_()`**: Defines the default branch to execute if no preceding conditions are met.
*   **`then(output)`**: Specifies the output of the current branch. This output can be a single promise, a tuple of promises, or a void promise.
*   **`fail(error_message)`**: Specifies that the branch should fail with a given error message.

All branches within a conditional block must produce a consistent set of outputs. If branches produce different outputs, Flytekit will attempt to find the least common set of outputs. If a branch produces no output (e.g., a task with no return value), the conditional block might return a `VoidPromise`.

Example of conditional execution:

```python
from flytekit import task, workflow, conditional

@task
def task_a(x: int) -> str:
    return f"Task A executed with {x}"

@task
def task_b(x: int) -> str:
    return f"Task B executed with {x}"

@task
def task_c() -> str:
    return "Task C executed (else branch)"

@workflow
def my_conditional_workflow(input_val: int) -> str:
    result = (
        conditional("my_condition")
        .if_(input_val > 10)
        .then(task_a(x=input_val))
        .elif_(input_val < 5)
        .then(task_b(x=input_val))
        .else_()
        .then(task_c())
    )
    return result
```

#### Local Execution Behavior

During local execution, the `LocalExecutedConditionalSection` evaluates the conditions directly and executes only the chosen branch. This provides a straightforward way to test conditional logic without deploying to a Flyte cluster. The `SkippedConditionalSection` is used internally for nested conditionals where a parent branch has already been evaluated to false, preventing unnecessary evaluation of inner branches.

### Human-in-the-Loop and Time-Based Gates

Gates introduce pauses in workflow execution, allowing for external intervention or timed delays. This is crucial for workflows that require manual approvals, waiting for external systems, or implementing scheduled delays.

The `Gate` construct supports:
*   **Timed Waits**: Use `sleep_duration` to pause the workflow for a specified duration.
*   **User Input**: Specify an `input_type` to prompt the user for input during local execution. The workflow will proceed with the provided input.
*   **Approval Gates**: Integrate manual approval steps into the workflow. The workflow pauses until a user explicitly approves its continuation. This is typically used with an upstream task's output that needs human review.

Example of gate usage:

```python
import datetime
from flytekit import workflow, task, Gate

@task
def prepare_for_approval(data: str) -> str:
    return f"Prepared data for approval: {data}"

@task
def final_processing(approved_data: str) -> str:
    return f"Final processing of: {approved_data}"

@workflow
def approval_workflow(initial_data: str) -> str:
    prepared = prepare_for_approval(data=initial_data)

    # Create an approval gate that waits for user input based on 'prepared' data
    approved_output = Gate(name="manual_approval_gate")(prepared)

    # Alternatively, a sleep gate:
    # Gate(name="wait_for_five_minutes", sleep_duration=datetime.timedelta(minutes=5))()

    # Or a gate waiting for specific input:
    # user_input = Gate(name="get_user_feedback", input_type=str)()

    return final_processing(approved_data=approved_output)
```

During local execution, approval gates will prompt the user via the command line.

### Resource Management

Efficient resource allocation is critical for cost-effective and performant workflows. Flytekit allows you to specify resource requests and limits for tasks.

#### Specifying Resource Requests and Limits

The `Resources` class defines the compute resources (CPU, memory, GPU, ephemeral storage) a task requires. You can specify both a request (minimum required) and a limit (maximum allowed).

*   **`cpu`**: CPU units (e.g., `"1"` for 1 CPU, `"100m"` for 0.1 CPU).
*   **`mem`**: Memory (e.g., `"2048"` for 2KB, `"2Gi"` for 2 Gigabytes).
*   **`gpu`**: GPU units.
*   **`ephemeral_storage`**: Temporary storage (e.g., `"1Gi"`).

When defining `Resources`, you can provide a single value (which sets both request and limit to that value) or a tuple/list `(request, limit)`.

The `ResourceSpec` class explicitly separates `requests` and `limits` into distinct `Resources` objects, providing a more granular way to define resource requirements.

Example of defining task resources:

```python
from flytekit import task
from flytekit.core.resources import Resources

@task(requests=Resources(cpu="500m", mem="1Gi"), limits=Resources(cpu="1", mem="2Gi"))
def my_resource_intensive_task(data: int) -> int:
    # This task will request 0.5 CPU and 1GB memory,
    # and will be limited to 1 CPU and 2GB memory.
    return data * 100
```

### Execution Configuration Options

The `Options` class provides a comprehensive way to configure various aspects of a workflow execution or a launch plan. These options allow for fine-tuning behavior, metadata, and security settings.

Key configurable options include:
*   **`labels`**: Custom key-value pairs for organizing and querying executions.
*   **`annotations`**: Non-identifying metadata for executions.
*   **`security_context`**: Defines the security identity (e.g., Kubernetes service account) under which the execution runs.
*   **`raw_output_data_config`**: Specifies the remote prefix for offloaded data storage (e.g., S3, GCS).
*   **`max_parallelism`**: Limits the total number of task nodes that can run concurrently across the entire workflow.
*   **`notifications`**: Configures email or Slack notifications for execution events (e.g., success, failure).
*   **`disable_notifications`**: A boolean flag to disable all notifications for an execution.
*   **`overwrite_cache`**: Forces re-execution even if cached results are available.

These options can be applied when registering a `LaunchPlan` or when triggering an execution.

Example of using execution options:

```python
from flytekit import workflow, LaunchPlan
from flytekit.core.options import Options
from flytekit.models.common import Labels, Annotations, RawOutputDataConfig
from flytekit.models.security import SecurityContext, Identity

@workflow
def my_simple_workflow() -> str:
    return "Hello Flyte!"

# Define custom options for a launch plan
custom_options = Options(
    labels=Labels({"team": "data-science"}),
    annotations=Annotations({"reviewer": "john.doe"}),
    security_context=SecurityContext(run_as=Identity(k8s_service_account="my-service-account")),
    raw_output_data_config=RawOutputDataConfig(output_location_prefix="s3://my-custom-bucket/flyte-outputs"),
    max_parallelism=10,
    disable_notifications=True,
)

# Create a launch plan with these options
my_launch_plan = LaunchPlan.create("my_custom_lp", my_simple_workflow, default_options=custom_options)
```

### Customizing Kubernetes Pods

For advanced scenarios requiring fine-grained control over the underlying Kubernetes Pods that execute tasks, the `PodTemplate` class allows direct specification of the Pod's configuration.

The `PodTemplate` enables:
*   **`pod_spec`**: Directly provide a Kubernetes `V1PodSpec` object to define containers, volumes, init containers, and other Pod-level settings.
*   **`primary_container_name`**: Specify the name of the main container within the Pod where the task's code will run.
*   **`labels` and `annotations`**: Apply Kubernetes labels and annotations directly to the Pod.

This feature is typically used for highly specialized task environments, such as integrating with custom sidecar containers, specific node affinities, or complex networking configurations.

Example of using a PodTemplate:

```python
from flytekit import task
from flytekit.core.pod_template import PodTemplate
from kubernetes.client import V1PodSpec, V1Container

@task(pod_template=PodTemplate(
    pod_spec=V1PodSpec(
        containers=[
            V1Container(name="primary", image="my-custom-image:latest"),
            V1Container(name="sidecar", image="my-sidecar-image:1.0", command=["/bin/sh", "-c", "sleep 3600"])
        ],
        node_selector={"kubernetes.io/hostname": "my-specific-node"}
    ),
    primary_container_name="primary",
    labels={"env": "production"},
    annotations={"owner": "platform-team"}
))
def my_custom_pod_task() -> str:
    # This task will run in a Pod with a custom spec, including a sidecar container
    # and scheduled on a specific node.
    return "Task executed in a custom Pod."
```
<!--
key: summary_advanced_workflow_patterns_ac84e1d8-a8f1-4b15-b62c-c82794596970
type: summary_end

-->
<!--
code_unit: flytekit.core.workflow.ImperativeWorkflow
code_unit_type: class
help_text: ''
key: example_2d6adeb0-0a69-49da-8730-4b0524cea149
type: example

-->
<!--
code_unit: flytekit.core.condition.ConditionalSection
code_unit_type: class
help_text: ''
key: example_f6cc7ba6-2112-4c27-be20-b4fda6d8755f
type: example

-->
<!--
code_unit: flytekit.core.array_node.ArrayNode
code_unit_type: class
help_text: ''
key: example_99038d9e-5ab0-468c-8b81-b685ce5db2d5
type: example

-->
<!--
code_unit: flytekit.core.gate.Gate
code_unit_type: class
help_text: ''
key: example_cff797f0-9fed-4f04-9a2d-a7b3ede8fc39
type: example

-->