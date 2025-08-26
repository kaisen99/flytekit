# HasFlyteInterface

This class defines a protocol for objects that interact with the Flyte platform. It provides a standardized interface for accessing the object&#x27;s name and typed interface. It also enables the construction of node metadata for workflow execution.

## Attributes

- **name**: str
  - The name of the interface.

- **interface**: _interface_models.TypedInterface
  - The typed interface model.



## Methods
```@classmethod
def name()
```
-  The name of the interface.

- **Return Value**:
**string**
  - The name of the interface.
```@classmethod
def interface()
```
-  The typed interface of the Flyte interface.

- **Return Value**:
**object**
  - The typed interface of the Flyte interface.
```@classmethod
def construct_node_metadata()
```
-  Constructs node metadata for the Flyte interface.

- **Return Value**:
**object**
  - The constructed node metadata.
