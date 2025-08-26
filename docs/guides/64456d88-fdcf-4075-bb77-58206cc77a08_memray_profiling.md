
<!--
help_text: ''
key: summary_memray_profiling_cfd6487a-3ab2-43d4-b1fb-231e2171f3d3
modules:
- flytekitplugins.memray
- flytekitplugins.memray.profiling
questions_to_answer: []
type: summary

-->
Memray Profiling

Memray Profiling provides a mechanism to analyze the memory consumption of tasks, helping identify memory leaks and optimize resource usage. This feature integrates with the `memray` library to capture detailed memory allocation traces and generate interactive HTML reports, which are then displayed directly within FlyteDeck.

### Usage

To enable memory profiling for a task, apply the `memray_profiling` decorator to the task function. This decorator wraps the task's execution, initiating memory tracking and report generation.

```python
from flytekitplugins.memray.profiling import memray_profiling
from flytekit import task, workflow

@memray_profiling(memray_html_reporter="flamegraph")
@task
def my_memory_intensive_task(n: int) -> list:
    """
    A task that allocates a significant amount of memory.
    """
    data = [i for i in range(n)]
    return data

@workflow
def my_profiling_workflow():
    my_memory_intensive_task(n=1000000)

```

### Configuration

The `memray_profiling` decorator offers several parameters to customize the profiling behavior:

*   **`native_traces`** (`bool`, default: `False`): When set to `True`, `memray` captures native stack frames in addition to Python stack frames. This is useful for understanding memory allocations originating from C/C++ extensions or libraries.
*   **`trace_python_allocators`** (`bool`, default: `False`): If `True`, Python's internal allocators are traced as independent allocations. This provides deeper insight into how Python itself manages memory.
*   **`follow_fork`** (`bool`, default: `False`): Determines whether memory tracking continues in subprocesses that are forked from the tracked process. Set this to `True` if your task spawns child processes whose memory usage you also want to monitor.
*   **`memory_interval_ms`** (`int`, default: `10`): Specifies the interval in milliseconds between periodic resident set size updates. These updates are used to generate the memory usage over time graph in the reports. A smaller interval provides more granular data but may incur slightly higher overhead.
*   **`memray_html_reporter`** (`str`, default: `"flamegraph"`): Defines the type of HTML report to generate.
    *   `"flamegraph"`: Produces an interactive flame graph, which is excellent for visualizing call stacks and identifying memory hot spots.
    *   `"table"`: Generates a detailed table of memory allocations, useful for inspecting individual allocations and their sizes.
*   **`memray_reporter_args`** (`List[str]`, optional): A list of additional command-line arguments to pass directly to the chosen `memray` reporter. These arguments must be strings and typically start with `--`. For example, `["--leaks"]` can be used with the `flamegraph` reporter to highlight potential memory leaks. Refer to the `memray` documentation for available reporter arguments.

### Generated Reports and Visualization

When a task decorated with `memray_profiling` executes, the following occurs:

1.  A directory named `memray_bin` is created within the task's execution environment if it does not already exist.
2.  A binary profiling file (`.bin`) is generated within `memray_bin`, named after the task function and a timestamp (e.g., `my_memory_intensive_task.20231027123456.bin`). This file contains the raw memory allocation data.
3.  After the task completes, the raw `.bin` file is processed by the specified `memray_html_reporter` (e.g., `flamegraph` or `table`) to generate an HTML report. This HTML file is saved alongside the binary file (e.g., `flamegraph.my_memory_intensive_task.20231027123456.html`).
4.  The content of the generated HTML report is then automatically embedded and displayed within the FlyteDeck UI for the task execution. This allows for interactive exploration of the memory profile directly from the execution details page.

### Considerations and Best Practices

*   **Performance Overhead:** Memory profiling introduces some overhead due to the continuous tracking of allocations. While generally low, it is recommended to use profiling for debugging and optimization purposes rather than for continuous monitoring in production environments where performance is critical.
*   **Choosing the Right Reporter:**
    *   Use the `flamegraph` reporter for a high-level, visual understanding of where memory is being allocated across your call stack. It's effective for quickly pinpointing functions or code paths that are major memory consumers.
    *   Opt for the `table` reporter when you need a detailed, line-by-line breakdown of allocations, including their sizes and origins.
*   **Interpreting Results:** Focus on large memory allocations, functions that repeatedly allocate memory without releasing it, and patterns of increasing memory usage over time. The interactive nature of the flame graph allows drilling down into specific call stacks to identify the exact lines of code responsible for allocations.
*   **Dependencies:** Ensure that the `memray` library is installed in the environment where your tasks will execute. This is a prerequisite for the profiling functionality to work correctly.
<!--
key: summary_memray_profiling_cfd6487a-3ab2-43d4-b1fb-231e2171f3d3
type: summary_end

-->
<!--
code_unit: flytekitplugins.memray.examples.memray_profiling_task
code_unit_type: class
help_text: ''
key: example_4bbe96a6-124e-4fad-99fb-66be92352a2a
type: example

-->