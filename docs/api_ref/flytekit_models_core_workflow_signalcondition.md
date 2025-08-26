# SignalCondition

This class defines a condition based on a signal from a user. It encapsulates the signal&#x27;s identifier, data type, and the name of the output variable. The class facilitates the conversion to and from a Flyte IDL representation for interoperability.

## Attributes

- **signal_id**: str
  - The node id of the signal, also the signal name.

- **type**: type_models.LiteralType
  - The type of the signal.

- **output_variable_name**: str
  - The name of the output variable for the signal.

## Constructors
def SignalCondition(signal_id: string, type: type_models.LiteralType, output_variable_name: string)
-  Represents a dependency on an signal from a user.
- **Parameters**

  - **signal_id**: string
    - The node id of the signal, also the signal name.
  - **type**: type_models.LiteralType
    - 
  - **output_variable_name**: string
    - 



## Methods
```@classmethod
def signal_id()
```
-  Represents a dependency on an signal from a user.

        :param signal_id: The node id of the signal, also the signal name.
        :param type: 

- **Return Value**:
**string**
  - The node id of the signal, also the signal name.
```@classmethod
def type()
```
-  Represents a dependency on an signal from a user.

        :param signal_id: The node id of the signal, also the signal name.
        :param type: 

- **Return Value**:
**type_models.LiteralType**
```@classmethod
def output_variable_name()
```
-  Represents a dependency on an signal from a user.

        :param signal_id: The node id of the signal, also the signal name.
        :param type: 

- **Return Value**:
**string**
```@classmethod
def to_flyte_idl()
```
-  Represents a dependency on an signal from a user.

        :param signal_id: The node id of the signal, also the signal name.
        :param type: 

- **Return Value**:
**_core_workflow.SignalCondition**
@classmethod
def from_flyte_idl(pb2_object: _core_workflow.SignalCondition) - > [SignalCondition](flytekit_models_core_workflow_signalcondition)
-  Represents a dependency on an signal from a user.

        :param signal_id: The node id of the signal, also the signal name.
        :param type: 
- **Parameters**

  - **pb2_object**: _core_workflow.SignalCondition
    - Represents a dependency on an signal from a user.

        :param signal_id: The node id of the signal, also the signal name.
        :param type: 

- **Return Value**:
**[SignalCondition](flytekit_models_core_workflow_signalcondition)**
