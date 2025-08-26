
<!--
help_text: ''
key: summary_secrets_management_2951c061-62b4-4e35-9ee2-4af61096d478
modules:
- flytekit.configuration.internal.Secrets
questions_to_answer: []
type: summary

-->
Secrets Management provides mechanisms for securely handling sensitive information, such as API keys, database credentials, and access tokens, within applications. It supports two primary methods for accessing secrets: runtime injection via environment variables and retrieval from a designated file system location.

### Runtime Secrets Injection

Applications access secrets injected as environment variables at runtime. This method is suitable for environments where secrets are dynamically provisioned and exposed to the application container.

The `Secrets` class defines the configuration for this mechanism:

*   `ENV_PREFIX`: Specifies the prefix used to identify injected secrets in environment variables. For example, if `ENV_PREFIX` is `MY_SECRET_`, then an environment variable named `MY_SECRET_API_KEY` is recognized as a secret.
    *   Developers can override this prefix by setting the `FLYTE_SECRETS_ENV_PREFIX` environment variable.

**Example: Accessing an Injected Secret**

Assume `Secrets.ENV_PREFIX` is configured to `MYAPP_SECRET_`. If an environment variable `MYAPP_SECRET_DB_PASSWORD` is set to `supersecurepassword`, an application retrieves it as follows:

```python
import os
from your_module import Secrets # Replace 'your_module' with the actual module path

# Retrieve the configured environment variable prefix
env_prefix = Secrets.ENV_PREFIX.get()

secret_name = "DB_PASSWORD"
full_env_var_name = f"{env_prefix}{secret_name}"
db_password = os.getenv(full_env_var_name)

if db_password:
    print(f"Database password retrieved: {db_password}")
else:
    print(f"Secret '{secret_name}' not found via environment variable '{full_env_var_name}'")

# To demonstrate overriding the prefix at runtime:
# os.environ["FLYTE_SECRETS_ENV_PREFIX"] = "CUSTOM_SECRET_"
# new_env_prefix = Secrets.ENV_PREFIX.get() # This would now reflect "CUSTOM_SECRET_"
```

### File-Based Secrets Retrieval

Secrets can also reside as individual files within a designated directory. This approach is useful for scenarios where secrets are mounted as files into the application's filesystem, such as Kubernetes secrets mounted as volumes.

The `Secrets` class provides configuration for file-based secrets:

*   `DEFAULT_DIR`: Defines the default directory where secret files are expected. For instance, `/etc/secrets/myapp`.
    *   Developers can override this default directory by setting the `FLYTE_SECRETS_DEFAULT_DIR` environment variable.
*   `FILE_PREFIX`: Specifies an optional prefix for the secret filenames within the `DEFAULT_DIR`. This helps organize and identify secret files.

**Example: Accessing a File-Based Secret**

Assume `Secrets.DEFAULT_DIR` is configured to `/run/secrets` and `Secrets.FILE_PREFIX` is `app_`. If a file named `/run/secrets/app_api_key` contains `myapikey123`, an application retrieves it as follows:

```python
import os
from your_module import Secrets # Replace 'your_module' with the actual module path

# Retrieve the configured default directory and file prefix
default_dir = Secrets.DEFAULT_DIR.get()
file_prefix = Secrets.FILE_PREFIX.get()

secret_name = "api_key"
secret_file_path = os.path.join(default_dir, f"{file_prefix}{secret_name}")

try:
    with open(secret_file_path, "r") as f:
        api_key = f.read().strip()
    print(f"API Key retrieved from file: {api_key}")
except FileNotFoundError:
    print(f"Secret file '{secret_file_path}' not found.")
except Exception as e:
    print(f"Error reading secret file '{secret_file_path}': {e}")

# To demonstrate overriding the default directory at runtime:
# os.environ["FLYTE_SECRETS_DEFAULT_DIR"] = "/var/run/custom_secrets"
# new_default_dir = Secrets.DEFAULT_DIR.get() # This would now reflect "/var/run/custom_secrets"
```

### Best Practices and Considerations

*   **Never hardcode secrets:** Always use environment variables, file mounts, or a dedicated secrets management system.
*   **Least privilege:** Ensure applications only have access to the secrets they require.
*   **Runtime overrides:** The ability to override `ENV_PREFIX` and `DEFAULT_DIR` using `FLYTE_SECRETS_ENV_PREFIX` and `FLYTE_SECRETS_DEFAULT_DIR` environment variables provides flexibility for different deployment environments without requiring code changes.
*   **Security:** While this system helps locate secrets, the underlying security of the secrets (e.g., encryption at rest, access control to environment variables/files) depends on the deployment environment (e.g., Kubernetes, Docker).
*   **Error Handling:** Implement robust error handling when accessing secrets, as they might not always be present or accessible.
<!--
key: summary_secrets_management_2951c061-62b4-4e35-9ee2-4af61096d478
type: summary_end

-->
<!--
code_unit: flytekit.configuration.internal
code_unit_type: class
help_text: ''
key: example_41b8376d-cc36-4b7a-85dd-cbf475afbd45
type: example

-->