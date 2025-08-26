# PickledEntity

This class defines the structure for pickled objects saved in .pkl files, specifically for interactive mode. It stores metadata about the pickled entities and a dictionary mapping entity names to their corresponding PythonAutoContainerTask instances. This class facilitates the preservation and retrieval of Python objects and their associated metadata.

## Attributes

- **metadata**: [PickledEntityMetadata](flytekit_core_python_auto_container_pickledentitymetadata)
  - Metadata about the pickled entities including Python version

- **entities**: Dict[str, [PythonAutoContainerTask](flytekit_core_python_auto_container_pythonautocontainertask)]
  - Dictionary mapping entity names to their PythonAutoContainerTask instances



