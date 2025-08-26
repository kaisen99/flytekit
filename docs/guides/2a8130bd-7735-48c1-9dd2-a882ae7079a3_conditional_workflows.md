
<!--
help_text: ''
key: summary_conditional_workflows_e7806d52-8869-4833-9337-b6bcd61c5de3
modules:
- flytekit.core.condition
questions_to_answer: []
type: summary

-->
Conditional Workflows enable dynamic execution paths within workflows based on runtime conditions. This capability allows for flexible and adaptive workflows that can respond to varying inputs or intermediate results, leading to more efficient and robust data pipelines.

## Defining Conditional Logic

Conditional logic is constructed using a fluent API that mirrors standard `if/elif/else` constructs. Each branch of a conditional block must specify the operations to perform if its condition is met.

To initiate a conditional block, use the `conditional` function, providing a unique name for the block. This returns a `Condition` object, which exposes methods for defining branches.

### `if_` Branch

The `if_` method defines the initial condition. It accepts an expression that evaluates to a boolean.

```python
from flytekit import conditional, workflow, task
from flytekit.types.literal import Literal

@task
def add_one(x: int) -> int:
    return x + 1

@task
def subtract_one(x: int) -> int:
    return x - 1

@workflow
def my_conditional_workflow(input_val: int) -> int:
    result = (
        conditional("my_condition")
        .if_(input_val > 0)
        .then(add_one(x=input_val))
    )
    return result
```

The `if_` method returns a `Case` object, which then allows chaining with the `then` method to specify the operations for that branch.

### `elif_` Branch

To add additional conditions, use the `elif_` method. Like `if_`, it takes an expression and is followed by a `then` clause.

```python
@workflow
def my_conditional_workflow(input_val: int) -> int:
    result = (
        conditional("my_condition")
        .if_(input_val > 0)
        .then(add_one(x=input_val))
        .elif_(input_val == 0) # Chaining an elif_
        .then(subtract_one(x=input_val))
    )
    return result
```

### `else_` Branch

The `else_` method defines a default branch that executes if none of the preceding `if_` or `elif_` conditions are met. It does not take an expression.

```python
@workflow
def my_conditional_workflow(input_val: int) -> int:
    result = (
        conditional("my_condition")
        .if_(input_val > 0)
        .then(add_one(x=input_val))
        .elif_(input_val == 0)
        .then(subtract_one(x=input_val))
        .else_() # The else_ branch
        .then(add_one(x=input_val * 2)) # Example: double and add one
    )
    return result
```

### Supported Expressions

Conditions must be built using `ComparisonExpression` (e.g., `==`, `!=`, `<`, `<=`, `>`, `>=`) and `ConjunctionExpression` (`&` for AND, `|` for OR).

**Supported:**
```python
# Comparison
input_val > 0
input_val == 10

# Conjunction
(input_val > 0) & (input_val < 100)
(input_val == 0) | (input_val == 10)
```

**Unsupported:**
Direct Python logical operators (`and`, `or`, `not`, `is`) are not supported on `Promise` objects. For example, `if input_val > 0 and input_val < 100` is invalid. Always use `&` and `|` for logical operations on expressions involving workflow inputs or task outputs.

Unary expressions of the form `if_(x)` where `x` is a workflow input or task output (a `Promise`) are also not supported. Conditions must be explicit comparisons or conjunctions.

### Branch Execution with `then`

The `then` method of a `Case` object specifies the operations to execute when that branch's condition is true. It accepts a `Promise` or a `Tuple[Promise]`, typically the output of a task or a literal value.

```python
# ... (tasks defined above)

@workflow
def example_workflow(x: int) -> int:
    # If x is positive, add one. Otherwise, subtract one.
    output = (
        conditional("positive_check")
        .if_(x > 0)
        .then(add_one(x=x))
        .else_()
        .then(subtract_one(x=x))
    )
    return output
```

### Handling Branch Failures with `fail`

A branch can explicitly indicate a failure using the `fail` method of a `Case` object. This immediately terminates the workflow execution with the provided error message.

```python
@workflow
def validate_input_workflow(x: int) -> int:
    output = (
        conditional("input_validation")
        .if_(x < 0)
        .fail("Input value cannot be negative.") # Workflow will fail here
        .else_()
        .then(add_one(x=x))
    )
    return output
```

## Conditional Workflow Outputs

A conditional block can produce outputs that are then used by subsequent nodes in the workflow. The output of a conditional block is determined by the outputs of its individual branches.

### Consistent Outputs

For a conditional block to produce a well-defined output, all branches (`if_`, `elif_`, `else_`) must produce a consistent set of outputs. This means:
*   If a conditional block is expected to return a single output, all branches must return a single output of a compatible type.
*   If a conditional block is expected to return multiple outputs (e.g., a `NamedTuple`), all branches must return the same number of outputs with the same names and compatible types.

The system automatically determines the least common set of output variables across all branches using the `compute_output_vars` method of `ConditionalSection`. If branches have different output structures, only the common variables are exposed as the conditional block's output.

```python
from typing import NamedTuple

@task
def task_a(x: int) -> int:
    return x + 1

@task
def task_b(x: int) -> int:
    return x * 2

class MyOutputs(NamedTuple):
    y: int
    z: str

@task
def task_c(x: int) -> MyOutputs:
    return MyOutputs(y=x + 10, z="hello")

@task
def task_d(x: int) -> MyOutputs:
    return MyOutputs(y=x - 5, z="world")

@workflow
def mixed_output_workflow(input_val: int) -> int:
    # This conditional block will only output 'y' because 'z' is not common
    # across all branches (task_a and task_b don't produce 'z').
    # In this specific example, task_a and task_b return int, while task_c and task_d return NamedTuple.
    # The common output will be an empty set, leading to a VoidPromise.
    # For a successful common output, all branches must return the same structure.
    # Let's correct the example to show common outputs.

    # Example with consistent outputs:
    output = (
        conditional("consistent_output")
        .if_(input_val > 5)
        .then(task_a(x=input_val)) # Returns int
        .else_()
        .then(task_b(x=input_val)) # Returns int
    )
    return output # This will be an int
```

### `VoidPromise` Implications

If any branch within a conditional block returns no output (e.g., a task that returns `None` or a `VoidPromise`), or if the `compute_output_vars` method determines there are no common outputs across all branches, the entire conditional block will produce a `VoidPromise`. This indicates that the conditional block does not yield any usable output for downstream nodes.

## Execution Modes

Conditional workflows behave differently depending on whether they are executed locally or compiled for remote execution.

### Compilation Mode (`ConditionalSection`)

When a workflow is compiled for remote execution, the `ConditionalSection` class is used. In this mode, the conditional logic is translated into an `IfElseBlock` within the Flyte graph. No actual code execution of the branches occurs during compilation. Instead, the system captures the potential execution paths and their associated outputs.

The `end_branch` method of `ConditionalSection` is responsible for:
1.  Marking the completion of a branch definition.
2.  If it's the last branch (`else_`), it pops the current context from the `FlyteContextManager`.
3.  It then creates a `Node` in the workflow graph that represents the entire conditional block. This node's outputs are `Promise` objects linked to the common outputs determined across all branches.

### Local Execution Mode (`LocalExecutedConditionalSection`)

For local execution (e.g., during testing or local development), the `LocalExecutedConditionalSection` class is employed. This class overrides the behavior of `ConditionalSection` to enable immediate evaluation of conditions and execution of the selected branch.

The `start_branch` method in `LocalExecutedConditionalSection` evaluates the condition (`c.expr.eval()`) directly. If a condition is met, that branch is marked as the `_selected_case`, and subsequent `elif_` or `else_` conditions are short-circuited (not evaluated).

The `end_branch` method then returns the actual output `Promise` (or `Tuple[Promise]`) from the `_selected_case`, allowing the local execution to proceed with the concrete values.

### Nested Conditionals and `SkippedConditionalSection`

When conditional blocks are nested, the `FlyteContextManager` manages the execution context. If an outer conditional branch evaluates to `false` during local execution, any nested conditional blocks within that branch are handled by `SkippedConditionalSection`. This ensures that tasks within the skipped branches are not executed locally, preventing unnecessary computations. The `SkippedConditionalSection`'s `end_branch` method returns `Promise` objects with `None` values for its outputs, effectively bypassing execution.

## Best Practices and Considerations

*   **Naming:** Always provide a descriptive name to your conditional blocks using `conditional("your_block_name")`. This improves readability and debugging.
*   **Exhaustive Conditions:** Ensure your `if_`/`elif_`/`else_` structure covers all possible input scenarios. Omitting an `else_` branch can lead to unexpected behavior if none of the `if_`/`elif_` conditions are met.
*   **Output Consistency:** Design your branches to produce consistent outputs. If branches yield different output structures, the conditional block will only expose the intersection of those outputs, or a `VoidPromise` if no common outputs exist.
*   **Complexity:** While powerful, overly complex nested conditionals can reduce readability. Consider breaking down very complex logic into smaller, more manageable conditional blocks or using sub-workflows.
*   **Debugging:** During local execution, you can inspect the `_selected_case` attribute of `LocalExecutedConditionalSection` to understand which branch was taken.
<!--
key: summary_conditional_workflows_e7806d52-8869-4833-9337-b6bcd61c5de3
type: summary_end

-->
<!--
code_unit: flytekit.core.condition.ConditionalSection.if_
code_unit_type: class
help_text: ''
key: example_0aebb724-b743-488f-84b1-44e83ff8884a
type: example

-->
<!--
code_unit: flytekit.core.condition.Condition.elif_
code_unit_type: class
help_text: ''
key: example_a9ca3063-4a70-4bd5-82cd-44290414e4e5
type: example

-->
<!--
code_unit: flytekit.core.condition.Condition.else_
code_unit_type: class
help_text: ''
key: example_463d7c11-ff26-49b7-bedd-15df96b927b5
type: example

-->