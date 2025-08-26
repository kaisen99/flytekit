# TaskMetadata

This class encapsulates metadata for a Flyte task, including caching, retries, and timeout configurations. It enables control over task behavior, such as enabling caching, specifying cache versions, and defining retry strategies. The class provides methods to convert the metadata into a task model suitable for execution.

## Attributes

- **cache**: bool = False
  - Indicates if caching should be enabled. See :std:ref:`Caching &lt; cookbook:caching &gt;`.

- **cache_serialize**: bool = False
  - Indicates if identical (i.e. same inputs) instances of this task should be executed in serial when caching is enabled. See :std:ref:`Caching &lt; cookbook:caching &gt;`.

- **cache_version**: str = &quot;&quot;
  - Version to be used for the cached value.

- **cache_ignore_input_vars**: Tuple[str, ...] = ()
  - Input variables that should not be included when calculating hash for cache.

- **interruptible**: Optional[bool] = None
  - Indicates that this task can be interrupted and/or scheduled on nodes with lower QoS guarantees that can include pre-emption.

- **deprecated**: str = &quot;&quot;
  - Can be used to provide a warning message for a deprecated task. An absence or empty string indicates that the task is active and not deprecated.

- **retries**: int = 0
  - for retries=n; n &gt; 0, on failures of this task, the task will be retried at-least n number of times.

- **timeout**: Optional[[Union](flytekit_models_literals_union)[datetime.timedelta, int]] = None
  - The maximum duration for which one execution of this task should run. The execution will be terminated if the runtime exceeds this timeout.

- **pod_template_name**: Optional[str] = None
  - The name of an existing PodTemplate resource in the cluster which will be used for this task.

- **generates_deck**: bool = False
  - Indicates whether the task will generate a Deck URI.

- **is_eager**: bool = False
  - Indicates whether the task should be treated as eager.

## Constructors
def TaskMetadata(cache: bool = False, cache_serialize: bool = False, cache_version: str = &quot;&quot;, cache_ignore_input_vars: Tuple[str, ...] = (), interruptible: Optional[bool] = None, deprecated: str = &quot;&quot;, retries: int = 0, timeout: Optional[[Union](flytekit_models_literals_union)[datetime.timedelta, int]] = None, pod_template_name: Optional[str] = None, generates_deck: bool = False, is_eager: bool = False)
-  Metadata for a Task. Things like retries and whether or not caching is turned on, and cache version are specified here.
- **Parameters**

  - **cache**: bool
    - Indicates if caching should be enabled. See :std:ref:`Caching &lt; cookbook:caching &gt;`.
  - **cache_serialize**: bool
    - Indicates if identical (i.e. same inputs) instances of this task should be executed in serial when caching is enabled. See :std:ref:`Caching &lt; cookbook:caching &gt;`.
  - **cache_version**: str
    - Version to be used for the cached value.
  - **cache_ignore_input_vars**: Tuple[str, ...]
    - Input variables that should not be included when calculating hash for cache.
  - **interruptible**: Optional[bool]
    - Indicates that this task can be interrupted and/or scheduled on nodes with lower QoS guarantees that can include pre-emption.
  - **deprecated**: str
    - Can be used to provide a warning message for a deprecated task. An absence or empty string indicates that the task is active and not deprecated.
  - **retries**: int
    - for retries=n; n &gt; 0, on failures of this task, the task will be retried at-least n number of times.
  - **timeout**: Optional[[Union](flytekit_models_literals_union)[datetime.timedelta, int]]
    - The maximum duration for which one execution of this task should run. The execution will be terminated if the runtime exceeds this timeout.
  - **pod_template_name**: Optional[str]
    - The name of an existing PodTemplate resource in the cluster which will be used for this task.
  - **generates_deck**: bool
    - Indicates whether the task will generate a Deck URI.
  - **is_eager**: bool
    - Indicates whether the task should be treated as eager.



## Methods
```@classmethod
def retry_strategy()
```
-  Returns a RetryStrategy object based on the number of retries configured for the task.

- **Return Value**:
**_literal_models.RetryStrategy**
  - A RetryStrategy object.
```@classmethod
def to_taskmetadata_model()
```
-  Converts the TaskMetadata object into a _task_model.TaskMetadata object, which is likely used for internal representation or serialization.

- **Return Value**:
**_task_model.TaskMetadata**
  - A _task_model.TaskMetadata object representing the task&#x27;s metadata.
