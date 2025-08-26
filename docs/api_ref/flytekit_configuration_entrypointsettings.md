# EntrypointSettings

This class encapsulates the path to the entrypoint command used during runtime. It provides the location of the `pyflyte-execute` code, which is essential for operations such as pyspark execution. The primary function of this class is to store and manage the entrypoint path.

## Attributes

- **path**: Optional[str] = None
  - This object carries information about the path of the entrypoint command that will be invoked at runtime. This is where `pyflyte-execute` code can be found. This is useful for cases like pyspark execution.



