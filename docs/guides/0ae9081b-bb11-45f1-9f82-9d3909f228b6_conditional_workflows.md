
<!--
help_text: ''
key: summary_conditional_workflows_2626a367-a72a-433c-851e-7f6be88dcc9c
modules:
- flytekit.core.condition
questions_to_answer: []
type: summary

-->
Conditional Workflows enable dynamic execution paths within a workflow based on runtime conditions. This capability allows for more flexible and adaptive workflows, where different branches of logic are executed depending on input values or the results of preceding tasks.

### Defining Conditional Logic

Conditional logic is defined using a fluent API that mirrors standard `if/elif/else` constructs. A conditional block begins with a named `conditional` construct, followed by one or more `if_` and `elif_` statements, and optionally an `else_` statement. Each conditional branch concludes with a `then` clause, which specifies the operations to perform if that branch is taken.

The `Condition` class provides the `if_`, `elif_`, and `else_` methods, which are chained to define the conditional structure. Each of these methods returns a `Case` object, allowing you to specify the actions for that particular branch.

**Example:**

```python
from flytekit import workflow, task, conditional
from flytekit.types.literal import Literal

@task
def process_positive(x: int) -> str:
    return f"Processing positive: {x}"

@task
def process_negative(x: int) -> str:
    return f"Processing negative: {x}"

@task
def process_zero() -> str:
    return "Processing zero"

@workflow
def my_conditional_workflow(input_val: int) -> str:
    result = (
        conditional("check_value")
        .if_(input_val > 0)
        .then(process_positive(x=input_val))
        .elif_(input_val < 0)
        .then(process_negative(x=input_val))
        .else_()
        .then(process_zero())
    )
    return result
```

In this example:
*   `conditional("check_value")` initiates a new conditional section, managed by the `ConditionalSection` class.
*   `if_(input_val > 0)` defines the first branch. The expression `input_val > 0` must be a `ComparisonExpression` or `ConjunctionExpression`.
*   `.then(process_positive(x=input_val))` specifies the task to execute if the condition is true. The output of `process_positive` becomes the output of this branch.
*   `elif_` and `else_` follow the same pattern for subsequent branches.

### Conditional Expressions

Expressions used in `if_` and `elif_` statements must be either `ComparisonExpression` (e.g., `a > b`, `x == y`, `c != d`) or `ConjunctionExpression` (e.g., `(a > b) & (x < y)`, `(c == d) | (e >= f)`).

**Unsupported Expressions:**

*   **Boolean Literals**: Directly using `True` or `False` as an expression is not supported, as conditions are evaluated at runtime or during compilation, not as static Python booleans.
*   **Unary Expressions on Promises**: An expression like `if_(my_input)` where `my_input` is a `Promise` (an input or output from a previous task) is not allowed. The expression must involve a comparison or conjunction.

The `Case` class validates these expression types during the definition of the conditional block.

### Branch Outputs and Consistency

A critical aspect of conditional workflows is ensuring consistency in outputs across all branches. All branches within a conditional block must produce the same set of outputs, both in terms of variable names and types. If a branch does not naturally produce an output, or if it produces a different set of outputs, the system will attempt to reconcile them.

The `ConditionalSection.compute_output_vars` method determines the common output variables across all defined `Case` objects. If no common outputs are found, or if any branch explicitly returns a `VoidPromise` (indicating no output), the entire conditional block will be treated as having no output.

**Example of consistent outputs:**

```python
from flytekit import workflow, task, conditional

@task
def branch_a_task(x: int) -> int:
    return x * 2

@task
def branch_b_task(x: int) -> int:
    return x + 10

@workflow
def consistent_output_workflow(val: int) -> int:
    output = (
        conditional("my_condition")
        .if_(val > 5)
        .then(branch_a_task(x=val))
        .else_()
        .then(branch_b_task(x=val))
    )
    return output
```

In this example, both `branch_a_task` and `branch_b_task` return a single integer, ensuring consistency.

### Execution Modes

Conditional workflows behave differently depending on the execution context:

#### Compilation Mode

During workflow compilation (when the workflow graph is being built for remote execution), the conditional logic is translated into a `BranchNode`. This node represents the `IfElseBlock` structure in the underlying workflow definition. Instead of executing the branches, the system captures the potential outputs of each branch as `Promise` objects.

The `ConditionalSection.end_branch` method, when in compilation mode, creates a `Node` in the workflow graph. This node's outputs are `Promise` objects that will resolve to the actual branch output at runtime on the Flyte engine. This ensures that the workflow graph is fully defined before execution, allowing the Flyte engine to optimize and manage the conditional execution.

#### Local Execution Mode

When a workflow is executed locally (e.g., for testing or development), the `LocalExecutedConditionalSection` class takes over. In this mode, the conditional expressions are evaluated directly using `c.expr.eval()`. The system then "takes" the first branch whose condition evaluates to `True`.

The `LocalExecutedConditionalSection.start_branch` method performs this evaluation and selects the appropriate `Case`. The `LocalExecutedConditionalSection.end_branch` method then directly returns the output of the selected branch, effectively short-circuiting the other branches and executing only the chosen path. This provides immediate feedback and simplifies local debugging.

#### Skipped Conditional Sections

For nested conditional workflows, or when a parent conditional branch is not taken, the `SkippedConditionalSection` ensures that unexecuted branches do not attempt to run local tasks or produce meaningful outputs. If a conditional section is skipped, its `end_branch` method will return `VoidPromise` or `Promise` objects with `None` values, indicating that no actual computation occurred within that section. This prevents unnecessary resource allocation or errors for unselected paths.

### Error Handling

The `Case` class provides a `fail` method, allowing you to explicitly define an error path for a branch. While the current implementation primarily sets an error string, this mechanism can be extended to trigger specific error handling logic or propagate custom error messages.

```python
from flytekit import workflow, task, conditional

@task
def my_task(x: int) -> int:
    if x < 0:
        raise ValueError("Input cannot be negative")
    return x * 2

@workflow
def error_handling_workflow(val: int) -> int:
    result = (
        conditional("check_input")
        .if_(val < 0)
        .fail("Input value must be non-negative.") # This branch fails the workflow
        .else_()
        .then(my_task(x=val))
    )
    return result
```

Using `fail` immediately terminates the workflow with the specified error message if that branch is taken.

### Best Practices and Considerations

*   **Keep Conditions Simple**: Complex conditions can be hard to read and debug. Break down very complex logic into smaller, chained conditional blocks or helper functions that return simple boolean expressions.
*   **Consistent Outputs are Key**: Always ensure that all branches of a conditional block produce the same output signature (number of variables, their names, and types). This is crucial for the workflow graph to be well-defined and for downstream tasks to correctly consume the conditional output.
*   **Understand Execution Modes**: Be aware of the differences between compilation and local execution. Local execution provides quick feedback, but the ultimate behavior on the Flyte engine is determined by the compiled graph.
*   **Performance**: While conditionals add flexibility, excessive nesting or very complex conditions might add a slight overhead during graph compilation. For most use cases, this impact is negligible.
*   **Avoid Side Effects in Conditions**: The expressions used in `if_` and `elif_` should ideally be pure functions of their inputs, without side effects, as they might be evaluated multiple times or in different contexts (e.g., during local execution).
<!--
key: summary_conditional_workflows_2626a367-a72a-433c-851e-7f6be88dcc9c
type: summary_end

-->
<!--
code_unit: flytekit.core.condition.ConditionalSection
code_unit_type: class
help_text: ''
key: example_988c1cdc-1894-415e-9ed1-b1ddebcee5a5
type: example

-->
<!--
code_unit: flytekit.core.condition.Condition
code_unit_type: class
help_text: ''
key: example_b5f09bda-59c2-4e0b-ab67-b40bd748597a
type: example

-->
<!--
code_unit: flytekit.core.condition.Case
code_unit_type: class
help_text: ''
key: example_eb8be392-1126-40f1-9c7c-e04e2adc83cf
type: example

-->