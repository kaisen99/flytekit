
<!--
help_text: ''
key: summary_distributed_training_&_computing_plugins_48e6e27c-57e5-47f2-8fe7-b2106325e503
modules:
- flytekitplugins.kfmpi.task
- flytekitplugins.kfpytorch.task
- flytekitplugins.kftensorflow.task
- flytekitplugins.dask.task
- flytekitplugins.dask.models
- flytekitplugins.ray.task
- flytekitplugins.ray.models
- flytekitplugins.spark.task
- flytekitplugins.spark.generic_task
- flytekitplugins.spark.connector
- flytekitplugins.spark.models
- flytekitplugins.slurm.function.task
- flytekitplugins.slurm.script.task
- flytekitplugins.slurm.function.connector
- flytekitplugins.slurm.script.connector
- flytekitplugins.slurm.ssh_utils
questions_to_answer: []
type: summary

-->
Distributed Training & Computing Plugins

Flytekit provides a suite of plugins to facilitate distributed training and computing across various platforms, including Kubernetes-native operators (Kubeflow MPI, PyTorch, TensorFlow), Dask, Ray, Spark (on Kubernetes and Databricks), and Slurm. These plugins enable seamless orchestration of complex distributed workloads within Flyte workflows, abstracting away much of the underlying infrastructure complexity.

## Kubeflow-based Distributed Training

Flytekit integrates with Kubeflow operators to provide first-class support for distributed machine learning frameworks like MPI (Horovod), PyTorch, and TensorFlow. These plugins allow you to define distributed training jobs directly within your Flyte tasks, leveraging Kubernetes for resource management and orchestration.

### MPI and Horovod

The MPI plugin supports distributed training using the MPI operator, including Horovod. It allows you to configure the launcher and worker replicas for your distributed job.

*   **`MPIJob`**: Configures a generic MPI job.
    *   `launcher`: Defines the configuration for the MPI launcher replica, including `command`, `image`, `requests`, `limits`, `replicas`, and `restart_policy`.
    *   `worker`: Defines the configuration for the MPI worker replicas, with similar options as `launcher`.
    *   `slots`: Specifies the number of slots per worker used in the hostfile (default: 1).
    *   `run_policy`: Controls the job's lifecycle, including `clean_pod_policy`, `ttl_seconds_after_finished`, `active_deadline_seconds`, and `backoff_limit`.
*   **`HorovodJob`**: Extends `MPIJob` with Horovod-specific configurations.
    *   `verbose`: Enables verbose logging for Horovod (default: `False`).
    *   `log_level`: Sets the Horovod log level (default: "INFO").
    *   `discovery_script_path`: Path to the host discovery script (default: "/etc/mpi/discover_hosts.sh").
    *   `elastic_timeout`: Horovod elastic timeout in seconds (default: 1200).

To use these configurations, decorate your Python function with `@task` and pass an instance of `MPIJob` or `HorovodJob` to the `task_config` argument.

```python
from flytekit import task
from flytekitplugins.kfmpi.task import HorovodJob, Worker, Launcher

@task(task_config=HorovodJob(
    worker=Worker(replicas=2, requests={"cpu": "1", "memory": "2Gi"}),
    launcher=Launcher(requests={"cpu": "0.5", "memory": "1Gi"}),
    slots=4,
    verbose=True,
))
def distributed_horovod_training(epochs: int) -> float:
    # Your Horovod training code here
    # This function will be executed by each worker
    pass
```

`MPIFunctionTask` and `HorovodFunctionTask` are the underlying task types that handle the serialization and execution of these distributed jobs. They automatically construct the `mpirun` or `horovodrun` commands based on your configuration.

### PyTorch Distributed Training

The PyTorch plugin supports distributed training using the Kubeflow PyTorch operator or PyTorch Elastic (torchrun).

*   **`PyTorch`**: Configures a PyTorch job using the Kubeflow PyTorch operator.
    *   `master`: Configuration for the master replica group. The master always has 1 replica.
    *   `worker`: Configuration for the worker replica group, including `replicas`, `image`, `requests`, `limits`, and `restart_policy`.
    *   `run_policy`: Controls the job's lifecycle.
    *   `increase_shared_mem`: (Deprecated, use `@task(shared_memory=...)` instead) Configures shared memory for multiprocessing.

*   **`Elastic`**: Configures PyTorch Elastic (torchrun) training. This can be used for single-node or multi-node distributed training.
    *   `nnodes`: Number of nodes, or a range (e.g., "1:2"). If `nnodes` is 1, the task runs in a standard Kubernetes pod without the Kubeflow operator.
    *   `nproc_per_node`: Number of processes per node.
    *   `start_method`: Multiprocessing start method (e.g., "spawn", "fork").
    *   `monitor_interval`: Interval to monitor worker state.
    *   `max_restarts`: Maximum worker group restarts before failing.
    *   `rdzv_configs`: Additional rendezvous configurations (e.g., `{"timeout": 900}`).
    *   `increase_shared_mem`: (Deprecated, use `@task(shared_memory=...)` instead) Configures shared memory.

To use these configurations, decorate your Python function with `@task` and pass an instance of `PyTorch` or `Elastic` to the `task_config` argument.

```python
from flytekit import task
from flytekitplugins.kfpytorch.task import PyTorch, Worker, Elastic

@task(task_config=PyTorch(
    worker=Worker(replicas=4, requests={"cpu": "2", "memory": "4Gi"}),
))
def distributed_pytorch_job(epochs: int) -> float:
    # Your PyTorch distributed training code here
    pass

@task(task_config=Elastic(
    nnodes="1:2",  # Min 1, Max 2 nodes
    nproc_per_node=2,
    start_method="spawn",
))
def elastic_pytorch_training(epochs: int) -> float:
    # Your PyTorch elastic training code here
    # This function will be launched by torch.distributed.launcher.api.elastic_launch
    pass
```

`PyTorchFunctionTask` and `PytorchElasticFunctionTask` manage the execution. `PytorchElasticFunctionTask` handles the `elastic_launch` mechanism, including serialization of the task function for `spawn` method and managing output metadata. It also sets `OMP_NUM_THREADS` to 1 by default to prevent system overload.

### TensorFlow Distributed Training

The TensorFlow plugin supports distributed training using the Kubeflow TF-Operator. It allows for flexible configurations of different replica types.

*   **`TfJob`**: Configures a TensorFlow job.
    *   `chief`: Configuration for the chief replica group.
    *   `ps`: Configuration for the parameter server (PS) replica group.
    *   `worker`: Configuration for the worker replica group.
    *   `evaluator`: Configuration for the evaluator replica group.
    *   Each replica type (`Chief`, `PS`, `Worker`, `Evaluator`) supports `image`, `requests`, `limits`, `replicas`, and `restart_policy`.
    *   `run_policy`: Controls the job's lifecycle.

To use this configuration, decorate your Python function with `@task` and pass an instance of `TfJob` to the `task_config` argument.

```python
from flytekit import task
from flytekitplugins.kftensorflow.task import TfJob, Worker, PS, Chief

@task(task_config=TfJob(
    chief=Chief(replicas=1, requests={"cpu": "1", "memory": "2Gi"}),
    worker=Worker(replicas=2, requests={"cpu": "2", "memory": "4Gi"}),
    ps=PS(replicas=1, requests={"cpu": "1", "memory": "2Gi"}),
))
def distributed_tensorflow_job(epochs: int) -> float:
    # Your TensorFlow distributed training code here
    pass
```

`TensorflowFunctionTask` handles the serialization and execution of these jobs, converting the Python configuration into the appropriate Kubeflow TFJob custom resource.

### Common Kubeflow Run Policies

Across Kubeflow-based plugins, `RunPolicy` defines common execution policies:
*   **`RunPolicy`**:
    *   `clean_pod_policy`: Defines how pods are cleaned up after the job (`NONE`, `ALL`, `RUNNING`).
    *   `ttl_seconds_after_finished`: Time-to-live for cleaning up finished jobs.
    *   `active_deadline_seconds`: Maximum duration for the job to remain active.
    *   `backoff_limit`: Number of retries before marking the job as failed.
*   **`CleanPodPolicy`**: An Enum for pod cleanup behavior.
*   **`RestartPolicy`**: An Enum for replica restart behavior (`ALWAYS`, `FAILURE`, `NEVER`).

## General-Purpose Distributed Computing

Flytekit provides plugins for general-purpose distributed computing frameworks like Dask and Ray, enabling you to orchestrate dynamic clusters for parallel processing.

### Dask Clusters

The Dask plugin allows you to launch and manage Dask clusters directly from Flyte tasks.

*   **`Dask`**: Configures a Dask cluster.
    *   `scheduler`: Configuration for the Dask scheduler pod, including `image`, `requests`, and `limits`.
    *   `workers`: Configuration for the default Dask worker group, including `number_of_workers`, `image`, `requests`, and `limits`.

To use this configuration, decorate your Python function with `@task` and pass an instance of `Dask` to the `task_config` argument.

```python
from flytekit import task
from flytekitplugins.dask.task import Dask, Scheduler, WorkerGroup

@task(task_config=Dask(
    scheduler=Scheduler(requests={"cpu": "0.5", "memory": "1Gi"}),
    workers=WorkerGroup(number_of_workers=3, requests={"cpu": "1", "memory": "2Gi"}),
))
def dask_processing_task(data_path: str) -> int:
    import dask.dataframe as dd
    # Dask client will automatically connect to the provisioned scheduler
    df = dd.read_csv(data_path)
    return df.shape[0].compute()
```

`DaskTask` is the underlying task type that handles the creation and management of the Dask cluster. The specified images for scheduler and workers must have `dask[distributed]` installed and should maintain a consistent Python environment with the task runner.

### Ray Clusters

The Ray plugin enables the orchestration of Ray clusters for distributed applications, supporting both fixed-size and autoscaling clusters.

*   **`RayJobConfig`**: Configures a Ray job.
    *   `worker_node_config`: A list of `WorkerNodeConfig` objects, each defining a worker group with `group_name`, `replicas`, `min_replicas`, `max_replicas`, `ray_start_params`, `pod_template`, `requests`, and `limits`.
    *   `head_node_config`: Optional `HeadNodeConfig` for the Ray head node, with `ray_start_params`, `pod_template`, `requests`, and `limits`.
    *   `enable_autoscaling`: Enables or disables autoscaling for the Ray cluster (default: `False`).
    *   `runtime_env`: Optional dictionary for Ray's runtime environment (deprecated in KubeRay &gt;= 1.1.0, use `runtime_env_yaml` for newer KubeRay versions).
    *   `address`: Optional Ray cluster address.
    *   `shutdown_after_job_finishes`: Specifies whether the RayCluster should be deleted after the RayJob finishes.
    *   `ttl_seconds_after_finished`: Specifies the number of seconds after which the RayCluster will be deleted after the RayJob finishes.

To use this configuration, decorate your Python function with `@task` and pass an instance of `RayJobConfig` to the `task_config` argument.

```python
from flytekit import task
from flytekitplugins.ray.task import RayJobConfig, WorkerNodeConfig, HeadNodeConfig

@task(task_config=RayJobConfig(
    head_node_config=HeadNodeConfig(requests={"cpu": "1", "memory": "2Gi"}),
    worker_node_config=[
        WorkerNodeConfig(group_name="default-workers", replicas=2, requests={"cpu": "2", "memory": "4Gi"})
    ],
    enable_autoscaling=True,
    runtime_env={"pip": ["pandas", "numpy"]},
))
def ray_processing_task(data_path: str) -> int:
    import ray
    ray.init() # Connects to the provisioned Ray cluster
    # Your Ray application code here
    @ray.remote
    def process_data(chunk):
        return len(chunk)
    
    # Example: dummy data processing
    futures = [process_data.remote([1,2,3]), process_data.remote([4,5])]
    results = ray.get(futures)
    return sum(results)
```

`RayFunctionTask` handles the lifecycle of the Ray cluster. It automatically initializes Ray within the task's `pre_execute` phase, connecting to the provisioned cluster. It also manages the `runtime_env` for dependencies.

## Spark and Databricks Integration

Flytekit offers robust integration with Apache Spark, allowing you to run PySpark jobs on Kubernetes or leverage Databricks for managed Spark execution.

### Spark on Kubernetes

The Spark plugin enables running PySpark applications natively on Kubernetes, providing fine-grained control over Spark and Hadoop configurations.

*   **`Spark`**: Configures a Spark job on Kubernetes.
    *   `spark_conf`: A dictionary of Spark configuration properties (e.g., `spark.executor.memory`).
    *   `hadoop_conf`: A dictionary of Hadoop configuration properties.
    *   `executor_path`: Path to the Python binary for PySpark execution (default: `/usr/bin/python3` for default images).
    *   `applications_path`: Path to the main application file (default: `local:///usr/local/bin/entrypoint.py` for default images).
    *   `driver_pod`: Optional `PodTemplate` for the Spark driver pod, allowing custom resource requests, limits, and other pod specifications.
    *   `executor_pod`: Optional `PodTemplate` for Spark executor pods.

To use this configuration, decorate your Python function with `@task` and pass an instance of `Spark` to the `task_config` argument.

```python
from flytekit import task
from flytekitplugins.spark.task import Spark
from flytekit.core.pod_template import PodTemplate

@task(task_config=Spark(
    spark_conf={"spark.executor.instances": "2", "spark.executor.memory": "2g"},
    driver_pod=PodTemplate(
        primary_container_name="spark-driver",
        pod_spec={"containers": [{"name": "spark-driver", "resources": {"requests": {"cpu": "1", "memory": "2Gi"}}}]}
    ),
    executor_pod=PodTemplate(
        primary_container_name="spark-executor",
        pod_spec={"containers": [{"name": "spark-executor", "resources": {"requests": {"cpu": "1", "memory": "2Gi"}}}]}
    ),
))
def pyspark_word_count(input_path: str, output_path: str) -> None:
    from pyspark.sql import SparkSession
    spark = SparkSession.builder.appName("WordCount").getOrCreate()
    
    # Your PySpark code here
    lines = spark.read.text(input_path)
    words = lines.rdd.flatMap(lambda row: row.value.split(" "))
    word_counts = words.map(lambda word: (word, 1)).reduceByKey(lambda a, b: a + b)
    word_counts.saveAsTextFile(output_path)
    
    spark.stop()
```

`PysparkFunctionTask` handles the execution, automatically creating a `SparkSession` for your task. It also supports fast serialization for local execution and propagates `PYTHONPATH` to executors for hermetic environments.

### Databricks Integration

Flytekit provides a connector to submit Spark jobs directly to Databricks, leveraging the Databricks Jobs API.

*   **`DatabricksV2`**: (Recommended) Configures a Databricks job.
    *   `databricks_conf`: A dictionary compliant with Databricks Jobs API version 2.1 request structure.
    *   `databricks_instance`: The domain name of your Databricks deployment (e.g., `<account>.cloud.databricks.com`). This can also be set via the `FLYTE_DATABRICKS_INSTANCE` environment variable.

*   **`Databricks`**: (Deprecated) Older version of the Databricks configuration. Use `DatabricksV2` instead.

To use this configuration, decorate your Python function with `@task` and pass an instance of `DatabricksV2` to the `task_config` argument.

```python
from flytekit import task
from flytekitplugins.spark.task import DatabricksV2

@task(task_config=DatabricksV2(
    databricks_instance="my-workspace.cloud.databricks.com",
    databricks_conf={
        "new_cluster": {
            "spark_version": "11.3.x-scala2.12",
            "node_type_id": "Standard_DS3_v2",
            "num_workers": 2,
        },
        "spark_python_task": {
            "python_file": "dbfs:/path/to/your_script.py",
            "parameters": ["--input", "dbfs:/input", "--output", "dbfs:/output"],
        },
    },
))
def databricks_job_example(input_data: str) -> str:
    # This Python function serves as a placeholder for the Flyte task.
    # The actual Spark job is defined by `databricks_conf` and executed on Databricks.
    print(f"Submitting Databricks job for input: {input_data}")
    return "Databricks job submitted"
```

`PysparkFunctionTask` handles the Databricks integration. When `DatabricksV2` is used, it leverages `DatabricksConnectorV2` to interact with the Databricks API for job submission, status retrieval, and cancellation. It also provides a direct link to the Databricks console for monitoring.

### Generic Spark Jobs

For non-Python Spark applications (e.g., Java, Scala, R), the `GenericSparkTask` can be used.

*   **`GenericSparkConf`**: Configures a generic Spark job.
    *   `main_class`: The main class to execute.
    *   `applications_path`: Path to the main application file (e.g., a JAR file).
    *   `spark_conf`: Spark configuration.
    *   `hadoop_conf`: Hadoop configuration.

```python
from flytekit import task
from flytekitplugins.spark.generic_task import GenericSparkTask, GenericSparkConf

@task(task_config=GenericSparkConf(
    main_class="com.example.MySparkApp",
    applications_path="s3://my-bucket/my-spark-app.jar",
    spark_conf={"spark.executor.memory": "2g"},
))
def generic_spark_app() -> None:
    # This task will trigger a spark-submit command with the specified configurations.
    pass
```

`GenericSparkTask` constructs the `spark-submit` command. In local execution, it directly runs the `spark-submit` command.

## High-Performance Computing (HPC) with Slurm

The Slurm plugin enables Flyte to interact with Slurm clusters, allowing you to submit and manage jobs on HPC environments. It supports executing Python functions, submitting existing batch scripts, or running inline shell scripts.

### Executing Python Functions on Slurm

This approach allows you to run your Flyte task's Python function directly on a Slurm cluster.

*   **`SlurmFunctionConfig`**: Configures a Slurm job for a Python function.
    *   `ssh_config`: Defines SSH client connection options, including `host` (required), `username`, and `client_keys` (paths to private keys).
    *   `sbatch_config`: A dictionary of `sbatch` command options (e.g., `{"time": "0-01:00", "nodes": "1"}`).
    *   `script`: An optional custom script template. The placeholder `{task.fn}` must be included where the Python task function should execute. If not provided, the task function will be executed directly.

To use this configuration, decorate your Python function with `@task` and pass an instance of `SlurmFunctionConfig` to the `task_config` argument.

```python
from flytekit import task
from flytekitplugins.slurm.function.task import SlurmFunctionConfig

@task(task_config=SlurmFunctionConfig(
    ssh_config={"host": "slurm-cluster.example.com", "username": "flyteuser", "client_keys": "~/.ssh/id_rsa"},
    sbatch_config={"time": "0-00:10", "nodes": "1", "ntasks-per-node": "1"},
    script="""#!/bin/bash
#SBATCH --job-name=my_flyte_job
#SBATCH --output=flyte_job_%j.out
#SBATCH --error=flyte_job_%j.err

echo "Starting Flyte task on Slurm"
{task.fn}
echo "Flyte task finished"
"""
))
def slurm_python_task(input_val: int) -> str:
    import time
    time.sleep(input_val)
    return f"Processed {input_val} on Slurm"
```

`SlurmFunctionTask` and `SlurmFunctionConnector` manage the remote execution. The connector establishes an SSH connection, uploads the script (if provided), submits the `sbatch` command, and monitors the job status.

### Submitting Existing Slurm Batch Scripts

This method allows you to submit a pre-existing batch script located on the Slurm cluster.

*   **`SlurmScriptConfig`**: Extends `SlurmConfig` with a required `batch_script_path`.
    *   `batch_script_path`: The absolute path to the batch script on the Slurm cluster.
    *   Inherits `ssh_config`, `sbatch_config`, and `batch_script_args` from `SlurmConfig`.

To use this, instantiate `SlurmTask` directly.

```python
from flytekitplugins.slurm.script.task import SlurmTask, SlurmScriptConfig

slurm_script_task = SlurmTask(
    name="my_pre_existing_slurm_script",
    task_config=SlurmScriptConfig(
        ssh_config={"host": "slurm-cluster.example.com", "username": "flyteuser", "client_keys": "~/.ssh/id_rsa"},
        batch_script_path="/home/flyteuser/my_training_script.sh",
        sbatch_config={"time": "0-01:00"},
        batch_script_args=["--data", "/data/input.csv"],
    ),
)
```

`SlurmTask` and `SlurmScriptConnector` handle the submission and monitoring of the specified batch script.

### Executing Inline Slurm Shell Scripts

This approach allows you to define the Slurm batch script content directly within your Flyte task, with support for input and output interpolation.

*   **`SlurmConfig`**: Base configuration for Slurm tasks.
    *   `ssh_config`: SSH client connection options.
    *   `sbatch_config`: `sbatch` command options.
    *   `batch_script_args`: Additional arguments passed to the batch script.

To use this, instantiate `SlurmShellTask` directly.

```python
from flytekitplugins.slurm.script.task import SlurmShellTask, SlurmConfig, OutputLocation
from flytekit.types.file import FlyteFile
from typing import Type

slurm_shell_task = SlurmShellTask(
    name="my_inline_slurm_script",
    task_config=SlurmConfig(
        ssh_config={"host": "slurm-cluster.example.com", "username": "flyteuser", "client_keys": "~/.ssh/id_rsa"},
        sbatch_config={"time": "0-00:05"},
    ),
    script="""#!/bin/bash
#SBATCH --job-name=inline_script
#SBATCH --output=output_%j.txt

echo "Input value: {inputs.my_input}"
echo "Hello from Slurm!" > {outputs.output_file}
""",
    inputs={"my_input": int},
    output_locs=[OutputLocation(var="output_file", var_type=FlyteFile, location="/tmp/slurm_output.txt")],
)
```

`SlurmShellTask` and `SlurmScriptConnector` handle the dynamic generation and interpolation of the script. Input values from the Flyte task are interpolated into the script using f-string like syntax (`{inputs.input_name}`). Output file paths can also be interpolated (`{outputs.output_name}`) and are automatically retrieved by Flyte. Only `FlyteFile` and `FlyteDirectory` types are supported for outputs.

### SSH Configuration

The Slurm plugins rely on SSH for communication with the Slurm cluster.

*   **`SSHConfig`**: Defines the essential SSH connection parameters.
    *   `host`: The hostname or IP address of the Slurm cluster's login node.
    *   `username`: The username for SSH authentication.
    *   `client_keys`: Paths to private SSH keys for authentication. Public key authentication is mandatory.

*   **`SlurmCluster`**: A simple data class used internally by the connectors to identify a unique Slurm cluster instance based on its host and username, enabling connection pooling.

The `SlurmFunctionConnector` and `SlurmScriptConnector` manage SSH connections, including connection pooling for multi-host environments, to efficiently interact with the Slurm cluster.
<!--
key: summary_distributed_training_&_computing_plugins_48e6e27c-57e5-47f2-8fe7-b2106325e503
type: summary_end

-->
<!--
code_unit: flytekitplugins.kfmpi.task.MPIFunctionTask
code_unit_type: class
help_text: ''
key: example_0830069f-d6c1-4982-bbc3-b9c0c1f60e8c
type: example

-->
<!--
code_unit: flytekitplugins.kfpytorch.task.PyTorchFunctionTask
code_unit_type: class
help_text: ''
key: example_dd091968-10cb-4773-8435-66128ad65275
type: example

-->
<!--
code_unit: flytekitplugins.kftensorflow.task.TensorflowFunctionTask
code_unit_type: class
help_text: ''
key: example_0b7e24ce-2fab-41a1-8693-c90cffa09c06
type: example

-->
<!--
code_unit: flytekitplugins.dask.task.DaskTask
code_unit_type: class
help_text: ''
key: example_f1eb9fba-0539-4815-bcb1-ea58b3bd06e5
type: example

-->
<!--
code_unit: flytekitplugins.ray.task.RayFunctionTask
code_unit_type: class
help_text: ''
key: example_896e1158-dda7-4914-beef-e974f42c37e6
type: example

-->
<!--
code_unit: flytekitplugins.spark.task.PysparkFunctionTask
code_unit_type: class
help_text: ''
key: example_921c7ec8-0074-4a4a-9094-0ef8ecb8aa93
type: example

-->
<!--
code_unit: flytekitplugins.slurm.function.task.SlurmFunctionTask
code_unit_type: class
help_text: ''
key: example_12ffce9b-9541-4e73-bb13-e6a20e5d8ec7
type: example

-->