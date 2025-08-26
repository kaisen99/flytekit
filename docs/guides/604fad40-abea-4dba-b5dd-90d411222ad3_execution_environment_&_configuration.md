
<!--
help_text: ''
key: summary_execution_environment_&_configuration_0f32cb5c-b4bd-49f7-bc1a-40613e92dc1f
modules:
- flytekit.configuration.configuration
- flytekit.configuration.default_images
- flytekit.configuration.feature_flags
- flytekit.configuration.file
- flytekit.configuration.internal
- flytekit.configuration.plugin
- flytekit.core.context_manager
- flytekit.core.resources
- flytekit.core.pod_template
- flytekit.core.notification
- flytekit.core.options
- flytekit.models.security
- flytekit.clients.auth.auth_client
- flytekit.clients.auth.authenticator
- flytekit.clients.auth.exceptions
- flytekit.clients.auth.keyring
- flytekit.clients.auth.token_client
questions_to_answer: []
type: summary

-->
# Execution Environment & Configuration

Flytekit provides a robust system for defining and managing the execution environment of your tasks and workflows, alongside comprehensive configuration options. This enables consistent behavior across local development and remote production deployments, while offering granular control over resource allocation, security, and external integrations.

## Execution Environment

The execution environment defines the runtime context available to your Flyte tasks and workflows. Flytekit distinguishes between an internal, framework-level context and a user-facing context.

### Runtime Context for Tasks

The primary user-facing context within a task is provided by the `ExecutionParameters` object, accessible via `flytekit.current_context()`. This object offers essential utilities and metadata for your task's execution:

*   **Stats:** A `taggable.TaggableStats` object for emitting metrics.
*   **Logging:** A `logging.Logger` instance for structured logging.
*   **Working Directory:** A temporary directory (`working_directory`) for storing intermediate files. This directory is automatically managed and cleaned up in most execution modes.
*   **Execution Identifiers:**
    *   `execution_date`: A `datetime` object representing the workflow's start time, consistent across all tasks in an execution.
    *   `execution_id`: A `WorkflowExecutionIdentifier` for the current workflow execution, useful for linking output data back to its origin.
    *   `task_id`: An `Identifier` for the currently executing task.
*   **Secrets:** A `SecretsManager` instance for securely accessing sensitive information.
*   **Checkpoints:** A `Checkpoint` object for enabling fault-tolerant task execution.
*   **Decks:** A list of `Deck` objects for generating rich HTML reports at the end of a task.

**Example: Accessing Runtime Parameters**

```python
import flytekit
from flytekit import task

@task
def my_data_processing_task():
    ctx = flytekit.current_context()

    # Access stats client
    ctx.stats.inc("my_task.executions")

    # Log messages
    ctx.logging.info(f"Starting task {ctx.task_id.name} for execution {ctx.execution_id.name}")

    # Use the working directory for temporary files
    temp_file_path = f"{ctx.working_directory}/temp_data.csv"
    with open(temp_file_path, "w") as f:
        f.write("data,more_data\n1,2\n")

    # Access secrets
    my_api_key = ctx.secrets.my_group.api_key
    ctx.logging.info(f"Using API key from secret group 'my_group'")

    # Add content to the default deck
    ctx.default_deck.append("<h3>Task Summary</h3><p>Data processed successfully.</p>")
```

### Internal Context Management

Flytekit manages its internal state using `FlyteContext` and `FlyteContextManager`. While `FlyteContext` holds the core state (file access, compilation state, execution state, client, serialization settings), `FlyteContextManager` provides a singleton stack to manage these contexts. Developers typically interact with this via context managers (`with flytekit.new_context() as ctx:`), which ensure proper setup and teardown of the environment.

*   **`CompilationState`**: Active during the compilation phase of workflows and tasks, tracking nodes and their relationships.
*   **`ExecutionState`**: Active during the execution phase (local or remote), defining the execution mode (e.g., `TASK_EXECUTION`, `LOCAL_WORKFLOW_EXECUTION`) and managing working directories.

### Secrets Management

The `SecretsManager` provides a unified interface for retrieving secrets within your tasks. It supports a flexible resolution order:

1.  **Environment Variables:** Looks for an environment variable constructed from a configurable prefix (`FLYTE_SECRETS_ENV_PREFIX`), the secret group, and the secret key (e.g., `FLYTE_SECRETS_MYGROUP_API_KEY`).
2.  **Files:** If not found in environment variables, it attempts to read from a file path derived from a configurable default directory (`FLYTE_SECRETS_DEFAULT_DIR`), the secret group, and the secret key (e.g., `/etc/secrets/my_group/api_key`).

Secrets can be accessed using attribute-style lookup (e.g., `ctx.secrets.my_group.api_key`) or explicitly via `get(group, key)`.

**Example: Secrets Resolution**

```python
import os
from flytekit import task, current_context

@task
def access_secret_task():
    # Simulate setting an environment variable for local testing
    os.environ["FLYTE_SECRETS_MYGROUP_DB_PASSWORD"] = "my_env_password"

    ctx = current_context()

    # Access using attribute style
    db_password = ctx.secrets.my_group.db_password
    print(f"DB Password (attribute): {db_password}")

    # Access using get method
    api_key = ctx.secrets.get(group="my_group", key="api_key")
    print(f"API Key (get method): {api_key}")

    # Clean up environment variable (important for local testing)
    del os.environ["FLYTE_SECRETS_MYGROUP_DB_PASSWORD"]

# When running locally, ensure FLYTE_SECRETS_DEFAULT_DIR is configured
# or secrets are set as environment variables.
```

### Output Metadata

The `OutputMetadataTracker` allows tasks to attach arbitrary metadata to their output literals. This can be useful for tracking lineage, quality metrics, or other relevant information alongside the data itself.

## Configuration Management

Flytekit's configuration system is designed for flexibility, allowing settings to be defined via environment variables, INI-style configuration files, or YAML files. The system prioritizes environment variables, followed by INI, then YAML.

### Configuration Sources

*   **`ConfigEntry`**: Represents a single configuration entry, capable of reading from environment variables, legacy INI files (`LegacyConfigEntry`), or YAML files (`YamlConfigEntry`). It also supports type transformation.
*   **`ConfigFile`**: Loads configuration from a specified file path, automatically detecting whether it's an INI or YAML format.

This layered approach ensures that configurations can be easily overridden for different environments or specific runs.

### Core Configuration Categories

Flytekit organizes configuration settings into logical categories, each managed by a dedicated class within the `flytekit.configuration.internal` package.

*   **Platform (`Platform`)**: Configures the Flyte Admin endpoint and related security settings.
    *   `URL`: The Flyte Admin endpoint (e.g., `flyte.mycompany.com`).
    *   `INSECURE`: Whether to use insecure connections.
    *   `INSECURE_SKIP_VERIFY`: Whether to skip TLS certificate verification.
    *   `CONSOLE_ENDPOINT`: The URL for the Flyte Console.
    *   `CA_CERT_FILE_PATH`: Path to a custom CA certificate bundle.
    *   `HTTP_PROXY_URL`: URL for an HTTP proxy.

*   **Credentials (`Credentials`)**: Manages authentication settings for connecting to Flyte Admin.
    *   `COMMAND`: An external command to execute for obtaining an authentication token.
    *   `PROXY_COMMAND`: An external command for proxy authentication.
    *   `CLIENT_ID`: OAuth2 client ID.
    *   `CLIENT_CREDENTIALS_SECRET` / `CLIENT_CREDENTIALS_SECRET_LOCATION` / `CLIENT_CREDENTIALS_SECRET_ENV_VAR`: For basic authentication using client secrets.
    *   `SCOPES`: OAuth2 scopes for authentication flows.
    *   `AUTH_MODE`: Defines the authentication flow (e.g., `standard`/`Pkce`, `DeviceFlow`, `basic`/`client_credentials`).

*   **Local SDK (`LocalSDK`)**: Settings primarily for local development and testing.
    *   `WORKFLOW_PACKAGES`: Comma-delimited list of Python packages for entity discovery.
    *   `LOCAL_SANDBOX`: Path for local execution files and data.
    *   `LOGGING_LEVEL`: Default logging level for the Python logging library.

*   **Cloud Storage (`AWS`, `GCP`, `AZURE`)**: Configures access to cloud object storage.
    *   `AWS`: S3 endpoint, access key ID, secret access key, debug mode, retries.
    *   `GCP`: `GSUTIL_PARALLELISM` for gsutil operations.
    *   `AZURE`: Storage account name/key, tenant ID, client ID/secret.

*   **Images (`Images`)**: Allows specifying custom container images by name.
    *   You can define a mapping of friendly names to fully qualified image names (e.g., `my_image=docker.io/flyte:tag`).
    *   The `DefaultImages` class provides default Flytekit images based on Python versions (e.g., `cr.flyte.org/flyteorg/flytekit:py3.9-latest`). This can be overridden by the `FLYTE_INTERNAL_IMAGE_ENV_VAR` environment variable or via a plugin.

*   **Persistence (`Persistence`)**:
    *   `ATTACH_EXECUTION_METADATA`: Whether to attach execution metadata to outputs.

*   **Feature Flags (`FeatureFlags`)**:
    *   `FLYTE_PYTHON_PACKAGE_ROOT`: Controls how Flytekit locates the root of your Python package for module resolution.

### Customizing Execution Behavior

Beyond core configuration, Flytekit offers several mechanisms to customize the behavior of tasks and workflows at definition or launch time.

*   **Execution Options (`Options`)**: Configurable for launch plans and individual executions.
    *   `labels`, `annotations`: Custom key-value pairs for Kubernetes resources.
    *   `security_context`: Defines the identity under which the execution runs (e.g., Kubernetes service account, IAM role).
    *   `raw_output_data_config`: Specifies a custom remote prefix for offloaded data (e.g., `s3://my-bucket/my-prefix`).
    *   `max_parallelism`: Limits the maximum number of parallel task nodes in a workflow.
    *   `notifications`: A list of `Notification` objects to send alerts on specific execution phases.
    *   `disable_notifications`: Boolean to disable all notifications for an execution.
    *   `overwrite_cache`: Boolean to force cache invalidation for a specific execution.

    **Example: Defining Options for a Launch Plan**

    ```python
    from flytekit import workflow, LaunchPlan
    from flytekit.core.options import Options
    from flytekit.models.core.execution import WorkflowExecutionPhase
    from flytekit.core.notification import Email

    @workflow
    def my_workflow():
        ...

    my_launch_plan = LaunchPlan.create(
        my_workflow,
        default_inputs={},
        options=Options(
            labels={"team": "data-science"},
            raw_output_data_config="s3://my-custom-bucket/workflow-outputs",
            max_parallelism=5,
            notifications=[
                Email(
                    phases=[WorkflowExecutionPhase.SUCCEEDED, WorkflowExecutionPhase.FAILED],
                    recipients_email=["dev-alerts@example.com"]
                )
            ]
        )
    )
    ```

*   **Resource Allocation (`Resources`, `ResourceSpec`)**: Defines compute resource requests and limits for tasks.
    *   `cpu`, `mem`, `gpu`, `ephemeral_storage`: Can be specified as single values (request and limit are the same) or as a tuple/list `(request, limit)`.
    *   Supports various units (e.g., `1`, `100m` for CPU; `2048`, `2Gi` for memory).

    **Example: Task Resource Configuration**

    ```python
    from flytekit import task
    from flytekit.core.resources import Resources

    @task(requests=Resources(cpu="500m", mem="1Gi"), limits=Resources(cpu="1", mem="2Gi"))
    def high_resource_task(a: int, b: int) -> int:
        return a + b

    @task(requests=Resources(cpu=("100m", "200m"), mem=("512Mi", "1Gi"), ephemeral_storage="10Gi"))
    def another_task(x: float) -> float:
        return x * 2
    ```

*   **Pod Template (`PodTemplate`)**: For advanced Kubernetes users, this allows direct customization of the underlying Kubernetes Pod specification for a task. This provides fine-grained control over aspects not covered by standard Flytekit options.

    **Example: Custom Pod Template**

    ```python
    from flytekit import task
    from flytekit.core.pod_template import PodTemplate
    from kubernetes.client import V1PodSpec, V1Container, V1EnvVar

    @task(pod_template=PodTemplate(
        pod_spec=V1PodSpec(
            containers=[
                V1Container(
                    name="primary",
                    env=[V1EnvVar(name="MY_CUSTOM_ENV", value="some_value")]
                )
            ],
            # Add other pod-level specs like node_selector, tolerations, etc.
            node_selector={"kubernetes.io/os": "linux"}
        ),
        primary_container_name="primary"
    ))
    def custom_pod_task():
        print(f"Custom env var: {os.getenv('MY_CUSTOM_ENV')}")
    ```

*   **Notifications (`Notification`, `Email`, `PagerDuty`, `Slack`)**: Configure alerts for workflow and task execution phases. Notifications can only be triggered for terminal phases (`SUCCEEDED`, `FAILED`, `ABORTED`, `TIMED_OUT`).

    **Example: Slack Notification**

    ```python
    from flytekit import workflow, LaunchPlan
    from flytekit.core.notification import Slack
    from flytekit.models.core.execution import WorkflowExecutionPhase

    @workflow
    def my_workflow_with_notifications():
        ...

    lp_with_slack = LaunchPlan.create(
        my_workflow_with_notifications,
        default_inputs={},
        options=Options(
            notifications=[
                Slack(
                    phases=[WorkflowExecutionPhase.FAILED],
                    recipients_email=["#my-slack-channel"]
                )
            ]
        )
    )
    ```

## Authentication and Authorization

Flytekit provides various authentication mechanisms to secure communication with the Flyte Admin backend. These are managed by different `Authenticator` implementations and leverage the `flytekit.clients.auth` package.

*   **`Authenticator`**: The base class for all authentication flows.
*   **`PKCEAuthenticator`**: Implements the Proof Key for Code Exchange (PKCE) flow, typically used for interactive user authentication where a browser window is opened for login. Credentials are securely cached using `KeyringStore`.
*   **`DeviceCodeAuthenticator`**: Implements the OAuth 2.0 Device Authorization Grant flow, suitable for headless environments (e.g., CLI tools) where a browser is not available. It provides a URL and user code for manual authentication.
*   **`ClientCredentialsAuthenticator`**: Uses a client ID and client secret for machine-to-machine authentication, often used for service accounts or automated processes.
*   **`CommandAuthenticator`**: Allows an external command to be executed to retrieve an authentication token. This is useful for integrating with custom or enterprise-specific identity providers.

**Credential Storage (`KeyringStore`)**

The `KeyringStore` provides a secure way to cache authentication `Credentials` (access tokens, refresh tokens, ID tokens) using the system's keyring service. This avoids repeated authentication prompts for interactive flows.

**Security Context (`SecurityContext`)**

The `SecurityContext` model allows you to define the identity under which a task or workflow execution runs and to specify secrets or tokens to be injected into the execution environment.

*   **`run_as` (`Identity`)**: Specifies the execution identity, which can be an IAM role (AWS), Kubernetes service account, or an OAuth2 client. This is crucial for defining permissions for your code running on the Flyte platform.
*   **`secrets` (`Secret`)**: A list of `Secret` objects to request specific secrets from the platform.
    *   `group`: The name of the secret group (e.g., Kubernetes secret name).
    *   `key`: An optional key within the secret group.
    *   `mount_requirement`: A hint for how the secret should be injected (`ANY`, `ENV_VAR`, `FILE`). `ENV_VAR` is suitable for symmetric keys, while `FILE` is for secrets that need to be accessed as files.
    *   `env_var`: An optional custom environment variable name for the secret.
    *   **Important**: Depending on the `FlytekitPlugin` configuration, the `group` field might be required during registration.
*   **`tokens` (`OAuth2TokenRequest`)**: A list of `OAuth2TokenRequest` objects for requesting OAuth2 tokens.

**Example: Defining Security Context with Secrets**

```python
from flytekit import task
from flytekit.models.security import SecurityContext, Secret, Identity, Secret.MountType

@task(
    security_context=SecurityContext(
        run_as=Identity(k8s_service_account="my-service-account"),
        secrets=[
            Secret(group="my-db-secrets", key="db_password", mount_requirement=Secret.MountType.ENV_VAR, env_var="DB_PASS"),
            Secret(group="my-cert-secrets", key="tls_cert", mount_requirement=Secret.MountType.FILE)
        ]
    )
)
def secure_task():
    # DB_PASS will be available as an environment variable
    db_password = os.getenv("DB_PASS")
    print(f"DB Password from env: {db_password}")

    # tls_cert will be available as a file at a path determined by Flytekit
    # The exact path can be retrieved via current_context().secrets.get_secrets_file()
    tls_cert_path = flytekit.current_context().secrets.get_secrets_file(group="my-cert-secrets", key="tls_cert")
    with open(tls_cert_path, "r") as f:
        tls_cert_content = f.read()
    print(f"TLS Cert content from file: {tls_cert_content[:20]}...")
```

## Extensibility with Plugins

The `FlytekitPlugin` and `FlytekitPluginProtocol` provide an extension point for customizing Flytekit's behavior. This allows users or platform administrators to override default settings and integrate with custom systems. Key capabilities include:

*   **`get_default_image()`**: Override the default container image used for tasks.
*   **`configure_pyflyte_cli()`**: Extend or modify the `pyflyte` command-line interface.
*   **`secret_requires_group()`**: Define whether secrets require a group name during registration.
*   **`get_default_cache_policies()`**: Set default caching policies for tasks.

## Best Practices and Considerations

*   **Environment Variables for Dynamic Configuration**: For values that change frequently or are sensitive (e.g., API keys, database connection strings), prefer using environment variables. Flytekit's configuration system prioritizes them, making it easy to manage different environments without code changes.
*   **Resource Management**: Always specify `requests` for CPU and memory to ensure your tasks get the minimum resources they need. Use `limits` to prevent tasks from consuming excessive resources and impacting other workloads.
*   **Local vs. Remote Execution**: Be mindful of the `ExecutionState.Mode`. Behavior can differ between local execution (e.g., `LOCAL_TASK_EXECUTION`) where tasks run as Python functions, and remote execution (`TASK_EXECUTION`) where they run within containers on the Flyte platform. Ensure your code handles both scenarios gracefully, especially regarding file paths and external dependencies.
*   **Secrets Security**: When defining `Secret` objects, carefully choose the `mount_requirement`. `ENV_VAR` is generally simpler but `FILE` is more secure for sensitive data like private keys, as it avoids exposing the secret directly in environment variables.
*   **Container Images**: Leverage the `DefaultImages` system for standard Python environments. For custom dependencies, build your own container images and register them using the `Images` configuration or specify them directly in task definitions.
*   **Debugging**: Utilize the `ExecutionParameters.logging` and `ExecutionParameters.stats` objects for effective debugging and monitoring of your tasks. The `working_directory` is also invaluable for inspecting intermediate outputs during local runs.
<!--
key: summary_execution_environment_&_configuration_0f32cb5c-b4bd-49f7-bc1a-40613e92dc1f
type: summary_end

-->
<!--
code_unit: flytekit.configuration.configuration
code_unit_type: class
help_text: ''
key: example_a9ce5714-6d9b-4bd4-88d5-cc6c5d067bb5
type: example

-->
<!--
code_unit: flytekit.core.resources
code_unit_type: class
help_text: ''
key: example_9ba10614-6a41-4c16-b735-ac745b858bae
type: example

-->
<!--
code_unit: flytekit.models.security
code_unit_type: class
help_text: ''
key: example_4217beed-2754-4e17-bc77-126227fa0d67
type: example

-->