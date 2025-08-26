# StructuredDatasetType

This class represents a structured dataset type, defining its schema and format. It allows specifying columns with their respective literal types. The class supports conversion to and from Flyte IDL representations, enabling interoperability with the Flyte platform.

## Attributes

- **columns**: List[DatasetColumn] = None
  - List of DatasetColumn objects.

- **format**: str = &quot;&quot;
  - Format of the dataset.

- **external_schema_type**: str = None
  - Type of the external schema.

- **external_schema_bytes**: bytes = None
  - Bytes of the external schema.

## Constructors
def StructuredDatasetType(columns: typing.List[DatasetColumn] = None, format: str = &quot;&quot;, external_schema_type: str = None, external_schema_bytes: bytes = None)
-  Initializes a new instance of the StructuredDatasetType class.
- **Parameters**

  - **columns**: typing.List[DatasetColumn]
    - A list of DatasetColumn objects representing the columns of the dataset.
  - **format**: str
    - The format of the dataset.
  - **external_schema_type**: str
    - The type of the external schema.
  - **external_schema_bytes**: bytes
    - The bytes of the external schema.



## Methods
```@classmethod
def columns()
```
-  Name for the column

- **Return Value**:
**typing.List[DatasetColumn]**
  - Name for the column
@classmethod
def columns(value: typing.List[DatasetColumn]) - > None
-  setter for columns
- **Parameters**

  - **value**: typing.List[DatasetColumn]
    - setter for columns

- **Return Value**:
**None**
  - setter for columns
```@classmethod
def format()
```
-  return the format

- **Return Value**:
**str**
  - return the format
@classmethod
def format(format: str) - > None
-  setter for format
- **Parameters**

  - **format**: str
    - setter for format

- **Return Value**:
**None**
  - setter for format
```@classmethod
def external_schema_type()
```
-  return the external schema type

- **Return Value**:
**str**
  - return the external schema type
```@classmethod
def external_schema_bytes()
```
-  return the external schema bytes

- **Return Value**:
**bytes**
  - return the external schema bytes
```@classmethod
def to_flyte_idl()
```
-  Convert the current object to flyte idl

- **Return Value**:
**_types_pb2.StructuredDatasetType**
  - Convert the current object to flyte idl
@classmethod
def from_flyte_idl(proto: _types_pb2.StructuredDatasetType) - > _types_pb2.StructuredDatasetType
-  Convert flyte idl to object
- **Parameters**

  - **proto**: _types_pb2.StructuredDatasetType
    - Convert flyte idl to object

- **Return Value**:
**_types_pb2.StructuredDatasetType**
  - Convert flyte idl to object
