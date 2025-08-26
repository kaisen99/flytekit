# CSVToPandasDecodingHandler

This class decodes structured datasets from CSV format into Pandas DataFrames. It leverages the Pandas library to read CSV files, handling potential storage options and column specifications. The class supports reading from various storage locations and includes error handling for credential-related issues.

## Constructors
```def CSVToPandasDecodingHandler()
```
-  Initializes the CSVToPandasDecodingHandler.



## Methods
@classmethod
def decode(ctx: [FlyteContext](flytekit_core_context_manager_flytecontext), flyte_value: literals.StructuredDataset, current_task_metadata: [StructuredDatasetMetadata](flytekit_models_literals_structureddatasetmetadata)) - > pd.DataFrame
-  Decodes a StructuredDataset into a pandas DataFrame. It handles reading CSV files, including authentication for S3 access.
- **Parameters**

  - **ctx**: [FlyteContext](flytekit_core_context_manager_flytecontext)
    - The Flyte context, providing access to file system and configuration.
  - **flyte_value**: literals.StructuredDataset
    - The StructuredDataset containing the CSV data&#x27;s URI.
  - **current_task_metadata**: [StructuredDatasetMetadata](flytekit_models_literals_structureddatasetmetadata)
    - Metadata about the dataset, potentially including column names.

- **Return Value**:
**pd.DataFrame**
  - A pandas DataFrame representing the decoded CSV data.
