
<!--
help_text: ''
key: summary_authentication_&_authorization_9cf3d0be-3d5e-49c6-be9b-215f71b0a8a5
modules:
- flytekit.clients.auth.auth_client
- flytekit.clients.auth.authenticator
- flytekit.clients.auth.keyring
- flytekit.models.security.SecurityContext
- flytekit.models.security.Identity
questions_to_answer: []
type: summary

-->
Authentication and authorization are fundamental security concepts that control access to resources and actions within the system. Authentication verifies the identity of a user or service, while authorization determines what an authenticated identity is permitted to do.

The system provides robust mechanisms for managing identities, defining security contexts for executions, and supporting various authentication flows to obtain necessary credentials.

### Defining Execution Identity and Security Context

The system uses specific constructs to define the identity under which an execution runs and to specify its overall security posture.

#### Identity

An `Identity` object defines the principal under which a task or workflow executes. This identity is crucial for authorization, as it dictates the permissions available to the running code. An identity can be specified using:

*   **IAM Role**: An AWS Identity and Access Management (IAM) role ARN, granting permissions within AWS.
*   **Kubernetes Service Account**: A Kubernetes service account name, granting permissions within the Kubernetes cluster.
*   **OAuth2 Client**: An OAuth2 client ID, typically used for programmatic access.
*   **Execution Identity**: A generic string identifier for the execution.

When defining a task or workflow, specifying an `Identity` ensures that the execution operates with the appropriate privileges, adhering to the principle of least privilege.

#### Security Context

The `SecurityContext` object serves as a comprehensive container for all security-related parameters associated with an execution. While users typically interact with higher-level abstractions like `Secret` objects, the `SecurityContext` internally aggregates:

*   **Run-as Identity**: The `Identity` object specifying who the execution runs as.
*   **Secrets**: A list of `Secret` objects, providing access to sensitive information.
*   **Tokens**: A list of `OAuth2TokenRequest` objects, used for requesting specific OAuth2 tokens.

The `SecurityContext` ensures that all necessary security configurations, including the execution identity and access to secrets, are bundled together for a given task or workflow.

### Authentication Mechanisms

The system supports multiple authentication flows, each tailored for different use cases and environments. All authentication flows are managed through concrete implementations of the abstract `Authenticator` base class, which provides a standardized interface for fetching and refreshing credentials.

#### PKCE Authenticator

The PKCE Authenticator implements the OAuth 2.0 Authorization Code Flow with Proof Key for Code Exchange (PKCE). This flow is designed for interactive user logins, typically involving a web browser.

*   **Purpose**: Enables user authentication in environments where a browser can be opened, such as a local development machine or a desktop application.
*   **Mechanism**:
    1.  Generates a `code_verifier` and `code_challenge`.
    2.  Opens a web browser to the authorization endpoint, including the `code_challenge` and a `redirect_uri`.
    3.  The user authenticates in the browser, and the authorization server redirects back to a local HTTP server (`OAuthHTTPServer` and `OAuthCallbackHandler`) running on the `redirect_uri`.
    4.  The local server captures the authorization code.
    5.  The authenticator exchanges the authorization code and `code_verifier` for an access token and refresh token at the token endpoint.
*   **Configuration**: Requires `ClientConfig` parameters such as `authorization_endpoint`, `token_endpoint`, `client_id`, `redirect_uri`, `audience`, and `scopes`.
*   **Credential Storage**: Automatically caches the obtained access and refresh tokens using the system's keyring (`KeyringStore`) for persistent storage, minimizing re-authentication prompts.
*   **Refresh**: Attempts to refresh the access token using the refresh token before initiating a full re-authentication flow.

#### Device Code Authenticator

The Device Code Authenticator implements the OAuth 2.0 Device Authorization Grant flow, suitable for headless environments or devices with limited input capabilities.

*   **Purpose**: Authenticates users in environments without a web browser or where direct browser interaction is not feasible, such as command-line interfaces (CLIs) or IoT devices.
*   **Mechanism**:
    1.  Requests a device code from the device authorization endpoint.
    2.  The server responds with a `verification_uri` and a `user_code`.
    3.  The system prompts the user to navigate to the `verification_uri` on a separate device (e.g., their phone or computer) and enter the `user_code`.
    4.  The authenticator polls the token endpoint until the user completes the authentication on the separate device.
    5.  Upon successful authentication, an access token and refresh token are obtained.
*   **Configuration**: Requires `ClientConfig` parameters such as `device_authorization_endpoint`, `token_endpoint`, `client_id`, `audience`, and `scopes`.
*   **Credential Storage**: Caches the obtained access and refresh tokens using the system's keyring (`KeyringStore`).
*   **Refresh**: Supports refreshing access tokens using the refresh token. If refresh fails, it initiates a new device code flow.

#### Client Credentials Authenticator

The Client Credentials Authenticator uses the OAuth 2.0 Client Credentials Grant flow, designed for machine-to-machine authentication.

*   **Purpose**: Authenticates services or applications directly, without user involvement. Ideal for backend services or automated scripts.
*   **Mechanism**: Uses a pre-configured `client_id` and `client_secret` to directly request an access token from the token endpoint.
*   **Configuration**: Requires the `client_id`, `client_secret`, `token_endpoint`, `audience`, and `scopes`.
*   **Credential Storage**: Stores the access token in memory for the session. This flow typically does not involve refresh tokens as it's for machine-to-machine.
*   **Refresh**: Re-requests a new access token using the client credentials when the current token expires or is invalid.

#### Command Authenticator

The Command Authenticator provides flexibility by allowing an external command or script to handle the authentication process.

*   **Purpose**: Integrates with custom or existing authentication systems that can output an access token. Useful for complex enterprise setups or when specific security tools are mandated.
*   **Mechanism**: Executes a specified command (e.g., `gcloud auth print-access-token`, `aws sts get-caller-identity`) and expects the access token to be printed to standard output.
*   **Configuration**: Requires a list of strings representing the command and its arguments.
*   **Credential Storage**: Stores the access token in memory for the session.
*   **Refresh**: Re-executes the command to obtain a new access token when needed.

### Credential Management

The system centralizes the management of authentication tokens to ensure secure and efficient access.

#### Credentials

The `Credentials` object encapsulates the tokens obtained during an authentication flow. It stores:

*   `access_token`: The primary token used for authorizing API requests.
*   `refresh_token`: (Optional) A token used to obtain new access tokens without re-authenticating the user.
*   `id_token`: (Optional) An OpenID Connect (OIDC) ID token, containing user identity information.
*   `expires_in`: (Optional) The lifetime of the access token in seconds.
*   `for_endpoint`: The endpoint for which these credentials are valid.

#### Keyring Storage

The `KeyringStore` provides a secure and persistent mechanism for caching `Credentials` using the operating system's native keyring service.

*   **Persistence**: Stores `access_token`, `refresh_token`, and `id_token` securely across sessions.
*   **Security**: Leverages the OS keyring, which is designed for sensitive data storage, preventing tokens from being exposed in plain text files.
*   **Convenience**: Automatically retrieves cached credentials, reducing the need for repeated interactive logins, especially for PKCE and Device Code flows.
*   **Endpoint-Specific**: Credentials are stored and retrieved based on the target endpoint, allowing multiple sets of credentials for different environments.

If the keyring is unavailable (e.g., in certain containerized environments), tokens are not cached, and re-authentication may be required more frequently.

### Client Configuration

Authentication flows rely on client-side configuration to connect to the correct identity provider endpoints.

#### Client Configuration (`ClientConfig` and `ClientConfigStore`)

The `ClientConfig` object defines the essential parameters for an OAuth2 client:

*   `token_endpoint`: The URL for exchanging authorization codes or refresh tokens for access tokens.
*   `authorization_endpoint`: The URL where the user is redirected to authorize the application (for PKCE).
*   `redirect_uri`: The URL where the authorization server redirects the user after authentication (for PKCE).
*   `client_id`: The public identifier for the client application.
*   `device_authorization_endpoint`: (Optional) The URL for initiating the device code flow.
*   `scopes`: A list of requested permissions (e.g., `offline_access`, `all`, `openid`).
*   `header_key`: The HTTP header key used for sending the access token (defaults to `authorization`).
*   `audience`: (Optional) The audience for the requested token, often used in Auth0.

The `ClientConfigStore` is an abstract interface for retrieving `ClientConfig`. The `StaticClientConfigStore` is a concrete implementation that provides a fixed `ClientConfig` object. In practice, client configuration is often loaded from a configuration file (e.g., `config.yaml`) or environment variables.

### Integration Patterns and Best Practices

*   **Applying Security Context**: When defining tasks or workflows, specify the `SecurityContext` to ensure they run with the correct `Identity` and have access to required `Secrets`. This is critical for fine-grained authorization.
*   **Automatic Authenticator Selection**: The system typically selects the appropriate authenticator based on the configured authentication mode (e.g., `PKCE`, `DeviceCode`, `ClientCredentials`, `Command`). Developers configure the mode, and the system handles the underlying authentication flow.
*   **Leveraging Keyring**: For interactive user flows (PKCE, Device Code), the `KeyringStore` significantly improves user experience by securely caching tokens. Ensure the keyring service is properly configured and accessible in the execution environment.
*   **Security Considerations**:
    *   **`verify` Parameter**: The `verify` parameter in `AuthorizationClient` and `Authenticator` controls TLS certificate verification. For production environments, always set `verify=True` (default) or provide a path to a CA bundle to prevent Man-in-the-Middle (MitM) attacks. Setting `verify=False` should only be used for local development or testing.
    *   **Client Secrets**: For `ClientCredentialsAuthenticator`, ensure `client_secret` is managed securely, preferably through environment variables or a secrets management system, and never hardcoded.
    *   **Scopes**: Request only the necessary scopes to adhere to the principle of least privilege.
*   **Error Handling**: Authentication failures (e.g., `AuthenticationError`, `AccessTokenNotFoundError`) indicate issues with credentials or configuration. Implement robust error handling to guide users or trigger re-authentication.
<!--
key: summary_authentication_&_authorization_9cf3d0be-3d5e-49c6-be9b-215f71b0a8a5
type: summary_end

-->
<!--
code_unit: flytekit.clients.auth.authenticator.PKCEAuthenticator
code_unit_type: class
help_text: ''
key: example_b78ffe81-3c27-43ec-870b-30b23ac275e1
type: example

-->
<!--
code_unit: flytekit.clients.auth.authenticator.ClientCredentialsAuthenticator
code_unit_type: class
help_text: ''
key: example_82ce914d-5c26-4ca8-810c-0793fd974598
type: example

-->
<!--
code_unit: flytekit.clients.auth.authenticator.DeviceCodeAuthenticator
code_unit_type: class
help_text: ''
key: example_d3fb4168-3fb4-46b4-9486-5c210f14e5a2
type: example

-->
<!--
code_unit: flytekit.models.security.SecurityContext
code_unit_type: class
help_text: ''
key: example_2487bcce-b173-49b5-80a3-a729542f6d72
type: example

-->