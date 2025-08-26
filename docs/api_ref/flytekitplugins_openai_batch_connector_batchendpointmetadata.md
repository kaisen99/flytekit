# BatchEndpointMetadata

This class encapsulates metadata associated with a batch endpoint. It provides methods for encoding and decoding the metadata, enabling serialization and deserialization. The class utilizes cloudpickle for object serialization and deserialization.

## Attributes

- **openai_org**: string
  - The OpenAI organization associated with the batch.

- **batch_id**: string
  - The unique identifier for the batch.



## Methods
```@classmethod
def encode()
```
-  Encodes the BatchEndpointMetadata object into bytes using cloudpickle.

- **Return Value**:
**bytes**
  - The encoded bytes representation of the object.
@classmethod
def decode(data: bytes) - > [BatchEndpointMetadata](flytekitplugins_openai_batch_connector_batchendpointmetadata)
-  Decodes bytes data into a BatchEndpointMetadata object using cloudpickle.
- **Parameters**

  - **data**: bytes
    - The bytes data to decode.

- **Return Value**:
**[BatchEndpointMetadata](flytekitplugins_openai_batch_connector_batchendpointmetadata)**
  - The decoded BatchEndpointMetadata object.
