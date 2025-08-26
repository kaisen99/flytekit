# ExecutionClusterLabel

This class represents a label used to specify the execution cluster for a task. It encapsulates a string value that identifies the target execution environment. The class provides methods for converting to and from Flyte IDL objects, facilitating its use in serialization and deserialization processes.

## Attributes

- **value**: string
  - Label value to determine where the execution will be run

## Constructors
def ExecutionClusterLabel(value: string)
-  Label value to determine where the execution will be run
- **Parameters**

  - **value**: string
    - Label value to determine where the execution will be run



## Methods
```@classmethod
def value()
```
-  Label value to determine where the execution will be run

- **Return Value**:
**string**
  - Text
```@classmethod
def to_flyte_idl()
```
-  Converts the ExecutionClusterLabel object to its Flyte IDL representation.

- **Return Value**:
**flyteidl.admin.matchable_resource_pb2.ExecutionClusterLabel**
  - flyteidl.admin.matchable_resource_pb2.ExecutionClusterLabel
@classmethod
def from_flyte_idl(pb2_object: flyteidl.admin.matchable_resource_pb2.ExecutionClusterLabel) - > [ExecutionClusterLabel](flytekit_models_matchable_resource_executionclusterlabel)
-  Creates an ExecutionClusterLabel object from its Flyte IDL representation.
- **Parameters**

  - **pb2_object**: flyteidl.admin.matchable_resource_pb2.ExecutionClusterLabel
    - flyteidl.admin.matchable_resource_pb2.ExecutionClusterLabel

- **Return Value**:
**[ExecutionClusterLabel](flytekit_models_matchable_resource_executionclusterlabel)**
  - ExecutionClusterLabel
