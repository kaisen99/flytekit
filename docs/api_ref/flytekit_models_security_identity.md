# Identity

This class represents an identity configuration, encapsulating information about how a system or process authenticates and authorizes itself. It supports various identity mechanisms, including IAM roles, Kubernetes service accounts, and OAuth2 clients. The class facilitates the conversion to and from a Flyte IDL representation for serialization and deserialization purposes.

## Attributes

- **iam_role**: Optional[str] = None
  - The IAM role associated with the identity.

- **k8s_service_account**: Optional[str] = None
  - The Kubernetes service account associated with the identity.

- **oauth2_client**: Optional[[OAuth2Client](flytekit_models_security_oauth2client)] = None
  - The OAuth2 client configuration for the identity.

- **execution_identity**: Optional[str] = None
  - The execution identity string.

## Constructors
def Identity(iam_role: Optional[str] = None, k8s_service_account: Optional[str] = None, oauth2_client: Optional[[OAuth2Client](flytekit_models_security_oauth2client)] = None, execution_identity: Optional[str] = None)
-  Initializes an Identity object.
- **Parameters**

  - **iam_role**: Optional[str]
    - The IAM role associated with the identity.
  - **k8s_service_account**: Optional[str]
    - The Kubernetes service account associated with the identity.
  - **oauth2_client**: Optional[[OAuth2Client](flytekit_models_security_oauth2client)]
    - The OAuth2 client configuration for the identity.
  - **execution_identity**: Optional[str]
    - The execution identity string.



## Methods
```@classmethod
def to_flyte_idl()
```
-  Converts the Identity object to its Flyte IDL representation.

- **Return Value**:
**_sec.Identity**
  - The Flyte IDL representation of the Identity object.
@classmethod
def from_flyte_idl(pb2_object: _sec.Identity) - > [Identity](flytekit_models_security_identity)
-  Creates an Identity object from its Flyte IDL representation.
- **Parameters**

  - **pb2_object**: _sec.Identity
    - The Flyte IDL representation of the Identity object.

- **Return Value**:
**[Identity](flytekit_models_security_identity)**
  - An Identity object created from the Flyte IDL representation.
