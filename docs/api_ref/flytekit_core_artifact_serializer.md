# Serializer

This class provides a centralized interface for serializing and deserializing artifacts. It leverages a configurable serialization handler to convert between Python objects and their IDL representations. Key functionalities include methods for handling partitions, time partitions, and artifact queries.

## Attributes

- **serializer**: [ArtifactSerializationHandler](flytekit_core_artifact_artifactserializationhandler) = DefaultArtifactSerializationHandler()
  - DefaultArtifactSerializationHandler



## Methods
@classmethod
def register_serializer(serializer: [ArtifactSerializationHandler](flytekit_core_artifact_artifactserializationhandler))
-  Registers a serializer to be used by the Serializer class.
- **Parameters**

  - **serializer**: [ArtifactSerializationHandler](flytekit_core_artifact_artifactserializationhandler)
    - The ArtifactSerializationHandler instance to register.

@classmethod
def partitions_to_idl(p: Optional[[Partitions](flytekit_core_artifact_partitions)], **kwargs: dict) - > Optional[art_id.Partitions]
-  Converts Partitions to an IDL representation using the registered serializer.
- **Parameters**

  - **p**: Optional[[Partitions](flytekit_core_artifact_partitions)]
    - The Partitions object to convert.
  - ****kwargs**: dict
    - Additional keyword arguments to pass to the serializer.

- **Return Value**:
**Optional[art_id.Partitions]**
  - The IDL representation of the Partitions, or None if conversion fails.
@classmethod
def time_partition_to_idl(tp: [TimePartition](flytekit_core_artifact_timepartition), **kwargs: dict) - > Optional[art_id.TimePartition]
-  Converts a TimePartition to an IDL representation using the registered serializer.
- **Parameters**

  - **tp**: [TimePartition](flytekit_core_artifact_timepartition)
    - The TimePartition object to convert.
  - ****kwargs**: dict
    - Additional keyword arguments to pass to the serializer.

- **Return Value**:
**Optional[art_id.TimePartition]**
  - The IDL representation of the TimePartition, or None if conversion fails.
@classmethod
def artifact_query_to_idl(aq: [ArtifactQuery](flytekit_core_artifact_artifactquery), **kwargs: dict) - > art_id.ArtifactQuery
-  Converts an ArtifactQuery to an IDL representation using the registered serializer.
- **Parameters**

  - **aq**: [ArtifactQuery](flytekit_core_artifact_artifactquery)
    - The ArtifactQuery object to convert.
  - ****kwargs**: dict
    - Additional keyword arguments to pass to the serializer.

- **Return Value**:
**art_id.ArtifactQuery**
  - The IDL representation of the ArtifactQuery.
