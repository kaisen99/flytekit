
<!--
help_text: ''
key: summary_ray_plugin_807793b8-2cb4-4178-be13-2d6d4cdfec04
modules:
- flytekitplugins.ray.models
- flytekitplugins.ray.task
questions_to_answer: []
type: summary

-->
The Ray Plugin enables users to define and execute Ray applications as tasks within Flyte workflows. It leverages KubeRay to provision and manage Ray clusters on Kubernetes, allowing for scalable distributed computing directly from Flyte.

### Core Concepts

The plugin orchestrates two primary components:

*   **Ray Job:** Represents a single execution of a Ray application. This is the user-defined Python function decorated as a Flyte task.
*   **Ray Cluster:** The underlying distributed compute environment that executes the Ray Job. A Ray Cluster consists of a head node and one or more worker node groups.

### Defining Ray Tasks

To define a Ray task, use the `@task` decorator and provide a `RayJobConfig` object to its `task_config` parameter. The decorated function's body will be executed within the Ray cluster.

```python
from flytekit import task
from flytekitplugins.ray import RayJobConfig, WorkerNodeConfig, HeadNodeConfig
from flytekit.core.resources import Resources
import ray

# Ensure your container image has Ray installed.
# Example: "rayproject/ray:2.9.0" or a custom image with Ray.
@task(
    task_config=RayJobConfig(
        worker_node_config=[
            WorkerNodeConfig(
                group_name="worker-group-1",
                replicas=2,
                min_replicas=1,
                max_replicas=4,
                requests=Resources(cpu="1", mem="2Gi"),
                limits=Resources(cpu="1", mem="2Gi"),
                ray_start_params={"num-cpus": "1"},
            )
        ],
        head_node_config=HeadNodeConfig(
            requests=Resources(cpu="1", mem="2Gi"),
            limits=Resources(cpu="1", mem="2Gi"),
            ray_start_params={"dashboard-host": "0.0.0.0"},
        ),
        enable_autoscaling=True,
        shutdown_after_job_finishes=True,
        ttl_seconds_after_finished=300, # Only applies if shutdown_after_job_finishes is False
        runtime_env_yaml="""
            pip:
              - pandas==2.1.4
              - numpy==1.26.2
            env_vars:
              MY_ENV_VAR: "my_value"
        """,
    ),
    container_image="ghcr.io/flyteorg/flytekit-ray-example:latest", # Replace with your Ray-enabled image
)
def my_ray_task(n: int) -> float:
    """
    A Flyte task that runs a simple Ray distributed computation.
    """
    @ray.remote
    def f(x):
        return x * x

    futures = [f.remote(i) for i in range(n)]
    results = ray.get(futures)
    return sum(results)

# Example usage in a workflow
# from flytekit import workflow
# @workflow
# def ray_workflow(num_elements: int = 10):
#     return my_ray_task(n=num_elements)
```

### Configuring Ray Clusters

The `RayJobConfig` class provides comprehensive options for configuring the Ray cluster and job behavior:

*   **`worker_node_config`**: A list of `WorkerNodeConfig` objects. Each `WorkerNodeConfig` defines a distinct group of worker nodes within the Ray cluster.
    *   `group_name` (str): A unique name for the worker group.
    *   `replicas` (int): The desired number of worker replicas for this group.
    *   `min_replicas` (Optional[int]): The minimum number of replicas for autoscaling. Defaults to `replicas`.
    *   `max_replicas` (Optional[int]): The maximum number of replicas for autoscaling. Defaults to `replicas`.
    *   `ray_start_params` (Optional[Dict[str, str]]): A dictionary of parameters passed to the `ray start` command on worker nodes.
    *   `pod_template` (Optional[PodTemplate]): Provides a Kubernetes `PodTemplate` for advanced pod configuration.
    *   `requests` (Optional[Resources]): Resource requests (CPU, memory, GPU) for worker pods.
    *   `limits` (Optional[Resources]): Resource limits for worker pods.
    *   **Important:** `pod_template` cannot be specified simultaneously with `requests` or `limits`.

*   **`head_node_config`** (Optional[HeadNodeConfig]): Configures the Ray head node.
    *   `ray_start_params` (Optional[Dict[str, str]]): Parameters passed to `ray start` on the head node.
    *   `pod_template` (Optional[PodTemplate]): Provides a Kubernetes `PodTemplate` for the head pod.
    *   `requests` (Optional[Resources]): Resource requests for the head pod.
    *   `limits` (Optional[Resources]): Resource limits for the head pod.
    *   **Important:** `pod_template` cannot be specified simultaneously with `requests` or `limits`.

*   **`enable_autoscaling`** (bool): If `True`, enables Ray's autoscaling feature for the cluster. Requires `min_replicas` and `max_replicas` to be set in `WorkerNodeConfig`.

*   **`runtime_env`** (Optional[dict]): A dictionary representing Ray's `runtime_env`. This is used to specify dependencies, environment variables, and working directories for the Ray job.
    *   **Note:** This field is deprecated for KubeRay versions 1.1.0 and above. Use `runtime_env_yaml` instead.

*   **`runtime_env_yaml`** (Optional[str]): A YAML string or dictionary for Ray's `runtime_env`. This is the preferred way to specify the runtime environment for KubeRay versions 1.1.0+.

*   **`address`** (Optional[str]): An optional Ray cluster address. Primarily used for local execution to connect to an existing Ray cluster. In a Flyte cluster environment, Flyte manages the cluster creation.

*   **`shutdown_after_job_finishes`** (bool): If `True`, the Ray cluster is deleted immediately after the Ray job completes (successfully or with failure). This is the recommended setting for ephemeral clusters.

*   **`ttl_seconds_after_finished`** (Optional[int]): Specifies the number of seconds after which the RayCluster will be deleted after the RayJob finishes. This parameter is only effective if `shutdown_after_job_finishes` is `False`.

### Resource Allocation

Resource requests and limits for both head and worker nodes are crucial for efficient cluster management. Use `requests` and `limits` within `HeadNodeConfig` and `WorkerNodeConfig` to specify CPU, memory, and GPU requirements. For more granular control over Kubernetes pod specifications, use the `pod_template` field. Remember that `pod_template` is mutually exclusive with `requests` and `limits`.

### Ray Runtime Environment

The `runtime_env` (or `runtime_env_yaml`) parameter in `RayJobConfig` is essential for managing dependencies and code distribution within the Ray cluster. When a Ray task runs on a Flyte cluster, the plugin automatically configures the `working_dir` in the `runtime_env` to ensure your task code is available to all Ray workers. This allows Ray to correctly import modules and execute functions defined in your Flyte project.

### Cluster Lifecycle Management

The plugin provides control over the Ray cluster's lifecycle:

*   Setting `shutdown_after_job_finishes=True` ensures that the Ray cluster is torn down as soon as the Ray job completes, minimizing resource consumption. This is the default and recommended behavior for most use cases.
*   If `shutdown_after_job_finishes` is `False`, the cluster will persist for the duration specified by `ttl_seconds_after_finished`. This can be useful for debugging or if you intend to run multiple Ray jobs on the same cluster, though this pattern is less common in Flyte workflows where tasks are typically isolated.

### Local Execution and Debugging

When a Ray task is executed locally (e.g., using `pyflyte run`), the `RayFunctionTask` automatically initializes a local Ray instance using `ray.init()`. This allows for seamless local development and testing of Ray-enabled Flyte tasks without requiring a full Kubernetes deployment. The `address` parameter in `RayJobConfig` can be used to connect to an existing Ray cluster for local testing, if needed.

### Important Considerations

*   **Container Image:** Ray tasks require a container image that has Ray installed and configured. Ensure your `container_image` in the `@task` decorator points to an image with the necessary Ray libraries.
*   **KubeRay Operator:** The Ray Plugin relies on the KubeRay operator being installed and configured in your Kubernetes cluster. This operator manages the lifecycle of Ray clusters based on the `RayJob` and `RayCluster` specifications.
*   **Resource Management:** Carefully configure resource requests and limits for head and worker nodes to ensure efficient cluster utilization and prevent resource contention within your Kubernetes environment.
*   **Autoscaling:** When `enable_autoscaling` is `True`, ensure that `min_replicas` and `max_replicas` are appropriately set in `WorkerNodeConfig` to define the scaling boundaries for your worker groups.
<!--
key: summary_ray_plugin_807793b8-2cb4-4178-be13-2d6d4cdfec04
type: summary_end

-->
<!--
code_unit: flytekitplugins.ray.task
code_unit_type: class
help_text: ''
key: example_0a02ef89-8f84-41bd-ac52-ebfafe0404ab
type: example

-->