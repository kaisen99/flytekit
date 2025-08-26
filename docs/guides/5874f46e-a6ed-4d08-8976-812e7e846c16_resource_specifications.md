
<!--
help_text: ''
key: summary_resource_specifications_0cc27729-c8bb-4475-a16b-3b7f600e4641
modules:
- flytekit.core.resources
questions_to_answer: []
type: summary

-->
Resource Specifications define the compute resources required for tasks, ensuring efficient and stable execution within an orchestration environment. These specifications allow developers to precisely control CPU, memory, GPU, and ephemeral storage allocations.

### Defining Resources with `Resources`

The `Resources` class is the primary mechanism for specifying resource requirements. It supports defining CPU, memory, GPU, and ephemeral storage.

**Supported Resource Types:**

*   `cpu`: Specifies CPU units. Can be an integer (e.g., `1` for 1 CPU core), a float (e.g., `0.5` for half a core), or a string (e.g., `"100m"` for 100 milliCPU, which is 1/10th of a CPU).
*   `mem`: Specifies memory in bytes. Can be an integer (e.g., `2048` for 2 KB) or a string with units (e.g., `"2Gi"` for 2 gigabytes).
*   `gpu`: Specifies the number of GPU units. Can be an integer (e.g., `1`) or a string.
*   `ephemeral_storage`: Specifies temporary, local storage for scratch space, caching, and logs. Can be an integer (e.g., `1024` for 1 KB) or a string with units (e.g., `"1Gi"` for 1 gigabyte).

**Specifying Requests and Limits:**

When defining resources, you can specify both a request and a limit. The request is the minimum amount of a resource guaranteed to be available, while the limit is the maximum amount a task can consume.

*   **Single Value:** If a single value is provided for a resource, both the request and the limit are set to that value.

    ```python
    from flytekit.core.resources import Resources

    # Request and limit for CPU are both 1 core, memory is 2048 bytes
    res_single = Resources(cpu="1", mem=2048)
    # Request and limit for CPU are both 0.5 cores, memory is 2 gigabytes
    res_single_with_units = Resources(cpu=0.5, mem="2Gi")
    ```

*   **Tuple or List:** To specify distinct request and limit values, provide a tuple or list where the first element is the request and the second is the limit.

    ```python
    from flytekit.core.resources import Resources

    # CPU request is 1 core, limit is 2 cores
    # Memory request and limit are both 1024 bytes
    res_tuple = Resources(cpu=("1", "2"), mem=1024)

    # GPU request is 1 unit, limit is 1 unit
    # Ephemeral storage request is 1Gi, limit is 2Gi
    res_mixed = Resources(gpu=1, ephemeral_storage=("1Gi", "2Gi"))
    ```

**Important Considerations:**

*   Resource units for CPU and memory generally follow Kubernetes conventions. For example, `m` denotes milliCPU, and `Gi` denotes gibibytes.
*   Ephemeral storage is for temporary data. Persistent storage is not currently supported on the Flyte backend.

### Explicitly Separating Requests and Limits with `ResourceSpec`

While the `Resources` class allows for combined request/limit definitions, the `ResourceSpec` class provides a more explicit structure by separating `requests` and `limits` into distinct `Resources` objects. This can enhance clarity and enforce a clear distinction between the two.

The `ResourceSpec` class has two attributes:
*   `requests`: An instance of `Resources` representing the requested resources.
*   `limits`: An instance of `Resources` representing the resource limits.

**Constructing `ResourceSpec`:**

You can directly construct a `ResourceSpec` by providing separate `Resources` objects for requests and limits.

```python
from flytekit.core.resources import Resources, ResourceSpec

# Define explicit request and limit objects
my_requests = Resources(cpu="500m", mem="1Gi")
my_limits = Resources(cpu="1", mem="2Gi")

# Create a ResourceSpec
task_spec = ResourceSpec(requests=my_requests, limits=my_limits)
```

**Converting Combined Resources with `ResourceSpec.from_multiple_resource`:**

The `ResourceSpec.from_multiple_resource` class method is a utility for converting a single `Resources` object (which might contain request/limit tuples) into a `ResourceSpec` with separate `requests` and `limits` objects. This is particularly useful when you initially define resources using the tuple/list syntax within a single `Resources` object and later need to access the requests and limits distinctly.

The method iterates through the fields of the input `Resources` object. If a field's value is a tuple or list, it splits the values into the `requests` and `limits` attributes of the new `ResourceSpec`. If a field's value is a single item, that value is applied to both the `requests` and `limits` attributes.

```python
from flytekit.core.resources import Resources, ResourceSpec

# Define resources using the combined tuple syntax
combined_resources = Resources(cpu=("1", "2"), mem="1Gi", gpu=1)

# Convert to a ResourceSpec with explicit requests and limits
explicit_spec = ResourceSpec.from_multiple_resource(combined_resources)

print(f"CPU Request: {explicit_spec.requests.cpu}")  # Output: CPU Request: 1
print(f"CPU Limit: {explicit_spec.limits.cpu}")    # Output: CPU Limit: 2
print(f"Memory Request: {explicit_spec.requests.mem}") # Output: Memory Request: 1Gi
print(f"Memory Limit: {explicit_spec.limits.mem}")   # Output: Memory Limit: 1Gi
print(f"GPU Request: {explicit_spec.requests.gpu}")  # Output: GPU Request: 1
print(f"GPU Limit: {explicit_spec.limits.gpu}")    # Output: GPU Limit: 1
```

### Integration and Best Practices

Resource specifications are typically integrated directly into task definitions, often through decorators like `@task(resources=...)`.

**Common Use Cases:**

*   **Basic Task Allocation:** Assigning a fixed amount of CPU and memory to a task.
    ```python
    # Example of how it might be used with a task decorator (conceptual)
    # @task(resources=Resources(cpu="500m", mem="512Mi"))
    # def my_task():
    #     pass
    ```
*   **Resource Bursting:** Allowing a task to use more resources than its guaranteed request, up to a defined limit, during peak demand.
    ```python
    # Example of how it might be used with a task decorator (conceptual)
    # @task(resources=Resources(cpu=("1", "2"), mem=("2Gi", "4Gi")))
    # def data_processing_task():
    #     pass
    ```
*   **GPU-Accelerated Tasks:** Specifying GPU requirements for machine learning workloads.
    ```python
    # Example of how it might be used with a task decorator (conceptual)
    # @task(resources=Resources(gpu=1, mem="8Gi"))
    # def ml_training_task():
    #     pass
    ```

**Best Practices:**

*   **Start with Requests:** Always define resource requests to ensure your tasks have the minimum necessary resources. This helps prevent resource starvation and improves scheduling predictability.
*   **Set Limits Judiciously:** While limits prevent tasks from consuming excessive resources and impacting other workloads, setting them too low can lead to tasks being throttled or OOMKilled (Out Of Memory Killed). Monitor task resource usage to fine-tune limits.
*   **Use Appropriate Units:** Be consistent and precise with resource units (e.g., `Mi`, `Gi` for memory; `m` for CPU).
*   **Consider Ephemeral Storage:** For tasks that generate large temporary files or logs, allocate sufficient `ephemeral_storage` to avoid disk pressure issues.

**Performance Considerations:**

*   **Under-provisioning:** Setting requests too low can lead to tasks running slowly due to resource contention or failing if they exceed their limits.
*   **Over-provisioning:** Setting requests or limits too high can lead to inefficient resource utilization, as allocated resources remain idle, potentially delaying other tasks or increasing infrastructure costs.
*   **Resource Quotas:** Be aware of any cluster-wide resource quotas that might override or constrain the resource specifications defined for individual tasks.
<!--
key: summary_resource_specifications_0cc27729-c8bb-4475-a16b-3b7f600e4641
type: summary_end

-->
<!--
code_unit: flytekit.core.resources.Resources
code_unit_type: class
help_text: ''
key: example_b3e74260-6518-4609-b540-1ec4eb1a6026
type: example

-->
<!--
code_unit: flytekit.core.resources.ResourceSpec
code_unit_type: class
help_text: ''
key: example_c8b15c4d-79ad-4dfd-a49a-e62dba1419e6
type: example

-->