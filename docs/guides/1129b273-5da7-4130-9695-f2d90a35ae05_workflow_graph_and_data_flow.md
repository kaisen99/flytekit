
<!--
help_text: ''
key: summary_workflow_graph_and_data_flow_6c04e9d3-1990-4ffb-a746-98bda4cddbca
modules:
- flytekit.core.node
- flytekit.core.promise
questions_to_answer: []
type: summary

-->
## Workflow Graph and Data Flow

Workflows are defined as Directed Acyclic Graphs (DAGs) where each computational step is represented by a Node, and data flows between these nodes via Promises. This structure enables the platform to manage dependencies, optimize execution, and provide robust lineage tracking.

### Defining Workflow Steps with Nodes

A `Node` represents a single, executable step within a workflow. It encapsulates the details necessary to execute a task, sub-workflow, or conditional logic.

**Key Characteristics of a Node:**

*   **Unique Identification:** Each node is assigned a unique `id` (accessible via the `name` or `id` property) within its workflow, ensuring clear identification and referencing.
*   **Metadata:** Nodes carry `metadata` that includes configuration such as timeouts, retry strategies, and caching settings.
*   **Bindings:** `bindings` define how inputs to the node are sourced, typically from outputs of upstream nodes or workflow inputs.
*   **Upstream Dependencies:** The `upstream_nodes` property explicitly lists the nodes that must complete successfully before the current node can begin execution.
*   **Associated Entity:** Each node is linked to a `flyte_entity` (e.g., a task, a sub-workflow, or a conditional branch) that it represents. The `run_entity` property provides access to the underlying executable entity.

**Defining Dependencies:**

Dependencies between nodes establish the execution order within the workflow graph. You can define these dependencies using two primary methods:

1.  **`runs_before` Method:** Explicitly declare that one node must run before another.

    ```python
    node_a.runs_before(node_b)
    ```

2.  **Right Shift Operator (`>>`):** A more concise and idiomatic way to express sequential execution. This operator modifies the `upstream_nodes` of the right-hand side node.

    ```python
    node_a >> node_b
    ```

    This means `node_b` will only execute after `node_a` has completed. Chaining these operators allows for complex dependency graphs:

    ```python
    node_a >> node_b >> node_c
    ```

    For parallel execution where multiple nodes depend on a single upstream node:

    ```python
    node_a >> [node_b, node_c]
    ```

    Or, for a node that depends on multiple upstream nodes:

    ```python
    [node_a, node_b] >> node_c
    ```

**Overriding Node Configurations:**

The `with_overrides` method allows you to customize the execution behavior of a specific node instance within a workflow, without altering the underlying task or workflow definition. This is crucial for fine-tuning resource allocation, retry policies, or caching for individual steps.

```python
from datetime import timedelta
from flytekit import Resources, Cache

# Assuming 'my_task' is a defined task and 'my_node' is its invocation within a workflow
my_node = my_task(input_arg="value")

# Override specific settings for 'my_node'
my_node.with_overrides(
    node_name="custom_node_name",  # Change the node's ID in the graph
    aliases={"output_var": "aliased_output"}, # Alias output variables
    requests=Resources(cpu="1", mem="500Mi"), # Request specific resources
    limits=Resources(cpu="2", mem="1Gi"),    # Set resource limits
    timeout=timedelta(minutes=10),           # Set a timeout for the node
    retries=3,                               # Configure retries
    interruptible=True,                      # Allow the node to be interrupted
    cache=Cache(serialize=True, version="v1.0"), # Enable caching with a version
    container_image="my_custom_image:latest", # Use a different container image
)
```

The `resources` parameter provides a consolidated way to specify both `requests` and `limits`. If `resources` is used, `requests` or `limits` should not be set separately.

### Managing Data Flow with Promises

`Promise` objects are fundamental to how data flows through a workflow. When a task or sub-workflow is invoked within a workflow definition, it does not immediately return concrete values. Instead, it returns one or more `Promise` objects. A `Promise` acts as a placeholder for a value that will only become available once the corresponding node has executed.

**Key Characteristics of a Promise:**

*   **Deferred Resolution:** A `Promise` represents a future value. During workflow compilation, it points to the output of an upstream node. During local execution, it may hold the actual computed value.
*   **`is_ready` Property:** This property indicates whether the `Promise` holds a concrete value (`True`) or is still a reference to a future output (`False`).
*   **`val` Property:** If `is_ready` is `True`, `val` contains the actual literal value.
*   **`ref` Property:** If `is_ready` is `False`, `ref` is a `NodeOutput` object, which links the `Promise` to its originating `Node` and the specific output variable name (`var`).

**Accessing Structured Outputs:**

When a task returns multiple outputs or complex data structures (like dataclasses or dictionaries), `Promise` objects allow you to access individual components using standard Python attribute (`.`) or item (`[]`) access.

```python
# Assuming 'my_task' returns a dataclass with fields 'a' and 'b'
# or a dictionary with keys 'a' and 'b'
output_promise = my_task(...)

# Accessing fields/keys on the promise
field_a_promise = output_promise.a
item_b_promise = output_promise["b"]

# Chaining access for nested structures
nested_value_promise = output_promise.some_dict["some_key"].another_field
```

When you access an attribute or item on a `Promise`, a *new* `Promise` object is returned. This new `Promise` includes an `attr_path` that specifies how to resolve the nested value once the upstream node completes. This mechanism ensures that the original `Promise` remains unchanged and can be used elsewhere.

**Limitations of Promises:**

*   **No Direct Truth Value Testing:** `Promise` objects cannot be directly evaluated in boolean contexts (e.g., `if my_promise:`). This will raise a `ValueError`.
*   **No Direct Iteration:** `Promise` objects are not iterable (e.g., `for x in my_promise:`). This will raise a `ValueError`.
*   **Bitwise Logical Operators for Conditionals:** Standard Python logical operators (`and`, `or`) cannot be used directly with `Promise` objects. Instead, use the bitwise operators (`&`, `|`) for constructing conditional expressions.

### Handling Tasks Without Outputs (VoidPromise)

For tasks that do not return any values (i.e., their interface is empty or they return `None`), a special `VoidPromise` object is returned.

*   **Purpose:** `VoidPromise` allows you to define dependencies for tasks that produce no data, ensuring they still participate in the workflow graph's execution order.
*   **Dependency Chaining:** You can use the `>>` operator with `VoidPromise` objects to establish execution order:

    ```python
    void_task_node = my_void_task()
    void_task_node >> next_task(...)
    ```

*   **No Data Operations:** `VoidPromise` objects explicitly disallow any operations that imply a return value or data manipulation (e.g., comparisons, arithmetic operations, attribute access). Attempting these operations will raise an `AssertionError`.

### Conditional Logic in Workflows

Workflows support conditional execution paths based on the values of `Promise` objects. This is achieved through `ComparisonExpression` and `ConjunctionExpression` objects.

*   **Comparison Expressions:** `Promise` objects can be compared using standard comparison operators (`==`, `!=`, `>`, `>=`, `<`, `<=`). These operations do not immediately evaluate but instead create a `ComparisonExpression` object.

    ```python
    # Assuming 'x' is a Promise from an upstream task
    is_greater = x > 10
    is_equal = x == "hello"
    ```

*   **Conjunction Expressions:** Multiple comparison or conjunction expressions can be combined using bitwise logical operators (`&` for AND, `|` for OR) to form complex conditions.

    ```python
    # Combine conditions
    complex_condition = (x > 10) & (x <= 20)
    another_condition = (y == "foo") | (z != "bar")
    ```

These expressions are then used within conditional workflow constructs (e.g., `if` statements in a workflow definition) to determine which branch of the workflow to execute.

**Important Note:** As mentioned, direct `bool()` conversion or standard `and`/`or` operators are not supported for `Promise` or expression objects. Always use the bitwise operators (`&`, `|`) for logical combinations.
<!--
key: summary_workflow_graph_and_data_flow_6c04e9d3-1990-4ffb-a746-98bda4cddbca
type: summary_end

-->
<!--
code_unit: flytekit.core.node.Node
code_unit_type: class
help_text: ''
key: example_427700b9-5403-4e5b-9515-42eaa8962b35
type: example

-->
<!--
code_unit: flytekit.core.promise.Promise
code_unit_type: class
help_text: ''
key: example_a84f6a6c-9b1f-42ca-9fc2-bb505d2ec145
type: example

-->
<!--
code_unit: flytekit.core.promise.ComparisonExpression
code_unit_type: class
help_text: ''
key: example_319899b3-6985-4e2e-bd3f-9447386c3947
type: example

-->
<!--
code_unit: flytekit.core.promise.VoidPromise
code_unit_type: class
help_text: ''
key: example_77b8878f-4930-404a-a6ae-8313d06b96a5
type: example

-->