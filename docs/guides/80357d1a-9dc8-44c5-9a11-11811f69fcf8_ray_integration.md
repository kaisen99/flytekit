
<!--
help_text: ''
key: summary_ray_integration_4836b65a-418a-42bf-bb36-22dbdbc3c8ba
modules:
- flytekitplugins.ray.models
- flytekitplugins.ray.task
questions_to_answer: []
type: summary

-->
## Ray Integration

Ray Integration enables the execution of distributed Ray applications as tasks within Flyte workflows. This integration leverages KubeRay to manage the lifecycle of Ray clusters on Kubernetes, allowing users to define and run scalable Ray jobs seamlessly.

### Defining Ray Tasks

To execute a Ray application as a Flyte task, wrap your Python function with `RayFunctionTask`. This task type requires a `RayJobConfig` object to specify the configuration for the underlying Ray cluster and job.

The `RayFunctionTask` takes your Python function and the `RayJobConfig` as arguments:

```python
from flytekitplugins.ray.task import RayFunctionTask, RayJobConfig, WorkerNodeConfig, HeadNodeConfig
from flytekit.types.resources import Resources

# Define your Ray application function
def my_ray_application(input_data):
    import ray
    ray.init() # Ray is initialized by the plugin for remote execution
    # Your distributed Ray code here
    results = ray.get([ray.remote(lambda x: x * 2).remote(i) for i in input_data])
    return sum(results)

# Configure the Ray cluster and job
ray_config = RayJobConfig(
    head_node_config=HeadNodeConfig(
        requests=Resources(cpu="1", mem="2Gi"),
        ray_start_params={"dashboard-host": "0.0.0.0"},
    ),
    worker_node_config=[
        WorkerNodeConfig(
            group_name="default-worker-group",
            replicas=2,
            min_replicas=1,
            max_replicas=5,
            requests=Resources(cpu="0.5", mem="1Gi"),
            ray_start_params={"num-cpus": "0.5"},
        )
    ],
    runtime_env={"pip": ["pandas", "numpy"]},
    shutdown_after_job_finishes=True,
    ttl_seconds_after_finished=300, # Keep cluster for 5 minutes after job finishes
    enable_autoscaling=True,
)

# Create the Ray task
ray_task = RayFunctionTask(
    task_config=ray_config,
    task_function=my_ray_application,
)
```

### Configuring Ray Cluster Nodes

The `RayJobConfig` object allows detailed specification of the Ray cluster's head and worker nodes.

#### Head Node Configuration

Use `HeadNodeConfig` to define the properties of the Ray head node:

*   **`ray_start_params`**: A dictionary of parameters passed directly to `ray start` on the head node.
*   **`pod_template`**: An optional `PodTemplate` object to provide a custom Kubernetes pod specification for the head node. This allows fine-grained control over the pod's configuration.
*   **`requests`** and **`limits`**: Optional `Resources` objects to specify CPU and memory requests and limits for the head node's primary container.

**Important:** You cannot specify both `pod_template` and `requests`/`limits` simultaneously for a `HeadNodeConfig`. Choose one method for resource and pod customization.

#### Worker Node Configuration

Define worker groups using a list of `WorkerNodeConfig` objects within `RayJobConfig`. Each `WorkerNodeConfig` specifies a distinct group of Ray workers:

*   **`group_name`**: A unique identifier for the worker group.
*   **`replicas`**: The desired number of worker replicas in this group.
*   **`min_replicas`**: The minimum number of worker replicas.
*   **`max_replicas`**: The maximum number of worker replicas. These are used when `enable_autoscaling` is set to `True`.
*   **`ray_start_params`**: A dictionary of parameters passed directly to `ray start` on the worker nodes in this group.
*   **`pod_template`**: An optional `PodTemplate` object for custom Kubernetes pod specifications for worker nodes in this group.
*   **`requests`** and **`limits`**: Optional `Resources` objects to specify CPU and memory requests and limits for the worker nodes' primary container.

**Important:** Similar to `HeadNodeConfig`, you cannot specify both `pod_template` and `requests`/`limits` simultaneously for a `WorkerNodeConfig`.

### Ray Job Lifecycle Management

Control the lifecycle of the Ray cluster after your job completes using these `RayJobConfig` parameters:

*   **`shutdown_after_job_finishes`**: A boolean flag (defaults to `False`). If set to `True`, the Ray cluster is deleted immediately after the Ray job finishes, regardless of success or failure.
*   **`ttl_seconds_after_finished`**: An optional integer. If specified, the Ray cluster will be retained for this many seconds after the job finishes before being deleted. This is useful for debugging or inspecting logs on the cluster after job completion. This setting takes precedence over `shutdown_after_job_finishes` if both are provided.

### Ray Runtime Environment

The `runtime_env` parameter in `RayJobConfig` is a dictionary that specifies the Ray runtime environment for your job. This allows you to define dependencies, working directories, and other environment configurations for your Ray application.

For remote execution, the `RayFunctionTask` automatically configures the `runtime_env` to include the current working directory of your Flyte task, ensuring that your code and its local dependencies are available to the Ray cluster. You can augment this with additional specifications, such as Python package dependencies:

```python
ray_config = RayJobConfig(
    # ... other configurations ...
    runtime_env={
        "pip": ["scikit-learn==1.0.2", "matplotlib"],
        "env_vars": {"MY_ENV_VAR": "value"},
    }
)
```

### Autoscaling

Set `enable_autoscaling` to `True` in `RayJobConfig` to enable KubeRay's autoscaling capabilities for your Ray cluster. When enabled, KubeRay dynamically adjusts the number of worker nodes within the `min_replicas` and `max_replicas` bounds defined in your `WorkerNodeConfig` based on the Ray cluster's workload.

### Local Execution Behavior

When executing a Flyte workflow locally, the `RayFunctionTask`'s `pre_execute` method calls `ray.init()` within the local environment. If an `address` is specified in `RayJobConfig`, it is passed to `ray.init()`. For remote (non-local) execution, the Flyte agent manages the Ray cluster, and `ray.init()` is not explicitly called by the user's task code; the `pre_execute` method primarily configures the `runtime_env` for the remote cluster.

### Important Considerations

*   **Resource Specification:** Ensure that `pod_template` and `requests`/`limits` are not simultaneously defined for `HeadNodeConfig` or `WorkerNodeConfig` to avoid configuration conflicts.
*   **Dependencies:** All Python dependencies required by your Ray application must be available in the task's container image or specified via the `runtime_env`'s `pip` or `conda` fields.
*   **Ray Start Parameters:** Use `ray_start_params` to pass any specific Ray configuration flags or arguments to the `ray start` command on your head and worker nodes.
<!--
key: summary_ray_integration_4836b65a-418a-42bf-bb36-22dbdbc3c8ba
type: summary_end

-->
<!--
code_unit: flytekitplugins.ray.examples.ray_job_example
code_unit_type: class
help_text: ''
key: example_265477d6-3791-490b-a0ae-f76541f1053c
type: example

-->