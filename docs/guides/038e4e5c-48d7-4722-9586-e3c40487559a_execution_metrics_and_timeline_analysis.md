
<!--
help_text: ''
key: summary_execution_metrics_and_timeline_analysis_840bd41b-122a-467e-8049-1a77cba1108b
modules:
- flytekit.remote.metrics
- flytekit.deck.deck
- flytekitplugins.deck.renderer
questions_to_answer: []
type: summary

-->
Execution Metrics and Timeline Analysis provides tools to capture, inspect, and visualize the temporal aspects of task execution within Flyte workflows. This capability is crucial for understanding performance characteristics, identifying bottlenecks, and gaining detailed insights into the lifecycle of operations. It leverages Flyte's extensible Deck feature to render interactive timeline visualizations directly alongside task outputs.

### Capturing Execution Metrics

The `FlyteExecutionSpan` object is the fundamental unit for capturing detailed execution metrics. Each `FlyteExecutionSpan` represents a specific operation or phase within a task's execution, recording its start time, end time, and duration. These spans can form a hierarchical structure, allowing for granular analysis of nested operations.

To inspect a `FlyteExecutionSpan` directly:

*   **`explain()`**: Prints a human-readable, tabular summary of the span, including the operation name, start and end timestamps, duration, and the associated entity. This is useful for quick console debugging.

    ```python
    # Assuming 'span_obj' is an instance of FlyteExecutionSpan
    span_obj.explain()
    ```

*   **`dump()`**: Generates a detailed YAML representation of the span and its aggregated information. This provides a comprehensive view of the collected metrics, suitable for programmatic parsing or in-depth analysis.

    ```python
    # Assuming 'span_obj' is an instance of FlyteExecutionSpan
    span_obj.dump()
    ```

The underlying data structure for `FlyteExecutionSpan` is a Protobuf `Span` object, facilitating efficient serialization and deserialization for remote communication and persistence within the Flyte system.

### Visualizing Timelines with `TimeLineDeck`

The `TimeLineDeck` component is specifically designed to render the execution flow of a task as a visual timeline. Unlike general-purpose decks, `TimeLineDeck` defers its HTML generation until all relevant time information is collected. This ensures that the resulting visualization is comprehensive and provides meaningful insights into the complete execution duration of various task components.

To populate a `TimeLineDeck` with execution data:

*   **`append_time_info(info: dict)`**: Use this method to add individual time entries to the deck. Each `info` dictionary should contain the necessary data points for a timeline entry, typically including `Start` (start time), `Finish` (end time), and a descriptive `Name` for the operation.

The `TimeLineDeck` automatically leverages the `GanttChartRenderer` to transform the collected time information into an interactive Gantt chart. This chart visually represents each operation as a bar, with its length corresponding to its duration and its position indicating its start and end times.

When the `html` property of `TimeLineDeck` is accessed, it generates the complete HTML content, including the Gantt chart and supplementary notes. The `DeckField.TIMELINE` enum member indicates that timeline information is a standard field that can be rendered within a deck.

### Integrating Timeline Analysis into Tasks

Integrating timeline analysis into a Flyte task involves creating a `TimeLineDeck` instance and populating it with relevant execution data. This deck is then automatically published as part of the task's outputs, accessible via the Flyte UI.

**Common Use Case: Profiling Task Sub-components**

Consider a task that performs several distinct processing steps. You can use `TimeLineDeck` to visualize the duration of each step:

```python
import time
from flytekit import task, current_context
from flytekit.deck.deck import TimeLineDeck

@task
def analyze_data_pipeline(input_data: str) -> str:
    # Create a TimeLineDeck instance. It automatically adds itself to the current context's decks.
    timeline_deck = TimeLineDeck("Task Execution Timeline")

    results = []

    # Step 1: Data Loading
    start_time_load = time.time()
    time.sleep(0.5) # Simulate data loading
    end_time_load = time.time()
    timeline_deck.append_time_info({
        "Start": start_time_load,
        "Finish": end_time_load,
        "Name": "Data Loading"
    })
    results.append("Data loaded.")

    # Step 2: Preprocessing
    start_time_preprocess = time.time()
    time.sleep(1.2) # Simulate preprocessing
    end_time_preprocess = time.time()
    timeline_deck.append_time_info({
        "Start": start_time_preprocess,
        "Finish": end_time_preprocess,
        "Name": "Data Preprocessing"
    })
    results.append("Data preprocessed.")

    # Step 3: Model Inference
    start_time_inference = time.time()
    time.sleep(0.8) # Simulate model inference
    end_time_inference = time.time()
    timeline_deck.append_time_info({
        "Start": start_time_inference,
        "Finish": end_time_inference,
        "Name": "Model Inference"
    })
    results.append("Inference complete.")

    # The deck is automatically published when the task completes
    return "\n".join(results)
```

In this example, `time.time()` is used for simplicity to demonstrate populating the `TimeLineDeck`. In a real Flyte environment, more precise timing mechanisms or integration with Flyte's internal span collection might be used to automatically populate `FlyteExecutionSpan` objects, which could then be converted into the format expected by `TimeLineDeck`.

### Considerations and Best Practices

*   **Granularity**: Determine the appropriate level of detail for your spans. Too fine-grained measurements can introduce overhead, while too coarse-grained measurements may hide critical bottlenecks. Focus on logical phases or significant operations within your task.
*   **Time Measurement Accuracy**: The `TimeLineDeck` provides a visual representation. For highly accurate performance measurements, always refer to system-level wall time and process time metrics, which are typically available through Flyte's monitoring integrations. The timeline visualization is best used for relative comparisons and identifying long-running operations.
*   **Small Durations**: Operations with very short durations (e.g., less than 1 millisecond) may be difficult to discern on the generated timeline graph due to visualization limitations.
*   **Accessing Output**: The generated timeline HTML is part of the task's output deck. It is typically accessible through the Flyte UI for the specific task execution.
*   **Dependencies**: The `GanttChartRenderer` relies on `plotly.express` for generating the interactive Gantt chart. `TimeLineDeck` handles this dependency internally for its default rendering, but if you are manually using renderers, ensure necessary visualization libraries are available in your task's execution environment.
<!--
key: summary_execution_metrics_and_timeline_analysis_840bd41b-122a-467e-8049-1a77cba1108b
type: summary_end

-->
<!--
code_unit: flytekit.remote.metrics.FlyteExecutionSpan
code_unit_type: class
help_text: ''
key: example_36d59ef0-6369-4d46-a801-31c7ebd45ff2
type: example

-->
<!--
code_unit: flytekit.deck.deck.TimeLineDeck
code_unit_type: class
help_text: ''
key: example_c829a1a8-0db5-4f98-bc9e-3e9c3a65c8fd
type: example

-->
<!--
code_unit: flytekitplugins.deck.renderer.GanttChartRenderer
code_unit_type: class
help_text: ''
key: example_191fb8df-0745-4955-8fa0-dd67d15f4cc9
type: example

-->