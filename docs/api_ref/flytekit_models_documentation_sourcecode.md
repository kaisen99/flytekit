# SourceCode

This class represents a link to the source code associated with a task or workflow definition. It provides a mechanism to store and retrieve the source code&#x27;s location. The class facilitates conversion to and from a protocol buffer representation for serialization and deserialization purposes.

## Attributes

- **link**: Optional[str] = None
  - Link to source code used to define this task or workflow.

## Constructors
def SourceCode(link: Optional[str] = None)
-  Initializes a new instance of the SourceCode class.
- **Parameters**

  - **link**: Optional[str]
    - A link to the source code. Defaults to None.



## Methods
```@classmethod
def to_flyte_idl()
```
-  Converts the SourceCode object to its Flyte IDL representation.

- **Return Value**:
**description_entity_pb2.SourceCode**
  - The Flyte IDL representation of the SourceCode object.
@classmethod
def from_flyte_idl(pb2_object: description_entity_pb2.SourceCode) - > [SourceCode](flytekit_models_documentation_sourcecode)
-  Creates a SourceCode object from its Flyte IDL representation.
- **Parameters**

  - **pb2_object**: description_entity_pb2.SourceCode
    - The Flyte IDL representation of the SourceCode object.

- **Return Value**:
**[SourceCode](flytekit_models_documentation_sourcecode)**
  - A SourceCode object created from the Flyte IDL representation, or None if the link is not present.
