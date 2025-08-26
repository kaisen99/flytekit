
<!--
help_text: ''
key: summary_resource_and_data_profiling_0e9ca83f-4a16-4d65-8cb1-cecb755975ff
modules:
- flytekitplugins.memray
- flytekitplugins.memray.profiling
- flytekitplugins.deck.renderer
questions_to_answer: []
type: summary

-->
# Resource and Data Profiling

Flyte provides robust capabilities for understanding the resource consumption and data characteristics of your tasks. This includes detailed memory profiling to identify and optimize memory usage, and comprehensive data profiling to gain insights into data quality and distribution. All profiling reports are seamlessly integrated into FlyteDeck for interactive visualization.

## Memory Profiling with Memray

Memory profiling helps identify memory allocation patterns, potential leaks, and opportunities for optimization within your Flyte tasks.

### Capabilities

The `memray_profiling` decorator enables memory profiling for any Flyte task. It leverages the `memray` library to capture detailed memory allocation traces and generate interactive HTML reports. Key capabilities include:

*   **Native and Python Tracing**: Capture memory allocations from both native C extensions and pure Python code.
*   **Fork Tracking**: Continue tracking memory usage in subprocesses created via `fork`.
*   **Configurable Interval**: Adjust the frequency of resident set size updates for granular memory usage graphs.
*   **Interactive Reports**: Generate `memray` flamegraphs or table reports, which are automatically embedded into FlyteDeck.

### Usage

To profile a task's memory, apply the `memray_profiling` decorator to your task function. Configure the profiling behavior using the decorator's arguments.

```python
from flytekit import task, workflow
from flytekitplugins.memray.profiling import memray_profiling
import numpy as np

@memray_profiling(
    native_traces=True,
    memray_html_reporter="flamegraph",
    memray_reporter_args=["--leaks"] # Example: pass --leaks to the flamegraph reporter
)
@task
def my_memory_intensive_task(size: int) -> int:
    """
    A task that allocates a large NumPy array to demonstrate memory profiling.
    """
    data = np.zeros((size, size), dtype=np.float64)
    # Perform some operations that might allocate more memory
    _ = data.sum()
    return data.nbytes

@workflow
def mem_profiling_wf(size: int = 1000):
    my_memory_intensive_task(size=size)
```

When `mem_profiling_wf` executes, the `my_memory_intensive_task` will be profiled. A `memray` binary file will be generated, and an HTML report (e.g., a flamegraph) will be created from it. This HTML report is then automatically rendered within a dedicated tab in the task's FlyteDeck output.

### Configuration Parameters

The `memray_profiling` decorator accepts the following arguments:

*   `native_traces` (bool): If `True`, captures native stack frames in addition to Python stack frames. Useful for profiling C extensions.
*   `trace_python_allocators` (bool): If `True`, traces Python allocators as independent allocations.
*   `follow_fork` (bool): If `True`, continues tracking memory in subprocesses created by `fork`.
*   `memory_interval_ms` (int): Interval in milliseconds between periodic resident set size updates. Defaults to 10ms.
*   `memray_html_reporter` (str): Specifies the type of HTML report to generate. Supported values are `"flamegraph"` and `"table"`.
*   `memray_reporter_args` (List[str], optional): A list of additional arguments to pass directly to the `memray` reporter command (e.g., `["--leaks"]` for flamegraph).

## Data Profiling with YData-Profiling

Data profiling provides a comprehensive overview of your dataset's quality, structure, and statistical properties. This is invaluable for data validation, anomaly detection, and understanding data distributions before further processing.

### Capabilities

The `FrameProfilingRenderer` facilitates the generation of detailed data profiles using the `ydata-profiling` library. It produces an extensive HTML report that includes:

*   **Overview**: Essential statistics like number of variables, observations, missing cells, and duplicate rows.
*   **Variables**: Type inference, unique values, missing values, and descriptive statistics for each column.
*   **Interactions**: Scatter plots for numerical variable pairs.
*   **Correlations**: Various correlation matrices (Pearson, Spearman, Kendall, Phik).
*   **Missing Values**: Visualizations of missing data patterns.
*   **Samples**: Head and tail of the dataset.
*   **Duplicates**: Identification of duplicate rows.

Other renderers within the `flytekitplugins.deck.renderer` package, such as `TableRenderer`, `BoxRenderer`, and `GanttChartRenderer`, can also be used to visualize specific aspects of data or task execution within FlyteDeck.

### Usage

To generate a data profile, create a pandas DataFrame within your task and use the `Deck` utility in conjunction with `FrameProfilingRenderer` to embed the report.

```python
from flytekit import task, workflow
from flytekit.deck import Deck
import pandas as pd
from flytekitplugins.deck.renderer import FrameProfilingRenderer

@task
def profile_my_data(num_rows: int) -> str:
    """
    A task that generates a DataFrame and creates a data profiling report.
    """
    data = {
        "numerical_col_1": [i * 1.5 for i in range(num_rows)],
        "numerical_col_2": [float(i % 10) + (i / 100.0) for i in range(num_rows)],
        "categorical_col": [f"category_{i % 3}" for i in range(num_rows)],
        "boolean_col": [bool(i % 2) for i in range(num_rows)],
        "missing_data_col": [None if i % 5 == 0 else i for i in range(num_rows)],
    }
    df = pd.DataFrame(data)

    # Generate and embed the data profile report
    Deck("Data Profile Report", FrameProfilingRenderer().to_html(df))

    # You can also add other visualizations
    # Deck("Sample Data Table", TableRenderer().to_html(df.head()))

    return "Data profiling report generated and embedded in FlyteDeck."

@workflow
def data_profiling_wf(num_rows: int = 100):
    profile_my_data(num_rows=num_rows)
```

When `data_profiling_wf` executes, the `profile_my_data` task will generate a comprehensive data profile report. This report will appear as a new tab in the task's FlyteDeck output, providing an interactive and detailed analysis of the DataFrame.

## Integration with FlyteDeck

Both memory and data profiling reports are seamlessly integrated into FlyteDeck. This means that after a task completes, you can access the interactive reports directly within the Flyte UI, typically under a dedicated "Deck" tab for the task. This eliminates the need for manual file downloads and external tools, streamlining the analysis and debugging process.

## Considerations and Best Practices

*   **Performance Overhead**: Profiling introduces some computational overhead. Use profiling judiciously, especially in production environments or for tasks processing extremely large datasets.
*   **Granularity vs. Overhead**: For memory profiling, adjust `memory_interval_ms` to balance the detail of the memory usage graph with the performance impact.
*   **Report Size**: Comprehensive data profiles or very long memory traces can result in large HTML files. For very large datasets, consider profiling a representative sample rather than the entire dataset to manage report size and generation time.
*   **Customization**: Leverage the `memray_reporter_args` parameter in `memray_profiling` to pass specific arguments to the `memray` reporters, allowing for fine-grained control over the generated memory reports (e.g., focusing on memory leaks).
*   **Iterative Profiling**: Start with broad profiling, then narrow down to specific sections or smaller data samples as you identify areas for deeper investigation.
<!--
key: summary_resource_and_data_profiling_0e9ca83f-4a16-4d65-8cb1-cecb755975ff
type: summary_end

-->
<!--
code_unit: flytekitplugins.memray.memray_profiling
code_unit_type: class
help_text: ''
key: example_5a3a2ef2-0ae9-4a5c-ae04-cb56b51fce0b
type: example

-->
<!--
code_unit: flytekitplugins.memray.profiling.memray_profiling
code_unit_type: class
help_text: ''
key: example_7cda8e0c-a1c1-4cb2-8b8f-9c0f800fd890
type: example

-->
<!--
code_unit: flytekitplugins.deck.renderer.FrameProfilingRenderer
code_unit_type: class
help_text: ''
key: example_18fc38d1-b587-43ef-954d-47240eea790b
type: example

-->