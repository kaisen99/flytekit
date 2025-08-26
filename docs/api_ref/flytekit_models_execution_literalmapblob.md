# LiteralMapBlob

This class represents a blob of literal map data. It encapsulates a collection of values and a URI. The class provides methods to convert to and from Flyte IDL representations.

## Attributes

- **values**: flytekit.models.literals.LiteralMap = None
  - This is a LiteralMapBlob.

- **uri**: Text = None
  - This is a LiteralMapBlob.

## Constructors
def LiteralMapBlob(values: flytekit.models.literals.LiteralMap = None, uri: Text = None)
-  Initializes a LiteralMapBlob object.
- **Parameters**

  - **values**: flytekit.models.literals.LiteralMap
    - The LiteralMap values.
  - **uri**: Text
    - The URI of the blob.

def LiteralMapBlob(values: flytekit.models.literals.LiteralMap = None, uri: Text = None)
-  Initializes a new instance of the LiteralMapBlob class.
- **Parameters**

  - **values**: flytekit.models.literals.LiteralMap
    - The LiteralMap values.
  - **uri**: Text
    - The URI.



## Methods
```@classmethod
def values()
```
-  Gets the LiteralMap values.

- **Return Value**:
**flytekit.models.literals.LiteralMap**
  - The LiteralMap values.
```@classmethod
def uri()
```
-  Gets the URI.

- **Return Value**:
**Text**
  - The URI.
```@classmethod
def to_flyte_idl()
```
-  Converts the LiteralMapBlob to its Flyte IDL representation.

- **Return Value**:
**flyteidl.admin.execution_pb2.LiteralMapBlob**
  - The Flyte IDL representation of the LiteralMapBlob.
@classmethod
def from_flyte_idl(pb: flyteidl.admin.execution_pb2.LiteralMapBlob) - > [LiteralMapBlob](flytekit_models_execution_literalmapblob)
-  Creates a LiteralMapBlob instance from its Flyte IDL representation.
- **Parameters**

  - **pb**: flyteidl.admin.execution_pb2.LiteralMapBlob
    - The Flyte IDL representation of the LiteralMapBlob.

- **Return Value**:
**[LiteralMapBlob](flytekit_models_execution_literalmapblob)**
  - A LiteralMapBlob instance.
