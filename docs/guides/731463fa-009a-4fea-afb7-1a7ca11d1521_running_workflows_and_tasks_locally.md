
<!--
help_text: ''
key: summary_running_workflows_and_tasks_locally_3b158f3c-a143-4942-a544-b235f2adf2aa
modules:
- flytekit.clis.sdk_in_container.run
- flytekit.clis.sdk_in_container.utils
- flytekit.interaction.click_types
questions_to_answer: []
type: summary

-->
The `run` command provides a powerful and flexible way to execute Flyte workflows, tasks, and launch plans directly from your local development environment. This capability extends to both entities defined in your local Python files and those already registered on a remote Flyte deployment.

### Executing Local Entities

The `run` command automatically discovers Flyte entities (workflows, tasks, and launch plans) within your Python files. When you invoke `pyflyte run` followed by a Python filename, it presents the available entities as subcommands.

The `RunCommand` class serves as the entry point for this functionality, scanning the current directory for Python files. The `WorkflowCommand` then takes over for a specific file, using the `Entities` class to identify all defined workflows, tasks, and launch plans.

**Example: Running a Task from a Local File**

Consider a file named `my_workflow.py` with a simple task:

```python
# my_workflow.py
from flytekit import task, workflow

@task
def my_task(x: int, y: str) -> str:
    return f"Input x: {x}, Input y: {y}"

@workflow
def my_wf(a: int = 10, b: str = "hello"):
    print(my_task(x=a, y=b))
```

To run `my_task` locally:

```bash
pyflyte run my_workflow.py my_task --x 5 --y "world"
```

This command executes `my_task` directly in your local Python environment.

**Example: Running a Workflow from a Local File**

To run `my_wf` from the same file:

```bash
pyflyte run my_workflow.py my_wf --a 20 --b "flyte"
```

### Executing Remote Entities

The `run` command also allows you to initiate executions of entities (launch plans, workflows, or tasks) that are already registered on a remote Flyte deployment. This is particularly useful for testing or re-running specific versions of deployed entities without needing to re-register them.

The `RemoteEntityGroup` class facilitates this by dynamically listing available remote entities. When you select a remote entity, the `DynamicEntityLaunchCommand` fetches its definition from the remote Flyte instance and constructs the necessary command-line arguments based on its inputs.

**Important Note:** When running remote entities, the execution itself occurs on the Flyte cluster, not locally. Your local `pyflyte run` command acts as a client to trigger and monitor this remote execution.

**Example: Running a Remote Launch Plan**

To list available remote launch plans:

```bash
pyflyte run remote-launchplan
```

To execute a specific remote launch plan (e.g., `my_remote_lp`):

```bash
pyflyte run remote-launchplan my_remote_lp --project flytesnacks --domain development --input_param "value"
```

You can also specify a version:

```bash
pyflyte run remote-launchplan my_remote_lp:v1.0.0 --project flytesnacks --domain development
```

Similarly, you can run remote workflows and tasks using `remote-workflow` and `remote-task` respectively.

### Providing Inputs

The `run` command offers flexible ways to provide inputs to your workflows and tasks.

#### Command-Line Arguments

For each input parameter of your task, workflow, or launch plan, the `run` command automatically generates a corresponding command-line option. The `WorkflowCommand` and `DynamicEntityLaunchCommand` classes are responsible for inspecting the entity's interface and creating these options.

Flytekit leverages custom Click parameter types (defined in `flytekit.interaction.click_types`) to handle various Python and Flyte-specific data types, ensuring correct parsing and validation of inputs from the command line.

**Supported Input Types:**

*   **Basic Types:** `str`, `int`, `float`, `bool`
*   **Enums:** Handled by `EnumParamType`.
*   **Durations:** Parsed from human-readable strings (e.g., "1 hour", "30m") by `DurationParamType`.
*   **Date/Time:** Supports various formats, including "now", "today", and relative offsets (e.g., "2023-01-01 - 1 day") via `DateTimeType`.
*   **JSON Objects:** Can accept JSON strings or paths to JSON/YAML files, converted by `JsonParamType`. This is crucial for complex inputs like `dict`, `list`, or custom dataclasses/Pydantic models.
*   **Files and Directories:** Paths to local or remote files/directories are handled by `FileParamType` and `DirParamType`, converting them to `FlyteFile` and `FlyteDirectory` objects.
*   **Structured Datasets:** Paths to structured datasets are handled by `StructuredDatasetParamType`.
*   **Union Types:** The `UnionParamType` attempts to convert the input value to one of the specified union types.
*   **Pickle Objects:** Allows passing Python objects by referencing their module and variable name (e.g., `my_module:my_object`) using `PickleParamType`.

#### Input Files (YAML/JSON)

For complex inputs or a large number of parameters, you can provide inputs via a YAML or JSON file using the `--inputs-file` option. The `YamlFileReadingCommand` handles parsing these files.

**Example: Using an Inputs File**

`inputs.yaml`:

```yaml
x: 100
y: "file_input"
```

Run the task:

```bash
pyflyte run my_workflow.py my_task --inputs-file inputs.yaml
```

You can also pipe inputs from stdin by using `-` as the last argument:

```bash
echo '{"x": 200, "y": "stdin_input"}' | pyflyte run my_workflow.py my_task -
```

### Configuring Execution Parameters

The `RunLevelParams` class encapsulates a comprehensive set of command-line options that control how your workflows and tasks are executed, whether locally or remotely. These parameters allow fine-grained control over the execution environment, metadata, and behavior.

Key parameters include:

*   **`--project` / `--domain`**: Specifies the Flyte project and domain for remote executions. These are mandatory for remote runs.
*   **`-i` / `--image`**: Defines the Docker image to use for remote executions. You can specify multiple images.
*   **`--service-account`**: Sets the service account for the execution, controlling permissions on the Flyte cluster.
*   **`--wait` / `--wait-execution`**: If set, the CLI will block and wait for the remote execution to complete, displaying its status.
*   **`--poll-interval`**: Specifies the interval in seconds to poll for execution status when `--wait` is enabled.
*   **`--dump-snippet`**: Prints a Python code snippet that can be used with `FlyteRemote` to load and inspect the execution programmatically.
*   **`--copy`**: Controls how local source code is copied to the remote execution environment.
    *   `auto` (default): Copies only loaded Python modules.
    *   `all`: Copies all files in the source root directory. This is useful if your code has non-Python dependencies or data files.
*   **`--envvars` / `--env`**: Sets environment variables within the execution container (e.g., `--envvars MY_VAR=my_value`).
*   **`--tags` / `--tag`**: Attaches tags to the execution for easier filtering and organization.
*   **`--name`**: Assigns a custom name to the execution.
*   **`--labels` / `--label`**: Attaches key-value labels to the execution (e.g., `--labels team=data_science`).
*   **`--annotations` / `--annotation`**: Attaches key-value annotations to the execution for additional metadata.
*   **`--raw-output-data-prefix`**: Specifies a custom location (e.g., `s3://my-bucket/outputs/`) for storing raw output data like `FlyteFile` or `FlyteDirectory` artifacts.
*   **`--max-parallelism`**: Limits the number of parallel nodes that can execute within a workflow.
*   **`--disable-notifications`**: Prevents notifications from being sent for the execution.
*   **`-r` / `--remote`**: This crucial flag determines whether the execution is truly local or if it should be registered and run on a remote Flyte deployment. When this flag is present, the `RunLevelParams` class initializes a `FlyteRemote` instance to interact with the cluster.
*   **`--limit`**: Used with remote entity listing commands to limit the number of entities fetched.
*   **`--cluster-pool`**: Assigns the execution to a specific cluster pool.
*   **`--execution-cluster-label` / `--ecl`**: Assigns the execution to a specific execution cluster label.

**Example: Running a Local Workflow Remotely with Custom Parameters**

```bash
pyflyte run my_workflow.py my_wf \
    --remote \
    --project my_project \
    --domain development \
    --image my_custom_image:latest \
    --service-account my_service_account \
    --wait \
    --name "my-custom-run" \
    --labels department=eng \
    --envvars DEBUG=true \
    --a 30 --b "remote_run"
```

This command takes the `my_wf` workflow from `my_workflow.py`, registers it (if not already registered), and then executes it on the `my_project` / `development` domain using `my_custom_image:latest`, waiting for its completion.

### Best Practices and Considerations

*   **Local vs. Remote Execution Context:** Understand the distinction. Running a local entity *without* `--remote` executes your Python code directly on your machine. Running a local entity *with* `--remote` or running a `remote-` entity means your code (or the remote entity definition) is packaged and executed on the Flyte cluster.
*   **Debugging:** Local execution is invaluable for rapid iteration and debugging of your task and workflow logic without the overhead of remote deployment.
*   **Environment Consistency:** When running locally, ensure your Python environment has all necessary dependencies installed that would also be present in your remote execution image. This helps prevent discrepancies.
*   **`--copy` Flag for Remote Runs:** When running a local file remotely, the `--copy` flag is critical. If your workflow or task depends on other local files (e.g., utility scripts, data files), ensure `--copy all` is used to bundle them into the remote execution. For standard Python module dependencies, `auto` is usually sufficient as it copies imported modules.
<!--
key: summary_running_workflows_and_tasks_locally_3b158f3c-a143-4942-a544-b235f2adf2aa
type: summary_end

-->
<!--
code_unit: flytekit.clis.sdk_in_container.run.RunCommand
code_unit_type: class
help_text: ''
key: example_b7ed29aa-4cf1-43ee-99ce-6b8476db72e9
type: example

-->
<!--
code_unit: flytekit.clis.sdk_in_container.run.WorkflowCommand
code_unit_type: class
help_text: ''
key: example_9e9a03d9-fac3-446e-98a2-d2cb1382d53d
type: example

-->
<!--
code_unit: flytekit.clis.sdk_in_container.run.YamlFileReadingCommand
code_unit_type: class
help_text: ''
key: example_fc520ca3-f967-48d0-9d30-b1559e168465
type: example

-->
<!--
code_unit: flytekit.clis.sdk_in_container.run.RunLevelParams
code_unit_type: class
help_text: ''
key: example_9567a2c0-7dcb-431e-88a6-71a64dcd4b87
type: example

-->