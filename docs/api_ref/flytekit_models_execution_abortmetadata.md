# AbortMetadata

This class encapsulates metadata related to the abortion of an execution. It provides access to the cause and principal responsible for the abortion. The class facilitates conversion to and from Flyte IDL representation for interoperability.

## Attributes

- **cause**: string
  - cause of the abort

- **principal**: string
  - principal that caused the abort

## Constructors
def AbortMetadata(cause: string, principal: string)
-  Initializes an AbortMetadata object.
- **Parameters**

  - **cause**: string
    - The reason for the abort.
  - **principal**: string
    - The entity that initiated the abort.



## Methods
```@classmethod
def cause()
```
-  Returns the cause of the abort.

- **Return Value**:
**string**
  - The cause of the abort.
```@classmethod
def principal()
```
-  Returns the principal who caused the abort.

- **Return Value**:
**string**
  - The principal who caused the abort.
```@classmethod
def to_flyte_idl()
```
-  Converts the AbortMetadata object to its Flyte IDL representation.

- **Return Value**:
**flyteidl.admin.execution_pb2.AbortMetadata**
  - The Flyte IDL representation of the AbortMetadata object.
@classmethod
def from_flyte_idl(pb2_object: flyteidl.admin.execution_pb2.AbortMetadata) - > [AbortMetadata](flytekit_models_execution_abortmetadata)
-  Creates an AbortMetadata object from its Flyte IDL representation.
- **Parameters**

  - **pb2_object**: flyteidl.admin.execution_pb2.AbortMetadata
    - The Flyte IDL representation of the AbortMetadata object.

- **Return Value**:
**[AbortMetadata](flytekit_models_execution_abortmetadata)**
  - An AbortMetadata object created from the Flyte IDL representation.
