# ClientConfig

This class encapsulates the configuration parameters required for authenticating a client. It provides access to essential settings such as endpoints, client identifiers, and scopes. The configuration facilitates secure communication and authorization processes within the client application.

## Attributes

- **token_endpoint**: str
  - The token endpoint URL for the OAuth 2.0 flow.

- **authorization_endpoint**: str
  - The authorization endpoint URL for the OAuth 2.0 flow.

- **redirect_uri**: str
  - The redirect URI for the OAuth 2.0 flow.

- **client_id**: str
  - The client ID for the OAuth 2.0 client.

- **device_authorization_endpoint**: typing.Optional[str] = None
  - The device authorization endpoint URL for the OAuth 2.0 device flow.

- **scopes**: typing.List[str] = None
  - A list of scopes to request for the OAuth 2.0 token.

- **header_key**: str = authorization
  - The key to use for the authorization header.

- **audience**: typing.Optional[str] = None
  - The audience for the OAuth 2.0 token.

## Constructors
def ClientConfig(token_endpoint: str, authorization_endpoint: str, redirect_uri: str, client_id: str, device_authorization_endpoint: typing.Optional[str] = None, scopes: typing.List[str] = None, header_key: str = &quot;authorization&quot;, audience: typing.Optional[str] = None)
-  Client Configuration that is needed by the authenticator
- **Parameters**

  - **token_endpoint**: str
    - The token endpoint
  - **authorization_endpoint**: str
    - The authorization endpoint
  - **redirect_uri**: str
    - The redirect URI
  - **client_id**: str
    - The client ID
  - **device_authorization_endpoint**: typing.Optional[str]
    - The device authorization endpoint
  - **scopes**: typing.List[str]
    - The scopes
  - **header_key**: str
    - The header key
  - **audience**: typing.Optional[str]
    - The audience



