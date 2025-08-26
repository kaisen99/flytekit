
<!--
help_text: ''
key: summary_machine-to-machine_authentication_(client_credentials)_b9687b24-234c-4bda-960d-774721c88730
modules:
- flytekit.clients.auth.authenticator.ClientCredentialsAuthenticator
- flytekit.clients.auth.token_client.GrantType
questions_to_answer: []
type: summary

-->
Machine-to-Machine (M2M) authentication using the Client Credentials grant type enables applications to authenticate directly with an authorization server to obtain an access token. This token grants the application access to protected resources without user involvement. This method is ideal for services, daemons, or other non-interactive clients that require secure access to APIs.

### The ClientCredentialsAuthenticator

The `ClientCredentialsAuthenticator` facilitates M2M authentication by leveraging a `client_id` and `client_secret`. This authenticator is designed to manage the lifecycle of access tokens, including their initial acquisition and subsequent refresh.

When initializing the `ClientCredentialsAuthenticator`, provide the following parameters:

*   `endpoint` (str): The base URL of the service or API to authenticate against.
*   `client_id` (str): The unique identifier for the client application.
*   `client_secret` (str): The confidential secret associated with the client ID. Both `client_id` and `client_secret` are mandatory; omitting either raises a `ValueError`.
*   `cfg_store`: An instance of `ClientConfigStore` which provides essential configuration details such as the `token_endpoint`, default `scopes`, and `audience`.
*   `header_key` (Optional[str]): An optional key for the authorization header. If not provided, the authenticator uses the `header_key` from `cfg_store`.
*   `scopes` (Optional[List[str]]): A list of desired permissions (scopes) for the access token. If not provided, the authenticator uses the default scopes from `cfg_store`.
*   `http_proxy_url` (Optional[str]): An optional URL for an HTTP proxy to use for network requests.
*   `verify` (Optional[Union[bool, str]]): Controls SSL certificate verification. Set to `False` to disable verification, or provide a path to a CA bundle.
*   `audience` (Optional[str]): The intended recipient of the access token. If not provided, the authenticator uses the default audience from `cfg_store`.
*   `session` (Optional[requests.Session]): An optional `requests.Session` object to reuse HTTP connections.

The authenticator internally retrieves the `token_endpoint`, `scopes`, and `audience` from the provided `cfg_store`, allowing for centralized configuration management.

#### Authentication Flow

The `refresh_credentials` method handles the core authentication logic. This method is typically invoked automatically when an access token is expired or invalid, or when an initial token is required.

The process involves:

1.  Constructing a Basic Authorization header using the `client_id` and `client_secret`. This header is essential for the OAuth 2.0 Client Credentials flow.
2.  Making a request to the `token_endpoint` (obtained from `cfg_store`) with the Basic Authorization header, specified `scopes`, and `audience`.
3.  The underlying `token_client` utility performs the HTTP request and parses the response.
4.  Upon successful response, the authenticator extracts the access token, refresh token (though not typically used in pure Client Credentials flow for direct token refresh, it might be present), and the token's expiration time.
5.  The newly acquired access token is stored internally for subsequent use in API requests.

This mechanism ensures that the application always operates with a valid access token, transparently handling token renewal.

### Usage and Integration

To integrate Machine-to-Machine authentication into an application, instantiate the `ClientCredentialsAuthenticator` with the necessary credentials and configuration.

```python
import requests
import logging
from typing import List, Optional, Union

# Assume these are defined elsewhere in the codebase
# from .authenticator import Authenticator
# from .config_store import ClientConfigStore
# from .token_client import get_basic_authorization_header, get_token
# from .credentials import Credentials

# For demonstration, mock ClientConfigStore and token_client
class MockClientConfig:
    token_endpoint = "https://your-auth-server.com/oauth/token"
    scopes = ["api:read", "api:write"]
    audience = "https://your-api.com"
    header_key = "Authorization"

class MockClientConfigStore:
    def get_client_config(self):
        return MockClientConfig()

class MockTokenClient:
    @staticmethod
    def get_basic_authorization_header(client_id: str, client_secret: str) -> str:
        import base64
        creds = f"{client_id}:{client_secret}".encode("utf-8")
        return f"Basic {base64.b64encode(creds).decode('utf-8')}"

    @staticmethod
    def get_token(
        token_endpoint: str,
        authorization_header: str,
        http_proxy_url: Optional[str] = None,
        verify: Optional[Union[bool, str]] = None,
        scopes: Optional[List[str]] = None,
        audience: Optional[str] = None,
        session: Optional[requests.Session] = None,
    ):
        # Simulate a successful token response
        print(f"Requesting token from {token_endpoint} with scopes={scopes}, audience={audience}")
        return "mock_access_token_12345", None, 3600 # token, refresh_token, expires_in

# Re-define ClientCredentialsAuthenticator for standalone example
class Authenticator:
    def __init__(self, endpoint: str, header_key: Optional[str] = None, http_proxy_url: Optional[str] = None, verify: Optional[Union[bool, str]] = None):
        self._endpoint = endpoint
        self._header_key = header_key
        self._http_proxy_url = http_proxy_url
        self._verify = verify
        self._creds = None

class Credentials:
    def __init__(self, token: str):
        self.token = token

token_client = MockTokenClient() # Use the mock
logging.basicConfig(level=logging.INFO) # Set logging level

class ClientCredentialsAuthenticator(Authenticator):
    def __init__(
        self,
        endpoint: str,
        client_id: str,
        client_secret: str,
        cfg_store: MockClientConfigStore, # Use mock
        header_key: Optional[str] = None,
        scopes: Optional[List[str]] = None,
        http_proxy_url: Optional[str] = None,
        verify: Optional[Union[bool, str]] = None,
        audience: Optional[str] = None,
        session: Optional[requests.Session] = None,
    ):
        if not client_id or not client_secret:
            raise ValueError("Client ID and Client SECRET both are required.")
        cfg = cfg_store.get_client_config()
        self._token_endpoint = cfg.token_endpoint
        self._scopes = scopes or cfg.scopes
        self._client_id = client_id
        self._client_secret = client_secret
        self._audience = audience or cfg.audience
        self._session = session or requests.Session()
        super().__init__(endpoint, cfg.header_key or header_key, http_proxy_url=http_proxy_url, verify=verify)

    def refresh_credentials(self):
        token_endpoint = self._token_endpoint
        scopes = self._scopes
        audience = self._audience

        logging.debug(f"Basic authorization flow with client id {self._client_id} scope {scopes}")
        authorization_header = token_client.get_basic_authorization_header(self._client_id, self._client_secret)

        token, refresh_token, expires_in = token_client.get_token(
            token_endpoint=token_endpoint,
            authorization_header=authorization_header,
            http_proxy_url=self._http_proxy_url,
            verify=self._verify,
            scopes=scopes,
            audience=audience,
            session=self._session,
        )

        logging.info("Retrieved new token, expires in {}".format(expires_in))
        self._creds = Credentials(token)

# Example Usage
if __name__ == "__main__":
    # Instantiate the configuration store
    config_store = MockClientConfigStore()

    # Instantiate the authenticator
    authenticator = ClientCredentialsAuthenticator(
        endpoint="https://api.example.com",
        client_id="your-client-id",
        client_secret="your-client-secret",
        cfg_store=config_store,
        scopes=["myapi:read", "myapi:write"],
        audience="https://myapi.example.com"
    )

    # Manually refresh credentials (this is often handled internally by a client library)
    authenticator.refresh_credentials()

    # Access the token
    if authenticator._creds:
        print(f"Access Token: {authenticator._creds.token}")
    else:
        print("Failed to retrieve access token.")

    # Example with missing credentials (will raise ValueError)
    try:
        ClientCredentialsAuthenticator(
            endpoint="https://api.example.com",
            client_id="",
            client_secret="your-client-secret",
            cfg_store=config_store
        )
    except ValueError as e:
        print(f"Error: {e}")
```

In a typical application, a client library or SDK integrates this authenticator. The library uses the authenticator to attach the access token to outgoing requests and automatically calls `refresh_credentials` when the current token expires or an authentication error occurs.

### Best Practices and Considerations

*   **Client Secret Security**: Treat `client_secret` as highly sensitive information. Avoid hardcoding it directly in source code. Instead, retrieve it from secure environment variables, a secrets management service, or a secure configuration store.
*   **Token Refresh**: The `refresh_credentials` method is designed to be called when a token is needed or has expired. While the example shows a manual call, client libraries typically abstract this, ensuring tokens are always fresh before making API calls.
*   **Scopes and Audience**: Carefully define the `scopes` and `audience` required by your application. Requesting only the necessary permissions adheres to the principle of least privilege, enhancing security.
*   **HTTP Proxy and SSL Verification**: Configure `http_proxy_url` and `verify` as needed for your network environment. For production systems, always ensure SSL verification is enabled (`verify=True` or a path to a CA bundle) to prevent man-in-the-middle attacks.
*   **Error Handling**: The `refresh_credentials` method relies on the `token_client` to handle network communication and token endpoint responses. Implement robust error handling around calls that depend on the authenticator to catch issues like network failures or invalid credentials.
<!--
key: summary_machine-to-machine_authentication_(client_credentials)_b9687b24-234c-4bda-960d-774721c88730
type: summary_end

-->
<!--
code_unit: flytekit.examples.authentication.client_credentials
code_unit_type: class
help_text: ''
key: example_2f2432fb-80c4-4cca-bbe8-1160c5677525
type: example

-->