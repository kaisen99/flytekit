
<!--
help_text: ''
key: summary_monitoring_&_observability_ae006c85-b20e-48de-b0c1-ab530377f5cd
modules:
- flytekit.remote.metrics
- flytekit.deck.deck
- flytekit.deck.renderer
- flytekitplugins.deck.renderer
- flytekitplugins.comet_ml.tracking
- flytekitplugins.wandb
- flytekitplugins.wandb.tracking
- flytekitplugins.memray
- flytekitplugins.memray.profiling
- flytekitplugins.whylogs
- flytekitplugins.whylogs.renderer
- flytekitplugins.whylogs.schema
- flytekitplugins.pandera.pandas_renderer
questions_to_answer: []
type: summary

-->
# Monitoring & Observability

Understanding and debugging the behavior of your workflows and tasks is crucial for reliable and efficient operations. This system provides comprehensive capabilities for monitoring execution, observing internal states, and integrating with specialized tools for experiment tracking, resource profiling, and data quality.

## Execution Tracing

The system captures detailed execution traces, allowing you to understand the flow and performance of individual operations within a task.

### Execution Spans

The `FlyteExecutionSpan` object, found in `flytekit.remote.metrics`, represents a segment of execution within a task. Each span records the operation performed, its start and end timestamps, and its duration. This hierarchical data provides a granular view of how time is spent during task execution.

You can inspect execution spans to:
*   Identify performance bottlenecks within specific operations.
*   Understand the sequence of internal calls.
*   Debug unexpected delays or hangs.

To view the execution trace, use the `explain()` method for a human-readable summary or `dump()` for a YAML representation:

```python
from flytekit.remote.metrics import FlyteExecutionSpan

# Assuming 'span_object' is an instance of FlyteExecutionSpan obtained from an execution context
span_object.explain()
span_object.dump()
```

## Rich Observability with Decks

Decks provide a powerful mechanism for generating rich, customizable HTML reports directly within your tasks. These reports offer deep insights into task inputs, outputs, internal states, and various analytical visualizations.

### Core Deck Functionality

The `Deck` class in `flytekit.deck.deck` serves as a container for HTML content. Each task automatically includes default decks for inputs, outputs, and general-purpose content. You can create additional named decks to organize specific types of information.

To add content to a deck, use the `append()` method with an HTML string. The `publish()` static method ensures all accumulated deck content is rendered and made available in the execution UI.

```python
import flytekit
from flytekit.deck.deck import Deck
from flytekit.deck.renderer import MarkdownRenderer

@flytekit.task
def my_observable_task():
    # Append markdown to the default deck
    flytekit.current_context().default_deck.append(MarkdownRenderer().to_html("# Task Insights"))

    # Create a custom deck
    custom_deck = Deck("MyCustomReport")
    custom_deck.append(MarkdownRenderer().to_html("## Detailed Analysis"))

    # Publish all decks (usually handled automatically at task completion)
    Deck.publish()
```

### Specialized Timeline Visualization

The `TimeLineDeck` in `flytekit.deck.deck` is a specialized deck designed to visualize the execution time of different parts of a task. It aggregates time-related information and renders it as a Gantt chart, providing a clear visual representation of task component durations.

Unlike generic decks, `TimeLineDeck` defers HTML generation until its `html` property is accessed, ensuring a complete dataset for meaningful visualizations.

```python
from flytekit.deck.deck import TimeLineDeck
import datetime

@flytekit.task
def timed_task():
    timeline_deck = TimeLineDeck("Task Timeline")
    
    # Simulate timed operations
    start_time_op1 = datetime.datetime.now()
    # ... perform operation 1 ...
    end_time_op1 = datetime.datetime.now()
    timeline_deck.append_time_info({"Start": start_time_op1, "Finish": end_time_op1, "Name": "Operation A"})

    start_time_op2 = datetime.datetime.now()
    # ... perform operation 2 ...
    end_time_op2 = datetime.datetime.now()
    timeline_deck.append_time_info({"Start": start_time_op2, "Finish": end_time_op2, "Name": "Operation B"})

    # The timeline will be rendered when the deck is published
```

### Built-in Renderers

The system provides a suite of renderers to convert common Python data types into HTML for display within decks. These renderers are available in `flytekit.deck.renderer` and `flytekitplugins.deck.renderer`.

*   **`MarkdownRenderer`**: Converts Markdown strings to HTML.
*   **`SourceCodeRenderer`**: Highlights Python source code for display.
*   **`TopFrameRenderer`**: Renders a preview of a Pandas DataFrame as an HTML table.
*   **`ArrowRenderer`**: Renders an Apache Arrow Table as a string.
*   **`PythonDependencyRenderer`**: Generates an HTML table of installed Python packages and their versions.
*   **`FrameProfilingRenderer`**: Creates a comprehensive Pandas profiling report for dataframes.
*   **`ImageRenderer`**: Embeds `FlyteFile` or `PIL.Image.Image` objects as base64-encoded images.
*   **`BoxRenderer`**: Generates Plotly box plots from DataFrame columns.
*   **`TableRenderer`**: Converts a Pandas DataFrame into a customizable HTML table.
*   **`GanttChartRenderer`**: Used internally by `TimeLineDeck` to visualize timelines from DataFrames with start, finish, and name columns.

Example usage of renderers:

```python
import flytekit
import pandas as pd
from flytekit.deck.renderer import MarkdownRenderer, SourceCodeRenderer, TopFrameRenderer
from flytekitplugins.deck.renderer import FrameProfilingRenderer

@flytekit.task
def render_example_task(df: pd.DataFrame):
    ctx = flytekit.current_context()

    # Render Markdown
    ctx.default_deck.append(MarkdownRenderer().to_html("### Data Overview"))

    # Render DataFrame preview
    ctx.default_deck.append(TopFrameRenderer(max_rows=5).to_html(df))

    # Render source code of the task
    with open(__file__, "r") as f:
        task_source = f.read()
    ctx.default_deck.append(SourceCodeRenderer().to_html(task_source))

    # Render a profiling report
    ctx.default_deck.append(FrameProfilingRenderer().to_html(df))
```

### Custom Renderers

For data types not covered by built-in renderers, you can implement the `Renderable` protocol from `flytekit.deck.renderer`. This allows you to define custom logic for converting any Python object into an HTML string for display in a Deck.

```python
from typing import Any, Protocol
from flytekit.deck.deck import Deck

class CustomObject:
    def __init__(self, value: str):
        self.value = value

class CustomRenderer(Protocol):
    def to_html(self, python_value: Any) -> str:
        return f"<div>Custom Object Value: {python_value.value}</div>"

@flytekit.task
def custom_render_task():
    my_custom_obj = CustomObject("Hello, Flyte!")
    Deck("Custom Data").append(CustomRenderer().to_html(my_custom_obj))
```

## Experiment Tracking

The system integrates with popular machine learning experiment tracking platforms, allowing you to log metrics, parameters, and artifacts directly from your tasks.

### Comet ML Integration

The Comet ML integration, provided by `flytekitplugins.comet_ml.tracking._comet_ml_login_class`, enables seamless logging of machine learning experiments to Comet ML. This allows for centralized tracking, visualization, and comparison of model training runs.

Apply this integration as a decorator to your tasks. It handles Comet ML login and experiment initialization, linking the experiment to your task execution.

```python
from flytekit import task, Secret
from flytekitplugins.comet_ml.tracking import _comet_ml_login_class as comet_ml_login

@task
@comet_ml_login(
    project_name="my-ml-project",
    workspace="my-workspace",
    secret=Secret(group="comet-ml", key="COMET_API_KEY"),
    # Optional: experiment_key="my-custom-experiment-id"
)
def train_model_comet():
    import comet_ml
    experiment = comet_ml.Experiment()
    experiment.log_metric("accuracy", 0.95)
    experiment.log_parameter("learning_rate", 0.01)
    # ... training logic ...
```

### Weights & Biases (W&B) Integration

The Weights & Biases (W&B) integration, provided by `flytekitplugins.wandb.tracking.wandb_init`, facilitates logging and visualizing machine learning experiments with W&B. It automatically initializes a W&B run and links it back to the current task execution URL.

Decorate your tasks with this integration to enable W&B tracking.

```python
from flytekit import task, Secret
from flytekitplugins.wandb.tracking import wandb_init

@task
@wandb_init(
    project="my-wandb-project",
    entity="my-wandb-entity",
    secret=Secret(group="wandb", key="WANDB_API_KEY"),
    # Optional: id="my-custom-run-id"
)
def train_model_wandb():
    import wandb
    wandb.log({"loss": 0.1, "epoch": 10})
    # ... training logic ...
```

## Resource Profiling

Understanding resource consumption is vital for optimizing task performance and cost. The system offers specialized tools for memory profiling.

### Memory Profiling with Memray

The `memray_profiling` decorator from `flytekitplugins.memray.profiling` enables detailed memory profiling of your Python tasks using Memray. It captures memory allocation traces and generates interactive HTML reports (flame graphs or tables) that are automatically integrated into your task's decks.

This is particularly useful for identifying memory leaks, excessive memory usage, and understanding the memory footprint of different code paths.

```python
from flytekit import task
from flytekitplugins.memray.profiling import memray_profiling

@task
@memray_profiling(
    memray_html_reporter="flamegraph",  # or "table"
    native_traces=True,
    memory_interval_ms=50,
)
def memory_intensive_task():
    data = [bytearray(1024 * 1024) for _ in range(100)]  # Allocate 100MB
    # ... further processing ...
    del data # Release memory
```

## Data Quality and Validation

Ensuring data quality and detecting data drift are critical for robust machine learning pipelines. The system integrates with tools like whylogs and Pandera for these purposes.

### Whylogs Integration

The whylogs integration provides capabilities for data profiling, quality checks, and drift detection.

*   **`WhylogsDatasetProfileTransformer`**: This component in `flytekitplugins.whylogs.schema` handles the serialization and deserialization of `whylogs.core.view.DatasetProfileView` objects, allowing them to be passed as inputs or outputs between tasks. It also provides a `to_html` method to render a summary report of a dataset profile.

*   **`WhylogsSummaryDriftRenderer`**: Located in `flytekitplugins.whylogs.renderer`, this renderer generates an HTML report comparing two Pandas DataFrames to detect data drift. It profiles both the reference and target datasets and visualizes the differences in their distributions.

    ```python
    import pandas as pd
    from flytekitplugins.whylogs.renderer import WhylogsSummaryDriftRenderer

    @flytekit.task
    def detect_data_drift(reference_df: pd.DataFrame, target_df: pd.DataFrame):
        drift_report_html = WhylogsSummaryDriftRenderer.to_html(reference_df, target_df)
        flytekit.current_context().default_deck.append(drift_report_html)
    ```

*   **`WhylogsConstraintsRenderer`**: Also in `flytekitplugins.whylogs.renderer`, this renderer generates an HTML report detailing the results of data quality constraints defined using whylogs. It visualizes which constraints passed or failed, providing immediate feedback on data quality.

    ```python
    import whylogs as why
    from whylogs.core.constraints import ConstraintsBuilder, MetricConstraint, MetricsSelector
    from flytekitplugins.whylogs.renderer import WhylogsConstraintsRenderer

    @flytekit.task
    def validate_data_quality(df: pd.DataFrame):
        profile_view = why.log(df).view()
        builder = ConstraintsBuilder(profile_view)
        
        # Example constraint: ensure 'sepal_length' is between 0 and 10
        num_constraint = MetricConstraint(
            name='sepal_length_range',
            condition=lambda x: x.min > 0 and x.max < 10,
            metric_selector=MetricsSelector(metric_name='distribution', column_name='sepal_length')
        )
        builder.add_constraint(num_constraint)
        constraints = builder.build()

        constraints_report_html = WhylogsConstraintsRenderer.to_html(constraints)
        flytekit.current_context().default_deck.append(constraints_report_html)
    ```

### Pandera Data Validation

The `PandasReportRenderer` in `flytekitplugins.pandera.pandas_renderer` is designed to generate detailed HTML reports for Pandas DataFrame schema validation. When a DataFrame is validated against a Pandera schema, this renderer can produce a report summarizing validation results, including schema-level and data-level errors.

This report provides a clear overview of data quality issues, including error codes, checks, and failure cases, making it easier to debug data inconsistencies.

```python
import pandas as pd
import pandera as pa
from flytekitplugins.pandera.pandas_renderer import PandasReportRenderer

@flytekit.task
def validate_dataframe(df: pd.DataFrame):
    schema = pa.DataFrameSchema({
        "column_a": pa.Column(int, checks=pa.Check.greater_than(0)),
        "column_b": pa.Column(str, checks=pa.Check.isin(["X", "Y", "Z"])),
    })

    renderer = PandasReportRenderer()
    try:
        schema.validate(df, lazy=True)
        report_html = renderer.to_html(df, schema) # Success report
    except pa.errors.SchemaErrors as err:
        report_html = renderer.to_html(df, schema, err) # Error report
    
    flytekit.current_context().default_deck.append(report_html)
```

## Best Practices

*   **Combine Tools:** For comprehensive observability, integrate execution tracing, custom decks, and specialized profiling/tracking tools. For example, use `FlyteExecutionSpan` for overall task timing, `memray_profiling` for memory details, and `wandb_init` for ML experiment metrics.
*   **Secure Credentials:** Always manage API keys for external services (like Comet ML or W&B) using the system's `Secret` mechanism rather than hardcoding them.
*   **Selective Profiling:** Resource profiling tools like Memray can introduce overhead. Use them judiciously, perhaps during development, testing, or for specific debugging scenarios, rather than on every production run unless critical.
*   **Leverage Decks for Domain-Specific Insights:** Beyond standard data types, use custom renderers and decks to visualize information unique to your domain, such as custom model evaluation plots, domain-specific data quality dashboards, or business metrics.
*   **Enable/Disable Decks:** Be aware of the `enable_deck` flag in the execution context. If decks are not appearing, ensure this flag is set to `True`.
<!--
key: summary_monitoring_&_observability_ae006c85-b20e-48de-b0c1-ab530377f5cd
type: summary_end

-->
<!--
code_unit: flytekit.remote.metrics.FlyteExecutionSpan
code_unit_type: class
help_text: ''
key: example_625275ba-4cdc-4dfe-8cbe-e5619e36786e
type: example

-->
<!--
code_unit: flytekit.deck.deck.Deck
code_unit_type: class
help_text: ''
key: example_103718bd-a80d-4b74-86cb-1d77b7be10db
type: example

-->
<!--
code_unit: flytekitplugins.comet_ml.tracking.comet_ml_init
code_unit_type: class
help_text: ''
key: example_60823e71-3702-4b21-a40c-9d3c1a35b373
type: example

-->
<!--
code_unit: flytekitplugins.wandb.tracking.wandb_init
code_unit_type: class
help_text: ''
key: example_31d0ac44-98b4-4dc6-8189-a5ad2ef3db2a
type: example

-->
<!--
code_unit: flytekitplugins.memray.profiling.memray_profiling
code_unit_type: class
help_text: ''
key: example_f2036d2e-9c46-48f0-bba8-43f7cfa5d6da
type: example

-->
<!--
code_unit: flytekitplugins.whylogs.schema.WhylogsDatasetProfileTransformer
code_unit_type: class
help_text: ''
key: example_25aaa9be-1875-4269-8400-2ed374776ef3
type: example

-->