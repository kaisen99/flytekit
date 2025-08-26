# LaunchPlanClosure

This class encapsulates the state and interface definitions for a Flyte Launch Plan. It stores the current state, expected inputs, and expected outputs of a launch plan. The class provides methods for converting to and from Flyte IDL representations, facilitating serialization and deserialization.

## Attributes

- **state**: [LaunchPlanState](flytekit_models_launch_plan_launchplanstate)
  - Indicate the Launch plan phase

- **expected_inputs**: flytekit.models.interface.ParameterMap
  - Indicates the set of inputs to execute the Launch plan

- **expected_outputs**: flytekit.models.interface.VariableMap
  - Indicates the set of outputs from the Launch plan

## Constructors
def LaunchPlanClosure(state: [LaunchPlanState](flytekit_models_launch_plan_launchplanstate), expected_inputs: flytekit.models.interface.ParameterMap, expected_outputs: flytekit.models.interface.VariableMap)
-  Initialize LaunchPlanClosure
- **Parameters**

  - **state**: [LaunchPlanState](flytekit_models_launch_plan_launchplanstate)
    - Indicate the Launch plan phase
  - **expected_inputs**: flytekit.models.interface.ParameterMap
    - Indicates the set of inputs to execute the Launch plan
  - **expected_outputs**: flytekit.models.interface.VariableMap
    - Indicates the set of outputs from the Launch plan



## Methods
```@classmethod
def state()
```
-  Indicate the Launch plan phase

- **Return Value**:
**[LaunchPlanState](flytekit_models_launch_plan_launchplanstate)**
  - Indicate the Launch plan phase
```@classmethod
def expected_inputs()
```
-  Indicates the set of inputs to execute the Launch plan

- **Return Value**:
**flytekit.models.interface.ParameterMap**
  - Indicates the set of inputs to execute the Launch plan
```@classmethod
def expected_outputs()
```
-  Indicates the set of outputs from the Launch plan

- **Return Value**:
**flytekit.models.interface.VariableMap**
  - Indicates the set of outputs from the Launch plan
```@classmethod
def to_flyte_idl()
```
-  Converts the LaunchPlanClosure object to its corresponding flyte IDL protobuf object.

- **Return Value**:
**flyteidl.admin.launch_plan_pb2.LaunchPlanClosure**
  - The flyte IDL protobuf representation of the LaunchPlanClosure object.
@classmethod
def from_flyte_idl(pb2_object: flyteidl.admin.launch_plan_pb2.LaunchPlanClosure) - > [LaunchPlanClosure](flytekit_models_launch_plan_launchplanclosure)
-  Creates a LaunchPlanClosure object from a flyte IDL protobuf object.
- **Parameters**

  - **pb2_object**: flyteidl.admin.launch_plan_pb2.LaunchPlanClosure
    - The flyte IDL protobuf object to convert from.

- **Return Value**:
**[LaunchPlanClosure](flytekit_models_launch_plan_launchplanclosure)**
  - A LaunchPlanClosure object created from the protobuf object.
