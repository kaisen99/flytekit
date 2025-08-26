
<!--
help_text: ''
key: summary_local_task_caching_a2943a02-95ea-4bcc-a91d-27cf536860a0
modules:
- flytekit.core.local_cache
questions_to_answer: []
type: summary

-->
Local Task Caching

Local Task Caching provides a persistent store for the results of local task executions. This mechanism significantly accelerates iterative development and testing by preventing redundant computation of tasks with identical inputs. When a task executes locally, its output can be stored, and subsequent executions with the same inputs retrieve the result directly from the cache, bypassing re-execution.

## Core Functionality

The central component for managing local task caching is the `LocalTaskCache` class. This class implements the logic for storing, retrieving, and managing cached task outputs. It operates as a static utility, meaning its methods are accessed directly on the class.

### Cache Initialization and Management

The `LocalTaskCache` requires initialization before use. This typically occurs automatically upon the first attempt to access or store data.

*   **Initialization**: The `LocalTaskCache.initialize()` method sets up the underlying persistent storage. While usually handled implicitly, explicit initialization ensures the cache is ready.
*   **Clearing the Cache**: To invalidate all stored results and start with a clean slate, use `LocalTaskCache.clear()`. This is useful when task logic or external dependencies change significantly, requiring all tasks to re-execute.

### Caching Mechanism

The effectiveness of local task caching relies on a robust key generation mechanism and efficient data handling.

#### Cache Key Generation

A unique cache key identifies each cached task result. This key is derived from a combination of factors to ensure that results are retrieved only when the task's inputs and version are consistent. The key incorporates:

*   **Task Name**: The unique identifier of the task.
*   **Cache Version**: A string representing the version of the task's logic. Incrementing this version explicitly invalidates previous cached results for that task, forcing re-execution. This is crucial for managing cache consistency when task code changes.
*   **Input Literal Map**: A representation of all input values provided to the task. The cache key is sensitive to changes in these inputs.
*   **Ignored Input Variables**: A tuple of input variable names that should be excluded from the cache key calculation. This is particularly useful for inputs that do not affect the task's output (e.g., timestamps, unique run IDs, or non-deterministic parameters) but might otherwise cause unnecessary cache misses.

#### Data Storage

Task results are stored as serialized `ModelLiteralMap` objects. When a result is stored using `LocalTaskCache.set()`, the `ModelLiteralMap` is converted into a Flyte IDL protocol buffer string (bytes) for efficient storage. Upon retrieval via `LocalTaskCache.get()`, the stored bytes are deserialized back into a `ModelLiteralMap` object, ensuring the correct data format.

### Retrieving Cached Results

To retrieve a cached result for a task, use the `LocalTaskCache.get()` method.

```python
from flytekit.core.literal import ModelLiteralMap
from flytekit.core.local_cache import LocalTaskCache
from typing import Optional, Tuple

def get_cached_result(
    task_name: str,
    cache_version: str,
    input_literal_map: ModelLiteralMap,
    cache_ignore_input_vars: Tuple[str, ...]
) -> Optional[ModelLiteralMap]:
    """
    Attempts to retrieve a task result from the local cache.
    """
    return LocalTaskCache.get(task_name, cache_version, input_literal_map, cache_ignore_input_vars)
```

The method returns a `ModelLiteralMap` if a matching result is found in the cache; otherwise, it returns `None`. The deserialization process handles both direct `ModelLiteralMap` objects and serialized byte strings, converting them to the expected `ModelLiteralMap` format.

### Storing Task Results

After a local task execution completes successfully, its output can be stored in the cache using the `LocalTaskCache.set()` method.

```python
from flytekit.core.literal import ModelLiteralMap
from flytekit.core.local_cache import LocalTaskCache
from typing import Tuple

def store_task_result(
    task_name: str,
    cache_version: str,
    input_literal_map: ModelLiteralMap,
    cache_ignore_input_vars: Tuple[str, ...],
    result_value: ModelLiteralMap
) -> None:
    """
    Stores a task result in the local cache.
    """
    LocalTaskCache.set(task_name, cache_version, input_literal_map, cache_ignore_input_vars, result_value)
```

The `result_value` (a `ModelLiteralMap`) is serialized to a protocol buffer string before being written to the cache.

## Usage Patterns and Best Practices

Local Task Caching is designed to optimize the developer experience during local iteration.

*   **Accelerating Local Development**: Leverage local task caching for computationally intensive tasks or those with long execution times. This significantly reduces the feedback loop during development.
*   **Effective Cache Versioning**: Always use a meaningful `cache_version` for your tasks. Increment this version whenever the task's logic changes in a way that would alter its output for the same inputs. This ensures that stale results are not served from the cache.
*   **Strategic Input Ignoring**: Utilize `cache_ignore_input_vars` for inputs that do not influence the task's deterministic output. For example, if a task accepts a `timestamp` input that is only used for logging and does not affect the core computation, adding `timestamp` to `cache_ignore_input_vars` will allow cache hits even if the timestamp varies between runs.
*   **Understanding Cache Scope**: The cache is local to the development environment. It does not affect remote executions of tasks.
*   **Data Consistency Considerations**: The cache operates purely on task inputs and outputs. If a task depends on external state (e.g., a database, an external API) that changes independently of its inputs, the cached result might become stale. In such cases, consider explicitly clearing the cache or incrementing the `cache_version` to force re-execution.
*   **Performance Overhead**: While caching significantly reduces overall execution time for repeated runs, there is a minor overhead associated with serialization, deserialization, and disk I/O for cache operations. This overhead is generally negligible compared to the cost of re-executing a complex task.
<!--
key: summary_local_task_caching_a2943a02-95ea-4bcc-a91d-27cf536860a0
type: summary_end

-->
<!--
code_unit: flytekit.core.local_cache.LocalTaskCache
code_unit_type: class
help_text: ''
key: example_f7394e75-b35c-4142-beee-bc77c5673f1e
type: example

-->