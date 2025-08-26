# RuntimeMetadata

This class encapsulates runtime metadata, providing information about the execution environment. It stores the runtime type, version, and an optional flavor identifier. The class facilitates conversion to and from Flyte IDL objects for interoperability.

## Attributes

- **type**: int
  - Enum type from RuntimeMetadata.RuntimeType

- **version**: string
  - Version string for SDK version. Can be used for metrics or managing breaking changes in Admin or Propeller

- **flavor**: string
  - Optional extra information about runtime environment (e.g. Python, GoLang, etc.)

## Constructors
def RuntimeMetadata(type: int, version: str, flavor: str)
-  Initializes a new instance of the RuntimeMetadata class.
- **Parameters**

  - **type**: int
    - Enum type from RuntimeMetadata.RuntimeType
  - **version**: str
    - Version string for SDK version. Can be used for metrics or managing breaking changes in Admin or Propeller
  - **flavor**: str
    - Optional extra information about runtime environment (e.g. Python, GoLang, etc.)



## Methods
```@classmethod
def type()
```
-  Enum type from RuntimeMetadata.RuntimeType

- **Return Value**:
**int**
  - Enum type from RuntimeMetadata.RuntimeType
```@classmethod
def version()
```
-  Version string for SDK version. Can be used for metrics or managing breaking changes in Admin or Propeller

- **Return Value**:
**Text**
  - Version string for SDK version. Can be used for metrics or managing breaking changes in Admin or Propeller
```@classmethod
def flavor()
```
-  Optional extra information about runtime environment (e.g. Python, GoLang, etc.)

- **Return Value**:
**Text**
  - Optional extra information about runtime environment (e.g. Python, GoLang, etc.)
```@classmethod
def to_flyte_idl()
```
-  Converts the RuntimeMetadata object to its corresponding Flyte IDL protobuf object.

- **Return Value**:
**flyteidl.core.tasks_pb2.RuntimeMetadata**
  - The Flyte IDL protobuf representation of the RuntimeMetadata.
@classmethod
def from_flyte_idl(pb2_object: flyteidl.core.tasks_pb2.RuntimeMetadata) - > [RuntimeMetadata](flytekit_models_task_runtimemetadata)
-  Creates a RuntimeMetadata object from its Flyte IDL protobuf representation.
- **Parameters**

  - **pb2_object**: flyteidl.core.tasks_pb2.RuntimeMetadata
    - The Flyte IDL protobuf object to convert from.

- **Return Value**:
**[RuntimeMetadata](flytekit_models_task_runtimemetadata)**
  - A RuntimeMetadata object created from the protobuf object.
