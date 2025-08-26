
<!--
help_text: ''
key: summary_map_tasks_and_array_jobs_b48cbc57-f7f8-407d-8e0d-e2d0ce15f247
modules:
- flytekit.core.array_node
- flytekit.core.array_node_map_task
- flytekit.core.legacy_map_task
questions_to_answer: []
type: summary

-->
Map tasks and array jobs enable the parallel execution of a single task over a collection of inputs. This capability is fundamental for processing large datasets or performing repetitive operations efficiently within a workflow.

### Core Concepts

Map tasks transform a single-instance task into a task that operates on collections. If a task accepts an `int` and returns a `str`, its mapped version accepts a `List[int]` and returns a `List[str]`. Each element in the input list triggers an independent execution of the underlying task.

#### Parallel Execution
The primary benefit of map tasks is the ability to distribute work. Instead of running a loop sequentially, Flyte dispatches each item in the input collection as a separate, parallel sub-task. This significantly reduces overall execution time for data-parallel workloads.

#### Input and Output Handling
When a task is designated as a map task, its interface automatically adapts. All inputs that are not explicitly "bound" (see Bound Inputs) are converted into lists of their original type. Similarly, the task's single output is transformed into a list of the original output type.

For example, a task `(x: int) -> str` becomes a map task `(x: List[int]) -> List[str]`.

#### Concurrency and Fault Tolerance
Map tasks offer fine-grained control over execution behavior:

*   **Concurrency:** The `concurrency` parameter limits the number of parallel sub-tasks that can run simultaneously. Setting it to `0` allows unbounded concurrency, while leaving it unspecified inherits parallelism from the workflow. This is crucial for managing resource consumption.
*   **Minimum Successes/Ratio:** To handle transient failures or allow for acceptable data loss, map tasks can be configured with `min_successes` or `min_success_ratio`.
    *   `min_successes`: Specifies the absolute minimum number of successful sub-task executions required for the overall map task to be considered successful.
    *   `min_success_ratio`: Specifies the minimum percentage of successful sub-task executions. If this ratio is not met, the map task fails.
    If `min_success_ratio` is set to a value less than 1.0 (or `min_successes` is set to a value less than the total number of inputs), the output list will contain `None` for any failed sub-tasks, indicating that the corresponding output was not produced.

#### Bound Inputs and Partial Application
Sometimes, a task needs to process a list of inputs while also receiving one or more constant, scalar inputs that apply to all sub-tasks. These are known as "bound inputs."

The system supports `functools.partial` to pre-bind certain arguments of the underlying task. When a task is wrapped with `functools.partial`, the arguments provided to `partial` are treated as bound inputs and are not converted to lists in the map task's interface. This allows for flexible task composition.

For example, if a task `(data: int, config: str) -> str` is mapped as `map_task(functools.partial(my_task, config="default"))`, the map task's interface will be `(data: List[int]) -> List[str]`, with `config` remaining a scalar `str` internally.

### Implementation Details

Map tasks are typically created using a `map_task` decorator or function, which wraps an existing Python task.

#### Defining a Mappable Task
Any `PythonFunctionTask` or `PythonInstanceTask` can be mapped, provided it adheres to the single-output constraint.

```python
from flytekit import task, map_task

@task
def process_item(item: int, multiplier: int) -> float:
    """Processes a single item."""
    return float(item * multiplier)

# Create a map task from the single-item task
mapped_process_item = map_task(process_item)
```

#### Creating a Map Task
The `map_task` function (or decorator) is the primary entry point. It takes the target task and optional configuration parameters.

```python
# Example of creating a map task with concurrency and success ratio
mapped_task_with_config = map_task(
    process_item,
    concurrency=10,
    min_success_ratio=0.8,
)
```

#### Configuring Map Tasks
Configuration parameters like `concurrency`, `min_successes`, and `min_success_ratio` are passed directly to the `map_task` function.

```python
# Map task with a minimum of 5 successful executions
mapped_task_min_successes = map_task(
    process_item,
    min_successes=5,
)
```

#### Local Execution
When executing a workflow containing a map task locally, the system simulates the parallel execution. It iterates through the input collection and calls the underlying task for each item, collecting the results into a list. This allows for quick testing and debugging without deploying to a Flyte cluster.

The `ArrayNodeMapTask` and `MapPythonTask` classes contain `_raw_execute` methods that implement this local simulation logic. They ensure that the full output collection is produced, mirroring the behavior of the map task on the platform.

#### Remote Execution and Array Job Indexing
On the Flyte platform, map tasks are executed as array jobs. The Flyte engine distributes the input collection across multiple parallel instances of the underlying task. Each instance is responsible for processing a specific element from the input collection.

The `_compute_array_job_index` static method within `ArrayNodeMapTask` and `MapPythonTask` retrieves the unique index of the current sub-task instance from environment variables (e.g., `BATCH_JOB_ARRAY_INDEX_OFFSET`, `BATCH_JOB_ARRAY_INDEX_VAR_NAME`). This index is then used to select the correct input element from the overall input collection for that specific sub-task instance.

The `execute` method of `ArrayNodeMapTask` and `MapPythonTask` intelligently switches between local execution simulation and remote execution logic based on the current `ExecutionState.Mode`.

### Advanced Usage

#### Mapping over Launch Plans
The `ArrayNode` component allows mapping over `LaunchPlan`s or `ReferenceTask`s, not just direct Python tasks. This provides flexibility to map over entire sub-workflows or pre-defined tasks.

When mapping over a `LaunchPlan`, the `ArrayNode` handles the transformation of the `LaunchPlan`'s interface to a list-based interface. It also determines the appropriate `data_mode` and `execution_mode` for the array node based on the target entity. For `LaunchPlan`s, it uses `SINGLE_INPUT_FILE` and `FULL_STATE`, while for `ReferenceTask`s, it uses `INDIVIDUAL_INPUT_FILES` and `MINIMAL_STATE`.

#### Handling Bound Inputs with `functools.partial`
When `functools.partial` is used to create a map task, the system automatically identifies the bound inputs from `partial.keywords`. This information is crucial for the `ArrayNodeMapTaskResolver` and `MapTaskResolver` to correctly reconstruct the map task's interface at runtime, ensuring that only the unbound inputs are treated as collections.

```python
import functools
from flytekit import task, map_task, workflow

@task
def greet_person(name: str, greeting: str) -> str:
    return f"{greeting}, {name}!"

# Bind the 'greeting' input using functools.partial
partial_greet = functools.partial(greet_person, greeting="Hello")

# Create a map task from the partially applied task
mapped_greet = map_task(partial_greet)

@workflow
def greeting_workflow(names: list[str]) -> list[str]:
    return mapped_greet(name=names)

# When greeting_workflow(["Alice", "Bob"]) is executed,
# mapped_greet will run greet_person twice:
# 1. greet_person(name="Alice", greeting="Hello")
# 2. greet_person(name="Bob", greeting="Hello")
```

### Key Components

*   **`ArrayNode`**: Represents a node in a workflow graph that performs mapping. It orchestrates the mapping over various Flyte entities like `LaunchPlan`s and `ReferenceTask`s, managing concurrency and success criteria at the workflow level.
*   **`ArrayNodeMapTask` and `MapPythonTask`**: These are specialized task definitions that wrap an underlying Python task (`PythonFunctionTask` or `PythonInstanceTask`). They handle the transformation of the task's interface for mapping, define the execution command for array jobs, and manage local vs. remote execution logic. `MapPythonTask` is an older implementation, while `ArrayNodeMapTask` is the current standard for array node mapping.
*   **`ArrayNodeMapTaskResolver` and `MapTaskResolver`**: These resolvers are critical for the Flyte engine to correctly load and execute map tasks. They reconstruct the map task definition at runtime, particularly by identifying and applying the "bound inputs" that were specified during task creation (e.g., via `functools.partial`). This ensures the correct interface and execution behavior for each sub-task instance.

### Limitations and Considerations

*   **Single Output Constraint:** Mapped tasks are currently limited to having a single output. Tasks with multiple outputs cannot be directly mapped.
*   **Supported Task Types:** `ArrayNodeMapTask` and `MapPythonTask` primarily support `PythonFunctionTask` (with default execution behavior) and `PythonInstanceTask`. Other task types or dynamic/eager execution modes are not supported for mapping.
*   **Local Execution of Remote Entities:** When `ArrayNode` maps over `ReferenceTask` or `FlyteLaunchPlan` (remote entities), local execution is not supported. This is because the local environment cannot simulate the execution of a remote entity.
*   **Partial Tasks with List Inputs:** `MapPythonTask` explicitly disallows `functools.partial` where the bound arguments themselves are lists. This is to prevent ambiguity with the mapped inputs.

### Best Practices

*   **Granularity of Mapped Tasks:** Design the underlying task to be granular enough to process a single item efficiently. Avoid making the mapped task too complex or resource-intensive, as this can lead to overhead when scaled across many instances.
*   **Error Handling:** Utilize `min_success_ratio` or `min_successes` to build resilient workflows that can tolerate partial failures in large-scale data processing. Implement robust error handling within the individual mapped task to provide meaningful logs and outputs for debugging.
*   **Resource Management:** Carefully consider the `concurrency` setting to avoid overwhelming the cluster with too many parallel sub-tasks, especially for resource-intensive operations.
*   **Input Data Size:** For very large input collections, consider using Flyte's data offloading capabilities to prevent excessive memory usage during serialization and deserialization.
<!--
key: summary_map_tasks_and_array_jobs_b48cbc57-f7f8-407d-8e0d-e2d0ce15f247
type: summary_end

-->
<!--
code_unit: flytekit.core.array_node.ArrayNode
code_unit_type: class
help_text: ''
key: example_0db0fb59-3159-467d-adb6-ee4f7787aef9
type: example

-->
<!--
code_unit: flytekit.core.array_node_map_task.ArrayNodeMapTask
code_unit_type: class
help_text: ''
key: example_41db37e8-9582-4f2a-a803-772ff4ac09fd
type: example

-->
<!--
code_unit: flytekit.core.legacy_map_task.MapPythonTask
code_unit_type: class
help_text: ''
key: example_bca8f631-cb7a-44c1-8657-76c2f1763b65
type: example

-->