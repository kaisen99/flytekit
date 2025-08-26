# ParquetToSparkDecodingHandler

This class decodes Parquet-formatted structured datasets into Spark DataFrames. It leverages the SparkSession to read Parquet files from a given URI. The class supports column selection based on the provided structured dataset metadata.

## Constructors
```def ParquetToSparkDecodingHandler()
```
-  Initializes the ParquetToSparkDecodingHandler with default values.



## Methods
@classmethod
def decode(ctx: [FlyteContext](flytekit_core_context_manager_flytecontext), flyte_value: literals.StructuredDataset, current_task_metadata: [StructuredDatasetMetadata](flytekit_models_literals_structureddatasetmetadata)) - > DataFrame
-  Decodes a StructuredDataset into a Spark DataFrame. It reads the Parquet data from the provided URI and optionally selects specific columns if metadata is available.
- **Parameters**

  - **ctx**: [FlyteContext](flytekit_core_context_manager_flytecontext)
    - The Flyte context object.
  - **flyte_value**: literals.StructuredDataset
    - The StructuredDataset to decode, containing the URI to the Parquet data.
  - **current_task_metadata**: [StructuredDatasetMetadata](flytekit_models_literals_structureddatasetmetadata)
    - Metadata about the structured dataset, potentially including column names.

- **Return Value**:
**DataFrame**
  - A Spark DataFrame representing the decoded data.
