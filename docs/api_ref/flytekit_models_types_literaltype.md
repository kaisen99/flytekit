# LiteralType

This class represents a literal type within a data processing framework. It encapsulates various data types, including simple types, schemas, collections, and blobs. The class utilizes a oneof pattern, allowing it to represent only one specific type at a time.

## Attributes

- **simple**: [SimpleType](flytekit_models_types_simpletype) = None
  - Enum type from SimpleType

- **schema**: [SchemaType](flytekit_models_types_schematype) = None
  - Type definition for a dataframe-like object.

- **collection_type**: [LiteralType](flytekit_models_types_literaltype) = None
  - For list-like objects, this is the type of each entry in the list.

- **map_value_type**: [LiteralType](flytekit_models_types_literaltype) = None
  - For map objects, this is the type of the value.  The key must always be a
            string.

- **blob**: flytekit.models.core.types.BlobType = None
  - For blob objects, this describes the type.

- **enum_type**: flytekit.models.core.types.EnumType = None
  - For enum objects, describes an enum

- **union_type**: [UnionType](flytekit_models_types_uniontype) = None
  - For union objects, describes an python union type.

- **structured_dataset_type**: flytekit.models.core.types.StructuredDatasetType = None
  - structured dataset

- **metadata**: dict[Text, T] = None
  - Additional data describing the type

- **structure**: flytekit.models.core.types.TypeStructure = None
  - Type matching hints

- **annotation**: flytekit.models.annotation.TypeAnnotation = None
  - Additional data
            describing the type intended to be saturated by the client

## Constructors
def LiteralType(simple: [SimpleType](flytekit_models_types_simpletype) = None, schema: [SchemaType](flytekit_models_types_schematype) = None, collection_type: [LiteralType](flytekit_models_types_literaltype) = None, map_value_type: [LiteralType](flytekit_models_types_literaltype) = None, blob: flytekit.models.core.types.BlobType = None, enum_type: flytekit.models.core.types.EnumType = None, union_type: flytekit.models.core.types.UnionType = None, structured_dataset_type: flytekit.models.core.types.StructuredDatasetType = None, metadata: dict[Text, T] = None, structure: flytekit.models.core.types.TypeStructure = None, annotation: flytekit.models.annotation.TypeAnnotation = None)
-  This is a oneof message, only one of the kwargs may be set, representing one of the Flyte types.
- **Parameters**

  - **simple**: [SimpleType](flytekit_models_types_simpletype)
    - Enum type from SimpleType
  - **schema**: [SchemaType](flytekit_models_types_schematype)
    - Type definition for a dataframe-like object.
  - **collection_type**: [LiteralType](flytekit_models_types_literaltype)
    - For list-like objects, this is the type of each entry in the list.
  - **map_value_type**: [LiteralType](flytekit_models_types_literaltype)
    - For map objects, this is the type of the value. The key must always be a
            string.
  - **blob**: flytekit.models.core.types.BlobType
    - For blob objects, this describes the type.
  - **enum_type**: flytekit.models.core.types.EnumType
    - For enum objects, describes an enum
  - **union_type**: flytekit.models.core.types.UnionType
    - For union objects, describes an python union type.
  - **structured_dataset_type**: flytekit.models.core.types.StructuredDatasetType
    - structured dataset
  - **metadata**: dict[Text, T]
    - Additional data describing the type
  - **structure**: flytekit.models.core.types.TypeStructure
    - Type matching hints
  - **annotation**: flytekit.models.annotation.TypeAnnotation
    - Additional data
            describing the type intended to be saturated by the client

def LiteralType(simple: [SimpleType](flytekit_models_types_simpletype) = None, schema: [SchemaType](flytekit_models_types_schematype) = None, collection_type: [LiteralType](flytekit_models_types_literaltype) = None, map_value_type: [LiteralType](flytekit_models_types_literaltype) = None, blob: flytekit.models.core.types.BlobType = None, enum_type: flytekit.models.core.types.EnumType = None, union_type: flytekit.models.core.types.UnionType = None, structured_dataset_type: flytekit.models.core.types.StructuredDatasetType = None, metadata: dict[Text, T] = None, structure: flytekit.models.core.types.TypeStructure = None, annotation: flytekit.models.annotation.TypeAnnotation = None)
-  This is a oneof message, only one of the kwargs may be set, representing one of the Flyte types.

:param SimpleType simple: Enum type from SimpleType
:param SchemaType schema: Type definition for a dataframe-like object.
:param LiteralType collection_type: For list-like objects, this is the type of each entry in the list.
:param LiteralType map_value_type: For map objects, this is the type of the value.  The key must always be a
    string.
:param flytekit.models.core.types.BlobType blob: For blob objects, this describes the type.
:param flytekit.models.core.types.EnumType enum_type: For enum objects, describes an enum
:param flytekit.models.core.types.UnionType union_type: For union objects, describes an python union type.
:param flytekit.models.core.types.TypeStructure structure: Type matching hints
:param flytekit.models.core.types.StructuredDatasetType structured_dataset_type: structured dataset
:param dict[Text, T] metadata: Additional data describing the type
:param flytekit.models.annotation.TypeAnnotation annotation: Additional data
    describing the type intended to be saturated by the client
- **Parameters**

  - **simple**: [SimpleType](flytekit_models_types_simpletype)
    - Enum type from SimpleType
  - **schema**: [SchemaType](flytekit_models_types_schematype)
    - Type definition for a dataframe-like object.
  - **collection_type**: [LiteralType](flytekit_models_types_literaltype)
    - For list-like objects, this is the type of each entry in the list.
  - **map_value_type**: [LiteralType](flytekit_models_types_literaltype)
    - For map objects, this is the type of the value.  The key must always be a
    string.
  - **blob**: flytekit.models.core.types.BlobType
    - For blob objects, this describes the type.
  - **enum_type**: flytekit.models.core.types.EnumType
    - For enum objects, describes an enum
  - **union_type**: flytekit.models.core.types.UnionType
    - For union objects, describes an python union type.
  - **structured_dataset_type**: flytekit.models.core.types.StructuredDatasetType
    - structured dataset
  - **metadata**: dict[Text, T]
    - Additional data describing the type
  - **structure**: flytekit.models.core.types.TypeStructure
    - Type matching hints
  - **annotation**: flytekit.models.annotation.TypeAnnotation
    - Additional data
    describing the type intended to be saturated by the client



## Methods
```@classmethod
def simple()
```
-  Getter for the simple type attribute.

- **Return Value**:
**[SimpleType](flytekit_models_types_simpletype)**
  - The simple type.
```@classmethod
def schema()
```
-  Getter for the schema type attribute.

- **Return Value**:
**[SchemaType](flytekit_models_types_schematype)**
  - The schema type.
```@classmethod
def collection_type()
```
-  Getter for the collection type attribute.

        The collection value type

- **Return Value**:
**[LiteralType](flytekit_models_types_literaltype)**
  - The collection type.
```@classmethod
def map_value_type()
```
-  Getter for the map value type attribute.

        The Value for a dictionary. Key is always string

- **Return Value**:
**[LiteralType](flytekit_models_types_literaltype)**
  - The map value type.
```@classmethod
def blob()
```
-  Getter for the blob type attribute.

- **Return Value**:
**_core_types.BlobType**
  - The blob type.
```@classmethod
def enum_type()
```
-  Getter for the enum type attribute.

- **Return Value**:
**_core_types.EnumType**
  - The enum type.
```@classmethod
def union_type()
```
-  Getter for the union type attribute.

- **Return Value**:
**[UnionType](flytekit_models_types_uniontype)**
  - The union type.
```@classmethod
def structure()
```
-  Getter for the structure type attribute.

- **Return Value**:
**[TypeStructure](flytekit_models_types_typestructure)**
  - The structure type.
```@classmethod
def structured_dataset_type()
```
-  Getter for the structured dataset type attribute.

- **Return Value**:
**[StructuredDatasetType](flytekit_models_types_structureddatasettype)**
  - The structured dataset type.
```@classmethod
def metadata()
```
-  Getter for the metadata attribute.

        :rtype: dict[Text, T]

- **Return Value**:
**dict[Text, T]**
  - The metadata.
```@classmethod
def annotation()
```
-  Getter for the annotation attribute.

        :rtype: flytekit.models.annotation.TypeAnnotation

- **Return Value**:
**TypeAnnotationModel**
  - The annotation.
@classmethod
def metadata(value: any)
-  Setter for the metadata attribute.
- **Parameters**

  - **value**: any
    - The metadata value to set.

@classmethod
def annotation(value: any)
-  Setter for the annotation attribute.
- **Parameters**

  - **value**: any
    - The annotation value to set.

```@classmethod
def to_flyte_idl()
```
-  Converts the LiteralType object to its corresponding Flyte IDL representation.
It handles the conversion of various type attributes (simple, schema, collection_type, etc.) to their IDL protobuf formats.
Metadata is parsed from JSON and converted to a Struct.

        :rtype: flyteidl.core.types_pb2.LiteralType

- **Return Value**:
**flyteidl.core.types_pb2.LiteralType**
  - The Flyte IDL LiteralType object.
@classmethod
def from_flyte_idl(proto: flyteidl.core.types_pb2.LiteralType) - > [LiteralType](flytekit_models_types_literaltype)
-  Creates a LiteralType object from its Flyte IDL representation.
It parses the IDL protobuf message and reconstructs the LiteralType object, including nested types like collection_type and map_value_type.
Metadata is converted from a Struct to a dictionary.

        :param flyteidl.core.types_pb2.LiteralType proto: The Flyte IDL LiteralType protobuf message.
        :rtype: LiteralType
- **Parameters**

  - **proto**: flyteidl.core.types_pb2.LiteralType
    - The Flyte IDL LiteralType protobuf message.

- **Return Value**:
**[LiteralType](flytekit_models_types_literaltype)**
  - A LiteralType object created from the IDL.
