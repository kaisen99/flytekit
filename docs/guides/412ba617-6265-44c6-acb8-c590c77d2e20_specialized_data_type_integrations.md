
<!--
help_text: ''
key: summary_specialized_data_type_integrations_4f20c8e4-1efb-4dea-81e6-4429f54c7b46
modules:
- flytekit.types.numpy.ndarray
- flytekit.types.file.image
- flytekit.types.iterator.iterator
- flytekit.types.iterator.json_iterator
questions_to_answer: []
type: summary

-->
# Specialized Data Type Integrations

Specialized data type integrations enable seamless handling of complex and domain-specific data structures within workflows. These integrations provide automatic serialization and deserialization, allowing developers to use native Python types like NumPy arrays, PIL images, and iterators directly as task inputs and outputs. This simplifies development by abstracting away the underlying data transfer mechanisms.

## NumPy Array Integration

The NumPy array integration allows tasks to exchange `numpy.ndarray` objects directly. This is crucial for machine learning and scientific computing workflows that heavily rely on numerical data.

### How it Works

The `NumpyArrayTransformer` class manages the conversion of `numpy.ndarray` objects. When a NumPy array is passed as an output from a task, the transformer serializes it into a `.npy` file. This file is then uploaded to remote storage. Conversely, when a task expects a `numpy.ndarray` as an input, the transformer downloads the `.npy` file from remote storage and deserializes it back into a `numpy.ndarray` object.

This integration supports `allow_pickle` and `mmap_mode` metadata, providing flexibility for handling various NumPy array configurations, including those containing Python objects or requiring memory-mapped access.

### Capabilities

-   **Automatic Serialization/Deserialization:** Seamlessly pass `numpy.ndarray` objects between tasks without manual file handling.
-   **Metadata Support:** Configure `allow_pickle` and `mmap_mode` for advanced array handling.

### Usage Example

```python
import numpy as np
from flytekit import task, workflow

@task
def create_random_array(rows: int, cols: int) -> np.ndarray:
    """
    Creates a random NumPy array of specified dimensions.
    """
    return np.random.rand(rows, cols)

@task
def calculate_array_sum(arr: np.ndarray) -> float:
    """
    Calculates the sum of all elements in a NumPy array.
    """
    return arr.sum()

@workflow
def numpy_workflow(rows: int = 10, cols: int = 10) -> float:
    """
    A workflow demonstrating NumPy array integration.
    """
    my_array = create_random_array(rows=rows, cols=cols)
    total_sum = calculate_array_sum(arr=my_array)
    return total_sum
```

### Considerations

Transferring large NumPy arrays incurs network overhead and disk I/O for serialization and deserialization. Optimize array sizes and consider data partitioning for very large datasets to improve performance.

## PIL Image Integration

The PIL Image integration enables direct use of `PIL.Image.Image` objects as inputs and outputs for tasks. This is particularly useful for computer vision and image processing workflows.

### How it Works

The `PILImageTransformer` class handles the conversion of `PIL.Image.Image` objects. When an image is produced by a task, the transformer saves it as a `.png` file (or another suitable format) to local storage, then uploads it to remote storage. For task inputs, the transformer downloads the image file and loads it back into a `PIL.Image.Image` object.

### Capabilities

-   **Native Image Handling:** Use `PIL.Image.Image` objects directly in task signatures.
-   **Automatic Visualization:** The `to_html` method provides built-in support for rendering images directly within the UI, which is invaluable for debugging and visualizing intermediate image processing results.

### Usage Example

```python
from flytekit import task, workflow
from PIL import Image
import io

@task
def generate_simple_image(width: int, height: int, color: str) -> Image.Image:
    """
    Generates a simple colored image.
    """
    img = Image.new('RGB', (width, height), color=color)
    return img

@task
def get_image_dimensions(img: Image.Image) -> tuple[int, int]:
    """
    Returns the width and height of an image.
    """
    return img.width, img.height

@workflow
def image_processing_workflow(width: int = 100, height: int = 100) -> tuple[int, int]:
    """
    A workflow demonstrating PIL Image integration.
    """
    red_image = generate_simple_image(width=width, height=height, color='red')
    dimensions = get_image_dimensions(img=red_image)
    return dimensions
```

### Best Practices

Leverage the automatic UI visualization for `PIL.Image.Image` outputs to quickly inspect image transformations and verify task correctness.

## Generic Iterator Integration

The generic iterator integration allows tasks to accept and return `typing.Iterator` objects. This is useful for passing sequences of data where the elements are standard Flyte-supported types.

### How it Works

The `IteratorTransformer` class converts a Python `typing.Iterator` into a `LiteralCollection`. This means all elements yielded by the iterator are collected into a list of `Literal` objects before being transferred. On the receiving end, the `FlyteIterator` class wraps this `LiteralCollection`, providing an iterator interface that lazily converts `Literal` elements back to their Python values as they are consumed.

### Capabilities

-   **Sequence Handling:** Pass sequences of data where the individual elements are recognized Flyte types.
-   **Type Inference:** Automatically infers the type of elements within the iterator.

### Usage Example

```python
from flytekit import task, workflow
from typing import Iterator

@task
def generate_sequence(start: int, end: int) -> Iterator[int]:
    """
    Generates a sequence of integers.
    """
    for i in range(start, end + 1):
        yield i

@task
def sum_sequence(numbers: Iterator[int]) -> int:
    """
    Sums all numbers in the provided iterator.
    """
    total = 0
    for num in numbers:
        total += num
    return total

@workflow
def sequence_workflow(start_num: int = 1, end_num: int = 5) -> int:
    """
    A workflow demonstrating generic iterator integration.
    """
    numbers_iterator = generate_sequence(start=start_num, end=end_num)
    result_sum = sum_sequence(numbers=numbers_iterator)
    return result_sum
```

### Limitations

The generic iterator integration materializes the entire iterator into memory as a `LiteralCollection` during serialization. This approach is not suitable for very large or infinite iterators, as it can lead to high memory consumption and performance bottlenecks. For streaming large datasets, especially structured records, consider using the JSON Iterator integration.

## JSON Iterator Integration

The JSON Iterator integration provides an efficient mechanism for streaming large collections of JSON objects. This is ideal for processing big data where full in-memory materialization is impractical.

### How it Works

The `JSONIteratorTransformer` class handles `Iterator[JSON]` (where `JSON` is typically `typing.Dict[str, typing.Any]`). When an iterator of JSON objects is produced, the transformer writes each object as a separate line to a JSON Lines (`.jsonl`) file. This file is then uploaded to remote storage. When a task consumes an `Iterator[JSON]`, the transformer downloads the `.jsonl` file and provides a `JSONIterator` object. The `JSONIterator` streams JSON objects from the file on demand, avoiding the need to load the entire dataset into memory.

### Capabilities

-   **Streaming Large Datasets:** Process vast amounts of structured JSON data without memory constraints.
-   **Efficient I/O:** Leverages the JSON Lines format for optimized record-by-record reading and writing.

### Usage Example

```python
from flytekit import task, workflow
from typing import Iterator, Dict, Any

# Define a type alias for JSON objects for clarity
JSON = Dict[str, Any]

@task
def generate_user_data(num_users: int) -> Iterator[JSON]:
    """
    Generates an iterator of mock user data.
    """
    for i in range(num_users):
        yield {"user_id": i + 1, "name": f"User_{i+1}", "email": f"user{i+1}@example.com"}

@task
def count_active_users(user_data: Iterator[JSON]) -> int:
    """
    Counts users based on a hypothetical 'is_active' field (demonstrates iteration).
    """
    active_count = 0
    for user in user_data:
        # In a real scenario, you might check user["is_active"]
        print(f"Processing user: {user['name']}")
        active_count += 1
    return active_count

@workflow
def process_large_json_data(num_records: int = 1000) -> int:
    """
    A workflow demonstrating JSON Iterator integration for large datasets.
    """
    data_stream = generate_user_data(num_users=num_records)
    total_users = count_active_users(user_data=data_stream)
    return total_users
```

### Best Practices

-   Use the JSON Iterator integration for processing large, sequential datasets of structured records.
-   Ensure that the `JSON` type alias is consistently defined (e.g., `Dict[str, Any]`) in your tasks.
-   Avoid operations that require random access or full materialization of the dataset within tasks consuming a `JSONIterator`, as this negates the benefits of streaming.

### Performance Considerations

This integration is optimized for performance with large datasets by minimizing in-memory data transfer. Data is streamed directly from remote storage, reducing memory footprint and improving execution efficiency for data-intensive workflows.
<!--
key: summary_specialized_data_type_integrations_4f20c8e4-1efb-4dea-81e6-4429f54c7b46
type: summary_end

-->
<!--
code_unit: flytekit.types.numpy.ndarray.NumpyArrayTransformer
code_unit_type: class
help_text: ''
key: example_2d672fe3-678f-45a2-80da-606399e04577
type: example

-->
<!--
code_unit: flytekit.types.file.image.PILImageTransformer
code_unit_type: class
help_text: ''
key: example_6177f4e1-e58f-4dbb-a55f-ae1f53ab8823
type: example

-->
<!--
code_unit: flytekit.types.iterator.iterator.IteratorTransformer
code_unit_type: class
help_text: ''
key: example_cf316ce9-a8a6-49a0-86f8-ba2758fe400a
type: example

-->
<!--
code_unit: flytekit.types.iterator.iterator.FlyteIterator
code_unit_type: class
help_text: ''
key: example_057a92b9-4f26-4f90-861f-25df4f65e764
type: example

-->
<!--
code_unit: flytekit.types.iterator.json_iterator.JSONIteratorTransformer
code_unit_type: class
help_text: ''
key: example_967d6337-4f8a-4f33-aa17-fa55ede4fed1
type: example

-->
<!--
code_unit: flytekit.types.iterator.json_iterator.JSONIterator
code_unit_type: class
help_text: ''
key: example_3ca4d6d9-0d55-4ad2-affd-c0ae8b10bf01
type: example

-->