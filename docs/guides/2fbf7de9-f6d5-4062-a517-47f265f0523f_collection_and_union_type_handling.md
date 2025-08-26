
<!--
help_text: ''
key: summary_collection_and_union_type_handling_3956e034-3af9-4464-a86d-0e193ed7b48c
modules:
- flytekit.core.type_engine.DictTransformer
- flytekit.core.type_engine.ListTransformer
- flytekit.core.type_engine.UnionTransformer
- flytekit.types.iterator.iterator.IteratorTransformer
- flytekit.types.iterator.iterator.FlyteIterator
- flytekit.types.iterator.json_iterator.JSONIteratorTransformer
- flytekit.types.iterator.json_iterator.JSONIterator
questions_to_answer: []
type: summary

-->
Collection and Union Type Handling

This section details how the system handles Python collection types, specifically iterators, for serialization and deserialization within the platform. It covers both generic iterator handling and specialized handling for JSON iterators, optimizing for different use cases and performance characteristics.

### Generic Iterator Handling

The system provides robust mechanisms for transforming standard Python iterators (`typing.Iterator`) into a platform-specific `LiteralCollection` and back. This allows for the seamless passing of iterable data between tasks.

#### `IteratorTransformer`

The `IteratorTransformer` is the core component responsible for converting generic Python iterators to `LiteralCollection` objects and for reconstructing Python iterators from `LiteralCollection` objects.

*   **Type Inference:** When a Python iterator type, such as `typing.Iterator[int]`, is encountered, the `get_literal_type` method extracts the inner type (e.g., `int`) and converts it to its corresponding `LiteralType`. This inner `LiteralType` then defines the `collection_type` within the overall `LiteralType` for the iterator.
*   **Serialization (`to_literal`):** To convert a Python iterator (`python_val`) into a `Literal`, the `to_literal` method iterates through each element of the `python_val`. For each element, it uses the `TypeEngine` to convert the element into its individual `Literal` representation, based on the inferred inner type. These individual `Literal` objects are then aggregated into a `LiteralCollection`.
*   **Deserialization (`to_python_value`):** When a `Literal` representing a collection is received, the `to_python_value` method extracts the underlying `LiteralCollection`. Instead of immediately converting all elements, it wraps the `LiteralCollection` within a `FlyteIterator`. This approach enables lazy loading and iteration, preventing the entire collection from being loaded into memory at once.

#### `FlyteIterator`

The `FlyteIterator` is a custom iterator implementation that provides a standard Pythonic iteration interface over a `LiteralCollection`. It acts as a proxy, allowing developers to consume the deserialized collection as if it were a native Python iterator.

*   **Lazy Loading:** The `FlyteIterator` does not materialize all elements of the `LiteralCollection` upfront. Instead, the `__next__` method retrieves and converts each `Literal` element to its Python value on demand, as iteration progresses. This is crucial for handling large collections efficiently.
*   **Standard Iterator Protocol:** It implements the `__len__`, `__iter__`, and `__next__` methods, ensuring compatibility with standard Python iteration constructs (e.g., `for` loops, `list()` conversion).

**Example Usage:**

```python
from typing import Iterator

def process_numbers(numbers: Iterator[int]) -> int:
    total = 0
    for num in numbers:
        total += num
    return total

# When 'numbers' is passed as an input, the system uses IteratorTransformer
# to convert it to a LiteralCollection, and then wraps it in a FlyteIterator
# when deserializing for the 'process_numbers' function.
```

### JSON Iterator Handling

For scenarios involving large datasets of JSON objects, the system provides specialized asynchronous handling for `Iterator[JSON]` types. This approach optimizes for streaming data by serializing iterators to and from JSON Lines (JSONL) files.

#### `JSONIteratorTransformer`

The `JSONIteratorTransformer` is an asynchronous type transformer designed specifically for `Iterator[JSON]`. It manages the conversion of a stream of JSON objects into a `Blob` (representing a JSONL file) and vice-versa.

*   **Format Specification:** It uses a predefined `JSON_ITERATOR_FORMAT` ("jsonl") and `JSON_ITERATOR_METADATA` to clearly identify the blob as a JSONL stream. The `get_literal_type` method reflects this by returning a `LiteralType` with a `BlobType` configured for single dimensionality and the specified format.
*   **Asynchronous Serialization (`async_to_literal`):**
    *   The method takes a Python `Iterator[JSON]` and writes each JSON object as a separate line to a temporary local JSONL file using the `jsonlines` library.
    *   After writing, the local file is asynchronously uploaded to the remote file store, and a `Literal` containing the `Blob` URI is returned.
    *   A `ValueError` is raised if the input iterator is empty, as an empty JSONL file might not be a valid representation for all downstream consumers.
*   **Asynchronous Deserialization (`async_to_python_value`):**
    *   The method retrieves the `Blob` URI from the incoming `Literal`.
    *   It then asynchronously downloads the JSONL file and opens it using the appropriate filesystem handler.
    *   A `jsonlines.Reader` is initialized from the file, and this reader is then wrapped in a `JSONIterator` for Pythonic consumption.
*   **Type Guessing (`guess_python_type`):** This method allows the system to infer that a `LiteralType` representing a `Blob` with the specific `jsonl` format and metadata should be deserialized back into an `Iterator[JSON]`.

#### `JSONIterator`

The `JSONIterator` is a custom iterator that provides a Pythonic interface for reading JSON objects from a `jsonlines.Reader`. It enables efficient, line-by-line processing of JSONL files.

*   **Stream Processing:** It wraps a `jsonlines.Reader` and exposes its `iter()` method, allowing for direct iteration over the JSON objects within the file.
*   **Resource Management:** Upon exhaustion of the iterator (when `StopIteration` is raised), the underlying file handler (`_reader`) is explicitly closed, ensuring proper resource cleanup.

**Example Usage:**

```python
from typing import Iterator, Dict, Any

JSON = Dict[str, Any] # Assuming JSON is defined as a dictionary

async def process_json_stream(data_stream: Iterator[JSON]) -> int:
    count = 0
    async for item in data_stream: # Note: The actual iteration is synchronous within the function
        # Process each JSON item
        print(f"Processing item: {item.get('id')}")
        count += 1
    return count

# When 'data_stream' is passed as an input, the system uses JSONIteratorTransformer
# to convert it to a Blob (JSONL file), and then wraps it in a JSONIterator
# when deserializing for the 'process_json_stream' function.
```

### Union Type Handling

Based on the provided code, the current implementation focuses exclusively on handling collection types, specifically `typing.Iterator` and `Iterator[JSON]`. There is no explicit mechanism detailed for handling `typing.Union` types within this specific context. If `Union` types were to be supported for collections, it would typically involve a more complex type resolution strategy within the `TypeEngine` to determine the concrete type of each element at runtime before applying the appropriate transformer.

### Common Use Cases and Best Practices

*   **Generic Iterators (`IteratorTransformer`):** Use for smaller collections or when the elements within the collection are of diverse types that can be individually transformed. This approach is suitable when the overhead of creating a `LiteralCollection` in memory is acceptable.
*   **JSON Iterators (`JSONIteratorTransformer`):** This is the preferred method for handling large datasets of structured JSON objects. It leverages file-based streaming (JSONL) to avoid memory constraints, making it highly efficient for data pipelines, ETL processes, and machine learning workflows that consume or produce large volumes of structured data.
*   **Performance Considerations:** For very large datasets, `JSONIteratorTransformer` offers significant performance advantages due to its streaming nature and reduced memory footprint compared to materializing an entire `LiteralCollection` in memory.
*   **Asynchronous Operations:** The `JSONIteratorTransformer` is an `AsyncTypeTransformer`, meaning its `to_literal` and `to_python_value` operations are asynchronous. This allows for non-blocking I/O operations when reading from or writing to remote storage, improving overall system responsiveness.
*   **Error Handling:** Be aware of the `ValueError` raised by `JSONIteratorTransformer` if an empty iterator is provided during serialization, as this indicates an empty data stream. `TypeTransformerFailedError` can occur if the literal structure does not match the expected type during deserialization.
<!--
key: summary_collection_and_union_type_handling_3956e034-3af9-4464-a86d-0e193ed7b48c
type: summary_end

-->
<!--
code_unit: flytekit.core.type_engine.DictTransformer
code_unit_type: class
help_text: ''
key: example_4d107cf2-7125-42ff-bd84-8309d5297a63
type: example

-->
<!--
code_unit: flytekit.core.type_engine.ListTransformer
code_unit_type: class
help_text: ''
key: example_d42aed98-814e-4416-acfa-7f8d849d5756
type: example

-->
<!--
code_unit: flytekit.core.type_engine.UnionTransformer
code_unit_type: class
help_text: ''
key: example_d9e5f341-e238-496b-9414-945a11d5c40f
type: example

-->
<!--
code_unit: flytekit.types.iterator.json_iterator.JSONIteratorTransformer
code_unit_type: class
help_text: ''
key: example_35d35fc5-5748-4e94-afde-0a98f661256a
type: example

-->