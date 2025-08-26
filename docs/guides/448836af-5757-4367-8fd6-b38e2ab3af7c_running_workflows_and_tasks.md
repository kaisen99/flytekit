
<!--
help_text: ''
key: summary_running_workflows_and_tasks_d32bd47e-27a7-407d-a973-c14a9fade025
modules:
- flytekit.clis.sdk_in_container.run
questions_to_answer: []
type: summary

-->
The `pyflyte run` command provides a comprehensive interface for executing Flyte workflows, tasks, and launch plans. It supports both local execution of entities defined in Python files and remote execution of entities already registered on a Flyte deployment.

## Running Entities

The `pyflyte run` command acts as the primary entry point. When invoked, it dynamically discovers available entities for execution.

### Running Local Entities

To execute a workflow, task, or launch plan defined within a local Python file, specify the filename as a subcommand to `pyflyte run`. The system automatically inspects the specified file, identifies all Flyte entities (workflows, tasks, and launch plans), and exposes them as further subcommands.

For example, if you have a file named `my_workflow_file.py` containing a workflow named `my_workflow`:

```bash
pyflyte run my_workflow_file.py my_workflow --input_param "value"
```

The `WorkflowCommand` class handles the parsing of the Python file, determining the project root, and loading the specified entity. It then constructs a command with appropriate input parameters derived from the entity's interface.

### Running Remote Entities

The `pyflyte run` command also facilitates the execution of entities that are already registered on a remote Flyte deployment. This is achieved through dedicated subcommands: `remote-launchplan`, `remote-workflow`, and `remote-task`.

These remote commands, powered by the `RemoteEntityGroup` and `DynamicEntityLaunchCommand` classes, dynamically fetch available entities from the configured Flyte project and domain.

To list available remote launch plans:

```bash
pyflyte run remote-launchplan
```

To execute a specific remote launch plan:

```bash
pyflyte run remote-launchplan my_launch_plan --input_param "value"
```

You can specify a particular version of a remote entity using the `<name>:<version>` format. If no version is provided, the latest version is used for tasks, and the active or latest version is used for launch plans.

```bash
pyflyte run remote-task my_task:v1.0.0 --another_input 123
```

## Configuring Execution Parameters

The `RunLevelParams` class defines a comprehensive set of parameters that control how an execution is performed. These parameters can be specified directly on the command line.

Key parameters include:

*   **`--project`**: Specifies the Flyte project for the execution.
*   **`--domain`**: Specifies the Flyte domain for the execution.
*   **`--image` (`-i`)**: Defines the Docker image to use for the execution. This is crucial for remote executions to ensure the correct environment. Multiple images can be specified.
*   **`--remote` (`-r`)**: A flag that, when set, instructs the system to register and run the workflow or task on a Flyte deployment instead of executing it locally. This flag is eagerly processed to determine the execution context.
*   **`--service-account`**: The service account to use for the execution, granting necessary permissions within the Flyte cluster.
*   **`--wait` (`--wait-execution`)**: If set, the CLI will block and wait for the execution to complete, displaying its status.
*   **`--poll-interval` (`-i`)**: The interval in seconds to poll for execution status when `--wait` is enabled.
*   **`--dump-snippet`**: Dumps a Python code snippet that can be used with `flyteremote` to load and inspect the created execution programmatically.
*   **`--overwrite-cache`**: Forces the re-execution of cached tasks, ignoring existing cache entries.
*   **`--envvars` (`--env`)**: Environment variables to set in the container, specified as `ENV_NAME=ENV_VALUE`. Multiple can be provided.
*   **`--tags`**: Arbitrary tags to associate with the execution for categorization and filtering.
*   **`--name`**: A custom name to assign to the execution. If not provided, a unique name is generated.
*   **`--labels`**: Key-value pairs (`key=value`) to attach as labels to the execution.
*   **`--annotations`**: Key-value pairs (`key=value`) to attach as annotations to the execution.
*   **`--raw-output-data-prefix`**: A file path prefix (e.g., `s3://my-bucket/`, `gs://my-bucket/`) to store raw output data. This is where Flytefile, Flytedirectory, and Structuredataset outputs are stored.
*   **`--max-parallelism`**: The maximum number of workflow nodes that can execute in parallel. A value of `0` indicates unlimited parallelism.
*   **`--disable-notifications`**: Disables notifications for the execution.
*   **`--limit`**: Used with remote entity listing commands (e.g., `pyflyte run remote-launchplan`) to limit the number of entities fetched.
*   **`--cluster-pool`**: Assigns the execution to a specific cluster pool.
*   **`--execution-cluster-label` (`--ecl`)**: Assigns the execution to a specific execution cluster label.

When running a local Python file with the `--remote` flag, the system automatically packages the necessary code. The `--copy` option (or the deprecated `--copy-all`) controls how files are copied into the execution image:
*   `--copy auto` (default): Copies only loaded Python modules.
*   `--copy all`: Copies all files in the source root directory.

## Providing Inputs

Inputs for workflows, tasks, and launch plans can be provided in several ways:

1.  **Command-line arguments**: Each input parameter of the entity is exposed as a command-line option.
    ```bash
    pyflyte run my_file.py my_workflow --my_string_input "hello" --my_int_input 123
    ```

2.  **Input file (`--inputs-file`)**: A YAML or JSON file containing the input values can be provided. This is handled by the `YamlFileReadingCommand` class, which parses the file and converts its contents into command-line arguments.
    `inputs.yaml`:
    ```yaml
    my_string_input: "hello from file"
    my_int_input: 456
    ```
    ```bash
    pyflyte run my_file.py my_workflow --inputs-file inputs.yaml
    ```

3.  **Standard input (`-`)**: Input values can also be piped directly to the command via standard input.
    ```bash
    echo '{"my_string_input": "hello from stdin", "my_int_input": 789}' | pyflyte run my_file.py my_workflow -
    ```

The system automatically infers the required input types from the Flyte entity's interface and generates appropriate Click options, including handling default values and required parameters.

## Execution Context

The `RunLevelParams` class maintains the execution context, including the `FlyteRemote` instance via its `remote_instance()` method. This instance is responsible for interacting with the Flyte backend for remote operations like fetching entities, registering code, and launching executions. The `RunLevelComputedParams` class stores dynamic information like the determined project root and module name during the execution setup.
<!--
key: summary_running_workflows_and_tasks_d32bd47e-27a7-407d-a973-c14a9fade025
type: summary_end

-->
<!--
code_unit: flytekit.clis.sdk_in_container.run.RunCommand
code_unit_type: class
help_text: ''
key: example_8365521f-eaa7-465a-b7d6-4796c1566f6b
type: example

-->
<!--
code_unit: flytekit.clis.sdk_in_container.run.WorkflowCommand
code_unit_type: class
help_text: ''
key: example_3a5468ed-e6ca-49b9-b67d-65bfcfccb35b
type: example

-->
<!--
code_unit: flytekit.clis.sdk_in_container.run.DynamicEntityLaunchCommand
code_unit_type: class
help_text: ''
key: example_629ee5e0-2205-486e-b4fe-1c2ab43012fe
type: example

-->