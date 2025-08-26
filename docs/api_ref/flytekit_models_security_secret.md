# Secret

This class represents a secret, providing a mechanism to securely inject sensitive information into a runtime environment. It supports different mount types, such as environment variables or files, offering flexibility in how secrets are accessed. The class includes methods for converting to and from Flyte IDL objects, facilitating serialization and deserialization.

## Attributes

- **group**: Optional[str] = None
  - Name of the secret. For example in kubernetes secrets is the name of the secret

- **key**: Optional[str] = None
  - optional and can be an individual secret identifier within the secret For k8s this is required

- **group_version**: Optional[str] = None
  - the version of the secret. This is an optional field

- **mount_requirement**: MountType = MountType.ANY
  - provides a hint to the system as to how the secret should be injected

- **env_var**: Optional[str] = None
  - optional. Custom environment name to set the value of the secret. If mount_requirement is ENV_VAR, then the value is the secret itself. If mount_requirement is FILE, then the value is the path to the secret file.

## Constructors
def Secret(group: Optional[str] = None, key: Optional[str] = None, version: Optional[str] = None, mount_requirement: MountType = MountType.ANY, env_var: Optional[str] = None)
-  See :std:ref:`cookbook:secrets` for usage examples.

Args:
    group is the Name of the secret. For example in kubernetes secrets is the name of the secret
    key is optional and can be an individual secret identifier within the secret For k8s this is required
    version is the version of the secret. This is an optional field
    mount_requirement provides a hint to the system as to how the secret should be injected
    env_var is optional. Custom environment name to set the value of the secret.
        If mount_requirement is ENV_VAR, then the value is the secret itself.
        If mount_requirement is FILE, then the value is the path to the secret file.
- **Parameters**

  - **group**: Optional[str]
    - is the Name of the secret. For example in kubernetes secrets is the name of the secret
  - **key**: Optional[str]
    - is optional and can be an individual secret identifier within the secret For k8s this is required
  - **version**: Optional[str]
    - is the version of the secret. This is an optional field
  - **mount_requirement**: MountType
    - provides a hint to the system as to how the secret should be injected
  - **env_var**: Optional[str]
    - is optional. Custom environment name to set the value of the secret.
            If mount_requirement is ENV_VAR, then the value is the secret itself.
            If mount_requirement is FILE, then the value is the path to the secret file.



## Methods
```@classmethod
def to_flyte_idl()
```
-  Converts the Secret object into its corresponding Flyte IDL (Protocol Buffer) representation.

- **Return Value**:
**_sec.Secret**
  - The Flyte IDL Secret object.
@classmethod
def from_flyte_idl(pb2_object: _sec.Secret) - > [Secret](flytekit_models_security_secret)
-  Creates a Secret object from a Flyte IDL (Protocol Buffer) object.
- **Parameters**

  - **pb2_object**: _sec.Secret
    - The Flyte IDL Secret object to convert from.

- **Return Value**:
**[Secret](flytekit_models_security_secret)**
  - A new Secret object initialized from the IDL object.
