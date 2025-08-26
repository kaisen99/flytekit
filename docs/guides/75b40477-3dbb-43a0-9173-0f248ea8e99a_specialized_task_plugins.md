
<!--
help_text: ''
key: summary_specialized_task_plugins_ec1f38f4-ff6a-40a9-ae00-8e799badad6d
modules:
- flytekitplugins.airflow.task
- flytekitplugins.airflow.connector
- flytekitplugins.awsbatch.task
- flytekitplugins.awssagemaker_inference.task
- flytekitplugins.awssagemaker_inference.connector
- flytekitplugins.awssagemaker_inference.boto3_connector
- flytekitplugins.awssagemaker_inference.boto3_mixin
- flytekitplugins.k8sdataservice.task
- flytekitplugins.k8sdataservice.connector
- flytekitplugins.k8sdataservice.sensor
- flytekitplugins.k8sdataservice.k8s.manager
- flytekitplugins.k8sdataservice.k8s.kube_config
- flytekitplugins.mmcloud.task
- flytekitplugins.mmcloud.connector
- flytekitplugins.papermill.task
- flytekitplugins.openai.batch.task
- flytekitplugins.openai.batch.connector
- flytekitplugins.openai.chatgpt.task
- flytekitplugins.openai.chatgpt.connector
- flytekitplugins.perian_job.task
- flytekitplugins.perian_job.connector
- flytekitplugins.pod.task
- flytekitplugins.envd.image_builder
- flytekitplugins.async_fsspec.s3fs.s3fs
- flytekitplugins.identity_aware_proxy.cli
- flytekitplugins.inference.sidecar_template
- flytekitplugins.inference.nim.serve
- flytekitplugins.inference.ollama.serve
- flytekitplugins.inference.vllm.serve
- flytekitplugins.optuna.optimizer
- flytekitplugins.optuna
- flytekitplugins.omegaconf.dictconfig_transformer
- flytekitplugins.omegaconf.listconfig_transformer
- flytekitplugins.omegaconf.config
- flytekitplugins.deck.renderer
- flytekit.core.task
questions_to_answer: []
type: summary

-->
Specialized Task Plugins

Specialized Task Plugins extend Flytekit's core task capabilities, enabling seamless integration with external systems, specialized compute environments, or advanced data processing paradigms. These plugins allow developers to define tasks that leverage the unique features of various platforms and tools, abstracting away the underlying complexities.

Each specialized task plugin typically consists of:
*   A **configuration object** (e.g., `AirflowObj`, `AWSBatchConfig`) that defines the plugin-specific parameters.
*   A **task class** (e.g., `AirflowTask`, `AWSBatchFunctionTask`) that inherits from a core Flytekit task type (like `PythonFunctionTask` or `PythonTask`) and uses the configuration object to customize its behavior.
*   An **asynchronous connector** (e.g., `AirflowConnector`, `AWSBatchConnector`) that handles the communication and lifecycle management with the external system, often running on the Flyte backend (FlytePropeller).

This architecture allows Flyte to orchestrate tasks on diverse platforms while maintaining a consistent interface for users.

## External Compute and Orchestration Plugins

These plugins enable Flyte tasks to offload execution to external compute services or integrate with other orchestration systems.

### Airflow Tasks

The Airflow plugin allows you to wrap existing Apache Airflow operators, sensors, or hooks as Flyte tasks. This is particularly useful for integrating legacy Airflow DAGs or leveraging Airflow's extensive ecosystem of connectors within Flyte workflows.

*   **`AirflowObj`**: A data class used to serialize the Airflow task's module, name, and parameters. This object is stored in the Flyte task configuration.
*   **`AirflowTask`**: Represents a deferrable Airflow task. These tasks are executed by the `AirflowConnector` on the FlytePropeller, which polls Airflow for status updates. This is suitable for long-running or asynchronous Airflow operations.
*   **`AirflowContainerTask`**: Used for non-deferrable Airflow operators that cannot be run asynchronously by the connector. These tasks execute the Airflow operator directly within the Flyte task's container.
*   **`AirflowTaskResolver`**: Responsible for loading the Airflow task definition within the container at execution time.
*   **`AirflowConnector`**: The asynchronous connector that interacts with Airflow. It handles the submission and monitoring of Airflow tasks. Sensors and hooks are typically invoked in the `get` method to check conditions or perform immediate actions, while deferrable operators are submitted in `create` and monitored in `get` via Airflow triggers.

**Capabilities:**
*   Execute any Airflow `BaseOperator`, `BaseSensorOperator`, or `BaseTrigger`.
*   Support for deferrable Airflow tasks, allowing FlytePropeller to offload polling to the connector, freeing up compute resources.
*   Direct execution of non-deferrable Airflow operators within the task container.

**Usage Example:**

```python
from flytekitplugins.airflow.task import AirflowTask, AirflowObj
from airflow.sensors.filesystem import FileSensor

# Define an Airflow FileSensor as a Flyte task
file_sensor_task = AirflowTask(
    name="my_file_sensor",
    task_config=AirflowObj(
        module="airflow.sensors.filesystem",
        name="FileSensor",
        parameters={"task_id": "check_file", "filepath": "/tmp/data.csv"}
    ),
    inputs={}, # Sensors typically don't have direct inputs in Flyte
    outputs={}
)

# In a workflow, you can then use:
# @workflow
# def my_airflow_workflow():
#     file_sensor_task()
```

### AWS Batch Tasks

The AWS Batch plugin enables you to execute Python functions as jobs on AWS Batch, leveraging its capabilities for managing large-scale, high-performance computing workloads.

*   **`AWSBatchConfig`**: Configures the AWS Batch job submission, including parameters like `schedulingPriority`, `platformCapabilities`, and `tags`. This directly maps to AWS Batch's `SubmitJobInput`.
*   **`AWSBatchFunctionTask`**: A specialized `PythonFunctionTask` that transforms a local Python function into an AWS Batch job. The task's container image and command are used to define the Batch job definition.

**Capabilities:**
*   Run Python functions as managed AWS Batch jobs.
*   Configure AWS Batch job parameters directly from Flytekit.
*   Utilize AWS Batch's scaling and scheduling features.

**Usage Example:**

```python
from flytekitplugins.awsbatch.task import AWSBatchFunctionTask, AWSBatchConfig
from flytekit import task

@task
def my_python_function(x: int) -> int:
    return x * 2

aws_batch_task = AWSBatchFunctionTask(
    task_function=my_python_function,
    task_config=AWSBatchConfig(
        platformCapabilities="EC2",
        parameters={"jobName": "my-flyte-batch-job"},
    ),
    # Other task arguments like requests, limits, container_image can also be set
)

# In a workflow:
# @workflow
# def my_batch_workflow(x: int) -> int:
#     return aws_batch_task(x=x)
```

### Memory Machine Cloud (MMCloud) Tasks

The MMCloud plugin allows you to submit and manage tasks on the Memory Machine Cloud OpCenter, a platform designed for memory-intensive workloads.

*   **`MMCloudConfig`**: Provides configuration options specific to MMCloud, such as `submit_extra` for additional arguments to the `float submit` command.
*   **`MMCloudTask`**: A `PythonFunctionTask` subclass that wraps a Python function for execution on MMCloud. It translates Flytekit resource requests into MMCloud-specific resource parameters.
*   **`MMCloudConnector`**: Manages the lifecycle of MMCloud jobs, including login, submission, status polling, and cancellation. It interacts with the `float` CLI tool.

**Capabilities:**
*   Execute Python functions on the Memory Machine Cloud platform.
*   Map Flytekit resource requests (CPU, memory) to MMCloud requirements.
*   Pass additional, custom arguments to the MMCloud job submission.

**Usage Example:**

```python
from flytekitplugins.mmcloud.task import MMCloudTask, MMCloudConfig
from flytekit import task, Resources

@task
def process_large_data(data_path: str) -> str:
    # Simulate a memory-intensive operation
    return f"Processed {data_path} on MMCloud"

mmcloud_task = MMCloudTask(
    task_function=process_large_data,
    task_config=MMCloudConfig(submit_extra="--project my-mmcloud-project"),
    requests=Resources(mem="100Gi", cpu="16"), # These are translated to MMCloud resources
)

# In a workflow:
# @workflow
# def mmcloud_workflow(path: str) -> str:
#     return mmcloud_task(data_path=path)
```

### Perian Job Platform Tasks

The Perian Job Platform plugin integrates Flyte with Perian.io, enabling the execution of containerized jobs with fine-grained control over compute resources and cloud providers.

*   **`PerianConfig`**: Defines the resource requirements and execution environment for a Perian job, including CPU cores, memory, accelerators, OS storage, country code, and cloud provider.
*   **`PerianTask`**: A `PythonFunctionTask` subclass for running Python functions on Perian. It automatically packages the function and its dependencies into a container for Perian execution.
*   **`PerianContainerTask`**: A `PythonTask` subclass for running pre-built container images on Perian. This is useful when you have a custom container with a specific command to execute.
*   **`PerianConnector`**: Handles the communication with the Perian API for job creation, status retrieval, and cancellation. It also manages storage credentials for data access.

**Capabilities:**
*   Execute Python functions or custom containers on the Perian Job Platform.
*   Specify detailed resource requirements (CPU, memory, GPU type and count).
*   Select the desired country code and cloud provider for job execution.
*   Automatic handling of storage credentials for Flyte's data plane.

**Usage Example:**

```python
from flytekitplugins.perian_job.task import PerianTask, PerianConfig
from flytekit import task

@task
def train_model(epochs: int) -> float:
    # Simulate model training
    return 0.95

perian_training_task = PerianTask(
    task_function=train_model,
    task_config=PerianConfig(
        cores=8,
        memory=32, # GB
        accelerators=1,
        accelerator_type="A100",
        country_code="US",
        provider="aws",
    ),
    container_image="my-custom-ml-image:latest", # Required for PerianTask
)

# For a pre-built container:
# perian_container_task = PerianContainerTask(
#     name="my_container_job",
#     task_config=PerianConfig(cores=4, memory=16),
#     image="my-docker-image:v1",
#     command=["python", "my_script.py", "--epochs", "{{.inputs.epochs}}"],
#     inputs={"epochs": int},
# )

# In a workflow:
# @workflow
# def perian_workflow(e: int) -> float:
#     return perian_training_task(epochs=e)
```

## Cloud Machine Learning Service Plugins

These plugins provide direct integrations with specific cloud-based machine learning services, simplifying the management of ML resources.

### AWS SageMaker Inference Tasks

The AWS SageMaker Inference plugin provides tasks for managing the lifecycle of SageMaker models and endpoints, from creation to invocation and deletion.

*   **`SageMakerModelTask`**: Creates a SageMaker model.
*   **`SageMakerEndpointConfigTask`**: Creates a SageMaker endpoint configuration.
*   **`SageMakerEndpointTask`**: Creates a SageMaker endpoint. This task is asynchronous and uses the `SageMakerEndpointConnector` to monitor the endpoint's status.
*   **`SageMakerInvokeEndpointTask`**: Invokes a SageMaker endpoint for inference.
*   **`SageMakerDeleteModelTask`**, **`SageMakerDeleteEndpointConfigTask`**, **`SageMakerDeleteEndpointTask`**: Tasks for cleaning up SageMaker resources.
*   **`SageMakerEndpointConnector`**: The asynchronous connector responsible for interacting with the SageMaker API to create, monitor, and delete endpoints.

**Capabilities:**
*   Programmatically manage SageMaker models, endpoint configurations, and endpoints.
*   Invoke SageMaker endpoints for real-time inference.
*   Automate the deployment and teardown of ML inference infrastructure.

**Usage Example (Endpoint Creation):**

```python
from flytekitplugins.awssagemaker_inference.task import SageMakerEndpointTask
from flytekit import task

# Example configuration for a SageMaker endpoint
endpoint_config = {
    "EndpointName": "my-test-endpoint",
    "EndpointConfigName": "my-test-endpoint-config",
    # ... other SageMaker endpoint configuration parameters
}

create_endpoint_task = SageMakerEndpointTask(
    name="create_sagemaker_endpoint",
    config=endpoint_config,
    region="us-east-1",
    inputs={},
    outputs={"result": dict},
)

# In a workflow:
# @workflow
# def deploy_sagemaker_endpoint():
#     endpoint_info = create_endpoint_task()
#     # Use endpoint_info for subsequent inference tasks
```

### OpenAI Tasks

The OpenAI plugin provides tasks for interacting with OpenAI's API, specifically for batch processing and chat completions.

*   **`OpenAIFileConfig`**: Configuration for OpenAI file operations, including secret management for API keys.
*   **`UploadJSONLFileTask`**: Uploads a JSONL file to OpenAI for batch processing.
*   **`DownloadJSONFilesTask`**: Downloads output and error JSONL files from completed OpenAI batch jobs.
*   **`BatchEndpointTask`**: Submits and monitors an OpenAI batch job. This task is asynchronous and uses the `BatchEndpointConnector`.
*   **`ChatGPTTask`**: Performs a chat completion using the OpenAI ChatGPT API. This task is synchronous and uses the `ChatGPTConnector`.
*   **`BatchEndpointConnector`**: Manages the lifecycle of OpenAI batch jobs.
*   **`ChatGPTConnector`**: Handles synchronous calls to the ChatGPT API.

**Capabilities:**
*   Automate data preparation (upload) and result retrieval (download) for OpenAI batch jobs.
*   Orchestrate long-running OpenAI batch inference tasks.
*   Integrate real-time ChatGPT interactions into workflows.

**Usage Example (ChatGPT):**

```python
from flytekitplugins.openai.chatgpt.task import ChatGPTTask
from flytekit import task, Secret

# Define a secret for your OpenAI API key
openai_secret = Secret(group="openai", key="api_key")

chat_task = ChatGPTTask(
    name="my_chat_task",
    chatgpt_config={"model": "gpt-3.5-turbo"},
    openai_organization="your-openai-org-id", # Optional
    secret_requests=[openai_secret],
)

# In a workflow:
# @workflow
# def chat_workflow(prompt: str) -> str:
#     return chat_task(message=prompt)
```

## Kubernetes Native Plugins

These plugins offer direct control over Kubernetes resources, allowing for advanced deployment patterns and stateful service management.

### K8s Data Service Tasks

The K8s Data Service plugin enables the deployment and management of stateful services (like graph engines) within a Kubernetes cluster directly from Flyte workflows.

*   **`DataServiceConfig`**: Defines the configuration for the data service, including image, command, port, replicas, and resource requests/limits. It also supports referencing an `ExistingReleaseName` for pre-existing services.
*   **`DataServiceTask`**: An asynchronous task that uses the `DataServiceConnector` to create, monitor, and manage Kubernetes StatefulSets and Services.
*   **`DataServiceConnector`**: Interacts with the Kubernetes API to deploy and manage the data service. It checks the StatefulSet status for task phase updates.
*   **`CleanupSensor`**: A specialized sensor that can be used in a workflow to trigger the deletion of data service resources after a dependent task or workflow completes.

**Capabilities:**
*   Deploy and manage stateful Kubernetes applications (e.g., databases, graph engines) as part of a Flyte workflow.
*   Control resource allocation, image, and command for the deployed service.
*   Monitor the health and readiness of the deployed service.
*   Automate cleanup of deployed resources.

**Usage Example:**

```python
from flytekitplugins.k8sdataservice.task import DataServiceTask, DataServiceConfig
from flytekit import task, workflow, Resources

# Define a data service configuration
data_service_config = DataServiceConfig(
    Name="my-graph-engine",
    Image="my-graph-engine-image:latest",
    Command=["/app/start-graph-engine.sh"],
    Port=8080,
    Replicas=1,
    Requests=Resources(cpu="1", mem="2Gi"),
)

# Define the data service task
deploy_data_service = DataServiceTask(
    name="deploy_graph_engine",
    task_config=data_service_config,
    inputs={},
    outputs={"name": str},
)

# In a workflow:
# @workflow
# def graph_engine_workflow():
#     service_name = deploy_data_service()
#     # Use the deployed service, e.g., connect to it
#     # cleanup_sensor(release_name=service_name, cleanup_data_service=True)
```

### Pod Tasks

The Pod plugin provides the most granular control over the Kubernetes Pod specification for task execution. It allows users to define a complete `V1PodSpec`, including multiple containers, volumes, and advanced Kubernetes features.

*   **`Pod`**: The configuration object that holds the `V1PodSpec` and specifies the `primary_container_name`.
*   **`PodFunctionTask`**: A `PythonFunctionTask` subclass that uses the provided `V1PodSpec` to define the Kubernetes Pod where the task will run. It automatically injects or modifies the primary container to ensure the Python function executes correctly within the custom pod.

**Capabilities:**
*   Define multi-container pods (sidecars) for tasks.
*   Mount custom volumes, configure network policies, and set node affinities.
*   Utilize advanced Kubernetes features not directly exposed by other Flytekit task types.
*   Deploy model servers as sidecars alongside the main task container (e.g., using `ModelInferenceTemplate` for NIM, Ollama, VLLM).

**Usage Example:**

```python
from flytekitplugins.pod.task import PodFunctionTask, Pod
from flytekit import task
from kubernetes.client.models import V1PodSpec, V1Container, V1Volume, V1VolumeMount

@task
def my_pod_task_function(x: int) -> int:
    # This function runs in the primary container of the custom pod
    return x * 2

# Define a custom pod spec with a sidecar
custom_pod_spec = V1PodSpec(
    containers=[
        V1Container(
            name="my-sidecar-container",
            image="busybox",
            command=["sh", "-c", "echo 'Hello from sidecar' && sleep 3600"],
            volume_mounts=[V1VolumeMount(name="shared-data", mount_path="/data")],
        ),
    ],
    volumes=[V1Volume(name="shared-data", empty_dir={})],
)

pod_task = PodFunctionTask(
    task_function=my_pod_task_function,
    task_config=Pod(
        pod_spec=custom_pod_spec,
        primary_container_name="flytekit-image", # Default name for the primary container
    ),
)

# In a workflow:
# @workflow
# def custom_pod_workflow(val: int) -> int:
#     return pod_task(x=val)
```

## Developer Productivity and Utility Plugins

These plugins enhance the developer experience by providing specialized tools for common development patterns, such as notebook execution, hyperparameter optimization, and rich data visualization.

### Papermill Notebook Tasks

The Papermill plugin allows you to execute Jupyter notebooks as Flyte tasks, enabling a seamless transition from interactive development to production workflows.

*   **`NotebookTask`**: Wraps a Jupyter notebook for execution. It handles parameter injection into the notebook, extracts outputs from designated cells, and can render the executed notebook to HTML for visualization.

**Capabilities:**
*   Execute Jupyter notebooks with Flyte task inputs as notebook parameters.
*   Extract outputs from specific notebook cells.
*   Generate an HTML rendering of the executed notebook, viewable in the Flyte UI.
*   Stream notebook logs to the task's stdout.

**Usage Example:**

```python
from flytekitplugins.papermill.task import NotebookTask
from flytekit import kwtypes

# Assuming 'my_notebook.ipynb' exists with a 'parameters' cell
# and an 'outputs' cell that calls record_outputs(x=..., y=...)

nb_task = NotebookTask(
    name="my_notebook_execution",
    notebook_path="my_notebook.ipynb",
    inputs=kwtypes(input_param=int),
    outputs=kwtypes(output_x=int, output_y=str),
    render_deck=True, # Render the executed notebook as a deck
    stream_logs=True, # Stream notebook output to task logs
)

# In a workflow:
# @workflow
# def notebook_workflow(param: int) -> (int, str):
#     x, y, _, _ = nb_task(input_param=param) # Implicit outputs for notebook and rendered notebook
#     return x, y
```

### Optuna Optimizer

The Optuna plugin facilitates distributed hyperparameter optimization by integrating Optuna studies with Flyte tasks.

*   **`Optimizer`**: The core class for defining an optimization study. It takes an `objective` function (which can be a Flyte `AsyncPythonFunctionTask` or a regular callable), `concurrency`, and `n_trials`. It manages the Optuna `Study` object and dispatches trials.
*   **`Integer`**, **`Float`**, **`Category`**: Classes representing different types of hyperparameter suggestions (search spaces) for Optuna trials.

**Capabilities:**
*   Perform hyperparameter optimization for Flyte tasks in a distributed manner.
*   Define search spaces for integer, float, and categorical parameters.
*   Control the number of concurrent trials and total trials.
*   Integrate with existing Optuna studies.

**Usage Example:**

```python
from flytekitplugins.optuna.optimizer import Optimizer, Float
from flytekit import task, workflow

@task
def train_model_with_hparams(learning_rate: float, batch_size: int) -> float:
    # Simulate training and return a metric to optimize (e.g., accuracy)
    return learning_rate * batch_size / 100.0

# Define the optimizer
optimizer = Optimizer(
    objective=train_model_with_hparams,
    concurrency=5,
    n_trials=20,
)

# In a workflow:
# @workflow
# def hyperparameter_tuning_workflow():
#     # Define the search space for the objective function's parameters
#     optimizer(
#         learning_rate=Float(low=0.001, high=0.1, log=True),
#         batch_size=Integer(low=16, high=128, step=16),
#     )
#     # Access the best trial from optimizer.study.best_trial
```

### OmegaConf Transformers

The OmegaConf plugin provides type transformers for `DictConfig` and `ListConfig` objects from the OmegaConf library, enabling their use as native Flyte task inputs and outputs.

*   **`DictConfigTransformer`**: Handles the serialization and deserialization of `DictConfig` objects.
*   **`ListConfigTransformer`**: Handles the serialization and deserialization of `ListConfig` objects.

**Capabilities:**
*   Pass complex, structured configurations defined with OmegaConf directly as task inputs and outputs.
*   Maintain type information and structure of OmegaConf objects across task boundaries.

**Usage Example:**

```python
from flytekitplugins.omegaconf.dictconfig_transformer import DictConfigTransformer
from omegaconf import DictConfig, OmegaConf
from flytekit import task

# Register the transformer (usually done automatically by the plugin)
# TypeEngine.register(DictConfigTransformer())

@task
def process_config(cfg: DictConfig) -> DictConfig:
    print(f"Processing config: {OmegaConf.to_yaml(cfg)}")
    cfg.processed = True
    return cfg

# In a workflow:
# @workflow
# def config_workflow():
#     my_config = OmegaConf.create({"model": {"name": "resnet", "version": 50}})
#     processed_cfg = process_config(cfg=my_config)
#     print(f"Processed config in workflow: {OmegaConf.to_yaml(processed_cfg)}")
```

### Deck Renderers

Deck renderers allow tasks to generate rich HTML visualizations that are displayed directly in the Flyte UI, providing immediate insights into task outputs without needing to download files.

*   **`BoxRenderer`**: Generates box plots from pandas DataFrames.
*   **`ImageRenderer`**: Converts `FlyteFile` or PIL `Image.Image` objects into HTML image tags.
*   **`SourceCodeRenderer`**: Renders Python source code with syntax highlighting.
*   **`TableRenderer`**: Converts pandas DataFrames into styled HTML tables.
*   **`MarkdownRenderer`**: Converts Markdown text into HTML.
*   **`GanttChartRenderer`**: Creates Gantt charts from DataFrames, useful for visualizing timelines.
*   **`FrameProfilingRenderer`**: Generates a detailed data profiling report (using `ydata-profiling`) from a pandas DataFrame.

**Capabilities:**
*   Embed interactive plots, tables, images, and reports directly into task execution details.
*   Improve observability and debugging of workflows.
*   Provide immediate visual feedback on data transformations or model performance.

**Usage Example:**

```python
from flytekitplugins.deck.renderer import TableRenderer
from flytekit import task, Deck
import pandas as pd

@task(render_deck=True)
def analyze_data(data: pd.DataFrame) -> pd.DataFrame:
    # Perform some analysis
    summary = data.describe()

    # Create a deck to display the summary table
    deck = Deck("Data Summary")
    deck.append(TableRenderer().to_html(summary))
    # Attach the deck to the task's execution context
    Deck.append_text(deck)

    return summary

# In a workflow:
# @workflow
# def data_analysis_workflow():
#     my_data = pd.DataFrame({"col1": [1, 2, 3], "col2": [4, 5, 6]})
#     analyze_data(data=my_data)
```

### Echo Task

The Echo task is a simple pass-through task. It takes inputs and returns them directly as outputs without executing any user-defined code within a container. This is primarily used for testing or specific internal FlytePropeller optimizations.

*   **`Echo`**: A `PythonTask` that mirrors its inputs to its outputs.

**Capabilities:**
*   Acts as a no-op task for data mirroring.
*   Executed directly by FlytePropeller, avoiding container startup overhead.

**Usage Example:**

```python
from flytekit.core.task import Echo
from flytekit import kwtypes

echo_task = Echo(
    name="my_echo_task",
    inputs=kwtypes(message=str, value=int),
)

# In a workflow:
# @workflow
# def echo_workflow(m: str, v: int) -> (str, int):
#     # The outputs will be (m, v)
#     return echo_task(message=m, value=v)
```
<!--
key: summary_specialized_task_plugins_ec1f38f4-ff6a-40a9-ae00-8e799badad6d
type: summary_end

-->
<!--
code_unit: flytekitplugins.airflow.task.AirflowTask
code_unit_type: class
help_text: ''
key: example_f4de7330-67fe-496c-950b-79a657866a28
type: example

-->
<!--
code_unit: flytekitplugins.awsbatch.task.AWSBatchFunctionTask
code_unit_type: class
help_text: ''
key: example_f0e62416-f983-4b8a-81f0-ff5ffd58449a
type: example

-->
<!--
code_unit: flytekitplugins.awssagemaker_inference.task.SageMakerModelTask
code_unit_type: class
help_text: ''
key: example_6943cd4c-e9c3-43bc-a1a0-1fec65e3da2e
type: example

-->
<!--
code_unit: flytekitplugins.k8sdataservice.task.DataServiceTask
code_unit_type: class
help_text: ''
key: example_3fb5dcf8-8d08-4c5d-9016-33bca2d87fa9
type: example

-->
<!--
code_unit: flytekitplugins.mmcloud.task.MMCloudTask
code_unit_type: class
help_text: ''
key: example_4f7ea357-9c76-4c21-8509-25f472dec043
type: example

-->
<!--
code_unit: flytekitplugins.papermill.task.NotebookTask
code_unit_type: class
help_text: ''
key: example_601a6f50-702c-4fea-a221-5a04b3c91d74
type: example

-->
<!--
code_unit: flytekitplugins.openai.batch.task.BatchEndpointTask
code_unit_type: class
help_text: ''
key: example_3e2ea888-08ac-4453-8b31-283ec02d242b
type: example

-->
<!--
code_unit: flytekitplugins.openai.chatgpt.task.ChatGPTTask
code_unit_type: class
help_text: ''
key: example_2a4e65ae-758d-4bb9-8aef-32600ba952a5
type: example

-->
<!--
code_unit: flytekitplugins.perian_job.task.PerianTask
code_unit_type: class
help_text: ''
key: example_0ae58c1e-c1d1-4dba-b11e-d72abc86d19e
type: example

-->
<!--
code_unit: flytekitplugins.pod.task.PodFunctionTask
code_unit_type: class
help_text: ''
key: example_58b39033-e516-479e-84e0-87c2510b989a
type: example

-->
<!--
code_unit: flytekitplugins.envd.image_builder.EnvdImageSpecBuilder
code_unit_type: class
help_text: ''
key: example_ec1c4736-4c99-4a75-a1fa-776fd7f1bd8e
type: example

-->
<!--
code_unit: flytekitplugins.async_fsspec.s3fs.s3fs.AsyncS3FileSystem
code_unit_type: class
help_text: ''
key: example_997e5f7a-2b7d-4b0e-91ec-4f9e1df7736f
type: example

-->
<!--
code_unit: flytekitplugins.identity_aware_proxy.cli.GCPIdentityAwareProxyAuthenticator
code_unit_type: class
help_text: ''
key: example_a8feb33d-9b9a-4f0b-9fed-6e3cfba75083
type: example

-->
<!--
code_unit: flytekitplugins.inference.nim.serve.NIM
code_unit_type: class
help_text: ''
key: example_3f77288a-5e28-4fcf-83b5-8c419307630d
type: example

-->
<!--
code_unit: flytekitplugins.inference.ollama.serve.Ollama
code_unit_type: class
help_text: ''
key: example_87e304d8-388a-4e5a-9c21-a7395dfc5562
type: example

-->
<!--
code_unit: flytekitplugins.inference.vllm.serve.VLLM
code_unit_type: class
help_text: ''
key: example_1dd36c20-5727-4f11-818f-919bf0a29a12
type: example

-->
<!--
code_unit: flytekitplugins.optuna.optimizer.Optimizer
code_unit_type: class
help_text: ''
key: example_d21b3965-8370-42fe-8654-5f2918cb065e
type: example

-->
<!--
code_unit: flytekitplugins.omegaconf.dictconfig_transformer.DictConfigTransformer
code_unit_type: class
help_text: ''
key: example_24721039-cc21-4e69-bb0c-7a4231e6ea43
type: example

-->
<!--
code_unit: flytekitplugins.deck.renderer.TableRenderer
code_unit_type: class
help_text: ''
key: example_636a5092-34ce-4124-bfc0-afdd5857b397
type: example

-->
<!--
code_unit: flytekit.core.task.Echo
code_unit_type: class
help_text: ''
key: example_0a07a3ad-085c-4054-b447-97f31a8bd013
type: example

-->