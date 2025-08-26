# OutputReference

This class represents a reference to an output produced by a node within a workflow. It allows access to the output of a node by specifying the node ID and the variable name. The class includes properties for accessing the node ID, variable name, and attribute path, and it provides methods for converting to and from Flyte IDL objects.

## Attributes

- **node_id**: string
  - Node id must exist at the graph layer.

- **var**: string
  - Variable name must refer to an output variable for the node.

- **attr_path**: list[union[str, int]] = []
  - The attribute path the promise will be resolved with.

## Constructors
def OutputReference(node_id: Text, var: Text, attr_path: List[[Union](flytekit_models_literals_union)[str, int]] = [])
-  A reference to an output produced by a node. The type can be retrieved -and validated- from
            the underlying interface of the node.
- **Parameters**

  - **node_id**: Text
    - Node id must exist at the graph layer.
  - **var**: Text
    - Variable name must refer to an output variable for the node.
  - **attr_path**: List[[Union](flytekit_models_literals_union)[str, int]]
    - The attribute path the promise will be resolved with.

def OutputReference(node_id: Text, var: Text, attr_path: typing.List[typing.Union[str, int]] = [])
-  A reference to an output produced by a node. The type can be retrieved -and validated- from the underlying interface of the node.
- **Parameters**

  - **node_id**: Text
    - Node id must exist at the graph layer.
  - **var**: Text
    - Variable name must refer to an output variable for the node.
  - **attr_path**: typing.List[typing.Union[str, int]]
    - The attribute path the promise will be resolved with.



## Methods
```@classmethod
def node_id()
```
-  Node id must exist at the graph layer.

- **Return Value**:
**Text**
  - Node id must exist at the graph layer.
```@classmethod
def var()
```
-  Variable name must refer to an output variable for the node.

- **Return Value**:
**Text**
  - Variable name must refer to an output variable for the node.
```@classmethod
def attr_path()
```
-  The attribute path the promise will be resolved with.

- **Return Value**:
**list[union[str, int]]**
  - The attribute path the promise will be resolved with.
@classmethod
def var(var_name: str)
-  Updates the variable name.
- **Parameters**

  - **var_name**: str
    - The new variable name.

```@classmethod
def to_flyte_idl()
```
-  Converts the OutputReference object to its Flyte IDL representation.

- **Return Value**:
**flyteidl.core.types.OutputReference**
  - The Flyte IDL representation of the OutputReference.
@classmethod
def from_flyte_idl(pb2_object: flyteidl.core.types.OutputReference) - > [OutputReference](flytekit_models_types_outputreference)
-  Creates an OutputReference object from its Flyte IDL representation.
- **Parameters**

  - **pb2_object**: flyteidl.core.types.OutputReference
    - The Flyte IDL OutputReference object.

- **Return Value**:
**[OutputReference](flytekit_models_types_outputreference)**
  - An OutputReference object.
