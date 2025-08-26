
<!--
help_text: ''
key: summary_pod_task_plugin_4514718f-a694-48f7-95bb-7f3ca8f4efbb
modules:
- flytekitplugins.pod.task
questions_to_answer: []
type: summary

-->
The Pod Task Plugin extends Flyte's capabilities by allowing users to define and customize the underlying Kubernetes Pod specification for their tasks. This plugin is essential for advanced use cases requiring fine-grained control over the task's execution environment, such as integrating with specialized hardware, running sidecar containers, or applying specific Kubernetes configurations like node selectors, tolerations, or custom volumes.

By default, Flyte launches every task as a container within a Kubernetes Pod. The Pod Task Plugin exposes the full `V1PodSpec` from the Kubernetes API, enabling comprehensive customization beyond standard Flyte resource requests or environment variables.

## Core Concepts

The Pod Task Plugin introduces two primary components:

*   **The `Pod` Configuration**: This class encapsulates the Kubernetes `V1PodSpec` and additional metadata for the Pod.
    *   `pod_spec`: A `kubernetes.client.V1PodSpec` object that defines the desired state of the Pod. This includes containers, volumes, init containers, node selectors, tolerations, and more.
    *   `primary_container_name`: A critical string that identifies the container within the `pod_spec` where the Flyte task's Python function executes. Flyte manages this container to ensure the task runs correctly.
    *   `labels` and `annotations`: Optional dictionaries for attaching metadata to the Pod, useful for Kubernetes-native tooling or organizational purposes.

*   **The `PodFunctionTask`**: This is a specialized Flyte task type that consumes the `Pod` configuration. It extends the standard `PythonFunctionTask`, allowing a Python function to be executed within the context of a custom Kubernetes Pod.

## Defining a Pod Task

To define a task using the Pod Task Plugin, create an instance of the `Pod` configuration and pass it to the `task` decorator's `task_config` argument.

### Basic Pod Configuration

The simplest Pod task defines a `V1PodSpec` and specifies the primary container name.

```python
from flytekit import task
from flytekitplugins.pod.task import Pod
from kubernetes.client.models import V1PodSpec, V1Container

@task(
    task_config=Pod(
        pod_spec=V1PodSpec(
            containers=[
                V1Container(name="my-primary-container", image="cr.flyte.org/flyteorg/flytekit:py3.9-latest"),
            ]
        ),
        primary_container_name="my-primary-container",
    )
)
def my_pod_task(a: int, b: int) -> int:
    """
    This task runs within a custom Pod defined by the V1PodSpec.
    """
    return a + b
```

### Managing the Primary Container

The `primary_container_name` is crucial for how Flyte manages the execution of your Python function.

*   **Flyte's Control**: When the `PodFunctionTask` is serialized, Flyte ensures that the container identified by `primary_container_name` is correctly configured to run your Python task. This involves:
    *   **Image**: Overwriting the container's image with the one specified during task serialization (e.g., the default image for your Flyte project).
    *   **Command and Arguments**: Clearing any existing `command` and `args` and injecting the necessary commands to execute your Python function.
    *   **Resources**: Applying the resource limits and requests defined for the Flyte task (e.g., using `@task(requests=...)`) to this primary container.
    *   **Environment Variables**: Injecting Flyte-specific environment variables and appending any user-defined environment variables from the `V1Container` definition.

*   **User-Defined Primary Container**: You can include a `V1Container` with the `primary_container_name` in your `pod_spec`. However, Flyte will overwrite its `image`, `command`, `args`, `resources`, and `env` (except for appending user-defined env vars) to ensure the Python function runs. This means you cannot fully control these specific attributes for the primary container via the `pod_spec`.

    ```python
    from flytekit import task
    from flytekitplugins.pod.task import Pod
    from kubernetes.client.models import V1PodSpec, V1Container, V1ResourceRequirements

    @task(
        task_config=Pod(
            pod_spec=V1PodSpec(
                containers=[
                    V1Container(
                        name="my-primary-container",
                        image="my-custom-image:latest", # This will be overwritten by Flyte's default image
                        command=["echo", "hello"],       # This will be overwritten by Flyte's command
                        resources=V1ResourceRequirements(requests={"cpu": "100m"}), # This will be overwritten by Flyte's task resources
                    ),
                ]
            ),
            primary_container_name="my-primary-container",
        )
    )
    def primary_container_override_example():
        print("This will run using Flyte's injected command and image.")
    ```

*   **Flyte-Injected Primary Container**: If your `pod_spec` does not contain a container with the `primary_container_name`, Flyte automatically injects a placeholder container with that name and configures it to run your Python function. This is useful when you primarily want to define sidecar containers or other Pod-level configurations without explicitly defining the primary container.

    ```python
    from flytekit import task
    from flytekitplugins.pod.task import Pod
    from kubernetes.client.models import V1PodSpec, V1Container

    @task(
        task_config=Pod(
            pod_spec=V1PodSpec(
                containers=[
                    V1Container(name="my-sidecar", image="busybox", command=["sleep", "3600"]),
                ]
            ),
            primary_container_name="flyte-container", # Flyte will inject this container
        )
    )
    def task_with_sidecar():
        print("This task runs alongside a busybox sidecar.")
    ```

### Adding Sidecar Containers

A common use case for Pod tasks is to run sidecar containers alongside your primary task container. These sidecars can provide services, perform data synchronization, or handle logging.

```python
from flytekit import task
from flytekitplugins.pod.task import Pod
from kubernetes.client.models import V1PodSpec, V1Container

@task(
    task_config=Pod(
        pod_spec=V1PodSpec(
            containers=[
                V1Container(name="my-primary-container", image="cr.flyte.org/flyteorg/flytekit:py3.9-latest"),
                V1Container(name="data-fetcher", image="alpine/git", command=["git", "clone", "https://github.com/my-org/my-data.git", "/data"]),
                V1Container(name="logger-agent", image="fluentd/fluentd:v1.14-debian", args=["-c", "/etc/fluentd/fluentd.conf"]),
            ]
        ),
        primary_container_name="my-primary-container",
    )
)
def task_with_multiple_sidecars():
    """
    This task runs with two sidecar containers: a data-fetcher and a logger-agent.
    """
    print("Main task logic executing...")
```

### Customizing Pod Metadata

You can attach Kubernetes labels and annotations to the Pod created for your task using the `labels` and `annotations` arguments in the `Pod` configuration.

```python
from flytekit import task
from flytekitplugins.pod.task import Pod
from kubernetes.client.models import V1PodSpec, V1Container

@task(
    task_config=Pod(
        pod_spec=V1PodSpec(
            containers=[
                V1Container(name="my-primary-container", image="cr.flyte.org/flyteorg/flytekit:py3.9-latest"),
            ]
        ),
        primary_container_name="my-primary-container",
        labels={"environment": "production", "team": "data-science"},
        annotations={"kubernetes.io/description": "A critical data processing task"},
    )
)
def task_with_custom_metadata():
    print("This task's Pod will have custom labels and annotations.")
```

## Important Considerations

*   **Primary Container Overwrites**: Be aware that Flyte will overwrite specific attributes (`image`, `command`, `args`, `resources`, `env`) of the container designated as `primary_container_name` to ensure the Python function executes correctly. If you need full control over these attributes, consider running your Python code within a sidecar and having the primary container simply wait for the sidecar to complete, or use a different Flyte task type.
*   **Local Execution**: When running Pod tasks locally (e.g., using `pyflyte run`), the custom Pod configuration is not applied. The task executes as a standard Python function. A warning is logged to indicate that the local environment may not match the remote Pod environment, which could lead to discrepancies.
*   **Kubernetes Knowledge**: Effective use of the Pod Task Plugin requires a good understanding of Kubernetes Pods, containers, and their specifications. Refer to the official Kubernetes documentation for `V1PodSpec` details.
*   **Resource Management**: While you can define resources for sidecar containers directly in the `V1PodSpec`, the resources for the primary container are still managed by Flyte's standard task resource definitions (e.g., `requests` and `limits` arguments in the `@task` decorator). Flyte will apply these to the primary container within the Pod.

## Best Practices

*   **Use for Advanced Scenarios**: Reserve the Pod Task Plugin for scenarios where standard Flyte task configurations are insufficient, such as:
    *   Running multiple containers (sidecars) within a single Pod.
    *   Mounting complex volumes (e.g., `hostPath`, `emptyDir`, `configMap`, `secret`).
    *   Applying specific Kubernetes scheduling constraints (e.g., `nodeSelector`, `affinity`, `tolerations`).
    *   Configuring Pod-level security contexts or service accounts.
    *   Utilizing init containers for setup tasks.
*   **Clear Primary Container Naming**: Always explicitly define `primary_container_name` to avoid ambiguity and ensure Flyte correctly identifies the main execution container.
*   **Separate Concerns**: If your task involves complex setup or external dependencies, consider using init containers or sidecar containers to manage those concerns, keeping your primary Python function focused on its core logic.
*   **Test Thoroughly**: Due to the direct interaction with Kubernetes, thoroughly test Pod tasks in your target Flyte environment to ensure the `V1PodSpec` behaves as expected.
<!--
key: summary_pod_task_plugin_4514718f-a694-48f7-95bb-7f3ca8f4efbb
type: summary_end

-->
<!--
code_unit: flytekitplugins.pod.task
code_unit_type: class
help_text: ''
key: example_a332d3b4-6a23-4d47-9cba-c729a6a86ece
type: example

-->