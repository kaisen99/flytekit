# Execution

This class represents the execution of a workflow within a system. It encapsulates the execution&#x27;s identifier, specification, and closure, providing access to its state and related metadata. The class facilitates the conversion to and from a Flyte IDL representation for serialization and deserialization purposes.

## Attributes

- **id**: flytekit.models.core.identifier.WorkflowExecutionIdentifier
  - WorkflowExecutionIdentifier

- **spec**: [ExecutionSpec](flytekit_models_execution_executionspec)
  - ExecutionSpec

- **closure**: [ExecutionClosure](flytekit_models_execution_executionclosure)
  - ExecutionClosure

## Constructors
def Execution(id: flytekit.models.core.identifier.WorkflowExecutionIdentifier, spec: [ExecutionSpec](flytekit_models_execution_executionspec), closure: [ExecutionClosure](flytekit_models_execution_executionclosure))
-  Initializes an Execution object.
- **Parameters**

  - **id**: flytekit.models.core.identifier.WorkflowExecutionIdentifier
    - The unique identifier for the workflow execution.
  - **spec**: [ExecutionSpec](flytekit_models_execution_executionspec)
    - The specification for the execution.
  - **closure**: [ExecutionClosure](flytekit_models_execution_executionclosure)
    - The closure information for the execution.

def Execution(id: flytekit.models.core.identifier.WorkflowExecutionIdentifier, spec: [ExecutionSpec](flytekit_models_execution_executionspec), closure: [ExecutionClosure](flytekit_models_execution_executionclosure))
-  Initializes an Execution object.
- **Parameters**

  - **id**: flytekit.models.core.identifier.WorkflowExecutionIdentifier
    - The unique identifier for the workflow execution.
  - **spec**: [ExecutionSpec](flytekit_models_execution_executionspec)
    - The specification of the execution.
  - **closure**: [ExecutionClosure](flytekit_models_execution_executionclosure)
    - The closure of the execution.



## Methods
```@classmethod
def id()
```
-  Returns the unique identifier for the workflow execution.

- **Return Value**:
**flytekit.models.core.identifier.WorkflowExecutionIdentifier**
  - The unique identifier for the workflow execution.
```@classmethod
def closure()
```
-  Returns the closure of the execution.

- **Return Value**:
**[ExecutionClosure](flytekit_models_execution_executionclosure)**
  - The closure of the execution.
```@classmethod
def spec()
```
-  Returns the specification of the execution.

- **Return Value**:
**[ExecutionSpec](flytekit_models_execution_executionspec)**
  - The specification of the execution.
```@classmethod
def to_flyte_idl()
```
-  Converts the Execution object to its Flyte IDL representation.

- **Return Value**:
**flyteidl.admin.execution_pb2.Execution**
  - The Flyte IDL representation of the Execution object.
@classmethod
def from_flyte_idl(pb: flyteidl.admin.execution_pb2.Execution) - > [Execution](flytekit_models_execution_execution)
-  Creates an Execution object from its Flyte IDL representation.
- **Parameters**

  - **pb**: flyteidl.admin.execution_pb2.Execution
    - The Flyte IDL representation of an Execution object.

- **Return Value**:
**[Execution](flytekit_models_execution_execution)**
  - An Execution object created from the Flyte IDL representation.
