# ParquetToPandasDecodingHandler

This class decodes Parquet files into Pandas DataFrames. It leverages the Pandas library to read Parquet files from a given URI. The class handles column selection and storage options, including authentication, for accessing the Parquet data.

## Constructors
```def ParquetToPandasDecodingHandler()
```
-  Initializes the ParquetToPandasDecodingHandler. It calls the superclass constructor with \&#x27;pd.DataFrame\&#x27;, \&#x27;None\&#x27;, and \&#x27;PARQUET\&#x27; as arguments.



## Methods
@classmethod
def decode(ctx: [FlyteContext](flytekit_core_context_manager_flytecontext), flyte_value: literals.StructuredDataset, current_task_metadata: [StructuredDatasetMetadata](flytekit_models_literals_structureddatasetmetadata)) - > pd.DataFrame
-  Decodes a StructuredDataset into a pandas DataFrame. It reads data from a given URI, optionally specifying columns and handling potential S3 access credentials.
- **Parameters**

  - **ctx**: [FlyteContext](flytekit_core_context_manager_flytecontext)
    - The Flyte context, providing access to file system and configuration.
  - **flyte_value**: literals.StructuredDataset
    - The StructuredDataset to decode, containing the URI of the data.
  - **current_task_metadata**: [StructuredDatasetMetadata](flytekit_models_literals_structureddatasetmetadata)
    - Metadata for the current task, which may include column information.

- **Return Value**:
**pd.DataFrame**
  - A pandas DataFrame containing the decoded data.
