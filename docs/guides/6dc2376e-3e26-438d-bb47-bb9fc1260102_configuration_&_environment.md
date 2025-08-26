
<!--
help_text: ''
key: summary_configuration_&_environment_241a4e61-617e-4512-a053-c5745927e428
modules:
- flytekit.configuration.configuration
- flytekit.configuration.default_images
- flytekit.configuration.feature_flags
- flytekit.configuration.file
- flytekit.configuration.internal
- flytekit.configuration.plugin
- flytekit.core.context_manager
- flytekit.core.resources
- flytekit.core.options
- flytekit.core.notification
- flytekit.core.schedule
- flytekit.core.pod_template
- flytekit.models.security
- flytekit.models.task
- flytekit.models.launch_plan
- flytekit.models.schedule
- flytekit.models.common
questions_to_answer: []
type: summary

-->
Flytekit's configuration and environment management provides a flexible and robust system for defining how your code executes, from local development to production deployments. This system encompasses how settings are loaded, how runtime context is provided to tasks, and how execution-specific options are applied.

### Configuration Loading and Prioritization

Flytekit loads configuration from multiple sources, prioritizing them to allow for flexible overrides. The primary mechanism for accessing configuration values is through `ConfigEntry` objects, which abstract the lookup logic.

Configuration values are resolved in the following order:
1.  **Environment Variables**: Values set as environment variables take the highest precedence.
2.  **Legacy INI Files**: Configuration files with `.ini` or `.config` extensions are read next.
3.  **YAML Files**: YAML configuration files (`.yaml` or `.yml`) are processed last.

The `ConfigFile` utility handles parsing these file types. When Flytekit initializes, it automatically attempts to load configuration from standard locations or a path specified by the user.

**Example: Accessing a Configuration Value**

To access a configuration value, you typically interact with predefined `ConfigEntry` objects. For instance, to retrieve the Flyte platform URL:

```python
from flytekit.configuration import Config, Platform

# Automatically loads configuration from environment variables,
# then config files (e.g., ~/.flyte/config.yaml or .flyte/config)
cfg = Config.auto()

# Access the platform URL
platform_url = cfg.platform.url
print(f"Flyte Platform URL: {platform_url}")

# You can also override via environment variables:
# export FLYTE_PLATFORM_URL="http://my-custom-flyte-endpoint.com"
```

#### Core Configuration Categories

Flytekit organizes configuration into logical categories, each managed by a dedicated class within the `internal` configuration namespace.

*   **Platform Connectivity**: The `Platform` class manages settings related to connecting to the Flyte backend. This includes the `URL` of the Flyte Admin service, `INSECURE` flags for TLS verification, `CA_CERT_FILE_PATH` for custom CA certificates, and `HTTP_PROXY_URL` for proxy settings.
*   **Cloud Storage**: Classes like `AWS`, `GCP`, and `AZURE` manage cloud-specific storage configurations. This includes credentials (`S3_ACCESS_KEY_ID`, `S3_SECRET_ACCESS_KEY`, `STORAGE_ACCOUNT_NAME`, `TENANT_ID`, `CLIENT_ID`, `CLIENT_SECRET`), endpoints (`S3_ENDPOINT`), and performance settings (`GSUTIL_PARALLELISM`).
*   **Secrets Management**: The `Secrets` class defines prefixes and default directories for secret resolution. The `SecretsManager` utility provides a runtime interface to retrieve secrets, prioritizing environment variables over files.
    *   Secrets can be accessed using `secrets.group.key` attribute-style lookup or `secrets.get("group", "key")`.
    *   The `Secret` model allows specifying `group`, `key`, `group_version`, `mount_requirement` (ANY, ENV_VAR, FILE), and an optional `env_var` name.
    *   **Example: Accessing a Secret**
        ```python
        from flytekit import task, Secret
        from flytekit.core.context_manager import FlyteContextManager

        @task(secret_requests=[Secret(group="my-secrets", key="api_key")])
        def my_task_with_secret():
            ctx = FlyteContextManager.current_context()
            api_key = ctx.user_space_params.secrets.my_secrets.api_key
            print(f"API Key: {api_key}")

            # Alternatively
            api_key_alt = ctx.user_space_params.secrets.get("my-secrets", "api_key")
            print(f"API Key (alt): {api_key_alt}")
        ```
*   **Default Images**: The `DefaultImages` utility determines the default Docker image used for tasks. It considers the current Python version (`PythonVersion` enum) and the Flytekit version. The `FLYTE_INTERNAL_IMAGE_ENV_VAR` environment variable can override this default.
    *   **Example: Retrieving the Default Image**
        ```python
        from flytekit.configuration.default_images import DefaultImages

        # Get the default image for the current Python environment
        image = DefaultImages.default_image()
        print(f"Default Flytekit image: {image}")

        # Override via environment variable:
        # export FLYTE_INTERNAL_IMAGE_ENV_VAR="my-custom-registry/my-image:my-tag"
        ```
*   **Custom Images**: The `Images` class allows defining custom image aliases within the configuration, enabling users to refer to images by friendly names.
*   **Local Development**: The `LocalSDK` class configures aspects of local execution, such as `WORKFLOW_PACKAGES` for entity discovery, `LOCAL_SANDBOX` for temporary file storage, and `LOGGING_LEVEL`.
*   **Internal Metrics**: The `StatsD` class provides configuration for StatsD metrics reporting, including `HOST`, `PORT`, and `DISABLED` flags.
*   **Persistence**: The `Persistence` class includes settings like `ATTACH_EXECUTION_METADATA`.

### Runtime Environment and Context

Flytekit manages the runtime environment through a context stack, providing essential information and utilities to your code during execution.

#### `FlyteContextManager` and `FlyteContext`

The `FlyteContextManager` is a singleton responsible for managing the `FlyteContext` stack. `FlyteContext` is an internal object that holds the global state for compilation or execution. It is not thread-safe and is designed for single-threaded operations.

While `FlyteContext` contains internal state like `CompilationState` (for tracking nodes during workflow compilation) and `ExecutionState` (for managing runtime parameters), users typically interact with a simplified, user-facing context.

#### `ExecutionParameters` (User-Facing Context)

The `ExecutionParameters` object is the primary user-facing context, accessible within any `@task` method via `flytekit.current_context()`. It provides runtime information and utilities relevant to the current task execution.

**Key Properties of `ExecutionParameters`:**

*   **`stats`**: A handle to a tagged StatsD object for emitting metrics.
*   **`logging`**: A Python `Logger` instance for task-specific logging.
*   **`execution_id`**: A `WorkflowExecutionIdentifier` uniquely identifying the current workflow execution. This is consistent across all tasks within a workflow.
*   **`task_id`**: An `Identifier` for the current task execution.
*   **`working_directory`**: A temporary directory for the task to write arbitrary files. This directory is managed by Flytekit.
*   **`raw_output_prefix`**: The remote prefix (e.g., S3, GCS path) where offloaded data (like large inputs/outputs) is stored.
*   **`secrets`**: An instance of `SecretsManager` for retrieving secrets.
*   **`checkpoint`**: A handle to the configured checkpointing system for resuming task execution.
*   **`decks`**: A list of `Deck` objects for generating rich HTML outputs for tasks. The `default_deck` property provides a convenient default.
*   **`enable_deck`**: A boolean indicating whether deck generation is enabled.
*   **`execution_date`**: A `datetime` object representing the start time of the workflow execution.

**Example: Using `ExecutionParameters` in a Task**

```python
from flytekit import task, current_context
import os

@task
def my_data_processing_task():
    ctx = current_context()

    # Log a message
    ctx.logging.info("Starting data processing task.")

    # Emit a custom metric
    ctx.stats.inc("data_processed_count")

    # Get the unique execution ID
    exec_id = ctx.execution_id.name
    ctx.logging.info(f"Current execution ID: {exec_id}")

    # Write a temporary file to the working directory
    temp_file_path = os.path.join(ctx.working_directory, "temp_output.txt")
    with open(temp_file_path, "w") as f:
        f.write("Hello from Flytekit task!")
    ctx.logging.info(f"Wrote temporary file to: {temp_file_path}")

    # Access a secret (if configured)
    # api_key = ctx.secrets.my_service.api_key
```

#### Execution Modes

The `ExecutionState.Mode` enum defines various execution contexts, influencing how tasks and workflows behave:

*   **`TASK_EXECUTION`**: Mimics the actual runtime environment on the Flyte platform.
*   **`LOCAL_WORKFLOW_EXECUTION`**: Used when running a workflow purely locally, where task outputs are wrapped in `NodeOutput` objects.
*   **`LOCAL_TASK_EXECUTION`**: For purely local task execution, without a container or the Flyte backend.
*   **`DYNAMIC_TASK_EXECUTION`**: Indicates execution within a dynamic task, where a runtime specification is extracted.
*   **`EAGER_EXECUTION`**, **`EAGER_LOCAL_EXECUTION`**, **`LOCAL_DYNAMIC_TASK_EXECUTION`**: Specific modes related to eager execution and local dynamic tasks.

These modes primarily affect Flytekit's internal behavior, such as how data is handled or how graph structures are built.

### Resource Allocation

Flytekit allows you to specify compute resources (CPU, memory, GPU, ephemeral storage) for tasks using the `Resources` class. This translates to Kubernetes resource requests and limits.

*   **`cpu`**: CPU units (e.g., "1", "500m", 0.5).
*   **`mem`**: Memory (e.g., "2048", "2Gi", "500Mi").
*   **`gpu`**: GPU units (e.g., "1").
*   **`ephemeral_storage`**: Local ephemeral storage (e.g., "1Gi").

When providing values:
*   A single value sets both the request and the limit.
*   A tuple or list `(request, limit)` sets distinct request and limit values.

**Example: Specifying Task Resources**

```python
from flytekit import task
from flytekit.core.resources import Resources

@task(resources=Resources(cpu="1", mem=("1Gi", "2Gi"), gpu="1"))
def my_resource_intensive_task():
    # This task will request 1 CPU, 1Gi memory, 1 GPU
    # and be limited to 2Gi memory.
    print("Running resource-intensive task.")
```

### Execution Options and Overrides

The `Options` class provides a comprehensive way to configure various aspects of a workflow or task execution, especially when registering Launch Plans or initiating executions. These options can be overridden at runtime.

*   **`labels`** and **`annotations`**: Custom Kubernetes labels and annotations applied to the execution resource. These are key-value pairs for metadata.
*   **`security_context`**: Configures the security identity for the execution. This uses the `SecurityContext` model, which can specify:
    *   `run_as`: An `Identity` (IAM role, Kubernetes service account, or OAuth2 client) for the execution.
    *   `secrets`: A list of `Secret` objects to be injected into the task environment.
    *   `tokens`: A list of `OAuth2TokenRequest` objects for obtaining access tokens.
*   **`raw_output_data_config`**: A `RawOutputDataConfig` object specifying the remote prefix (e.g., `s3://my-bucket/output/`) where large offloaded data (Blobs, Schemas) will be stored. If not specified, the platform's default is used.
*   **`max_parallelism`**: Controls the maximum number of task nodes that can run concurrently within the entire workflow.
*   **`notifications`**: A list of `Notification` objects to send alerts based on workflow execution status transitions. Flytekit supports `Email`, `Slack`, and `PagerDuty` notifications, which can be configured for specific `WorkflowExecutionPhase` states (e.g., `SUCCEEDED`, `FAILED`, `ABORTED`, `TIMED_OUT`).
*   **`overwrite_cache`**: A boolean flag to force re-execution of a task even if cached results are available.

**Example: Applying Options to a Launch Plan**

```python
from flytekit import workflow, LaunchPlan
from flytekit.core.options import Options
from flytekit.models.core.execution import WorkflowExecutionPhase
from flytekit.core.notification import Email
from flytekit.models.common import AuthRole
from flytekit.models.security import Secret, MountType

@workflow
def my_workflow(x: int) -> int:
    return x + 1

# Define execution options
my_options = Options(
    labels={"team": "data-science"},
    annotations={"owner": "john.doe"},
    raw_output_data_config="s3://my-custom-output-bucket/workflow-data",
    max_parallelism=5,
    notifications=[
        Email(phases=[WorkflowExecutionPhase.SUCCEEDED, WorkflowExecutionPhase.FAILED], recipients_email=["alerts@example.com"])
    ],
    security_context=AuthRole(kubernetes_service_account="my-service-account"),
    overwrite_cache=True,
)

# Create a Launch Plan with these options
my_launch_plan = LaunchPlan.create("my_workflow_lp", my_workflow, default_inputs={"x": 10}, options=my_options)
```

### Scheduling

Flytekit supports scheduling Launch Plans to run automatically at specified intervals or times.

*   **`OnSchedule`**: The base trigger for scheduled Launch Plans.
*   **`FixedRate`**: Schedules a Launch Plan to run at a fixed interval (e.g., every 10 minutes, every day). The `duration` parameter takes a `datetime.timedelta`.
*   **`CronSchedule`**: Schedules a Launch Plan using a cron expression (e.g., `*/1 * * * *` for every minute). It also supports cron aliases like `@hourly`, `@daily`. An optional `offset` can be specified using ISO 8601 duration format.
*   **`kickoff_time_input_arg`**: A convenient argument to pass the scheduled kickoff time as an input to the workflow.

**Example: Scheduling a Workflow**

```python
from flytekit import workflow, LaunchPlan
from flytekit.core.schedule import FixedRate, CronSchedule
from datetime import timedelta, datetime

@workflow
def daily_report_workflow(report_date: datetime):
    print(f"Generating report for: {report_date}")

# Schedule to run every day
daily_schedule = FixedRate(duration=timedelta(days=1), kickoff_time_input_arg="report_date")
daily_lp = LaunchPlan.create("daily_report_lp", daily_report_workflow, schedule=daily_schedule)

# Schedule using a cron expression (e.g., every Monday at 9 AM UTC)
cron_schedule = CronSchedule(schedule="0 9 * * MON", kickoff_time_input_arg="report_date")
cron_lp = LaunchPlan.create("weekly_report_lp", daily_report_workflow, schedule=cron_schedule)
```

### Custom Pod Templates

For advanced Kubernetes users, Flytekit allows specifying a custom `PodTemplate` for tasks. This enables fine-grained control over the underlying Kubernetes Pod specification, including labels, annotations, and container definitions.

The `PodTemplate` class allows you to define a `V1PodSpec` (from `kubernetes.client`) and specify the `primary_container_name`. This is useful for scenarios requiring specific node selectors, tolerations, sidecar containers, or other Kubernetes-specific configurations.

**Example: Using a Custom Pod Template**

```python
from flytekit import task
from flytekit.core.pod_template import PodTemplate
from kubernetes.client import V1PodSpec, V1Container

# Define a custom pod template
custom_pod_template = PodTemplate(
    pod_spec=V1PodSpec(
        node_selector={"node-type": "high-mem"},
        containers=[
            V1Container(name="primary", image="my-custom-image:latest", command=["python"], args=["my_script.py"]),
            V1Container(name="sidecar", image="my-sidecar-image:1.0", command=["sidecar-app"]),
        ],
    ),
    primary_container_name="primary",
    labels={"environment": "production"},
    annotations={"cost-center": "engineering"},
)

@task(pod_template=custom_pod_template)
def my_custom_pod_task():
    print("Running inside a custom Kubernetes pod.")
```

### Plugin System

Flytekit's plugin system, defined by the `FlytekitPluginProtocol` and implemented by `FlytekitPlugin`, allows for extending or overriding core Flytekit behaviors. This includes:

*   **`get_remote`**: Customizing how `FlyteRemote` objects are created for CLI sessions.
*   **`configure_pyflyte_cli`**: Extending the `pyflyte` command-line interface.
*   **`secret_requires_group`**: Defining whether secrets require a group during registration.
*   **`get_default_image`**: Providing a custom default Docker image, overriding the `DefaultImages` utility.
*   **`get_auth_success_html`**: Customizing the HTML page displayed after successful authentication.
*   **`get_default_cache_policies`**: Defining default caching policies for tasks.

This extensibility allows for deep integration with specific platform requirements or custom development workflows.
<!--
key: summary_configuration_&_environment_241a4e61-617e-4512-a053-c5745927e428
type: summary_end

-->
<!--
code_unit: flytekit.configuration.configuration.Config
code_unit_type: class
help_text: ''
key: example_a78cb314-328b-4964-b512-e3fe8a533e14
type: example

-->
<!--
code_unit: flytekit.core.context_manager.ExecutionParameters
code_unit_type: class
help_text: ''
key: example_a586628a-5d76-4f3e-a465-862833f4f7f8
type: example

-->