
<!--
help_text: ''
key: summary_openai_batch_plugin_5d8fe447-beb4-4ef7-925b-10c29b70eb2b
modules:
- flytekitplugins.openai.batch.connector
- flytekitplugins.openai.batch.task
questions_to_answer: []
type: summary

-->
The OpenAI Batch Plugin integrates OpenAI's asynchronous batch processing capabilities directly into Flyte workflows. This plugin enables developers to manage large-scale AI inference jobs by facilitating the upload of input data, the execution of batch requests, and the retrieval of results, all within the Flyte ecosystem. It is designed for high-throughput, long-running AI tasks that do not require immediate, synchronous responses.

### Core Concepts

The plugin leverages OpenAI's Batch API, which is inherently asynchronous. Flyte handles this by using an asynchronous connector pattern, allowing workflows to initiate a batch job and then poll for its completion without blocking the Flyte engine.

*   **Batch Processing**: OpenAI's Batch API allows submitting a file of API requests (e.g., chat completions) for asynchronous processing. This is ideal for large datasets where immediate responses are not critical.
*   **Asynchronous Execution**: The plugin's `BatchEndpointConnector` manages the lifecycle of an OpenAI batch job. It initiates the job, periodically checks its status, and reports its progress back to Flyte. This allows Flyte workflows to efficiently manage the waiting period for batch job completion.
*   **File Handling**: Input and output for OpenAI batch jobs are typically JSONL files. The plugin provides dedicated tasks to manage the upload of input JSONL files to OpenAI and the download of result JSONL files from OpenAI.
*   **State Management**: The `State` enum within the `flytekitplugins.openai.batch.connector` package maps OpenAI batch statuses (e.g., `in_progress`, `completed`, `failed`) to Flyte's execution phases, ensuring consistent status reporting within Flyte workflows.

### Plugin Components and Capabilities

The plugin provides a set of specialized tasks and a connector to interact with the OpenAI Batch API.

#### Uploading Input Files

The `UploadJSONLFileTask` facilitates uploading local JSONL files to OpenAI, which are then used as input for batch processing.

*   **Purpose**: Prepares input data for an OpenAI batch job by uploading a local JSONL file to OpenAI's file storage.
*   **Inputs**: Expects a `JSONLFile` object representing the local path to the input data.
*   **Outputs**: Returns a string, which is the unique file ID assigned by OpenAI to the uploaded file. This ID is crucial for initiating a batch job.
*   **Configuration**: Requires an `OpenAIFileConfig` object, which specifies the OpenAI organization and a `Secret` object for API key retrieval. The `UploadJSONLFileExecutor` uses this configuration to authenticate with OpenAI.
*   **Usage**:
    ```python
    from flytekitplugins.openai.batch.task import UploadJSONLFileTask, OpenAIFileConfig
    from flytekit import Secret, task, workflow, JSONLFile

    # Define your OpenAI API key secret
    # Ensure this secret is configured in your Flyte environment
    OPENAI_SECRET = Secret(group="openai-api-key", key="api_key", mount_requirement=Secret.MountType.ENV_VAR)

    upload_config = OpenAIFileConfig(
        secret=OPENAI_SECRET,
        openai_organization="your-openai-organization-id" # Optional
    )

    upload_jsonl_task = UploadJSONLFileTask(
        name="upload_batch_input_file",
        task_config=upload_config,
    )

    @task
    def create_input_file() -> JSONLFile:
        # Logic to create your JSONL input file
        # For example:
        file_path = "input.jsonl"
        with open(file_path, "w") as f:
            f.write('{"custom_id": "request-1", "method": "POST", "url": "/v1/chat/completions", "body": {"model": "gpt-3.5-turbo", "messages": [{"role": "user", "content": "Hello world!"}]}}\n')
            f.write('{"custom_id": "request-2", "method": "POST", "url": "/v1/chat/completions", "body": {"model": "gpt-3.5-turbo", "messages": [{"role": "user", "content": "How are you?"}]}}\n')
        return JSONLFile(file_path)

    @workflow
    def upload_workflow():
        input_file = create_input_file()
        file_id = upload_jsonl_task(jsonl_in=input_file)
        # file_id can now be used by BatchEndpointTask
    ```

#### Executing OpenAI Batch Jobs

The `BatchEndpointTask` initiates and monitors an OpenAI batch job. It acts as the primary interface for running the batch process itself.

*   **Purpose**: Submits a batch request to OpenAI using a previously uploaded file ID and monitors its execution until completion or failure.
*   **Inputs**: Expects `input_file_id` (str), which is the OpenAI file ID obtained from `UploadJSONLFileTask`.
*   **Outputs**: Returns a dictionary (`Dict`) containing the full result of the OpenAI batch retrieval, including `output_file_id`, `error_file_id`, and the final status.
*   **Configuration**:
    *   `name`: A unique name for the task.
    *   `config`: A dictionary of configuration parameters for the OpenAI batch API call (e.g., `completion_window`, `endpoint`, `model`). The `completion_window` defaults to "24h" and `endpoint` to "/v1/chat/completions" if not specified.
    *   `openai_organization`: Optional string specifying the OpenAI organization ID.
*   **Underlying Mechanism**: The `BatchEndpointConnector` handles the asynchronous communication. It calls `async_client.batches.create` to start the job and `async_client.batches.retrieve` to poll for status updates. If the job fails, it extracts error messages from the OpenAI response.
*   **Usage**:
    ```python
    from flytekitplugins.openai.batch.task import BatchEndpointTask
    from flytekit import workflow

    batch_task = BatchEndpointTask(
        name="run_openai_batch_job",
        config={
            "model": "gpt-3.5-turbo",
            "completion_window": "24h", # Optional, default is 24h
            "endpoint": "/v1/chat/completions", # Optional, default is /v1/chat/completions
        },
        openai_organization="your-openai-organization-id", # Optional
    )

    @workflow
    def batch_execution_workflow(input_file_id: str) -> dict:
        batch_result = batch_task(input_file_id=input_file_id)
        return batch_result
    ```

#### Downloading Batch Results

The `DownloadJSONFilesTask` retrieves the output and error files generated by a completed OpenAI batch job.

*   **Purpose**: Downloads the result files (output and error JSONL files) from OpenAI's storage to the Flyte task's working directory.
*   **Inputs**: Expects `batch_endpoint_result` (Dict), which is the output from the `BatchEndpointTask`. This dictionary contains `output_file_id` and `error_file_id`.
*   **Outputs**: Returns a `BatchResult` object. This object contains `output_file` and `error_file` attributes, which are `JSONLFile` instances pointing to the locally downloaded files.
*   **Configuration**: Similar to `UploadJSONLFileTask`, it requires an `OpenAIFileConfig` for authentication.
*   **Underlying Mechanism**: The `DownloadJSONFilesExecutor` uses `openai.OpenAI().files.with_streaming_response.content` to stream the files to the local filesystem.
*   **Usage**:
    ```python
    from flytekitplugins.openai.batch.task import DownloadJSONFilesTask, OpenAIFileConfig, BatchResult
    from flytekit import workflow, Secret

    # Define your OpenAI API key secret
    OPENAI_SECRET = Secret(group="openai-api-key", key="api_key", mount_requirement=Secret.MountType.ENV_VAR)

    download_config = OpenAIFileConfig(
        secret=OPENAI_SECRET,
        openai_organization="your-openai-organization-id" # Optional
    )

    download_files_task = DownloadJSONFilesTask(
        name="download_batch_results",
        task_config=download_config,
    )

    @workflow
    def download_workflow(batch_endpoint_result: dict) -> BatchResult:
        downloaded_results = download_files_task(batch_endpoint_result=batch_endpoint_result)
        return downloaded_results
    ```

### End-to-End Workflow Example

A typical usage pattern involves chaining these tasks together in a Flyte workflow:

1.  Create a local JSONL input file.
2.  Upload the input file using `UploadJSONLFileTask` to get an OpenAI file ID.
3.  Execute the batch job using `BatchEndpointTask` with the file ID.
4.  Download the results using `DownloadJSONFilesTask` with the batch job's output.

```python
from flytekit import workflow, task, JSONLFile, Secret
from flytekitplugins.openai.batch.task import (
    UploadJSONLFileTask,
    BatchEndpointTask,
    DownloadJSONFilesTask,
    OpenAIFileConfig,
    BatchResult,
)

# Define your OpenAI API key secret
# Ensure this secret is configured in your Flyte environment
OPENAI_SECRET = Secret(group="openai-api-key", key="api_key", mount_requirement=Secret.MountType.ENV_VAR)
OPENAI_ORG_ID = "your-openai-organization-id" # Replace with your actual organization ID

@task
def create_sample_input_file() -> JSONLFile:
    """Creates a dummy JSONL file for batch processing."""
    file_path = "sample_input.jsonl"
    with open(file_path, "w") as f:
        f.write('{"custom_id": "request-1", "method": "POST", "url": "/v1/chat/completions", "body": {"model": "gpt-3.5-turbo", "messages": [{"role": "user", "content": "What is the capital of France?"}]}}\n')
        f.write('{"custom_id": "request-2", "method": "POST", "url": "/v1/chat/completions", "body": {"model": "gpt-3.5-turbo", "messages": [{"role": "user", "content": "Who wrote Romeo and Juliet?"}]}}\n')
    return JSONLFile(file_path)

# Configure the file tasks with secrets and organization ID
file_config = OpenAIFileConfig(
    secret=OPENAI_SECRET,
    openai_organization=OPENAI_ORG_ID,
)

upload_task = UploadJSONLFileTask(
    name="upload_openai_batch_input",
    task_config=file_config,
)

batch_execution_task = BatchEndpointTask(
    name="execute_openai_batch_job",
    config={
        "model": "gpt-3.5-turbo",
        "completion_window": "24h",
        "endpoint": "/v1/chat/completions",
    },
    openai_organization=OPENAI_ORG_ID,
)

download_task = DownloadJSONFilesTask(
    name="download_openai_batch_results",
    task_config=file_config,
)

@workflow
def openai_batch_workflow() -> BatchResult:
    """
    An end-to-end workflow demonstrating OpenAI batch processing.
    """
    # 1. Create the local input file
    input_file = create_sample_input_file()

    # 2. Upload the input file to OpenAI
    openai_file_id = upload_task(jsonl_in=input_file)

    # 3. Execute the OpenAI batch job
    batch_job_result = batch_execution_task(input_file_id=openai_file_id)

    # 4. Download the output and error files
    downloaded_files = download_task(batch_endpoint_result=batch_job_result)

    return downloaded_files

```

### Important Considerations

*   **Secret Management**: The plugin relies on Flyte's secret management for securely accessing your OpenAI API key. Ensure your `Secret` object is correctly configured and the corresponding secret is available in your Flyte environment. The `get_connector_secret` utility is used internally by the `BatchEndpointConnector` to retrieve the API key.
*   **Asynchronous Nature**: OpenAI batch jobs can take a significant amount of time to complete. The `BatchEndpointTask` is designed to wait for this completion, but it does not consume active compute resources during the waiting period, thanks to Flyte's asynchronous connector pattern.
*   **Resource Requirements**: The file upload and download tasks (`UploadJSONLFileTask`, `DownloadJSONFilesTask`) are configured with default memory requests (`700Mi`). Adjust these as necessary based on the size of the JSONL files being processed.
*   **Container Images**: The plugin uses specific container images for its tasks, defined by `OpenAIFileDefaultImages`. These images include the necessary OpenAI SDK and other dependencies. Ensure your Flyte environment can access these images or provide custom ones if needed.
*   **Error Handling**: The `BatchEndpointConnector` attempts to extract and surface error messages from OpenAI's batch job response if the job fails. These messages are included in the `message` field of the `Resource` object returned by the connector.
*   **OpenAI Organization ID**: While optional for some OpenAI API calls, specifying the `openai_organization` ID is a best practice, especially in environments with multiple organizations or for billing purposes.
<!--
key: summary_openai_batch_plugin_5d8fe447-beb4-4ef7-925b-10c29b70eb2b
type: summary_end

-->
<!--
code_unit: flytekitplugins.openai.batch.task
code_unit_type: class
help_text: ''
key: example_d9b95762-b33a-453c-93e0-d8e1ff57199f
type: example

-->