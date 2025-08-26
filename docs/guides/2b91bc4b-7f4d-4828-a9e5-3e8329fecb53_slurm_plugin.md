
<!--
help_text: ''
key: summary_slurm_plugin_ef450aad-e90b-498b-a2bb-5feb07fc8fc0
modules:
- flytekitplugins.slurm.function.connector
- flytekitplugins.slurm.function.task
- flytekitplugins.slurm.script.connector
- flytekitplugins.slurm.script.task
- flytekitplugins.slurm.ssh_utils
questions_to_answer: []
type: summary

-->
The Slurm Plugin enables Flyte tasks to execute on a Slurm cluster, leveraging SSH for secure communication and job management. It provides two primary modes of operation: executing Python functions directly on Slurm and running arbitrary shell scripts.

### Core Concepts

The plugin integrates with Slurm by establishing SSH connections to a specified Slurm host. All job lifecycle operations—submission, status monitoring, and termination—are performed via SSH commands (`sbatch`, `scontrol`, `scancel`). To optimize performance, the plugin maintains a pool of SSH connections, identified by the Slurm cluster's host and username.

### Connectivity Configuration

Connectivity to the Slurm cluster is managed through SSH configurations. The `SSHConfig` object defines the necessary parameters for establishing an SSH client connection.

*   **`host`**: The hostname or IP address of the Slurm cluster's login node. This is a mandatory field.
*   **`username`**: The username to use for authentication on the Slurm server. If not provided, the default SSH client behavior will apply (e.g., using the current user's name).
*   **`client_keys`**: Paths to private key files used for public key authentication. This is the recommended and most secure method for authentication. It can be a single string path, a list of paths, or a tuple of paths.

Example `ssh_config` dictionary:

```python
ssh_config = {
    "host": "slurm-cluster.example.com",
    "username": "flyteuser",
    "client_keys": "/path/to/your/private_key",
}
```

### Slurm Task Configuration

All Slurm tasks require a configuration object that specifies how the task should interact with the Slurm cluster. This includes the SSH connection details and options for the `sbatch` command.

The base configuration, `SlurmConfig`, provides:

*   **`ssh_config`**: A dictionary conforming to the `SSHConfig` structure, detailing how to connect to the Slurm cluster.
*   **`sbatch_config`**: An optional dictionary of key-value pairs that are translated into `sbatch` command-line options (e.g., `{"time": "01:00:00", "nodes": "1"}` becomes `--time=01:00:00 --nodes=1`). Refer to the Slurm `sbatch` documentation for available options.
*   **`batch_script_args`**: An optional list of strings representing additional arguments passed directly to the batch script executed on the Slurm cluster.

### Python Function Tasks

The Slurm Plugin allows you to execute standard Python functions directly on a Slurm cluster using the `SlurmFunctionTask`. This is ideal for existing Python code that needs to leverage Slurm's resource management.

To define a Python function task for Slurm, use the `SlurmFunctionConfig` object. This configuration extends the base Slurm settings with an optional `script` parameter.

*   **`script`**: A custom shell script template. Within this script, the placeholder `{task.fn}` is crucial. It will be replaced at runtime with the command to execute your Python task function. This allows you to wrap your Python function execution within a custom Slurm batch script, enabling advanced setup or environment configurations before your Python code runs. If `script` is not provided, the task function will be executed directly without a custom wrapper script.

**Example `SlurmFunctionTask` Definition:**

```python
from flytekit import task
from flytekitplugins.slurm import SlurmFunctionConfig, SlurmFunctionTask

slurm_config = SlurmFunctionConfig(
    ssh_config={
        "host": "slurm-cluster.example.com",
        "username": "flyteuser",
        "client_keys": "/path/to/your/private_key",
    },
    sbatch_config={"time": "00:10:00", "mem": "4G"},
    script="""
#!/bin/bash
#SBATCH --job-name=my_flyte_job
#SBATCH --output=slurm-%j.out
#SBATCH --error=slurm-%j.err

echo "Starting Flyte task..."
{task.fn}
echo "Flyte task finished."
""",
)

@task(task_config=slurm_config)
def my_slurm_python_task(a: int, b: int) -> int:
    """A Python function that will run on Slurm."""
    print(f"Running on Slurm: {a} + {b}")
    return a + b
```

When `my_slurm_python_task` is executed, the `SlurmFunctionConnector` is responsible for:
1.  Generating the final batch script by replacing `{task.fn}` with the actual command to run the Python function.
2.  Uploading this script to the Slurm cluster via SFTP.
3.  Submitting the job using `sbatch`.
4.  Monitoring the job's status using `scontrol`.
5.  Retrieving standard output from the Slurm job.
6.  Canceling the job using `scancel` if needed.

For local execution, the `SlurmFunctionTask` bypasses the Slurm submission and executes the Python function directly, mimicking the behavior of the Flyte propeller in a local connector test environment.

### Shell Script Tasks

The Slurm Plugin supports executing arbitrary shell scripts on a Slurm cluster, offering two distinct approaches:

#### 1. Executing a Pre-existing Script (`SlurmTask`)

The `SlurmTask` is designed for scenarios where your batch script already exists on the Slurm cluster's filesystem.

To use `SlurmTask`, you provide a `SlurmScriptConfig` object, which extends `SlurmConfig` by requiring a `batch_script_path`.

*   **`batch_script_path`**: The absolute path to the script on the Slurm cluster that you want to execute.

**Example `SlurmTask` Definition:**

```python
from flytekit import task
from flytekitplugins.slurm import SlurmScriptConfig, SlurmTask

slurm_script_config = SlurmScriptConfig(
    ssh_config={
        "host": "slurm-cluster.example.com",
        "username": "flyteuser",
        "client_keys": "/path/to/your/private_key",
    },
    sbatch_config={"time": "00:05:00"},
    batch_script_path="/home/flyteuser/my_existing_script.sh",
    batch_script_args=["arg1", "arg2"],
)

@task(task_config=slurm_script_config)
def run_existing_slurm_script():
    """Runs a pre-existing script on Slurm."""
    pass # No Python logic needed, the script handles execution
```

The `SlurmScriptConnector` will verify the existence of the script at `batch_script_path` on the Slurm cluster before submitting the job.

#### 2. Executing an Inline Script (`SlurmShellTask`)

The `SlurmShellTask` allows you to define the batch script content directly within your Flyte task definition. This is highly flexible, as it supports interpolating Flyte task inputs into the script and specifying output locations.

Key features of `SlurmShellTask`:

*   **`script`**: The full content of the batch script as a string.
*   **`inputs`**: A dictionary mapping input variable names to their Python types. These inputs can be interpolated into the `script` using f-string like syntax (e.g., `{inputs.my_input_var}`).
*   **`output_locs`**: A list of `OutputLocation` objects. Each `OutputLocation` specifies an output variable name (`var`), its type (`var_type`), and the file path on the Slurm cluster where the output will be written (`location`). Currently, only `FlyteFile` and `FlyteDirectory` types are supported for outputs. The `location` string can also be interpolated with input values.

**Example `SlurmShellTask` Definition:**

```python
from flytekit import task, FlyteFile
from flytekitplugins.slurm import SlurmConfig, SlurmShellTask, OutputLocation

slurm_shell_config = SlurmConfig(
    ssh_config={
        "host": "slurm-cluster.example.com",
        "username": "flyteuser",
        "client_keys": "/path/to/your/private_key",
    },
    sbatch_config={"time": "00:05:00", "cpus-per-task": "1"},
)

@task
def my_slurm_shell_task(input_data: str) -> FlyteFile:
    """
    Executes an inline shell script on Slurm, interpolating inputs and handling outputs.
    """
    return SlurmShellTask(
        name="my_shell_job",
        task_config=slurm_shell_config,
        script=f"""
#!/bin/bash
#SBATCH --job-name=shell_task
#SBATCH --output=slurm-%j.out
#SBATCH --error=slurm-%j.err

echo "Input data: {{inputs.input_data}}"
echo "Processing data..."
echo "Output content for {{inputs.input_data}}" > {{outputs.output_file}}
""",
        inputs={"input_data": str},
        output_locs=[
            OutputLocation(var="output_file", var_type=FlyteFile, location="/tmp/output_{{inputs.input_data}}.txt")
        ],
    )(input_data=input_data)

```

The `SlurmScriptConnector` handles the interpolation of inputs and output paths into the `script` content. It then uploads this dynamically generated script to the Slurm cluster, submits it, and, upon completion, retrieves the specified output files from the Slurm cluster to make them available as Flyte task outputs.

### Job Management and Metadata

Both `SlurmFunctionConnector` and `SlurmScriptConnector` manage the lifecycle of Slurm jobs. They use `SlurmJobMetadata` to track essential information about a running job:

*   **`job_id`**: The unique identifier assigned by Slurm to the submitted job.
*   **`ssh_config`**: The SSH configuration used to connect to the cluster where the job is running.
*   **`outputs`** (for `SlurmScriptConnector` only): A mapping from output variable names to their final locations on the Slurm cluster, used for retrieving results.

These connectors periodically query the Slurm cluster (using `scontrol`) to determine the job's current state and convert it into a Flyte phase (e.g., `RUNNING`, `SUCCEEDED`, `FAILED`). They also retrieve standard output logs from the Slurm job for debugging and monitoring.

### Best Practices and Considerations

*   **SSH Key Management**: Ensure that the `client_keys` specified in `ssh_config` are accessible from where your Flyte tasks are executed (e.g., the Flyte Propeller environment or your local development machine during local execution). For production deployments, consider using SSH agents or secrets management systems to securely provide keys.
*   **Slurm `sbatch` Options**: Leverage `sbatch_config` to fine-tune resource requests (CPU, memory, time, GPU, etc.) and other Slurm-specific parameters for optimal job scheduling and performance.
*   **Script Idempotency**: For `SlurmShellTask` and `SlurmFunctionTask` with custom scripts, ensure your scripts are robust and handle re-execution gracefully if Flyte retries the task.
*   **Output Handling**: When using `SlurmShellTask`, carefully define `output_locs` to ensure that the script writes its outputs to the specified paths and that these paths are correctly interpolated. Only `FlyteFile` and `FlyteDirectory` are currently supported for direct output retrieval.
*   **Error Handling**: The plugin checks command execution results (`check=True`). Slurm job failures will be reflected in the Flyte task's status. Review Slurm job logs (standard output and error files) for detailed debugging.
*   **Connection Pooling**: The plugin automatically manages an SSH connection pool (`slurm_cluster_to_ssh_conn`) to reuse connections to the same Slurm cluster, improving efficiency for multiple tasks targeting the same cluster.
<!--
key: summary_slurm_plugin_ef450aad-e90b-498b-a2bb-5feb07fc8fc0
type: summary_end

-->
<!--
code_unit: flytekitplugins.slurm.function.task
code_unit_type: class
help_text: ''
key: example_6c209492-12d5-4bba-850b-1f0a90abdb5f
type: example

-->
<!--
code_unit: flytekitplugins.slurm.script.task
code_unit_type: class
help_text: ''
key: example_1e681030-7b85-4c3f-9997-5c02ad5c3d68
type: example

-->