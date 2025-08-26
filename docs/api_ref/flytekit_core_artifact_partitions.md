# Partitions

This class manages a collection of partitions, each representing a specific data division. It allows for the setting of a reference artifact and provides access to individual partitions by name. The class supports conversion to a Flyte IDL representation and handles different input types for partition creation.

## Attributes

- **reference_artifact**: Optional[[Artifact](flytekit_core_artifact_artifact)] = None
  - The artifact that this partition belongs to.

## Constructors
def Partitions(partitions: Optional[typing.Mapping[str, [Union](flytekit_models_literals_union)[str, art_id.InputBindingData, [Partition](flytekit_core_artifact_partition)]]])
-  Initializes the Partitions object. It takes an optional mapping of partition names to their values, which can be of type str, InputBindingData, or Partition. If a value is not a Partition object, it is converted into one.
- **Parameters**

  - **partitions**: Optional[typing.Mapping[str, [Union](flytekit_models_literals_union)[str, art_id.InputBindingData, [Partition](flytekit_core_artifact_partition)]]]
    - An optional mapping where keys are partition names (strings) and values can be strings, art_id.InputBindingData objects, or Partition objects.



## Methods
```@classmethod
def partitions()
```
-  This is a property that returns the internal dictionary of partitions.

- **Return Value**:
**Optional[typing.Dict[str, [Partition](flytekit_core_artifact_partition)]]**
  - The dictionary of partitions.
@classmethod
def set_reference_artifact(artifact: [Artifact](flytekit_core_artifact_artifact)) - > None
-  Sets the reference artifact for all partitions within this object. It also updates the reference_artifact for each individual partition if they exist.
- **Parameters**

  - **artifact**: [Artifact](flytekit_core_artifact_artifact)
    - The artifact to set as the reference.

- **Return Value**:
**None**
  - None
@classmethod
def to_flyte_idl(**kwargs: Any) - > Optional[art_id.Partitions]
-  Converts the Partitions object to its Flyte IDL (Interface Definition Language) representation using a Serializer.
- **Parameters**

  - ****kwargs**: Any
    - Additional keyword arguments to pass to the serializer.

- **Return Value**:
**Optional[art_id.Partitions]**
  - The Flyte IDL representation of the partitions.
