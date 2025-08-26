
<!--
help_text: ''
key: summary_experiment_tracking_&_profiling_plugins_f8981116-2f86-40fd-8643-d74698d0b754
modules:
- flytekitplugins.comet_ml.tracking
- flytekitplugins.neptune.tracking
- flytekitplugins.wandb.tracking
- flytekitplugins.memray.profiling
- flytekitplugins.whylogs.schema
- flytekitplugins.whylogs.renderer
questions_to_answer: []
type: summary

-->
Experiment Tracking & Profiling Plugins

Experiment tracking and profiling are crucial for understanding, debugging, and optimizing machine learning workflows. These plugins provide seamless integrations with popular tools to log experiment metadata, visualize performance, and analyze data quality directly within your workflows.

## Experiment Tracking

Experiment tracking plugins enable you to log parameters, metrics, artifacts, and other relevant information from your tasks to external platforms. This provides a centralized view of your experiment runs, facilitates reproducibility, and helps in comparing different model iterations.

### Comet ML Tracking

The Comet ML tracking plugin integrates with Comet ML to log experiment details. It automatically handles API key management and experiment key generation for remote executions.

To use this plugin, decorate your task function with `_comet_ml_login_class`.

```python
from flytekitplugins.comet_ml.tracking import _comet_ml_login_class
from flytekit import task, Secret

@_comet_ml_login_class(
    project_name="my-ml-project",
    workspace="my-workspace",
    secret=Secret(group="comet-ml", key="COMET_API_KEY"),
    # Optional: provide a specific experiment_key, otherwise one is generated
    # experiment_key="my-custom-experiment-id",
    # Optional: host="https://my-private-comet-instance.com",
    # Additional kwargs are passed to comet_ml.login or comet_ml.init
    tags=["flyte", "example"],
)
@task
def my_comet_ml_task(x: int) -> int:
    # Your task logic here
    # You can now use comet_ml.log_metric, comet_ml.log_parameter, etc.
    import comet_ml
    experiment = comet_ml.get_experiment()
    if experiment:
        experiment.log_metric("input_value", x)
    return x * 2
```

**Parameters:**

*   `project_name` (str): The name of the Comet ML project to send your experiment to.
*   `workspace` (str): The workspace associated with the project.
*   `secret` (Secret or Callable): A `Secret` object referencing your `COMET_API_KEY` or a callable that returns the API key as a string. For remote executions, the API key is retrieved from Flyte secrets.
*   `experiment_key` (Optional[str]): A unique key for the experiment. If not provided, a key is generated based on the Flyte execution ID for remote runs, or Comet ML generates one for local runs.
*   `host` (str): The URL to your Comet ML service. Defaults to `"https://www.comet.com"`.
*   `**login_kwargs` (dict): Any additional keyword arguments are passed directly to `comet_ml.login` or `comet_ml.init`.

**Behavior:**

*   **Local Execution:** For local runs, the plugin uses the provided `experiment_key` or allows Comet ML to generate one.
*   **Remote Execution:** For remote runs, the `api_key` is securely fetched using the provided `secret`. If `experiment_key` is not specified, a unique key is derived from the Flyte execution's hostname or name, ensuring traceability back to the Flyte execution.
*   The plugin automatically initializes the Comet ML experiment before your task function executes and ensures it's properly configured.

### Neptune.ai Tracking

The Neptune.ai tracking plugin integrates with Neptune.ai to log experiment runs. It automatically captures Flyte execution metadata, providing rich context for your Neptune runs.

To use this plugin, decorate your task function with `_neptune_init_run_class`.

```python
from flytekitplugins.neptune.tracking import _neptune_init_run_class
from flytekit import task, Secret
import neptune

@_neptune_init_run_class(
    project="my-neptune-workspace/my-ml-project",
    secret=Secret(group="neptune", key="NEPTUNE_API_TOKEN"),
    # Optional: host="https://my-private-neptune-instance.com",
    # Additional kwargs are passed to neptune.init_run
    tags=["flyte", "example"],
)
@task
def my_neptune_task(x: int) -> int:
    # Your task logic here
    # Access the Neptune run object from the Flyte context
    run = neptune.get_run()
    if run:
        run["input_value"] = x
        run["metrics/output"] = x * 2
    return x * 2
```

**Parameters:**

*   `project` (str): The Neptune project name (e.g., `"my-workspace/my-project"`).
*   `secret` (Secret or Callable): A `Secret` object referencing your `NEPTUNE_API_TOKEN` or a callable that returns the API token as a string.
*   `host` (str): The URL to your Neptune service. Defaults to `"https://app.neptune.ai"`.
*   `**init_run_kwargs` (dict): Any additional keyword arguments are passed directly to `neptune.init_run`.

**Behavior:**

*   **API Token Management:** For remote executions, the `api_token` is securely retrieved from Flyte secrets.
*   **Metadata Injection:** For remote executions, the plugin automatically logs comprehensive Flyte execution metadata to the Neptune run, including:
    *   `flyte/execution_id`
    *   `flyte/project`, `flyte/domain`, `flyte/name` (for the execution)
    *   `flyte/raw_output_prefix`, `flyte/output_metadata_prefix`, `flyte/working_directory`
    *   `flyte/task/name`, `flyte/task/project`, `flyte/task/domain`, `flyte/task/version`
    *   `flyte/execution_url` (if available)
*   **Run Lifecycle:** The Neptune run is initialized before the task executes and automatically stopped after the task completes, ensuring proper resource management.
*   **Context Propagation:** The Neptune run object is made available within the Flyte context, allowing you to access it inside your task using `neptune.get_run()`.

### Weights & Biases (W&B) Tracking

The Weights & Biases (W&B) tracking plugin integrates with W&B to log experiment runs. It handles API key authentication and can link back to your Flyte execution.

To use this plugin, decorate your task function with `wandb_init`.

```python
from flytekitplugins.wandb.tracking import wandb_init
from flytekit import task, Secret
import wandb

@wandb_init(
    project="my-wandb-project",
    entity="my-wandb-entity",
    secret=Secret(group="wandb", key="WANDB_API_KEY"),
    # Optional: id="my-custom-run-id",
    # Optional: host="https://my-private-wandb-instance.com",
    # Optional: api_host="https://api.my-private-wandb-instance.com",
    # Additional kwargs are passed to wandb.init
    tags=["flyte", "example"],
)
@task
def my_wandb_task(x: int) -> int:
    # Your task logic here
    # You can now use wandb.log, wandb.config, etc.
    wandb.log({"input_value": x, "output_value": x * 2})
    return x * 2
```

**Parameters:**

*   `project` (str): The name of the W&B project.
*   `entity` (str): The W&B username or team name.
*   `secret` (Secret or Callable): A `Secret` object referencing your `WANDB_API_KEY` or a callable that returns the API key as a string.
*   `id` (Optional[str]): A unique ID for the W&B run. If not provided, a run ID is generated based on the Flyte execution ID for remote runs, or W&B generates one for local runs.
*   `host` (str): The URL to your W&B service. Defaults to `"https://wandb.ai"`.
*   `api_host` (str): The URL to your W&B API host. Defaults to `"https://api.wandb.ai"`.
*   `**init_kwargs` (dict): Any additional keyword arguments are passed directly to `wandb.init`.

**Behavior:**

*   **API Key Authentication:** For remote executions, the `api_key` is securely retrieved from Flyte secrets and used to log in to W&B.
*   **Run ID Generation:** If `id` is not specified, a unique run ID is derived from the Flyte execution's hostname or name for remote runs.
*   **Flyte Execution Link:** If the `FLYTE_EXECUTION_URL` environment variable is set, the plugin automatically injects a link to the Flyte execution into the W&B run notes, enhancing traceability.
*   **Run Lifecycle:** The W&B run is initialized before the task executes and automatically finalized (`wandb.finish()`) after the task completes.

## Memory Profiling

Memory profiling helps identify memory leaks, excessive memory consumption, and inefficient memory usage patterns within your tasks.

### Memray Profiling

The Memray profiling plugin integrates with Memray to provide detailed memory usage reports for your tasks. It generates interactive HTML reports that can be viewed directly within FlyteDeck.

To use this plugin, decorate your task function with `memray_profiling`.

```python
from flytekitplugins.memray.profiling import memray_profiling
from flytekit import task

@memray_profiling(
    native_traces=True,
    trace_python_allocators=False,
    follow_fork=False,
    memory_interval_ms=10,
    memray_html_reporter="flamegraph", # or "table"
    memray_reporter_args=["--leaks"], # Example: pass arguments to the reporter
)
@task
def my_memory_intensive_task(n: int) -> list:
    # Simulate memory allocation
    data = [bytearray(1024 * 1024) for _ in range(n)] # Allocate n MB
    return data
```

**Parameters:**

*   `native_traces` (bool): Whether to capture native stack frames in addition to Python stack frames.
*   `trace_python_allocators` (bool): Whether to trace Python allocators as independent allocations.
*   `follow_fork` (bool): Whether to continue tracking in subprocesses forked from the tracked process.
*   `memory_interval_ms` (int): The interval in milliseconds between periodic resident set size updates. These updates are used to generate the memory usage graph in the reports.
*   `memray_html_reporter` (str): The type of HTML report to generate. Supported values are `"flamegraph"` and `"table"`.
*   `memray_reporter_args` (Optional[List[str]]): A list of additional arguments to pass to the selected Memray reporter command (e.g., `["--leaks"]` for flamegraph). Arguments must start with `--`.

**Behavior:**

*   **Automatic Profiling:** The plugin automatically starts a Memray tracker before your task function executes and stops it afterward.
*   **Report Generation:** After task completion, a Memray binary file is generated. The plugin then uses the Memray CLI to convert this binary file into an HTML report (either a flamegraph or a table) based on the `memray_html_reporter` setting.
*   **FlyteDeck Integration:** The generated HTML report is automatically embedded into FlyteDeck, allowing you to visualize memory usage patterns directly from the Flyte UI.

**Considerations:**

*   Ensure `memray` is installed in your task's execution environment.
*   Invalid `memray_reporter_args` will raise a `ValueError`. Refer to the Memray documentation for supported arguments for `flamegraph` and `table` reporters.

## Data Profiling and Quality

Data profiling and quality plugins integrate with whylogs to enable data quality checks, schema validation, and drift detection within your workflows.

### whylogs Dataset Profile Transformation

The `WhylogsDatasetProfileTransformer` enables seamless handling of `whylogs.DatasetProfileView` objects as inputs and outputs in your Flyte tasks. This allows you to pass data profiles between tasks, facilitating advanced data quality workflows.

```python
from flytekitplugins.whylogs.schema import WhylogsDatasetProfileTransformer
from flytekit import task
import whylogs as why
import pandas as pd
from whylogs.core.view.dataset_profile_view import DatasetProfileView

# Register the transformer (usually done automatically by the plugin)
# TypeEngine.register(WhylogsDatasetProfileTransformer())

@task
def profile_data(df: pd.DataFrame) -> DatasetProfileView:
    """Profiles a pandas DataFrame and returns a DatasetProfileView."""
    profile = why.log(df)
    return profile.view()

@task
def analyze_profile(profile_view: DatasetProfileView) -> str:
    """Analyzes a DatasetProfileView (e.g., prints summary) and returns a string."""
    # You can now work with the DatasetProfileView object
    summary = profile_view.to_pandas()
    return summary.to_string()

# Example usage in a workflow
# @workflow
# def my_data_quality_workflow(df: pd.DataFrame):
#     profile = profile_data(df=df)
#     analysis_result = analyze_profile(profile_view=profile)
#     # ... further tasks using the profile or analysis_result
```

**Capabilities:**

*   **Serialization/Deserialization:** Automatically converts `DatasetProfileView` objects to and from Flyte's `BlobType` for storage and retrieval.
*   **HTML Rendering:** Provides a `to_html` method that generates a `ProfileSummaryReport` HTML visualization of the `DatasetProfileView`, which can be rendered in FlyteDeck.

### whylogs Constraints Reporting

The `WhylogsConstraintsRenderer` generates an HTML report from a `whylogs.constraints.Constraints` object. This report visually indicates which data quality constraints passed or failed, providing immediate feedback on data integrity.

```python
from flytekitplugins.whylogs.renderer import WhylogsConstraintsRenderer
from flytekit import task
import whylogs as why
import pandas as pd
from whylogs.core.constraints import ConstraintsBuilder, MetricConstraint
from whylogs.core.metrics.selectors import MetricsSelector

@task
def check_data_constraints(df: pd.DataFrame) -> str:
    """
    Profiles data, defines constraints, and generates an HTML report.
    The HTML report will be rendered in FlyteDeck.
    """
    profile_view = why.log(df).view()
    builder = ConstraintsBuilder(profile_view)

    # Example constraint: check if 'sepal_length' is between 4.0 and 8.0
    num_constraint = MetricConstraint(
        name='sepal_length_range_check',
        condition=lambda x: x.min > 4.0 and x.max < 8.0,
        metric_selector=MetricsSelector(
            metric_name='distribution',
            column_name='sepal_length'
        )
    )
    builder.add_constraint(num_constraint)
    constraints = builder.build()

    # The renderer's to_html method is automatically called by FlyteDeck
    # when a Constraints object is returned and configured for rendering.
    return WhylogsConstraintsRenderer.to_html(constraints)

# Example usage in a workflow
# @workflow
# def my_constraint_workflow(df: pd.DataFrame):
#     constraint_report_html = check_data_constraints(df=df)
#     # The HTML will be visible in the Flyte UI for the task output
```

**Usage:**

*   Create a `Constraints` object using `whylogs.core.constraints.ConstraintsBuilder` based on a `DatasetProfileView`.
*   Pass this `Constraints` object to `WhylogsConstraintsRenderer.to_html()`. The returned HTML string can be displayed in FlyteDeck.

### whylogs Summary Drift Reporting

The `WhylogsSummaryDriftRenderer` generates an HTML report that visualizes data drift between two pandas DataFrames: a reference dataset and a target dataset. This is essential for monitoring changes in data distributions over time, which can indicate model degradation or data quality issues.

```python
from flytekitplugins.whylogs.renderer import WhylogsSummaryDriftRenderer
from flytekit import task
import pandas as pd

@task
def generate_drift_report(reference_data: pd.DataFrame, target_data: pd.DataFrame) -> str:
    """
    Generates an HTML summary drift report between two DataFrames.
    The HTML report will be rendered in FlyteDeck.
    """
    # The renderer's to_html method profiles the data internally
    # and then generates the drift report.
    return WhylogsSummaryDriftRenderer.to_html(reference_data, target_data)

# Example usage in a workflow
# @workflow
# def my_drift_detection_workflow(historical_data: pd.DataFrame, new_data: pd.DataFrame):
#     drift_report_html = generate_drift_report(
#         reference_data=historical_data,
#         target_data=new_data
#     )
#     # The HTML will be visible in the Flyte UI for the task output
```

**Usage:**

*   Provide two pandas DataFrames: `reference_data` (the baseline) and `target_data` (the data to compare against).
*   Call `WhylogsSummaryDriftRenderer.to_html(reference_data, target_data)`. The method internally profiles both DataFrames and generates an HTML report summarizing the drift, which can be displayed in FlyteDeck.
<!--
key: summary_experiment_tracking_&_profiling_plugins_f8981116-2f86-40fd-8643-d74698d0b754
type: summary_end

-->
<!--
code_unit: flytekitplugins.comet_ml.tracking._comet_ml_login_class
code_unit_type: class
help_text: ''
key: example_91ecce61-8b4b-44b1-9b36-a24cf19e960a
type: example

-->
<!--
code_unit: flytekitplugins.neptune.tracking._neptune_init_run_class
code_unit_type: class
help_text: ''
key: example_5fdaaa0d-a076-43c3-a6fe-e668b605b7cc
type: example

-->
<!--
code_unit: flytekitplugins.wandb.tracking.wandb_init
code_unit_type: class
help_text: ''
key: example_122d1fc9-577d-416d-b9d9-75781e412e40
type: example

-->
<!--
code_unit: flytekitplugins.memray.profiling.memray_profiling
code_unit_type: class
help_text: ''
key: example_84e7c324-fc6b-4289-bd1e-0b55aeaae97e
type: example

-->
<!--
code_unit: flytekitplugins.whylogs.schema.WhylogsDatasetProfileTransformer
code_unit_type: class
help_text: ''
key: example_9603eb33-a2e0-4109-b348-0c4dda07d6aa
type: example

-->