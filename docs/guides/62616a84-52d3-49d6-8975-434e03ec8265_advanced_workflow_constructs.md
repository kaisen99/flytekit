
<!--
help_text: ''
key: summary_advanced_workflow_constructs_360f6d70-960a-4901-83cd-9bea64196a73
modules:
- flytekit.core.condition
- flytekit.core.array_node
- flytekit.core.array_node_map_task
- flytekit.core.legacy_map_task
- flytekit.core.gate
- flytekit.models.core.workflow
- flytekit.models.core.condition
- flytekit.models.dynamic_job
- flytekit.models.array_job
questions_to_answer: []
type: summary

-->
# Advanced Workflow Constructs

Flytekit provides advanced workflow constructs that enable complex control flow, parallel processing, and human-in-the-loop interactions within your workflows. These constructs allow you to build more dynamic, resilient, and interactive data pipelines.

## Conditional Workflows

Conditional workflows allow you to define branches of execution based on runtime conditions. This enables dynamic decision-making within your Flyte workflows, executing different tasks or sub-workflows based on input values or the results of preceding nodes.

### Concept and Purpose

A conditional workflow evaluates a boolean expression and executes a specific branch (`then` block) if the condition is met. You can chain multiple `elif_` (else-if) conditions and provide a final `else_` block for cases where no preceding conditions are satisfied. This mirrors standard programming language `if/elif/else` constructs.

Under the hood, conditional logic is represented by a `BranchNode` in the workflow graph, which contains an `IfElseBlock`. Each `IfBlock` within the `IfElseBlock` consists of a `BooleanExpression` (the condition) and the `Node` to execute if that condition is true.

### Defining Conditions

Conditions are built using `ComparisonExpression` and `ConjunctionExpression` objects.
*   **Comparison Expressions:** Compare two values using operators like equality (`==`), inequality (`!=`), greater than (`>`), greater than or equal to (`>=`), less than (`<`), and less than or equal to (`<=`). The values can be workflow inputs, outputs of previous tasks, or literal values.
*   **Conjunction Expressions:** Combine multiple boolean expressions using logical `AND` (`&`) or `OR` (`|`).

**Limitations:**
*   Only comparison and conjunction expressions are supported. Direct logical operations (`and`, `or`, `is`, `not`) on Python booleans are not supported within the conditional expression itself, as they would be evaluated at definition time, not at runtime.
*   Unary expressions (e.g., `if_(x)` where `x` is a `Promise`) are not supported. Conditions must be explicit comparisons or conjunctions.

### Structuring Branches

The `conditional` function initiates a conditional block, which then uses a fluent API for defining branches:

```python
from flytekit import conditional, task, workflow

@task
def process_data_a(x: int) -> str:
    return f"Processed A: {x}"

@task
def process_data_b(x: int) -> str:
    return f"Processed B: {x}"

@task
def default_process(x: int) -> str:
    return f"Default Process: {x}"

@workflow
def my_conditional_workflow(input_val: int) -> str:
    result = (
        conditional("my_condition")
        .if_(input_val > 10)
        .then(process_data_a(x=input_val))
        .elif_(input_val <= 5)
        .then(process_data_b(x=input_val))
        .else_()
        .then(default_process(x=input_val))
    )
    return result
```

In this example:
*   `conditional("my_condition")` creates a `ConditionalSection` and a `Condition` object.
*   `.if_(input_val > 10)` creates a `Case` with a `ComparisonExpression`.
*   `.then(process_data_a(x=input_val))` associates a node (the `process_data_a` task) with the `Case`.
*   `.elif_(input_val <= 5)` adds another `Case` for an "else-if" scenario.
*   `.else_()` defines the fallback branch.

### Handling Outputs

A crucial aspect of conditional workflows is how outputs are handled. All branches within a conditional block must produce a compatible set of outputs. Flytekit automatically determines the "least common set" of outputs across all branches.

*   If all branches produce the same output variables with compatible types, the conditional block will output those variables.
*   If branches produce different sets of output variables, only the variables common to *all* branches will be exposed as outputs of the conditional block.
*   If any branch returns a `VoidPromise` (i.e., no output) or `None`, the entire conditional block will be treated as having no output (`VoidPromise`).

This ensures type safety and predictability in the workflow graph.

### Local Execution vs. Remote Compilation

During local execution, `LocalExecutedConditionalSection` evaluates conditions directly and executes only the chosen branch. This allows for interactive debugging and testing.

During remote compilation, `ConditionalSection` (the base class) generates the `BranchNode` structure, which is then sent to the Flyte backend. The backend's execution engine evaluates the conditions at runtime and orchestrates the execution of the appropriate branch. `SkippedConditionalSection` is used internally for nested conditionals where an outer branch has already been determined as false, preventing unnecessary evaluation of inner branches.

## Map Tasks

Map tasks provide a powerful way to parallelize the execution of a single task over a collection of inputs. This is analogous to the `map` function in functional programming, applying a function to each item in an iterable.

### Concept and Purpose

A map task takes a list of inputs and executes a specified sub-task for each element in the list, potentially in parallel. This is highly efficient for batch processing, data transformations, or any scenario where the same operation needs to be applied independently to multiple data points.

The core component is `ArrayNodeMapTask`, which wraps a `PythonFunctionTask` or `PythonInstanceTask`. When placed in a workflow, it becomes an `ArrayNode`.

### Defining Map Tasks

You define a map task by wrapping an existing task with the `map_task` function:

```python
from flytekit import map_task, task, workflow
from typing import List

@task
def square(x: int) -> int:
    return x * x

@workflow
def map_example_workflow(numbers: List[int]) -> List[int]:
    # Map the 'square' task over the 'numbers' list
    squared_numbers = map_task(square)(x=numbers)
    return squared_numbers
```

### Configuration Options

Map tasks offer several configuration options to control parallelism and success criteria:

*   **`concurrency`**: Limits the number of parallel executions of the mapped task. If the input size exceeds this value, tasks run in batches. A value of `0` means unbounded concurrency. If unspecified, it inherits parallelism from the workflow.
*   **`min_successes`**: Specifies the absolute minimum number of successful sub-task executions required for the map task to be considered successful. If this threshold is not met, the map task fails.
*   **`min_success_ratio`**: Specifies the minimum ratio (0.0 to 1.0) of successful sub-task executions. For example, `0.8` means 80% of tasks must succeed. If set to anything less than `1.0` and the mapped task has a single output, the output list will contain `Optional[T]` to account for potential failures.

These options are configured when defining the `map_task`:

```python
@workflow
def configured_map_workflow(numbers: List[int]) -> List[int]:
    # Run with a maximum of 5 parallel tasks, requiring 80% success
    squared_numbers = map_task(square, concurrency=5, min_success_ratio=0.8)(x=numbers)
    return squared_numbers
```

### Handling Bound Inputs

Sometimes, a mapped task might have inputs that are *not* meant to be mapped over, but rather remain constant for all parallel executions. These are called "bound inputs." This is particularly useful when using `functools.partial` to pre-fill some arguments of the underlying task.

The `ArrayNodeMapTaskResolver` plays a role in reconstructing the interface at runtime, ensuring that only the unbound inputs are treated as lists.

```python
import functools
from flytekit import map_task, task, workflow
from typing import List

@task
def multiply_and_add(x: int, multiplier: int, offset: int) -> int:
    return (x * multiplier) + offset

@workflow
def partial_map_workflow(numbers: List[int]) -> List[int]:
    # Create a partial task where 'multiplier' and 'offset' are fixed
    partial_multiply = functools.partial(multiply_and_add, multiplier=10, offset=5)
    
    # Map the partial task over 'numbers'. 'x' will be mapped, 'multiplier' and 'offset' will be bound.
    results = map_task(partial_multiply)(x=numbers)
    return results
```

### Output Handling

Map tasks currently support only tasks with a single output. The output of a map task is always a list of the individual task outputs. If `min_success_ratio` is less than `1.0`, the output list will be `List[Optional[T]]`, where `None` indicates a failed sub-task.

### Performance Considerations

Map tasks are highly optimized for parallel execution on the Flyte platform. They leverage array job capabilities of underlying compute engines (e.g., Kubernetes batch jobs) to efficiently manage large-scale parallel computations.

### Limitations

*   Only tasks with a single output are supported for mapping.
*   Map tasks do not support `functools.partial` with list inputs for the bound arguments.

## Gate Nodes

Gate nodes introduce human interaction or timed waits into your workflows, allowing for manual approvals, data input, or scheduled pauses.

### Concept and Purpose

A `GateNode` acts as a special type of node in a workflow that does not execute user-defined code but instead pauses execution until a specific condition is met. This condition can be:
1.  A specified duration has passed (sleep gate).
2.  User input is provided (input gate).
3.  User approval is given (approval gate).

This enables building workflows that require manual intervention, scheduled delays, or external data injection.

### Types of Gate Nodes

The `gate` function is used to create different types of gate nodes:

*   **Sleep Gate:** Pauses the workflow for a specified duration. This is useful for introducing delays, waiting for external systems, or scheduling.
    ```python
    from flytekit import gate, workflow
    import datetime

    @workflow
    def sleep_workflow():
        gate.sleep("wait_for_a_bit", sleep_duration=datetime.timedelta(minutes=5))
        # Workflow continues after 5 minutes
    ```
    Internally, this uses `SleepCondition`.

*   **Input Gate:** Pauses the workflow and waits for a user to provide input of a specified type. The workflow then proceeds with the provided input.
    ```python
    from flytekit import gate, workflow
    from typing import Dict

    @workflow
    def input_workflow() -> Dict[str, str]:
        user_data = gate.input("get_user_data", input_type=Dict[str, str])
        return user_data
    ```
    This uses `SignalCondition` to define the expected input type.

*   **Approval Gate:** Pauses the workflow and waits for a user to approve or reject a specific value (typically an output from a previous task). If approved, the workflow continues; if rejected, the workflow fails.
    ```python
    from flytekit import gate, task, workflow

    @task
    def generate_report_summary() -> str:
        return "Summary of critical report: All systems nominal."

    @workflow
    def approval_workflow() -> str:
        summary = generate_report_summary()
        approved_summary = gate.approve("review_summary", upstream_item=summary)
        # If approved, approved_summary will contain the value of 'summary'
        return approved_summary
    ```
    This uses `ApproveCondition`. The `upstream_item` is the `Promise` whose value needs approval.

### Local Execution Behavior

During local execution, gate nodes provide interactive prompts in the console:
*   **Sleep gates:** Print a message indicating the simulated sleep duration.
*   **Input gates:** Prompt the user to enter data of the specified type via standard input.
*   **Approval gates:** Prompt the user for confirmation (`y/n`) to approve the `upstream_item`'s value. If the user declines, a `FlyteDisapprovalException` is raised, causing the local execution to fail.

This interactive behavior allows for testing workflows with human intervention without deploying to a Flyte cluster.

### Considerations

*   Gate nodes are primarily for human interaction or simple timed waits. For complex scheduling or external system integration, consider using event-driven patterns or external services.
*   The `timeout` parameter in `NodeMetadata` can be applied to gate nodes to specify how long to wait for a signal or approval before timing out.
<!--
key: summary_advanced_workflow_constructs_360f6d70-960a-4901-83cd-9bea64196a73
type: summary_end

-->
<!--
code_unit: flytekit.core.condition.ConditionalSection
code_unit_type: class
help_text: ''
key: example_e301efa6-59ec-477b-acdd-9326b31ab514
type: example

-->
<!--
code_unit: flytekit.core.array_node.ArrayNode
code_unit_type: class
help_text: ''
key: example_90fadd35-95bc-4ea9-aa97-8647198d55e4
type: example

-->