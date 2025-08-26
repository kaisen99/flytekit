# TimePartition

This class represents a time-based partition, handling datetime values and operations. It enables time-based calculations and comparisons, supporting different granularities. The class utilizes the art_id.LabelValue for storing time-related data and provides methods for arithmetic operations with timedelta objects.

## Attributes

- **value**: [Union](flytekit_models_literals_union)[art_id.LabelValue, art_id.InputBindingData, str, datetime.datetime, None] = None
  - The time value for the partition.

- **op**: Optional[Op] = None
  - The operation applied to the time partition.

- **other**: Optional[timedelta] = None
  - Another timedelta value, used in conjunction with &#x27;op&#x27;.

- **granularity**: Granularity = Granularity.DAY
  - The granularity of the time partition (e.g., DAY, HOUR).

- **reference_artifact**: Optional[[Artifact](flytekit_core_artifact_artifact)] = None
  - A reference to an artifact.

## Constructors
def TimePartition(value: [Union](flytekit_models_literals_union)[art_id.LabelValue, art_id.InputBindingData, str, datetime.datetime, None], op: Optional[Op] = None, other: Optional[timedelta] = None, granularity: Granularity = Granularity.DAY)
-  Initializes a TimePartition object.
- **Parameters**

  - **value**: [Union](flytekit_models_literals_union)[art_id.LabelValue, art_id.InputBindingData, str, datetime.datetime, None]
    - The time value for the partition. Can be a LabelValue, InputBindingData, datetime object, or a string (which will raise a ValueError). If it&#x27;s a string, it raises a ValueError. If it&#x27;s a datetime object, it&#x27;s converted to a Timestamp. If it&#x27;s InputBindingData, it&#x27;s used to create a LabelValue.
  - **op**: Optional[Op]
    - An optional operation to be applied.
  - **other**: Optional[timedelta]
    - An optional timedelta value.
  - **granularity**: Granularity
    - The granularity of the time partition, defaulting to DAY.



## Methods
@classmethod
def to_flyte_idl(**kwargs: dict) - > Optional[art_id.TimePartition]
-  Converts the TimePartition object to its corresponding Flyte IDL (Interface Definition Language) representation.
This is useful for serialization and interoperability with Flyte systems.
- **Parameters**

  - ****kwargs**: dict
    - Additional keyword arguments to pass to the serializer.

- **Return Value**:
**Optional[art_id.TimePartition]**
  - The Flyte IDL representation of the TimePartition, or None if conversion is not possible.
