
<!--
help_text: ''
key: summary_papermill_notebook_execution_f26d0fc0-b29a-4747-a580-404ee5c0b25a
modules:
- flytekitplugins.papermill.task
questions_to_answer: []
type: summary

-->
Papermill Notebook Execution enables the seamless integration and execution of Jupyter notebooks as tasks within Flyte workflows. This capability allows data scientists and engineers to leverage their existing notebook-based workflows, benefiting from Flyte's orchestration, reproducibility, and scalability features.

### Core Component: NotebookTask

The central component for Papermill Notebook Execution is the `NotebookTask` class. This task type wraps a standard Jupyter notebook, allowing it to be executed as a Flyte task. It handles the injection of inputs into the notebook, the extraction of outputs, and the capture of the executed notebook for provenance and visualization.

### Defining a Notebook for Flyte

To be compatible with `NotebookTask`, a Jupyter notebook must adhere to two primary properties:

1.  **Parameters Cell:** One cell in the notebook, typically the first, must be tagged as `parameters`. `NotebookTask` injects task inputs into this cell, making them available as variables within the notebook's execution environment.
2.  **Output Recording:** For notebooks that produce outputs intended for downstream Flyte tasks, use the `record_outputs` function. Call this function within your notebook after the outputs are ready, passing all desired outputs as keyword arguments.

    Example within a notebook cell:
    ```python
    # cell begin
    from flytekitplugins.papermill import record_outputs

    val_x = 10
    val_y = "hello"

    record_outputs(x=val_x, y=val_y)
    # cell end
    ```
    `NotebookTask` automatically parses outputs from a cell tagged `outputs`. The `record_outputs` function ensures that the necessary output structure is created for Flyte to consume.

### Creating a NotebookTask

Instantiate `NotebookTask` by providing the notebook's path and defining its inputs and outputs. The `name` parameter should be unique across all tasks, often incorporating the module name.

Example:
```python
from flytekitplugins.papermill.task import NotebookTask
from flytekit import kwtypes
from flytekit.core.metadata import TaskMetadata

nb_task = NotebookTask(
    name="my_module.my_notebook_task",
    notebook_path="./path/to/my_notebook.ipynb", # Relative or absolute path
    render_deck=True,
    inputs=kwtypes(v=int),
    outputs=kwtypes(x=int, y=str),
    metadata=TaskMetadata(retries=3, cache=True, cache_version="1.0"),
)
```

### Input and Output Handling

*   **Inputs:** `NotebookTask` injects inputs defined in the `inputs` parameter into the notebook's `parameters` cell. Note that Papermill has limitations on input types; only `str`, `int`, `float`, and `bool` are directly supported for injection into cells.
*   **Explicit Outputs:** Outputs explicitly defined in the `outputs` parameter of `NotebookTask` are extracted from the notebook using the `record_outputs` mechanism.
*   **Implicit Outputs:** `NotebookTask` automatically produces two implicit outputs:
    *   `out_nb`: The complete executed notebook, available as a `PythonNotebook` type. This provides a record of the exact execution, including all cell outputs.
    *   `out_rendered_nb`: An HTML rendering of the executed notebook, available as an `HTMLPage` type. This is particularly useful for quick visualization in the Flyte Console.

### Rendering and Logging

*   **`render_deck`:** When `render_deck=True` is passed during `NotebookTask` instantiation, the `out_rendered_nb` HTML content is automatically embedded into the Flyte Console's execution deck. This provides an immediate visual summary of the notebook's execution results directly within the UI.
*   **`stream_logs`:** By default, print statements within a notebook are not forwarded to the Flyte pod logs. To enable this, set `stream_logs=True`. This directs Papermill's internal logging to standard output, making it visible in the pod logs. Be aware that notebook logs can be verbose, potentially increasing log ingestion costs.

### Limitations and Considerations

*   **Input Type Restrictions:** Only `str`, `int`, `float`, and `bool` types are directly supported for input injection into notebook cells by Papermill. For other types (e.g., `FlyteFile`, `FlyteDirectory`), ensure they are handled appropriately within the notebook (e.g., by reading from the provided path).
*   **Spark Configuration:** Implicit extraction of Spark configuration from the notebook is not currently supported. Users must configure Spark sessions explicitly within their notebooks or via the underlying task configuration.
*   **Remote Notebook Execution:** Direct support for remote notebook execution (e.g., fetching notebooks from S3) is not yet available. Notebooks must be accessible within the container's file system.
*   **Log Verbosity:** Enabling `stream_logs` can result in a large volume of logs, which may impact log management and cost.
*   **Task Naming:** Each `NotebookTask` instance requires a unique `name` to prevent conflicts during serialization and execution.

### Best Practices

*   **Modular Notebooks:** Design notebooks to be modular, with clear inputs and outputs, making them easier to integrate as Flyte tasks.
*   **Version Control:** Keep notebooks under version control alongside your Flyte workflows to ensure reproducibility.
*   **Clear Parameters:** Clearly define and document the expected inputs in your notebook's `parameters` cell.
*   **Explicit Outputs:** Always use `record_outputs` to explicitly define what data should be passed out of the notebook and consumed by downstream Flyte tasks.
*   **Error Handling:** Implement robust error handling within your notebooks to provide clear diagnostics in case of failures.
*   **Resource Management:** Be mindful of the computational resources required by your notebooks and configure the underlying task resources accordingly.
<!--
key: summary_papermill_notebook_execution_f26d0fc0-b29a-4747-a580-404ee5c0b25a
type: summary_end

-->
<!--
code_unit: flytekitplugins.papermill.examples.notebook_task_example
code_unit_type: class
help_text: ''
key: example_0d887b80-2802-4bc9-b8c2-6996f4e24988
type: example

-->