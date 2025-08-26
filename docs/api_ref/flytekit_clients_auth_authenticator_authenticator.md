# Authenticator

This class serves as a base authenticator, providing a foundation for various authentication flows. It manages credentials and includes methods for fetching authentication metadata. The class supports HTTP proxy configuration and verification settings, with an abstract method for refreshing credentials.

## Attributes

- **endpoint**: str
  - The endpoint for the authentication.

- **header_key**: str = &quot;authorization&quot;
  - The key for the authorization header.

- **credentials**: [Credentials](flytekit_configuration_internal_credentials)
  - The credentials to use for authentication.

- **http_proxy_url**: typing.Optional[str]
  - The URL for the HTTP proxy.

- **verify**: typing.Optional[typing.Union[bool, str]]
  - Whether to verify the SSL certificate.

## Constructors
def Authenticator(endpoint: str, header_key: str, credentials: [Credentials](flytekit_configuration_internal_credentials) = None, http_proxy_url: typing.Optional[str] = None, verify: typing.Optional[typing.Union[bool, str]] = None)
-  Base authenticator for all authentication flows
- **Parameters**

  - **endpoint**: str
    - The endpoint for the authentication.
  - **header_key**: str
    - The key for the header, defaults to &#x27;authorization&#x27;.
  - **credentials**: [Credentials](flytekit_configuration_internal_credentials)
    - The credentials to use for authentication.
  - **http_proxy_url**: typing.Optional[str]
    - The URL for an HTTP proxy.
  - **verify**: typing.Optional[typing.Union[bool, str]]
    - Specifies whether to verify the SSL certificate.



## Methods
```@classmethod
def get_credentials()
```
-  Returns the current credentials.

- **Return Value**:
**[Credentials](flytekit_configuration_internal_credentials)**
  - The current credentials.
```@classmethod
def fetch_grpc_call_auth_metadata()
```
-  Fetches the authentication metadata for a gRPC call.

- **Return Value**:
**Optional[Tuple[str, str]]**
  - A tuple containing the header key and the authentication token, or None if no credentials are set.
```@classmethod
def refresh_credentials()
```
-  Refreshes the authentication credentials. This is an abstract method and must be implemented by subclasses.

- **Return Value**:
**None**
  - None
