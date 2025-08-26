
<!--
help_text: ''
key: summary_kubeflow_mpi_integration_50d150d0-a681-481c-85f7-b569657d22cd
modules:
- flytekitplugins.kfmpi.task
questions_to_answer: []
type: summary

-->
Kubeflow MPI Integration

This integration facilitates running distributed training jobs on Kubernetes using the MPI (Message Passing Interface) operator. It is particularly useful for frameworks like Horovod that leverage MPI for inter-process communication. The core components enable defining and executing MPI-based distributed training workflows.

### Defining MPI Jobs

The `MPIJob` object configures a generic MPI distributed training job. It specifies the setup for the `launcher` and `worker` replica groups, along with overall `run_policy` settings.

*   **Slots:** The `slots` attribute defines the number of processing slots per worker, which is crucial for constructing the `mpirun` command.

#### Replica Configuration

The `Launcher` and `Worker` objects define the configuration for the respective replica groups.

*   **Launcher:** Configures the single launcher replica responsible for initiating the MPI job. Properties include:
    *   `command`: An optional list of strings specifying the command to run. If not provided, the launcher uses the command specified in the task signature.
    *   `image`: The container image to use for the launcher pod.
    *   `requests` and `limits`: Resource requirements (CPU, memory, GPU) for the launcher pod.
    *   `replicas`: The number of launcher replicas (typically 1).
    *   `restart_policy`: Defines how the launcher pod should be restarted (`ALWAYS`, `FAILURE`, `NEVER`).
*   **Worker:** Configures the distributed worker replicas. Properties are similar to `Launcher`, but `replicas` will typically be greater than 1 to enable distributed execution.
    *   Resource management: `requests` and `limits` allow specifying CPU, memory, and GPU requirements for each worker replica.
    *   `restart_policy`: Controls how individual worker pods are restarted.

#### Execution Policies

The `RunPolicy` object governs the overall job execution lifecycle.

*   `clean_pod_policy`: Determines how pods are handled after job completion. Options include `NONE` (no cleanup), `ALL` (clean up all pods), and `RUNNING` (clean up only running pods).
*   `ttl_seconds_after_finished`: Sets a time-to-live (in seconds) for finished jobs before they are automatically cleaned up.
*   `active_deadline_seconds`: Specifies the maximum duration (in seconds) the job can remain active before it is terminated.
*   `backoff_limit`: The number of retries before marking the job as failed.

### Defining Horovod Jobs

The `HorovodJob` object extends `MPIJob` to provide specific configurations for Horovod-based distributed training. It inherits all `MPIJob` properties and adds Horovod-specific parameters.

*   `verbose`: An optional flag to enable verbose logging for Horovod.
*   `log_level`: An optional string specifying the Horovod log level (e.g., "INFO", "DEBUG").
*   `discovery_script_path`: The path to the discovery script used by Horovod for host discovery within the cluster. The default is `/etc/mpi/discover_hosts.sh`.
*   `elastic_timeout`: The timeout in seconds for Horovod's elastic functionality.

### Implementing MPI Tasks

Tasks are defined using Python functions and configured with the appropriate job objects.

#### Generic MPI Tasks

Use `MPIFunctionTask` to define a task that executes a generic MPI job. The task automatically constructs the `mpirun` command, including arguments like `-np` (number of processes), which is derived from `worker.replicas * slots`.

```python
from flytekitplugins.kfmpi.task import MPIFunctionTask, MPIJob, Worker, Launcher
from flytekit.types.resources import Resources
from flytekit import task

@task
def my_mpi_training_job(epochs: int) -> float:
    """
    This function will be executed by each MPI process.
    Your MPI training code should be placed here.
    """
    import os
    # Example: Accessing MPI rank (requires MPI to be initialized in the container)
    # This is a placeholder, actual MPI initialization depends on your library (e.g., mpi4py)
    rank = os.environ.get("OMPI_COMM_WORLD_RANK", "0")
    size = os.environ.get("OMPI_COMM_WORLD_SIZE", "1")
    print(f"Running MPI training for {epochs} epochs on rank {rank} of {size}")
    return 0.5 # Example metric

# Define the MPI task
mpi_task = MPIFunctionTask(
    task_config=MPIJob(
        worker=Worker(
            replicas=2,
            requests=Resources(cpu="1", mem="2Gi"),
            image="my-custom-mpi-image:latest", # Image must have MPI installed
        ),
        launcher=Launcher(
            requests=Resources(cpu="0.5", mem="1Gi"),
            image="my-custom-mpi-image:latest",
        ),
        slots=1,
    ),
    task_function=my_mpi_training_job,
)
```

#### Horovod Tasks

Use `HorovodFunctionTask` for tasks specifically designed for Horovod. This task automatically generates the `horovodrun` command, leveraging `HorovodJob` for configuration. The command includes Horovod-specific arguments such as `--np`, `--log-level`, `--network-interface`, `--slots-per-host`, and `--host-discovery-script`.

```python
from flytekitplugins.kfmpi.task import HorovodFunctionTask, HorovodJob, Worker, Launcher, RestartPolicy
from flytekit.types.resources import Resources
from flytekit import task

@task
def my_horovod_training_job(epochs: int) -> float:
    """
    This function will be executed by each Horovod process.
    Your Horovod training code should be placed here.
    """
    import horovod.tensorflow as hvd
    hvd.init() # Initialize Horovod
    print(f"Running Horovod training for {epochs} epochs on rank {hvd.rank()} of {hvd.size()}")
    # Example: Simulate training
    return 0.9 # Example metric

# Define the Horovod task
horovod_task = HorovodFunctionTask(
    task_config=HorovodJob(
        worker=Worker(
            replicas=4,
            requests=Resources(cpu="2", mem="4Gi", gpu="1"), # Requesting 1 GPU per worker
            image="my-horovod-gpu-image:latest", # Image must have Horovod and GPU drivers
            restart_policy=RestartPolicy.FAILURE,
        ),
        launcher=Launcher(
            requests=Resources(cpu="1", mem="2Gi"),
            image="my-horovod-gpu-image:latest",
        ),
        slots=1,
        verbose=True,
        log_level="DEBUG",
        elastic_timeout=1800, # 30 minutes
    ),
    task_function=my_horovod_training_job,
)
```

### Important Considerations

*   **Image Requirements:** The container images specified for `launcher` and `worker` must have MPI (e.g., Open MPI) and/or Horovod installed, along with all necessary dependencies for your training code. For GPU-accelerated training, ensure the image includes appropriate GPU drivers and CUDA libraries.
*   **Resource Allocation:** Carefully configure `requests` and `limits` for CPU, memory, and especially GPU resources. This ensures efficient scheduling on the Kubernetes cluster and prevents resource starvation or over-provisioning.
*   **Deprecated Arguments:** The `num_workers` and `num_launcher_replicas` arguments within `MPIJob` and `HorovodJob` are deprecated. Always use `worker.replicas` and `launcher.replicas` respectively for configuring replica counts.
*   **Host Discovery:** For Horovod, ensure the `discovery_script_path` points to a valid script within your container image that can correctly identify hosts in the Kubernetes cluster. The default `/etc/mpi/discover_hosts.sh` is typically provided by the MPI operator.
*   **Distributed Environment Variables:** The `MPIFunctionTask` automatically sets common MPI environment variables like `LD_LIBRARY_PATH`, `PATH`, `NCCL_DEBUG`, `OMPI_MCA_orte_default_hostfile`, etc., to facilitate proper MPI communication and performance.
*   **Error Handling and Debugging:** Leverage `RunPolicy` settings like `backoff_limit` to manage job failures and `clean_pod_policy` for inspecting logs of failed pods. The `log_level` and `verbose` options in `HorovodJob` are particularly useful for debugging Horovod-specific issues.
<!--
key: summary_kubeflow_mpi_integration_50d150d0-a681-481c-85f7-b569657d22cd
type: summary_end

-->
<!--
code_unit: flytekitplugins.kfmpi.examples.mpi_training_job
code_unit_type: class
help_text: ''
key: example_f2d70a55-ba42-4689-89f2-eb6e768e1dc7
type: example

-->