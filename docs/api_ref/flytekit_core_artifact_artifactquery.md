# ArtifactQuery

This class represents a query associated with an artifact, enabling the specification of filters such as time partitions, project, domain, and tags. It facilitates the creation of queries with dependencies on other artifacts through bindings. The class provides methods to convert the query to a Flyte IDL representation and generate string representations for debugging and informational purposes.

## Attributes

- **artifact**: [Artifact](flytekit_core_artifact_artifact)
  - The artifact to query.

- **name**: str
  - The name of the query.

- **project**: Optional[str] = None
  - The project of the artifact.

- **domain**: Optional[str] = None
  - The domain of the artifact.

- **time_partition**: Optional[[TimePartition](flytekit_core_artifact_timepartition)] = None
  - The time partition to apply to the query.

- **partitions**: Optional[[Partitions](flytekit_core_artifact_partitions)] = None
  - The partitions to apply to the query.

- **tag**: Optional[str] = None
  - The tag to filter the artifact by.

- **binding**: Optional[[Artifact](flytekit_core_artifact_artifact)] = None
  - The artifact that this query is bound to.

## Constructors
def ArtifactQuery(artifact: [Artifact](flytekit_core_artifact_artifact), name: str, project: Optional[str] = None, domain: Optional[str] = None, time_partition: Optional[[TimePartition](flytekit_core_artifact_timepartition)] = None, partitions: Optional[[Partitions](flytekit_core_artifact_partitions)] = None, tag: Optional[str] = None)
-  Initializes an ArtifactQuery object.
- **Parameters**

  - **artifact**: [Artifact](flytekit_core_artifact_artifact)
    - The artifact to query.
  - **name**: str
    - The name of the query.
  - **project**: Optional[str]
    - The project associated with the query.
  - **domain**: Optional[str]
    - The domain associated with the query.
  - **time_partition**: Optional[[TimePartition](flytekit_core_artifact_timepartition)]
    - The time partition for the query.
  - **partitions**: Optional[[Partitions](flytekit_core_artifact_partitions)]
    - The partitions for the query.
  - **tag**: Optional[str]
    - The tag for the query.



## Methods
```@classmethod
def bound()
```
-  Returns True if the query is bound, False otherwise. A query is bound if all its partition keys are specified.

- **Return Value**:
**bool**
  - True if the query is bound, False otherwise.
@classmethod
def to_flyte_idl(kwargs: **kwargs) - > art_id.ArtifactQuery
-  Converts the ArtifactQuery object to its corresponding Flyte IDL representation.
- **Parameters**

  - **kwargs**: **kwargs
    - Additional keyword arguments.

- **Return Value**:
**art_id.ArtifactQuery**
  - The Flyte IDL representation of the ArtifactQuery.
@classmethod
def get_time_partition_str(kwargs: **kwargs) - > str
-  Returns a string representation of the time partition, including its value or input binding.
- **Parameters**

  - **kwargs**: **kwargs
    - Additional keyword arguments, potentially containing input binding values.

- **Return Value**:
**str**
  - String representation of the time partition.
@classmethod
def get_partition_str(kwargs: **kwargs) - > str
-  Returns a string representation of the partitions, including their static values or input bindings.
- **Parameters**

  - **kwargs**: **kwargs
    - Additional keyword arguments, potentially containing input binding values.

- **Return Value**:
**str**
  - String representation of the partitions.
@classmethod
def get_str(kwargs: **kwargs) - > str
-  Returns a detailed string representation of the ArtifactQuery, including artifact name, time partition, and other partitions.
- **Parameters**

  - **kwargs**: **kwargs
    - Additional keyword arguments to pass to get_time_partition_str and get_partition_str.

- **Return Value**:
**str**
  - Detailed string representation of the ArtifactQuery.
