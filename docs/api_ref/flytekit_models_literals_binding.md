# Binding

This class represents a binding between a variable and either a static value or a node output. It facilitates the mapping of variable names to their corresponding data. The class utilizes BindingData to handle the actual data binding.

## Attributes

- **var**: string
  - A variable name, must match an input or output variable of the node.

- **binding**: [BindingData](flytekit_models_literals_bindingdata)
  - Data to use to bind this variable.

## Constructors
def Binding(var: Text, binding: [BindingData](flytekit_models_literals_bindingdata))
-  An input/output binding of a variable to either static value or a node output.
- **Parameters**

  - **var**: Text
    - A variable name, must match an input or output variable of the node.
  - **binding**: [BindingData](flytekit_models_literals_bindingdata)
    - Data to use to bind this variable.

def Binding(var: Text, binding: [BindingData](flytekit_models_literals_bindingdata))
-  An input/output binding of a variable to either static value or a node output.
- **Parameters**

  - **var**: Text
    - A variable name, must match an input or output variable of the node.
  - **binding**: [BindingData](flytekit_models_literals_bindingdata)
    - Data to use to bind this variable.



## Methods
```@classmethod
def var()
```
-  A variable name, must match an input or output variable of the node.

- **Return Value**:
**Text**
```@classmethod
def binding()
```
-  Data to use to bind this variable.

- **Return Value**:
**[BindingData](flytekit_models_literals_bindingdata)**
```@classmethod
def to_flyte_idl()
```
-  Converts the Binding object to its corresponding Flyte IDL protobuf object.

- **Return Value**:
**flyteidl.core.literals_pb2.Binding**
@classmethod
def from_flyte_idl(pb2_object: flyteidl.core.literals_pb2.Binding) - > flytekit.core.models.literals.Binding
-  Creates a Binding object from a Flyte IDL protobuf object.
- **Parameters**

  - **pb2_object**: flyteidl.core.literals_pb2.Binding
    - The Flyte IDL Binding protobuf object.

- **Return Value**:
**flytekit.core.models.literals.Binding**
