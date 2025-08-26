
<!--
help_text: ''
key: summary_advanced_workflow_constructs_d05c36ea-d3ef-4a17-9a65-c6bc0e9b897a
modules:
- flytekit.core.condition
- flytekit.core.array_node
- flytekit.core.array_node_map_task
- flytekit.core.legacy_map_task
- flytekit.core.gate
- flytekit.core.schedule
questions_to_answer: []
type: summary

-->
Advanced Workflow Constructs

This section details advanced constructs available for building sophisticated and robust workflows. These constructs enable dynamic behavior, parallel processing, human intervention, and scheduled executions within your workflows.

### Conditional Workflows

Conditional workflows allow you to introduce branching logic into your execution paths, enabling different tasks or sub-workflows to run based on specific conditions. This provides flexibility and control, allowing workflows to adapt to varying inputs or intermediate results.

**Core Concepts**

At the heart of conditional workflows are the `ConditionalSection` and `Case` constructs.
*   A `ConditionalSection` represents the entire `if/elif/else` block. It manages the sequence of conditions and their corresponding actions.
*   A `Case` defines a single branch within the conditional section, comprising an expression and the operations to execute if that expression evaluates to true.

**Defining Conditions**

Conditions are defined using the `conditional` helper, followed by `if_`, `elif_`, and `else_` clauses. Each clause takes an expression and a `then` block, which specifies the task or workflow to execute.

```python
from flytekit import conditional, task, workflow

@task
def process_positive(x: int) -> str:
    return f"Processing positive: {x}"

@task
def process_negative(x: int) -> str:
    return f"Processing negative: {x}"

@task
def process_zero(x: int) -> str:
    return f"Processing zero: {x}"

@workflow
def my_conditional_workflow(value: int) -> str:
    result = (
        conditional("check_value")
        .if_(value > 0)
        .then(process_positive(x=value))
        .elif_(value < 0)
        .then(process_negative(x=value))
        .else_()
        .then(process_zero(x=value))
    )
    return result
```

**Supported Expressions**

Conditional expressions support standard comparison operators (`<`, `<=`, `>`, `>=`, `==`, `!=`) and logical conjunctions (`&` for AND, `|` for OR).

```python
from flytekit import conditional, task, workflow

@task
def handle_range(x: float) -> str:
    return f"Value {x} is in range (0.1, 1.0)"

@task
def handle_out_of_range(x: float) -> str:
    return f"Value {x} is out of range"

@workflow
def range_check_workflow(my_input: float) -> str:
    # Using conjunction expression
    result = (
        conditional("fractions")
        .if_((my_input > 0.1) & (my_input < 1.0))
        .then(handle_range(x=my_input))
        .else_()
        .then(handle_out_of_range(x=my_input))
    )
    return result
```

**Important Considerations**

*   **Workflow Context**: Conditions can only be used within a workflow definition.
*   **Expression Limitations**: Unary expressions (e.g., `if_(x)` where `x` is an input or promise) are not supported. Expressions must be explicit comparisons or conjunctions.
*   **Output Consistency**: All branches within a conditional block must produce outputs of the same type(s). If a branch does not produce an output, it should explicitly return `None` or a `VoidPromise` to maintain consistency. The `compute_output_vars` method on `ConditionalSection` ensures that the minimum set of common outputs is determined across all cases.
*   **Error Handling**: A `Case` can also use `fail(err: str)` to explicitly fail the workflow with a given error message if a condition is met.

### Map Tasks

Map tasks enable the parallel execution of a single task over a collection of inputs. This is highly efficient for batch processing and scaling computations.

**Core Concepts**

The `ArrayNode` represents the mapped node in the workflow graph, while `ArrayNodeMapTask` is the specialized task type that handles the mapping logic. The `ArrayNodeMapTaskResolver` facilitates the serialization and deserialization of these tasks, especially when dealing with partially bound inputs.

**Capabilities**

*   **Parallel Execution**: Run a single task function across a list of inputs concurrently.
*   **Concurrency Control**: Limit the number of parallel executions using the `concurrency` parameter. If set to `0`, concurrency is unbounded.
*   **Minimum Success Criteria**: Define `min_successes` (absolute count) or `min_success_ratio` (percentage) to determine the minimum number of successful sub-tasks required for the map task to be considered successful. If the threshold is not met, the map task fails.
*   **Partial Input Binding**: Use `functools.partial` to bind some inputs of the underlying task to fixed values, while mapping over others. The system automatically infers which inputs are bound and which should be mapped as lists.
*   **Interface Transformation**: The `ArrayNodeMapTask` automatically transforms the interface of the underlying task. For example, if the original task takes `(i: int, j: str)`, the map task will expose an interface like `(i: List[int], j: List[str])`. If `min_success_ratio` is set and not 1.0, outputs will be `List[Optional[T]]`.

**Defining a Map Task**

Use the `map_task` decorator to define a map task.

```python
from flytekit import map_task, task, workflow
from typing import List, Optional

@task
def square(x: int) -> int:
    return x * x

@workflow
def map_square_workflow(numbers: List[int]) -> List[int]:
    # This will run the 'square' task for each number in the 'numbers' list in parallel
    squared_numbers = map_task(square)(x=numbers)
    return squared_numbers

# Example with concurrency and min_success_ratio
@task
def process_item(item: str) -> Optional[str]:
    # Simulate some failures
    if item == "fail":
        raise ValueError("Simulated failure")
    return f"Processed {item}"

@workflow
def resilient_map_workflow(items: List[str]) -> List[Optional[str]]:
    # Run with max 2 concurrent tasks, allow 70% success
    results = map_task(process_item, concurrency=2, min_success_ratio=0.7)(item=items)
    return results
```

**Partial Input Binding Example**

```python
import functools
from flytekit import map_task, task, workflow
from typing import List

@task
def greet(name: str, greeting: str) -> str:
    return f"{greeting}, {name}!"

@workflow
def partial_map_workflow(names: List[str]) -> List[str]:
    # Bind 'greeting' to a fixed value, map over 'name'
    partial_greet_task = functools.partial(greet, greeting="Hello")
    greetings = map_task(partial_greet_task)(name=names)
    return greetings
```

**Important Considerations**

*   **Single Output**: Only tasks with a single output are supported for mapping.
*   **Local Execution**: During local execution, the `_raw_execute` method of `ArrayNodeMapTask` processes all inputs sequentially and collects all outputs. On the Flyte platform, individual instances of the map task (array task jobs) are executed, each producing a single output, which the array plugin handler then aggregates into a collection.
*   **Performance**: Map tasks are designed for high parallelism and are ideal for processing large datasets where each item can be processed independently.

### Gates

Gates introduce control points in a workflow, allowing for pauses, human intervention, or external input before proceeding. They behave like tasks but do not execute user-defined code.

**Types of Gates**

*   **Sleep Gate**: Pauses the workflow for a specified duration. This is useful for introducing delays or waiting for external systems.
*   **Input Gate**: Halts the workflow and waits for a user to provide input of a specified type. The provided input becomes the output of the gate node.
*   **Approval Gate**: Pauses the workflow and waits for explicit user approval. This is often used to gate critical operations based on the output of an upstream task.

**Defining Gates**

Gates are created using the `gate` helper.

```python
from flytekit import gate, task, workflow
from datetime import timedelta
from typing import List

@task
def prepare_report(data: List[str]) -> str:
    return f"Report prepared for: {', '.join(data)}"

@task
def send_email(report_summary: str) -> str:
    return f"Email sent with summary: {report_summary}"

@workflow
def gate_workflow(data_items: List[str], approval_needed: bool = False) -> str:
    report = prepare_report(data=data_items)

    # Example 1: Sleep Gate
    # Pause for 5 seconds before proceeding
    gate.sleep(name="wait_for_data_sync", duration=timedelta(seconds=5))

    # Example 2: Input Gate
    # Wait for user to provide a comment string
    user_comment = gate.input("user_feedback", input_type=str)

    # Example 3: Approval Gate (conditional)
    if approval_needed:
        # Wait for approval based on the report summary
        gate.approve(name="review_report", upstream_item=report)

    final_status = send_email(report_summary=report)
    return final_status
```

**Local Execution Behavior**

*   **Sleep Gate**: During local execution, a sleep gate will print a message indicating the simulated sleep duration.
*   **Input Gate**: During local execution, an input gate will prompt the user for input via the command line.
*   **Approval Gate**: During local execution, an approval gate will prompt the user for confirmation via the command line. If the user declines, a `FlyteDisapprovalException` is raised.

**Important Considerations**

*   **Timeout**: Gates can be configured with a `timeout` to automatically fail if the required input or approval is not received within a specified duration.
*   **Upstream Items**: For approval gates, the `upstream_item` parameter is used to pass the output of a previous node, which can then be displayed to the user for context during the approval prompt.

### Schedules

Schedules enable the recurring execution of workflows (Launch Plans) at predefined intervals or times. This is crucial for automating routine data processing, reporting, or maintenance tasks.

**Core Concepts**

Schedules are defined using `OnSchedule`, which wraps either a `CronSchedule` for cron-based scheduling or a `FixedRate` for interval-based scheduling.

**Types of Schedules**

*   **Cron Schedule**: Provides flexible scheduling using cron expressions. It supports both standard 5-field croniter parseable expressions and AWS-style 6-field expressions. It also supports common cron aliases.
*   **Fixed Rate Schedule**: Defines a schedule that runs at a fixed interval (e.g., every 10 minutes, every day).

**Defining Schedules**

Schedules are typically associated with a `LaunchPlan`.

```python
from flytekit import workflow, LaunchPlan
from flytekit.core.schedule import CronSchedule, FixedRate
from datetime import timedelta, datetime
from typing import List

@workflow
def daily_report_workflow(report_date: datetime) -> str:
    return f"Generating daily report for {report_date.strftime('%Y-%m-%d')}"

# Example 1: Daily Cron Schedule
daily_lp = LaunchPlan.create(
    "daily_report_lp",
    daily_report_workflow,
    schedule=OnSchedule(
        schedule=CronSchedule(
            schedule="0 0 * * *",  # Every day at midnight UTC
            kickoff_time_input_arg="report_date" # Pass kickoff time to workflow input
        )
    )
)

@workflow
def hourly_data_ingestion(data_source: str) -> str:
    return f"Ingesting data from {data_source}"

# Example 2: Fixed Rate Schedule
hourly_lp = LaunchPlan.create(
    "hourly_ingestion_lp",
    hourly_data_ingestion,
    schedule=OnSchedule(
        schedule=FixedRate(
            duration=timedelta(hours=1) # Every hour
        )
    ),
    default_inputs={"data_source": "external_api"}
)
```

**`kickoff_time_input_arg`**

The `kickoff_time_input_arg` parameter in `CronSchedule` and `FixedRate` allows you to inject the actual time the scheduled run was initiated into a workflow input. This is useful for time-sensitive workflows that need to know their exact start time.

**Important Considerations**

*   **Cron Expression Format**: For `CronSchedule`, use the `schedule` parameter for 5-field croniter parseable expressions or cron aliases. The deprecated `cron_expression` parameter was for AWS-style 6-field expressions. Ensure that for 6-field expressions, either the day-of-month or day-of-week field is `?`.
*   **Fixed Rate Granularity**: `FixedRate` schedules support granularity down to minutes, hours, or days. Sub-minute intervals are not supported.
*   **Time Zones**: Schedules are typically interpreted in UTC by the Flyte platform. Consider this when defining cron expressions.
<!--
key: summary_advanced_workflow_constructs_d05c36ea-d3ef-4a17-9a65-c6bc0e9b897a
type: summary_end

-->
<!--
code_unit: flytekit.core.condition
code_unit_type: class
help_text: ''
key: example_e1ba5994-7c5d-4faf-8db3-426c25ff0166
type: example

-->
<!--
code_unit: flytekit.core.array_node_map_task
code_unit_type: class
help_text: ''
key: example_56627e45-1f0d-4619-b9d6-7c23648a22e0
type: example

-->
<!--
code_unit: flytekit.core.gate
code_unit_type: class
help_text: ''
key: example_4696cdc5-cb17-465f-b551-e6499630a2c0
type: example

-->