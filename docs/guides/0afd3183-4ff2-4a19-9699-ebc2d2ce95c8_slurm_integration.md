
<!--
help_text: ''
key: summary_slurm_integration_99f1d799-c109-4f35-a745-c159c8481889
modules:
- flytekitplugins.slurm.function.connector
- flytekitplugins.slurm.function.task
- flytekitplugins.slurm.script.connector
- flytekitplugins.slurm.script.task
- flytekitplugins.slurm.ssh_utils
questions_to_answer: []
type: summary

-->
## Slurm Integration

Slurm integration enables Flyte tasks to execute directly on a Slurm Workload Manager cluster, leveraging its capabilities for resource management and job scheduling. This integration provides two primary patterns: executing Python functions and running arbitrary shell scripts. All interactions with the Slurm cluster are facilitated via SSH.

### Core Concepts

At the heart of Slurm integration are several key components that manage connectivity, job submission, and lifecycle.

*   **SSH Connectivity**: All Slurm tasks require SSH configuration to establish a secure connection to the remote Slurm cluster. This configuration is encapsulated by `SSHConfig`, which specifies the `host`, an optional `username`, and `client_keys` for authentication. The system maintains a connection pool, identified by `SlurmCluster` instances (a combination of host and username), to efficiently reuse SSH connections across multiple tasks targeting the same cluster.
*   **Slurm Job Management**: Tasks interact with the Slurm cluster using standard Slurm commands:
    *   `sbatch`: Submits a job script to the Slurm scheduler.
    *   `scontrol`: Retrieves detailed information about a running or completed job, including its state and standard output path.
    *   `scancel`: Terminates a running Slurm job.
*   **Job Metadata**: `SlurmJobMetadata` tracks essential information for a submitted Slurm job, including its `job_id` and the `ssh_config` used to connect to the cluster. This metadata is crucial for monitoring job status and managing its lifecycle.

### Python Function Tasks

The Python function task pattern allows you to execute a Python function defined within a Flyte task directly on a Slurm cluster. This is ideal for compute-intensive Python workloads that benefit from Slurm's resource allocation.

#### Configuration

To define a Python function task for Slurm, use `SlurmFunctionConfig`. This configuration object specifies:

*   `ssh_config`: A dictionary containing SSH connection options. The `host` key is mandatory.
*   `sbatch_config`: An optional dictionary mapping `sbatch` command options (e.g., `{"time": "01:00:00", "nodes": "1"}`) to their values. These options are passed directly to the `sbatch` command.
*   `script`: An optional custom shell script template. If provided, the execution of the Python task function is embedded within this script using the `{task.fn}` placeholder. If `script` is not provided, the task function is executed directly within a generated script.

The `SlurmFunctionTask` wraps your Python function and serializes this configuration into the task's custom attributes.

#### Execution Flow

When a `SlurmFunctionTask` executes, the `SlurmFunctionConnector` handles the remote execution:

1.  A unique temporary script file is generated on the Slurm host.
2.  If a custom `script` is provided in `SlurmFunctionConfig`, the task's entrypoint (derived from `task_template.container.args`) is interpolated into the `{task.fn}` placeholder within the custom script. Otherwise, a default script is generated to execute the task's entrypoint.
3.  The generated script is uploaded to the Slurm host via SFTP.
4.  The `sbatch` command, constructed with `sbatch_config` and the path to the uploaded script, is executed remotely via SSH.
5.  The Slurm `job_id` is extracted from the `sbatch` command's output.
6.  For status updates, the `scontrol show job <job_id>` command is executed remotely. The job state is then converted to a Flyte phase.
7.  The standard output of the Slurm job is retrieved from the remote host via SFTP.
8.  To terminate a job, the `scancel <job_id>` command is executed remotely.

**Use Case**: Running a distributed machine learning training job or a large-scale simulation written in Python that requires specific Slurm resources like GPUs or high-memory nodes.

### Shell Script Tasks

The shell script task pattern allows for executing arbitrary shell scripts on a Slurm cluster. This provides flexibility for integrating with existing shell-based workflows or running non-Python binaries. There are two variants: `SlurmShellTask` for inline scripts and `SlurmTask` for pre-existing scripts.

#### Common Configuration (`SlurmConfig`)

Both shell script task types utilize `SlurmConfig` for common settings:

*   `ssh_config`: Mandatory SSH connection options.
*   `sbatch_config`: Optional dictionary for `sbatch` command options.
*   `batch_script_args`: Optional list of additional arguments to pass to the batch script on the Slurm cluster.

#### `SlurmShellTask` (Inline Script)

The `SlurmShellTask` allows you to define the shell script content directly within your Flyte task.

*   **Script Content**: The `script` is provided as a string during task initialization.
*   **Input/Output Interpolation**: The `SlurmScriptConnector` supports interpolating inputs and output file paths directly into the script.
    *   Inputs are referenced using `{inputs.variable_name}`.
    *   Output file paths are referenced using `{outputs.variable_name}`.
    *   The system interpolates these placeholders with actual values and generated output locations before uploading the script.
*   **Output Handling**: Only `FlyteFile` and `FlyteDirectory` types are supported for task outputs. The `_validate_output_locs` method ensures that all declared outputs are of these types and have a specified `location`. The script is expected to write its results to these interpolated output locations.

#### `SlurmTask` (Pre-existing Script)

The `SlurmTask` is designed for scenarios where the batch script already exists on the Slurm cluster.

*   **Configuration**: Uses `SlurmScriptConfig`, which extends `SlurmConfig` and *requires* the `batch_script_path` attribute. This path specifies the location of the script on the remote Slurm cluster.
*   **Script Content**: The script content is not part of the Flyte task definition. The system assumes the script is already present at `batch_script_path`.
*   **Input/Output**: This task type does not support inline input/output interpolation within the script itself. Inputs can be passed as `batch_script_args`.

#### Execution Flow (`SlurmScriptConnector`)

The `SlurmScriptConnector` manages the execution for both `SlurmShellTask` and `SlurmTask`:

1.  For `SlurmShellTask`, the `_interpolate_script` method processes the provided `script`, replacing input and output placeholders with concrete values and generated paths. The interpolated script is then uploaded to a unique temporary file on the Slurm host via SFTP.
2.  For `SlurmTask`, the connector verifies the existence of the script at `batch_script_path` on the remote host.
3.  The `sbatch` command is constructed using `sbatch_config`, the script path (either the temporary uploaded path or `batch_script_path`), and `batch_script_args`.
4.  The `sbatch` command is executed remotely via SSH, and the Slurm `job_id` is retrieved.
5.  Status updates, standard output retrieval, and job cancellation follow the same pattern as the Python function tasks, using `scontrol` and `scancel`.

**Use Cases**:
*   `SlurmShellTask`: Running a custom data processing pipeline written in Bash, or executing a compiled binary that requires dynamic input parameters and produces specific output files.
*   `SlurmTask`: Orchestrating a complex, pre-deployed scientific simulation or a legacy shell-based application on the Slurm cluster without embedding its code in Flyte.

### Limitations and Considerations

*   **SSH Authentication**: Robust SSH key management is critical for secure and reliable connections. Ensure the `client_keys` specified in `ssh_config` are correctly configured and accessible.
*   **Output Types**: For `SlurmShellTask`, outputs are strictly limited to `FlyteFile` and `FlyteDirectory` types. Ensure your shell scripts write results to the interpolated paths provided by the system.
*   **Script Interpolation**: The interpolation mechanism for `SlurmShellTask` uses a custom f-string-like syntax. Adhere to the `{inputs.var}` and `{outputs.var}` patterns.
*   **Error Handling**: Remote commands executed via SSH are checked for success (`check=True`). Failures in `sbatch`, `scontrol`, or `scancel` commands will result in task failures.
*   **Connection Pooling**: The `slurm_cluster_to_ssh_conn` pool optimizes performance by reusing SSH connections to the same Slurm host and username, reducing connection overhead.
*   **Resource Management**: Carefully configure `sbatch_config` to request appropriate resources (e.g., CPU, memory, time, GPUs) from the Slurm scheduler to prevent job failures or resource contention.
<!--
key: summary_slurm_integration_99f1d799-c109-4f35-a745-c159c8481889
type: summary_end

-->
<!--
code_unit: flytekitplugins.slurm.examples.slurm_function_task
code_unit_type: class
help_text: ''
key: example_e142034a-3829-4852-af76-fa6ba9c94877
type: example

-->
<!--
code_unit: flytekitplugins.slurm.examples.slurm_shell_task
code_unit_type: class
help_text: ''
key: example_d1731560-fc43-4c47-a34f-038e3a29abab
type: example

-->