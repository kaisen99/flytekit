# StaticClientConfigStore

This class provides a static configuration store for client configurations. It stores a pre-defined client configuration and offers a method to retrieve it. The class implements the ClientConfigStore interface, ensuring a consistent way to access client settings.

## Attributes

- **_cfg**: [ClientConfig](flytekit_clients_auth_authenticator_clientconfig)
  - The client configuration object.

## Constructors
def StaticClientConfigStore(cfg: [ClientConfig](flytekit_clients_auth_authenticator_clientconfig))
-  Initializes the StaticClientConfigStore with a ClientConfig object.
- **Parameters**

  - **cfg**: [ClientConfig](flytekit_clients_auth_authenticator_clientconfig)
    - The ClientConfig object to store.



## Methods
```@classmethod
def get_client_config()
```
-  Returns the client configuration.

- **Return Value**:
**[ClientConfig](flytekit_clients_auth_authenticator_clientconfig)**
  - The client configuration.
