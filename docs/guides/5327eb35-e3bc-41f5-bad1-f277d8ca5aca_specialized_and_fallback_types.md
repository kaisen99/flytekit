
<!--
help_text: ''
key: summary_specialized_and_fallback_types_65a997d9-0c92-40ed-82e6-c12835e5d18b
modules:
- flytekit.types.error.error.FlyteError
- flytekit.types.error.error.ErrorTransformer
- flytekit.types.pickle.pickle.FlytePickle
- flytekit.types.pickle.pickle.FlytePickleTransformer
- flytekit.types.numpy.ndarray.NumpyArrayTransformer
questions_to_answer: []
type: summary

-->
## Specialized and Fallback Types

Flyte's type system efficiently handles data passing between tasks and workflows. It achieves this through a combination of *specialized types*, which offer optimized handling for common data structures, and *fallback types*, which provide a robust mechanism for any Python object not explicitly supported.

### Specialized Types

Specialized types are Python classes that have a direct, optimized mapping to Flyte's internal literal types. This ensures efficient serialization, deserialization, and often, better interoperability with the underlying execution engine.

#### Error Handling with `FlyteError`

The `FlyteError` class provides a structured way to represent and propagate errors within Flyte workflows. It is specifically designed for use in failure nodes, allowing workflows to capture detailed error information from a failed task.

*   **Purpose:** `FlyteError` captures a `message` describing the error and the `failed_node_id` where the error originated. This structured information is crucial for building robust error handling and recovery logic in workflows.
*   **Usage in Workflows:** When a task fails, the Flyte engine can pass an instance of `FlyteError` to a designated failure task. To receive this error, the failure task must define an input parameter explicitly typed as `FlyteError`.

    ```python
    from flytekit.core.type_engine import FlyteError
    from flytekit import task, workflow

    @task
    def failing_task() -> int:
        raise ValueError("Something went wrong in the task!")

    @task
    def error_handler_task(error: FlyteError):
        print(f"Workflow failed! Error message: {error.message}")
        print(f"Failed node ID: {error.failed_node_id}")

    @workflow
    def my_workflow():
        failing_node = failing_task()
        # In a real workflow, you would configure a failure handler
        # that passes the error to error_handler_task.
        # For demonstration, imagine error_handler_task is called on failure.
        # error_handler_task(error=failing_node.error) # Conceptual usage
    ```

*   **Underlying Mechanism:** The `ErrorTransformer` handles the conversion between a Python `FlyteError` object and Flyte's internal `LiteralType.ERROR`. This transformer ensures that `FlyteError` instances are correctly serialized and deserialized across the system.

#### Numerical Data with NumPy Arrays

Flyte provides native support for `numpy.ndarray` objects, enabling efficient handling of numerical data commonly used in data science and machine learning workloads.

*   **Purpose:** The `NumpyArrayTransformer` allows `np.ndarray` instances to be passed directly as inputs and outputs of tasks without manual serialization. This avoids the overhead and potential issues associated with generic pickling for large arrays.
*   **Usage:** Simply type hint your task inputs and outputs with `numpy.ndarray`.

    ```python
    import numpy as np
    from flytekit import task, workflow

    @task
    def process_array(input_array: np.ndarray) -> np.ndarray:
        print(f"Processing array of shape: {input_array.shape}")
        return input_array * 2

    @workflow
    def numpy_workflow() -> np.ndarray:
        initial_array = np.array([1, 2, 3, 4, 5])
        processed = process_array(input_array=initial_array)
        return processed
    ```

*   **Serialization Format:** `NumpyArrayTransformer` serializes `np.ndarray` objects into a binary format (`.npy` files) and stores them as single-dimensional blobs. This format is optimized for NumPy arrays, ensuring efficient storage and retrieval.
*   **Considerations:** When dealing with very large arrays, consider the memory footprint during task execution and the network bandwidth required for transferring the data. The `allow_pickle` and `mmap_mode` options can be configured via metadata if needed for advanced scenarios, though typically not required for basic usage.

### Fallback Types

Fallback types provide a universal mechanism for handling any Python object that does not have a dedicated specialized type transformer. This ensures that Flyte can process a wide range of custom or complex data structures, albeit with certain trade-offs.

#### Generic Python Objects with `FlytePickle`

The `FlytePickle` type serves as Flyte's primary fallback mechanism. When Flyte encounters a Python type for which no specific `TypeTransformer` is registered, it defaults to using `FlytePickle` to serialize the object.

*   **Purpose:** `FlytePickle` ensures that virtually any Python object can be passed between tasks, even if Flyte does not have explicit knowledge of its structure. It acts as a catch-all for unsupported types.
*   **Internal Use Only:** Developers should **not** explicitly type hint their task inputs or outputs with `FlytePickle`. This type is used internally by Flytekit. When you pass an object of an unrecognized type, Flytekit automatically wraps it with `FlytePickle` behind the scenes.
*   **Serialization Mechanism:** The `FlytePickleTransformer` uses `cloudpickle` to serialize the Python object into a byte stream. This stream is then stored as a single-dimensional blob in remote storage. When the object is needed by a downstream task, the blob is downloaded and deserialized using `cloudpickle`.

    ```python
    from flytekit import task, workflow
    import collections

    # Define a custom, complex Python object
    class MyCustomObject:
        def __init__(self, name: str, data: dict):
            self.name = name
            self.data = data

        def __repr__(self):
            return f"MyCustomObject(name='{self.name}', data={self.data})"

    @task
    def produce_custom_object() -> MyCustomObject:
        return MyCustomObject("example", {"key": [1, 2, 3], "value": "test"})

    @task
    def consume_custom_object(obj: MyCustomObject):
        # Flytekit automatically handles serialization/deserialization
        # of MyCustomObject using FlytePickle
        print(f"Received custom object: {obj}")
        print(f"Object name: {obj.name}")
        print(f"Object data: {obj.data}")

    @workflow
    def custom_object_workflow():
        custom_obj = produce_custom_object()
        consume_custom_object(obj=custom_obj)
    ```

*   **Limitations and Best Practices:**
    *   **Performance:** Pickling can be less performant than specialized serialization methods, especially for large objects, due to the overhead of serialization/deserialization and potential data transfer sizes.
    *   **Interoperability:** Pickled objects are highly coupled to the Python environment (version, installed libraries) in which they were created. Deserializing a pickled object in a different environment can lead to compatibility issues or errors.
    *   **Security:** Deserializing untrusted pickled data can pose security risks.
    *   **Debugging:** Debugging issues with pickled data can be more challenging as the internal structure is opaque to Flyte's type system.
    *   **Recommendation:** While `FlytePickle` is convenient for rapid prototyping or handling truly unique types, it is generally recommended to use or create specialized transformers for frequently used or critical data types to leverage better performance, interoperability, and type safety.
<!--
key: summary_specialized_and_fallback_types_65a997d9-0c92-40ed-82e6-c12835e5d18b
type: summary_end

-->
<!--
code_unit: flytekit.types.error.error.FlyteError
code_unit_type: class
help_text: ''
key: example_2d3a9ff8-1d23-41b1-b12b-3c60d9ba1613
type: example

-->
<!--
code_unit: flytekit.types.pickle.pickle.FlytePickle
code_unit_type: class
help_text: ''
key: example_0687d76a-c66f-455b-9c84-b3697cdc7838
type: example

-->
<!--
code_unit: flytekit.types.numpy.ndarray.NumpyArrayTransformer
code_unit_type: class
help_text: ''
key: example_15e8f860-6373-4966-a1d2-2a39b547ed81
type: example

-->