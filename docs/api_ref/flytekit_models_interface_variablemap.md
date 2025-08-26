# VariableMap

This class represents a mapping of variables, providing a structured way to manage and access them. It allows for storing and retrieving variables using a dictionary-like interface. The class facilitates conversion to and from a Flyte IDL representation for serialization and deserialization purposes.

## Attributes

- **variables**: dict[Text, [Variable](flytekit_models_interface_variable)]
  - A map of Variables

## Constructors
def VariableMap(variables: dict[Text, [Variable](flytekit_models_interface_variable)])
-  A map of Variables
- **Parameters**

  - **variables**: dict[Text, [Variable](flytekit_models_interface_variable)]
    - A map of Variables

def VariableMap(variables: dict[Text, [Variable](flytekit_models_interface_variable)])
-  A map of Variables
- **Parameters**

  - **variables**: dict[Text, [Variable](flytekit_models_interface_variable)]
    - 



## Methods
```@classmethod
def variables()
```
-  

- **Return Value**:
**dict[Text, [Variable](flytekit_models_interface_variable)]**
```@classmethod
def to_flyte_idl()
```
-  

- **Return Value**:
**dict[Text, [Variable](flytekit_models_interface_variable)]**
@classmethod
def from_flyte_idl(pb2_object: dict[Text, [Variable](flytekit_models_interface_variable)]) - > [VariableMap](flytekit_models_interface_variablemap)
-  
- **Parameters**

  - **pb2_object**: dict[Text, [Variable](flytekit_models_interface_variable)]
    - 

- **Return Value**:
**[VariableMap](flytekit_models_interface_variablemap)**
