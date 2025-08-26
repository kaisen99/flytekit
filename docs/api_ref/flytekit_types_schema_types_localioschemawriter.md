# LocalIOSchemaWriter

This class facilitates writing data schemas to local storage. It extends a base SchemaWriter class, providing a mechanism for saving data in a specified format. The class supports writing multiple dataframes and relies on an abstract method for the actual writing process, which must be implemented by subclasses.

## Constructors
def LocalIOSchemaWriter(to_local_path: str, cols: typing.Optional[typing.Dict[str, type]], fmt: [SchemaFormat](flytekit_types_schema_types_schemaformat))
-  Initializes the LocalIOSchemaWriter with a local path, optional column mappings, and a schema format.
- **Parameters**

  - **to_local_path**: str
    - The local path to write the schema to.
  - **cols**: typing.Optional[typing.Dict[str, type]]
    - An optional dictionary mapping column names to their types.
  - **fmt**: [SchemaFormat](flytekit_types_schema_types_schemaformat)
    - The format of the schema to be written.



## Methods
@classmethod
def write(dfs: tuple, kwargs: dict) - > null
-  Writes one or more DataFrames to local files. It iterates through the provided DataFrames and calls the _write method for each one with a generated file name.
- **Parameters**

  - **dfs**: tuple
    - A variable number of DataFrames to write.
  - **kwargs**: dict
    - Additional keyword arguments to pass to the _write method.

- **Return Value**:
**null**
  - This method does not return anything.
