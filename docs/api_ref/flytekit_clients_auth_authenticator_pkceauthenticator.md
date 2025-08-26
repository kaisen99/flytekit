# PKCEAuthenticator

This class implements the Proof Key for Code Exchange (PKCE) authentication flow, handling the entire process and automatically opening a browser for user login. It leverages an underlying authorization client to manage the interaction with the authentication server. The class supports refreshing credentials and stores them using a KeyringStore.

## Attributes

- **endpoint**: str
  - The endpoint of the PKCE flow.

- **cfg_store**: [ClientConfigStore](flytekit_clients_auth_authenticator_clientconfigstore)
  - The client configuration store.

- **scopes**: typing.Optional[typing.List[str]] = None
  - A list of scopes for the authentication.

- **header_key**: typing.Optional[str] = None
  - The header key for authentication.

- **verify**: typing.Optional[typing.Union[bool, str]] = None
  - Whether to verify the SSL certificate.

- **session**: typing.Optional[requests.Session] = None
  - The requests session to use.

## Constructors
def PKCEAuthenticator(endpoint: str, cfg_store: [ClientConfigStore](flytekit_clients_auth_authenticator_clientconfigstore), scopes: typing.Optional[typing.List[str]] = None, header_key: typing.Optional[str] = None, verify: typing.Optional[typing.Union[bool, str]] = None, session: typing.Optional[requests.Session] = None)
-  Initialize with default creds from KeyStore using the endpoint name
- **Parameters**

  - **endpoint**: str
    - The endpoint of the authentication server.
  - **cfg_store**: [ClientConfigStore](flytekit_clients_auth_authenticator_clientconfigstore)
    - A store for client configurations.
  - **scopes**: typing.Optional[typing.List[str]]
    - A list of scopes to request.
  - **header_key**: typing.Optional[str]
    - The key for the authentication header.
  - **verify**: typing.Optional[typing.Union[bool, str]]
    - Controls SSL certificate verification.
  - **session**: typing.Optional[requests.Session]
    - An optional requests session object.



## Methods
```@classmethod
def refresh_credentials()
```
-  Refreshes the access token using the AuthorizationClient. If a valid access token exists, it attempts to refresh it. If refreshing fails or no token exists, it initiates a new authorization flow to obtain fresh credentials. The refreshed or newly obtained credentials are then stored.

