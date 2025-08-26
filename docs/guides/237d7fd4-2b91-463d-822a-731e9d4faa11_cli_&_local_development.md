
<!--
help_text: ''
key: summary_cli_&_local_development_12f89ad1-9324-4eb2-b540-7b6d5dc1d10f
modules:
- flytekit.clis.sdk_in_container.build
- flytekit.clis.sdk_in_container.run
- flytekit.clis.sdk_in_container.serialize
- flytekit.clis.sdk_in_container.utils
- flytekit.clis.flyte_cli.main
- flytekit.interactive.vscode_lib.config
- flytekit.interactive.vscode_lib.decorator
- flytekitplugins.flyteinteractive.jupyter_lib.decorator
- flytekit.deck.deck
- flytekit.deck.renderer
- flytekit.tools.fast_registration
- flytekit.tools.ignore
- flytekit.tools.repo
- flytekit.image_spec.default_builder
- flytekit.image_spec.image_spec
- flytekit.image_spec.noop_builder
- flytekit.interfaces.cli_identifiers
- flytekit.interaction.click_types
- flytekit.interaction.rich_utils
- setup
- plugins.setup
questions_to_answer: []
type: summary

-->
The Flyte CLI provides a robust set of commands for local development, testing, and interaction with a Flyte cluster. It enables developers to run Flyte tasks, workflows, and launch plans directly from their local environment, build custom container images, and leverage interactive debugging tools.

### Running Flyte Entities

The `pyflyte run` command facilitates executing Flyte tasks, workflows, and launch plans. It supports both local execution for rapid iteration and remote execution for testing against a live Flyte cluster.

#### Local Execution

To run a Flyte entity locally, specify the Python file containing the entity and the entity's name. For example, if `my_workflow.py` defines a workflow named `my_wf`:

```bash
pyflyte run my_workflow.py my_wf
```

The CLI automatically discovers Flyte entities within the specified file. When running locally, the system executes the Python code directly without containerization, making it ideal for quick development cycles.

#### Remote Execution

To execute an entity on a remote Flyte cluster, use the `--remote` flag. This command registers the entity with the cluster and then launches an execution.

```bash
pyflyte run --remote my_workflow.py my_wf --project flytesnacks --domain development
```

When `--remote` is specified, the CLI automatically packages the necessary code and dependencies into a Docker image (if not already built or specified) and pushes it to the configured registry. It then initiates an execution on the Flyte cluster, providing a link to monitor its progress.

#### Input Handling

Inputs for tasks, workflows, or launch plans can be provided directly as command-line arguments or through a YAML/JSON file.

*   **Command-line arguments:**
    ```bash
    pyflyte run my_workflow.py my_wf --input_param "hello" --another_param 123
    ```
    The CLI automatically infers the expected Python type for each input and converts the provided string arguments accordingly. Custom Click parameter types are used internally to handle Flyte-specific types like `FlyteFile`, `FlyteDirectory`, `StructuredDataset`, `datetime.timedelta`, and `datetime.datetime`. For instance, `FlyteFile` and `FlyteDirectory` inputs can accept local file paths or remote URIs. JSON and YAML inputs are also supported, allowing complex data structures to be passed.

*   **YAML/JSON input file:**
    Use the `--inputs-file` flag to provide inputs from a file. This is particularly useful for complex inputs or when managing many parameters.

    `inputs.yaml`:
    ```yaml
    input_param: "hello"
    another_param: 123
    ```

    ```bash
    pyflyte run my_workflow.py my_wf --inputs-file inputs.yaml
    ```
    The `YamlFileReadingCommand` class handles parsing these files, supporting both YAML and JSON formats.

#### Common Parameters

The `RunLevelParams` class defines a comprehensive set of parameters available for `pyflyte run` and `pyflyte build` commands. These parameters control various aspects of execution and image building:

*   `--project`, `--domain`: Specify the Flyte project and domain for remote executions.
*   `--image`: Define the Docker image to use for the execution. This can be a pre-existing image or a reference to an `ImageSpec` for on-the-fly building.
*   `--service-account`: Set the service account for the execution on the cluster.
*   `--wait-execution`: Block the CLI until the remote execution completes.
*   `--poll-interval`: Configure the polling frequency when waiting for execution completion.
*   `--dump-snippet`: Output a Python snippet to load the execution using `FlyteRemote`.
*   `--envvars`, `--tags`, `--name`, `--labels`, `--annotations`: Provide metadata and configuration for the remote execution.
*   `--raw-output-data-prefix`: Specify a custom location for raw output data (e.g., `s3://my-bucket/outputs`).
*   `--max-parallelism`: Control the maximum number of parallel nodes in a workflow.
*   `--disable-notifications`: Suppress notifications for the execution.
*   `--limit`: Limit the number of remote entities fetched for listing commands.
*   `--cluster-pool`, `--execution-cluster-label`: Assign the execution to specific cluster resources.

### Building Custom Docker Images

The `pyflyte build` command is used to create Docker images tailored for Flyte tasks and workflows. This is essential for deploying custom code and dependencies to a Flyte cluster.

```bash
pyflyte build my_workflow.py my_wf
```

The `BuildCommand` and `BuildWorkflowCommand` classes manage the image building process.

#### Image Specification

The `ImageSpec` class provides a declarative way to define the characteristics of a Docker image. It allows specifying:

*   `name`: The name of the image.
*   `python_version`: The Python version to use.
*   `builder`: The image builder to use (e.g., `default` for Docker/BuildKit, `noop` if the image already exists).
*   `base_image`: The base Docker image to build upon.
*   `packages`, `conda_packages`, `apt_packages`: Lists of Python, Conda, and APT packages to install.
*   `requirements`: Path to a `requirements.txt` file for Python dependencies.
*   `env`: Environment variables to set in the image.
*   `commands`: Custom shell commands to execute during the build process.
*   `source_root`: The root directory containing the source code to be included.
*   `source_copy_mode`: Controls how source files are copied into the image (`all`, `auto`, `no_copy`). `auto` copies only loaded Python modules, while `all` copies the entire source root.
*   `copy`: A list of specific files or directories to copy into the image's `/root` directory.
*   `registry`: The Docker registry to push the built image to.
*   `platform`: Target platform for the image (e.g., `linux/amd64`).
*   `pip_index`, `pip_extra_index_url`, `pip_secret_mounts`, `pip_extra_args`: Advanced pip configuration.
*   `runtime_packages`: Packages to be installed at runtime within the container, requiring `pip` to be present in the base image.

**Example `ImageSpec` usage:**

```python
from flytekit.image_spec import ImageSpec

my_image = ImageSpec(
    name="my-custom-image",
    python_version="3.9",
    packages=["pandas", "scikit-learn"],
    commands=["apt-get update -y", "apt-get install -y git"],
    source_root=".", # Copy current directory
    source_copy_mode="auto",
)

@task(container_image=my_image)
def my_task(a: int) -> int:
    return a + 1
```

The `ImageBuildEngine` manages the building process, automatically selecting the appropriate builder (e.g., `DefaultImageBuilder` for Docker/BuildKit). The `ImageSpec` generates a unique hash based on its configuration and dependencies, which is used as the image tag, ensuring reproducibility and efficient caching.

#### Fast Serialization

The `--fast` flag in `pyflyte build` enables fast serialization. In this mode, the source code is not embedded directly into the Docker image. Instead, it is packaged separately and uploaded to the Flyte blob store. This allows for quicker image builds and smaller image sizes, as only the environment and dependencies are part of the base image. When an execution runs, the Flyte engine downloads the source code package.

**Consideration:** Using `--fast` means the Docker image itself does not contain the application source code. This can impact debugging workflows that rely on source code presence within the container.

#### Ignoring Files

During the image building and packaging process, certain files or directories can be excluded. The system supports various ignore mechanisms:

*   `.dockerignore`: Standard Docker ignore patterns.
*   `.flyteignore`: Flyte-specific ignore patterns.
*   `.gitignore`: Git-ignored files (if Git is available).
*   Standard ignore patterns: Common build artifacts and temporary files.

The `IgnoreGroup` class combines these different ignore strategies, ensuring that any file matched by any of the configured ignore rules is excluded from the build context. This helps in creating leaner images and faster builds.

### Interactive Development and Debugging

Flyte provides decorators to enable interactive development environments directly within your Flyte tasks, facilitating debugging and data exploration in a remote cluster environment. These decorators only activate when a task is executed remotely.

#### VSCode Integration

The `@vscode` decorator transforms a Flyte task to launch a VSCode server within its container. This allows developers to connect to the running task from their local VSCode IDE, enabling remote development and debugging capabilities.

```python
from flytekit import task
from flytekit.interactive import vscode

@vscode(max_idle_seconds=3600, port=8080)
@task
def debuggable_task(a: int, b: int) -> int:
    # Your task logic here
    result = a + b
    return result
```

**Key features:**

*   **Remote VSCode Server:** Launches `code-server` inside the task container.
*   **Configurable:** Set `max_idle_seconds` for automatic shutdown after inactivity, and `port` for server access.
*   **Pre/Post Execution Hooks:** `pre_execute` and `post_execute` callbacks allow custom setup or cleanup logic.
*   **Task Resumption:** The decorator prepares scripts for interactive debugging and task resumption, allowing you to pause the VSCode session and continue the original task execution.
*   **`run_task_first`:** If set to `True`, the user's task function executes first. The VSCode server only launches if the task fails, providing a debugging environment for failed executions.

**Usage:**
When a task decorated with `@vscode` is executed remotely, the Flyte UI or CLI output will provide a link to access the VSCode server running in the task's pod. You can then connect to it using your local VSCode.

#### Jupyter Notebook Integration

The `@jupyter` decorator enables launching a Jupyter Notebook server within a Flyte task's container, providing an interactive environment for data analysis and experimentation.

```python
from flytekit import task
from flytekitplugins.flyteinteractive import jupyter

@jupyter(max_idle_seconds=3600, port=8888, notebook_dir="/root/notebooks")
@task
def interactive_data_task(data_path: str) -> str:
    # Your task logic here
    # Data can be explored interactively in the Jupyter Notebook
    return "Analysis complete"
```

**Key features:**

*   **Jupyter Server:** Starts a Jupyter Notebook server within the task container.
*   **Configurable:** Set `max_idle_seconds` for idle shutdown, `port` for access, and `notebook_dir` for the working directory.
*   **Pre/Post Execution Hooks:** Similar to `@vscode`, `pre_execute` and `post_execute` allow custom logic.
*   **`run_task_first`:** If `True`, the task function runs first, and Jupyter launches only on failure.

**Usage:**
Similar to `@vscode`, when a task decorated with `@jupyter` runs remotely, a link to the Jupyter Notebook server will be available in the Flyte UI or CLI output.

**Consideration:** Both `@vscode` and `@jupyter` decorators are designed for remote execution. They do not activate when `pyflyte run` is used for local execution.

### Enhancing Visibility with Flyte Decks

Flyte Decks provide a powerful mechanism to embed rich, customizable HTML reports directly into task execution details within the Flyte UI. This enhances visibility into task inputs, outputs, intermediate data, and execution timelines.

The `Deck` class allows users to create custom HTML content associated with a task.

```python
from flytekit.deck import Deck, DeckField
from flytekit.deck.renderer import MarkdownRenderer, TopFrameRenderer
import pandas as pd
from flytekit import task, current_context

@task
def process_data(df: pd.DataFrame) -> pd.DataFrame:
    # Create a custom deck
    my_deck = Deck("My Custom Report")
    my_deck.append(MarkdownRenderer().to_html("## Data Processing Summary"))
    my_deck.append(TopFrameRenderer(max_rows=5).to_html(df.head()))

    # Add to the default deck (e.g., for source code, dependencies)
    current_context().default_deck.append(MarkdownRenderer().to_html("### Task Execution Details"))

    # Publish all decks
    Deck.publish()
    return df * 2
```

**Key features:**

*   **Customizable Reports:** Append HTML strings to a `Deck` object.
*   **Multiple Decks:** Tasks can have multiple named decks.
*   **Default Decks:** Flyte automatically provides `input`, `output`, and `default` decks. The `default_deck` is useful for general-purpose content.
*   **Built-in Renderers:**
    *   `MarkdownRenderer`: Converts Markdown text to HTML.
    *   `TopFrameRenderer`: Renders Pandas DataFrames as HTML tables, with options for `max_rows` and `max_cols`.
    *   `ArrowRenderer`: Renders PyArrow Tables.
    *   `SourceCodeRenderer`: Highlights Python source code in HTML.
    *   `PythonDependencyRenderer`: Generates an HTML table of installed Python packages and their versions, including a `requirements.txt` export option.
*   **`TimeLineDeck`:** A specialized deck for visualizing the execution time of different parts of a task.
*   **`DeckField`:** An enum to specify standard fields that can be rendered in a deck (e.g., `INPUT`, `OUTPUT`, `SOURCE_CODE`, `TIMELINE`, `DEPENDENCIES`).
*   **`Deck.publish()`:** Call this method within a task to finalize and publish all associated decks to the Flyte UI.

**Consideration:** Decks are primarily for enhancing visibility in the Flyte UI and are not directly part of the local CLI output. They are generated when tasks execute, typically remotely.

### Error Handling and Verbosity

The CLI incorporates robust error handling. The `ErrorHandlingCommand` class wraps Click commands to catch exceptions during execution and present them in a user-friendly format. The verbosity of error messages can be controlled via CLI flags, allowing developers to get detailed stack traces for debugging or concise messages for general use.
<!--
key: summary_cli_&_local_development_12f89ad1-9324-4eb2-b540-7b6d5dc1d10f
type: summary_end

-->
<!--
code_unit: flytekit.clis.sdk_in_container.run
code_unit_type: class
help_text: ''
key: example_dc4ed1b6-3a85-41d2-86c2-44f4ddd63617
type: example

-->
<!--
code_unit: flytekit.interactive.vscode_lib.decorator
code_unit_type: class
help_text: ''
key: example_177394c9-f46b-4572-bb1d-32eefecd366e
type: example

-->
<!--
code_unit: flytekit.deck.deck
code_unit_type: class
help_text: ''
key: example_6508e92b-bee8-44ac-97aa-050b0f8eb544
type: example

-->