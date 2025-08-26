# PandasSchemaWriter

This class specializes in writing Pandas DataFrames to local storage, specifically using the Parquet format. It extends a base class for local I/O operations, handling schema and format specifications. The class leverages a ParquetIO engine for the actual DataFrame writing process, ensuring efficient data serialization.

## Constructors
def PandasSchemaWriter(local_dir: str, cols: typing.Optional[typing.Dict[str, type]], fmt: [SchemaFormat](flytekit_types_schema_types_schemaformat))
-  Initializes the PandasSchemaWriter with a local directory, optional column types, and schema format.
- **Parameters**

  - **local_dir**: str
    - The local directory path where schemas will be written.
  - **cols**: typing.Optional[typing.Dict[str, type]]
    - An optional dictionary mapping column names to their types.
  - **fmt**: [SchemaFormat](flytekit_types_schema_types_schemaformat)
    - The format of the schema to be written.



