
<!--
help_text: ''
key: summary_managing_cli_parameters_and_inputs_e4871817-ff99-46e3-b8c1-ac4445466e04
modules:
- flytekit.clis.sdk_in_container.run
- flytekit.interaction.click_types
- flytekit.clis.sdk_in_container.utils
questions_to_answer: []
type: summary

-->
This section describes how the command-line interface (CLI) handles parameters and inputs when executing Flyte workflows, tasks, and launch plans. It covers global execution parameters, entity-specific inputs, various input sources, and the underlying type conversion mechanisms.

### Global Run-Level Parameters

When you execute a Flyte entity using the CLI, a set of common parameters control the execution environment and behavior. These parameters are defined by the `RunLevelParams` structure and are available across all `run` commands.

Examples of global parameters include:

*   `--project` and `--domain`: Specify the Flyte project and domain for the execution.
*   `--image`: Defines the Docker image used for the execution. Multiple images can be specified.
*   `--service-account`: Sets the Kubernetes service account for the execution.
*   `--wait-execution` (`--wait`): Instructs the CLI to wait for the execution to complete and display its status.
*   `--poll-interval`: Configures the polling frequency when waiting for execution completion.
*   `--dump-snippet`: Generates a Python code snippet to load the execution using the remote client.
*   `--overwrite-cache`: Forces the system to re-execute even if cached results exist.
*   `--envvars` (`--env`): Allows passing environment variables to the execution container in `KEY=VALUE` format.
*   `--tags`: Attaches tags to the execution for categorization.
*   `--name`: Assigns a custom name to the execution.
*   `--labels` and `--annotations`: Attach Kubernetes labels and annotations to the execution.
*   `--raw-output-data-prefix`: Specifies a custom location for raw output data (e.g., `s3://my-bucket/data`).
*   `--max-parallelism`: Limits the number of parallel nodes in a workflow.
*   `--disable-notifications`: Prevents notifications for the execution.
*   `--remote` (`-r`): Crucially, this flag determines whether the execution runs locally or is registered and launched on a remote Flyte cluster. This flag significantly impacts input handling and type conversion.
*   `--limit`: Used when listing remote entities to control the number of results fetched.
*   `--cluster-pool` and `--execution-cluster-label`: Assigns the execution to a specific cluster pool or label.

These parameters are processed by the main `RunCommand` group and are then made available to subcommands through the Click context object.

### Entity-Specific Inputs

Beyond the global run-level parameters, each Flyte workflow, task, or launch plan defines its own set of inputs. The CLI dynamically generates command-line options for these inputs, making them accessible directly.

When you select a specific entity (e.g., `pyflyte run my_file.py my_workflow`), the `WorkflowCommand` (for local entities) or `DynamicEntityLaunchCommand` (for remote entities) inspects the entity's interface to determine its expected inputs. For each input, a corresponding CLI option is created.

For example, if a workflow `my_workflow` has an input `my_string_input: str` and `my_int_input: int = 10`, the CLI will expose options like:

```bash
pyflyte run my_file.py my_workflow --my-string-input "hello" --my-int-input 20
```

**Launch Plan Specifics:**
Launch plans can have fixed inputs and default inputs defined at the plan level. When executing a launch plan, the CLI combines these with the underlying workflow's interface inputs. Fixed inputs cannot be overridden via the CLI, while default inputs can be. The `WorkflowCommand` and `DynamicEntityLaunchCommand` ensure that the correct set of available inputs is presented as CLI options.

### Input Sources

The CLI provides flexible ways to supply inputs to your Flyte entities:

1.  **Direct Command-Line Arguments:**
    The most common method is to pass inputs directly as `--input-name value` pairs. This is suitable for a small number of simple inputs.

    ```bash
    pyflyte run my_file.py my_task --input_a "value1" --input_b 123
    ```

2.  **YAML or JSON Input Files:**
    For complex inputs, or when dealing with many parameters, you can provide inputs from a YAML or JSON file using the `--inputs-file` option. This is handled by the `YamlFileReadingCommand`.

    Create a file (e.g., `inputs.yaml`):
    ```yaml
    input_a: "value from file"
    input_b: 456
    complex_data:
      key1: "nested_value"
      key2: [1, 2, 3]
    ```

    Then execute:
    ```bash
    pyflyte run my_file.py my_workflow --inputs-file inputs.yaml
    ```

    The CLI parses the file and converts its contents into the equivalent command-line arguments. Boolean values are handled correctly (e.g., `my_flag: true` becomes `--my-flag`). Complex types like lists and dictionaries are serialized to JSON strings if passed as direct arguments, but the file input mechanism handles them natively.

3.  **Standard Input (stdin):**
    You can also pipe inputs directly to the command using a hyphen (`-`) as the last argument. This is useful for scripting or when inputs are generated by another process.

    ```bash
    echo '{"input_a": "from stdin", "input_b": 789}' | pyflyte run my_file.py my_workflow -
    ```

    The `YamlFileReadingCommand` intercepts the `"-"` argument and reads the content from stdin, treating it as a YAML or JSON string.

### Type Conversion and Handling

A critical aspect of CLI input management is the seamless conversion of command-line string arguments into the appropriate Python types and, for remote executions, into Flyte `Literal` objects. This conversion is managed by the `FlyteLiteralConverter` and a set of custom Click parameter types.

*   **`FlyteLiteralConverter`:** This component acts as a bridge. It takes the raw string value from the CLI, the expected Python type (e.g., `str`, `int`, `datetime.datetime`), and the Flyte `LiteralType` (e.g., `STRING`, `INTEGER`, `DATETIME`).
    *   For **local execution** (when `--remote` is not used), it converts the input string directly to the corresponding Python native type.
    *   For **remote execution** (when `--remote` is used), it converts the input string into a Flyte `Literal` object, which is the canonical representation for data passed to the Flyte backend. This ensures type fidelity and compatibility with the Flyte platform.

*   **Custom Parameter Types:** The `click_types` module provides specialized Click parameter types to handle complex Flyte data models and common Python types gracefully:
    *   `DateTimeType`: Parses various date/time formats, including "now", "today", and relative durations (e.g., "today - 1 day").
    *   `DurationParamType`: Converts human-readable duration strings (e.g., "1h 30m", "5 days") into `datetime.timedelta` objects.
    *   `FileParamType` and `DirParamType`: Handle paths for `FlyteFile` and `FlyteDirectory` inputs, validating local paths and preparing remote paths.
    *   `JsonParamType`: Parses JSON strings or file paths (JSON/YAML) into Python dictionaries, lists, dataclasses, or Pydantic models. It supports nested structures and automatically deserializes to the target Python type.
    *   `UnionParamType`: Enables a single CLI option to accept values that could belong to one of several types (e.g., `Union[str, int]`), attempting conversion across the specified types.
    *   `PickleParamType`: Allows passing a Python object by reference using its module and variable name (e.g., `my_module:my_object`).
    *   `StructuredDatasetParamType`: Handles inputs for `StructuredDataset` types, accepting URIs or local paths.
    *   `EnumParamType`: Converts string inputs into corresponding Python `enum.Enum` members.

This robust type system ensures that inputs provided via the CLI are correctly interpreted and transformed, whether for local testing or remote deployment.

### Remote vs. Local Execution

The `--remote` flag significantly influences how CLI inputs are processed:

*   **Local Execution (default):** When `--remote` is not specified, `pyflyte run` executes the Flyte entity directly in your local Python environment. In this mode, the `FlyteLiteralConverter` returns native Python types (e.g., `str`, `int`, `datetime.datetime`, `dict`, `list`, `FlyteFile`, `FlyteDirectory`). This allows for direct debugging and testing of your Flyte code without interacting with a Flyte cluster.

*   **Remote Execution (`--remote`):** When `--remote` is used, `pyflyte run` registers and launches the Flyte entity on a configured Flyte cluster. In this scenario, the `FlyteLiteralConverter` converts all inputs into Flyte `Literal` objects. These `Literal` objects are then serialized and sent to the Flyte backend, ensuring that the data types and values are correctly understood by the remote execution engine. The `DynamicEntityLaunchCommand` is specifically designed to fetch and execute remote entities, dynamically generating input options based on the remote entity's interface.

The `RunLevelParams` object, specifically its `is_remote` property, is propagated through the Click context to inform the type conversion logic about the target execution environment.

### Best Practices and Considerations

*   **Use `--inputs-file` for Complex Inputs:** For workflows or tasks with many inputs, or inputs that are complex data structures (e.g., large JSON objects), using a YAML or JSON inputs file is highly recommended for readability and maintainability.
*   **Understand Type Mappings:** Be aware of how your Python types map to CLI options and how the custom parameter types handle specific formats (e.g., `datetime` strings, durations). Refer to the `click_types` definitions for detailed parsing rules.
*   **Distinguish Global vs. Entity Inputs:** Clearly separate global run-level parameters (like `--project`, `--image`) from your workflow/task/launch plan's specific inputs.
*   **Leverage `--remote` for Testing:** Use the default local execution mode for rapid iteration and debugging. Once confident, switch to `--remote` for testing against a real Flyte cluster.
*   **Error Handling:** The `ErrorHandlingCommand` ensures that any exceptions during CLI execution are caught and presented in a user-friendly format, aiding in debugging.
*   **Dynamic Command Generation:** The CLI's ability to dynamically generate commands and options based on your Python files or remote entities simplifies usage, as you don't need to manually define CLI arguments for each entity.
*   **Launch Plan Versioning:** When executing remote launch plans, you can specify a version (e.g., `my_lp:version`) or rely on the active or latest version if no version is provided.
<!--
key: summary_managing_cli_parameters_and_inputs_e4871817-ff99-46e3-b8c1-ac4445466e04
type: summary_end

-->
<!--
code_unit: flytekit.clis.sdk_in_container.run.RunLevelParams
code_unit_type: class
help_text: ''
key: example_ef7f4659-6af3-45f3-97d7-3996b7ea6b92
type: example

-->
<!--
code_unit: flytekit.interaction.click_types.JsonParamType
code_unit_type: class
help_text: ''
key: example_9552f91e-158f-4c99-a188-c28a1c9df5f8
type: example

-->
<!--
code_unit: flytekit.interaction.click_types.FileParamType
code_unit_type: class
help_text: ''
key: example_fa95980b-97fa-4040-a5f6-542078cce1b4
type: example

-->
<!--
code_unit: flytekit.interaction.click_types.DirParamType
code_unit_type: class
help_text: ''
key: example_fcec791a-72ed-4679-a9ab-f78bfe3f52f7
type: example

-->
<!--
code_unit: flytekit.interaction.click_types.DateTimeType
code_unit_type: class
help_text: ''
key: example_3d0815ef-5f82-4b37-a05e-142ce2000003
type: example

-->