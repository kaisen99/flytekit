
<!--
help_text: ''
key: summary_execution_context_and_environment_management_ca1cb29b-5d45-49e0-842c-29c267e08739
modules:
- flytekit.core.context_manager
- flytekit.models.core.identifier
questions_to_answer: []
type: summary

-->
## Execution Context and Environment Management

Flytekit provides a robust system for managing the execution context and environment, ensuring that tasks and workflows have access to the necessary information and resources during both compilation and runtime. This system differentiates between internal, framework-level context and user-facing context, offering a clear separation of concerns.

### FlyteContext: The Internal Context

The `FlyteContext` serves as Flytekit's internal, low-level context object. It is a comprehensive container for various settings and objects essential for Flytekit's operations, including type conversion, workflow compilation, entity serialization, and file access. Most users will not interact with `FlyteContext` directly; instead, it is managed by the `FlyteContextManager`.

Key components and capabilities of `FlyteContext` include:

*   **File Access (`file_access`):** Provides mechanisms for interacting with local and remote file systems, crucial for handling inputs, outputs, and intermediate data.
*   **Compilation State (`compilation_state`):** Stores information relevant during the compilation phase of workflows and tasks, such as the graph of nodes being built and naming prefixes.
*   **Execution State (`execution_state`):** Holds the state specific to task or local workflow execution, including the execution mode, working directories, and user-space parameters.
*   **Serialization Settings (`serialization_settings`):** Configures how Flyte entities are serialized for transmission to the Flyte backend.
*   **Flyte Client (`flyte_client`):** Provides an interface for communicating with the Flyte API.
*   **Conditional Logic (`in_a_condition`):** Tracks whether the current context is within a conditional branch, influencing how nodes are evaluated.
*   **Output Metadata Tracking (`output_metadata_tracker`):** Allows for attaching arbitrary metadata to task outputs.
*   **Worker Queue (`worker_queue`):** Manages asynchronous operations, particularly relevant for local execution.

`FlyteContext` instances are typically created and modified using a builder pattern (`FlyteContext.Builder`). This allows for creating new contexts or deriving new contexts from existing ones by overriding specific parameters.

**Example: Accessing the Deck**

While `FlyteContext` is primarily internal, it provides a method to retrieve the generated HTML deck from the last execution, which can be useful for debugging or visualization in interactive environments like notebooks.

```python
import flytekit
from IPython import display

# Example: Running a task within a new context to capture its deck
@flytekit.task
def my_simple_task(a: int, b: int) -> int:
    flytekit.current_context().default_deck.append("<h1>My Task Output</h1>")
    flytekit.current_context().default_deck.append(f"<p>Input a: {a}, Input b: {b}</p>")
    return a + b

with flytekit.new_context() as ctx:
    my_simple_task(a=10, b=20)

# Retrieve and display the deck
display(ctx.get_deck())
```

### ExecutionParameters: The User-Facing Task Context

`ExecutionParameters` is the primary user-facing context object accessible within any `@task` method. It provides essential runtime information and utilities that tasks can leverage. This object is distinct from `FlyteContext` and is designed for direct consumption by user code.

You can access the current `ExecutionParameters` object within a task using `flytekit.current_context()`.

Key properties available through `ExecutionParameters`:

*   **Stats (`stats`):** A handle to a statsd object for emitting metrics, often pre-tagged with relevant execution identifiers.
*   **Logging (`logging`):** A logger instance configured for the current execution, allowing tasks to emit structured logs.
*   **Working Directory (`working_directory`):** A path to a temporary directory where tasks can write arbitrary files. This directory is typically managed by Flytekit and cleaned up after execution.
*   **Execution Date (`execution_date`):** A `datetime` object representing the start time of the workflow execution. This is consistent across all tasks within the same workflow or sub-workflow.
    *   **Consideration:** This date should not be used for production logic but is useful for debugging or tagging output data.
*   **Execution ID (`execution_id`):** The unique identifier of the workflow execution within the Flyte engine. Consistent across all tasks in a workflow.
    *   **Consideration:** Similar to `execution_date`, this ID is primarily for debugging and linking data, not for driving production logic.
*   **Task ID (`task_id`):** The identifier for the currently executing task.
*   **Secrets (`secrets`):** An instance of `SecretsManager` for securely retrieving sensitive information.
*   **Checkpoint (`checkpoint`):** A handle to the configured checkpointing system, enabling tasks to save and restore state.
*   **Decks (`decks`):** A list of `Deck` objects that can be used to generate rich HTML outputs for tasks, visible in the Flyte UI.
*   **Enable Deck (`enable_deck`):** A boolean indicating whether deck rendering is enabled for the current execution.
*   **Raw Output Prefix (`raw_output_prefix`):** The base path for raw output data.
*   **Output Metadata Prefix (`output_metadata_prefix`):** The base path for output metadata.
*   **Task-Specific Attributes:** `ExecutionParameters` can also house task-type-specific context (e.g., a SparkSession for Spark tasks) accessible via attribute lookup (`ctx.SPARK_SESSION`) or the `get()` method.

**Example: Using ExecutionParameters within a Task**

```python
import flytekit
import os
from datetime import datetime

@flytekit.task
def process_data(input_path: str) -> str:
    ctx = flytekit.current_context()

    # Accessing execution details
    ctx.logging.info(f"Task '{ctx.task_id.name}' started at {ctx.execution_date}")
    ctx.logging.info(f"Workflow execution ID: {ctx.execution_id.name}")

    # Using the working directory
    temp_file_path = os.path.join(ctx.working_directory, "processed_output.txt")
    with open(temp_file_path, "w") as f:
        f.write(f"Processed data from {input_path} at {datetime.now()}")
    ctx.logging.info(f"Wrote temporary file to: {temp_file_path}")

    # Emitting a custom metric
    ctx.stats.incr("data_processed_count")

    # Adding content to the default deck
    ctx.default_deck.append(f"<h2>Processing Complete</h2><p>Output written to {temp_file_path}</p>")

    return temp_file_path

# When this task runs, these parameters will be automatically populated.
```

### FlyteContextManager: Context Lifecycle

The `FlyteContextManager` is responsible for managing the stack of `FlyteContext` objects. It ensures that the correct context is active at any given time, whether Flytekit is compiling a workflow or executing a task.

*   **Singleton Stack:** It maintains a stack of `FlyteContext` instances. When a new operation (like compiling a sub-workflow or executing a task) begins, a new context is pushed onto the stack. When the operation completes, the context is popped.
*   **`initialize()`:** This static method re-initializes the context, setting up a default `FlyteContext` and `ExecutionParameters` for local execution. It also configures signal handlers.
*   **`current_context()`:** Retrieves the `FlyteContext` currently at the top of the stack.
*   **`push_context()` and `pop_context()`:** These methods are used internally to manage the context stack. Direct calls to these methods are generally discouraged for users.
*   **`with_context()`:** This is the recommended and safest way to manage context scope. It's a context manager that pushes a new `FlyteContext` onto the stack upon entry and ensures it's popped upon exit, even if errors occur. This is crucial for maintaining a clean context state, especially in complex scenarios like nested conditionals.

**Example: Using `with_context`**

```python
from flytekit.core.context_manager import FlyteContextManager, FlyteContext
from flytekit.file_access import LocalFileAccessProvider

# Create a builder for a new context
builder = FlyteContext.Builder(file_access=LocalFileAccessProvider("/tmp/flytekit_sandbox"))
builder.with_new_compilation_state() # Example: setting up for compilation

# Use the context manager to activate the new context
with FlyteContextManager.with_context(builder) as ctx:
    # Inside this block, ctx is the active FlyteContext
    print(f"Current context level: {ctx.level}")
    print(f"Is in compilation mode: {ctx.compilation_state is not None}")

# Outside the block, the previous context is restored
print(f"Context level after block: {FlyteContextManager.current_context().level}")
```

**Important Considerations:**

*   **Thread Safety:** The `FlyteContextManager` is currently *not* thread-safe. Flytekit operations should ideally run in a single-threaded environment or manage their own context explicitly if multi-threading is required.
*   **Context Leaks:** The `with_context` manager includes logic to prevent context leaks, especially in conditional sections where explicit context management might be challenging. It ensures that the context stack is correctly unwound.

### SecretsManager: Secure Credential Handling

The `SecretsManager` provides a secure and standardized way to retrieve sensitive information (secrets) within tasks. It abstracts away the underlying storage mechanism, offering a consistent interface.

**Resolution Order:**

1.  **Environment Variables:** The `SecretsManager` first attempts to resolve secrets from environment variables. These variables should be prefixed with `configuration.SECRETS_ENV_PREFIX` (e.g., `FLYTE_SECRETS_`). The environment variable name will be all uppercase. During local execution, it also checks for the key without a prefix.
2.  **Files:** If not found in environment variables, it then looks for the secret in a file. The file path is constructed based on `configuration.SECRETS_DEFAULT_DIR/<group>/configuration.SECRETS_FILE_PREFIX<key>`.

**Accessing Secrets:**

Secrets can be accessed in two primary ways:

1.  **Attribute-style lookup:** `ctx.secrets.group_name.key_name`
2.  **`get()` method:** `ctx.secrets.get("group_name", "key_name")`

**Example: Retrieving Secrets**

```python
import flytekit
from flytekit.core.secrets import Secret, SecretStrategy

@flytekit.task(secret_requests=[Secret(group="my_secrets", key="api_key")])
def use_secret_task():
    ctx = flytekit.current_context()

    # Attribute-style access
    api_key = ctx.secrets.my_secrets.api_key
    ctx.logging.info(f"Retrieved API Key (attribute-style): {api_key[:3]}...") # Print partial for security

    # Using the get() method
    db_password = ctx.secrets.get("my_secrets", "db_password")
    ctx.logging.info(f"Retrieved DB Password (get method): {db_password[:3]}...")

    # Example of a secret that might not be defined, leading to ValueError
    try:
        non_existent_secret = ctx.secrets.get("non_existent_group", "non_existent_key")
    except ValueError as e:
        ctx.logging.error(f"Failed to retrieve non-existent secret: {e}")

# To run this locally, you would set environment variables or create files:
# export FLYTE_SECRETS_MY_SECRETS_API_KEY="your_api_key_here"
# export FLYTE_SECRETS_MY_SECRETS_DB_PASSWORD="your_db_password_here"
# Or create files like:
# /tmp/flytekit/secrets/my_secrets/flyte_api_key
# /tmp/flytekit/secrets/my_secrets/flyte_db_password
```

**Best Practice:** Always declare required secrets using the `secret_requests` argument in the `@task` decorator. This ensures that the Flyte backend provisions the necessary secrets to your task's execution environment.

### OutputMetadataTracker: Attaching Metadata to Outputs

The `OutputMetadataTracker` allows users to associate arbitrary metadata with the output literals of a task. This metadata can provide additional context or information about the generated outputs, which can be useful for downstream processing, auditing, or visualization.

*   **`add(obj, metadata)`:** Associates an `OutputMetadata` object with a specific Python object that will be returned as a task output.
*   **`get(obj)`:** Retrieves the `OutputMetadata` associated with a given object.

The `OutputMetadata` object itself can contain:

*   `artifact`: Reference to an `Artifact`.
*   `dynamic_partitions`: A dictionary of string key-value pairs representing dynamic partitions.
*   `time_partition`: A `datetime` object for time-based partitioning.
*   `additional_items`: A list of `SerializableToString` objects for custom metadata.

**Example: Adding Output Metadata**

```python
import flytekit
from flytekit.core.context_manager import OutputMetadata
from datetime import datetime

@flytekit.task
def generate_report(data_size: int) -> str:
    ctx = flytekit.current_context()
    report_content = f"Report generated for {data_size} records at {datetime.now()}"
    report_path = "/tmp/my_report.txt" # In a real task, this would be a remote path

    # Simulate writing report content
    with open(report_path, "w") as f:
        f.write(report_content)

    # Create metadata for the report output
    metadata = OutputMetadata(
        dynamic_partitions={"region": "us-east-1", "format": "txt"},
        time_partition=datetime.now(),
        # additional_items=[MyCustomSerializableObject()] # If you have custom serializable objects
    )

    # Associate the metadata with the output object (report_path in this case)
    ctx.output_metadata_tracker.add(report_path, metadata)

    return report_path

# When this task executes, the metadata will be attached to the literal representing report_path.
```

### ExecutionState and CompilationState

These are internal state objects managed by `FlyteContext` that define the specific operational mode of Flytekit:

*   **`ExecutionState`:** Governs the behavior during runtime. It defines the `Mode` (e.g., `TASK_EXECUTION`, `LOCAL_WORKFLOW_EXECUTION`, `DYNAMIC_TASK_EXECUTION`), manages working directories (`working_dir`, `engine_dir`), and handles branch evaluation for conditionals (`branch_eval_mode`). It also holds the `user_space_params` (`ExecutionParameters`).
*   **`CompilationState`:** Manages the state during the compilation of Flyte entities into their protobuf representations. It tracks the nodes being built (`nodes`), applies naming prefixes (`prefix`) for unique identification, and uses a `task_resolver` to locate task implementations.

These states are crucial for Flytekit to correctly interpret and process user code, adapting its behavior based on whether it's building a workflow graph or executing a task.
<!--
key: summary_execution_context_and_environment_management_ca1cb29b-5d45-49e0-842c-29c267e08739
type: summary_end

-->
<!--
code_unit: flytekit.core.context_manager
code_unit_type: class
help_text: ''
key: example_fb613e44-7afe-42dc-b613-3068b1bf0dbb
type: example

-->
<!--
code_unit: flytekit.models.core.identifier
code_unit_type: class
help_text: ''
key: example_a5026b58-cc34-4cdc-a9f5-ec9bbdaa9435
type: example

-->