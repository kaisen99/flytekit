
<!--
help_text: ''
key: summary_cli_configuration_and_global_parameters_a0b85dee-f91d-4c2f-8a51-07ba3f56553e
modules:
- flytekit.clis.sdk_in_container.run
- flytekit.clis.sdk_in_container.utils
- flytekit.clis.flyte_cli.main
questions_to_answer: []
type: summary

-->
The Flyte CLI and `pyflyte` commands offer extensive configuration options and global parameters to control execution behavior, resource allocation, and interaction with the Flyte platform. Understanding these parameters is crucial for effectively managing and deploying Flyte entities.

### CLI Configuration Hierarchy

The Flyte CLI processes configuration settings in a specific order, with command-line arguments taking precedence over values defined in configuration files.

1.  **Configuration File:** By default, the CLI attempts to load configuration from `~/.flyte/config`. This file can define global settings such as the Flyte host, project, and domain.
2.  **Command-Line Arguments:** Any parameters explicitly provided on the command line override corresponding settings from the configuration file. This allows for flexible, per-command adjustments without modifying the default configuration.

For example, if `host` is set in `~/.flyte/config`, but a different host is specified using `--host` on the command line, the command-line value is used. This behavior is managed by the `_FlyteSubCommand` class, which ensures global parameters like `project`, `domain`, `host`, `config`, `insecure`, and `cacert` are correctly propagated and overridden.

### Global CLI Parameters

Several parameters apply broadly across various Flyte CLI commands, influencing how the CLI connects to and interacts with the Flyte platform. These are often specified directly after `flyte-cli` or `pyflyte`.

*   **`--config`**: Specifies the path to a Flyte configuration file. If not provided, the CLI defaults to `~/.flyte/config`.
*   **`--host`**: Defines the Flyte API endpoint (e.g., `localhost:30081`). This is a critical parameter for connecting to a Flyte deployment.
*   **`--insecure`**: A flag indicating whether to use an insecure connection (HTTP) to the Flyte API. By default, secure connections (HTTPS) are attempted.
*   **`--cacert`**: Specifies the path to a CA certificate file for secure connections.
*   **`--project`**: Sets the Flyte project context for the command.
*   **`--domain`**: Sets the Flyte domain context for the command.
*   **`--verbose`**: Increases the verbosity of the CLI output, providing more detailed logs for debugging. This parameter is handled by the `ErrorHandlingCommand` to ensure consistent error reporting and logging levels.

**Example:**

```bash
flyte-cli --host my.flyte.example.com --project my-dev-project --domain development list-workflows
```

### `pyflyte run` Execution Parameters

The `pyflyte run` command, powered by the `RunCommand` and `WorkflowCommand` classes, provides a comprehensive set of parameters to control the execution of workflows, tasks, and launch plans. These parameters are encapsulated within the `RunLevelParams` class, which extends `PyFlyteParams` to include execution-specific options.

These parameters allow fine-grained control over how your code is packaged, where it runs, and how its execution is managed on the Flyte platform.

*   **`--project`**: (String) The Flyte project to which the execution belongs.
*   **`--domain`**: (String) The Flyte domain within the project for the execution.
*   **`--remote`, `-r`**: (Flag) Specifies that the workflow or task should be registered and executed on a remote Flyte deployment. This is a crucial flag that switches the execution context from local simulation to remote interaction.
*   **`--image`, `-i`**: (Multiple, String) Specifies the container image(s) to use for the execution. Can be specified multiple times for different image names.
    *   **Example:** `--image my_image:latest --image another_image:v1`
*   **`--service-account`**: (String) The Kubernetes service account to use for the execution, controlling permissions within the cluster.
*   **`--wait`, `--wait-execution`**: (Flag) If set, the CLI waits for the execution to complete and displays its status.
*   **`--poll-interval`, `-i`**: (Integer) When `--wait` is enabled, this sets the interval (in seconds) at which the CLI polls for execution status updates.
*   **`--dump-snippet`**: (Flag) If set, the CLI outputs a Python code snippet that can be used with `flyteremote` to load and interact with the created execution programmatically.
*   **`--overwrite-cache`**: (Flag) Forces the re-execution of cached tasks, ignoring existing cached results.
*   **`--envvars`, `--env`**: (Multiple, String) Environment variables to set within the container where the task or workflow runs. Format: `KEY=VALUE`.
    *   **Example:** `--envvars MY_VAR=my_value --envvars ANOTHER_VAR=another_value`
*   **`--tags`, `--tag`**: (Multiple, String) Arbitrary tags to associate with the execution for categorization and filtering.
*   **`--name`**: (String) A custom name to assign to the execution. If not provided, a unique name is generated.
*   **`--labels`, `--label`**: (Multiple, String) Key-value labels to attach to the execution. Format: `key=value`.
    *   **Example:** `--labels team=data --labels env=production`
*   **`--annotations`, `--annotation`**: (Multiple, String) Key-value annotations to attach to the execution, often used for metadata or external system integration. Format: `key=value`.
*   **`--raw-output-data-prefix`, `--raw-data-prefix`**: (String) A file path prefix (e.g., `s3://my-bucket/`, `gs://my-bucket/`) where raw output data (Flytefile, Flytedirectory, Structuredataset) will be stored. If not specified, data is stored in the default configured location.
*   **`--max-parallelism`**: (Integer) The maximum number of workflow nodes that can execute concurrently. A value of `0` indicates unlimited parallelism.
*   **`--disable-notifications`**: (Flag) Disables email or other configured notifications for the execution.
*   **`--limit`**: (Integer) Used when listing remote entities (e.g., `remote-launchplan`) to limit the number of entities fetched. Default is 50.
*   **`--cluster-pool`**: (String) Assigns the newly created execution to a specific cluster pool, if configured in the Flyte deployment.
*   **`--execution-cluster-label`, `--ecl`**: (String) Assigns the execution to a specific execution cluster label, influencing where the execution runs.
*   **`--destination-dir`**: (String) The directory inside the container image where the packaged code will be copied. Default is `.`.
*   **`--copy`**: (Choice: `all`, `auto`) Specifies how source files are copied into the container image.
    *   `all`: Copies all files in the source root directory. (Behaves like the deprecated `--copy-all` flag).
    *   `auto`: Copies only Python modules that are loaded during the packaging process. This is the default and recommended for efficiency.
*   **`--copy-all`**: (Flag, Deprecated) Copies all files in the source root directory to the destination directory. Use `--copy all` instead.

**Example `pyflyte run` usage:**

```bash
pyflyte run --remote --project flytesnacks --domain development \
    --image my-custom-image:v1.0 \
    --service-account my-service-account \
    --wait --poll-interval 10 \
    --envvars MY_ENV=test_value \
    --labels team=ml --name my-first-remote-run \
    my_workflow_file.py my_workflow_function
```

### Input Handling for Executions

When executing workflows, tasks, or launch plans, inputs can be provided in several ways, managed by the `YamlFileReadingCommand` class:

*   **Command-Line Arguments:** Individual inputs can be passed directly as command-line arguments, using the format `--input_name value`. The CLI dynamically generates these options based on the entity's interface.
*   **`--inputs-file`**: Specifies a path to a YAML or JSON file containing all inputs for the execution. This is particularly useful for complex inputs or when managing many parameters.
    *   **Example `inputs.yaml`:**
        ```yaml
        input_string: "hello"
        input_int: 123
        input_list: [1, 2, 3]
        ```
    *   **Usage:** `pyflyte run my_file.py my_workflow --inputs-file inputs.yaml`
*   **Standard Input (`-`)**: Inputs can be piped directly to the command via standard input. This is useful for programmatic execution or when inputs are generated on the fly.
    *   **Usage:** `cat inputs.json | pyflyte run my_file.py my_workflow -`

The `YamlFileReadingCommand` intelligently parses these inputs, converting them into the appropriate format for the Flyte entity. Boolean flags are handled correctly (e.g., `--my-flag` for `True`, absence for `False`), and complex types are serialized as JSON strings.

### Executing Remote Entities

The `pyflyte run` command also supports executing entities (launch plans, workflows, tasks) that are already registered on a remote Flyte deployment without needing the local Python file. This functionality is provided by the `RemoteEntityGroup` and `DynamicEntityLaunchCommand` classes.

*   **`pyflyte run remote-launchplan <name>[:<version>]`**: Executes a launch plan from the remote Flyte deployment. If a version is not specified, the active launch plan is used, or the latest version by creation time if no active one is found.
*   **`pyflyte run remote-workflow <name>[:<version>]`**: Executes a workflow from the remote Flyte deployment. If a version is not specified, the latest version by creation time is used.
*   **`pyflyte run remote-task <name>[:<version>]`**: Executes a task from the remote Flyte deployment. If a version is not specified, the latest version by creation time is used.

When executing remote entities, the CLI dynamically fetches the entity's interface from the Flyte API and generates the appropriate command-line options for its inputs. This allows you to provide inputs to remote entities just as you would for local ones.

**Example:**

```bash
# Execute a remote launch plan named 'my_lp' with a specific version
pyflyte run --remote --project flytesnacks --domain development \
    remote-launchplan my_lp:abcdef123 --input_param "some_value"

# Execute the latest version of a remote task named 'my_task'
pyflyte run --remote --project flytesnacks --domain development \
    remote-task my_task --task_input 42
```

The `RemoteEntityGroup` handles listing available remote entities, providing a convenient way to discover what can be executed. It uses the `--limit` parameter from `RunLevelParams` to control the number of entities fetched from the remote API.
<!--
key: summary_cli_configuration_and_global_parameters_a0b85dee-f91d-4c2f-8a51-07ba3f56553e
type: summary_end

-->
<!--
code_unit: pyflyte --project my-project --domain development run ...
code_unit_type: class
help_text: ''
key: example_2f140030-6c1d-4e93-9fc5-492ac73b59ad
type: example

-->
<!--
code_unit: pyflyte run --image my-custom-image --service-account my-sa ...
code_unit_type: class
help_text: ''
key: example_a1488e5e-09ab-480a-95c9-e6dbee7b4406
type: example

-->