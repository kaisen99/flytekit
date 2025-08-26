
<!--
help_text: ''
key: summary_cli_&_local_development_29809949-3fe9-41dd-91ba-39ae92f05a28
modules:
- flytekit.clis.sdk_in_container.build
- flytekit.clis.sdk_in_container.run
- flytekit.clis.sdk_in_container.serialize
- flytekit.clis.sdk_in_container.utils
- flytekit.clis.flyte_cli.main
- flytekit.tools.fast_registration
- flytekit.tools.ignore
- flytekit.interactive.vscode_lib.config
- flytekit.interactive.vscode_lib.decorator
- flytekitplugins.flyteinteractive.jupyter_lib.decorator
questions_to_answer: []
type: summary

-->
## CLI & Local Development

The Flyte Command Line Interface (CLI) is the primary tool for interacting with Flyte, enabling developers to manage and execute Flyte entities both locally and on a remote cluster. This section details the core capabilities for local development, including running and building Flyte entities, and leveraging interactive development environments.

### Running Flyte Entities

The `pyflyte run` command facilitates the execution of Flyte workflows, tasks, and launch plans. It supports running entities defined in local Python files and interacting with entities already registered on a remote Flyte deployment.

#### Executing Local Entities

To run a workflow or task defined in a local Python file, specify the file and the entity name. The CLI dynamically inspects the Python file, identifies available workflows, tasks, and launch plans, and generates command-line options corresponding to their inputs.

**Input Handling:**
Inputs for an entity can be provided directly as command-line arguments. For complex inputs or to manage inputs separately, the CLI supports reading them from a YAML or JSON file using the `--inputs-file` option. This allows for structured input definition, enhancing reproducibility and ease of use. For example:

```bash
pyflyte run my_workflow_file.py my_workflow --input_arg1 "value" --input_arg2 123
pyflyte run my_workflow_file.py my_workflow --inputs-file inputs.yaml
```

The `YamlFileReadingCommand` component handles parsing these input files, converting their contents into the appropriate command-line arguments for the target entity.

#### Executing Remote Entities

Beyond local files, `pyflyte run` can also execute entities registered on a remote Flyte cluster. This is achieved by specifying special subcommands: `remote-launchplan`, `remote-workflow`, or `remote-task`. These commands dynamically fetch available entities from the configured Flyte project and domain.

For instance, to list remote launch plans:
```bash
pyflyte run remote-launchplan
```

To execute a specific remote launch plan:
```bash
pyflyte run remote-launchplan <launch_plan_name>
```

The `RemoteEntityGroup` and `DynamicEntityLaunchCommand` components enable this dynamic interaction. They retrieve entity definitions from the remote Flyte instance, including their input interfaces, and then construct a command-line interface that mirrors the remote entity's expected inputs. This allows for seamless execution of remote entities as if they were local.

#### Execution Parameters

A wide range of parameters can be configured when running Flyte entities, whether locally or remotely. These parameters are managed by the `RunLevelParams` component and include:

*   **Project and Domain:** `--project` and `--domain` specify the Flyte project and domain for remote operations.
*   **Image Configuration:** `--image` allows specifying the container image to use for execution. Multiple images can be provided.
*   **Service Account:** `--service-account` assigns a Kubernetes service account for the execution.
*   **Execution Control:** `--wait-execution` to block until the execution completes, `--poll-interval` for status checks, `--name` to assign a custom name to the execution.
*   **Metadata and Resource Management:** `--envvars`, `--tags`, `--labels`, `--annotations` for adding metadata; `--max-parallelism` to control concurrent node execution; `--raw-output-data-prefix` for custom output storage locations.
*   **Remote Execution:** The `--remote` flag is crucial for submitting the execution to a Flyte cluster instead of running it locally.
*   **Caching:** `--overwrite-cache` to bypass existing cached results.

These parameters provide fine-grained control over how Flyte entities are executed, enabling developers to tailor runs to specific requirements.

### Building Container Images

The `pyflyte build` command is used to create Docker images containing your Flyte workflows and tasks, preparing them for registration and execution on a Flyte cluster. This process involves packaging your Python code and its dependencies into a container.

```bash
pyflyte build my_workflow_file.py my_workflow
```

The `BuildCommand` and `BuildWorkflowCommand` components orchestrate this process.

#### Fast Serialization

A key feature of the build process is "fast serialization," enabled by the `--fast` flag. When fast serialization is used, the generated Docker image does not include the source code of your Flyte entities. Instead, the serialized Flyte definitions (protobufs) are registered, and the actual source code is fetched at runtime from a pre-defined location (e.g., a Git repository or an S3 bucket). This can significantly reduce image size and build times, especially for large codebases.

```bash
pyflyte build --fast my_workflow_file.py my_workflow
```

The `BuildParams` component includes the `fast` option to control this behavior.

#### File Packaging and Ignoring

When building an image or preparing code for remote execution, the CLI needs to determine which files from your local project should be included in the container. The `copy` option (managed by `RunLevelParams`) controls this behavior:

*   `--copy auto` (default): Copies only the Python modules that are loaded by your Flyte entities. This is generally the most efficient option.
*   `--copy all`: Copies all files in the source root directory to the destination directory within the image. This behaves like the deprecated `--copy-all` flag.

To manage which files are included or excluded during packaging, Flyte leverages various ignore mechanisms, implemented by the `Ignore` family of components:

*   **`.flyteignore`**: A custom ignore file specific to Flyte, similar to `.gitignore`. Patterns defined here are used to exclude files. The `FlyteIgnore` component handles this.
*   **`.gitignore`**: If a Git repository is detected, patterns from `.gitignore` are respected to exclude version-controlled ignored files. The `GitIgnore` component provides this functionality.
*   **`.dockerignore`**: If a Dockerfile is present, patterns from `.dockerignore` can also be used for exclusion. The `DockerIgnore` component handles this.
*   **Standard Ignore Patterns**: A set of common patterns (e.g., for virtual environments, build artifacts) are applied by default. The `StandardIgnore` component manages these.

The `IgnoreGroup` component combines these different ignore mechanisms, ensuring that a file is excluded if any of the configured ignore rules match. This robust system allows developers to precisely control the contents of their build artifacts, optimizing image size and security.

### Interactive Development Environments

Flyte provides decorators to integrate interactive development environments (IDEs) directly into your task containers, enabling a powerful remote development and debugging experience. This allows you to connect to a running task and interact with its environment, inspect variables, and debug code as if it were running locally.

#### VSCode Integration

The `@vscode` decorator transforms a Flyte task into an interactive VSCode server environment. When a task decorated with `@vscode` is executed on a Flyte cluster, it launches a VSCode server within the task's container.

```python
from flytekit.interactive.vscode_lib.decorator import vscode
from flytekit import task

@vscode(port=8080, max_idle_seconds=3600)
@task
def my_debuggable_task(x: int, y: int) -> int:
    # Your task logic here
    result = x + y
    return result
```

Key features of the VSCode integration:

*   **Server Launch:** Downloads and launches the VSCode server within the container.
*   **Port Exposure:** Exposes a specified port (e.g., 8080) for connecting to the VSCode server.
*   **Idle Timeout:** Automatically terminates the server after a period of inactivity (`max_idle_seconds`).
*   **Task-First Execution:** The `run_task_first` option allows the original task function to execute first. If it fails, the VSCode server is launched, providing an environment for debugging the failure.
*   **Pre/Post Execution Hooks:** `pre_execute` and `post_execute` callbacks allow custom logic to run before the VSCode server setup or before its termination.
*   **Debugging Setup:** Prepares necessary files (e.g., Python debugging scripts, `launch.json`) to facilitate remote debugging.

The `vscode` decorator, along with `VscodeConfig`, manages the download of the VSCode server and extensions, the setup of the interactive environment, and the lifecycle of the VSCode server within the task container.

#### Jupyter Notebook Integration

Similarly, the `@jupyter` decorator enables launching a Jupyter Notebook server within a Flyte task container. This is ideal for interactive data exploration, prototyping, and debugging directly within the execution environment.

```python
from flytekitplugins.flyteinteractive.jupyter_lib.decorator import jupyter
from flytekit import task

@jupyter(port=8888, notebook_dir="/root/notebooks")
@task
def my_interactive_data_task(data_path: str) -> None:
    # Load data, perform analysis interactively
    pass
```

Key features of the Jupyter integration:

*   **Server Launch:** Starts a Jupyter Notebook server within the container.
*   **Port Exposure:** Configures the server to listen on a specified port (e.g., 8888).
*   **Notebook Directory:** Allows specifying the directory where Jupyter Notebooks will be stored and accessed (`notebook_dir`).
*   **Idle Timeout:** Automatically shuts down the server after a period of inactivity (`max_idle_seconds`).
*   **Task-First Execution:** Similar to VSCode, `run_task_first` can be used to run the original task first and launch Jupyter only upon failure.
*   **Pre/Post Execution Hooks:** `pre_execute` and `post_execute` callbacks provide hooks for custom setup or teardown.
*   **Example Notebook:** An example notebook is written to the specified directory to help users get started.

Both interactive decorators provide a `get_extra_config` method to expose the link type and port, which can be used by Flyte UI or other tools to provide direct access links to the interactive session.

### Common CLI Utilities

The CLI incorporates several utility components to ensure a consistent and robust user experience:

*   **Error Handling:** The `ErrorHandlingCommand` component wraps CLI commands to catch exceptions and present them to the user in a clear, formatted manner, including detailed stack traces when verbose logging is enabled. This improves the debugging experience for CLI users.
*   **Base Parameters:** The `PyFlyteParams` component defines common parameters available across many `pyflyte` commands, such as `--config` for specifying a Flyte configuration file, `--verbose` for controlling log verbosity, and `--pkgs` for specifying Python packages to scan. These parameters ensure a consistent interface for fundamental CLI operations.
<!--
key: summary_cli_&_local_development_29809949-3fe9-41dd-91ba-39ae92f05a28
type: summary_end

-->
<!--
code_unit: flytekit.clis.sdk_in_container.run.RunCommand
code_unit_type: class
help_text: ''
key: example_a68add34-32e7-4cdc-81e6-ba1ac33df570
type: example

-->
<!--
code_unit: flytekit.clis.sdk_in_container.build.BuildCommand
code_unit_type: class
help_text: ''
key: example_c284da99-a2a3-4877-88e1-ee9d73d1165e
type: example

-->
<!--
code_unit: flytekit.interactive.vscode_lib.decorator.vscode
code_unit_type: class
help_text: ''
key: example_282d70f5-bb8a-4beb-ba7f-3d191826542f
type: example

-->
<!--
code_unit: flytekitplugins.flyteinteractive.jupyter_lib.decorator.jupyter
code_unit_type: class
help_text: ''
key: example_28685a37-818d-4e8d-9d3c-1fc4d91890cf
type: example

-->