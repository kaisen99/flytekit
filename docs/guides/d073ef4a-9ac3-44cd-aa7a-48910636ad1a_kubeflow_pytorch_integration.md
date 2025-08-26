
<!--
help_text: ''
key: summary_kubeflow_pytorch_integration_42d53449-a2cb-4ec9-8b58-bcc7f547da66
modules:
- flytekitplugins.kfpytorch.task
questions_to_answer: []
type: summary

-->
Kubeflow PyTorch Integration enables running distributed PyTorch training jobs on Kubernetes, leveraging Kubeflow's capabilities. This integration provides two primary modes for executing PyTorch workloads: standard distributed training using the Kubeflow PyTorch Operator, and elastic training powered by `torch.distributed.elastic` (torchrun).

## Standard Distributed PyTorch Training

This mode is suitable for fixed-size distributed PyTorch training jobs, where the number of master and worker replicas is predetermined. It utilizes the Kubeflow PyTorch Operator to manage the lifecycle of the training job on the Kubernetes cluster.

### Configuration

The `PyTorch` configuration object defines the parameters for your distributed PyTorch job.

*   **Master Replica Configuration**: The `master` field, an instance of `Master`, configures the single master replica. You can specify:
    *   `image`: The Docker image for the master pod. If not specified, it defaults to the task's container image.
    *   `requests` and `limits`: Resource requests and limits (CPU, memory) for the master pod.
    *   `restart_policy`: Defines how the master pod should be restarted if it fails. Options are `RestartPolicy.ALWAYS`, `RestartPolicy.FAILURE`, or `RestartPolicy.NEVER`.

*   **Worker Replica Configuration**: The `worker` field, an instance of `Worker`, configures the worker replica group. You can specify:
    *   `image`: The Docker image for worker pods. Defaults to the task's container image.
    *   `requests` and `limits`: Resource requests and limits for worker pods.
    *   `replicas`: The number of worker replicas. This is the primary way to scale your distributed training.
    *   `restart_policy`: Defines how worker pods should be restarted.

*   **Run Policy**: The `run_policy` field, an optional instance of `RunPolicy`, controls the overall execution policy of the PyTorch job:
    *   `clean_pod_policy`: Determines how pods are cleaned up after the job completes. Options include `CleanPodPolicy.NONE` (no cleanup), `CleanPodPolicy.ALL` (clean up all pods), or `CleanPodPolicy.RUNNING` (clean up only running pods).
    *   `ttl_seconds_after_finished`: Specifies the time (in seconds) before a finished PyTorchJob is cleaned up.
    *   `active_deadline_seconds`: The maximum duration (in seconds) the job can remain active.
    *   `backoff_limit`: The number of retries before the job is marked as failed.

**Deprecated Fields**:
*   `num_workers`: This field is deprecated. Use `worker.replicas` instead to specify the number of worker replicas.
*   `increase_shared_mem`: This field is deprecated. Use the `shared_memory` argument in the `@task` decorator to configure shared memory for the task's pod.

### Implementation

To use standard distributed PyTorch training, decorate your Python function with `@task` and provide an instance of `PyTorch` to the `task_config` argument. The `PyTorchFunctionTask` plugin handles the conversion of this configuration into a Kubeflow PyTorchJob definition.

```python
from flytekit import task
from flytekitplugins.kfpytorch.task import PyTorch, Worker, Master, RestartPolicy, CleanPodPolicy, RunPolicy

@task(
    task_config=PyTorch(
        worker=Worker(replicas=2, restart_policy=RestartPolicy.ON_FAILURE),
        master=Master(restart_policy=RestartPolicy.ON_FAILURE),
        run_policy=RunPolicy(clean_pod_policy=CleanPodPolicy.ALL, ttl_seconds_after_finished=300)
    ),
    container_image="your_pytorch_image:latest",
    requests="cpu=2,mem=4Gi",
    limits="cpu=4,mem=8Gi",
    shared_memory="2Gi" # Preferred way to increase shared memory
)
def my_distributed_pytorch_training_job(epochs: int) -> float:
    # Your distributed PyTorch training code here
    # This function will be executed by each replica (master and workers)
    # Use torch.distributed for communication
    import torch.distributed as dist
    if dist.is_initialized():
        print(f"Rank {dist.get_rank()} / {dist.get_world_size()} running...")
    else:
        print("Distributed not initialized, running in single process mode.")
    return 0.95 # Example accuracy
```

The `PyTorchFunctionTask` automatically sets up the necessary environment variables for `torch.distributed` to function correctly within the Kubeflow PyTorchJob.

## PyTorch Elastic Training

PyTorch Elastic Training, powered by `torch.distributed.elastic` (also known as `torchrun`), provides fault-tolerant and elastic distributed training. This mode is ideal for scenarios where worker nodes might be preempted or where you need dynamic scaling. It supports both single-node and multi-node distributed training.

### Configuration

The `Elastic` configuration object defines the parameters for your elastic PyTorch training job.

*   **Node and Process Configuration**:
    *   `nnodes`: Specifies the number of nodes. This can be an integer (e.g., `1` for single-node, `2` for two nodes) or a range string (e.g., `"1:2"` for 1 to 2 nodes).
    *   `nproc_per_node`: The number of worker processes to launch per node.
    *   `start_method`: The multiprocessing start method, typically `"spawn"` (default, recommended for robustness) or `"fork"`. When using `"spawn"`, the task function is serialized using `cloudpickle` to ensure it can be correctly launched in child processes.

*   **Elasticity Parameters**:
    *   `monitor_interval`: The interval (in seconds) to monitor the state of workers.
    *   `max_restarts`: The maximum number of worker group restarts before the job fails.

*   **Rendezvous Configuration**:
    *   `rdzv_configs`: A dictionary for additional rendezvous configurations, such as `timeout` and `join_timeout`. Default timeouts are set to 15 minutes to accommodate varying pod startup times. The `c10d` backend is used for rendezvous, which does not require a separate backend server.

*   **Run Policy**: The `run_policy` field, an optional instance of `RunPolicy`, is identical to the one used in standard PyTorch training, controlling cleanup, TTL, active deadline, and backoff limits.

**Deprecated Field**:
*   `increase_shared_mem`: This field is deprecated. Use the `shared_memory` argument in the `@task` decorator to configure shared memory for the task's pod.

### Implementation

To enable PyTorch Elastic training, decorate your Python function with `@task` and provide an instance of `Elastic` to the `task_config` argument. The `PytorchElasticFunctionTask` plugin handles the execution.

```python
from flytekit import task
from flytekitplugins.kfpytorch.task import Elastic, RunPolicy, CleanPodPolicy

@task(
    task_config=Elastic(
        nnodes="1:2", # Elastic training across 1 to 2 nodes
        nproc_per_node=4, # 4 processes per node
        start_method="spawn",
        rdzv_configs={"timeout": 1800, "join_timeout": 1200},
        run_policy=RunPolicy(clean_pod_policy=CleanPodPolicy.ALL)
    ),
    container_image="your_pytorch_elastic_image:latest",
    requests="cpu=2,mem=4Gi",
    limits="cpu=4,mem=8Gi",
    shared_memory="2Gi"
)
def my_elastic_pytorch_training_job(data_path: str) -> float:
    # Your elastic PyTorch training code here
    # This function will be executed by each worker process
    import os
    import torch.distributed as dist
    if not dist.is_initialized():
        # Initialize distributed process group
        rank = int(os.environ["RANK"])
        world_size = int(os.environ["WORLD_SIZE"])
        master_addr = os.environ["MASTER_ADDR"]
        master_port = os.environ["MASTER_PORT"]
        dist.init_process_group(backend="nccl", rank=rank, world_size=world_size,
                                init_method=f"tcp://{master_addr}:{master_port}")
    
    print(f"Elastic worker: Rank {dist.get_rank()} / {dist.get_world_size()} on node {os.environ.get('GROUP_RANK')}")
    # Perform training...
    return 0.98 # Example accuracy
```

#### Execution Behavior

The `PytorchElasticFunctionTask` adapts its execution based on the `nnodes` configuration:

*   **Single-Node Elastic Training (`nnodes=1`)**: The task runs as a standard Python task within a single Kubernetes pod. The `elastic_launch` utility from `torch.distributed.launcher.api` is used internally to manage the multiple processes within that single pod.
*   **Multi-Node Elastic Training (`nnodes > 1`)**: The task is submitted as a Kubeflow PyTorchJob. The Kubeflow PyTorch Operator then manages the creation and orchestration of multiple pods (nodes) for the elastic training.

#### `OMP_NUM_THREADS` Environment Variable

For optimal performance and to prevent system overload, the `PytorchElasticFunctionTask` automatically sets the `OMP_NUM_THREADS` environment variable to `1` if it's not already set and `nproc_per_node` is greater than 1. This mirrors the default behavior of `torchrun`. You can override this by explicitly setting `OMP_NUM_THREADS` in your task's environment variables.

#### Output Handling

In multi-process elastic training, only the result from the global rank 0 worker is returned as the task's output. Results from other workers are ignored. The `ElasticWorkerResult` named tuple is used internally to encapsulate the return value, decks, and output metadata from each worker process.

#### Error Handling

The `PytorchElasticFunctionTask` handles exceptions from worker processes. If a `FlyteRecoverableException` occurs in any worker, the entire task is marked as recoverable. Other exceptions are re-raised as `FlyteUserRuntimeException`. `SignalException` from the elastic launcher indicates a graceful termination, leading to `IgnoreOutputs`.

## Shared Memory Configuration

PyTorch applications, especially those using multi-processed data loaders, often require more shared memory than the default container settings provide. To address this, you can configure the shared memory size for your task's pod.

The recommended way to increase shared memory is by using the `shared_memory` argument in the `@task` decorator:

```python
from flytekit import task
from flytekitplugins.kfpytorch.task import PyTorch, Worker

@task(
    task_config=PyTorch(worker=Worker(replicas=1)),
    shared_memory="2Gi" # Allocate 2GB of shared memory
)
def my_pytorch_task():
    # ...
    pass
```

This configures the task's pod template to mount an `emptyDir` volume with `Memory` medium to `/dev/shm`, effectively increasing the shared memory available to processes within the pod. The upper limit for shared memory is the sum of the memory limits of the containers in the pod.

The `increase_shared_mem` field in both `PyTorch` and `Elastic` configuration objects is deprecated. While it still functions, it is recommended to use the `shared_memory` argument in the `@task` decorator for consistency and clarity.

## Common Policies and Resource Management

Both standard and elastic PyTorch integrations allow fine-grained control over job execution and resource allocation.

*   **Resource Allocation**: Use the `requests` and `limits` arguments in the `@task` decorator or within the `Master` and `Worker` configurations to specify CPU and memory requirements for your pods. This ensures your jobs receive adequate resources and helps with cluster scheduling.
*   **Restart Policies**: The `RestartPolicy` enum (`ALWAYS`, `FAILURE`, `NEVER`) provides control over how individual replica pods are restarted upon termination.
*   **Clean Pod Policies**: The `CleanPodPolicy` enum (`NONE`, `ALL`, `RUNNING`) within `RunPolicy` dictates the cleanup behavior of pods after the PyTorch job completes, which is crucial for resource management on the cluster.

## Best Practices and Considerations

*   **Choosing the Right Mode**:
    *   Use **Standard Distributed PyTorch Training** for simpler, fixed-size distributed jobs where fault tolerance beyond basic restarts is not a primary concern.
    *   Opt for **PyTorch Elastic Training** when you need fault tolerance against node failures, dynamic scaling, or when running on preemptible instances.
*   **Resource Sizing**: Carefully determine the `requests` and `limits` for CPU and memory for your master and worker pods. Under-provisioning can lead to out-of-memory errors or slow training, while over-provisioning wastes cluster resources.
*   **Shared Memory**: Always consider increasing shared memory (`shared_memory` in `@task`) if your PyTorch code uses multiprocessing (e.g., for data loading with multiple workers).
*   **Rendezvous Configuration (Elastic)**: For elastic training, adjust `rdzv_configs` timeouts (`timeout`, `join_timeout`) based on your cluster's scaling behavior and image pull times. Longer timeouts might be necessary if new nodes need to be provisioned or large images pulled.
*   **Debugging Distributed Jobs**: Distributed training can be complex to debug. Ensure your logging is robust and consider using tools like TensorBoard for monitoring training progress.
*   **Deprecations**: Pay attention to deprecation warnings for `num_workers` and `increase_shared_mem`. Update your code to use `worker.replicas` and the `shared_memory` argument in `@task` respectively.
<!--
key: summary_kubeflow_pytorch_integration_42d53449-a2cb-4ec9-8b58-bcc7f547da66
type: summary_end

-->
<!--
code_unit: flytekitplugins.kfpytorch.examples.pytorch_distributed_training
code_unit_type: class
help_text: ''
key: example_557d1c1f-7b65-46b7-8c32-f0d1f7fbbfd7
type: example

-->