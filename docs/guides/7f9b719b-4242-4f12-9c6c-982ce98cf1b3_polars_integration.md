
<!--
help_text: ''
key: summary_polars_integration_9252a5b6-3e80-427d-a8c8-44d9b60138a0
modules:
- flytekitplugins.polars.sd_transformers
questions_to_answer: []
type: summary

-->
# Polars Integration

This integration provides seamless capabilities for handling Polars `DataFrame` and `LazyFrame` objects within the platform's data processing workflows. It enables automatic serialization and deserialization of Polars data structures to and from the Parquet format, facilitating efficient data exchange and persistence. Additionally, it includes functionality for rendering Polars data for visualization purposes.

### Core Data Handling

The integration automatically manages the conversion of Polars data structures to and from the Parquet format when they are passed as `StructuredDataset` types in tasks. This ensures data integrity and efficient storage.

#### Serializing Polars DataFrames

When a `polars.DataFrame` is returned from a task and declared as a `StructuredDataset`, the `PolarsDataFrameToParquetEncodingHandler` encodes it into a Parquet file. The handler writes the DataFrame to a `BytesIO` buffer and then persists it to the designated storage URI. If a URI is not explicitly provided, a default filename is used, and the data is stored in a temporary location managed by the platform's file access system.

Conceptual example of a task returning a Polars DataFrame:

```python
import polars as pl
from flytekit.types.structured.structured_dataset import StructuredDataset

def my_dataframe_task() -> StructuredDataset:
    df = pl.DataFrame({"a": [1, 2, 3], "b": ["x", "y", "z"]})
    # The platform automatically handles the conversion to StructuredDataset
    # and subsequent encoding to Parquet via PolarsDataFrameToParquetEncodingHandler
    return StructuredDataset(dataframe=df)
```

#### Deserializing to Polars DataFrames

When a task expects a `polars.DataFrame` as an input parameter, and the input is provided as a `StructuredDataset` pointing to a Parquet file, the `ParquetToPolarsDataFrameDecodingHandler` automatically decodes the Parquet data into a `polars.DataFrame`. The handler leverages `polars.read_parquet` and `fsspec` for flexible data access from various storage backends. If schema metadata (columns) is available in the `StructuredDatasetMetadata`, it is used to optimize the read operation by selecting only the required columns.

Conceptual example of a task consuming a Polars DataFrame:

```python
import polars as pl
from flytekit.types.structured.structured_dataset import StructuredDataset

def process_dataframe_task(sd: StructuredDataset) -> int:
    # The platform automatically decodes the StructuredDataset into a pl.DataFrame
    df: pl.DataFrame = sd.open(pl.DataFrame).all()
    return df.shape[0]
```

#### Serializing Polars LazyFrames

The integration also supports `polars.LazyFrame` objects. When a `polars.LazyFrame` is returned as a `StructuredDataset`, the `PolarsLazyFrameToParquetEncodingHandler` manages its serialization. Currently, `LazyFrame` objects are first `collect()`ed into a `DataFrame` before being written to Parquet. This approach ensures stability, as direct streaming writes for `LazyFrame` are still considered experimental in Polars. The data is then written to the specified URI or a temporary location.

Conceptual example of a task returning a Polars LazyFrame:

```python
import polars as pl
from flytekit.types.structured.structured_dataset import StructuredDataset

def my_lazyframe_task() -> StructuredDataset:
    lf = pl.scan_csv("my_large_data.csv").filter(pl.col("value") > 100)
    # The platform automatically handles the conversion to StructuredDataset
    # and subsequent encoding to Parquet via PolarsLazyFrameToParquetEncodingHandler
    return StructuredDataset(dataframe=lf)
```

#### Deserializing to Polars LazyFrames

For tasks that consume `polars.LazyFrame` inputs from `StructuredDataset`s, the `ParquetToPolarsLazyFrameDecodingHandler` decodes the Parquet data. It reads the Parquet file into a `polars.DataFrame` using `polars.read_parquet` and then converts it to a `polars.LazyFrame` using the `.lazy()` method. This is done because `polars.scan_parquet` currently has limitations when used with `fsspec` for remote storage. Similar to DataFrame decoding, column metadata is used if available.

Conceptual example of a task consuming a Polars LazyFrame:

```python
import polars as pl
from flytekit.types.structured.structured_dataset import StructuredDataset

def process_lazyframe_task(sd: StructuredDataset) -> int:
    # The platform automatically decodes the StructuredDataset into a pl.LazyFrame
    lf: pl.LazyFrame = sd.open(pl.LazyFrame).all()
    return lf.select(pl.count()).collect().item()
```

### Data Visualization and Rendering

The `PolarsDataFrameRenderer` provides a mechanism to generate an HTML representation of Polars `DataFrame` and `LazyFrame` objects. This is particularly useful for displaying summary statistics or previews of data within user interfaces or reports. The renderer computes descriptive statistics using the `describe()` method (collecting `LazyFrame` if necessary for older Polars versions) and then formats the output as an HTML table.

Conceptual example of using the renderer:

```python
import polars as pl
from flytekitplugins.polars.sd_transformers import PolarsDataFrameRenderer

df = pl.DataFrame({"col1": [1, 2, 3], "col2": [4.0, 5.0, 6.0]})
renderer = PolarsDataFrameRenderer()
html_output = renderer.to_html(df)
# html_output can then be embedded in a UI or report
```

### Important Considerations

*   **LazyFrame Collection**: When serializing `polars.LazyFrame` objects, the data is currently collected into a `polars.DataFrame` in memory before being written to Parquet. For very large `LazyFrame`s, ensure sufficient memory is available during the serialization step. This is a temporary measure until Polars' `sink_parquet` method (for streaming writes) becomes stable and compatible with the underlying file access mechanisms.
*   **LazyFrame Decoding**: When deserializing to `polars.LazyFrame`, `polars.read_parquet` is used followed by `.lazy()`, rather than `polars.scan_parquet`. This is due to current compatibility issues between `polars.scan_parquet` and `fsspec` for remote file systems. This means the entire Parquet file is read into memory before the `LazyFrame` is created, which might impact performance for extremely large datasets.
*   **Polars Version Compatibility**: The integration handles variations in Polars API (e.g., `to_parquet` vs `write_parquet`, `describe` method for `LazyFrame`) to maintain compatibility across different Polars versions.
*   **Storage Options**: The integration leverages `fsspec` for flexible storage backend support. Ensure the necessary `fsspec` and backend-specific libraries (e.g., `s3fs`, `gcsfs`) are installed in your environment if you are working with remote storage.
<!--
key: summary_polars_integration_9252a5b6-3e80-427d-a8c8-44d9b60138a0
type: summary_end

-->
<!--
code_unit: flytekitplugins.polars.examples.polars_dataframe_example
code_unit_type: class
help_text: ''
key: example_a80de2b8-8944-45e4-85f1-17b957b159dc
type: example

-->