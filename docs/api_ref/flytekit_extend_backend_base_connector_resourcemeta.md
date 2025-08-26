# ResourceMeta

This class encapsulates metadata associated with a resource, such as a job identifier. It provides methods for encoding the metadata into a byte representation and decoding it back from bytes. The class leverages JSON serialization for data transformation.



## Methods
```@classmethod
def encode()
```
-  Encode the resource meta to bytes.

- **Return Value**:
**bytes**
  - The encoded resource meta as bytes.
@classmethod
def decode(data: bytes) - > [ResourceMeta](flytekit_extend_backend_base_connector_resourcemeta)
-  Decode the resource meta from bytes.
- **Parameters**

  - **data**: bytes
    - The bytes data to decode.

- **Return Value**:
**[ResourceMeta](flytekit_extend_backend_base_connector_resourcemeta)**
  - The decoded ResourceMeta object.
