
<!--
help_text: ''
key: summary_advanced_workflow_and_task_configuration_cd045e51-e1ec-47f4-9825-049305371397
modules:
- flytekit.core.resources
- flytekit.core.options
- flytekit.core.pod_template
questions_to_answer: []
type: summary

-->
Advanced Workflow and Task Configuration provides granular control over how Flyte tasks and workflows execute, manage resources, and interact with the underlying infrastructure. This capability allows developers to fine-tune performance, security, and operational behavior to meet specific application requirements.

### Resource Management

Resource management enables precise allocation of computational resources (CPU, memory, GPU, and ephemeral storage) for individual tasks. This ensures tasks have sufficient resources to run efficiently while preventing resource contention.

The `Resources` class specifies both resource requests and limits. When defining a task, you can use this class to declare the minimum resources a task needs (requests) and the maximum resources it can consume (limits).

**Key Attributes of `Resources`:**

*   `cpu`: Specifies CPU resources. Can be an integer (e.g., `1` for 1 CPU core), a float (e.g., `0.5` for 500m CPU), or a string (e.g., `"100m"` for 1/10th of a CPU).
*   `mem`: Specifies memory resources. Can be an integer (e.g., `1024` for 1KB) or a string (e.g., `"2Gi"` for 2 gigabytes).
*   `gpu`: Specifies GPU resources. Can be an integer or a string.
*   `ephemeral_storage`: Specifies local ephemeral storage for scratch space, caching, and logs. Can be an integer or a string (e.g., `"1Gi"`).

**Specifying Requests and Limits:**

*   **Single Value:** If a single value is provided for an attribute (e.g., `cpu="1"`), both the request and limit for that resource are set to that value.
*   **Tuple or List:** If a tuple or list of two values is provided (e.g., `cpu=("1", "2")`), the first value sets the request, and the second value sets the limit.

**Example Usage:**

```python
from flytekit import task
from flytekit.core.resources import Resources

@task(resources=Resources(cpu="1", mem="2Gi"))
def my_cpu_intensive_task(a: int, b: int) -> int:
    # Task logic that requires 1 CPU and 2GiB memory
    return a + b

@task(resources=Resources(cpu=("500m", "1"), mem=("1Gi", "2Gi"), gpu=1))
def my_gpu_task(data: str) -> str:
    # Task logic that requests 500m CPU, 1GiB memory, 1 GPU
    # and can burst up to 1 CPU, 2GiB memory
    return f"Processed {data} with GPU"

@task(resources=Resources(ephemeral_storage="10Gi"))
def my_data_processing_task(input_path: str) -> str:
    # Task that requires 10GiB of ephemeral storage for temporary files
    return f"Processed data from {input_path}"
```

**Important Considerations:**

*   The `Resources` class performs validation to ensure values are of the correct type (string, int, float) and that tuples/lists have exactly two elements.
*   Persistent storage is not directly supported through this mechanism on the Flyte backend.
*   The `ResourceSpec` class is an internal representation that converts the `Resources` object into distinct `requests` and `limits` objects. While `ResourceSpec.from_multiple_resource` can perform this conversion, developers typically interact directly with the `Resources` class when defining tasks.

### Execution Options

Execution options provide a mechanism to configure various aspects of a workflow execution or launch plan, including metadata, security, data handling, and parallelism. These options are applied at the workflow or launch plan level, affecting all tasks within that execution unless overridden at the task level.

The `Options` class encapsulates these configurable settings.

**Key Attributes of `Options`:**

*   `labels`: Custom key-value pairs applied as labels to the execution resource (e.g., for organizational or filtering purposes).
*   `annotations`: Custom key-value pairs applied as annotations to the execution resource (e.g., for non-identifying metadata).
*   `security_context`: Defines the security context for permissions, typically specifying a Kubernetes service account for the execution.
*   `raw_output_data_config`: Specifies an optional remote prefix (e.g., `s3://<bucket>/key...`) for storing offloaded data. If not specified, the platform's default is used.
*   `max_parallelism`: Controls the maximum number of task nodes that can run concurrently within the entire workflow execution.
*   `notifications`: A list of notification configurations (e.g., email, Slack) for the execution.
*   `disable_notifications`: A boolean flag to disable all notifications for the execution.
*   `overwrite_cache`: A boolean flag to force re-execution even if cached results are available.

**Convenience Method:**

*   `Options.default_from(k8s_service_account: str, raw_data_prefix: str)`: A class method to easily create an `Options` object with a specified Kubernetes service account and raw data output prefix.

**Example Usage:**

```python
from flytekit import workflow, task
from flytekit.core.options import Options
from flytekit.core.security import Identity, SecurityContext
from flytekit.models.common import RawOutputDataConfig, Labels, Annotations

@task
def my_simple_task(x: int) -> int:
    return x * 2

@workflow
def my_workflow(input_val: int) -> int:
    return my_simple_task(x=input_val)

# Example 1: Launching a workflow with custom labels and annotations
my_workflow_with_metadata = my_workflow.with_options(
    options=Options(
        labels=Labels(values={"team": "data-science", "project": "feature-eng"}),
        annotations=Annotations(values={"reviewer": "john.doe", "priority": "high"})
    )
)

# Example 2: Launching a workflow with a specific service account and custom output location
my_workflow_secure_output = my_workflow.with_options(
    options=Options.default_from(
        k8s_service_account="my-service-account",
        raw_data_prefix="s3://my-custom-bucket/flyte-outputs"
    )
)

# Example 3: Limiting parallelism and disabling notifications
my_workflow_controlled = my_workflow.with_options(
    options=Options(
        max_parallelism=5,
        disable_notifications=True
    )
)
```

### Custom Pod Configuration

Custom Pod configuration provides the most granular control over the underlying Kubernetes Pod that executes a task. This is useful for advanced scenarios requiring specific Kubernetes features not directly exposed through higher-level Flytekit constructs, such as adding sidecar containers, configuring node selectors, or setting specific container security contexts.

The `PodTemplate` class allows you to define a custom Kubernetes Pod specification.

**Key Attributes of `PodTemplate`:**

*   `pod_spec`: An instance of `kubernetes.client.V1PodSpec`. This is the core Kubernetes object where you define the entire Pod configuration, including containers, volumes, init containers, node selectors, tolerations, etc.
*   `primary_container_name`: Specifies the name of the primary container within the `pod_spec` that Flytekit manages for task execution. By default, this is `primary`.
*   `labels`: Custom key-value pairs applied as labels to the Pod.
*   `annotations`: Custom key-value pairs applied as annotations to the Pod.

**Important Considerations:**

*   Using `PodTemplate` requires familiarity with Kubernetes Pod specifications and the `kubernetes.client` library. You must install the `kubernetes` Python client (`pip install kubernetes`) to construct `V1PodSpec` objects.
*   The `primary_container_name` must be defined and correspond to a container within the `pod_spec`. Flytekit injects the task's execution logic into this primary container.
*   This configuration is applied at the task level.

**Example Usage:**

```python
from flytekit import task
from flytekit.core.pod_template import PodTemplate
from kubernetes.client import V1PodSpec, V1Container, V1EnvVar, V1Volume, V1VolumeMount

@task(pod_template=PodTemplate(
    pod_spec=V1PodSpec(
        containers=[
            V1Container(
                name="primary",
                image="ghcr.io/flyteorg/flytekit:py3.9-latest", # Base image for the primary container
                env=[V1EnvVar(name="MY_ENV_VAR", value="my_value")],
                volume_mounts=[V1VolumeMount(name="data-volume", mount_path="/data")]
            ),
            V1Container( # Example of a sidecar container
                name="sidecar-logger",
                image="busybox",
                command=["sh", "-c", "while true; do echo 'Sidecar logging...'; sleep 5; done"],
                volume_mounts=[V1VolumeMount(name="data-volume", mount_path="/data")]
            )
        ],
        volumes=[V1Volume(name="data-volume")],
        node_selector={"kubernetes.io/hostname": "my-specific-node"},
        tolerations=[{"key": "dedicated", "operator": "Exists", "effect": "NoSchedule"}]
    ),
    primary_container_name="primary",
    labels={"app": "my-custom-app"},
    annotations={"owner": "dev-team"}
))
def my_custom_pod_task(message: str) -> str:
    # Task logic that runs within the primary container of the custom pod
    # It can interact with the shared volume or rely on node selectors.
    with open("/data/output.txt", "w") as f:
        f.write(message)
    return f"Task completed with custom pod config. Message written to /data/output.txt"
```
<!--
key: summary_advanced_workflow_and_task_configuration_cd045e51-e1ec-47f4-9825-049305371397
type: summary_end

-->
<!--
code_unit: flytekit.core.resources.Resources
code_unit_type: class
help_text: ''
key: example_957becbc-72aa-453d-8e64-927b5450de4d
type: example

-->
<!--
code_unit: flytekit.core.options.Options
code_unit_type: class
help_text: ''
key: example_3db11843-96f8-4173-8c33-632405615b48
type: example

-->
<!--
code_unit: flytekit.core.pod_template.PodTemplate
code_unit_type: class
help_text: ''
key: example_a4a87eeb-3374-48d3-99c4-4cfaa4be3f57
type: example

-->