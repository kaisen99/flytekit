# SupportsNodeCreation

This class defines an interface for objects that support node creation within a workflow system. It provides methods to retrieve the node&#x27;s name, Python interface, and construct node metadata. Implementing this interface allows for the creation and management of nodes within a larger workflow execution context.

## Attributes

- **name**: str
  - The name of the node.

- **python_interface**: flyte_interface.Interface
  - The Python interface of the node.



## Methods
```@classmethod
def name()
```
-  The name of the node.

- **Return Value**:
**string**
  - The name of the node.
```@classmethod
def python_interface()
```
-  The Flyte interface of the node.

- **Return Value**:
**flyte_interface.Interface**
  - The Flyte interface of the node.
```@classmethod
def construct_node_metadata()
```
-  Constructs the node metadata.

- **Return Value**:
**_workflow_model.NodeMetadata**
  - The constructed node metadata.
