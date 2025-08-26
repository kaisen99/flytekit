
<!--
help_text: ''
key: summary_kubeflow_mpi_plugin_42ea2117-77b6-4744-ae40-5b41bc86886b
modules:
- flytekitplugins.kfmpi.task
questions_to_answer: []
type: summary

-->
The Kubeflow MPI Plugin enables the execution of distributed Message Passing Interface (MPI) jobs, including Horovod training, directly within Flyte workflows. This plugin leverages the Kubeflow MPI Operator to orchestrate MPI applications on a Kubernetes cluster, allowing users to define and manage distributed training tasks as native Flyte tasks.

## Core Concepts

The plugin introduces several key components to configure and manage distributed MPI jobs:

*   **MPIJob and HorovodJob:** These are configuration objects that define the parameters for an MPI or Horovod distributed training job. They encapsulate settings for the launcher, workers, and the overall job execution policy.
*   **Launcher:** Represents the single replica responsible for initiating the MPI job. It typically runs the `mpirun` command. You can customize its container image, resource requests and limits, and the command it executes.
*   **Worker:** Represents the distributed replicas that perform the main computational work, such as training a machine learning model. You define the number of worker replicas, their container image, resource requests and limits, and an optional custom command.
*   **RunPolicy:** Defines the lifecycle management policies for the Kubeflow MPI job. This includes how pods are cleaned up after the job completes, the time-to-live (TTL) for finished jobs, the active deadline for the job, and the backoff limit for retries.
*   **CleanPodPolicy and RestartPolicy:** These are enumerations used within `RunPolicy` and `Worker`/`Launcher` configurations, respectively, to specify cleanup behavior and pod restart behavior.

## Defining MPI Tasks

To define a generic MPI job, use the `MPIFunctionTask` class. This class extends `PythonFunctionTask`, allowing you to wrap a standard Python function that will serve as the entrypoint for your distributed MPI application.

The `MPIFunctionTask` automatically constructs the `mpirun` command based on your configuration. The `_MPI_BASE_COMMAND` provides a default set of MPI arguments, which can be extended or overridden.

**Example:**

```python
from flytekit import task
from flytekitplugins.kfmpi.task import MPIFunctionTask, MPIJob, Worker, Launcher, RunPolicy, CleanPodPolicy

# Define worker and launcher configurations
mpi_worker_config = Worker(
    replicas=2,  # Number of worker replicas
    image="your_mpi_image:latest", # Image with MPI installed
    requests={"cpu": "1", "memory": "2Gi"},
    limits={"cpu": "2", "memory": "4Gi"},
)

mpi_launcher_config = Launcher(
    image="your_mpi_image:latest", # Image with MPI installed
    requests={"cpu": "0.5", "memory": "1Gi"},
    limits={"cpu": "1", "memory": "2Gi"},
)

# Define the MPI job configuration
mpi_job_config = MPIJob(
    worker=mpi_worker_config,
    launcher=mpi_launcher_config,
    slots=1, # Number of slots per worker
    run_policy=RunPolicy(clean_pod_policy=CleanPodPolicy.ALL),
)

# Define the MPI task using MPIFunctionTask
@task(task_config=mpi_job_config)
def mpi_hello_world_task(message: str) -> str:
    """
    A simple MPI "hello world" task.
    This function will be executed by each MPI process.
    """
    import mpi4py.MPI as MPI
    comm = MPI.COMM_WORLD
    rank = comm.Get_rank()
    size = comm.Get_size()
    print(f"Hello from rank {rank} of {size}: {message}")
    return f"MPI job completed by rank {rank}"

# In a workflow, you would call:
# @workflow
# def my_mpi_workflow(msg: str = "Distributed computation!"):
#     mpi_hello_world_task(message=msg)
```

## Horovod Integration

For distributed training with Horovod, use the `HorovodFunctionTask`. This class extends `MPIFunctionTask` and provides specialized configuration for Horovod jobs, automatically generating the `horovodrun` command prefix.

The `HorovodFunctionTask` expects a `HorovodJob` configuration object, which includes Horovod-specific parameters in addition to the general MPI job settings.

**Example:**

```python
from flytekit import task
from flytekitplugins.kfmpi.task import HorovodFunctionTask, HorovodJob, Worker, Launcher, RunPolicy

# Define worker and launcher configurations for Horovod
horovod_worker_config = Worker(
    replicas=2,
    image="your_horovod_image:latest", # Image with Horovod and MPI installed
    requests={"cpu": "2", "memory": "4Gi", "gpu": "1"},
    limits={"cpu": "4", "memory": "8Gi", "gpu": "1"},
)

horovod_launcher_config = Launcher(
    image="your_horovod_image:latest", # Image with Horovod and MPI installed
    requests={"cpu": "1", "memory": "2Gi"},
    limits={"cpu": "2", "memory": "4Gi"},
)

# Define the Horovod job configuration
horovod_job_config = HorovodJob(
    worker=horovod_worker_config,
    launcher=horovod_launcher_config,
    slots=1, # Number of slots per worker (usually 1 for GPU training)
    verbose=True,
    log_level="DEBUG",
    discovery_script_path="/etc/mpi/discover_hosts.sh", # Default path for host discovery
    elastic_timeout=1800, # Horovod elastic timeout in seconds
    run_policy=RunPolicy(),
)

# Define the Horovod task using HorovodFunctionTask
@task(task_config=horovod_job_config)
def horovod_training_task(epochs: int) -> float:
    """
    A placeholder for a Horovod distributed training task.
    This function would contain your Horovod-enabled training loop.
    """
    import horovod.tensorflow as hvd
    import tensorflow as tf

    hvd.init()
    gpus = tf.config.experimental.list_physical_devices('GPU')
    for gpu in gpus:
        tf.config.experimental.set_memory_growth(gpu, True)
    if gpus:
        tf.config.experimental.set_visible_devices(gpus[hvd.local_rank()], 'GPU')

    # Your Horovod training logic here
    print(f"Horovod rank {hvd.rank()} of {hvd.size()} starting training for {epochs} epochs.")
    # Simulate training
    loss = 0.5 * hvd.rank()
    return float(loss)

# In a workflow, you would call:
# @workflow
# def my_horovod_workflow(num_epochs: int = 10):
#     horovod_training_task(epochs=num_epochs)
```

## Configuration Details

### Replica Configuration (`Worker`, `Launcher`)

Both `Worker` and `Launcher` objects share common configuration attributes:

*   `command` (Optional[List[str]]): Specifies a custom command to override the default entrypoint for the replica. If not provided, the MPI operator generates a default command.
*   `image` (Optional[str]): Defines the Docker image to use for the replica. This is crucial for ensuring the correct MPI environment and application dependencies are available.
*   `requests` (Optional[Resources]): Specifies the Kubernetes resource requests (e.g., CPU, memory, GPU) for the replica's container.
*   `limits` (Optional[Resources]): Specifies the Kubernetes resource limits for the replica's container.
*   `replicas` (Optional[int]): The number of instances for the replica group. For `Worker`, this defines the number of distributed workers. For `Launcher`, this is typically 1.
*   `restart_policy` (Optional[RestartPolicy]): Defines how the replica's pod should be restarted if it terminates. Options include `ALWAYS`, `FAILURE`, and `NEVER`.

### Job Policy (`RunPolicy`)

The `RunPolicy` object controls the overall execution behavior of the MPI job:

*   `clean_pod_policy` (Optional[CleanPodPolicy]): Determines how pods are handled after the job completes.
    *   `NONE`: Pods are not cleaned up.
    *   `ALL`: All pods are cleaned up.
    *   `RUNNING`: Only running pods are cleaned up.
*   `ttl_seconds_after_finished` (Optional[int]): Specifies the duration (in seconds) after which a finished job (succeeded or failed) should be automatically cleaned up by Kubernetes.
*   `active_deadline_seconds` (Optional[int]): Sets the maximum duration (in seconds) for which the job can remain active before it is terminated.
*   `backoff_limit` (Optional[int]): The number of retries before marking the job as failed.

### MPIJob Specifics

*   `slots` (int): The number of slots per worker used in the hostfile. This value directly influences the `-np` argument passed to `mpirun` (total processes = `worker.replicas` * `slots`). Defaults to 1.
*   **Deprecated Fields:** `num_workers` and `num_launcher_replicas` are deprecated. Use `worker.replicas` and `launcher.replicas` respectively for configuring replica counts.

### HorovodJob Specifics

`HorovodJob` inherits all attributes from `MPIJob` and adds Horovod-specific configurations:

*   `verbose` (Optional[bool]): If `True`, enables verbose logging for `horovodrun`. Defaults to `False`.
*   `log_level` (Optional[str]): Sets the log level for `horovodrun` (e.g., "INFO", "DEBUG"). Defaults to "INFO".
*   `discovery_script_path` (Optional[str]): Path to the host discovery script used by Horovod. Defaults to "/etc/mpi/discover_hosts.sh".
*   `elastic_timeout` (Optional[int]): Horovod elastic timeout in seconds. Defaults to 1200.

## Important Considerations

*   **Resource Allocation:** Carefully configure `requests` and `limits` for both `Launcher` and `Worker` replicas. Insufficient resources can lead to job failures or poor performance. For GPU-enabled training, ensure GPU resources are correctly requested.
*   **Container Images:** The specified `image` for `Launcher` and `Worker` must contain the necessary MPI libraries (e.g., Open MPI) and any required machine learning frameworks (e.g., PyTorch, TensorFlow, Horovod). The plugin relies on these being present in the container.
*   **Entrypoint and Command:** The plugin automatically sets the Python function wrapped by `MPIFunctionTask` or `HorovodFunctionTask` as the entrypoint for the MPI processes. If you provide a custom `command` in `Worker` or `Launcher` configurations, ensure it correctly invokes your application or the default MPI/Horovod entrypoint.
*   **Distributed Communication:** MPI jobs require robust inter-pod network communication. Verify that your Kubernetes cluster's network policies allow the necessary communication between MPI pods.
*   **Deprecated Fields:** Always use `worker.replicas` and `launcher.replicas` instead of the deprecated `num_workers` and `num_launcher_replicas` for specifying replica counts.
<!--
key: summary_kubeflow_mpi_plugin_42ea2117-77b6-4744-ae40-5b41bc86886b
type: summary_end

-->
<!--
code_unit: flytekitplugins.kfmpi.task
code_unit_type: class
help_text: ''
key: example_fc0f9b07-3963-416a-86e4-4e0d3a0dcde8
type: example

-->