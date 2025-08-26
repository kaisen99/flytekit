
<!--
help_text: ''
key: summary_local_execution_and_development_settings_45de1ed0-0698-4c06-b540-64725b1e2a54
modules:
- flytekit.configuration.internal
- flytekit.core.context_manager
questions_to_answer: []
type: summary

-->
## Local Execution and Development Settings

Flytekit provides robust capabilities for developing and testing workflows and tasks locally before deploying them to a Flyte cluster. This local environment closely mimics the production runtime, allowing for rapid iteration and debugging. Understanding how to configure and interact with these local settings is crucial for an efficient development workflow.

### Core Local Execution Configuration

The primary settings for local execution are managed through the `LocalSDK` configuration. These settings control how Flytekit discovers entities, manages local files, and handles logging during local runs.

*   **Workflow Package Discovery (`WORKFLOW_PACKAGES`)**:
    This setting specifies a comma-delimited list of Python packages where Flytekit should look for workflow and task definitions. When running local commands (e.g., `pyflyte run`), Flytekit uses this list to discover and load your code.
    ```python
    # Example configuration in a Flyte config file (e.g., config.yaml)
    [sdk]
    workflow_packages = my_project.workflows,my_project.tasks
    ```

*   **Local Sandbox Directory (`LOCAL_SANDBOX`)**:
    Flytekit uses a designated local directory to store temporary files, outputs, and other artifacts generated during local executions and testing. This `LOCAL_SANDBOX` path is where the SDK places these files. It is important to note that Flytekit does not automatically clean up data in this directory, allowing for inspection and debugging after a run.
    The `FlyteContextManager` initializes a `user_space` directory within this sandbox for user-specific files.

*   **Logging Level (`LOGGING_LEVEL`)**:
    This setting controls the default logging level for the Python `logging` library within your local Flytekit environment. It is applied before user code executes, allowing you to control the verbosity of logs during development. This is a runtime setting, affecting the output you see in your console.

### Managing Local Data and Files

During local execution, tasks and workflows often need to read from or write to a local file system. Flytekit provides a consistent way to manage these interactions.

The `ExecutionParameters` object, accessible via `flytekit.current_context()`, exposes a `working_directory` property. This directory is a temporary location where tasks can store intermediate data. For local runs, this typically points to a subdirectory within the `LOCAL_SANDBOX`.

```python
import flytekit
import os

@flytekit.task
def my_local_task():
    ctx = flytekit.current_context()
    local_path = os.path.join(ctx.working_directory, "my_data.txt")
    with open(local_path, "w") as f:
        f.write("Hello from local execution!")
    print(f"Data written to: {local_path}")
```

The `FlyteContext` also manages a `FileAccessProvider` which determines how files are accessed. In local execution, this defaults to a local file system provider, ensuring that operations like `ctx.file_access.get_random_local_directory()` provide paths within the `LOCAL_SANDBOX`.

### Secrets Management in Local Development

Tasks often require access to sensitive information (secrets). Flytekit's `SecretsManager` provides a mechanism to resolve these secrets during local execution, mimicking how they would be injected in a production environment.

The `Secrets` configuration defines the lookup behavior:

*   **Environment Variable Prefix (`ENV_PREFIX`)**:
    Secrets are first looked up as environment variables. The environment variable name is constructed using this prefix, followed by the group and key (e.g., `FLYTE_SECRETS_MYGROUP_MYKEY`).
*   **Default Secrets Directory (`DEFAULT_DIR`)**:
    If a secret is not found in environment variables, Flytekit looks for it as a file within this directory. The file path is constructed as `<DEFAULT_DIR>/<group>/<FILE_PREFIX><key>`.
*   **File Prefix (`FILE_PREFIX`)**:
    This prefix is added to the secret key when looking for secrets as files.

Developers can access secrets within their tasks using `flytekit.current_context().secrets`.

```python
import flytekit

@flytekit.task(secret_requests=[flytekit.Secret(group="my_group", key="my_secret_key")])
def my_secret_task():
    ctx = flytekit.current_context()
    secret_value = ctx.secrets.my_group.my_secret_key
    print(f"My secret: {secret_value}")

# To run locally, you can set the environment variable:
# export FLYTE_SECRETS_MY_GROUP_MY_SECRET_KEY="my_value"
# Or create a file:
# mkdir -p /tmp/flyte/secrets/my_group
# echo "my_value" > /tmp/flyte/secrets/my_group/my_secret_key
# And configure DEFAULT_DIR in your Flyte config:
# [secrets]
# default_dir = /tmp/flyte/secrets
```

During local execution, the `SecretsManager` also checks for environment variables without the `ENV_PREFIX`, providing flexibility for quick local testing.

### Local Caching

The `Local` configuration section allows you to control caching behavior for local task executions.

*   **Cache Enabled (`CACHE_ENABLED`)**:
    When set to `True`, Flytekit attempts to cache the results of local task executions. If a task with the same inputs has been executed before, Flytekit can retrieve the cached output instead of re-running the task.
*   **Cache Overwrite (`CACHE_OVERWRITE`)**:
    If caching is enabled, setting `CACHE_OVERWRITE` to `True` forces Flytekit to re-execute the task and overwrite any existing cached results, even if inputs are identical. This is useful for ensuring fresh results during development.

These settings can significantly speed up local development by avoiding redundant computations.

### Connecting to Remote Flyte (Development Settings)

While local execution runs entirely on your machine, development often involves interacting with a remote Flyte cluster for registration, inspection, or triggering remote executions. The `Platform` and `Credentials` configurations facilitate this interaction.

*   **Platform Configuration (`Platform`)**:
    This section defines how Flytekit connects to the Flyte Admin service.
    *   `URL`: The endpoint of the Flyte Admin service.
    *   `INSECURE`: Whether to use an insecure connection (e.g., HTTP instead of HTTPS).
    *   `INSECURE_SKIP_VERIFY`: Whether to skip SSL certificate verification.
    *   `CONSOLE_ENDPOINT`: The URL for the Flyte Console UI.
    *   `CA_CERT_FILE_PATH`: Path to a CA certificate file for secure connections.
    *   `HTTP_PROXY_URL`: URL for an HTTP proxy to connect to Admin.

*   **Credentials Configuration (`Credentials`)**:
    This section manages authentication details for connecting to the Flyte Admin service.
    *   `COMMAND`, `PROXY_COMMAND`: External commands to execute for obtaining authentication tokens.
    *   `CLIENT_ID`: The OAuth2 client ID for your application.
    *   `CLIENT_CREDENTIALS_SECRET`, `CLIENT_CREDENTIALS_SECRET_LOCATION`, `CLIENT_CREDENTIALS_SECRET_ENV_VAR`: Methods for providing a client secret for basic authentication. Using a file location or environment variable is generally more secure than embedding the secret directly.
    *   `SCOPES`: OAuth2 scopes required for authentication.
    *   `AUTH_MODE`: Defines the authentication flow (e.g., `standard` (PKCE), `DeviceFlow`, `basic`, `client_credentials`).

These settings are crucial when you use `pyflyte register` or `pyflyte run --remote` commands, allowing your local Flytekit installation to securely communicate with your deployed Flyte environment.

### Cloud Storage Access for Local Runs

Flyte tasks often interact with cloud storage (e.g., S3, GCS, Azure Blob Storage). For local execution to accurately simulate production behavior, Flytekit needs to be configured with credentials and endpoints for these storage services.

*   **AWS Configuration (`AWS`)**:
    *   `S3_ENDPOINT`: Custom S3 endpoint (useful for localstack or private S3).
    *   `S3_ACCESS_KEY_ID`, `S3_SECRET_ACCESS_KEY`: AWS credentials for S3 access.
    *   `ENABLE_DEBUG`, `RETRIES`, `BACKOFF_SECONDS`: Control client behavior for S3 interactions.

*   **AZURE Configuration (`AZURE`)**:
    *   `STORAGE_ACCOUNT_NAME`, `STORAGE_ACCOUNT_KEY`: Azure Storage account credentials.
    *   `TENANT_ID`, `CLIENT_ID`, `CLIENT_SECRET`: Azure Active Directory (AAD) credentials for service principal authentication.

*   **GCP Configuration (`GCP`)**:
    *   `GSUTIL_PARALLELISM`: Controls parallelism for `gsutil` operations.

By configuring these sections, your locally executed tasks can seamlessly read from and write to cloud storage, enabling comprehensive local testing of data pipelines.

### Understanding Execution Modes

The `ExecutionState.Mode` enum within Flytekit's internal context manager defines the specific environment in which a task or workflow is running. While primarily an internal concept, understanding these modes helps in debugging and predicting behavior.

*   `LOCAL_TASK_EXECUTION`: A task is running purely locally, without a container or the Flyte engine (Propeller).
*   `LOCAL_WORKFLOW_EXECUTION`: A workflow is running locally, with tasks potentially executing in `LOCAL_TASK_EXECUTION` mode. In this mode, workflow constructs like conditional branches are evaluated locally.
*   `EAGER_LOCAL_EXECUTION`: A mode for immediate local execution, often used for quick testing of individual tasks.
*   `LOCAL_DYNAMIC_TASK_EXECUTION`: A dynamic task is being executed locally.

When Flytekit runs locally, it sets the appropriate `ExecutionState.Mode`, which influences how certain operations (like handling outputs or conditional logic) are performed compared to a remote execution.

### Accessing Runtime Information (`ExecutionParameters`)

The `ExecutionParameters` object is the user-facing context provided to every `@task` method via `flytekit.current_context()`. It offers essential runtime information and utilities for local development.

*   `stats`: A handle to a statsd object for emitting metrics. While in local execution, this defaults to a mock stats object (`mock_stats.MockStats`), allowing your code to run without error even if metrics are not being collected.
*   `logging`: A handle to a Python `logging.Logger` instance, configured according to the `LOGGING_LEVEL` setting.
*   `working_directory`: A temporary local directory for the task to use for intermediate files.
*   `execution_date`: The datetime when the local execution started. This is consistent across all tasks in a local workflow run.
*   `execution_id`: A unique identifier for the local execution. For local runs, this is typically a placeholder like `local:local:local`.
*   `secrets`: An instance of `SecretsManager` for accessing configured secrets.
*   `checkpoint`: A handle to a checkpointing system. For local runs, this defaults to a `SyncCheckpoint` writing to a local directory within the task's sandbox.
*   `decks`: A list of `Deck` objects, which can be used to generate rich HTML reports for task outputs. The `enable_deck` property controls whether deck generation is active.

Developers should use `flytekit.current_context()` to access these parameters, ensuring their tasks behave consistently whether run locally or on a remote cluster.

```python
import flytekit
import logging

@flytekit.task
def my_context_aware_task():
    ctx = flytekit.current_context()
    ctx.logging.info(f"Task started at {ctx.execution_date}")
    ctx.stats.inc("my_task.runs")
    # Access other properties like ctx.working_directory, ctx.execution_id, ctx.secrets
```

### Best Practices and Considerations

*   **Consistent Configuration**: Maintain a consistent `config.yaml` (or environment variables) across your local development and CI/CD environments to minimize discrepancies.
*   **Isolate Local Runs**: Leverage the `LOCAL_SANDBOX` to keep local execution artifacts separate from your source code.
*   **Secrets Management**: Always use the `SecretsManager` for accessing sensitive data, even in local development, to ensure your code is portable and secure. Avoid hardcoding secrets.
*   **Debugging**: The `LOGGING_LEVEL` and the ability to inspect the `LOCAL_SANDBOX` are powerful tools for debugging local task and workflow failures.
*   **Cloud Storage Emulators**: For comprehensive local testing of cloud-dependent tasks, consider using local emulators (e.g., LocalStack for AWS S3, MinIO for S3-compatible storage) in conjunction with the cloud storage configurations.
*   **Performance**: Utilize local caching (`CACHE_ENABLED`) to accelerate iterative development, especially for long-running tasks.
<!--
key: summary_local_execution_and_development_settings_45de1ed0-0698-4c06-b540-64725b1e2a54
type: summary_end

-->
<!--
code_unit: flytekit.configuration.internal.LocalSDK
code_unit_type: class
help_text: ''
key: example_68733724-b61c-4835-ba42-6509ddf7a874
type: example

-->
<!--
code_unit: flytekit.core.context_manager.ExecutionParameters
code_unit_type: class
help_text: ''
key: example_bcef2330-442d-4f5d-bf20-4279dd220b12
type: example

-->
<!--
code_unit: flytekit.core.context_manager.SecretsManager
code_unit_type: class
help_text: ''
key: example_a33bc842-d232-4802-a97d-a9417b926c43
type: example

-->