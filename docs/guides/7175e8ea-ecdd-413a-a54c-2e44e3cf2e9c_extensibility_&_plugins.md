
<!--
help_text: ''
key: summary_extensibility_&_plugins_47dd2aa5-d6a5-46f9-84ea-ce2e4bf8c127
modules:
- flytekit.core.python_auto_container
- flytekit.core.python_customized_container_task
- flytekit.core.shim_task
- flytekit.core.class_based_resolver
- flytekit.core.task
- flytekit.extend.backend.base_connector
- flytekit.extend.backend.connector_service
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
- flytekitplugins.deck.renderer
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
Extensibility & Plugins

Flytekit provides a robust framework for extending its core capabilities, allowing developers to integrate custom logic, specialized data types, and external systems seamlessly. This extensibility is crucial for adapting Flyte to diverse use cases and integrating with a wide array of technologies.

## Customizing Task Execution

Flytekit offers several mechanisms to customize how tasks execute, ranging from simple resource adjustments to full control over the underlying Kubernetes Pod.

### Python Auto-Container Tasks

The `PythonAutoContainerTask` serves as the foundational class for most Python tasks. It automatically handles container image selection and environment setup. Developers can customize task execution by configuring:

*   **Resources**: Specify CPU, memory, and GPU requests and limits using `requests` and `limits` parameters.
*   **Environment Variables**: Inject custom environment variables into the task container via the `environment` parameter.
*   **Secrets**: Securely access sensitive information by requesting secrets using `secret_requests`. These secrets are mounted into the container and accessible via the `SecretManager`.
*   **Pod Templates**: For advanced Kubernetes configurations, a `pod_template` can be provided to define the entire Pod specification. This allows for custom volumes, init containers, and other Kubernetes-native features.
*   **Accelerators**: Attach specific hardware accelerators to tasks using the `accelerator` parameter.
*   **Shared Memory**: Configure shared memory for inter-process communication within the container using `shared_memory`.

### Python Customized Container Tasks

For scenarios requiring more granular control over the container execution environment or integration with external container-based systems, the `PythonCustomizedContainerTask` is used. This class is typically employed when building new Flytekit-only plugins that interact with external services.

To implement a customized container task:

1.  **Define an Executor**: Subclass `ShimTaskExecutor` and override the `execute_from_model` method. This method contains the core business logic that will run inside the custom container. It receives the `TaskTemplate` (the serialized form of the task) and the task's inputs.
2.  **Define the Task**: Subclass `PythonCustomizedContainerTask`. In its constructor, provide the custom `executor_type` (the `ShimTaskExecutor` subclass).
3.  **Pass Custom Configuration**: Override the `get_custom` method in your `PythonCustomizedContainerTask` to serialize any necessary configuration into the `TaskTemplate`'s `custom` field. This data will be accessible to your `ShimTaskExecutor` at runtime.

This pattern ensures that only the essential configuration is serialized into the `TaskTemplate`, keeping it lightweight, while the heavy lifting of execution logic resides in the `ShimTaskExecutor`.

### Pod Tasks

The `PodFunctionTask` provides direct access to the underlying Kubernetes Pod specification. This is useful for advanced use cases where tasks require specific Kubernetes features like sidecar containers, complex volume mounts, or custom network configurations.

When using a `PodFunctionTask`:

*   Provide a `V1PodSpec` object to define the entire Pod.
*   Specify a `primary_container_name`. If a container with this name exists in the `pod_spec`, Flytekit will inject the Python task's execution logic into it. Otherwise, Flytekit will add a new container for the Python function.
*   Labels and annotations can be attached to the Pod for organizational or operational purposes.

## Task Resolution

Task resolvers are fundamental components that enable Flyte to locate and load task definitions at runtime. The `TaskResolverMixin` defines the interface for these resolvers.

*   **DefaultTaskResolver**: This is the standard resolver used for most Python tasks. It locates tasks by their module and name.
*   **DefaultNotebookTaskResolver**: Specifically designed for tasks defined within Jupyter notebooks, allowing them to be loaded correctly in a Flyte execution environment.
*   **TaskTemplateResolver**: Used by `PythonCustomizedContainerTask` to resolve tasks based on their serialized `TaskTemplate`, ensuring that the `ShimTaskExecutor` can be correctly instantiated.
*   **ClassStorageTaskResolver**: A specialized resolver that stores tasks in a class variable. This can be useful for scenarios like interactive development or testing where tasks are dynamically created and need to be referenced by an index.

## Backend Connectors (Agent-based Plugins)

Backend connectors, also known as Agent-based plugins, provide a powerful way to integrate Flyte with external systems and services without requiring custom Flyte backend deployments. They abstract the interaction with these external systems, allowing Flyte to orchestrate jobs on platforms like Databricks, Snowflake, or Airflow.

### Core Concepts

*   **ConnectorRegistry**: This is the central registry where all connectors are registered. Flyte's Agent service uses this registry to look up the appropriate connector based on the task type.
*   **ConnectorBase**: The abstract base class for all connectors. It defines the common properties and methods that all connectors must implement.
*   **ResourceMeta**: A dataclass used to store metadata about an external job (e.g., a job ID). This metadata is passed between Flyte and the connector to track the job's lifecycle.
*   **Resource**: A dataclass representing the status and outputs of an external job. It includes the job's phase, messages, log links, and any returned outputs.

### Asynchronous vs. Synchronous Connectors

Connectors can be either asynchronous or synchronous, depending on the nature of the external service they interact with.

*   **AsyncConnectorBase**: Used for long-running external jobs that require polling for status updates (e.g., a BigQuery query, an Airflow DAG run). Implementations must provide:
    *   `create`: Submits the job to the external system and returns a `ResourceMeta`.
    *   `get`: Polls the external system for job status and returns a `Resource`.
    *   `delete`: Cancels or cleans up the external job.
    *   `get_metrics` and `get_logs`: Optional methods to retrieve metrics and logs.
*   **SyncConnectorBase**: Used for external calls that return results immediately (e.g., a ChatGPT API call, a Boto3 API call). Implementations must provide:
    *   `do`: Executes the operation and returns a `Resource` directly.

### Local Execution with Mixins

`AsyncConnectorExecutorMixin` and `SyncConnectorExecutorMixin` are provided to enable local execution and testing of tasks that rely on backend connectors. By inheriting from these mixins, tasks can simulate the interaction with the Agent service, allowing developers to test their connector logic without deploying to a Flyte cluster.

### Implementing a New Connector

To create a new connector:

1.  Define a `ResourceMeta` dataclass to hold the external job's unique identifier and any other necessary state.
2.  Inherit from `AsyncConnectorBase` or `SyncConnectorBase`.
3.  Implement the required abstract methods (`create`, `get`, `delete` for async; `do` for sync).
4.  Register your connector with the `ConnectorRegistry` using `ConnectorRegistry.register()`.

## Extending the Type System (Type Transformers)

Type transformers are a core extensibility point that allows Flytekit to understand and handle custom Python types. A `TypeTransformer` defines how a Python object is serialized to and deserialized from Flyte's internal literal representation.

Key examples of type transformers include:

*   **Structured Dataset Transformers**: These transformers enable seamless integration with various data frame and dataset libraries by converting them to and from Flyte's `StructuredDataset` literal type. Examples include:
    *   `PanderaPandasTransformer`: For Pandas DataFrames with Pandera schemas.
    *   `PolarsDataFrameToParquetEncodingHandler` and `ParquetToPolarsDataFrameDecodingHandler`: For Polars DataFrames.
    *   `SparkToParquetEncodingHandler` and `ParquetToSparkDecodingHandler`: For PySpark DataFrames.
    *   `VaexDataFrameToParquetEncodingHandler` and `ParquetToVaexDataFrameDecodingHandler`: For Vaex DataFrames.
    *   `GeoPandasEncodingHandler` and `GeoPandasDecodingHandler`: For GeoPandas GeoDataFrames.
    *   `HuggingFaceDatasetToParquetEncodingHandler` and `ParquetToHuggingFaceDatasetDecodingHandler`: For HuggingFace Datasets.
*   **Configuration Transformers**:
    *   `DictConfigTransformer` and `ListConfigTransformer`: For handling OmegaConf `DictConfig` and `ListConfig` objects, preserving their structure and types.
*   **Model Format Transformers**:
    *   `PyTorch2ONNXTransformer`, `ScikitLearn2ONNXTransformer`, `TensorFlow2ONNXTransformer`: For converting machine learning models from popular frameworks (PyTorch, Scikit-learn, TensorFlow) to the ONNX format.
*   **Database/Data Versioning Transformers**:
    *   `DoltTableNameTransformer`: For integrating with Dolt, a version-controlled SQL database.
*   **Data Quality/Observability Transformers**:
    *   `GreatExpectationsTypeTransformer`: For integrating Great Expectations data validation directly into the type system.
    *   `WhylogsDatasetProfileTransformer`: For handling whylogs data profiles, enabling data observability.

## Customizing the Flyte UI (Deck Renderers)

Deck renderers allow developers to generate custom HTML content that is displayed in the Flyte UI, providing rich visualizations and interactive reports alongside task execution details.

Examples of deck renderers include:

*   `ImageRenderer`: Displays image data (from `FlyteFile` or PIL images).
*   `TableRenderer`: Renders Pandas DataFrames as HTML tables.
*   `BoxRenderer`: Generates Plotly box plots from DataFrames.
*   `MarkdownRenderer`: Converts Markdown text to HTML.
*   `GanttChartRenderer`: Creates Plotly Gantt charts, often used for visualizing timelines.
*   `FrameProfilingRenderer`: Generates detailed profiling reports for Pandas DataFrames (e.g., using `ydata-profiling`).
*   `SourceCodeRenderer`: Displays formatted source code.

These renderers can be used within tasks to publish custom reports that enhance the observability of your workflows.

## Common Plugin Use Cases

Flytekit's extensibility is demonstrated through a variety of built-in and community plugins, covering diverse domains:

*   **SQL and Query Tasks**:
    *   `AthenaTask`, `BigQueryTask`, `HiveTask`, `SnowflakeTask`: Execute SQL queries against respective data warehouses.
    *   `DuckDBQuery`: Run DuckDB queries, supporting various providers.
    *   `SQLAlchemyTask`: Execute client-side SQLAlchemy queries against any supported database.
*   **Distributed Machine Learning and Compute**:
    *   `PysparkFunctionTask`: Run PySpark jobs on Kubernetes.
    *   `RayFunctionTask`: Orchestrate Ray jobs on a KubeRay cluster.
    *   `PyTorchFunctionTask`, `TensorflowFunctionTask`, `MPIFunctionTask`: Run distributed training jobs using Kubeflow operators (PyTorch, TensorFlow, MPI).
    *   `DaskTask`: Execute Dask computations on a distributed Dask cluster.
*   **External Orchestrators**:
    *   `AirflowTask`: Integrate and run Airflow tasks within Flyte workflows.
    *   `PerianTask`, `PerianContainerTask`: Submit and manage jobs on the Perian Job Platform.
    *   `MMCloudTask`: Run tasks on the Memory Machine Cloud platform.
*   **MLOps Tracking and Observability**:
    *   `wandb_init`, `_neptune_init_run_class`, `_comet_ml_login_class`: Integrate with Weights & Biases, Neptune, and Comet ML for experiment tracking.
    *   `WhylogsDatasetProfileTransformer`: Enable data profiling and quality checks with whylogs.
*   **Data Validation**:
    *   `GreatExpectationsTask`: Perform data validation using Great Expectations.
    *   `PanderaPandasTransformer`: Validate Pandas DataFrames against Pandera schemas.
*   **Notebook Execution**:
    *   `NotebookTask`: Execute Jupyter notebooks as Flyte tasks using Papermill, capturing inputs, outputs, and rendered HTML.
*   **Infrastructure Management**:
    *   `DataServiceTask`: Manage Kubernetes data service resources directly from a Flyte task.
*   **Cloud API Interaction**:
    *   `BotoTask`: A generic task for invoking any Boto3 method for AWS services.
    *   `SageMakerEndpointTask`, `SageMakerModelTask`, `SageMakerInvokeEndpointTask`, `SageMakerEndpointConfigTask`, `SageMakerDeleteEndpointTask`, `SageMakerDeleteModelTask`, `SageMakerDeleteEndpointConfigTask`: Specialized tasks for managing SageMaker inference resources.
    *   `ChatGPTTask`: Interact with the OpenAI ChatGPT API.
    *   `BatchEndpointTask`, `UploadJSONLFileTask`, `DownloadJSONFilesTask`: Manage OpenAI batch processing jobs.
*   **Profiling**:
    *   `memray_profiling`: Decorator for memory profiling Python tasks using Memray.
*   **Image Building**:
    *   `EnvdImageSpecBuilder`: Integrates with Envd for building custom container images.

## Best Practices and Considerations

*   **Choosing the Right Extensibility Point**:
    *   For simple Python function tasks with custom resource needs, use `PythonAutoContainerTask`.
    *   For integrating with external services that have their own job lifecycle, consider building a backend connector.
    *   For fine-grained Kubernetes control or sidecar patterns, use `PodFunctionTask`.
    *   For custom data types, implement a `TypeTransformer`.
    *   For rich UI visualizations, leverage Deck renderers.
*   **Serialization**: Any custom objects or configurations passed between tasks or to external systems must be serializable. Flytekit often uses JSON, Protobuf, or Cloudpickle for this purpose. Be mindful of the size of serialized objects, especially for `TaskTemplate`s.
*   **Resource Management**: Always specify appropriate `requests` and `limits` for CPU, memory, and GPU to ensure efficient resource allocation and prevent resource starvation or over-provisioning.
*   **Secrets Management**: Use Flyte's built-in `Secret` mechanism to handle sensitive information like API keys or database credentials. Avoid hardcoding secrets.
*   **Performance**: For large datasets or computationally intensive tasks, consider using distributed computing plugins (Spark, Ray, Kubeflow) and optimizing data transfer mechanisms (e.g., using `StructuredDataset` with efficient formats like Parquet).
*   **Dependencies**: Ensure that all necessary Python packages and system dependencies are included in your custom container images. For plugins, the base image often needs specific tools or libraries (e.g., `dbt`, `airflow`, `boto3`).
*   **Error Handling**: Implement robust error handling within your custom task logic and connectors to provide clear messages and facilitate debugging.
<!--
key: summary_extensibility_&_plugins_47dd2aa5-d6a5-46f9-84ea-ce2e4bf8c127
type: summary_end

-->
<!--
code_unit: flytekitplugins.pod.task
code_unit_type: class
help_text: ''
key: example_53d7cec6-18e5-4ad0-84e8-b47338641cda
type: example

-->
<!--
code_unit: flytekitplugins.spark.task
code_unit_type: class
help_text: ''
key: example_00a08a45-2a40-4c02-a0d0-565432072501
type: example

-->
<!--
code_unit: flytekitplugins.athena.task
code_unit_type: class
help_text: ''
key: example_33526a09-1dcd-402b-96d5-a80b752969d9
type: example

-->