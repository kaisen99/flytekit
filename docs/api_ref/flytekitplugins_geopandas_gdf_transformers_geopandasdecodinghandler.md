# GeoPandasDecodingHandler

This class decodes structured datasets into GeoPandas GeoDataFrames. It leverages the GeoPandas library to read geospatial data from various formats, primarily Parquet and other file types. The class handles the decoding process, converting data from a structured dataset representation into a usable GeoDataFrame object.



## Methods
@classmethod
def decode(ctx: [FlyteContext](flytekit_core_context_manager_flytecontext), flyte_value: literals.StructuredDataset, current_task_metadata: [StructuredDatasetMetadata](flytekit_models_literals_structureddatasetmetadata)) - > gpd.GeoDataFrame
-  Decodes a StructuredDataset into a GeoDataFrame. It attempts to read the data as Parquet first, and if that fails due to an ArrowInvalid error, it falls back to reading it as a generic file.
- **Parameters**

  - **ctx**: [FlyteContext](flytekit_core_context_manager_flytecontext)
    - The Flyte context.
  - **flyte_value**: literals.StructuredDataset
    - The StructuredDataset to decode.
  - **current_task_metadata**: [StructuredDatasetMetadata](flytekit_models_literals_structureddatasetmetadata)
    - Metadata for the current task.

- **Return Value**:
**gpd.GeoDataFrame**
  - The decoded GeoDataFrame.
