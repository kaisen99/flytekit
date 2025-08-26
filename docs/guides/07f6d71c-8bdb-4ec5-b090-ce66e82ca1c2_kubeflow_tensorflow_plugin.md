
<!--
help_text: ''
key: summary_kubeflow_tensorflow_plugin_84572c03-4191-4982-bc26-4e0a3c502d3e
modules:
- flytekitplugins.kftensorflow.task
questions_to_answer: []
type: summary

-->
The Kubeflow TensorFlow Plugin enables defining and executing distributed TensorFlow training jobs directly within Flyte workflows. It integrates with Kubeflow's TFJob operator to orchestrate TensorFlow training on Kubernetes clusters, allowing users to specify the architecture of their distributed training setup, including chief, worker, parameter server (PS), and evaluator replica groups.

### Core Concepts

The plugin centers around the `TensorflowFunctionTask` which serves as the primary interface for submitting TensorFlow jobs. This task leverages a `TfJob` configuration object to define the distributed training setup.

*   **`TensorflowFunctionTask`**: This class is a plugin that submits a TFJob to a Kubernetes cluster. It wraps a Python function that contains the TensorFlow training logic. When a task is defined using `TensorflowFunctionTask`, the plugin translates the provided configuration into a Kubeflow TFJob resource, which is then submitted to the Kubernetes API server. The task type is internally identified as `"tensorflow"`.

*   **`TfJob`**: This is the central configuration object for a TensorFlow job. It allows users to specify the details for each replica group involved in the distributed training. A `TfJob` instance defines the desired state for the Chief, PS, Worker, and Evaluator components, along with overall job execution policies.

*   **Replica Configuration (`Chief`, `PS`, `Worker`, `Evaluator`)**: These classes define the specifications for each type of replica in a distributed TensorFlow job. Each replica configuration allows specifying:
    *   `image`: The Docker image to use for the replica pods.
    *   `requests` and `limits`: Resource requests and limits (CPU, memory) for the pods. These are defined using a `Resources` object.
    *   `replicas`: The number of instances for this replica group. For `Evaluator`, `replicas` defaults to 0, meaning no evaluators are launched unless explicitly configured.
    *   `restart_policy`: How pods in this replica group should be restarted if they fail. Options are defined by `RestartPolicy`.

*   **`RestartPolicy`**: An enumeration that specifies the restart behavior for replica pods.
    *   `ALWAYS`: Always restart the pod.
    *   `FAILURE`: Restart the pod only on failure.
    *   `NEVER`: Never restart the pod.

*   **`RunPolicy`**: This class defines a set of policies that apply to the overall execution of the Kubeflow job. It includes:
    *   `clean_pod_policy`: Specifies how to handle pods after the job completes, defined by `CleanPodPolicy`.
    *   `ttl_seconds_after_finished`: The time-to-live for cleaning up finished jobs.
    *   `active_deadline_seconds`: The maximum duration a job can remain active.
    *   `backoff_limit`: The number of retries before marking the job as failed.

*   **`CleanPodPolicy`**: An enumeration that describes how to deal with pods when the job is finished.
    *   `NONE`: Do not clean up pods.
    *   `ALL`: Clean up all pods.
    *   `RUNNING`: Clean up only running pods.

### Usage

To use the Kubeflow TensorFlow Plugin, define a Python function that encapsulates your TensorFlow training logic and decorate it with `TensorflowFunctionTask`, providing a `TfJob` configuration.

#### Defining a TensorFlow Training Task

First, import the necessary components:

```python
from flytekitplugins.kftensorflow.task import (
    Chief,
    Evaluator,
    PS,
    RunPolicy,
    TensorflowFunctionTask,
    TfJob,
    Worker,
    RestartPolicy,
    CleanPodPolicy,
)
from flytekit.types.resources import Resources
from typing import Tuple
```

Next, define your `TfJob` configuration. This example sets up a distributed training job with one chief, two workers, and one parameter server, along with resource requests and a run policy.

```python
tensorflow_job_config = TfJob(
    chief=Chief(
        replicas=1,
        image="your-custom-tensorflow-image:latest",
        requests=Resources(cpu="1", mem="2Gi"),
        restart_policy=RestartPolicy.ON_FAILURE,
    ),
    worker=Worker(
        replicas=2,
        image="your-custom-tensorflow-image:latest",
        requests=Resources(cpu="2", mem="4Gi"),
        restart_policy=RestartPolicy.ALWAYS,
    ),
    ps=PS(
        replicas=1,
        image="your-custom-tensorflow-image:latest",
        requests=Resources(cpu="0.5", mem="1Gi"),
        restart_policy=RestartPolicy.ALWAYS,
    ),
    evaluator=Evaluator(
        replicas=0, # No evaluators for this example
    ),
    run_policy=RunPolicy(
        clean_pod_policy=CleanPodPolicy.ALL,
        ttl_seconds_after_finished=300,
        backoff_limit=3,
    ),
)
```

Finally, define your Python function and apply the `TensorflowFunctionTask` decorator with the `TfJob` configuration:

```python
@TensorflowFunctionTask(task_config=tensorflow_job_config)
def distributed_tensorflow_training_task(epochs: int, learning_rate: float) -> Tuple[float, float]:
    """
    This function represents your distributed TensorFlow training logic.
    The actual training code would run within the Kubeflow TFJob pods.
    """
    import os
    import tensorflow as tf

    # Example: Accessing TF_CONFIG for distributed setup
    tf_config = os.environ.get("TF_CONFIG")
    if tf_config:
        print(f"TF_CONFIG: {tf_config}")
    else:
        print("TF_CONFIG not found, running in non-distributed mode or misconfigured.")

    # Your TensorFlow model definition and training loop here
    # This is a placeholder for actual TensorFlow code
    print(f"Running TensorFlow training for {epochs} epochs with learning rate {learning_rate}")

    # Simulate training results
    final_loss = 0.123
    accuracy = 0.987
    return final_loss, accuracy
```

When this Flyte task executes, it will launch a Kubeflow TFJob on the Kubernetes cluster according to the `tensorflow_job_config`. The Python function `distributed_tensorflow_training_task` will be executed within the chief, worker, PS, and evaluator pods as defined by the TFJob specification.

### Advanced Configuration

#### Resource Management

Resource requests and limits for CPU and memory can be specified for each replica group using the `Resources` object. This allows fine-grained control over the compute resources allocated to your TensorFlow training pods.

```python
from flytekit.types.resources import Resources

# ... inside Chief, PS, Worker, or Evaluator configuration
requests=Resources(cpu="2", mem="4Gi", gpu="1"), # Request 2 CPU, 4GB memory, 1 GPU
limits=Resources(cpu="4", mem="8Gi", gpu="1"),   # Limit to 4 CPU, 8GB memory, 1 GPU
```

#### Image Customization

Each replica group can specify its own Docker image. This is useful if different roles (e.g., chief vs. worker) require different dependencies or TensorFlow versions. If not specified, the default image configured for the Flyte execution environment will be used.

```python
# ... inside Chief, PS, Worker, or Evaluator configuration
image="my-custom-tf-worker-image:v2.0",
```

### Important Considerations

*   **Deprecated Configuration Fields**: The `TfJob` configuration previously supported top-level fields like `num_workers`, `num_ps_replicas`, `num_chief_replicas`, and `num_evaluator_replicas`. These fields are deprecated. You must use the nested `worker.replicas`, `ps.replicas`, `chief.replicas`, and `evaluator.replicas` fields respectively. The plugin includes validation to prevent using both the deprecated and new fields simultaneously.
*   **Kubeflow Environment**: This plugin requires a Kubernetes cluster with the Kubeflow TFJob operator installed and configured. Without the operator, the submitted TFJobs will not be reconciled or executed.
*   **Distributed Training Setup**: Your TensorFlow code within the decorated function must be written to correctly handle distributed training, typically by leveraging the `TF_CONFIG` environment variable that Kubeflow's TFJob operator injects into the pods.
*   **Performance**: The performance of your distributed TensorFlow job is highly dependent on the allocated resources, network configuration within your Kubernetes cluster, and the efficiency of your TensorFlow training code. Ensure appropriate resource requests and limits are set based on your model's requirements.
<!--
key: summary_kubeflow_tensorflow_plugin_84572c03-4191-4982-bc26-4e0a3c502d3e
type: summary_end

-->
<!--
code_unit: flytekitplugins.kftensorflow.task
code_unit_type: class
help_text: ''
key: example_1c12f68e-62e4-43cf-bae7-79f12d9016cb
type: example

-->