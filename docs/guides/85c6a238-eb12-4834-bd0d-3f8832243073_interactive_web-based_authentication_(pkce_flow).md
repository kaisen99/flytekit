
<!--
help_text: ''
key: summary_interactive_web-based_authentication_(pkce_flow)_3f0137c8-43af-47ee-8d20-90669819b732
modules:
- flytekit.clients.auth.authenticator.PKCEAuthenticator
- flytekit.clients.auth.auth_client.AuthorizationClient
- flytekit.clients.auth.auth_client.OAuthCallbackHandler
- flytekit.clients.auth.auth_client.OAuthHTTPServer
- flytekit.clients.auth.auth_client.AuthorizationCode
- flytekit.clients.auth.auth_client.EndpointMetadata
questions_to_answer: []
type: summary

-->
Interactive Web-Based Authentication (PKCE Flow)

This documentation describes the implementation of the Proof Key for Code Exchange (PKCE) flow for interactive web-based authentication. This mechanism provides a secure way for client applications, such as command-line interfaces or desktop applications, to authenticate with an OAuth 2.0 authorization server without requiring a client secret. The flow automatically opens a browser window for user interaction, streamlining the login process.

## Core Capabilities

The PKCE flow implementation provides:

*   **Automated Browser-Based Login:** Initiates the authentication process by automatically opening a web browser, guiding the user through the Identity Provider's (IdP) login page.
*   **Secure Authorization Code Handling:** Leverages the PKCE extension to securely exchange the authorization code for an access token, mitigating interception attacks.
*   **Automatic Token Refresh:** Manages the lifecycle of access tokens by automatically refreshing them using a refresh token when available and necessary.
*   **Credential Persistence:** Securely stores obtained credentials (access token, refresh token, ID token) using a keyring mechanism, allowing for persistence across application sessions.
*   **Configurable Endpoints and Scopes:** Supports flexible configuration of OAuth 2.0 endpoints, client IDs, redirect URIs, and requested scopes.

## PKCE Authenticator

The `PKCEAuthenticator` serves as the primary interface for initiating and managing the PKCE authentication flow. It encapsulates the complexities of the OAuth 2.0 and PKCE protocols, providing a simplified API for developers.

### Initialization

To initialize the PKCE authenticator, provide the following parameters:

*   `endpoint`: The target service endpoint for which authentication is required.
*   `cfg_store`: A `ClientConfigStore` instance that provides essential OAuth parameters such as `redirect_uri`, `client_id`, `audience`, `authorization_endpoint`, and `token_endpoint`.
*   `scopes` (optional): A list of OAuth 2.0 scopes to request. If not provided, scopes from the `ClientConfigStore` are used. For Auth0, including `offline_access` or similar scopes is necessary to obtain a refresh token for caching.
*   `header_key` (optional): The HTTP header key to use for sending the access token.
*   `verify` (optional): Controls TLS certificate verification. Set to `False` for development or testing environments where self-signed certificates might be used.
*   `session` (optional): A custom `requests.Session` object for making HTTP requests.

The authenticator extends a base `Authenticator` class and initializes it with credentials retrieved from a `KeyringStore` associated with the given `endpoint`.

### Flow Orchestration

The authenticator orchestrates the PKCE flow by:

1.  **Generating PKCE Parameters:** It generates a `code_verifier` and derives a `code_challenge` from it. These are crucial for the PKCE security extension.
2.  **Initializing the Authorization Client:** An `AuthorizationClient` instance is created, configured with the necessary OAuth parameters from the `ClientConfigStore` and the generated PKCE parameters (`code_challenge`, `code_challenge_method`, `code_verifier`).
3.  **Credential Management:** The `refresh_credentials` method is invoked to either refresh an existing access token or initiate a full authentication flow if no valid token is present or if refresh fails.

## Authorization Client

The `AuthorizationClient` handles the direct communication with the OAuth 2.0 authorization and token endpoints. It implements the core logic for requesting authorization codes, exchanging them for access tokens, and refreshing tokens.

### Key Operations

*   **Requesting Authorization Code:** The `_request_authorization_code` method constructs the authorization URL with parameters like `client_id`, `response_type` (set to `code`), `scope`, `redirect_uri`, `state`, and the `code_challenge`. It then opens this URL in a new browser tab, prompting the user for authentication.
*   **Exchanging Authorization Code for Access Token:** After the user authenticates and the IdP redirects back to the local callback server, the `_request_access_token` method is called. It sends a POST request to the token endpoint, including the received `code`, `grant_type` (set to `authorization_code`), and crucially, the `code_verifier` that matches the initial `code_challenge`.
*   **Refreshing Access Token:** The `refresh_access_token` method uses a stored `refresh_token` to obtain a new access token without requiring user interaction. This involves a POST request to the token endpoint with `grant_type` set to `refresh_token` and the `refresh_token` itself. If the refresh token is invalid or expired, an `AccessTokenNotFoundError` is raised, signaling the need for a full re-authentication.
*   **Retrieving Credentials:** The `get_creds_from_remote` method is the entry point for obtaining credentials. It manages the entire interactive flow, including spinning up the local callback server, opening the browser, waiting for the authorization code, and then exchanging it for the access token. This method is thread-safe, using a lock to prevent multiple concurrent browser openings and caching results for a short duration.

### PKCE Implementation Details

The authorization client explicitly incorporates PKCE parameters:

*   `code_challenge`: Sent during the initial authorization request to the IdP.
*   `code_challenge_method`: Set to `S256`, indicating the SHA256 hash method used for the code challenge.
*   `code_verifier`: Sent during the access token request. The IdP verifies that the `code_verifier` matches the `code_challenge` it received earlier.

The `add_request_auth_code_params_to_request_access_token_params` parameter in the `AuthorizationClient`'s constructor is important. When set to `True`, it ensures that parameters sent during the initial authorization code request (like `code_challenge`) are also included when exchanging the authorization code for the access token. This is required for compatibility with certain IdPs, such as FlyteAdmin.

## Local Callback Server

To capture the authorization code redirected from the Identity Provider, a temporary local HTTP server is spun up. This server listens on the `redirect_uri` specified in the client configuration.

*   **OAuthHTTPServer:** This class extends `BaseHTTPServer.HTTPServer` and binds to the host and port specified in the `redirect_uri`. It is responsible for handling incoming HTTP requests and passing the authorization code to the `AuthorizationClient`. It can also be configured with `EndpointMetadata` to display custom success or failure HTML pages in the browser after authentication.
*   **OAuthCallbackHandler:** This handler processes the GET request to the `redirect_uri`. It extracts the `code` and `state` parameters from the URL query string and encapsulates them in an `AuthorizationCode` object. This object is then placed into a queue, which the `AuthorizationClient` monitors to retrieve the authorization code.

This local server mechanism is crucial for the interactive flow, as it allows the client application to receive the authorization code securely without exposing it directly in the application's logs or command line.

## Credential Management and Persistence

Credentials obtained through the PKCE flow are managed to ensure secure storage and efficient reuse.

*   **KeyringStore:** Access tokens, refresh tokens, their expiry times, and ID tokens are securely stored using a `KeyringStore`. This provides a persistent and secure storage mechanism that leverages the operating system's native keyring services (e.g., macOS Keychain, Windows Credential Manager, Linux Keyring). This eliminates the need for users to re-authenticate frequently.
*   **Token Refresh:** The `refresh_access_token` method is central to maintaining an active session. When an access token expires, the system attempts to use the stored refresh token to obtain a new access token. If this refresh operation fails (e.g., the refresh token itself has expired or been revoked), an `AccessTokenNotFoundError` is raised. This exception signals to the `PKCEAuthenticator` that a full re-authentication flow, involving browser interaction, is required.

## Configuration and Customization

The PKCE flow implementation offers several configuration points for integration with various OAuth 2.0 providers and specific application requirements.

*   **Client Configuration:** The `PKCEAuthenticator` relies on a `ClientConfigStore` to retrieve critical OAuth parameters. This store typically provides:
    *   `redirect_uri`: The URI where the IdP redirects the user after authentication. This must be a local URI (e.g., `http://localhost:8000/callback`) and must be registered with the IdP.
    *   `client_id`: The public client identifier registered with the IdP.
    *   `audience`: (Optional) An identifier for the API or resource server that the access token is intended for, particularly relevant for Auth0.
    *   `authorization_endpoint`: The IdP's endpoint for initiating the authorization flow.
    *   `token_endpoint`: The IdP's endpoint for exchanging authorization codes or refresh tokens for access tokens.
*   **Scopes:** Scopes define the permissions requested from the user. They can be specified during the `PKCEAuthenticator`'s initialization or retrieved from the `ClientConfigStore`. For Auth0, to ensure a refresh token is issued, scopes like `offline_access` or `offline` must be requested.
*   **TLS Verification (`verify`):** The `verify` parameter in the `PKCEAuthenticator` and `AuthorizationClient` controls whether TLS certificates are verified during HTTP requests. While `True` (default) is recommended for production, setting it to `False` can be useful for local development or testing with self-signed certificates.
*   **Custom HTTP Session (`session`):** A custom `requests.Session` object can be provided to the `PKCEAuthenticator` and `AuthorizationClient`. This allows for advanced HTTP client configurations, such as custom headers, proxies, or connection pooling.
*   **Endpoint Metadata (`EndpointMetadata`):** This class allows customization of the HTML pages displayed in the browser after a successful or failed login. Developers can provide custom `success_html` and `failure_html` content to enhance the user experience.

## Usage Example

To use the PKCE authenticator:

```python
import requests
from your_package.auth.authenticator import PKCEAuthenticator
from your_package.auth.config import ClientConfigStore # Assuming this exists
from your_package.auth.credentials import Credentials # Assuming this exists

# Example ClientConfigStore (in a real application, this would be loaded from config)
class MyClientConfigStore(ClientConfigStore):
    def get_client_config(self):
        # Replace with your actual configuration values
        return type('ClientConfig', (object,), {
            'redirect_uri': 'http://localhost:8000/callback',
            'client_id': 'YOUR_CLIENT_ID',
            'audience': 'YOUR_AUDIENCE_URL', # Optional, e.g., for Auth0
            'authorization_endpoint': 'https://YOUR_AUTH_DOMAIN/authorize',
            'token_endpoint': 'https://YOUR_AUTH_DOMAIN/oauth/token',
            'scopes': ['openid', 'profile', 'email', 'offline_access']
        })()

# Initialize the authenticator
# The endpoint typically identifies the service you are authenticating against
authenticator = PKCEAuthenticator(
    endpoint="my_service_endpoint",
    cfg_store=MyClientConfigStore(),
    # Optional: override scopes from config store
    # scopes=["openid", "profile", "email", "offline_access"],
    # Optional: disable TLS verification for local testing
    # verify=False,
)

try:
    # This will trigger the browser-based login if no valid token is found
    # or if the refresh token is expired.
    credentials: Credentials = authenticator.refresh_credentials()

    if credentials:
        print(f"Successfully obtained access token: {credentials.access_token[:10]}...")
        print(f"Refresh token available: {credentials.refresh_token is not None}")
        print(f"Token expires in: {credentials.expires_in} seconds")

        # Example of using the access token in a request
        # session = requests.Session()
        # session.headers.update({"Authorization": f"Bearer {credentials.access_token}"})
        # response = session.get("https://your-api.com/data")
        # print(response.json())
    else:
        print("Failed to obtain credentials.")

except Exception as e:
    print(f"Authentication failed: {e}")

```

## Important Considerations

*   **Auth0 Specifics:** When using Auth0 as an Identity Provider, ensure that the `offline_access` scope is explicitly requested to enable the issuance of a refresh token. Without a refresh token, the system cannot automatically renew access tokens, leading to more frequent interactive logins.
*   **Redirect URI Configuration:** The `redirect_uri` configured in the `ClientConfigStore` must precisely match the redirect URI registered with your Identity Provider. Mismatches will prevent the authorization code from being delivered to the local callback server, causing the authentication flow to fail.
*   **Browser Interaction Requirement:** This authentication flow inherently requires a graphical environment capable of opening a web browser. It is not suitable for headless environments or servers without browser access.
*   **Error Handling:** The `PKCEAuthenticator` handles `AccessTokenNotFoundError` by attempting a full re-authentication. Developers should be aware of potential network issues or IdP-specific errors that might prevent successful token acquisition or refresh.
<!--
key: summary_interactive_web-based_authentication_(pkce_flow)_3f0137c8-43af-47ee-8d20-90669819b732
type: summary_end

-->
<!--
code_unit: flytekit.examples.authentication.pkce_flow
code_unit_type: class
help_text: ''
key: example_0e5d0b90-57ce-439b-8fe1-dd0d1aef26ac
type: example

-->