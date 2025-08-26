
<!--
help_text: ''
key: summary_map_tasks_(array_jobs)_6b8a16c4-3e46-4cb1-86b8-82cade091860
modules:
- flytekit.core.array_node
- flytekit.core.array_node_map_task
- flytekit.core.legacy_map_task
questions_to_answer: []
type: summary

-->
Map Tasks, also known as Array Jobs, provide a powerful mechanism to execute a single task or a sub-workflow multiple times in parallel over a collection of inputs. This pattern is particularly useful for processing large datasets or performing embarrassingly parallel computations efficiently.

### Mapping Python Tasks

To execute a Python function task in parallel as an array job, use the `ArrayNodeMapTask` component. This component wraps an existing Python task, transforming its interface to accept lists of inputs and produce a list of outputs.

#### Interface Transformation

When a Python task is mapped, its input signature is automatically transformed. For example, a task defined as `def my_task(x: int, y: str) -> float:` will be mapped to an `ArrayNodeMapTask` with an interface like `(x: List[int], y: List[str]) -> List[float]:`. This allows the map task to accept collections of inputs, where each element in the input list corresponds to an individual execution of the underlying task.

#### Bound Inputs

Sometimes, certain inputs to the underlying task should remain constant across all parallel executions, rather than being mapped over. These are referred to as "bound inputs."

You can specify bound inputs in two ways:
1.  **Explicitly**: By providing a `bound_inputs` set during the `ArrayNodeMapTask` initialization.
2.  **Using `functools.partial`**: If the underlying Python task is wrapped with `functools.partial`, any arguments provided to `partial` are automatically treated as bound inputs.

For example, if `my_task` has inputs `(x: int, y: str)`, and you map `functools.partial(my_task, y="fixed_value")`, then `y` becomes a bound input, and the map task's interface will be `(x: List[int]) -> List[float]:`. The `ArrayNodeMapTaskResolver` ensures that this binding information is correctly preserved and reconstructed during serialization and remote execution.

#### Concurrency and Success Criteria

Map tasks offer fine-grained control over their execution behavior:

*   **`concurrency`**: This parameter limits the number of parallel sub-tasks that can run simultaneously. Setting it to `0` implies unbounded concurrency, allowing as many sub-tasks to run in parallel as resources permit. If unspecified, the array node inherits parallelism from the workflow.
*   **`min_successes`**: Specifies the absolute minimum number of successful sub-task executions required for the overall map task to be considered successful. This takes precedence over `min_success_ratio`.
*   **`min_success_ratio`**: Defines the minimum ratio (as a float between 0.0 and 1.0) of successful sub-task executions required. For example, a `min_success_ratio` of `0.8` means at least 80% of the sub-tasks must succeed. If this ratio is not 1.0 and the underlying task has an output, failed sub-tasks will result in `None` values in the corresponding positions of the output list.

#### Execution Behavior

The execution of an `ArrayNodeMapTask` differs between local development and remote execution on the Flyte platform:

*   **Local Execution**: When executed locally, the `_raw_execute` method iterates through the input collections and sequentially calls the wrapped Python task for each set of inputs. It then aggregates all individual outputs into a single list, simulating the parallel execution.
*   **Remote Execution**: On the Flyte platform, the `ArrayNodeMapTask` is deployed as an array job. Each individual instance of the array job (a "sub-task") receives a specific index (`BATCH_JOB_ARRAY_INDEX_OFFSET` + `BATCH_JOB_ARRAY_INDEX_VAR_NAME`) and processes only the corresponding element from the input collections. The platform then collects the outputs from all successful sub-tasks to form the final output list.

#### Example: Mapping a Python Task

```python
from flytekit import task, workflow, map_task
from typing import List

@task
def process_item(item: int, multiplier: int) -> float:
    """A simple task to process an item."""
    return float(item * multiplier)

@workflow
def my_map_workflow(items: List[int], global_multiplier: int) -> List[float]:
    # Map the process_item task over the 'items' list
    # 'global_multiplier' is a bound input, constant for all sub-tasks
    # Concurrency is limited to 5, and 80% success is required
    mapped_results = map_task(process_item, concurrency=5, min_success_ratio=0.8)(
        item=items,
        multiplier=global_multiplier # This input is bound
    )
    return mapped_results

# Example usage:
# if __name__ == "__main__":
#     results = my_map_workflow(items=[1, 2, 3, 4, 5, 6, 7, 8, 9, 10], global_multiplier=2)
#     print(results)
```

### Mapping Workflows and Reference Entities

The `ArrayNode` component provides a more general capability to map over any Flyte entity that can be executed, including `LaunchPlan`s, `ReferenceTask`s, or `FlyteLaunchPlan`s. This allows for fanning out complex sub-workflows or pre-registered tasks.

#### Target Entities

The `ArrayNode` is initialized with a `target` parameter, which specifies the Flyte entity to be mapped over. This can be:
*   A `LaunchPlan`: For mapping over a defined sub-workflow.
*   A `ReferenceTask`: For mapping over a task registered on the Flyte platform.
*   A `FlyteLaunchPlan`: A remote representation of a launch plan.

#### Data and Execution Modes

The `ArrayNode` adapts its behavior based on the type of target entity:

*   **`DataMode.SINGLE_INPUT_FILE`**: Used when mapping over `LaunchPlan`s. This implies that all inputs for the array job are provided in a single file, which the platform then distributes to individual sub-executions.
*   **`DataMode.INDIVIDUAL_INPUT_FILES`**: Used when mapping over `ReferenceTask`s. In this mode, each sub-execution receives its inputs from a separate, dedicated input file.
*   **`ExecutionMode.FULL_STATE`**: Applicable to `LaunchPlan`s, indicating that the full state of the sub-workflow is managed for each execution.
*   **`ExecutionMode.MINIMAL_STATE`**: Applicable to `ReferenceTask`s, implying a lighter-weight state management for individual task executions.

#### Limitations

*   **Single Output**: The target entity (task or launch plan) being mapped over must have exactly one output. Mapping entities with multiple outputs is not supported.
*   **Bound Inputs**: While the `ArrayNode` internally handles interface transformations, direct user-defined "bound inputs" for the `ArrayNode` itself (similar to `ArrayNodeMapTask`'s `bound_inputs` or `functools.partial`) are not explicitly exposed for `LaunchPlan` or `ReferenceTask` targets. Fixed inputs for a `LaunchPlan` are handled by the `LaunchPlan` definition itself.

### Common Parameters and Considerations

Regardless of whether you are mapping a Python task or another Flyte entity, the core parameters for controlling parallelism and success criteria remain consistent: `concurrency`, `min_successes`, and `min_success_ratio`.

*   **Output Type**: The output of a map task is always a list of the original task's output type. If `min_success_ratio` is less than 1.0, the output list will contain `None` for any sub-tasks that failed, requiring downstream tasks to handle these optional values.
*   **Performance**: Map tasks are highly optimized for parallel execution. Tuning the `concurrency` parameter is crucial for maximizing resource utilization and minimizing execution time, balancing between available cluster resources and the overhead of launching many small tasks.

### Best Practices

*   **Embarrassingly Parallel Problems**: Map tasks are ideal for problems where individual computations are independent and can run in parallel without inter-dependencies.
*   **Input Granularity**: Ensure that the input collection is sufficiently large to benefit from parallelization. For very small collections, the overhead of array jobs might outweigh the benefits.
*   **Error Handling**: When using `min_success_ratio < 1.0`, design downstream tasks to gracefully handle `None` values in the output list, as these indicate failed sub-tasks.
*   **Resource Allocation**: Consider the resource requirements of the individual sub-tasks when setting `concurrency` to avoid over-provisioning or under-utilizing cluster resources.
<!--
key: summary_map_tasks_(array_jobs)_6b8a16c4-3e46-4cb1-86b8-82cade091860
type: summary_end

-->
<!--
code_unit: flytekit.core.array_node.ArrayNode
code_unit_type: class
help_text: ''
key: example_ffec5ba3-fb1d-4ea6-b79b-1d84d4045735
type: example

-->
<!--
code_unit: flytekit.core.array_node_map_task.ArrayNodeMapTask
code_unit_type: class
help_text: ''
key: example_81e7fe45-40e6-4904-b9e2-52321c0d6caf
type: example

-->
<!--
code_unit: flytekit.core.legacy_map_task.MapPythonTask
code_unit_type: class
help_text: ''
key: example_ec2f8ffb-67c4-419d-a2d2-b454d2831a44
type: example

-->