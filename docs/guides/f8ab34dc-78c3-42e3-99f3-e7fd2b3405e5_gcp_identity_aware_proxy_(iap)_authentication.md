
<!--
help_text: ''
key: summary_gcp_identity_aware_proxy_(iap)_authentication_191c3140-9412-4c7a-8a7c-944f001ef4a5
modules:
- flytekitplugins.identity_aware_proxy.cli
questions_to_answer: []
type: summary

-->
GCP Identity Aware Proxy (IAP) Authentication

GCP Identity Aware Proxy (IAP) provides a secure way to control access to applications running on Google Cloud. It verifies a user's identity and determines if they are authorized to access the application, allowing you to secure your applications without requiring a VPN.

This section details how to integrate and use the authentication mechanism for IAP-protected resources.

### The `GCPIdentityAwareProxyAuthenticator` Class

The `GCPIdentityAwareProxyAuthenticator` class encapsulates the entire OAuth 2.0 authorization flow required to authenticate with GCP Identity Aware Proxy. It streamlines the process of obtaining and refreshing access tokens, enabling seamless interaction with IAP-secured applications.

**Key Capabilities:**

*   **Automated OAuth 2.0 Flow:** Manages the complete OAuth 2.0 authorization code grant flow, including redirecting users to Google's authentication endpoint and handling the callback.
*   **Browser-Based Login:** Automatically opens a browser window to facilitate user login and consent for the initial authentication.
*   **Token Management:** Handles the exchange of authorization codes for access tokens and manages the refreshing of expired tokens using refresh tokens.
*   **Secure Credential Storage:** Integrates with a secure `KeyringStore` to persist credentials (ID tokens and refresh tokens) across sessions, minimizing the need for repeated user logins.

**Initialization:**

To instantiate the authenticator, provide the following parameters:

*   `audience` (str): The audience for the IAP-protected resource. This is typically the OAuth 2.0 client ID of your IAP application or the backend service's audience.
*   `client_id` (str): Your application's OAuth 2.0 client ID, obtained from the Google Cloud Console.
*   `client_secret` (str): Your application's OAuth 2.0 client secret, obtained from the Google Cloud Console.
*   `verify` (Optional[Union[bool, str]]): Controls SSL certificate verification. Set to `False` to disable, or provide a path to a CA bundle. Defaults to `None` (uses default CA certificates).

**Example Initialization:**

```python
from flytekitplugins.identity_aware_proxy.cli import GCPIdentityAwareProxyAuthenticator

# Replace with your actual values
IAP_AUDIENCE = "your-iap-client-id.apps.googleusercontent.com"
OAUTH_CLIENT_ID = "your-oauth-client-id.apps.googleusercontent.com"
OAUTH_CLIENT_SECRET = "your-oauth-client-secret"

authenticator = GCPIdentityAwareProxyAuthenticator(
    audience=IAP_AUDIENCE,
    client_id=OAUTH_CLIENT_ID,
    client_secret=OAUTH_CLIENT_SECRET,
)
```

### Authentication Flow and Credential Management

The primary method for managing credentials is `refresh_credentials`. This method intelligently handles both initial authentication and subsequent token refreshes.

**`refresh_credentials()` Method:**

This method performs the following actions:

1.  **Attempt Token Refresh:** If existing credentials (an ID token and refresh token) are found in the `KeyringStore`, the authenticator first attempts to refresh the access token using the stored refresh token. This is the most common scenario for subsequent interactions after the initial login.
2.  **Full Authorization Flow:** If no credentials are found, or if the token refresh fails (e.g., due to an expired refresh token or revocation), the authenticator initiates a full OAuth 2.0 authorization flow. This involves:
    *   Opening a browser window to Google's OAuth 2.0 authorization endpoint (`https://accounts.google.com/o/oauth2/v2/auth`).
    *   Prompting the user to log in with their Google account and grant consent to your application.
    *   Listening for the OAuth callback on `http://localhost:4444`.
    *   Exchanging the authorization code for an ID token and refresh token.
3.  **Credential Storage:** Upon successful authentication or refresh, the new credentials are securely stored in the `KeyringStore` associated with the provided `audience`. If a refresh fails and a full flow is initiated, any old, invalid credentials are removed from the `KeyringStore`.

**Usage Example:**

```python
from flytekitplugins.identity_aware_proxy.cli import GCPIdentityAwareProxyAuthenticator

# Initialize the authenticator as shown above
IAP_AUDIENCE = "your-iap-client-id.apps.googleusercontent.com"
OAUTH_CLIENT_ID = "your-oauth-client-id.apps.googleusercontent.com"
OAUTH_CLIENT_SECRET = "your-oauth-client-secret"

authenticator = GCPIdentityAwareProxyAuthenticator(
    audience=IAP_AUDIENCE,
    client_id=OAUTH_CLIENT_ID,
    client_secret=OAUTH_CLIENT_SECRET,
)

try:
    authenticator.refresh_credentials()
    print("Successfully authenticated with GCP IAP.")
    # The authenticator now holds valid credentials internally.
    # These credentials will be used by the underlying client for requests.
except Exception as e:
    print(f"Authentication failed: {e}")
```

Calling `refresh_credentials()` ensures that the authenticator always has valid and up-to-date credentials for making requests to IAP-protected resources.

### Key Considerations and Best Practices

*   **OAuth Client Configuration:**
    *   Ensure your OAuth 2.0 client ID and secret are correctly configured in the Google Cloud Console.
    *   The `redirect_uri` for your OAuth client *must* be set to `http://localhost:4444`. This is a fixed value used by the authenticator to receive the OAuth callback.
    *   The OAuth client type should be "Web application".
*   **Audience Matching:** The `audience` parameter passed to the `GCPIdentityAwareProxyAuthenticator` must precisely match the `audience` configured for your IAP-protected resource in Google Cloud. This is crucial for token validation.
*   **Security of Client Secret:** Treat your `client_secret` as sensitive information. Do not hardcode it directly in production code. Use environment variables, a secure configuration management system, or a secrets manager to provide it at runtime.
*   **Browser Availability:** The initial authentication flow requires a graphical environment where a web browser can be launched. This means the authenticator is best suited for client-side applications or development environments where user interaction is possible. For server-to-server authentication, consider using service accounts or other Google Cloud authentication methods.
*   **Error Handling:** Implement robust error handling around calls to `refresh_credentials()` to gracefully manage scenarios where authentication fails (e.g., network issues, invalid credentials, user denial).
<!--
key: summary_gcp_identity_aware_proxy_(iap)_authentication_191c3140-9412-4c7a-8a7c-944f001ef4a5
type: summary_end

-->
<!--
code_unit: flytekitplugins.identity_aware_proxy.examples.iap_auth_setup
code_unit_type: class
help_text: ''
key: example_af9e2c54-6fb9-4d15-b655-ce356ae3a71e
type: example

-->