
<!--
help_text: ''
key: summary_conditional_logic_and_control_flow_ffcd4c0a-f36c-4e3d-8949-9a2f34d1321f
modules:
- flytekit.core.promise
- flytekit.models.core.workflow
questions_to_answer: []
type: summary

-->
Conditional Logic and Control Flow enable dynamic execution paths within workflows, allowing different tasks or sub-workflows to run based on the evaluation of conditions. This capability is crucial for building robust and adaptive data pipelines and machine learning workflows.

### Promises: The Foundation of Dynamic Workflows

At the core of Flyte's conditional logic is the concept of a `Promise`. A `Promise` object represents the *future* output of a task or workflow node. Unlike immediate values, a `Promise` is a placeholder that will eventually resolve to a concrete value during workflow execution. This deferred evaluation is fundamental to how Flyte constructs a directed acyclic graph (DAG) of operations.

When a task or workflow is invoked within a workflow definition, it returns `Promise` objects instead of direct Python values. These `Promise` objects carry metadata about their origin (the node that produces them) and the variable they represent.

*   **`Promise`**: The primary class for representing a future value. It allows for operations like comparison and chaining, which are essential for building conditions.
    *   `is_ready`: Indicates if the `Promise` has been resolved to a concrete value (typically during local execution).
    *   `val`: If `is_ready` is `True`, this property holds the resolved literal value.
    *   `ref`: If `is_ready` is `False`, this property holds a `NodeOutput` reference to the originating node and variable.
    *   `__getitem__` and `__getattr__`: `Promise` objects support attribute and item access (e.g., `my_promise.some_field` or `my_promise[0]`). This allows referencing nested outputs from complex types (like dataclasses or lists) produced by upstream tasks. These operations return new `Promise` objects with an updated `attr_path`, indicating the specific nested value to be resolved.

*   **`NodeOutput`**: A specialized `Promise` that explicitly references an output variable from a specific `Node` in the workflow graph. It links the output of one node to the input of another, forming dependencies.

*   **`VoidPromise`**: For tasks that do not return any outputs (i.e., their interface is empty), a `VoidPromise` is returned. This object explicitly disallows any operations (comparisons, logical operations, arithmetic) to prevent misuse, as there is no value to operate on. Attempting to use a `VoidPromise` in an expression will raise an `AssertionError`.

### Building Conditions with Expressions

Conditions in Flyte workflows are constructed using `ComparisonExpression` and `ConjunctionExpression` objects. These expressions are declarative; they define *how* a condition should be evaluated, rather than evaluating it immediately.

#### Comparison Expressions

A `ComparisonExpression` represents a single comparison between two operands. Operands can be `Promise` objects (referencing future values) or literal Python values.

*   **`ComparisonExpression`**:
    *   Takes a left-hand side (`lhs`), a `ComparisonOps` operator, and a right-hand side (`rhs`).
    *   `lhs` and `rhs` can be `Promise` objects or any Python value that can be converted to a Flyte literal.
    *   The `eval()` method is primarily for local execution, resolving the `Promise` objects to their concrete values and performing the comparison.

*   **`ComparisonOps`**: An enumeration defining the supported comparison operators:
    *   `EQ` (`==`): Equal to
    *   `NE` (`!=`): Not equal to
    *   `GT` (`>`): Greater than
    *   `GE` (`>=`): Greater than or equal to
    *   `LT` (`<`): Less than
    *   `LE` (`<=`): Less than or equal to

**Usage Example:**

When you compare a `Promise` object with another `Promise` or a literal, Flytekit automatically creates a `ComparisonExpression`.

```python
from flytekit import task, workflow
from flytekit.core.promise import Promise

@task
def get_number() -> int:
    return 5

@task
def check_number(num: int) -> bool:
    return num > 3

@workflow
def my_conditional_workflow():
    x = get_number()
    # This creates a ComparisonExpression: (x > 3)
    condition_expr = x > 3
    # condition_expr is an instance of ComparisonExpression, not a boolean
    # It represents the condition to be evaluated at runtime.
```

`Promise` objects also provide convenience methods for common comparisons:
*   `is_(v: bool)`: Equivalent to `self == v`.
*   `is_false()`: Equivalent to `self == False`.
*   `is_true()`: Equivalent to `self == True`.
*   `is_none()`: Equivalent to `self == None`.

#### Logical (Conjunction) Expressions

A `ConjunctionExpression` combines two other expressions (either `ComparisonExpression` or other `ConjunctionExpression`s) using a logical operator.

*   **`ConjunctionExpression`**:
    *   Takes a left-hand side expression (`lhs`), a `ConjunctionOps` operator, and a right-hand side expression (`rhs`).
    *   The `eval()` method performs the logical operation on the evaluated truth values of its constituent expressions.

*   **`ConjunctionOps`**: An enumeration defining the supported logical operators:
    *   `AND` (`&`): Logical AND
    *   `OR` (`|`): Logical OR

**Important Considerations for Expressions:**

*   **Bitwise Operators for Logical Operations**: Due to Python's language limitations (specifically PEP-335), you *must* use the bitwise `&` (AND) and `|` (OR) operators to combine `ComparisonExpression` or `ConjunctionExpression` objects. Do *not* use the Python keywords `and` and `or`, as these attempt to perform immediate truth-value testing, which is not supported for `Promise` or expression objects.
    *   `Promise` and expression objects explicitly raise a `ValueError` if `__bool__` (truth value testing) is attempted directly.
    *   They also raise `ValueError` if Python's `and` or `or` keywords are used.

**Usage Example:**

```python
from flytekit import task, workflow
from flytekit.core.promise import Promise

@task
def get_value_a() -> int:
    return 10

@task
def get_value_b() -> int:
    return 20

@workflow
def complex_conditional_workflow():
    a = get_value_a()
    b = get_value_b()

    # Correct way to combine conditions using bitwise operators
    condition_1 = (a > 5) & (b < 25)  # ConjunctionExpression: (Comp(a > 5) AND Comp(b < 25))
    condition_2 = (a == 10) | (b == 15) # ConjunctionExpression: (Comp(a == 10) OR Comp(b == 15))

    # Incorrect: This will raise a ValueError
    # invalid_condition = (a > 5) and (b < 25)
```

### Implementing Control Flow with Branch Nodes

Flyte translates Python's `if/elif/else` constructs within a workflow definition into a `BranchNode` in the compiled workflow graph. A `BranchNode` encapsulates an `IfElseBlock`, which defines the sequence of conditions and their corresponding execution paths.

*   **`Node`**: The fundamental building block of a Flyte workflow. A `Node` can represent a task, a sub-workflow, or a branch.
*   **`BranchNode`**: A special type of `Node` that enables conditional execution. It contains an `IfElseBlock`.
*   **`IfElseBlock`**: Defines the complete conditional structure:
    *   `case`: The initial `IfBlock` containing the first condition and the node to execute if it's true.
    *   `other`: An optional list of additional `IfBlock`s, representing `elif` clauses.
    *   `else_node`: An optional `Node` to execute if none of the preceding conditions are met (the `else` clause).
    *   `error`: An optional error to raise if no conditions are met and no `else_node` is specified.
*   **`IfBlock`**: Associates a `condition` (a `BooleanExpression` like `ComparisonExpression` or `ConjunctionExpression`) with a `then_node` (the `Node` to execute if the condition evaluates to true).

**Usage Example:**

```python
from flytekit import task, workflow, conditional

@task
def process_data_a(input_val: int) -> str:
    return f"Processed A: {input_val}"

@task
def process_data_b(input_val: int) -> str:
    return f"Processed B: {input_val}"

@task
def handle_default(input_val: int) -> str:
    return f"Default handling for: {input_val}"

@workflow
def my_conditional_workflow(input_num: int) -> str:
    return (
        conditional("check_input_num")
        .if_((input_num > 10) & (input_num < 20))  # Condition using Promise and logical AND
        .then(process_data_a(input_val=input_num))
        .elif_(input_num >= 20)  # Another condition
        .then(process_data_b(input_val=input_num))
        .else_()  # The 'else' clause
        .then(handle_default(input_val=input_num))
        .end()
    )

# When input_num = 15, process_data_a runs.
# When input_num = 25, process_data_b runs.
# When input_num = 5, handle_default runs.
```

In this example:
1.  `conditional("check_input_num")` initiates a conditional block, creating a `BranchNode`.
2.  `.if_((input_num > 10) & (input_num < 20))` defines the first `IfBlock` with a `ConjunctionExpression` as its condition.
3.  `.then(process_data_a(...))` specifies the `Node` (in this case, a task node) to execute if the `if_` condition is true.
4.  `.elif_(input_num >= 20)` adds another `IfBlock` for an `elif` clause.
5.  `.else_().then(handle_default(...))` defines the final `else_node` to execute if no preceding conditions are met.
6.  `.end()` finalizes the conditional block.

The output of a conditional block is the output of the chosen branch's node. All branches within a conditional block must produce outputs of the same type and shape to ensure a consistent workflow interface.

### Practical Usage and Best Practices

*   **Declarative Nature**: Remember that conditional logic in Flyte is declarative. You are defining the *structure* of the decision-making process, not executing it immediately. This allows Flyte to compile a complete workflow graph before execution.
*   **Use `&` and `|`**: Always use the bitwise `&` and `|` operators for logical AND and OR operations on `Promise` objects and expressions. Avoid Python's `and` and `or` keywords.
*   **Consistent Outputs**: Ensure all branches of a conditional block (`if`, `elif`, `else`) return outputs that conform to the same type signature. This is crucial for the workflow's type safety and for downstream nodes to correctly consume the conditional output.
*   **Local Execution for Debugging**: The `eval()` method on `Promise` and expression objects, along with Flytekit's local execution capabilities, allows you to test and debug conditional logic without deploying to a Flyte cluster.
*   **Simplicity**: While complex conditions are possible, strive for clarity. Break down very complex conditions into smaller, more manageable expressions or even separate tasks that compute boolean flags.
*   **Performance**: Conditional logic itself adds minimal overhead to workflow execution. The primary cost comes from the execution of the chosen branch's tasks or sub-workflows.
*   **Error Handling**: The `.else_().fail("error message")` construct provides a way to explicitly fail a workflow if none of the conditions are met, which can be useful for enforcing business logic or data validation.
<!--
key: summary_conditional_logic_and_control_flow_ffcd4c0a-f36c-4e3d-8949-9a2f34d1321f
type: summary_end

-->
<!--
code_unit: flytekit.core.promise
code_unit_type: class
help_text: ''
key: example_59e56355-6801-42db-bab7-8988a464c4f1
type: example

-->
<!--
code_unit: flytekit.models.core.workflow
code_unit_type: class
help_text: ''
key: example_8a9b846a-aaa3-4b4d-abe9-06ebcb477898
type: example

-->