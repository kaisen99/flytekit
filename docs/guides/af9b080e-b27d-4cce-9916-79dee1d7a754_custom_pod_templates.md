
<!--
help_text: ''
key: summary_custom_pod_templates_b79afcb2-0c93-438a-8846-aa0e9f3b2a0b
modules:
- flytekit.core.pod_template
questions_to_answer: []
type: summary

-->
## Custom Pod Templates

Custom Pod Templates provide a powerful mechanism to extend the default Kubernetes Pod specification used by Flyte for task execution. This capability allows developers to define fine-grained control over the underlying infrastructure, enabling advanced use cases such as integrating sidecar containers, mounting custom volumes, or specifying node affinities.

The core component for defining a custom Pod Template is the `PodTemplate` class.

### The `PodTemplate` Class

The `PodTemplate` class, located in `flytekit.core.pod_template`, serves as the blueprint for customizing the Kubernetes Pod that executes a Flyte task. It encapsulates the necessary attributes to modify the Pod's behavior and environment.

**Attributes:**

*   **`pod_spec: Optional["V1PodSpec"]`**: This is the primary attribute for defining the Kubernetes Pod specification. It directly accepts an instance of `kubernetes.client.V1PodSpec`. Developers must import `V1PodSpec` from the `kubernetes.client` library to construct complex Pod configurations. If `pod_spec` is not provided during initialization, it defaults to an empty `V1PodSpec` instance.
*   **`primary_container_name: str`**: This attribute specifies the name of the main container within the `pod_spec` that Flyte uses to execute the task's code. Flyte injects the task's executable into this container. It defaults to `PRIMARY_CONTAINER_DEFAULT_NAME` and must not be an empty string.
*   **`labels: Optional[Dict[str, str]]`**: A dictionary of key-value pairs representing Kubernetes labels to apply to the Pod. These labels can be used for organization, selection, or policy enforcement.
*   **`annotations: Optional[Dict[str, str]]`**: A dictionary of key-value pairs representing Kubernetes annotations to apply to the Pod. Annotations are typically used for non-identifying metadata.

### Integrating Custom Pod Templates with Flyte Tasks

To apply a custom Pod Template to a Flyte task, instantiate the `PodTemplate` class and pass it to the `pod_template` argument of the `@task` decorator.

**Example: Adding a Sidecar Container**

This example demonstrates how to add a simple `nginx` sidecar container to a task's Pod. The sidecar runs alongside the primary task container, sharing the same network namespace and volumes.

```python
from flytekit import task, workflow
from flytekit.core.pod_template import PodTemplate
from kubernetes.client import V1PodSpec, V1Container

# Define the PodTemplate with an Nginx sidecar
nginx_sidecar_pod_template = PodTemplate(
    pod_spec=V1PodSpec(
        containers=[
            V1Container(
                name="nginx-sidecar",
                image="nginx:latest",
                ports=[{"containerPort": 80}],
            )
        ]
    ),
    primary_container_name="flytekit-container", # Ensure this matches Flyte's default or your custom primary container name
)

@task(pod_template=nginx_sidecar_pod_template)
def my_task_with_sidecar(name: str) -> str:
    """
    A task that runs with an Nginx sidecar.
    """
    print(f"Hello, {name}! This task is running with an Nginx sidecar.")
    # You could potentially interact with the sidecar here, e.g., via localhost
    return f"Task completed with sidecar for {name}"

@workflow
def sidecar_workflow(name: str) -> str:
    return my_task_with_sidecar(name=name)

```

**Example: Mounting a Host Path Volume and Setting Resources**

This example illustrates how to mount a host path volume into the primary container and specify custom CPU and memory requests and limits.

```python
from flytekit import task, workflow
from flytekit.core.pod_template import PodTemplate
from kubernetes.client import V1PodSpec, V1Container, V1ResourceRequirements, V1Volume, V1VolumeMount

# Define the PodTemplate with a host path volume and resource limits
custom_resource_pod_template = PodTemplate(
    pod_spec=V1PodSpec(
        volumes=[
            V1Volume(
                name="host-data",
                host_path={"path": "/mnt/data", "type": "DirectoryOrCreate"}
            )
        ],
        containers=[
            V1Container(
                name="flytekit-container", # Must match the primary container name
                resources=V1ResourceRequirements(
                    requests={"cpu": "500m", "memory": "1Gi"},
                    limits={"cpu": "1", "memory": "2Gi"},
                ),
                volume_mounts=[
                    V1VolumeMount(
                        name="host-data",
                        mount_path="/app/data"
                    )
                ]
            )
        ]
    ),
    primary_container_name="flytekit-container",
    labels={"environment": "dev"},
    annotations={"owner": "data-team"},
)

@task(pod_template=custom_resource_pod_template)
def process_data_from_host(input_path: str) -> str:
    """
    A task that processes data from a host path volume.
    """
    # In a real scenario, you would read/write from /app/data
    print(f"Processing data from {input_path} mounted at /app/data")
    return f"Data processed from {input_path}"

@workflow
def data_processing_workflow(path: str) -> str:
    return process_data_from_host(input_path=path)

```

### Best Practices and Considerations

*   **Kubernetes Client Dependency:** Using `PodTemplate` with `pod_spec` requires the `kubernetes` Python client library to be installed in your environment.
*   **Primary Container Name:** Always ensure the `primary_container_name` attribute in your `PodTemplate` matches the name Flyte assigns to its main task container. By default, Flyte uses `flytekit-container`. If you define a `V1Container` within `pod_spec.containers` with this name, Flyte will merge its configuration (like image and command) into that specific container. If no container with this name exists, Flyte will add its primary container to the `pod_spec`.
*   **Merging Behavior:** Flyte intelligently merges the provided `pod_spec` with its internally generated Pod specification.
    *   Containers defined in your `pod_spec` (other than the primary container) are added as sidecars or init containers.
    *   Volumes, volume mounts, and other Pod-level configurations are merged.
    *   For the primary container, Flyte typically overrides the `image` and `command` to ensure task execution. Other attributes like `resources`, `env`, `volume_mounts` are merged.
*   **Validation:** The `PodTemplate` class validates that `primary_container_name` is not empty, raising a `FlyteValidationException` if it is.
*   **Security Implications:** When using host path volumes or other privileged Pod configurations, be mindful of the security implications. Ensure your Kubernetes cluster policies are configured appropriately.
*   **Debugging:** If tasks fail when using custom Pod Templates, inspect the actual Pod definition on the Kubernetes cluster using `kubectl describe pod <pod-name>` and `kubectl logs <pod-name> -c <container-name>` to understand the merged configuration and container logs.
*   **Limitations:** While `PodTemplate` offers extensive customization, some aspects of the Pod lifecycle or specific fields might be managed or overridden by Flyte's orchestration layer to ensure proper task execution. For instance, the primary container's entrypoint and arguments are typically controlled by Flyte.
*   **Performance:** Carefully consider resource requests and limits. Over-requesting resources can lead to inefficient cluster utilization, while under-requesting can cause tasks to be throttled or evicted.
<!--
key: summary_custom_pod_templates_b79afcb2-0c93-438a-8846-aa0e9f3b2a0b
type: summary_end

-->
<!--
code_unit: flytekit.core.pod_template.PodTemplate
code_unit_type: class
help_text: ''
key: example_bc96e35d-2277-4eec-b22d-abfbd65e3934
type: example

-->