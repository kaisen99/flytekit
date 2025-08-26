
<!--
help_text: ''
key: summary_secrets_management_ac2e4486-bdb1-44d8-b673-ab52485125af
modules:
- flytekit.configuration.internal.Secrets
- flytekit.core.context_manager.SecretsManager
- flytekit.models.security.Secret
questions_to_answer: []
type: summary

-->
## Secrets Management

Secrets Management provides a secure and flexible mechanism for applications to access sensitive information such as API keys, database credentials, and tokens. This system ensures that secrets are not hardcoded into your application logic or configuration files, enhancing security and maintainability.

The system distinguishes between **declaring** that a task requires a secret and **accessing** that secret at runtime.

### Declaring Secrets

To inform the system that a task requires access to a secret, use the `Secret` class. This declaration is typically made when defining a task, allowing the underlying platform to inject the necessary secrets into the task's execution environment.

The `Secret` class defines the metadata for a secret request:

*   **`group`**: The primary identifier for the secret. In many environments, this corresponds to the name of the secret object (e.g., a Kubernetes Secret name). This field is often required by the platform.
*   **`key`**: An optional identifier for a specific value within a secret group. For example, a secret group named `my-api-keys` might contain individual keys like `github_token` and `stripe_key`.
*   **`group_version`**: An optional version for the secret group, allowing for versioning of secret sets.
*   **`mount_requirement`**: Specifies how the secret should be made available to the task. This is a hint to the platform and influences how the secret is resolved at runtime.
    *   **`MountType.ANY`**: The most flexible option. The platform decides whether to inject the secret as an environment variable or a file. This is the default.
    *   **`MountType.ENV_VAR`**: The secret is injected as an environment variable. Suitable for short, non-binary secrets like API keys or passwords.
    *   **`MountType.FILE`**: The secret is made available as a file at a specific path. This is suitable for certificates, large binary secrets, or secrets that cannot be exposed as environment variables. Support for file mounting may vary across environments.
*   **`env_var`**: An optional custom environment variable name. If specified, the secret's value (or its file path, depending on `mount_requirement`) will be exposed under this exact environment variable name. If not specified, a default environment variable name is constructed based on the `group`, `group_version`, and `key`.

**Example: Declaring a Secret in a Task**

```python
from flytekit import task, Secret
from flytekit.core.secrets import SecretsManager

# Assume secrets_manager is initialized elsewhere, typically available via context
secrets_manager = SecretsManager()

@task(secret_requests=[
    Secret(group="my-api-keys", key="github_token", mount_requirement=Secret.MountType.ENV_VAR),
    Secret(group="my-certs", key="tls_cert", mount_requirement=Secret.MountType.FILE, env_var="TLS_CERT_PATH")
])
def my_secure_task():
    # Accessing secrets at runtime (see next section)
    github_token = secrets_manager.my_api_keys.github_token
    tls_cert_path = secrets_manager.my_certs.tls_cert # This will return the file path
    
    print(f"GitHub Token: {github_token}")
    print(f"TLS Cert Path: {tls_cert_path}")
    # Further logic using the secrets
```

### Accessing Secrets at Runtime

Secrets are accessed within your task's execution using the `SecretsManager`. This component provides a consistent interface for retrieving secrets, abstracting away the underlying storage mechanism.

The `SecretsManager` resolves secrets based on a predefined order:

1.  **Environment Variable**: It first attempts to find the secret as an environment variable. The environment variable name is constructed using a configurable prefix, the secret group, group version, and key (e.g., `FLYTE_SECRETS_MY_GROUP_MY_KEY`).
2.  **File System**: If the secret is not found as an environment variable, it then checks a default directory for a file matching the secret's group, group version, and key (e.g., `/etc/secrets/my_group/my_key`).

**Initialization and Configuration:**

The `SecretsManager` is initialized with configuration parameters that define the environment variable prefix and the default file directory. These parameters are derived from the `Secrets` configuration class.

*   **`Secrets.ENV_PREFIX`**: The prefix used for environment variables (default: `FLYTE_SECRETS_`). This can be overridden by the `FLYTE_SECRETS_ENV_PREFIX` environment variable.
*   **`Secrets.DEFAULT_DIR`**: The base directory where secrets are expected to be found as individual files (default: `/etc/secrets`). This can be overridden by the `FLYTE_SECRETS_DEFAULT_DIR` environment variable.
*   **`Secrets.FILE_PREFIX`**: A prefix for the secret file name within the default directory (default: empty string).

**Access Methods:**

The `SecretsManager` offers two primary ways to retrieve secrets:

1.  **Attribute-Style Access (Recommended for simplicity):**
    You can access secrets using a dot notation, where the first attribute is the secret `group` and the second is the `key`. This provides a clean and readable way to retrieve secrets.

    ```python
    # Assuming secrets_manager is an instance of SecretsManager
    my_secret_value = secrets_manager.my_group.my_key
    ```

2.  **Direct `get()` Method:**
    For more explicit control or when `group` or `key` names are dynamic, use the `get()` method.

    ```python
    # Assuming secrets_manager is an instance of SecretsManager
    my_secret_value = secrets_manager.get(group="my_group", key="my_key")
    
    # To read a binary file
    binary_secret_data = secrets_manager.get(group="my_certs", key="tls_cert", encode_mode="rb")
    ```

If a secret cannot be found through either environment variables or files, a `ValueError` is raised, indicating that the secret was not available in the expected locations.

**Local Execution Behavior:**

During local execution, the `SecretsManager` includes an additional check for environment variables *without* the configured prefix. This simplifies local development and testing, as you can set environment variables like `MY_GROUP_MY_KEY` directly without needing the full `FLYTE_SECRETS_` prefix. This behavior is automatically enabled when the execution context is detected as local.

### Best Practices and Considerations

*   **Never Hardcode Secrets**: Always declare secrets using the `Secret` class and access them via `SecretsManager` at runtime.
*   **Granularity**: Define secrets with appropriate granularity. Group related secrets together, and use keys to differentiate individual values within a group.
*   **`MountType` Selection**:
    *   Use `MountType.ENV_VAR` for simple string secrets like API keys or passwords.
    *   Use `MountType.FILE` for larger secrets, binary data, or when the secret store explicitly requires file-based mounting (e.g., certificates, SSH keys). Be aware that file mounting might have platform-specific limitations.
    *   `MountType.ANY` is a good default if you want the platform to optimize the injection method.
*   **Error Handling**: Be prepared to handle `ValueError` if a required secret is not found at runtime. This indicates a misconfiguration or missing secret injection by the platform.
*   **Environment Variable Naming**: When using `env_var` in `Secret` declaration, ensure the name follows standard environment variable naming conventions (uppercase letters, numbers, and underscores, starting with a letter or underscore). The system validates this format.
*   **Security Context**: Understand that the actual injection of secrets into the task's runtime environment is handled by the underlying execution platform (e.g., Kubernetes). The `Secret` declaration serves as a request to this platform.
<!--
key: summary_secrets_management_ac2e4486-bdb1-44d8-b673-ab52485125af
type: summary_end

-->
<!--
code_unit: flytekit.configuration.internal.Secrets
code_unit_type: class
help_text: ''
key: example_171a9fb2-b8b0-4762-82d0-d56235c61ae4
type: example

-->
<!--
code_unit: flytekit.core.context_manager.SecretsManager
code_unit_type: class
help_text: ''
key: example_f0f5c8be-24f1-44a1-a969-0fa525ffb1d1
type: example

-->
<!--
code_unit: flytekit.models.security.Secret
code_unit_type: class
help_text: ''
key: example_59ace12a-99a4-42d6-adb0-5e6ad2d68568
type: example

-->