
<!--
help_text: ''
key: summary_papermill_plugin_d0f6d1c9-94b6-4eda-a0c9-e8fee6f0fec3
modules:
- flytekitplugins.papermill.task
questions_to_answer: []
type: summary

-->
# Papermill Plugin

The Papermill Plugin enables the execution of Jupyter notebooks as Flyte tasks. It provides the `NotebookTask` class, which wraps a Python Jupyter notebook, allowing it to receive inputs from Flyte workflows and produce outputs consumable by subsequent tasks. This integration streamlines data science workflows by allowing notebooks to be first-class citizens in Flyte.

## Core Concepts

A Jupyter notebook integrated with Flyte via the Papermill Plugin must adhere to two primary properties:

1.  **Parameters Cell**: One cell within the notebook, typically the first, must be tagged as `parameters`. The `NotebookTask` injects all task inputs into this cell.
2.  **Output Recording**: To expose outputs from the notebook to Flyte, use the `record_outputs` function. This function, imported from the Papermill Plugin, captures Python variables and makes them available as task outputs.

### Recording Outputs

The `record_outputs` function is crucial for defining notebook outputs. After your notebook generates the desired values, call `record_outputs` with keyword arguments corresponding to your declared task outputs.

**Example Notebook Cell:**

```python
# cell begin
from flytekitplugins.papermill import record_outputs

val_x = 10
val_y = "hello"

record_outputs(x=val_x, y=val_y)
# cell end
```

## Defining a Notebook Task

To create a Flyte task from a Jupyter notebook, instantiate the `NotebookTask` class.

**Example `NotebookTask` Definition:**

```python
from flytekitplugins.papermill.task import NotebookTask
from flytekit import kwtypes
from flytekit.core.metadata import TaskMetadata

nb_task = NotebookTask(
    name="my_module.my_notebook_task",  # Unique name for the task
    notebook_path="./path/to/my_notebook.ipynb",
    render_deck=True,
    enable_deck=True,
    inputs=kwtypes(v=int),
    outputs=kwtypes(x=int, y=str),
    metadata=TaskMetadata(retries=3, cache=True, cache_version="1.0"),
)
```

**Key Parameters for `NotebookTask`:**

*   `name` (str): A unique name for the task within your project. Using the module name as a prefix is a common practice.
*   `notebook_path` (str): The absolute path to the Jupyter notebook file (`.ipynb`).
*   `inputs` (Dict[str, Type], optional): A dictionary defining the input variables expected by the notebook, mapping input names to their Flyte types. These inputs are injected into the notebook's `parameters` cell.
*   `outputs` (Dict[str, Type], optional): A dictionary defining the output variables produced by the notebook, mapping output names to their Flyte types. These outputs are extracted using `record_outputs`.
*   `render_deck` (bool, optional): If `True`, the executed notebook's HTML rendering (`out_rendered_nb`) is automatically embedded into a Flyte Deck, providing a rich visualization in the Flyte Console. Defaults to `False`.
*   `stream_logs` (bool, optional): If `True`, Papermill's internal logs are streamed directly to the pod's stdout, making them visible in the Flyte task logs. Defaults to `False`.

## Input and Output Handling

### Input Injection

The `NotebookTask` leverages Papermill to inject inputs into the notebook. When the task executes, the values provided to the task are passed as parameters to the notebook. Papermill then inserts these parameters into the cell tagged `parameters`.

**Supported Input Types:**
Only basic Python types that can be directly passed into a notebook cell are supported: `str`, `int`, `float`, and `bool`. For more complex types (e.g., `FlyteFile`, `FlyteDirectory`), ensure they are handled within the notebook logic after being passed as a string path.

### Output Extraction

The `NotebookTask` automatically extracts outputs from the executed notebook. The `record_outputs` function, when called within the notebook, writes the specified variables into a dedicated output cell. The `extract_outputs` static method of `NotebookTask` parses this cell, which must be tagged `outputs`, to retrieve the `LiteralMap` representing the notebook's declared outputs.

### Implicit Outputs

Beyond the user-defined outputs, `NotebookTask` produces two implicit outputs:

1.  `out_nb` (type `PythonNotebook`): The complete executed notebook, including all inputs, outputs, and execution results. This allows for full reproducibility and inspection.
2.  `out_rendered_nb` (type `HTMLPage`): An HTML rendering of the executed notebook. This is particularly useful for quick visual inspection and is used for Deck rendering.

These implicit outputs are automatically added to the task's `outputs` interface if `output_notebooks` is `True` (which is the default).

## Advanced Features and Considerations

### Deck Rendering

When `render_deck=True` is set during `NotebookTask` initialization, the `post_execute` hook automatically reads the generated HTML notebook (`out_rendered_nb`) and appends it to the task's execution decks. This allows the Flyte Console to display the executed notebook directly within the task's details page, providing immediate visual feedback on the notebook's execution and results.

### Logging

By default, print statements and other logs within a notebook are not directly streamed to the Flyte pod logs. To enable this, set `stream_logs=True` when defining `NotebookTask`. This configures the Papermill logger to output to `sys.stdout`, making notebook execution logs visible in the Flyte task logs. Be aware that notebook logs can be verbose, potentially increasing log ingestion costs.

### Underlying Task Configuration

`NotebookTask` internally wraps an underlying `PythonInstanceTask`. This allows it to inherit and apply configurations from other Flytekit plugins, such as `SparkTask` or `K8sPod`. For example, if you pass a `SparkConfig` object as `task_config` to `NotebookTask`, the underlying container will be configured with Spark settings.

The `get_container` and `get_k8s_pod` methods ensure that the container or Kubernetes pod specifications are correctly derived from this internal task instance, allowing for custom resource requests, environment variables, and other runtime configurations.

### Limitations

*   **Input Type Restrictions**: Papermill has limitations on the types it can directly inject into notebook cells. Only `str`, `int`, `float`, and `bool` are directly supported. For other types, consider passing file paths or serialized data.
*   **Spark Configuration**: Implicit extraction of Spark configuration directly from the notebook content is not supported. Spark configurations must be explicitly provided via `task_config` when defining the `NotebookTask`.
*   **Remote Notebook Execution**: The current implementation primarily assumes the notebook is available locally within the container image. Support for fetching notebooks from remote locations (e.g., S3, Git) is a future consideration.

## Best Practices

*   **Unique Task Names**: Ensure the `name` parameter for `NotebookTask` is unique across your project to avoid conflicts during serialization and registration.
*   **Clear Parameters Cell**: Always clearly mark your `parameters` cell in the notebook. It's best practice to place it early in the notebook.
*   **Explicit Outputs**: Use `record_outputs` consistently and explicitly for all data you intend to pass out of the notebook.
*   **Manage Log Verbosity**: While `stream_logs=True` is useful for debugging, consider its impact on log volume in production environments.
*   **Version Control Notebooks**: Treat your notebooks as code. Store them in version control alongside your Flyte workflows.
<!--
key: summary_papermill_plugin_d0f6d1c9-94b6-4eda-a0c9-e8fee6f0fec3
type: summary_end

-->
<!--
code_unit: flytekitplugins.papermill.task
code_unit_type: class
help_text: ''
key: example_88377b86-0c14-4b93-adbf-feb59d9bf548
type: example

-->