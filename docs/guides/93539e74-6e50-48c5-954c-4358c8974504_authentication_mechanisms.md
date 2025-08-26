
<!--
help_text: ''
key: summary_authentication_mechanisms_b02d819c-005a-4ea1-b06e-b8621bcb6122
modules:
- flytekit.clients.auth.auth_client
- flytekit.clients.auth.authenticator
- flytekit.clients.auth.keyring
- flytekit.clients.auth.token_client
- flytekit.clients.auth.exceptions
- flytekit.clients.auth_helper
- flytekit.clients.grpc_utils.auth_interceptor
questions_to_answer: []
type: summary

-->
Authentication Mechanisms

Authentication mechanisms secure communication with the platform, ensuring that only authorized clients can interact with services. This system supports various OAuth2-based flows and integrates seamlessly with both gRPC and HTTP clients.

## Authentication Flows

The system provides several authentication flows tailored for different client types and environments. Each flow is managed by a dedicated authenticator.

### PKCE Flow

The Proof Key for Code Exchange (PKCE) flow is an OAuth2 extension designed for public clients (e.g., mobile apps, desktop applications) where a client secret cannot be securely stored. This flow enhances security by preventing authorization code interception attacks.

The `PKCEAuthenticator` orchestrates this flow. When authentication is required, it initiates a browser-based login. A local HTTP server, managed by the `OAuthHTTPServer` and `OAuthCallbackHandler` classes, listens for the authorization code callback from the identity provider. Once the code is received, the `AuthorizationClient` exchanges it for an access token and, if available, a refresh token.

Key aspects:
*   **Browser Interaction**: Automatically opens a web browser for user authentication.
*   **Local Callback Server**: A temporary HTTP server handles the redirect from the identity provider.
*   **Secure Token Exchange**: Uses code verifiers and challenges to secure the authorization code exchange.
*   **Credential Caching**: Successfully obtained credentials (access and refresh tokens) are securely stored using the keyring for subsequent use.

### Device Code Flow

The Device Code flow is suitable for input-constrained devices or headless environments where a browser-based redirect flow is not feasible.

The `DeviceCodeAuthenticator` implements this flow. Instead of opening a browser, it provides a URL and a user code that the user must manually enter into a browser on another device. The authenticator then polls the token endpoint until the user completes the authentication.

Key aspects:
*   **Headless Authentication**: Ideal for command-line tools or devices without a web browser.
*   **User-Friendly Prompt**: Displays a verification URL and a unique user code.
*   **Polling Mechanism**: Continuously checks the token endpoint for successful authentication.
*   **Credential Caching**: Stores credentials in the keyring upon successful authentication.

### Client Credentials Flow

The Client Credentials flow is used for machine-to-machine authentication, where an application acts on its own behalf rather than on behalf of a user. This flow requires a client ID and a client secret.

The `ClientCredentialsAuthenticator` handles this flow. It directly exchanges the client ID and client secret for an access token with the identity provider's token endpoint. This flow is typically used for service accounts or backend services.

Key aspects:
*   **Machine-to-Machine**: Designed for applications that need to access resources without user involvement.
*   **Client ID and Secret**: Requires pre-configured client credentials.
*   **Direct Token Acquisition**: Obtains an access token directly from the token endpoint.

### Command-based Authentication

For advanced scenarios, the system supports retrieving an access token from an external process.

The `CommandAuthenticator` executes a specified command. The standard output of this command is then used as the access token. This provides flexibility for integrating with custom authentication systems or existing credential management tools.

Key aspects:
*   **External Process Integration**: Leverages an external command to supply the access token.
*   **Flexible**: Supports any command that outputs a valid access token to stdout.
*   **Error Handling**: Reports errors if the command fails to execute or produce output.

## Credential Management

The system prioritizes secure and efficient credential handling.

### Credentials Object

The `Credentials` object encapsulates the authentication tokens:
*   `access_token`: The primary token used for authorizing requests.
*   `refresh_token`: (Optional) Used to obtain new access tokens without re-authenticating the user.
*   `id_token`: (Optional) An OpenID Connect token containing user identity information.
*   `expires_in`: (Optional) The lifetime of the access token in seconds.
*   `for_endpoint`: The endpoint for which these credentials are valid.

### Keyring Storage

The `KeyringStore` provides a secure mechanism for persisting credentials across sessions. It leverages the operating system's native keyring service (e.g., macOS Keychain, Windows Credential Manager, Linux Secret Service).

Key aspects:
*   **Secure Persistence**: Stores sensitive tokens securely, preventing them from being exposed in plain text.
*   **Automatic Caching**: Authenticators automatically store and retrieve credentials from the keyring, improving user experience by reducing the need for repeated logins.
*   **Error Handling**: Gracefully handles cases where a keyring service is not available, though tokens will not be cached in such scenarios.

### Token Refresh

Authenticators automatically manage the lifecycle of access tokens. When an access token expires or is rejected (e.g., with an `UNAUTHENTICATED` status), the authenticator attempts to refresh it using the stored refresh token. If refreshing fails or no refresh token is available, a full authentication flow is re-initiated. This ensures continuous access without manual intervention.

## Integration Points

Authentication is seamlessly integrated into both gRPC and HTTP communication.

### gRPC Interception

The `AuthUnaryInterceptor` is a gRPC client interceptor that automatically injects authentication headers into outgoing gRPC requests. This is critical for interacting with the Flyte Admin service.

Key aspects:
*   **Automatic Header Injection**: Adds the `Authorization` header with the current access token to every gRPC call.
*   **Transparent Refresh**: If a gRPC call returns an `UNAUTHENTICATED` status, the interceptor automatically triggers a token refresh and retries the request with the new token. This provides a robust and self-healing authentication layer for gRPC communications.

### HTTP Request Adaptation

For HTTP-based interactions, the `AuthenticationHTTPAdapter` integrates with `requests.Session` objects. This adapter ensures that all HTTP requests made through the session include the necessary authentication headers.

Key aspects:
*   **Session Integration**: Mounts onto a `requests.Session` to intercept and modify outgoing HTTP requests.
*   **Header Addition**: Adds the `Authorization` header to HTTP requests.
*   **Retry on Unauthorized**: If an HTTP request receives a `401 Unauthorized` response, the adapter attempts to refresh the credentials and retries the request, similar to the gRPC interceptor.

## Client Configuration

The system provides flexible ways to configure authentication parameters.

### ClientConfig

The `ClientConfig` class defines the essential parameters required by authenticators, such as:
*   `token_endpoint`: URL for obtaining tokens.
*   `authorization_endpoint`: URL for initiating authorization flows.
*   `redirect_uri`: Callback URL for browser-based flows.
*   `client_id`: The client identifier.
*   `scopes`: Permissions requested from the identity provider.
*   `header_key`: The HTTP header key for authorization (defaults to `authorization`).
*   `audience`: (Optional) The audience for the token.
*   `device_authorization_endpoint`: (Optional) URL for the device authorization flow.

### ClientConfigStore

The `ClientConfigStore` is an abstract interface for retrieving `ClientConfig` instances. This abstraction allows for different sources of configuration.

### RemoteClientConfigStore

The `RemoteClientConfigStore` is a concrete implementation that dynamically fetches client configuration from the Flyte Admin service. It queries the `AuthMetadataService` to retrieve public client configuration and OAuth2 metadata. This is the recommended approach for most deployments as it allows the Flyte backend to dictate the necessary authentication parameters.

### StaticClientConfigStore

The `StaticClientConfigStore` provides a way to supply a fixed `ClientConfig` object. This is useful for testing or environments where configuration is known beforehand and does not need to be fetched dynamically.

## Error Handling

The system defines specific exceptions to indicate authentication-related issues:
*   `AuthenticationError`: A general error indicating a failure in the authentication process.
*   `AccessTokenNotFoundError`: Raised when an access token is not found or cannot be refreshed.
*   `AuthenticationPending`: Specific to the Device Code flow, indicating that the user has not yet completed the authentication on their device.

## Best Practices and Considerations

*   **Security**: Avoid setting `verify` to `False` in production environments, as it disables TLS certificate verification and makes your application vulnerable to man-in-the-middle attacks.
*   **Keyring Availability**: Ensure a keyring service is available on the system for persistent and secure credential storage. If not, tokens will not be cached, potentially requiring re-authentication on every session.
*   **Choosing the Right Flow**: Select the authentication flow that best suits your application's environment and security requirements:
    *   **PKCE**: For interactive user applications (e.g., CLI tools, desktop apps).
    *   **Device Code**: For headless environments or devices with limited input capabilities.
    *   **Client Credentials**: For server-to-server or machine-to-machine communication.
    *   **Command-based**: For highly customized authentication setups.
*   **Scopes and Audience**: Configure the appropriate scopes and audience parameters to ensure the obtained tokens have the necessary permissions for interacting with the Flyte platform.
<!--
key: summary_authentication_mechanisms_b02d819c-005a-4ea1-b06e-b8621bcb6122
type: summary_end

-->
<!--
code_unit: flytekit.clients.auth.auth_client.AuthorizationClient
code_unit_type: class
help_text: ''
key: example_d3599535-59b4-4d8b-b15e-0f13bd98201c
type: example

-->
<!--
code_unit: flytekit.clients.auth.authenticator.PKCEAuthenticator
code_unit_type: class
help_text: ''
key: example_e11b83c2-e791-4a11-b5ab-a800cba3d7cf
type: example

-->
<!--
code_unit: flytekit.clients.auth.authenticator.ClientCredentialsAuthenticator
code_unit_type: class
help_text: ''
key: example_c7555338-f4fc-4374-8bd4-21d4c564b8f0
type: example

-->
<!--
code_unit: flytekit.clients.auth.authenticator.DeviceCodeAuthenticator
code_unit_type: class
help_text: ''
key: example_dc4dbaf9-b1a1-4e94-afc3-60dcc7e0bb6e
type: example

-->
<!--
code_unit: flytekit.clients.auth.keyring.KeyringStore
code_unit_type: class
help_text: ''
key: example_de0cd0a3-313b-42eb-b353-9933c397d097
type: example

-->
<!--
code_unit: flytekit.clients.auth_helper.AuthenticationHTTPAdapter
code_unit_type: class
help_text: ''
key: example_1f38aff3-0938-4b54-9913-089ffccd7956
type: example

-->
<!--
code_unit: flytekit.clients.auth_helper.RemoteClientConfigStore
code_unit_type: class
help_text: ''
key: example_b1df126d-0835-45a2-98b1-c4d547327340
type: example

-->
<!--
code_unit: flytekit.clients.grpc_utils.auth_interceptor.AuthUnaryInterceptor
code_unit_type: class
help_text: ''
key: example_71ce6d18-c16f-424c-8994-4e4ec01b698b
type: example

-->