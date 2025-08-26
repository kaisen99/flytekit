
<!--
help_text: ''
key: summary_managing_secrets_and_identities_in_workflows_ed2f7cd8-7218-49b8-81de-b7020aa6be98
modules:
- flytekit.models.security.SecurityContext
- flytekit.models.security.Secret
- flytekit.models.security.Identity
- flytekit.models.security.OAuth2Client
- flytekit.models.security.OAuth2TokenRequest
questions_to_answer: []
type: summary

-->
## Managing Secrets and Identities in Workflows

Workflows often require access to sensitive information, such as API keys, database credentials, or cloud service accounts. They also need to operate with specific permissions to interact with external systems. This capability provides a robust mechanism to manage these sensitive assets (secrets) and define the execution persona (identity) for your workflows and tasks.

### Security Context

The `SecurityContext` serves as the central configuration object for all security-related aspects of a workflow or task. It encapsulates the identity under which the workflow runs, the secrets it requires, and any OAuth2 token requests needed for authentication.

While `SecurityContext` is the underlying structure, developers primarily interact with the `Secret`, `Identity`, and `OAuth2TokenRequest` objects directly, which are then aggregated into a `SecurityContext` by the system.

A `SecurityContext` can include:
*   An `Identity` to define the execution persona.
*   A list of `Secret` objects for sensitive data injection.
*   A list of `OAuth2TokenRequest` objects for dynamic token acquisition.

### Managing Secrets

Secrets represent sensitive data that your workflow tasks need to access securely. Instead of hardcoding credentials, you define a `Secret` object, which the underlying platform then injects into the task's execution environment.

To define a secret, instantiate the `Secret` class:

```python
from flytekit import Secret

my_api_key = Secret(group="my-api-keys", key="api-key-prod")
```

Key attributes of a `Secret` include:

*   **`group`**: The primary identifier for the secret. This typically maps to a secret management system's concept of a secret, such as a Kubernetes Secret name or a vault path. This field is required during registration if the platform plugin mandates it.
*   **`key`**: An optional identifier for a specific data entry within the `group`. For example, a Kubernetes Secret might contain multiple key-value pairs; `key` specifies which one to retrieve.
*   **`group_version`**: An optional version string for the secret group, allowing for versioned secret management.
*   **`mount_requirement`**: Specifies how the secret should be exposed to the task. This is a crucial security and usability consideration.
    *   **`Secret.MountType.ANY`**: The most flexible option. The platform decides whether to inject the secret as an environment variable or a file.
    *   **`Secret.MountType.ENV_VAR`**: Injects the secret's value directly into an environment variable. Suitable for symmetric keys or simple passwords.
    *   **`Secret.MountType.FILE`**: Mounts the secret as a file in the task's container. This is preferred for certificates, large keys, or when the secret content cannot be safely exposed as an environment variable.
*   **`env_var`**: An optional custom environment variable name. If `mount_requirement` is `ENV_VAR`, the secret's value is set to this environment variable. If `mount_requirement` is `FILE`, this environment variable will contain the path to the mounted secret file. The `env_var` name must adhere to standard environment variable naming conventions (e.g., `^[A-Z_][A-Z0-9_]*$`).

**Example: Injecting a Secret as an Environment Variable**

```python
from flytekit import Secret, task, workflow

# Define a secret to be injected as an environment variable
# The platform will look for a secret named 'my-credentials' with a key 'db_password'
# and expose its value as an environment variable named 'DB_PASSWORD'.
db_password_secret = Secret(
    group="my-credentials",
    key="db_password",
    mount_requirement=Secret.MountType.ENV_VAR,
    env_var="DB_PASSWORD",
)

@task(secrets=[db_password_secret])
def connect_to_db():
    import os
    password = os.getenv("DB_PASSWORD")
    if password:
        print(f"Connecting to DB with password: {password[:3]}...") # Print partial for security
    else:
        print("DB_PASSWORD environment variable not found.")

@workflow
def my_data_workflow():
    connect_to_db()
```

**Example: Injecting a Secret as a File**

```python
from flytekit import Secret, task, workflow

# Define a secret to be injected as a file
# The platform will look for a secret named 'my-certs' with a key 'tls.key'
# and expose its path via an environment variable named 'TLS_KEY_PATH'.
tls_key_secret = Secret(
    group="my-certs",
    key="tls.key",
    mount_requirement=Secret.MountType.FILE,
    env_var="TLS_KEY_PATH",
)

@task(secrets=[tls_key_secret])
def use_tls_key():
    import os
    tls_key_path = os.getenv("TLS_KEY_PATH")
    if tls_key_path and os.path.exists(tls_key_path):
        with open(tls_key_path, "r") as f:
            key_content = f.read()
        print(f"TLS key content loaded from: {tls_key_path}")
        # Process key_content (e.g., load into a TLS library)
    else:
        print(f"TLS key file not found at {tls_key_path}.")

@workflow
def secure_communication_workflow():
    use_tls_key()
```

### Defining Execution Identities

An `Identity` specifies the security principal under which a workflow or task executes. This allows for fine-grained access control and auditing, ensuring that operations are performed with the minimum necessary privileges.

To define an identity, instantiate the `Identity` class:

```python
from flytekit import Identity

# Example: Using a Kubernetes Service Account
k8s_identity = Identity(k8s_service_account="my-workflow-sa")

# Example: Using an AWS IAM Role
iam_identity = Identity(iam_role="arn:aws:iam::123456789012:role/my-workflow-role")
```

Key attributes of an `Identity` include:

*   **`iam_role`**: Specifies an AWS IAM Role ARN. The workflow or task will assume this role, inheriting its permissions.
*   **`k8s_service_account`**: Specifies a Kubernetes Service Account name. The task's pod will be configured to use this service account, granting it the associated RBAC permissions.
*   **`oauth2_client`**: An `OAuth2Client` object, used for client credentials flow.
*   **`execution_identity`**: A generic string identifier for the execution identity, useful for custom identity management systems.

**Example: Running a Task with a Specific Service Account**

```python
from flytekit import Identity, task, workflow

# Define an identity using a Kubernetes Service Account
my_service_account = Identity(k8s_service_account="data-processing-sa")

@task(container_image="my-custom-image:latest", run_as=my_service_account)
def process_sensitive_data():
    # This task will run with the permissions granted to 'data-processing-sa'
    print("Processing sensitive data with specific service account permissions.")

@workflow
def secure_data_pipeline():
    process_sensitive_data()
```

### OAuth2 Token Requests

For scenarios requiring dynamic acquisition of OAuth2 tokens (e.g., for interacting with external APIs that use OAuth2), you can define an `OAuth2TokenRequest`. This allows the platform to obtain a token on behalf of your task.

To define an OAuth2 token request, instantiate the `OAuth2TokenRequest` class:

```python
from flytekit import OAuth2Client, OAuth2TokenRequest

# Define the OAuth2 client credentials
my_oauth_client = OAuth2Client(
    client_id="my-app-client-id",
    client_secret="my-app-client-secret-from-secret-manager" # In a real scenario, this would come from a Secret
)

# Define the token request
api_token_request = OAuth2TokenRequest(
    name="external-api-access",
    client=my_oauth_client,
    token_endpoint="https://auth.example.com/oauth2/token",
    type_=OAuth2TokenRequest.Type.CLIENT_CREDENTIALS,
)
```

Key attributes of an `OAuth2TokenRequest` include:

*   **`name`**: A unique name for this token request.
*   **`client`**: An `OAuth2Client` object containing the `client_id` and `client_secret`.
*   **`idp_discovery_endpoint`**: Optional. The OpenID Connect discovery endpoint.
*   **`token_endpoint`**: Optional. The direct OAuth2 token endpoint. If not provided, it can be discovered via `idp_discovery_endpoint`.
*   **`type_`**: The type of OAuth2 grant. Currently, only `OAuth2TokenRequest.Type.CLIENT_CREDENTIALS` is supported.

**Example: Using an OAuth2 Token Request in a Task**

```python
from flytekit import OAuth2Client, OAuth2TokenRequest, task, workflow

# In a real application, client_secret would be retrieved from a Secret
# For demonstration, we'll use a placeholder.
# You would typically define a Secret for the client_secret and reference it.
# For example:
# client_secret_secret = Secret(group="oauth-secrets", key="my-app-client-secret", mount_requirement=Secret.MountType.ENV_VAR, env_var="OAUTH_CLIENT_SECRET")
# client_secret = os.getenv("OAUTH_CLIENT_SECRET")

my_oauth_client = OAuth2Client(
    client_id="my-app-client-id",
    client_secret="dummy-client-secret" # Replace with actual secret retrieval
)

api_token_request = OAuth2TokenRequest(
    name="external-api-access",
    client=my_oauth_client,
    token_endpoint="https://auth.example.com/oauth2/token",
    type_=OAuth2TokenRequest.Type.CLIENT_CREDENTIALS,
)

@task(tokens=[api_token_request])
def call_external_api():
    # The platform will ensure the token is available, typically via an environment variable
    # or a mounted file, depending on the underlying implementation.
    # The exact mechanism for accessing the token within the task depends on the platform.
    # For example, it might be available as an environment variable named after the request name.
    import os
    token = os.getenv("EXTERNAL_API_ACCESS_TOKEN") # Example: assuming token is exposed as ENV_VAR
    if token:
        print(f"Obtained OAuth2 token: {token[:10]}...")
        # Use the token to make API calls
    else:
        print("OAuth2 token not found.")

@workflow
def oauth_workflow():
    call_external_api()
```

### Best Practices and Considerations

*   **Principle of Least Privilege**: Always define identities and secrets with the minimum necessary permissions required for the task.
*   **Secret Management System Integration**: This capability relies on an underlying secret management system (e.g., Kubernetes Secrets, AWS Secrets Manager, HashiCorp Vault) configured in your environment. The `group` and `key` attributes of a `Secret` map directly to how secrets are organized in these systems.
*   **Avoid Hardcoding**: Never hardcode sensitive information directly in your workflow definitions. Always use the `Secret` mechanism.
*   **Mount Type Selection**: Choose `Secret.MountType.FILE` for highly sensitive secrets like private keys or certificates, as environment variables can sometimes be inadvertently exposed (e.g., in logs or process lists). Use `Secret.MountType.ENV_VAR` for less sensitive data or when file mounting is not feasible.
*   **Platform-Specific Behavior**: The exact mechanism by which secrets and identities are injected (e.g., specific environment variable names for OAuth2 tokens, file paths for mounted secrets) can vary slightly depending on the execution platform and its configuration. Refer to your platform's documentation for specifics.
*   **Auditing and Logging**: Ensure your secret management system and execution environment provide adequate auditing and logging capabilities for secret access and identity usage.
<!--
key: summary_managing_secrets_and_identities_in_workflows_ed2f7cd8-7218-49b8-81de-b7020aa6be98
type: summary_end

-->
<!--
code_unit: flytekit.examples.security.secrets
code_unit_type: class
help_text: ''
key: example_6b244e8e-f52c-48af-95ec-5943a09838e0
type: example

-->
<!--
code_unit: flytekit.examples.security.identity
code_unit_type: class
help_text: ''
key: example_5ebf2b21-75ae-41f9-ad12-9da8939cda29
type: example

-->