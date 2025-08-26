
<!--
help_text: ''
key: summary_geopandas_integration_1ea83996-bbfa-45e4-a2c4-30f813e9c0fd
modules:
- flytekitplugins.geopandas.gdf_transformers
questions_to_answer: []
type: summary

-->
GeoPandas Integration provides seamless handling of GeoPandas `GeoDataFrame` objects within workflows, enabling the direct use of geospatial data as inputs and outputs for tasks. This integration simplifies the development of geospatial data processing pipelines by abstracting the underlying data serialization and deserialization mechanisms.

## Handling GeoDataFrame Objects

The integration automatically manages the conversion of `GeoDataFrame` objects to and from a persistent storage format, treating them as structured datasets. This allows `GeoDataFrame`s to be passed between tasks, stored, and retrieved efficiently.

### Serialization of GeoDataFrames

When a task produces a `GeoDataFrame` as an output, the `GeoPandasEncodingHandler` serializes the `GeoDataFrame` into a Parquet file. This process ensures that the geospatial data, including its geometry and attributes, is preserved in a columnar storage format optimized for performance and interoperability.

The handler automatically determines a storage URI, typically within the task's output prefix. If the specified URI is local, it creates the necessary directory structure. The `GeoDataFrame` is then written to a `data.parquet` file within that URI. The structured dataset's metadata is updated to reflect the Parquet format.

For example, a task returning a `GeoDataFrame` might look like this:

```python
import geopandas as gpd
from flytekit import task, workflow

@task
def process_geospatial_data() -> gpd.GeoDataFrame:
    # Example: Create a simple GeoDataFrame
    from shapely.geometry import Point
    data = {'col1': [1, 2], 'geometry': [Point(1, 2), Point(2, 1)]}
    gdf = gpd.GeoDataFrame(data, crs="EPSG:4326")
    return gdf

@workflow
def geospatial_workflow():
    processed_gdf = process_geospatial_data()
    # The processed_gdf is automatically serialized by GeoPandasEncodingHandler
```

### Deserialization of GeoDataFrames

When a task consumes a `GeoDataFrame` as an input, the `GeoPandasDecodingHandler` is responsible for loading the data from its stored URI. The decoding process prioritizes efficiency and compatibility:

1.  It first attempts to read the data using `geopandas.read_parquet()`. This is the preferred method as `GeoPandasEncodingHandler` primarily writes to Parquet.
2.  If the initial attempt fails (e.g., due to the file not being a valid Parquet file, indicating it might have been generated externally or by a different process), it falls back to `geopandas.read_file()`. This fallback provides broad compatibility, allowing the integration to read various geospatial file formats (e.g., Shapefile, GeoJSON, FlatGeobuf) that `geopandas.read_file()` supports, provided the necessary underlying drivers (like Fiona, GDAL) are available in the execution environment.

This dual-approach ensures robust handling of `GeoDataFrame` inputs, whether they originate from within the system's serialization process or from external sources.

A task consuming a `GeoDataFrame` might be defined as:

```python
import geopandas as gpd
from flytekit import task, workflow

@task
def analyze_geospatial_data(input_gdf: gpd.GeoDataFrame):
    # The input_gdf is automatically deserialized by GeoPandasDecodingHandler
    print(f"Received GeoDataFrame with {len(input_gdf)} rows.")
    print(input_gdf.head())
    # Perform analysis...

@workflow
def geospatial_analysis_workflow():
    # Assume 'some_source_gdf' is a GeoDataFrame output from a previous task or an external source
    some_source_gdf = process_geospatial_data() # From the previous example
    analyze_geospatial_data(input_gdf=some_source_gdf)
```

## Visualizing GeoDataFrame Summaries

The `GeoPandasDataFrameRenderer` provides a utility for generating an HTML representation of a `GeoDataFrame`'s descriptive statistics. This is particularly useful for debugging, quick data inspection, or for displaying summaries in user interfaces or reports.

The `to_html` method of the renderer takes a `GeoDataFrame` and returns an HTML string containing the output of the `describe()` method, which provides statistical summaries for numerical columns and basic information for other column types.

While not directly part of the data passing mechanism, this renderer enhances the observability of geospatial data within the system.

```python
import geopandas as gpd
from flytekitplugins.geopandas.gdf_transformers import GeoPandasDataFrameRenderer

# Assume 'my_gdf' is a GeoDataFrame instance
# my_gdf = gpd.read_file("path/to/some/geospatial_data.shp")

renderer = GeoPandasDataFrameRenderer()
html_summary = renderer.to_html(my_gdf)
# The html_summary string can then be embedded in a report or displayed in a web interface.
```

## Usage Considerations and Best Practices

*   **Direct Type Hinting:** Always use `geopandas.GeoDataFrame` as the type hint for task inputs and outputs to leverage the automatic serialization and deserialization capabilities.
*   **Performance with Large Datasets:** For very large `GeoDataFrame`s, Parquet is generally more performant for both writing and reading due to its columnar nature and compression capabilities. When dealing with external data, converting it to Parquet format before ingestion can improve workflow efficiency.
*   **Environment Dependencies:** Ensure that the execution environment for tasks processing `GeoDataFrame`s has all necessary geospatial libraries installed. This includes `geopandas` itself, along with its core dependencies like `fiona`, `shapely`, `pyproj`, and `rtree` (for spatial indexing). The `GeoPandasDecodingHandler`'s `read_file` fallback relies on these underlying libraries to handle diverse file formats.
*   **CRS Management:** `GeoDataFrame`s inherently manage Coordinate Reference Systems (CRS). Ensure that CRS information is correctly set and handled within your tasks to maintain data integrity across processing steps. The serialization process preserves CRS metadata.
<!--
key: summary_geopandas_integration_1ea83996-bbfa-45e4-a2c4-30f813e9c0fd
type: summary_end

-->
<!--
code_unit: flytekitplugins.geopandas.examples.geopandas_dataframe_example
code_unit_type: class
help_text: ''
key: example_9ec59c28-e27b-4659-bb85-3db6320f0f08
type: example

-->