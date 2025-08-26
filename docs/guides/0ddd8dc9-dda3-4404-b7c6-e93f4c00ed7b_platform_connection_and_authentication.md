
<!--
help_text: ''
key: summary_platform_connection_and_authentication_37f1ddb4-408b-4b8d-82fb-8baf52ac6ff6
modules:
- flytekit.configuration.internal
- flytekit.models.security
- flytekit.models.common
questions_to_answer: []
type: summary

-->
# Platform Connection and Authentication

Flytekit provides robust mechanisms for connecting to the Flyte platform and managing authentication and authorization for both client-side operations and workflow executions. This section details how to configure platform connectivity, authenticate with the Admin service, and manage secrets and execution identities.

## Platform Connectivity Configuration

The core configuration for connecting to the Flyte Admin service is managed through the `Platform` configuration. This configuration defines the endpoint, security settings, and proxy details for client-side interactions.

Key configuration options include:

*   **URL**: Specifies the endpoint of the Flyte Admin service. This is the primary address for all API interactions.
*   **INSECURE**: A boolean flag indicating whether to use an insecure connection (e.g., HTTP instead of HTTPS). This should generally be `False` for production environments.
*   **INSECURE_SKIP_VERIFY**: When `True`, this flag disables SSL certificate verification. Use with caution, primarily for development or testing environments where certificate issues might arise.
*   **CONSOLE_ENDPOINT**: Defines the URL for the Flyte Console, providing a web-based interface to monitor and manage Flyte resources.
*   **CA_CERT_FILE_PATH**: Specifies the path to a custom CA certificate file for secure connections, useful in environments with self-signed certificates.
*   **HTTP_PROXY_URL**: Configures an HTTP proxy for all outgoing requests to the Flyte Admin service.

These settings are typically defined in a Flyte configuration file (e.g., `config.yaml`) or via environment variables.

## Authentication Mechanisms

Authentication with the Flyte Admin service is handled by the `Credentials` configuration, supporting various methods to obtain and refresh access tokens.

The `AUTH_MODE` setting dictates the authentication flow:

*   **`standard` or `Pkce`**: Utilizes the PKCE-enhanced Authorization Code Flow. This mode typically opens a browser window to facilitate user login and token acquisition.
*   **`DeviceFlow`**: Implements the OAuth 2.0 Device Authorization Grant, suitable for devices with limited input capabilities (e.g., CLI tools).
*   **`basic`**, **`client_credentials`**, or **`clientSecret`**: Employs symmetric key authentication. This method requires a `CLIENT_ID` and a `CLIENT_CREDENTIALS_SECRET`. The secret can be provided directly, from a file specified by `CLIENT_CREDENTIALS_SECRET_LOCATION`, or from an environment variable defined by `CLIENT_CREDENTIALS_SECRET_ENV_VAR`. Using file mounts or environment variables for secrets is recommended over direct configuration for enhanced security.
*   **`None`**: Disables authentication attempts. This is typically used when the Flyte Admin service does not require authentication.

For advanced scenarios, the `COMMAND` and `PROXY_COMMAND` options allow specifying external processes to generate authentication tokens. The `SCOPES` setting can be used to manually pass OAuth2 scopes, which is useful for compatibility with certain identity providers like Auth0.

Internally, authentication requests are modeled using `OAuth2TokenRequest`, which encapsulates the `OAuth2Client` (containing `client_id` and `client_secret`) along with identity provider and token endpoints.

## Execution Identity and Authorization

Beyond client-side authentication, Flyte provides mechanisms to define the identity under which your workflows and tasks execute on the platform. This is crucial for granting necessary permissions to access external resources (e.g., cloud storage, databases).

The `Identity` model allows specifying:

*   **`iam_role`**: For cloud environments like AWS, this specifies an IAM role that the execution assumes. This role dictates the permissions available to the running task or workflow.
*   **`k8s_service_account`**: In Kubernetes-native deployments, this specifies a Kubernetes Service Account. The permissions associated with this service account are granted to the execution pod.
*   **`oauth2_client`**: An `OAuth2Client` can be specified to enable client-based authentication for the execution itself, allowing the workflow to obtain tokens for external services.
*   **`execution_identity`**: A generic identifier for the execution's identity.

These identity settings are aggregated within a `SecurityContext` object, which can also include `Secret` definitions and `OAuth2TokenRequest` configurations. The `SecurityContext` provides a comprehensive way to define the security posture for a Flyte execution.

For configuring execution roles, the `AuthRole` component allows specifying an `assumable_iam_role` or a `kubernetes_service_account`. This is typically set at the workflow or launch plan level to define the runtime permissions.

## Secret Management

Flytekit offers robust secret management capabilities, allowing sensitive information to be securely injected into your task and workflow executions without hardcoding.

The `Secrets` configuration defines how secrets are discovered and accessed at runtime:

*   **`ENV_PREFIX`**: A prefix used to look up injected secrets from environment variables. This can be overridden by the `FLYTE_SECRETS_ENV_PREFIX` environment variable.
*   **`DEFAULT_DIR`**: The default directory where secrets are expected to be found as individual files. This can be overridden by `FLYTE_SECRETS_DEFAULT_DIR`.
*   **`FILE_PREFIX`**: A prefix for secret filenames within the `DEFAULT_DIR`.

At the workflow or task definition level, secrets are declared using the `Secret` model. Each `Secret` can specify:

*   **`group`**: The logical name of the secret (e.g., Kubernetes secret name). This is a required parameter during registration if the plugin enforces it.
*   **`key`**: An optional identifier for a specific secret within a group (e.g., a key within a Kubernetes secret).
*   **`group_version`**: An optional version for the secret group.
*   **`mount_requirement`**: Defines how the secret should be injected into the execution environment:
    *   **`ANY`**: The platform decides whether to inject as an environment variable or a file. This is the most flexible option.
    *   **`ENV_VAR`**: Injects the secret value directly as an environment variable. Suitable for symmetric keys or passwords.
    *   **`FILE`**: Mounts the secret as a file, with the path exposed via an environment variable. Recommended for sensitive data that should not be in environment variables or for larger secrets.
*   **`env_var`**: An optional custom environment variable name to expose the secret. If `mount_requirement` is `ENV_VAR`, this variable holds the secret value. If `FILE`, it holds the path to the secret file. Environment variable names must adhere to standard naming conventions (uppercase letters, numbers, and underscores, starting with a letter or underscore).

Secrets are typically managed by the underlying execution environment (e.g., Kubernetes secrets, AWS Secrets Manager) and then made available to Flyte tasks based on these configurations. This ensures that sensitive data is not exposed in code or logs.

## Best Practices and Considerations

*   **Secure Configuration**: Always prefer using environment variables or mounted files for sensitive information like client secrets and API keys over hardcoding them in configuration files.
*   **Least Privilege**: Configure execution identities (IAM roles, Kubernetes service accounts) with the minimum necessary permissions required for your workflows and tasks to function.
*   **Centralized Secret Management**: Leverage your cloud provider's or Kubernetes' native secret management solutions, and configure Flytekit to access them securely.
*   **Network Security**: Ensure that the Flyte Admin endpoint is accessible from your client environment, and consider network policies (firewalls, VPCs) to restrict access.
*   **Certificate Management**: For production deployments, always use valid SSL/TLS certificates and avoid `INSECURE` or `INSECURE_SKIP_VERIFY` flags. If using self-signed certificates, ensure `CA_CERT_FILE_PATH` is correctly configured.
*   **Proxy Configuration**: If operating within a corporate network that requires a proxy, ensure `HTTP_PROXY_URL` is correctly set to allow Flytekit to communicate with the Admin service.
<!--
key: summary_platform_connection_and_authentication_37f1ddb4-408b-4b8d-82fb-8baf52ac6ff6
type: summary_end

-->
<!--
code_unit: flytekit.configuration.internal.Platform
code_unit_type: class
help_text: ''
key: example_c848a1d2-c0db-4b13-a245-db224a4b5d8d
type: example

-->
<!--
code_unit: flytekit.configuration.internal.Credentials
code_unit_type: class
help_text: ''
key: example_ab7204ca-eede-48c6-80d2-4f610cfd85f3
type: example

-->
<!--
code_unit: flytekit.models.security.SecurityContext
code_unit_type: class
help_text: ''
key: example_048edc33-1281-47f1-9a8e-4fe087c67c79
type: example

-->
<!--
code_unit: flytekit.models.security.Secret
code_unit_type: class
help_text: ''
key: example_6c7e5a6e-79e8-4685-aa0e-eeaf4a45a959
type: example

-->