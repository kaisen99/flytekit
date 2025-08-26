
<!--
help_text: ''
key: summary_perian_job_platform_integration_dc1d33b0-7148-498e-b5d8-d1d5c4d16bc0
modules:
- flytekitplugins.perian_job.connector
- flytekitplugins.perian_job.task
questions_to_answer: []
type: summary

-->
# Perian Job Platform Integration

The Perian Job Platform Integration enables Flyte tasks to execute workloads directly on the Perian.io platform. This integration extends Flyte's capabilities, allowing users to leverage Perian's specialized compute resources and job management features for containerized applications and Python functions.

## Core Components

The integration is built around a dedicated connector and specialized task types that abstract the complexities of interacting with the Perian API.

### Perian Connector

The `PerianConnector` acts as the primary interface between Flyte and the Perian Job Platform. It manages the lifecycle of jobs submitted to Perian, including creation, status retrieval, and cancellation.

*   **Job Creation:** The `create` method translates a Flyte `TaskTemplate` and its inputs into a `CreateJobRequest` for the Perian API. It extracts resource requirements (CPU, memory, accelerators, storage, region, provider) from the task configuration and handles Docker image details and command arguments. Crucially, it manages the secure injection of cloud storage credentials (AWS S3, GCP GCS) and Docker registry credentials into the Perian job environment, ensuring the Perian job can access necessary data.
*   **Status Retrieval:** The `get` method polls the Perian API for the status of a submitted job using its `job_id`. It maps Perian's job statuses (e.g., `QUEUED`, `RUNNING`, `DONE`, `SERVERERROR`, `USERERROR`, `CANCELLED`) to Flyte's `TaskExecution.Phase` values, providing a consistent view within the Flyte UI.
*   **Job Cancellation:** The `delete` method sends a cancellation request to the Perian API for a specified `job_id`, allowing Flyte to terminate running Perian jobs.

The `PerianMetadata` class is a simple data structure used by the connector to store the `job_id` returned by Perian, facilitating subsequent `get` and `delete` operations.

### Perian Task Types

Two specialized task types are provided to define workloads for the Perian platform:

*   **PerianTask:** This task type is designed for executing Python functions on the Perian Job Platform. It extends `PythonFunctionTask` and integrates with the `PerianConnector` for remote execution. When a `PerianTask` is executed, it attempts to use the Perian connector if a remote output prefix is configured. If not, or if local execution is intended, it falls back to standard Python function execution.
*   **PerianContainerTask:** This task type is for running arbitrary container images with specified commands on the Perian Job Platform. Unlike `PerianTask`, it does not execute a Python function directly but rather a pre-built container image. It explicitly disallows `outputs` or `output_data_dir` arguments, as output handling is managed by the Perian job itself.

Both task types use the `_TASK_TYPE = "perian_task"` identifier, which signals Flyte to use the `PerianConnector` for execution.

### Perian Configuration

The `PerianConfig` class defines the configurable parameters for Perian tasks. These parameters allow users to specify the compute resources and environment for their jobs:

*   `cores`: Number of CPU cores.
*   `memory`: Amount of RAM in GB.
*   `accelerators`: Number of accelerators (e.g., GPUs).
*   `accelerator_type`: Specific type of accelerator (e.g., 'A100').
*   `os_storage_size`: Size of the ephemeral OS storage in GB.
*   `country_code`: Geographic region for job execution (e.g., 'DE').
*   `provider`: Cloud provider to run the job on (e.g., 'AWS', 'GCP').

These configuration options are passed as `custom` parameters within the Flyte `TaskTemplate` and are interpreted by the `PerianConnector` to build the `CreateJobRequest`.

## Integration and Usage

Integrating with the Perian Job Platform involves defining tasks with specific configurations and ensuring proper secret management.

### Defining Perian Tasks

To define a task that runs on Perian, use either `PerianTask` for Python functions or `PerianContainerTask` for container images.

**Example: PerianTask (Python Function)**

```python
from flytekit import task
from flytekitplugins.perian_job.task import PerianTask, PerianConfig

@task(
    task_type="perian_task",
    task_config=PerianConfig(
        cores=2,
        memory=4, # GB
        accelerators=1,
        accelerator_type="A100",
        country_code="US",
        provider="AWS",
        os_storage_size=50 # GB
    )
)
def my_perian_function_task(input_data: str) -> str:
    # This function's code will be executed on the Perian platform
    print(f"Processing input: {input_data} on Perian")
    return f"Processed: {input_data}"

# When defining a workflow, this task will be submitted to Perian
# from flytekit import workflow
# @workflow
# def my_perian_workflow(data: str) -> str:
#     return my_perian_function_task(input_data=data)
```

**Example: PerianContainerTask (Container Image)**

```python
from flytekit import task
from flytekitplugins.perian_job.task import PerianContainerTask, PerianConfig

# Define a task that runs a specific Docker image and command on Perian
perian_container_task = PerianContainerTask(
    name="my_custom_container_job",
    task_config=PerianConfig(
        cores=1,
        memory=2, # GB
        country_code="DE",
    ),
    image="my-docker-registry/my-image:latest",
    command=["/bin/bash", "-c", "echo 'Hello from Perian container! Input: {{.inputs.message}}' && sleep 10"],
    inputs={"message": str},
)

# When defining a workflow, this task will be submitted to Perian
# from flytekit import workflow
# @workflow
# def my_container_workflow(msg: str):
#     perian_container_task(message=msg)
```

### Resource Configuration

Resource requirements are specified directly within the `task_config` argument using `PerianConfig`. This allows granular control over the compute environment for each Perian job. The `PerianConnector` translates these settings into Perian's instance type queries.

### Secrets Management

Securely providing credentials is a critical aspect of Perian integration. The `PerianConnector` relies on Flyte secrets for authentication with the Perian API and for accessing cloud storage buckets.

*   **Perian API Authentication:**
    *   `perian_organization`: Your Perian organization ID.
    *   `perian_token`: Your Perian API token.
    These secrets are used to construct the `X-PERIAN-AUTH-ORG` and `Authorization` headers for all Perian API calls.
*   **Cloud Storage Credentials:**
    *   **AWS:** `aws_access_key_id`, `aws_secret_access_key`
    *   **GCP:** `google_application_credentials` (JSON content)
    These credentials enable the Perian job to access Flyte's raw output prefix (e.g., S3 or GCS buckets) for data transfer. The connector automatically injects these as environment variables (`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`) or mounts a credentials file (`GOOGLE_APPLICATION_CREDENTIALS`) within the Perian job's container.

**Important:** If cloud storage credentials are not provided, the connector raises a `FlyteUserException`, as it cannot ensure the Perian job can access the necessary data.

### Command Templating for Container Tasks

For `PerianContainerTask`, input values can be injected into the container's command using a templating syntax: `{{.inputs.<input_name>}}`. The `_render_command_template` method within the `PerianConnector` replaces these placeholders with the actual input values before submitting the job to Perian.

## Execution Flow

When a Perian task is triggered within a Flyte workflow, the following occurs:

1.  Flyte identifies the task as a "perian_task" and dispatches it to the registered `PerianConnector`.
2.  The `PerianConnector`'s `create` method is invoked. It gathers task details, input values, and configuration from the `TaskTemplate`.
3.  It constructs a `CreateJobRequest` payload, including resource requirements, Docker image, command, environment variables, and securely injected credentials.
4.  The request is sent to the Perian API.
5.  Upon successful job creation, the `PerianConnector` returns a `PerianMetadata` object containing the Perian `job_id`.
6.  Flyte periodically calls the `PerianConnector`'s `get` method with the `job_id` to poll for the job's status on Perian.
7.  Once the Perian job completes (succeeds, fails, or is cancelled), the `PerianConnector` maps the final Perian status to a Flyte `TaskExecution.Phase`, and the Flyte task transitions accordingly.
8.  If the Flyte task is aborted, the `PerianConnector`'s `delete` method is called to cancel the corresponding Perian job.

## Limitations and Considerations

*   **Output Handling:** `PerianContainerTask` does not directly support Flyte's `outputs` or `output_data_dir` arguments. Output data from Perian jobs must be handled by the container's logic, typically by writing to a pre-configured cloud storage path accessible by Flyte.
*   **Local Execution:** To run Perian tasks locally (e.g., for testing), a remote `raw_output_data_prefix` (e.g., `s3://my-bucket/`, `gcs://my-bucket/`) must be configured. This is because the Perian platform itself is a remote execution environment, and it needs a remote location to interact with Flyte's data plane.
*   **Mandatory Secrets:** The `perian_organization`, `perian_token`, and relevant cloud storage credentials are mandatory for the `PerianConnector` to function correctly. Failure to provide these will result in `FlyteUserException` errors.
*   **Status Mapping:** While the connector maps Perian statuses to Flyte phases, detailed logs and specific error messages from Perian are typically available through the Perian platform's own UI or API. The `message` field in the Flyte `Resource` object provides basic logs from Perian.

## Best Practices

*   **Centralized Secret Management:** Always use Flyte's secret management capabilities to store Perian API tokens and cloud credentials. Avoid hardcoding sensitive information.
*   **Granular Resource Allocation:** Define `PerianConfig` parameters precisely for each task to optimize cost and performance on the Perian platform.
*   **Container Image Management:** Ensure that the Docker images used with `PerianContainerTask` are accessible from the Perian platform (e.g., public registries or private registries configured with credentials).
*   **Error Logging:** Monitor Flyte task logs for `FlyteException` or `FlyteUserException` messages originating from the `PerianConnector` to diagnose issues with Perian job submission or execution.
*   **Perian CLI for Accelerator Types:** Use the Perian CLI's `list-accelerators` command to discover the full list of supported `accelerator_type` values for `PerianConfig`.
<!--
key: summary_perian_job_platform_integration_dc1d33b0-7148-498e-b5d8-d1d5c4d16bc0
type: summary_end

-->
<!--
code_unit: flytekitplugins.perian_job.examples.perian_task_example
code_unit_type: class
help_text: ''
key: example_8f7df1d2-c01e-47cc-ab22-19648e31ec8b
type: example

-->