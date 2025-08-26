# OutputMetadataTracker

This class enables users to associate arbitrary metadata with output literals. It allows for setting and retrieving metadata for specific output objects. The class utilizes a dictionary to store metadata, keyed by the object&#x27;s ID, and provides methods for adding and retrieving metadata.

## Attributes

- **output_metadata**: typing.Dict[typing.Any, [OutputMetadata](flytekit_core_context_manager_outputmetadata)] = field(default_factory=dict)
  - is a sparse dictionary of metadata that the user wants to attach to each output of a task. The key is the output value (object) and the value is an OutputMetadata object.

## Constructors
def OutputMetadataTracker(output_metadata: Optional[TaskOutputMetadata] = None)
-  Initializes the OutputMetadataTracker with optional output metadata.
- **Parameters**

  - **output_metadata**: Optional[TaskOutputMetadata]
    - An optional dictionary of metadata to attach to outputs. If not provided, an empty dictionary is used.



## Methods
@classmethod
def add(obj: typing.Any, metadata: [OutputMetadata](flytekit_core_context_manager_outputmetadata)) - > None
-  Adds metadata to an object.
- **Parameters**

  - **obj**: typing.Any
    - The object to add metadata to.
  - **metadata**: [OutputMetadata](flytekit_core_context_manager_outputmetadata)
    - The metadata to add.

- **Return Value**:
**None**
  - None
@classmethod
def get(obj: typing.Any) - > Optional[[OutputMetadata](flytekit_core_context_manager_outputmetadata)]
-  Retrieves metadata for a given object.
- **Parameters**

  - **obj**: typing.Any
    - The object to retrieve metadata for.

- **Return Value**:
**Optional[[OutputMetadata](flytekit_core_context_manager_outputmetadata)]**
  - The metadata associated with the object, or None if not found.
@classmethod
def with_params(output_metadata: Optional[TaskOutputMetadata] = None) - > [OutputMetadataTracker](flytekit_core_context_manager_outputmetadatatracker)
-  Produces a copy of the current object and sets new metadata.
- **Parameters**

  - **output_metadata**: Optional[TaskOutputMetadata]
    - Optional new metadata to set on the copied object.

- **Return Value**:
**[OutputMetadataTracker](flytekit_core_context_manager_outputmetadatatracker)**
  - A new OutputMetadataTracker object with the specified metadata.
