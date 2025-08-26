
<!--
help_text: ''
key: summary_huggingface_datasets_integration_4c283f6d-3621-4e22-9807-d8440308a243
modules:
- flytekitplugins.huggingface.sd_transformers
questions_to_answer: []
type: summary

-->
This documentation describes the integration of HuggingFace `datasets.Dataset` objects within the platform, enabling seamless data handling for machine learning workflows. This integration allows `datasets.Dataset` instances to be used as inputs and outputs for tasks, automatically managing their serialization, storage, and retrieval.

## Core Capabilities

The integration provides robust mechanisms for handling HuggingFace datasets, focusing on efficient storage and retrieval.

### Dataset Encoding

When a task produces a `datasets.Dataset` as an output, the system automatically encodes it into a `StructuredDataset` backed by Parquet files. This process is managed by the `HuggingFaceDatasetToParquetEncodingHandler`.

The encoding handler performs the following steps:
1.  **Type Conversion**: It recognizes `datasets.Dataset` objects as the source data type.
2.  **Column Filtering**: If a schema is specified for the `StructuredDataset` output, the handler filters the `datasets.Dataset` to include only the specified columns before saving. This is useful for selecting a subset of features or enforcing a specific output schema.
3.  **Local Storage**: The `datasets.Dataset` is converted and saved to Parquet format in a temporary local directory.
4.  **Remote Upload**: The local Parquet files are then uploaded to the platform's remote file store, making the dataset accessible across tasks and workflows.
5.  **Metadata Generation**: A `literals.StructuredDataset` object is returned, containing the URI to the remote Parquet files and metadata about the dataset's structure.

This ensures that `datasets.Dataset` objects are efficiently stored in a columnar format, optimizing for subsequent reads and analytical operations.

### Dataset Decoding

When a task consumes a `StructuredDataset` that originated from a `datasets.Dataset` (and is stored as Parquet), the system automatically decodes it back into a `datasets.Dataset` object. This process is handled by the `ParquetToHuggingFaceDatasetDecodingHandler`.

The decoding handler performs these actions:
1.  **Remote Download**: It retrieves the Parquet files from the specified remote URI to a temporary local directory.
2.  **Dataset Reconstruction**: The local Parquet files are loaded into a `datasets.Dataset` object using the `from_parquet` method.
3.  **Column Selection**: If the `StructuredDataset` metadata includes column information, the decoder can load only the specified columns, which can improve performance and reduce memory usage when only a subset of the data is required.

This automatic decoding simplifies data access, allowing developers to work directly with `datasets.Dataset` objects within their task logic without managing file I/O.

### Dataset Visualization

For improved user experience, the integration includes a `HuggingFaceDatasetRenderer`. This component provides a basic HTML representation of a `datasets.Dataset` object. When a `datasets.Dataset` is an output of a task, its string representation is converted to HTML, making it viewable directly within the platform's UI. This offers a quick way to inspect dataset contents and structure without needing to download or process the data externally.

## Usage Patterns

Integrating HuggingFace datasets into workflows is straightforward, leveraging the platform's type system.

### Defining `datasets.Dataset` as Task Inputs/Outputs

To use a `datasets.Dataset` in a task, declare it as a type hint for inputs or outputs. The platform's type engine automatically handles the serialization and deserialization using the handlers described above.

```python
import datasets
from flytekit import task, workflow
from typing import Annotated

@task
def create_hf_dataset() -> datasets.Dataset:
    """
    Creates a simple HuggingFace dataset.
    """
    data = {"text": ["hello world", "flyte is awesome"], "label": [0, 1]}
    return datasets.Dataset.from_dict(data)

@task
def process_hf_dataset(input_dataset: datasets.Dataset) -> datasets.Dataset:
    """
    Processes the input HuggingFace dataset (e.g., adds a new column).
    """
    def add_length(example):
        example["text_length"] = len(example["text"])
        return example
    return input_dataset.map(add_length)

@task
def analyze_hf_dataset(processed_dataset: datasets.Dataset):
    """
    Analyzes the processed HuggingFace dataset.
    """
    print(f"Dataset features: {processed_dataset.features}")
    print(f"First row: {processed_dataset[0]}")
    print(f"Average text length: {sum(processed_dataset['text_length']) / len(processed_dataset)}")

@workflow
def hf_dataset_workflow():
    initial_dataset = create_hf_dataset()
    processed_dataset = process_hf_dataset(input_dataset=initial_dataset)
    analyze_hf_dataset(processed_dataset=processed_dataset)
```

### Working with Column Subsets

When defining a `StructuredDataset` output, you can specify the desired columns. This allows for schema enforcement and efficient data transfer by only storing or loading relevant columns.

```python
import datasets
from flytekit import task, workflow
from flytekit.types.structured.structured_dataset import StructuredDataset
from flytekit.types.schema import FlyteSchema
from typing import Annotated

# Define a schema for the output dataset
# This schema will be used to filter columns during encoding and decoding
MyDatasetSchema = FlyteSchema[
    {"text": str, "label": int}
]

@task
def create_and_filter_hf_dataset() -> Annotated[datasets.Dataset, MyDatasetSchema]:
    """
    Creates a HuggingFace dataset with extra columns, but only 'text' and 'label'
    will be stored due to the schema annotation.
    """
    data = {
        "text": ["example A", "example B"],
        "label": [0, 1],
        "extra_info": ["info1", "info2"]
    }
    return datasets.Dataset.from_dict(data)

@task
def consume_filtered_hf_dataset(input_dataset: Annotated[datasets.Dataset, MyDatasetSchema]):
    """
    Consumes the dataset. Only 'text' and 'label' columns will be available.
    'extra_info' will not be loaded.
    """
    print(f"Consumed dataset features: {input_dataset.features}")
    # This will raise an error if 'extra_info' was not filtered out during encoding
    # print(input_dataset['extra_info'])

@workflow
def hf_filtered_workflow():
    filtered_dataset = create_and_filter_hf_dataset()
    consume_filtered_hf_dataset(input_dataset=filtered_dataset)
```
In this example, even though `create_and_filter_hf_dataset` creates a dataset with `extra_info`, the `MyDatasetSchema` annotation ensures that only `text` and `label` columns are persisted to Parquet and subsequently loaded by `consume_filtered_hf_dataset`.

## Architectural Details

The integration relies on the platform's `StructuredDataset` type, which provides a generic interface for handling tabular data. `datasets.Dataset` objects are mapped to this `StructuredDataset` type, with Parquet chosen as the underlying storage format due to its efficiency for columnar data.

The `HuggingFaceDatasetToParquetEncodingHandler` and `ParquetToHuggingFaceDatasetDecodingHandler` are registered with the platform's type transformer system. This registration allows the platform to automatically invoke these handlers whenever a `datasets.Dataset` type is encountered during task execution, abstracting away the complexities of data serialization and deserialization.

## Considerations and Best Practices

*   **Performance with Large Datasets**: For very large `datasets.Dataset` objects, the local download and upload steps can incur significant I/O overhead. Ensure sufficient local disk space and network bandwidth for efficient data transfer.
*   **Schema Definition**: While not strictly required, defining a `FlyteSchema` for `datasets.Dataset` outputs is a best practice. It enables column filtering during encoding and decoding, which can save storage space and improve performance by only transferring and loading necessary data. It also provides clear documentation of the dataset's structure.
*   **Parquet Format**: The current implementation exclusively uses Parquet for storage. This choice is optimized for columnar data access and analytical workloads.
*   **Renderer Limitations**: The `HuggingFaceDatasetRenderer` provides a basic string-based HTML representation. For complex dataset exploration or interactive visualization, consider integrating with external tools or custom rendering solutions.
*   **Data Versioning**: When `datasets.Dataset` objects are used as outputs, they are automatically versioned by the platform's data catalog. This ensures reproducibility and traceability of data artifacts across workflow executions.
<!--
key: summary_huggingface_datasets_integration_4c283f6d-3621-4e22-9807-d8440308a243
type: summary_end

-->
<!--
code_unit: flytekitplugins.huggingface.examples.huggingface_dataset_example
code_unit_type: class
help_text: ''
key: example_64d1de45-e02c-4d53-89f4-7f817541a562
type: example

-->