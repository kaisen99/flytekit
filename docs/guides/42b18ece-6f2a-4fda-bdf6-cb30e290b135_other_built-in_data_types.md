
<!--
help_text: ''
key: summary_other_built-in_data_types_0e8cbc5a-0037-403f-a588-f127c5050ad1
modules:
- flytekit.types.numpy.ndarray
- flytekit.types.pickle.pickle
- flytekit.types.error.error
- flytekit.types.iterator.iterator
- flytekit.types.iterator.json_iterator
questions_to_answer: []
type: summary

-->
# Other Built-in Data Types

Flytekit extends its type system beyond standard Python primitives to support specialized data types crucial for machine learning and data processing workflows. These types are handled by dedicated type transformers that manage their serialization and deserialization to and from Flyte's underlying literal formats, often leveraging blob storage for efficient data transfer.

## Numpy Arrays

Flytekit provides native support for `numpy.ndarray` objects, enabling seamless passing of numerical arrays between tasks.

### Capabilities

The `NumpyArrayTransformer` handles the conversion of `numpy.ndarray` instances. When a NumPy array is used as a task input or output, it is automatically serialized to the `.npy` file format and stored as a single blob. During deserialization, the `.npy` file is loaded back into a `numpy.ndarray` object.

This transformer supports additional metadata for `np.load` and `np.save` operations:

*   `allow_pickle`: Controls whether object arrays can be pickled. Defaults to `False` for security.
*   `mmap_mode`: Specifies the memory-mapping mode for loading large arrays.

### Usage

Declare `numpy.ndarray` as a type hint for task inputs or outputs:

```python
import numpy as np
from flytekit import task, workflow

@task
def process_array(arr: np.ndarray) -> np.ndarray:
    return arr * 2

@workflow
def my_workflow(input_array: np.ndarray) -> np.ndarray:
    return process_array(arr=input_array)
```

To specify `allow_pickle` or `mmap_mode`, use `Annotated` from `typing_extensions`:

```python
from typing_extensions import Annotated
import numpy as np
from flytekit import task

@task
def load_pickled_array(arr: Annotated[np.ndarray, {"allow_pickle": True}]) -> np.ndarray:
    # This task can load an array that was saved with allow_pickle=True
    return arr
```

### Considerations

*   **Security:** Setting `allow_pickle=True` can introduce security vulnerabilities if the source of the NumPy array is untrusted, as it allows arbitrary code execution during deserialization. Use with caution.
*   **Performance:** For very large arrays, consider `mmap_mode` to avoid loading the entire array into memory.

## Pickled Python Objects

Flytekit offers a fallback mechanism for Python objects that do not have a dedicated type transformer. These objects are handled by the `FlytePickleTransformer`, which serializes them using `cloudpickle`.

### Capabilities

The `FlytePickleTransformer` converts any Python object that `cloudpickle` can serialize into a blob with the `PythonPickle` format. This allows complex or custom Python objects, including functions, classes, and instances, to be passed between tasks.

The `FlytePickle` class is an internal representation. It manages the serialization to and deserialization from a remote blob storage location.

### Usage

Developers typically do not explicitly declare `FlytePickle` as a type hint. Instead, Flytekit automatically infers the use of the pickle transformer when it encounters a Python type for which no other specific transformer is registered.

For example, if you define a custom class:

```python
from flytekit import task, workflow

class MyCustomObject:
    def __init__(self, value):
        self.value = value

    def __repr__(self):
        return f"MyCustomObject({self.value})"

@task
def create_custom_object() -> MyCustomObject:
    return MyCustomObject(42)

@task
def process_custom_object(obj: MyCustomObject) -> str:
    return f"Processed: {obj.value}"

@workflow
def custom_object_workflow() -> str:
    obj = create_custom_object()
    return process_custom_object(obj=obj)
```

In this scenario, `MyCustomObject` will be implicitly handled by the `FlytePickleTransformer`.

### Considerations

*   **Portability:** Pickling can lead to portability issues. A pickled object might not deserialize correctly if the Python version, operating system, or library versions differ between the serialization and deserialization environments.
*   **Security:** Deserializing pickled data from untrusted sources is a security risk, as it can lead to arbitrary code execution.
*   **Performance:** Pickling large objects can be slow and consume significant memory.
*   **Maintainability:** Relying heavily on pickling can make debugging and understanding data flow more challenging compared to using explicitly defined, structured types.

## Flyte Errors

The `FlyteError` type provides a structured way to represent errors within Flyte workflows, particularly useful for defining inputs to failure handling tasks.

### Capabilities

The `ErrorTransformer` maps the Python `FlyteError` object to Flyte's `LiteralType.ERROR`. A `FlyteError` object encapsulates a `message` (the error description) and a `failed_node_id` (the identifier of the node that failed).

When a task fails, Flyte can propagate this structured error information to a designated failure task, allowing for more granular error handling and reporting.

### Usage

To define a task that receives error information from a failed upstream node:

```python
from flytekit import task, workflow
from flytekit.types.error import FlyteError

@task
def failing_task(x: int) -> int:
    if x < 0:
        raise ValueError("Input must be non-negative")
    return x * 2

@task
def error_handler_task(error: FlyteError):
    print(f"Workflow failed! Error message: {error.message}")
    print(f"Failed node ID: {error.failed_node_id}")

@workflow
def my_error_workflow(x: int):
    failing_task(x=x) # This task might fail
    # In a real workflow, this error_handler_task would be linked to the failure of failing_task
    # For example, using a failure node in the workflow definition.
```

### Considerations

*   `FlyteError` is specifically designed for Flyte's error propagation mechanism. It is not a general-purpose exception class.
*   The `failed_node_id` provides context about where the failure originated within the workflow graph.

## Iterators

Flytekit supports Python iterators (`typing.Iterator`) as task inputs and outputs, allowing for processing sequences of data.

### Capabilities

The `IteratorTransformer` handles generic `typing.Iterator` types. When an iterator is passed as an output, its contents are fully materialized into a `LiteralCollection` (a list of literals) during serialization. Each element within the iterator is converted to its corresponding literal type.

When an iterator is received as an input, the `FlyteIterator` class provides an iterator interface over the deserialized `LiteralCollection`. This means the entire collection is available in memory, and `FlyteIterator` simply provides a Pythonic way to traverse it.

### Usage

Declare `typing.Iterator` with a specific element type:

```python
from typing import Iterator
from flytekit import task, workflow

@task
def generate_numbers(count: int) -> Iterator[int]:
    for i in range(count):
        yield i

@task
def sum_numbers(numbers: Iterator[int]) -> int:
    total = 0
    for num in numbers:
        total += num
    return total

@workflow
def iterator_workflow(count: int) -> int:
    nums = generate_numbers(count=count)
    return sum_numbers(numbers=nums)
```

### Considerations

*   **Memory Usage:** The entire content of the iterator is materialized into memory as a list during serialization. This can be inefficient and lead to out-of-memory errors for very large iterators. For large datasets, consider using `Iterator[JSON]` or other streaming-optimized types.
*   **Lazy Evaluation:** While `FlyteIterator` provides an iterator interface, the underlying data is already fully loaded. It does not offer true lazy evaluation across task boundaries.

## JSON Line Iterators

For handling large streams of JSON objects, Flytekit provides specialized support for `Iterator[JSON]`, which leverages the JSON Lines (JSONL) format.

### Capabilities

The `JSONIteratorTransformer` is optimized for streaming JSON data. When an `Iterator[JSON]` is used as an output, the `JSONIteratorTransformer` writes each JSON object from the iterator into a JSONL file, which is then stored as a blob. This avoids materializing the entire dataset in memory.

When an `Iterator[JSON]` is received as an input, the `JSONIteratorTransformer` downloads the JSONL blob and provides a `JSONIterator` that streams the JSON objects directly from the file. This enables efficient processing of large datasets without loading everything into memory.

### Usage

Declare `typing.Iterator[JSON]` for streaming JSON data:

```python
from typing import Iterator
from flytekit import task, workflow
from flytekit.types.json import JSON

@task
def generate_json_data(count: int) -> Iterator[JSON]:
    for i in range(count):
        yield {"id": i, "value": f"item_{i}"}

@task
def process_json_data(data_stream: Iterator[JSON]) -> int:
    total_items = 0
    for item in data_stream:
        print(f"Processing item: {item['id']}")
        total_items += 1
    return total_items

@workflow
def json_iterator_workflow(count: int) -> int:
    json_stream = generate_json_data(count=count)
    return process_json_data(data_stream=json_stream)
```

### Considerations

*   **Format Specificity:** This type specifically handles data in the JSON Lines format, where each line in the file is a valid JSON object.
*   **Efficiency for Large Data:** This is the preferred method for passing large collections of JSON objects between tasks, as it minimizes memory footprint by streaming data directly from storage.
*   **Empty Iterators:** An empty iterator will raise a `ValueError` during serialization, as the `JSONIteratorTransformer` expects at least one item.
<!--
key: summary_other_built-in_data_types_0e8cbc5a-0037-403f-a588-f127c5050ad1
type: summary_end

-->