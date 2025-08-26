
<!--
help_text: ''
key: summary_conditional_workflows_0ff941a6-55cc-49f8-b9c4-78f1e8e33a51
modules:
- flytekit.core.condition
questions_to_answer: []
type: summary

-->
Conditional Workflows enable dynamic execution paths within a workflow, allowing different branches of logic to run based on specific conditions. This capability is essential for building flexible and adaptive workflows that respond to varying input data or runtime states.

## Defining Conditional Logic

Conditional logic is defined using a fluent API that starts with the `conditional` construct. This construct initiates a `ConditionalSection`, which orchestrates the evaluation and execution of different branches.

A conditional block consists of one or more `if_` branches, optionally followed by `elif_` branches, and can conclude with an `else_` branch.

**Supported Expressions:**
Conditions must be expressed using `ComparisonExpression` (e.g., `==`, `!=`, `<`, `<=`, `>`, `>=`) or `ConjunctionExpression` (`&` for logical AND, `|` for logical OR). Standard Python logical operators (`and`, `or`, `not`) are not supported directly on `Promise` objects; use `&` and `|` instead. Unary expressions like `if_(x)` where `x` is an input value or the output of a previous node are also not supported.

**Example:**

```python
from flytekit import workflow, task, conditional
from flytekit.types.file import FlyteFile
from flytekit.types.directory import FlyteDirectory

@task
def process_data_a(input_val: int) -> str:
    return f"Processed A: {input_val * 2}"

@task
def process_data_b(input_val: int) -> str:
    return f"Processed B: {input_val + 10}"

@task
def handle_error(error_msg: str) -> str:
    return f"Error handled: {error_msg}"

@workflow
def my_conditional_workflow(input_param: int) -> str:
    result = (
        conditional("my_condition")
        .if_(input_param > 10)
        .then(process_data_a(input_param))
        .elif_(input_param == 5)
        .then(process_data_b(input_param))
        .else_()
        .then(handle_error("Input out of range"))
    )
    return result
```

In this example:
- `conditional("my_condition")` starts a new conditional block named "my_condition".
- `.if_(input_param > 10)` defines the first branch, which executes `process_data_a` if `input_param` is greater than 10.
- `.elif_(input_param == 5)` defines an alternative branch, executing `process_data_b` if `input_param` is 5.
- `.else_()` defines the default branch, executing `handle_error` if none of the preceding conditions are met.

## Branch Execution and Outputs

Each branch within a conditional block must specify the logic to execute when its condition is met. This is achieved using the `then()` method of a `Case` object.

The `then()` method accepts a `Promise` or a `Tuple[Promise]` representing the output of a task or sub-workflow call. This output becomes the result of the conditional branch.

**Output Consistency:**
A critical aspect of conditional workflows is ensuring output consistency across all branches. All branches within a single conditional block must produce outputs with the same variable names and types. The system automatically computes the minimum set of common output variables across all defined cases. If branches produce different outputs, only the variables common to all branches will be available as the output of the conditional block. If no common outputs exist, the conditional block will return a `VoidPromise`.

**Error Handling with `fail()`:**
Instead of executing a task, a branch can explicitly fail using the `fail()` method. This is useful for signaling an error condition when a specific branch is taken.

```python
from flytekit import workflow, task, conditional

@task
def success_task() -> str:
    return "Success!"

@workflow
def my_failing_workflow(input_val: int) -> str:
    result = (
        conditional("check_input")
        .if_(input_val < 0)
        .fail("Input cannot be negative!") # This branch explicitly fails
        .else_()
        .then(success_task())
    )
    return result
```

## Compilation vs. Local Execution

Conditional workflows behave differently depending on whether they are being compiled for execution on a Flyte backend or executed locally.

### Compilation Mode

During compilation, the `ConditionalSection` aggregates all defined branches and their associated logic. It does not execute any tasks. Instead, it constructs a single `BranchNode` in the workflow's Directed Acyclic Graph (DAG). This `BranchNode` encapsulates the entire `if-else` block, allowing the Flyte backend to evaluate the conditions and execute the appropriate branch at runtime. The `BranchNode` ensures that the workflow's structure, including conditional logic, is fully represented in the compiled Flyte IDL.

### Local Execution Mode

For local execution, the `LocalExecutedConditionalSection` is used. This specialized section evaluates the conditions immediately and short-circuits the execution. Only the tasks within the first matching branch are executed locally. This behavior is crucial for efficient local testing and debugging, as it avoids unnecessary computation of branches that would not be taken.

The `SkippedConditionalSection` is a further optimization for nested conditionals during local execution. If an outer conditional branch is not taken, any nested conditional sections within that skipped branch will use `SkippedConditionalSection`. This prevents the evaluation and execution of tasks within those nested, unselected branches, further improving local execution performance.

## Key Concepts and Components

-   **`ConditionalSection`**: The core component that manages the entire conditional block. It collects all `Case` definitions, handles the context for compilation, and orchestrates the finalization of the conditional output.
-   **`Case`**: Represents a single `if_`, `elif_`, or `else_` branch within a `ConditionalSection`. It holds the condition expression and the `Promise` (or `Tuple[Promise]`) representing the output of the branch's logic.
-   **`Condition`**: Provides the fluent API (`if_`, `elif_`, `else_`) for defining the branches of a conditional block. It acts as a builder for `Case` objects within a `ConditionalSection`.
-   **`BranchNode`**: The internal representation of a compiled conditional block within the Flyte workflow DAG. It allows the Flyte backend to execute the conditional logic.

## Best Practices and Considerations

-   **Consistent Outputs**: Always strive for consistent output signatures (variable names and types) across all branches of a conditional block. This makes the workflow's output predictable and easier to consume by subsequent tasks. If outputs differ, only the common subset will be available.
-   **Expression Types**: Remember to use `ComparisonExpression` and `ConjunctionExpression` (`&`, `|`) for conditions. Avoid standard Python `and`, `or`, `not` on `Promise` objects.
-   **Nesting**: Conditional workflows can be nested, allowing for complex decision-making logic. Be mindful of the output consistency requirements at each level of nesting.
-   **Debugging**: Leverage local execution for quick iteration and debugging of conditional logic, as it provides immediate feedback by short-circuiting branch execution.
-   **Clarity**: Name your conditional blocks descriptively (e.g., `conditional("validate_input")`) to improve workflow readability and debugging.
<!--
key: summary_conditional_workflows_0ff941a6-55cc-49f8-b9c4-78f1e8e33a51
type: summary_end

-->
<!--
code_unit: flytekit.core.condition.ConditionalSection
code_unit_type: class
help_text: ''
key: example_57c71e54-7d09-4800-9b97-10081f89749a
type: example

-->
<!--
code_unit: flytekit.core.condition.Condition
code_unit_type: class
help_text: ''
key: example_b571a14f-3aae-43bd-8eed-786669dd4a3f
type: example

-->
<!--
code_unit: flytekit.core.condition.Case
code_unit_type: class
help_text: ''
key: example_c880f433-e4b2-42e0-a39d-27106af52af1
type: example

-->