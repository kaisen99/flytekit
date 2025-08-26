# Blob

This class represents binary data stored in a specific location, identified by a unique URI. It encapsulates metadata associated with the blob and provides methods for converting to and from Flyte IDL representations. The class relies on BlobMetadata for managing metadata and utilizes a URI to locate the binary data.

## Attributes

- **metadata**: [BlobMetadata](flytekit_models_literals_blobmetadata) = None
  - BlobMetadata metadata

- **uri**: Text = None
  - Text uri: The location of this blob

## Constructors
def Blob(metadata: [BlobMetadata](flytekit_models_literals_blobmetadata), uri: Text)
-  This literal model is used to represent binary data offloaded to some storage location which is identifiable with a unique string. See {{&lt; py_class_ref flytekit.FlyteFile &gt;}} as an example.
- **Parameters**

  - **metadata**: [BlobMetadata](flytekit_models_literals_blobmetadata)
    - 
  - **uri**: Text
    - The location of this blob



## Methods
```@classmethod
def uri()
```
-  The location of this blob

- **Return Value**:
**string**
  - The location of this blob
```@classmethod
def metadata()
```
-  This literal model is used to represent binary data offloaded to some storage location which is identifiable with a unique string. See {{&lt; py_class_ref flytekit.FlyteFile &gt;}} as an example.

- **Return Value**:
**[BlobMetadata](flytekit_models_literals_blobmetadata)**
  - BlobMetadata
```@classmethod
def to_flyte_idl()
```
-  Converts the Blob object to its corresponding Flyte IDL protobuf representation.

- **Return Value**:
**flyteidl.core.literals_pb2.Blob**
  - The Flyte IDL Blob protobuf object.
@classmethod
def from_flyte_idl(proto: flyteidl.core.literals_pb2.Blob) - > [Blob](flytekit_models_literals_blob)
-  Creates a Blob object from its Flyte IDL protobuf representation.
- **Parameters**

  - **proto**: flyteidl.core.literals_pb2.Blob
    - The Flyte IDL Blob protobuf object.

- **Return Value**:
**[Blob](flytekit_models_literals_blob)**
  - A Blob object created from the protobuf.
