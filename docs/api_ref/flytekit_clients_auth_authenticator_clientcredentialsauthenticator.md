# ClientCredentialsAuthenticator

This class authenticates clients using their client ID and client secret. It retrieves and manages access tokens for secure API interactions. It implements the Authenticator interface and relies on a ClientConfigStore for configuration.

## Attributes

- **endpoint**: str
  - The endpoint to authenticate against.

- **client_id**: str
  - The client ID for authentication.

- **client_secret**: str
  - The client secret for authentication.

- **cfg_store**: [ClientConfigStore](flytekit_clients_auth_authenticator_clientconfigstore)
  - A store for client configuration.

- **header_key**: typing.Optional[str] = None
  - The key for the authorization header.

- **scopes**: typing.Optional[typing.List[str]] = None
  - The scopes for the authentication.

- **http_proxy_url**: typing.Optional[str] = None
  - The URL for the HTTP proxy.

- **verify**: typing.Optional[typing.Union[bool, str]] = None
  - Whether to verify the SSL certificate.

- **audience**: typing.Optional[str] = None
  - The audience for the authentication.

- **session**: typing.Optional[requests.Session] = None
  - The requests session to use.

## Constructors
def ClientCredentialsAuthenticator(endpoint: string, client_id: string, client_secret: string, cfg_store: [ClientConfigStore](flytekit_clients_auth_authenticator_clientconfigstore), header_key: typing.Optional[str] = None, scopes: typing.Optional[typing.List[str]] = None, http_proxy_url: typing.Optional[str] = None, verify: typing.Optional[typing.Union[bool, str]] = None, audience: typing.Optional[str] = None, session: typing.Optional[requests.Session] = None)
-  This Authenticator uses ClientId and ClientSecret to authenticate
- **Parameters**

  - **endpoint**: string
    - The endpoint to authenticate against.
  - **client_id**: string
    - The client ID for authentication.
  - **client_secret**: string
    - The client secret for authentication.
  - **cfg_store**: [ClientConfigStore](flytekit_clients_auth_authenticator_clientconfigstore)
    - A store for client configurations.
  - **header_key**: typing.Optional[str]
    - An optional header key for authentication.
  - **scopes**: typing.Optional[typing.List[str]]
    - An optional list of scopes for authentication.
  - **http_proxy_url**: typing.Optional[str]
    - An optional HTTP proxy URL.
  - **verify**: typing.Optional[typing.Union[bool, str]]
    - An optional value to control SSL certificate verification.
  - **audience**: typing.Optional[str]
    - An optional audience for token requests.
  - **session**: typing.Optional[requests.Session]
    - An optional requests session object.



## Methods
```@classmethod
def refresh_credentials()
```
-  This function is used by the _handle_rpc_error() decorator, depending on the AUTH_MODE config object. This handler is meant for SDK use-cases of auth (like pyflyte, or when users call SDK functions that require access to Admin, like when waiting for another workflow to complete from within a task). This function uses basic auth, which means the credentials for basic auth must be present from wherever this code is running.

