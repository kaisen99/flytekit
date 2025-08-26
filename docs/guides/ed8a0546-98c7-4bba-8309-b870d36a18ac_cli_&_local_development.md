
<!--
help_text: ''
key: summary_cli_&_local_development_d8674b25-22d4-44a6-82a3-c52bc01c528f
modules:
- flytekit.clis.sdk_in_container.build
- flytekit.clis.sdk_in_container.run
- flytekit.clis.sdk_in_container.serialize
- flytekit.clis.sdk_in_container.utils
- flytekit.clis.flyte_cli.main
- flytekit.interaction.click_types
- flytekit.interaction.rich_utils
- flytekit.tools.fast_registration
- flytekit.tools.ignore
- flytekit.tools.repo
- setup
- plugins.setup
questions_to_answer: []
type: summary

-->
The `pyflyte` command-line interface (CLI) provides a robust set of tools for developing, testing, and deploying Flyte workflows and tasks. It supports both local execution for rapid iteration and seamless interaction with a remote Flyte cluster.

### Local Development and Execution

The `pyflyte run` command is the primary entry point for executing Flyte entities directly from Python files. This capability is managed by the `RunCommand` group, which dynamically discovers Python files in the current directory. For each discovered file, a subcommand is created, represented by `WorkflowCommand`.

To execute a specific workflow or task within a Python file:

```bash
pyflyte run <filename.py> <workflow_or_task_name> [OPTIONS]
```

For example, to run a workflow named `my_workflow` in `my_file.py`:

```bash
pyflyte run my_file.py my_workflow
```

The CLI dynamically generates command-line options for each input parameter of the workflow or task. These options are derived from the Python type hints of the entity's inputs, leveraging `FlyteLiteralConverter` and various `click.ParamType` implementations (e.g., `FileParamType`, `JsonParamType`, `DateTimeType`, `DurationParamType`). This ensures type-safe input parsing directly from the command line.

**Input Handling:**
Inputs can be provided directly as command-line arguments:

```bash
pyflyte run my_file.py my_workflow --input_arg "value" --another_arg 123
```

For complex inputs, such as dictionaries or lists, or when managing many inputs, the `YamlFileReadingCommand` enables loading inputs from a YAML or JSON file, or even from standard input (`-`):

```bash
# inputs.yaml
# input_arg: "value"
# complex_input:
#   key: "val"
#   list_of_items: [1, 2, 3]

pyflyte run my_file.py my_workflow --inputs-file inputs.yaml
```

### Building Container Images

Before deploying Flyte entities to a remote cluster, they must be packaged into a Docker image. The `pyflyte build` command facilitates this process. The `BuildCommand` group orchestrates the image creation, with `BuildWorkflowCommand` handling the build process for individual workflows or tasks within a specified Python file.

To build an image for a workflow or task:

```bash
pyflyte build <filename.py> <workflow_or_task_name> [OPTIONS]
```

For example, to build an image for `my_workflow` in `my_file.py`:

```bash
pyflyte build my_file.py my_workflow
```

**Fast Serialization:**
The `--fast` option, managed by `BuildParams` and `SerializationMode`, enables fast serialization. This mode optimizes the image build by not including the source code directly in the image, which can significantly reduce image size and build time. This is particularly useful during rapid development cycles where code changes are frequent.

```bash
pyflyte build my_file.py my_workflow --fast
```

**File Packaging and Exclusion:**
When building an image, the CLI determines which files from the local project root should be copied into the container. The `RunLevelParams` class includes a `--copy` option that controls this behavior:
*   `--copy auto` (default): Copies only the Python modules that are loaded by the Flyte entities.
*   `--copy all`: Copies all files in the source root directory to the destination. This behaves like the deprecated `--copy-all` flag.

File exclusion during packaging is managed by the `Ignore` classes:
*   `FlyteIgnore`: Uses patterns defined in a `.flyteignore` file.
*   `DockerIgnore`: Uses patterns defined in a `.dockerignore` file.
*   `GitIgnore`: Leverages `git ls-files -io --exclude-standard` to respect `.gitignore` rules.
*   `StandardIgnore`: Applies a set of default ignore patterns.

The `IgnoreGroup` aggregates these different ignore strategies, ensuring that if any configured ignore rule matches a file, it is excluded from the build. This allows developers to precisely control the contents of their build artifacts, minimizing image size and build complexity.

### Remote Interaction

The `pyflyte` CLI extends its capabilities to interact with a remote Flyte cluster, allowing for registration and execution of entities. The `RunCommand` group includes subcommands for remote operations, such as `remote-launchplan`, `remote-workflow`, and `remote-task`, which are managed by `RemoteEntityGroup`.

To execute a remote LaunchPlan, Workflow, or Task:

```bash
pyflyte run --remote remote-launchplan <launchplan_name>[:version] [OPTIONS]
pyflyte run --remote remote-workflow <workflow_name>[:version] [OPTIONS]
pyflyte run --remote remote-task <task_name>[:version] [OPTIONS]
```

The `DynamicEntityLaunchCommand` is responsible for fetching the specified remote entity and dynamically generating its input parameters as CLI options, similar to local execution. If a version is not provided, the latest version is used for tasks, and the active or latest version for launch plans.

### Execution Customization

The `RunLevelParams` class centralizes a wide array of options to customize the execution environment and behavior for both local and remote runs. These options are exposed as CLI flags:

*   `--project`, `--domain`: Specify the Flyte project and domain for remote executions.
*   `-i`, `--image`: Define the Docker image(s) to use for the execution. Multiple images can be specified.
*   `--service-account`: Assign a service account for the execution on the cluster.
*   `--wait`, `--wait-execution`: Block the CLI until the remote execution completes.
*   `--poll-interval`: Set the polling interval (in seconds) to check execution status when waiting.
*   `--dump-snippet`: Output a Python code snippet to load the execution using `flyteremote`.
*   `--overwrite-cache`: Force overwriting of cached results.
*   `--envvars`, `--env`: Pass environment variables to the container (e.g., `--env MY_VAR=my_value`).
*   `--tags`, `--tag`: Attach tags to the execution for categorization.
*   `--name`: Assign a custom name to the execution.
*   `--labels`, `--label`: Add key-value labels to the execution (e.g., `--label team=data`).
*   `--annotations`, `--annotation`: Add key-value annotations to the execution.
*   `--raw-output-data-prefix`: Specify a custom prefix for storing raw output data (e.g., `s3://my-bucket/outputs`).
*   `--max-parallelism`: Control the maximum number of parallel nodes in a workflow execution.
*   `--disable-notifications`: Suppress notifications for the execution.
*   `--cluster-pool`: Assign the execution to a specific cluster pool.
*   `--execution-cluster-label`, `--ecl`: Assign the execution to a specific execution cluster label.

### Error Handling and Verbosity

The `ErrorHandlingCommand` utility ensures consistent and user-friendly error reporting across the CLI. It catches exceptions during command invocation and presents them clearly. The `--verbose` flag, inherited from `PyFlyteParams`, controls the logging level, providing more detailed output for debugging purposes.

### Best Practices

*   **Leverage `.flyteignore`:** For efficient image builds, create a `.flyteignore` file in your project root to explicitly exclude unnecessary files and directories from being copied into the Docker image. This reduces image size and build times.
*   **Manage Dependencies:** Ensure all necessary Python dependencies for your workflows and tasks are specified in your `requirements.txt` or equivalent, as these will be installed in the Docker image.
*   **Differentiate Local vs. Remote:** Understand that local execution runs your code directly on your machine, while remote execution packages your code into a Docker image and runs it on a Flyte cluster. Use local execution for rapid development and remote for testing against the actual Flyte environment.
*   **Use Configuration Files:** For common project and domain settings, configure them in your `~/.flyte/config` file. CLI arguments will override these settings, providing flexibility. The `_FlyteSubCommand` ensures these configurations are consistently applied across commands.
<!--
key: summary_cli_&_local_development_d8674b25-22d4-44a6-82a3-c52bc01c528f
type: summary_end

-->
<!--
code_unit: flytekit.clis.sdk_in_container.run.RunCommand
code_unit_type: class
help_text: ''
key: example_8ae8da5c-d0db-4952-a4fd-568254df8ddd
type: example

-->
<!--
code_unit: flytekit.clis.sdk_in_container.build.BuildCommand
code_unit_type: class
help_text: ''
key: example_5d0f0162-5543-43ab-9698-e13e5ed5409f
type: example

-->