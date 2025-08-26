
<!--
help_text: ''
key: summary_custom_reporting_and_output_metadata_f96e29da-1ce4-4bb3-b753-68d67049f453
modules:
- flytekit.core.context_manager
questions_to_answer: []
type: summary

-->
Custom Reporting and Output Metadata capabilities enable developers to gain deeper insights into task execution and manage structured metadata for task outputs. These features are integrated into the core execution context, allowing for flexible and extensible reporting and data management.

### Custom Reporting with Decks

Custom reporting in Flytekit is primarily facilitated through **Decks**. Decks allow tasks to generate rich, interactive HTML reports or visualizations that are then associated with the task's execution. These reports can include anything from data visualizations to detailed logs or custom diagnostic information, providing a powerful way to convey insights beyond standard task outputs.

The core mechanism for managing decks is through the `FlyteContext` and `ExecutionParameters`.

**Enabling and Using Decks**

To enable deck generation for a task, ensure the `enable_deck` flag is set to `True` when defining the task or workflow. Within a task, you can access the list of decks via the current execution context.

```python
from flytekit import task, current_context, Deck
from IPython.display import HTML # For local rendering

@task
def my_reporting_task(input_data: int) -> str:
    # Access the current execution parameters
    ctx = current_context()

    # Create a custom HTML report
    report_content = f"<h1>Task Report for Input: {input_data}</h1>"
    report_content += "<p>This is a custom report generated during task execution.</p>"
    report_content += f"<p>Execution ID: {ctx.execution_id.name}</p>"

    # Create a Deck object and add content
    custom_deck = Deck("my_custom_report", report_content)

    # Add the custom deck to the list of decks in the context
    ctx.decks.append(custom_deck)

    # You can also use the default deck
    ctx.default_deck.append("<h2>Default Deck Content</h2><p>More info here.</p>")

    return "Report generated"

# To run and view locally (e.g., in a Jupyter notebook)
# from flytekit import workflow
# @workflow
# def my_workflow():
#     my_reporting_task(input_data=10)

# if __name__ == "__main__":
#     # When running locally, the context manager captures the deck
#     with flytekit.new_context() as ctx:
#         my_reporting_task(input_data=10)
#     
#     # Retrieve and display the combined deck content
#     display(ctx.get_deck())
```

The `FlyteContext.get_deck()` method aggregates all `Deck` objects added to the `decks` list within the `ExecutionParameters` and returns a combined representation suitable for display (e.g., an `IPython.core.display.HTML` object in a notebook environment, or a string).

The `ExecutionParameters` object, accessible via `flytekit.current_context()`, provides the `decks` property, which is a list of `Deck` objects. It also offers `default_deck` for general content and `timeline_deck` for time-series related visualizations.

### Attaching Output Metadata

Beyond the primary return values, tasks can attach arbitrary, structured metadata to their outputs. This is particularly useful for adding contextual information, such as data lineage, partitioning details, or custom tags, directly to the output literals. This metadata is stored alongside the output and can be retrieved later for analysis or auditing.

The `OutputMetadataTracker` class, available through the `FlyteContext`, is the central component for managing output metadata.

**Defining Output Metadata**

Metadata for a specific output object is encapsulated by the `OutputMetadata` class. This class provides fields to describe the output:

*   `artifact`: A reference to an associated artifact (e.g., a dataset version).
*   `dynamic_partitions`: A dictionary (`Dict[str, str]`) for key-value pairs representing dynamic partitions (e.g., `{"year": "2023", "month": "01"}`).
*   `time_partition`: A `datetime` object for time-based partitioning.
*   `additional_items`: A list of custom objects that conform to the `SerializableToString` protocol.

**Attaching Metadata to Outputs**

To attach metadata, use the `OutputMetadataTracker.add()` method, passing the output object and an `OutputMetadata` instance. The `OutputMetadataTracker` stores metadata by the object's identity (`id(obj)`).

```python
from flytekit import task, current_context
from flytekit.core.context_manager import OutputMetadata, SerializableToString
from datetime import datetime
import typing

# Define a custom serializable metadata object
class MyCustomMetadata(SerializableToString):
    def __init__(self, version: str, author: str):
        self.version = version
        self.author = author

    def serialize_to_string(self, ctx, variable_name: str) -> typing.Tuple[str, str]:
        # This method defines how your custom object is serialized for metadata
        return "text/plain", f"CustomMetadata(version={self.version}, author={self.author})"

@task
def process_data(input_val: int) -> typing.Tuple[str, int]:
    ctx = current_context()

    # Simulate producing two outputs
    output_str = f"Processed string for {input_val}"
    output_int = input_val * 2

    # Attach metadata to the string output
    ctx.output_metadata_tracker.add(
        output_str,
        OutputMetadata(
            dynamic_partitions={"region": "us-east-1", "data_type": "processed"},
            time_partition=datetime.now(),
            additional_items=[MyCustomMetadata(version="1.0", author="Flyte User")]
        )
    )

    # Attach metadata to the integer output (e.g., a simple artifact reference)
    ctx.output_metadata_tracker.add(
        output_int,
        OutputMetadata(artifact="my_processed_integer_artifact_v2")
    )

    return output_str, output_int

# When this task executes, the associated metadata will be captured by the Flyte backend.
```

The `SerializableToString` protocol is crucial for including custom Python objects in `additional_items`. Any class implementing this protocol must provide a `serialize_to_string` method that returns a tuple of `(media_type: str, serialized_string: str)`. This allows Flyte to store and represent complex metadata objects in a standardized way.

### Runtime Context for Broader Reporting

The `ExecutionParameters` object, accessible via `flytekit.current_context()`, provides a comprehensive set of runtime information that can be leveraged for various reporting and output purposes:

*   **`stats`**: A `taggable.TaggableStats` object for emitting metrics. This allows tasks to report custom metrics (e.g., processing time, data quality scores) that can be aggregated and visualized in monitoring systems.
*   **`logging`**: A standard Python `logging.Logger` instance configured for the current execution. Tasks can use this logger to emit structured logs, which are crucial for debugging and operational monitoring.
*   **`working_directory`**: A temporary directory where tasks can write arbitrary files. This is useful for generating intermediate reports, plots, or other artifacts that are not direct task outputs but are part of the execution's context. These files can then be uploaded to a persistent store if needed.
*   **`execution_id`**: The unique identifier for the current workflow execution. This can be used to tag output data or reports, linking them back to the specific run that produced them.
*   **`task_id`**: The identifier for the current task execution.

These properties provide a robust foundation for tasks to produce various forms of custom output and integrate with external reporting and monitoring systems.

### Integration and Best Practices

*   **Centralized Context**: The `FlyteContext` acts as the central hub for all runtime information, including `ExecutionState` and `OutputMetadataTracker`. Always access these capabilities through `flytekit.current_context()`.
*   **Metadata Granularity**: Attach metadata at the most appropriate level. For data lineage or partitioning, attach it directly to the output literal. For overall task performance, use `stats` or `logging`.
*   **Serialization Overhead**: When using `SerializableToString` for `additional_items` in `OutputMetadata`, be mindful of the size and complexity of the objects being serialized. Excessive data can impact performance and storage.
*   **Local vs. Remote Execution**: Decks and output metadata are designed to work seamlessly across local and remote Flyte executions. When running locally, `FlyteContext.get_deck()` can be used to retrieve the generated HTML. For remote executions, this information is captured by the Flyte backend and made available through the UI or API.
*   **Immutability of Context**: While you can append to `ctx.decks` or add to `ctx.output_metadata_tracker`, the core context objects (`FlyteContext`, `ExecutionState`, `ExecutionParameters`) are designed to be largely immutable and are managed by the `FlyteContextManager` using a stack-based approach. This ensures consistent state during execution.
<!--
key: summary_custom_reporting_and_output_metadata_f96e29da-1ce4-4bb3-b753-68d67049f453
type: summary_end

-->
<!--
code_unit: flytekit.core.context_manager.OutputMetadataTracker
code_unit_type: class
help_text: ''
key: example_91e7caf6-1fba-4421-be3b-7c9c9216dd5f
type: example

-->
<!--
code_unit: flytekit.core.context_manager.ExecutionParameters
code_unit_type: class
help_text: ''
key: example_0b649b88-a07a-4825-8757-88e7f742eaf2
type: example

-->
<!--
code_unit: flytekit.core.context_manager.FlyteContext
code_unit_type: class
help_text: ''
key: example_4e65ca15-1562-4429-a681-115c51beedc1
type: example

-->