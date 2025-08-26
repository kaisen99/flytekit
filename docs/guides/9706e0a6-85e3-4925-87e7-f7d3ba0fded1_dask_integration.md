
<!--
help_text: ''
key: summary_dask_integration_f9dc4a29-a08d-49c6-95fd-aba03cd24495
modules:
- flytekitplugins.dask.models
- flytekitplugins.dask.task
questions_to_answer: []
type: summary

-->
Dask integration enables the execution of Dask computations as native Flyte tasks. This integration leverages Flyte's orchestration capabilities to manage the lifecycle of a Dask cluster, including provisioning the scheduler and worker pods, executing the Dask-based Python function, and tearing down the cluster upon completion.

## Dask Task Configuration

To define a Dask task, use the `Dask` configuration object within the `@task` decorator. This object allows specifying the resources and images for the Dask scheduler and its worker groups.

The `Dask` configuration object comprises two main components:

*   **Scheduler Configuration**: Defines the properties for the Dask scheduler pod.
*   **Worker Group Configuration**: Defines the properties for the Dask worker pods.

### Scheduler Configuration

The `Scheduler` configuration object allows customizing the Dask scheduler pod:

*   **`image`**: Specifies a custom Docker image for the scheduler. If not provided, the task's default image is used. The image must have `dask[distributed]` installed and maintain a consistent Python environment with the worker pods and the task runner.
*   **`requests`**: Defines the CPU and memory resources requested for the scheduler pod. If not specified, the resources requested for the overall task are applied.
*   **`limits`**: Defines the CPU and memory resource limits for the scheduler pod. If not specified, the resource limits for the overall task are applied.

### Worker Group Configuration

The `WorkerGroup` configuration object allows customizing the Dask worker pods:

*   **`number_of_workers`**: Specifies the number of worker pods in the group. This value must be at least 1. Defaults to 1 if not specified.
*   **`image`**: Specifies a custom Docker image for the worker pods. If not provided, the task's default image is used. Similar to the scheduler image, it must have `dask[distributed]` installed and maintain a consistent Python environment.
*   **`requests`**: Defines the CPU and memory resources requested for each worker pod. If not specified, the resources requested for the overall task are applied to each worker.
*   **`limits`**: Defines the CPU and memory resource limits for each worker pod. If not specified, the resource limits for the overall task are applied to each worker.

## How Dask Tasks Operate

When a Flyte task is decorated with a `Dask` configuration, the Dask task plugin intercepts the task definition. During the serialization process, the user-provided `Dask` configuration (which includes `Scheduler` and `WorkerGroup` details) is transformed into an internal Dask job model. This model is then serialized into a protobuf message.

FlytePropeller, the Flyte execution engine, uses this protobuf message to provision a dedicated Dask cluster. This cluster consists of a Dask scheduler pod and the specified number of Dask worker pods. Once the cluster is ready, the user's Python function, which typically contains Dask computations, executes within this distributed environment. After the task completes, the Dask cluster is automatically torn down.

## Defining Dask Tasks

To define a Dask task, import the `Dask`, `Scheduler`, and `WorkerGroup` configuration objects and pass an instance of `Dask` to the `@task` decorator's `task_config` parameter.

### Basic Dask Task

A basic Dask task runs with default scheduler and worker configurations (one worker, using the task's default image and resources).

```python
from flytekit import task
from flytekitplugins.dask import Dask
import dask.dataframe as dd

@task(task_config=Dask())
def my_dask_dataframe_task(csv_path: str) -> float:
    df = dd.read_csv(csv_path)
    result = df.x.mean().compute()
    return float(result)
```

### Customizing Scheduler and Worker Resources

You can specify custom resource requests and limits for both the scheduler and worker pods.

```python
from flytekit import task, Resources
from flytekitplugins.dask import Dask, Scheduler, WorkerGroup
import dask.array as da

@task(
    task_config=Dask(
        scheduler=Scheduler(
            requests=Resources(cpu="1", mem="2Gi"),
            limits=Resources(cpu="2", mem="4Gi"),
        ),
        workers=WorkerGroup(
            number_of_workers=5,
            requests=Resources(cpu="0.5", mem="1Gi"),
            limits=Resources(cpu="1", mem="2Gi"),
        ),
    )
)
def my_distributed_array_task(size: int) -> float:
    x = da.random.random((size, size), chunks=(1000, 1000))
    result = x.mean().compute()
    return float(result)
```

### Using Custom Images for Dask Components

It is possible to specify a custom Docker image for the scheduler and worker pods. This is useful when your Dask environment requires specific libraries or versions not present in the default task image.

```python
from flytekit import task
from flytekitplugins.dask import Dask, Scheduler, WorkerGroup
import dask.dataframe as dd

# Assume 'my_custom_dask_image:latest' is an image with dask[distributed] installed
# and the necessary Python environment.
CUSTOM_DASK_IMAGE = "my_custom_dask_image:latest"

@task(
    task_config=Dask(
        scheduler=Scheduler(image=CUSTOM_DASK_IMAGE),
        workers=WorkerGroup(number_of_workers=3, image=CUSTOM_DASK_IMAGE),
    )
)
def process_large_dataset_with_custom_dask(data_path: str) -> int:
    df = dd.read_parquet(data_path)
    count = df.shape[0].compute()
    return int(count)
```

## Important Considerations and Best Practices

*   **Image Consistency**: Ensure that the Docker images used for the Dask scheduler, workers, and the main task runner pod (where the Python function is defined) have `dask[distributed]` installed and maintain a consistent Python environment. Mismatched environments can lead to serialization errors or unexpected behavior.
*   **Ephemeral Clusters**: Each Dask task creates a new, dedicated Dask cluster for its execution. This cluster is automatically provisioned before the task runs and torn down immediately after completion. This ensures isolation and efficient resource utilization but means state is not persisted between Dask tasks.
*   **Resource Allocation**: Carefully configure `requests` and `limits` for both the scheduler and worker pods. Over-provisioning can lead to wasted resources, while under-provisioning can cause performance bottlenecks or out-of-memory errors. Remember that worker resources are applied *per worker*.
*   **Worker Count**: The `number_of_workers` must be at least 1. Setting an appropriate number of workers depends on the nature of your Dask computation and the available cluster resources.
*   **Dask Client**: Within your task function, Dask automatically detects the cluster environment. You typically do not need to explicitly create a `dask.distributed.Client` unless you require specific client configurations.
*   **Error Handling**: Dask computations can fail due to various reasons (e.g., OOM errors on workers, network issues). Implement robust error handling within your Dask code and leverage Flyte's retry mechanisms for transient failures.
<!--
key: summary_dask_integration_f9dc4a29-a08d-49c6-95fd-aba03cd24495
type: summary_end

-->
<!--
code_unit: flytekitplugins.dask.examples.distributed_dask_task
code_unit_type: class
help_text: ''
key: example_07d3084a-7c34-48e0-a572-c5239ac3a89b
type: example

-->