# Schema

This class represents a strongly typed schema for data retrieved from storage. It defines the interface for accessing data. The class provides properties for accessing the schema&#x27;s URI and type.

## Attributes

- **uri**: Text
  - A strongly typed schema that defines the interface of data retrieved from the underlying storage medium.

- **type**: flytekit.models.types.SchemaType
  - A strongly typed schema that defines the interface of data retrieved from the underlying storage medium.

## Constructors
def Schema(uri: Text, type: flytekit.models.types.SchemaType)
-  A strongly typed schema that defines the interface of data retrieved from the underlying storage medium.
- **Parameters**

  - **uri**: Text
    - 
  - **type**: flytekit.models.types.SchemaType
    - 

def Schema(uri: Text, type: flytekit.models.types.SchemaType)
-  A strongly typed schema that defines the interface of data retrieved from the underlying storage medium.
- **Parameters**

  - **uri**: Text
    - 
  - **type**: flytekit.models.types.SchemaType
    - 



## Methods
```@classmethod
def uri()
```
-  

- **Return Value**:
**Text**
```@classmethod
def type()
```
-  

- **Return Value**:
**flytekit.models.types.SchemaType**
```@classmethod
def to_flyte_idl()
```
-  

- **Return Value**:
**flyteidl.core.literals_pb2.Schema**
@classmethod
def from_flyte_idl(pb2_object: flyteidl.core.literals_pb2.Schema) - > [Schema](flytekit_models_literals_schema)
-  
- **Parameters**

  - **pb2_object**: flyteidl.core.literals_pb2.Schema
    - 

- **Return Value**:
**[Schema](flytekit_models_literals_schema)**
