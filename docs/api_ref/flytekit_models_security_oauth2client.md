# OAuth2Client

This class represents an OAuth2 client, encapsulating the client ID and secret. It facilitates the conversion to and from Flyte IDL&#x27;s OAuth2Client representation. The class provides methods for serializing and deserializing OAuth2 client information.

## Attributes

- **client_id**: string
  - client_id

- **client_secret**: string
  - client_secret



## Methods
```@classmethod
def to_flyte_idl()
```
-  Converts the OAuth2Client object to its corresponding Flyte IDL representation.

- **Return Value**:
**_sec.OAuth2Client**
  - The Flyte IDL representation of the OAuth2Client object.
@classmethod
def from_flyte_idl(pb2_object: _sec.OAuth2Client) - > [OAuth2Client](flytekit_models_security_oauth2client)
-  Creates an OAuth2Client object from its Flyte IDL representation.
- **Parameters**

  - **pb2_object**: _sec.OAuth2Client
    - The Flyte IDL representation of an OAuth2Client object.

- **Return Value**:
**[OAuth2Client](flytekit_models_security_oauth2client)**
  - An OAuth2Client object.
