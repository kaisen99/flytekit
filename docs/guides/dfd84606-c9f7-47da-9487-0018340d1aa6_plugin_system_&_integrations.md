
<!--
help_text: ''
key: summary_plugin_system_&_integrations_233e78ed-63e0-49e4-9ee7-988b53b709dd
modules:
- flytekit.extend.backend.base_connector
- flytekit.extend.backend.connector_service
- flytekit.core.python_customized_container_task
- flytekit.core.shim_task
- flytekit.core.task
- flytekitplugins.airflow.connector
- flytekitplugins.airflow.task
- flytekitplugins.async_fsspec.s3fs.s3fs
- flytekitplugins.athena.task
- flytekitplugins.awsbatch.task
- flytekitplugins.awssagemaker_inference.boto3_connector
- flytekitplugins.awssagemaker_inference.boto3_mixin
- flytekitplugins.awssagemaker_inference.boto3_task
- flytekitplugins.awssagemaker_inference.connector
- flytekitplugins.awssagemaker_inference.task
- flytekitplugins.bigquery.connector
- flytekitplugins.bigquery.task
- flytekitplugins.comet_ml.tracking
- flytekitplugins.dask.models
- flytekitplugins.dask.task
- flytekitplugins.dbt.error
- flytekitplugins.dbt.schema
- flytekitplugins.dbt.task
- flytekitplugins.dolt.schema
- flytekitplugins.duckdb.task
- flytekitplugins.envd.image_builder
- flytekitplugins.geopandas.gdf_transformers
- flytekitplugins.great_expectations.schema
- flytekitplugins.great_expectations.task
- flytekitplugins.hive.task
- flytekitplugins.huggingface.sd_transformers
- flytekitplugins.identity_aware_proxy.cli
- flytekitplugins.inference.nim.serve
- flytekitplugins.inference.ollama.serve
- flytekitplugins.inference.sidecar_template
- flytekitplugins.inference.vllm.serve
- flytekitplugins.k8sdataservice.connector
- flytekitplugins.k8sdataservice.k8s.kube_config
- flytekitplugins.k8sdataservice.k8s.manager
- flytekitplugins.k8sdataservice.sensor
- flytekitplugins.k8sdataservice.task
- flytekitplugins.kfmpi.task
- flytekitplugins.kfpytorch.task
- flytekitplugins.kftensorflow.task
- flytekitplugins.memray
- flytekitplugins.memray.profiling
- flytekitplugins.mmcloud.connector
- flytekitplugins.mmcloud.task
- flytekitplugins.modin.schema
- flytekitplugins.neptune.tracking
- flytekitplugins.omegaconf
- flytekitplugins.omegaconf.config
- flytekitplugins.omegaconf.dictconfig_transformer
- flytekitplugins.omegaconf.listconfig_transformer
- flytekitplugins.onnxpytorch.schema
- flytekitplugins.onnxscikitlearn.schema
- flytekitplugins.onnxtensorflow.schema
- flytekitplugins.openai.batch.connector
- flytekitplugins.openai.batch.task
- flytekitplugins.openai.chatgpt.connector
- flytekitplugins.openai.chatgpt.task
- flytekitplugins.optuna
- flytekitplugins.optuna.optimizer
- flytekitplugins.pandera.config
- flytekitplugins.pandera.pandas_renderer
- flytekitplugins.pandera.pandas_transformer
- flytekitplugins.papermill.task
- flytekitplugins.perian_job.connector
- flytekitplugins.perian_job.task
- flytekitplugins.pod.task
- flytekitplugins.polars.sd_transformers
- flytekitplugins.ray.models
- flytekitplugins.ray.task
- flytekitplugins.slurm.function.connector
- flytekitplugins.slurm.function.task
- flytekitplugins.slurm.script.connector
- flytekitplugins.slurm.script.task
- flytekitplugins.slurm.ssh_utils
- flytekitplugins.snowflake.connector
- flytekitplugins.snowflake.task
- flytekitplugins.spark.connector
- flytekitplugins.spark.generic_task
- flytekitplugins.spark.models
- flytekitplugins.spark.pyspark_transformers
- flytekitplugins.spark.schema
- flytekitplugins.spark.sd_transformers
- flytekitplugins.spark.task
- flytekitplugins.sqlalchemy.task
- flytekitplugins.vaex.sd_transformers
- flytekitplugins.wandb
- flytekitplugins.wandb.tracking
- flytekitplugins.whylogs
- flytekitplugins.whylogs.renderer
- flytekitplugins.whylogs.schema
- flytekitplugins.xarray.xarray_transformers
questions_to_answer: []
type: summary

-->
Flytekit's Plugin System & Integrations provide powerful mechanisms to extend the platform's capabilities, enabling seamless interaction with external services, custom execution environments, and specialized data formats. This system allows developers to tailor Flytekit to unique infrastructure requirements and integrate with a wide array of third-party tools.

### Core Extension Patterns

Flytekit offers several patterns for extending its functionality:

1.  **Backend Connectors (Agents):** These integrate Flytekit with external services, allowing tasks to delegate their execution to a remote system (e.g., a managed database service, a distributed training platform, or a SaaS API).
2.  **Customized Container Tasks:** This pattern enables defining tasks where the execution logic is handled by a custom executor within a container, rather than directly executing Python user code. It is ideal for tasks that primarily orchestrate external commands or interact with APIs.
3.  **Python Function Task Plugins:** This is the traditional method for extending `PythonFunctionTask` with domain-specific configurations and serialization logic, often used for distributed computing frameworks.
4.  **Data Type Extensions (Transformers & Renderers):** These extend Flytekit's type system to support custom data formats, ensuring proper serialization, deserialization, and visualization within the Flyte UI.
5.  **Observability & Tracking Integrations:** These provide decorators to integrate tasks with external experiment tracking and profiling platforms.

### Backend Connectors (Agents)

Backend connectors, also known as Agents, allow Flytekit tasks to offload their execution to external services. This is particularly useful for long-running or specialized operations that are best managed by a dedicated external system.

#### Connector Lifecycle

All connectors adhere to a standardized lifecycle, defined by base classes:

*   **`ConnectorBase`**: The abstract base class for all connectors, defining common properties like `task_category`. A `TaskCategory` uniquely identifies the type and version of a task that a connector supports.
*   **`AsyncConnectorBase`**: Used for asynchronous operations where task execution is initiated, and its status is polled over time. This is suitable for long-running jobs like database queries or distributed training.
    *   `create`: Initiates the external job and returns a `ResourceMeta` object.
    *   `get`: Polls the status of the external job using the `ResourceMeta` and returns a `Resource` object, which includes the current phase, messages, and optional outputs.
    *   `delete`: Requests the termination of the external job. This operation should be idempotent.
    *   `get_metrics` and `get_logs`: Optional methods to retrieve metrics and logs from the external system.
*   **`SyncConnectorBase`**: Used for synchronous operations where the task execution completes and returns results within a single call. This is suitable for quick API calls or metadata retrieval.
    *   `do`: Executes the external operation and returns a `Resource` object directly.

#### Resource Management

*   **`ResourceMeta`**: This class encapsulates metadata about an external job, such as its ID. It provides `encode` and `decode` methods to serialize and deserialize this metadata, allowing FlytePropeller to persist and retrieve job-specific information.
*   **`Resource`**: Represents the current state and output of an external job. It includes:
    *   `phase`: The current execution phase (e.g., `RUNNING`, `SUCCEEDED`, `FAILED`).
    *   `message`: An optional message providing more details about the job's status.
    *   `log_links`: Links to external logs (e.g., a BigQuery console link).
    *   `outputs`: Optional literal map containing the job's outputs.
    *   `custom_info`: Additional custom information.

#### Connector Registry

The `ConnectorRegistry` (`flytekit.extend.backend.base_connector.ConnectorRegistry`) acts as a central lookup for all registered connectors.
*   `register`: Connectors register themselves with the registry, mapping a `TaskCategory` to a specific connector instance. Each `TaskCategory` can only have one associated connector.
*   `get_connector`: Retrieves the appropriate connector based on a given task type name and version.
*   `list_connectors` and `get_connector_metadata`: Provide introspection capabilities to list available connectors and their supported task types.

#### Local Execution of Connector-Based Tasks

Flytekit provides mixin classes to enable local execution of tasks that would otherwise run via a backend connector:

*   **`AsyncConnectorExecutorMixin`**: For tasks using `AsyncConnectorBase`. During local execution, this mixin simulates the asynchronous polling behavior by repeatedly calling the connector's `get` method until a terminal phase is reached. It also includes signal handling for graceful cleanup.
*   **`SyncConnectorExecutorMixin`**: For tasks using `SyncConnectorBase`. During local execution, this mixin directly invokes the connector's `do` method.

These mixins allow developers to test connector-based tasks locally without deploying to a Flyte cluster or connecting to the actual external service, provided the connector itself can run in the local environment.

### Customized Container Tasks

The customized container task pattern offers a flexible way to define tasks where the primary execution logic resides within a custom container image and is executed by a specialized Python class. This is distinct from standard Python tasks where Flytekit directly invokes a Python function.

*   **`PythonCustomizedContainerTask`**: This task type (`flytekit.core.python_customized_container_task.PythonCustomizedContainerTask`) is the entry point for defining such tasks. It requires:
    *   A `container_image` where the custom execution logic will run.
    *   An `executor_type` which is a subclass of `ShimTaskExecutor`.
    *   An overridden `get_custom` method to serialize any necessary configuration for the `ShimTaskExecutor` into the `TaskTemplate`.

*   **`ExecutableTemplateShimTask`**: This is a base class that `PythonCustomizedContainerTask` inherits from. It represents a task whose execution is driven by its `TaskTemplate` and a `ShimTaskExecutor`. At runtime, it reconstructs the `TaskTemplate` and passes it to the executor.

*   **`ShimTaskExecutor`**: This abstract base class (`flytekit.core.shim_task.ShimTaskExecutor`) defines the core execution logic for customized container tasks.
    *   `execute_from_model`: This method must be overridden by concrete executor implementations. It receives the `TaskTemplate` (including any custom data serialized via `get_custom`) and the task inputs. All business logic for the task should reside here.

*   **`TaskTemplateResolver`**: This resolver (`flytekit.core.python_customized_container_task.TaskTemplateResolver`) is specifically designed for `PythonCustomizedContainerTask`. It loads the `TaskTemplate` at execution time and uses it to instantiate the appropriate `ShimTaskExecutor`.

This pattern is ideal for building plugins that wrap external CLI tools, call REST APIs, or manage resources in a third-party system, where the Python function itself doesn't contain the direct computational logic but rather orchestrates an external process.

### Python Function Task Plugins

Flytekit provides a plugin system for `PythonFunctionTask` that allows developers to extend its behavior with custom configurations and serialization. This is the most common way to integrate distributed computing frameworks or specialized execution environments.

*   **`TaskPlugins`**: This factory (`flytekit.core.task.TaskPlugins`) manages the registration and lookup of `PythonFunctionTask` derivatives.
    *   `register_pythontask_plugin`: Used to associate a custom `task_config` type with a specific `PythonFunctionTask` subclass.
    *   `find_pythontask_plugin`: Retrieves the appropriate plugin based on the `task_config` type provided to the `@task` decorator.

*   **Plugin Implementation:** A plugin typically involves:
    *   Defining a custom configuration class (e.g., `Spark` for Spark tasks, `AWSBatchConfig` for AWS Batch tasks).
    *   Subclassing `PythonFunctionTask` (e.g., `PysparkFunctionTask`, `AWSBatchFunctionTask`).
    *   Overriding the `get_custom` method to serialize the plugin-specific configuration into the `TaskTemplate`'s `custom` field. This `custom` field is then interpreted by the Flyte backend plugin (e.g., the Spark plugin in FlytePropeller).
    *   Optionally overriding `get_command` or `get_container` to customize how the task's container is launched.

### Data Type Extensions (Transformers & Renderers)

Flytekit's type engine can be extended to support custom Python data types, ensuring they are correctly serialized to and deserialized from Flyte's `Literal` format. This also includes rendering capabilities for rich visualization in the Flyte UI.

*   **`TypeTransformer`**: The core component for type extension. Subclasses of `TypeTransformer` define:
    *   `get_literal_type`: How the Python type maps to a Flyte `LiteralType`.
    *   `to_literal`: Converts a Python object to a Flyte `Literal`.
    *   `to_python_value`: Converts a Flyte `Literal` back to a Python object.
    *   `to_html`: (Optional) Generates an HTML representation of the Python object for display in the Flyte UI.

*   **`StructuredDatasetEncoder` / `StructuredDatasetDecoder`**: These are specialized components for handling types that map to Flyte's `StructuredDataset` literal type. They define how dataframes (e.g., Pandas, Spark, Polars, Vaex, GeoPandas, HuggingFace Datasets) are written to and read from remote storage (e.g., Parquet files in S3/GCS).

*   **`StructuredDatasetRenderer`**: Provides the `to_html` method for `StructuredDataset` types, enabling rich visualizations of dataframes directly in the Flyte UI.

### Observability & Tracking Integrations

Flytekit offers decorators to integrate tasks with external experiment tracking and profiling platforms, enhancing observability of machine learning workflows.

*   **`ClassDecorator`**: A base class for creating decorators that wrap task functions, allowing for pre- and post-execution logic injection.
*   **`wandb_init`**: Integrates with Weights & Biases. When applied to a task, it initializes a W&B run, logs task metadata, and ensures the run is finalized upon task completion. It handles API key retrieval from Flyte secrets.
*   **`_neptune_init_run_class`**: Integrates with Neptune.ai. Similar to W&B, it initializes a Neptune run, logs execution and task-specific metadata, and manages API key access.
*   **`_comet_ml_login_class`**: Integrates with Comet ML. It handles logging into Comet ML and associating experiments with Flyte task executions, including generating experiment keys and linking to Flyte execution URLs.
*   **`memray_profiling`**: Integrates with Memray for memory profiling. This decorator wraps the task function with a Memray tracker, generates a memory usage report (e.g., flame graph), and publishes it to the Flyte Deck for visualization.

### Specific Plugin Implementations

This section highlights common use cases and implementation details for various plugins.

#### Airflow Integration

The Airflow plugin enables Flyte tasks to execute Airflow operators, sensors, and hooks.

*   **`AirflowConnector`**: An `AsyncConnectorBase` that interacts with Airflow. It handles the submission of Airflow operators (especially deferrable ones) and polls for their status. Sensors and hooks are typically executed directly within the `get` method.
*   **`AirflowTask`**: A `PythonTask` that uses `AsyncConnectorExecutorMixin` to leverage the `AirflowConnector`. It serializes the Airflow object's module, name, and parameters into the task's custom configuration.
*   **`AirflowContainerTask`**: For Airflow operators that are *not* deferrable (e.g., some Beam operators), this task type runs the Airflow operator directly within a Flyte container, as they cannot be managed asynchronously by the connector.
*   **`AirflowObj`**: A data class used to serialize Airflow operator/sensor/hook details for the connector.

#### SQL Query Tasks (BigQuery, Snowflake, Athena)

Flytekit provides specialized tasks for executing SQL queries against various data warehouses. These tasks typically use `AsyncConnectorBase` for long-running queries.

*   **`BigQueryTask`**: Executes BigQuery SQL queries. It uses `BigQueryConnector` to submit and monitor BigQuery jobs. Outputs can be captured as `StructuredDataset` pointing to the resulting BigQuery table.
*   **`SnowflakeTask`**: Executes Snowflake SQL queries. It uses `SnowflakeConnector` to manage query execution and status. Outputs can be captured as `StructuredDataset`.
*   **`AthenaTask`**: Executes Athena SQL queries. It uses the `presto` backend plugin (which is a generic SQL plugin) and serializes Athena-specific configurations like database, workgroup, and catalog.

#### OpenAI Integrations

Flytekit integrates with OpenAI APIs for various AI capabilities.

*   **`ChatGPTTask`**: A `SyncConnectorExecutorMixin` based task that uses `ChatGPTConnector` to interact with OpenAI's chat completion API. It's designed for quick, synchronous interactions.
*   **`BatchEndpointTask`**: An `AsyncConnectorExecutorMixin` based task that uses `BatchEndpointConnector` for submitting and monitoring OpenAI batch jobs. This is suitable for larger, asynchronous processing.
*   **`UploadJSONLFileTask` / `DownloadJSONFilesTask`**: These are `PythonCustomizedContainerTask` implementations that use `ShimTaskExecutor` to handle file uploads and downloads to/from OpenAI's file storage, which is a prerequisite for batch processing. They demonstrate the use of `ShimTaskExecutor` for specific utility operations.

#### Kubernetes Data Service

The K8s Data Service plugin enables Flyte to manage and interact with stateful Kubernetes services, such as graph databases or other long-running data services.

*   **`DataServiceConnector`**: An `AsyncConnectorBase` that interacts with the Kubernetes API (`K8sManager`) to create, get the status of, and delete StatefulSets and Services.
*   **`DataServiceTask`**: A `PythonTask` using `AsyncConnectorExecutorMixin` to define and manage the lifecycle of a Kubernetes data service. It allows specifying image, command, resources, and replicas for the StatefulSet.
*   **`CleanupSensor`**: A specialized sensor task that can be used in a workflow to monitor the status of a related job (e.g., a training job) and trigger the cleanup (deletion) of the data service resources once the job is complete. This ensures resource efficiency.

#### Slurm Integration

The Slurm plugin allows Flyte tasks to execute jobs on Slurm clusters via SSH.

*   **`SlurmFunctionTask`**: A `PythonFunctionTask` using `AsyncConnectorExecutorMixin` that executes a Python function on a Slurm cluster. It allows specifying SSH connection details and `sbatch` configurations. The task function's execution is wrapped within a generated Slurm script.
*   **`SlurmShellTask`**: Similar to `SlurmFunctionTask`, but designed to execute an arbitrary shell script on the Slurm cluster. It supports interpolating inputs and capturing outputs from specified file locations.
*   **`SlurmFunctionConnector` / `SlurmScriptConnector`**: These `AsyncConnectorBase` implementations handle the SSH connection, script transfer, job submission (`sbatch`), status polling (`scontrol`), and job cancellation (`scancel`). They maintain an SSH connection pool for efficiency.

#### Perian Job Platform

The Perian Job plugin integrates Flyte with the Perian Job Platform for executing containerized jobs.

*   **`PerianConnector`**: An `AsyncConnectorBase` that interacts with the Perian API to create, get status, and cancel jobs. It handles authentication via Flyte secrets.
*   **`PerianTask`**: A `PythonFunctionTask` using `AsyncConnectorExecutorMixin` to run Python functions as jobs on Perian. It allows configuring job requirements like CPU, memory, accelerators, and cloud provider.
*   **`PerianContainerTask`**: A `PythonTask` using `AsyncConnectorExecutorMixin` for running arbitrary container images and commands on Perian, similar to `PerianTask` but without direct Python function execution.

#### Model Inference Serving (NIM, Ollama)

Flytekit provides plugins for deploying and managing model inference servers as sidecars within Flyte tasks.

*   **`ModelInferenceTemplate`**: A base class that provides common functionality for defining Kubernetes pod templates for model servers. It handles container configuration, resource requests, health checks, and optional input downloading.
*   **`NIM`**: Extends `ModelInferenceTemplate` to deploy NVIDIA Inference Microservices (NIM). It includes specific configurations for NGC API keys, shared memory, and downloading LoRA adapters from Hugging Face.
*   **`Ollama`**: Extends `ModelInferenceTemplate` to deploy Ollama for local large language model serving. It handles pulling or creating models based on a Modelfile, and ensures the Ollama service is ready before the main task execution.

#### Papermill Integration

The Papermill plugin enables executing Jupyter notebooks as Flyte tasks.

*   **`NotebookTask`**: A `PythonInstanceTask` that wraps a Jupyter notebook. It uses Papermill to execute the notebook, injects inputs into a designated "parameters" cell, and extracts outputs from a cell tagged "outputs".
*   **Implicit Outputs**: `NotebookTask` automatically produces two outputs: the executed notebook (`out_nb` as `PythonNotebook`) and an HTML-rendered version of the notebook (`out_rendered_nb` as `HTMLPage`), which can be displayed in the Flyte UI.
*   **`render_deck`**: An option to automatically render the HTML notebook into a Flyte Deck for immediate visualization in the console.

#### SQLAlchemy Integration

The SQLAlchemy plugin allows executing client-side SQL queries using SQLAlchemy.

*   **`SQLAlchemyTask`**: A `PythonCustomizedContainerTask` that uses `SQLAlchemyTaskExecutor`. It takes a SQL query template and SQLAlchemy connection configuration (URI, connect arguments, and secrets).
*   **`SQLAlchemyTaskExecutor`**: A `ShimTaskExecutor` that establishes a SQLAlchemy connection, interpolates the query with task inputs, executes the query, and optionally returns results as a Pandas DataFrame (which can then be converted to a `FlyteSchema`). This demonstrates how `ShimTaskExecutor` can be used for database interactions without requiring a full backend connector.

### Important Considerations

*   **Secrets Management:** Many integrations, especially those with external services (e.g., OpenAI, Perian, Snowflake, Slurm, W&B, Neptune, Comet ML), rely on Flyte's secrets management system to securely access API keys, credentials, or other sensitive information. Ensure secrets are properly configured and mounted.
*   **Idempotency:** For `AsyncConnectorBase` implementations, the `create` and `delete` methods should ideally be idempotent to handle retries and ensure consistent state.
*   **Error Handling:** Connectors should map external service errors and statuses to Flyte's `TaskExecution.Phase` and provide informative messages and log links in the `Resource` object.
*   **Performance:** Be mindful of data transfer costs and latency when integrating with external services or using custom data type transformers that involve large datasets.
*   **Container Images:** Many plugins require specific dependencies or base images. Ensure your task's container image includes all necessary libraries for the chosen plugin. For `PythonCustomizedContainerTask`, the `executor_type`'s dependencies must be present in the `container_image`.
*   **Local vs. Remote Execution:** While mixins enable local testing of connector-based tasks, the full behavior and performance characteristics are best observed in a deployed Flyte environment. Some plugins (e.g., Spark, Ray) have specific considerations for local execution.
*   **Resource Management:** When using plugins that launch external resources (e.g., Spark clusters, K8s Data Services), ensure proper cleanup mechanisms are in place (e.g., `CleanupSensor` for K8s Data Service, `ttl_seconds_after_finished` for Kubeflow jobs) to avoid orphaned resources and unnecessary costs.
*   **Task Template Size:** For `PythonCustomizedContainerTask`, the `TaskTemplate` (including custom data) should remain small, as it is frequently accessed by the Flyte engine. Large configurations should be stored in external systems and referenced.
<!--
key: summary_plugin_system_&_integrations_233e78ed-63e0-49e4-9ee7-988b53b709dd
type: summary_end

-->
<!--
code_unit: flytekitplugins.spark.task.PysparkFunctionTask
code_unit_type: class
help_text: ''
key: example_ede65532-c9a6-4b29-a5d7-27abf43edd67
type: example

-->
<!--
code_unit: flytekitplugins.athena.task.AthenaTask
code_unit_type: class
help_text: ''
key: example_39c8e4fc-3fe6-445b-bf67-5057625ab632
type: example

-->