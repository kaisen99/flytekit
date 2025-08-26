
<!--
help_text: ''
key: summary_monitoring_task_execution_and_performance_252b0ebf-9235-497b-912a-17cde4a6ef47
modules:
- flytekit.remote.metrics
- flytekit.core.base_task
- flytekit.core.context_manager
questions_to_answer: []
type: summary

-->
Monitoring task execution and performance is crucial for debugging, optimizing, and understanding the behavior of your workflows. Flytekit provides a comprehensive set of tools and capabilities to observe, analyze, and configure the runtime characteristics of your tasks. This includes built-in HTML reports (Decks), mechanisms for custom metrics and logging, detailed execution span analysis, and configurable task metadata for performance and resilience.

### Observing Task Execution with Decks

Decks are interactive HTML reports generated during task execution, offering a visual summary of a task's inputs, outputs, timeline, source code, and dependencies. They are invaluable for quickly understanding what happened during a task run, especially during local development and debugging.

**Enabling and Configuring Decks**
Decks can be enabled and configured directly within your task definitions.
*   To enable decks for a task, set `enable_deck=True` in the task decorator or `PythonTask` constructor.
*   You can specify which types of information to include in the deck using the `deck_fields` parameter, which accepts a tuple of `DeckField` enum values. Common fields include `DeckField.INPUT`, `DeckField.OUTPUT`, `DeckField.TIMELINE`, `DeckField.SOURCE_CODE`, and `DeckField.DEPENDENCIES`.

```python
from flytekit import task
from flytekit.deck import DeckField

@task(enable_deck=True, deck_fields=(DeckField.INPUT, DeckField.OUTPUT, DeckField.TIMELINE))
def my_monitored_task(a: int, b: int) -> int:
    # Task logic here
    return a + b
```

**Accessing Decks**
During local execution, you can retrieve the generated deck content using the `get_deck()` method of the `FlyteContext` object. This method returns an HTML string or an IPython display object, suitable for rendering in notebooks.

```python
from flytekit import task, current_context
from flytekit.deck import DeckField
from IPython.display import HTML # For notebook display, if running in an IPython environment

@task(enable_deck=True, deck_fields=(DeckField.INPUT, DeckField.OUTPUT))
def example_deck_task(x: int, y: int) -> int:
    print(f"Executing with x={x}, y={y}")
    return x * y

if __name__ == "__main__":
    # Local execution to generate a deck
    result = example_deck_task(x=5, y=10)

    # Access the deck from the current context
    deck_content = current_context().get_deck()

    # In a Jupyter notebook, you would simply display it:
    # HTML(deck_content)
    # For console output, you might save it to a file:
    with open("example_deck.html", "w") as f:
        f.write(str(deck_content))
    print("Deck saved to example_deck.html")
```

The `_write_decks` method within the `PythonTask` class is responsible for orchestrating the generation and writing of these HTML reports based on the configured `deck_fields`.

### Emitting Custom Metrics and Logs

For more granular monitoring and debugging, tasks can emit custom metrics and logs. The `ExecutionParameters` object, accessible via `current_context()`, provides dedicated handles for these purposes.

**Custom Metrics**
The `stats` property of the `ExecutionParameters` object provides a `taggable.TaggableStats` instance, allowing you to emit custom metrics (e.g., counters, gauges, timers) that can be collected and visualized by your monitoring system. These metrics are automatically tagged with relevant execution metadata (e.g., workflow name, task name).

```python
from flytekit import task, current_context

@task
def process_data(data_size: int) -> float:
    ctx = current_context()
    
    # Emit a counter metric
    ctx.stats.incr("data_processed_count")
    
    # Emit a gauge metric
    ctx.stats.gauge("current_data_size", data_size)
    
    # Measure execution time
    with ctx.stats.timer("data_processing_time"):
        # Simulate data processing
        import time
        time.sleep(data_size / 100.0)
        processed_result = float(data_size * 2)
    
    return processed_result
```

**Structured Logging**
The `logging` property of the `ExecutionParameters` object provides a standard Python `logging.Logger` instance. This logger is pre-configured with execution-specific context, ensuring that your log messages are automatically enriched with details like the execution ID and task ID, making them easier to trace in centralized logging systems.

```python
from flytekit import task, current_context

@task
def analyze_results(results: dict):
    ctx = current_context()
    ctx.logging.info(f"Starting analysis for results: {results.keys()}")
    
    if not results:
        ctx.logging.warning("No results provided for analysis.")
        return
    
    for key, value in results.items():
        ctx.logging.debug(f"Analyzing key: {key}, value: {value}")
        # Perform analysis
    
    ctx.logging.info("Analysis complete.")
```

**Best Practices:**
*   **Granularity:** Emit metrics and logs at appropriate levels of granularity to provide actionable insights without excessive overhead.
*   **Tagging:** Leverage the automatic tagging provided by the `stats` object. For custom tags, ensure consistency.
*   **Context:** Always use `current_context().logging` and `current_context().stats` to ensure your monitoring data is correctly associated with the Flyte execution.

### Analyzing Execution Spans

Execution spans provide a detailed, hierarchical view of the time spent within different operations during a task's lifecycle. These spans are captured internally by Flytekit and can be inspected for deep performance analysis.

The `FlyteExecutionSpan` object represents a single execution span. It encapsulates timing information and the operation performed.

**Inspecting Spans**
After an execution, you can retrieve and analyze `FlyteExecutionSpan` objects.
*   The `explain()` method provides a human-readable, indented output of the span hierarchy, showing operations, start/end timestamps, and durations. This is useful for quick visual inspection.
*   The `dump()` method outputs the span information in a YAML format, suitable for programmatic parsing or more detailed analysis.

```python
# Example of how FlyteExecutionSpan might be used for post-execution analysis.
# Direct access to FlyteExecutionSpan from a running task is not typical;
# it's usually part of the execution metadata retrieved from the Flyte backend
# after a workflow or task has completed.

from flytekit.remote.metrics import FlyteExecutionSpan
from datetime import datetime, timedelta

# This is a conceptual representation of a Span protobuf object,
# as you would typically deserialize it from the Flyte backend's execution data.
class MockSpanPB:
    def __init__(self, operation_name, start_time, end_time, spans=None):
        self.operation_name = operation_name
        self.start_time = start_time
        self.end_time = end_time
        self.spans = spans if spans is not None else []

# Conceptual creation of a Span PB hierarchy
span_pb_example = MockSpanPB(
    operation_name="my_workflow_execution",
    start_time=datetime.now(),
    end_time=datetime.now() + timedelta(seconds=15),
    spans=[
        MockSpanPB(
            operation_name="task_one_execution",
            start_time=datetime.now() + timedelta(seconds=1),
            end_time=datetime.now() + timedelta(seconds=6),
            spans=[
                MockSpanPB("input_translation", datetime.now() + timedelta(seconds=1), datetime.now() + timedelta(seconds=2)),
                MockSpanPB("user_code_execution", datetime.now() + timedelta(seconds=2), datetime.now() + timedelta(seconds=5)),
                MockSpanPB("output_translation", datetime.now() + timedelta(seconds=5), datetime.now() + timedelta(seconds=6)),
            ]
        ),
        MockSpanPB(
            operation_name="task_two_execution",
            start_time=datetime.now() + timedelta(seconds=7),
            end_time=datetime.now() + timedelta(seconds=14),
        ),
    ]
)

# Create a FlyteExecutionSpan object from the conceptual protobuf
flyte_span = FlyteExecutionSpan(span=span_pb_example)

print("--- Explaining Span ---")
flyte_span.explain()

print("\n--- Dumping Span (YAML) ---")
flyte_span.dump()
```
The `timeit` calls within the `dispatch_execute` method of `PythonTask` are key internal mechanisms that contribute to the generation of these execution spans, capturing the duration of critical phases like input translation and user code execution.

### Configuring Task Performance and Resilience

Task metadata allows you to configure various aspects of task execution that directly impact performance, resource utilization, and resilience. These settings are defined using the `TaskMetadata` object, which is part of the `Task` and `PythonTask` definitions.

**Key Configuration Parameters:**
*   **`timeout`**: Specifies the maximum duration a task execution is allowed to run. If the task exceeds this duration, it will be terminated. This prevents runaway tasks and helps manage resource consumption.
    *   Type: `datetime.timedelta` or `int` (seconds).
*   **`retries`**: Defines the number of times a task should be retried upon failure. This enhances the resilience of your workflows against transient errors.
    *   Type: `int` (number of retries).
*   **`cache`**: Enables caching for the task. If enabled, Flyte will store the outputs of the task and reuse them for subsequent executions with identical inputs, significantly improving performance for idempotent tasks.
    *   Type: `bool`.
*   **`cache_version`**: A required string version when caching is enabled. Changing this version invalidates previous cached results.
    *   Type: `str`.
*   **`cache_serialize`**: When `True` and caching is enabled, identical task instances will execute in serial. This can be useful for tasks that access shared, non-concurrent resources.
    *   Type: `bool`.
*   **`cache_ignore_input_vars`**: A tuple of input variable names that should be excluded when calculating the cache hash. This is useful if certain inputs (e.g., timestamps, ephemeral IDs) do not affect the task's output and should not prevent cache hits.
    *   Type: `Tuple[str, ...]`.
*   **`interruptible`**: Indicates that the task can be interrupted and potentially scheduled on lower-cost, pre-emptible compute instances. This can lead to cost savings but might result in longer execution times or more retries.
    *   Type: `Optional[bool]`.

These parameters are set via the `metadata` argument in the `@task` decorator or `PythonTask` constructor.

```python
from flytekit import task, TaskMetadata
import datetime

@task(
    metadata=TaskMetadata(
        retries=3,
        timeout=datetime.timedelta(minutes=5),
        cache=True,
        cache_version="v1.0",
        cache_ignore_input_vars=("debug_mode",),
        interruptible=True,
    )
)
def long_running_task(input_data: str, debug_mode: bool = False) -> str:
    # Simulate a long-running, potentially flaky operation
    import random, time
    if random.random() < 0.1:
        raise ValueError("Simulated transient error")
    time.sleep(10)
    return f"Processed {input_data}"
```

**Considerations:**
*   **Caching:** Ensure tasks are truly idempotent before enabling caching. Incorrect caching can lead to stale results.
*   **Retries:** Use retries for transient failures. For persistent errors, retries will only delay the inevitable.
*   **Interruptible:** Balance cost savings with potential increases in execution time or failure rates.

### Attaching Output Metadata

Flytekit allows you to attach arbitrary metadata to the output literals of your tasks. This capability, managed by the `OutputMetadataTracker`, is useful for enriching your data with information about its quality, lineage, or any custom metrics relevant to the specific output.

The `OutputMetadataTracker` is accessible via `current_context().output_metadata_tracker`. You can use its `add()` method to associate an `OutputMetadata` object with a specific Python object that will be returned as a task output. The `OutputMetadata` object can contain dynamic partitions, time partitions, and additional serializable items.

```python
from flytekit import task, current_context
from flytekit.core.context_manager import OutputMetadata, SerializableToString
from datetime import datetime

# Custom serializable metadata item
class DataQualityReport(SerializableToString):
    def __init__(self, score: float, checks_passed: int):
        self.score = score
        self.checks_passed = checks_passed

    def serialize_to_string(self, ctx, variable_name: str) -> tuple[str, str]:
        return f"data_quality_report_{variable_name}", f"Score: {self.score}, Checks Passed: {self.checks_passed}"

@task
def generate_report(data: dict) -> dict:
    ctx = current_context()
    
    # Simulate report generation
    report_content = {"summary": "Report generated successfully", "data_points": len(data)}
    
    # Attach custom metadata to the 'report_content' output
    output_metadata = OutputMetadata(
        artifact=None, # Artifact is not directly used here for simplicity
        dynamic_partitions={"region": "us-east-1", "date": datetime.now().strftime("%Y-%m-%d")},
        time_partition=datetime.now(),
        additional_items=[DataQualityReport(score=0.95, checks_passed=10)]
    )
    ctx.output_metadata_tracker.add(report_content, output_metadata)
    
    return {"report": report_content}

# When this task runs, the 'report' output literal will carry the attached metadata,
# which can be inspected downstream or by the Flyte platform.
```
The `_output_to_literal_map` method within `PythonTask` is responsible for iterating through task outputs and attaching any metadata registered with the `OutputMetadataTracker` to the corresponding literal.

### Limitations and Considerations

*   **Local vs. Remote Execution:** While many monitoring features (like decks, logging, and metrics) are designed to work seamlessly across local and remote environments, some behaviors might differ. For instance, `FlyteExecutionSpan` objects are primarily for post-execution analysis on the Flyte platform, not direct introspection during a local task run.
*   **Overhead:** Extensive logging, metric emission, or very large decks can introduce minor overhead. Balance the need for observability with performance requirements.
*   **Context Management:** The `FlyteContextManager` is an internal component responsible for managing the execution context stack. Developers should primarily interact with the context through `current_context()` and avoid direct manipulation of the context stack.
*   **Data Volume:** Be mindful of the volume of data generated by decks or detailed logs, especially for large-scale workflows.
<!--
key: summary_monitoring_task_execution_and_performance_252b0ebf-9235-497b-912a-17cde4a6ef47
type: summary_end

-->
<!--
code_unit: flytekit.remote.metrics.FlyteExecutionSpan
code_unit_type: class
help_text: ''
key: example_90fbc770-7fb3-4bcd-809d-49d4bfb3fc24
type: example

-->
<!--
code_unit: flytekit.core.base_task.TaskMetadata
code_unit_type: class
help_text: ''
key: example_67e69238-5e93-4db2-b1e0-7efba5f841fc
type: example

-->
<!--
code_unit: flytekit.core.base_task.PythonTask
code_unit_type: class
help_text: ''
key: example_c5afb807-f82f-4cd7-85ba-b3413d3378f4
type: example

-->
<!--
code_unit: flytekit.core.context_manager.ExecutionParameters
code_unit_type: class
help_text: ''
key: example_5fd97208-0481-414e-8afe-0737ad97f3be
type: example

-->