
<!--
help_text: ''
key: summary_perian_job_plugin_9154cd71-1563-4aef-b58a-3167a6994332
modules:
- flytekitplugins.perian_job.connector
- flytekitplugins.perian_job.task
questions_to_answer: []
type: summary

-->
The Perian Job Plugin integrates Flyte with the PERIAN Job Platform, enabling Flyte tasks to execute as jobs on PERIAN's infrastructure. This allows offloading compute-intensive or specialized workloads to a dedicated platform while leveraging Flyte's orchestration capabilities. The plugin manages the complete lifecycle of PERIAN jobs, including creation, status monitoring, and cancellation.

## Connecting to PERIAN

The `PerianConnector` is the core component responsible for all interactions with the PERIAN API. It extends `AsyncConnectorBase`, providing asynchronous operations for managing remote PERIAN jobs.

The connector performs the following key operations:
*   **Job Creation:** The `create` method submits a new job to PERIAN. It translates Flyte `TaskTemplate` details, including task inputs and custom configurations from `PerianConfig`, into a `CreateJobRequest` for the PERIAN API. This includes specifying resource requirements (CPU, memory, accelerators), Docker image details, command arguments, environment variables, and storage configurations.
*   **Status Retrieval:** The `get` method queries the status of an existing PERIAN job using its unique `job_id`. It maps PERIAN's job statuses (e.g., `QUEUED`, `RUNNING`, `DONE`, `SERVERERROR`) to Flyte's standard `TaskExecution.Phase` (e.g., `QUEUED`, `RUNNING`, `SUCCEEDED`, `FAILED`).
*   **Job Cancellation:** The `delete` method sends a request to cancel a running PERIAN job.

The `PerianConnector` relies on the `PERIAN_API_URL` environment variable to connect to the PERIAN platform.

## Perian Job Metadata

The `PerianMetadata` class serves as a simple data structure to store the `job_id` returned by the PERIAN platform upon job creation. This `job_id` is crucial for the `PerianConnector` to track, query the status of, and manage the lifecycle of the corresponding remote PERIAN job.

## Defining Perian Tasks

The plugin provides two primary task types for executing workloads on PERIAN:

### PerianTask

The `PerianTask` is designed for executing standard Python functions on the PERIAN Job Platform. It extends `PythonFunctionTask` and integrates with the `AsyncConnectorExecutorMixin` to leverage the `PerianConnector` for remote execution.

When a `PerianTask` is executed, the plugin automatically packages the Python function and its dependencies, then submits it to PERIAN.

**Usage:**
Define a Python function and decorate it with `@task`, providing a `PerianConfig` instance to specify PERIAN-specific settings.

```python
from flytekit import task
from flytekitplugins.perian_job.task import PerianConfig, PerianTask

@task(task_config=PerianConfig(cores=2, memory=4, country_code="US"), task_type="perian_task")
def my_perian_function(input_data: str) -> str:
    # Your Python logic here
    print(f"Processing {input_data} on PERIAN.")
    return f"Processed: {input_data}"

# Alternatively, explicitly instantiate PerianTask
# perian_task_instance = PerianTask(
#     task_config=PerianConfig(cores=2, memory=4),
#     task_function=my_perian_function
# )
```

### PerianContainerTask

The `PerianContainerTask` is used for executing pre-built Docker images directly on the PERIAN Job Platform. This task type is suitable when you have an existing containerized application or require fine-grained control over the execution environment.

When defining a `PerianContainerTask`, you must specify the Docker `image` and the `command` to be executed within the container.

**Limitations:**
`PerianContainerTask` does not directly support `outputs` or `output_data_dir` arguments during initialization. Output handling must be managed within the container itself, typically by writing results to a shared remote storage location that Flyte can subsequently access.

**Usage:**
Instantiate `PerianContainerTask` directly, providing the task name, `PerianConfig`, image, and command.

```python
from flytekit import task, Interface
from flytekitplugins.perian_job.task import PerianConfig, PerianContainerTask
from typing import List, OrderedDict, Type

perian_container_task = PerianContainerTask(
    name="my_perian_container_job",
    task_config=PerianConfig(cores=4, memory=8, accelerator_type="A100", accelerators=1),
    image="my_docker_repo/my_image:latest",
    command=["python", "my_script.py", "--input", "{{.inputs.data}}"],
    inputs=OrderedDict([("data", str)])
)
```

## Configuring Perian Tasks

The `PerianConfig` class allows you to specify resource requirements and execution parameters for tasks running on the PERIAN platform. These configurations are translated into the `CreateJobRequest` sent to the PERIAN API.

Available configuration parameters include:
*   `cores` (Optional[int]): Number of CPU cores requested.
*   `memory` (Optional[int]): Amount of RAM in GB requested.
*   `accelerators` (Optional[int]): Number of accelerators (e.g., GPUs) requested.
*   `accelerator_type` (Optional[str]): Specific type of accelerator (e.g., 'A100'). Use the PERIAN CLI `list-accelerators` command for a full list of supported types.
*   `os_storage_size` (Optional[int]): Ephemeral OS storage size in GB for the job.
*   `country_code` (Optional[str]): Two-letter country code (e.g., 'DE', 'US') to specify the geographic region for job execution.
*   `provider` (Optional[str]): Cloud provider (e.g., 'AWS', 'GCP') on which to run the job.

## Secrets Management

Proper secret configuration is essential for the Perian Job Plugin to function correctly.

*   **PERIAN API Authentication:** The `PerianConnector` requires authentication credentials to interact with the PERIAN API. You must configure the following as Flyte secrets:
    *   `perian_organization`: Your PERIAN organization ID.
    *   `perian_token`: Your PERIAN API token.
    If these secrets are not provided, a `FlyteUserException` will be raised.

*   **Storage Access:** To enable PERIAN jobs to access data in Flyte's configured storage bucket (e.g., S3, GCS), the corresponding cloud credentials must be provided as Flyte secrets. The plugin automatically injects these into the PERIAN job's environment:
    *   **AWS:** `aws_access_key_id` and `aws_secret_access_key`.
    *   **GCP:** `google_application_credentials` (the content of your service account JSON key file).
    Failure to provide appropriate storage credentials will result in a `FlyteUserException`.

*   **Docker Registry Credentials:** For pulling images from private Docker registries, you can provide the following secrets:
    *   `docker_registry_url`
    *   `docker_registry_username`
    *   `docker_registry_password`

## Command Templating and Environment Variables

For `PerianContainerTask` and `PerianTask` (when a custom command is specified), the plugin supports templating input values directly into the command arguments. Input values can be referenced using the `{{.inputs.input_name}}` syntax.

```python
# Example of command templating
command=["bash", "-c", "echo 'Hello {{.inputs.name}}'"]
```

Environment variables can be passed to the PERIAN job through the `environment` attribute of the `PerianConfig` or directly on the `PerianTask` or `PerianContainerTask` instance. These variables will be set in the container's environment.

## Local Execution Considerations

When executing Flyte workflows locally that include `PerianTask` or `PerianContainerTask`, it is mandatory to set the `--raw-output-data-prefix` argument to a remote path (e.g., `s3://your-bucket/`, `gcs://your-bucket/`). This is because the PERIAN platform requires a remote location to store job outputs and intermediate data. If a remote prefix is not provided during local execution, a `ValueError` will be raised.

## Best Practices

*   **Secure Secrets:** Always manage PERIAN API tokens and cloud credentials as Flyte secrets. Avoid hardcoding them in your task definitions.
*   **Remote Output Prefix:** Ensure your Flyte environment is configured with a remote `raw_output_data_prefix` when using Perian tasks, especially during local testing, to prevent execution failures.
*   **Resource Allocation:** Carefully specify `cores`, `memory`, and `accelerators` in `PerianConfig` to match your workload's requirements and optimize cost and performance on the PERIAN platform.
*   **Error Handling:** Be prepared to handle `FlyteException` and `FlyteUserException` which indicate issues with PERIAN job creation, status retrieval, or misconfigured secrets.
<!--
key: summary_perian_job_plugin_9154cd71-1563-4aef-b58a-3167a6994332
type: summary_end

-->
<!--
code_unit: flytekitplugins.perian_job.task
code_unit_type: class
help_text: ''
key: example_6b49eea7-cfe6-42f1-978e-58d02ab2cfb8
type: example

-->