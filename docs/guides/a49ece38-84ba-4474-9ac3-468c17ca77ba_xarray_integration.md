
<!--
help_text: ''
key: summary_xarray_integration_6c43b805-2dfb-4e88-9524-35655aef5dd5
modules:
- flytekitplugins.xarray.xarray_transformers
questions_to_answer: []
type: summary

-->
The Xarray integration enables seamless handling of Xarray objects within workflows. This capability allows developers to define tasks that consume or produce `xarray.DataArray` and `xarray.Dataset` objects, treating them as native Python types. This is particularly beneficial for scientific computing and data analysis workflows that heavily rely on N-dimensional labeled arrays.

### Supported Xarray Types

The integration supports two primary Xarray data structures:

*   `xarray.DataArray`: A labeled, N-dimensional array.
*   `xarray.Dataset`: A dictionary-like container for `DataArray` objects, with aligned dimensions.

### Data Handling and Serialization

Under the hood, Xarray objects are serialized to the Zarr format. Zarr is a cloud-native, chunked, compressed, N-dimensional array storage format, making it highly efficient for large datasets. When an Xarray object is passed as an input or output, it is converted into a multipart binary blob and stored remotely. Upon retrieval, the blob is deserialized back into the corresponding Xarray object.

This process is managed by dedicated type transformers:

*   The `XarrayDaZarrTypeTransformer` handles `xarray.DataArray` objects.
*   The `XarrayZarrTypeTransformer` handles `xarray.Dataset` objects.

A key feature of these transformers is their implicit handling of Dask clients during serialization. When an Xarray object is written to Zarr, a Dask client is temporarily attached. This simplifies the user experience by eliminating the need for explicit Dask client connections within tasks that receive Xarray objects, especially when those objects are backed by Dask arrays.

### Practical Usage

To use Xarray objects in your workflows, simply type-hint them in your task definitions. The integration automatically handles the serialization and deserialization.

```python
import xarray as xr
import numpy as np
from flytekit import task, workflow

@task
def create_dataset_task() -> xr.Dataset:
    """Creates a sample xarray Dataset."""
    data = np.random.rand(10, 5)
    coords = {"time": np.arange(10), "lat": np.arange(5)}
    ds = xr.Dataset({"temperature": (("time", "lat"), data)}, coords=coords)
    return ds

@task
def process_dataset_task(input_ds: xr.Dataset) -> xr.DataArray:
    """Processes the dataset and returns a DataArray."""
    # Perform some computation, e.g., calculate mean temperature
    mean_temp = input_ds["temperature"].mean(dim="lat")
    return mean_temp

@task
def analyze_dataarray_task(input_da: xr.DataArray):
    """Analyzes the DataArray."""
    print(f"Mean temperature over time: {input_da.mean().item()}")

@workflow
def xarray_workflow():
    ds = create_dataset_task()
    da = process_dataset_task(input_ds=ds)
    analyze_dataarray_task(input_da=da)
```

In this example:

*   `create_dataset_task` produces an `xr.Dataset`.
*   `process_dataset_task` consumes an `xr.Dataset` and produces an `xr.DataArray`.
*   `analyze_dataarray_task` consumes an `xr.DataArray`.

The system automatically manages the transfer and conversion of these Xarray objects between tasks.

### Advanced Considerations and Best Practices

#### Performance with Large Datasets

The choice of Zarr as the underlying storage format is crucial for performance. Zarr's chunking and compression capabilities allow for efficient storage and retrieval of large N-dimensional arrays. When working with very large Xarray objects, consider:

*   **Chunking Strategy:** Optimize the chunking of your Xarray objects (or underlying Dask arrays) to align with your access patterns. This can significantly reduce I/O overhead.
*   **Distributed Computing:** For computationally intensive tasks on large Xarray objects, leverage Dask's distributed capabilities. While the type transformers handle Dask client attachment during serialization, you might need to explicitly configure and connect to a Dask cluster within your tasks for parallel processing.

#### HTML Representation

When viewing task outputs in the UI, Xarray objects benefit from rich HTML representations. The type transformers provide a `to_html` method that renders the Xarray object's default HTML representation, offering a convenient way to inspect data directly in the console or UI.

### Limitations

Currently, the Xarray integration primarily supports Zarr as the serialization format. While Zarr is highly versatile and performant for most use cases, other Xarray-compatible formats (e.g., NetCDF, Parquet) are not directly supported for automatic serialization/deserialization. If you need to work with these formats, you would typically handle the file I/O within your tasks using standard Python libraries.
<!--
key: summary_xarray_integration_6c43b805-2dfb-4e88-9524-35655aef5dd5
type: summary_end

-->
<!--
code_unit: flytekitplugins.xarray.examples.xarray_dataset_example
code_unit_type: class
help_text: ''
key: example_1c30e3bc-7c29-4a3a-9813-759654e743f8
type: example

-->