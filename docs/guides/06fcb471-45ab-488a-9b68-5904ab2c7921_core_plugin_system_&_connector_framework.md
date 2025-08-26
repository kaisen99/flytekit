
<!--
help_text: ''
key: summary_core_plugin_system_&_connector_framework_b03d5246-934e-4916-af44-230e845ac247
modules:
- flytekit.extend.backend.base_connector
- flytekit.extend.backend.connector_service
- flytekit.core.python_customized_container_task
- flytekit.core.shim_task
- flytekit.core.task
questions_to_answer: []
type: summary

-->
The Core Plugin System and Connector Framework in Flytekit enable developers to extend Flyte's capabilities by defining custom task types and integrating with external services. This framework provides a structured approach for creating tasks that execute in specialized environments or interact with third-party systems, supporting both synchronous and asynchronous operations.

## Core Plugin System: Defining Custom Task Types

The Core Plugin System allows developers to define custom task types where the execution logic is entirely encapsulated within the task's serialized `TaskTemplate`. This is particularly useful for tasks that require specific runtime environments, interact with non-Python services, or need to be lightweight without carrying extensive Python code.

### Key Components

*   **ShimTaskExecutor**: This abstract base class (`flytekit.core.shim_task.ShimTaskExecutor`) is the core of any custom task's business logic. Developers subclass `ShimTaskExecutor` and implement the `execute_from_model` method. This method receives the `TaskTemplate` (the serialized form of the task) and the task's inputs as Python native values. All execution logic must derive from the information available in the `TaskTemplate` and the provided inputs.
*   **ExecutableTemplateShimTask**: This class (`flytekit.core.shim_task.ExecutableTemplateShimTask`) acts as a wrapper, making a `TaskTemplate` executable by a `ShimTaskExecutor`. It implements the `dispatch_execute` and `execute` methods, allowing it to be treated similarly to a standard `PythonTask` by the Flytekit execution engine, even though it does not inherit from `PythonTask`. It infers the Python interface from the `TaskTemplate`'s Flyte IDL interface before execution.
*   **PythonCustomizedContainerTask**: This is the primary class (`flytekit.core.python_customized_container_task.PythonCustomizedContainerTask`) for users to define custom task types. It extends `ExecutableTemplateShimTask` and `PythonTask`, providing the scaffolding for tasks that run in custom containers.
    *   It requires specifying a `container_image` where the task will execute.
    *   An `executor_type` (a subclass of `ShimTaskExecutor`) must be provided, which will perform the actual computation.
    *   The `get_custom` method must be overridden to serialize any necessary configuration or data into the `TaskTemplate`'s `custom` field. This data is then accessible to the `ShimTaskExecutor` at runtime.
*   **TaskTemplateResolver**: The `TaskTemplateResolver` (`flytekit.core.python_customized_container_task.TaskTemplateResolver`) is a specialized resolver responsible for loading `PythonCustomizedContainerTask` instances at execution time. Unlike other resolvers that might restore the original Python object, this resolver always loads an `ExecutableTemplateShimTask` using only the `TaskTemplate` and the specified `ShimTaskExecutor` class. This ensures that only serialized information is used for execution.

### Implementation Guide

To create a custom task using the Core Plugin System:

1.  **Define a `ShimTaskExecutor`**:
    ```python
    from flytekit.core.shim_task import ShimTaskExecutor
    from flytekit.models.core import tasks as _task_model
    from typing import Any, Dict

    class MyCustomExecutor(ShimTaskExecutor):
        def execute_from_model(self, tt: _task_model.TaskTemplate, **kwargs) -> Any:
            # Access custom data from the task template
            custom_data = tt.custom
            # Perform business logic using inputs and custom_data
            print(f"Executing with custom data: {custom_data}")
            print(f"Inputs: {kwargs}")
            # Return outputs
            return {"output_key": f"Processed {kwargs.get('input_key', 'nothing')}"}
    ```
2.  **Define a `PythonCustomizedContainerTask`**:
    ```python
    from flytekit.core.python_customized_container_task import PythonCustomizedContainerTask
    from flytekit.core.resources import Resources
    from flytekit.core.interface import Interface
    from flytekit.models.core import tasks as _task_model
    from flytekit.core.task import TaskMetadata
    from flytekit.core.type_engine import TypeEngine
    from flytekit.core.context_manager import FlyteContext
    from flytekit.models.literals import LiteralMap
    from flytekit.models.types import LiteralType
    from flytekit.models.core.types import BlobType, BlobTypeFormat
    from flytekit.models.types import SimpleType
    from flytekit.models.interface import Variable
    from flytekit.models.literals import Literal, Scalar, Primitive, StringValue

    class MyCustomTaskConfig:
        def __init__(self, message: str):
            self.message = message

    class MyCustomTask(PythonCustomizedContainerTask[MyCustomTaskConfig]):
        def __init__(self, name: str, config: MyCustomTaskConfig, **kwargs):
            super().__init__(
                name=name,
                task_config=config,
                container_image="my-custom-image:latest", # Replace with your actual image
                executor_type=MyCustomExecutor,
                task_type="my_custom_task_type",
                interface=Interface(
                    inputs={"input_key": str},
                    outputs={"output_key": str}
                ),
                **kwargs
            )

        def get_custom(self, settings) -> Dict[str, Any]:
            # Serialize custom configuration for the executor
            return {"config_message": self.task_config.message}

    # Example usage:
    # my_task = MyCustomTask(name="my_first_custom_task", config=MyCustomTaskConfig(message="Hello from config!"))
    # @task(task_config=MyCustomTaskConfig(message="Hello from config!"))
    # def my_custom_task_func(input_key: str) -> str:
    #     # This function body is not executed, only its signature is used for interface
    #     pass
    # my_custom_task_func._task_instance = MyCustomTask(
    #     name="my_custom_task_func",
    #     config=MyCustomTaskConfig(message="Hello from config!"),
    #     interface=my_custom_task_func.python_interface,
    # )
    ```
    When defining a task using the `@task` decorator, the `task_config` argument can be used to pass an instance of `MyCustomTaskConfig`. Flytekit's internal mechanisms will then use `TaskPlugins.find_pythontask_plugin` to locate the appropriate `PythonCustomizedContainerTask` subclass (if registered) or default to `PythonFunctionTask`. For `PythonCustomizedContainerTask`, the `get_custom` method is crucial for passing data to the `ShimTaskExecutor`.

### Use Cases

*   **Specialized Runtimes**: Running tasks that require specific binaries, libraries, or environments not easily managed by standard Python tasks (e.g., R scripts, Go programs, custom ML frameworks).
*   **Lightweight Tasks**: When the task logic is simple and can be fully described by a small configuration in the `TaskTemplate`, avoiding the need to ship large Python dependencies.
*   **Integrating with Non-Python Services**: When the core logic involves calling an external service directly from the container, and the Python wrapper is minimal.

## Connector Framework: Integrating with External Services

The Connector Framework provides a standardized way for Flyte tasks to interact with external services, abstracting away the complexities of managing external job lifecycles. It supports both synchronous (blocking) and asynchronous (non-blocking) interactions.

### Key Components

*   **ConnectorBase**: This abstract base class (`flytekit.extend.backend.base_connector.ConnectorBase`) defines the common interface for all connectors. It primarily provides the `task_category` property, which identifies the specific task type and version that the connector supports.
*   **SyncConnectorBase**: Subclasses of `SyncConnectorBase` (`flytekit.extend.backend.base_connector.SyncConnectorBase`) are used for operations that complete quickly and return results instantly.
    *   The `do` method is the primary interface, taking a `TaskTemplate`, `output_prefix`, and `inputs`, and returning a `Resource` object indicating success or failure.
*   **AsyncConnectorBase**: Subclasses of `AsyncConnectorBase` (`flytekit.extend.backend.base_connector.AsyncConnectorBase`) are designed for long-running, asynchronous operations.
    *   `create`: Initiates an external job and returns a `ResourceMeta` object, which is a lightweight identifier for the created job.
    *   `get`: Polls the status of the external job using the `ResourceMeta` and returns a `Resource` object, which includes the current phase, messages, logs, and potentially outputs.
    *   `delete`: Terminates or cleans up the external job identified by `ResourceMeta`. This operation should be idempotent.
    *   `get_metrics` and `get_logs`: Optional methods to retrieve metrics and log links for the external job.
*   **ResourceMeta**: This class (`flytekit.extend.backend.base_connector.ResourceMeta`) represents a lightweight, serializable identifier for an external job or resource managed by a connector. It typically contains just enough information (e.g., an external job ID) to interact with the external service.
*   **Resource**: This class (`flytekit.extend.backend.base_connector.Resource`) encapsulates the status and output of an external job. It includes the `phase` (e.g., `SUCCEEDED`, `RUNNING`, `FAILED`), `message`, `log_links`, and `outputs` (either a `LiteralMap` or a Python dictionary).
*   **ConnectorRegistry**: The `ConnectorRegistry` (`flytekit.extend.backend.base_connector.ConnectorRegistry`) is a central singleton that manages all registered connectors.
    *   `register`: Connectors are registered with the registry, mapping a `TaskCategory` (task type name and version) to a specific connector instance. Each `TaskCategory` can only have one registered connector.
    *   `get_connector`: Retrieves a connector based on its `TaskCategory`.
    *   `list_connectors` and `get_connector_metadata`: Provide ways to discover and inspect registered connectors and their supported task types.
*   **Local Execution Mixins**:
    *   `SyncConnectorExecutorMixin` (`flytekit.extend.backend.base_connector.SyncConnectorExecutorMixin`) and `AsyncConnectorExecutorMixin` (`flytekit.extend.backend.base_connector.AsyncConnectorExecutorMixin`) are mixin classes that enable local execution of tasks backed by connectors. When a task inherits from one of these mixins, its `execute` method is overridden to interact with the `ConnectorRegistry` and invoke the appropriate connector's methods (`do` for sync, `create`/`get` for async) in a local asyncio loop. This greatly simplifies local testing and debugging of connector-backed tasks.
*   **Connector Services (Backend Integration)**:
    *   `SyncConnectorService` (`flytekit.extend.backend.connector_service.SyncConnectorService`) and `AsyncConnectorService` (`flytekit.extend.backend.connector_service.AsyncConnectorService`) are gRPC services that expose the registered connectors to the Flyte backend (Propeller). These services receive requests from Propeller (e.g., `ExecuteTaskSyncRequest`, `CreateTaskRequest`, `GetTaskRequest`) and delegate them to the appropriate connector retrieved from the `ConnectorRegistry`. This enables Flyte to orchestrate external jobs without direct knowledge of their underlying implementation details.

### Implementation Guide

To create a custom connector:

1.  **Define a `TaskCategory`**: This identifies the task type your connector will handle.
    ```python
    from flytekit.extend.backend.base_connector import TaskCategory
    my_task_category = TaskCategory(name="my_external_job", version=1)
    ```
2.  **Implement a `SyncConnectorBase` or `AsyncConnectorBase`**:
    ```python
    from flytekit.extend.backend.base_connector import SyncConnectorBase, AsyncConnectorBase, Resource, ResourceMeta
    from flytekit.models.core.tasks import TaskTemplate
    from flytekit.models.literals import LiteralMap
    from flytekit.models.core.execution import TaskExecution
    from typing import Optional, Dict, Any

    # Example Sync Connector
    class MySyncConnector(SyncConnectorBase):
        name = "MySyncConnector"
        def __init__(self, **kwargs):
            super().__init__(task_type_name="my_sync_task", **kwargs)

        def do(self, task_template: TaskTemplate, output_prefix: str, inputs: Optional[LiteralMap] = None, **kwargs) -> Resource:
            print(f"Executing sync task: {task_template.id.name}")
            print(f"Inputs: {inputs.to_json()}")
            # Simulate external call
            result_data = {"sync_output": "hello from sync connector"}
            return Resource(phase=TaskExecution.SUCCEEDED, outputs=result_data)

    # Example Async Connector
    class MyAsyncResourceMeta(ResourceMeta):
        job_id: str

    class MyAsyncConnector(AsyncConnectorBase):
        name = "MyAsyncConnector"
        def __init__(self, **kwargs):
            super().__init__(task_type_name="my_async_task", metadata_type=MyAsyncResourceMeta, **kwargs)

        def create(self, task_template: TaskTemplate, output_prefix: str, inputs: Optional[LiteralMap], task_execution_metadata: Optional[Any], **kwargs) -> ResourceMeta:
            print(f"Creating async job: {task_template.id.name}")
            # Simulate creating an external job and getting an ID
            job_id = f"job-{hash(task_template.id.name)}"
            return MyAsyncResourceMeta(job_id=job_id)

        def get(self, resource_meta: ResourceMeta, **kwargs) -> Resource:
            meta = resource_meta # type: MyAsyncResourceMeta
            print(f"Getting status for job: {meta.job_id}")
            # Simulate polling external job status
            if meta.job_id.endswith("completed"): # Simplified logic
                return Resource(phase=TaskExecution.SUCCEEDED, outputs={"async_output": f"Job {meta.job_id} finished"})
            return Resource(phase=TaskExecution.RUNNING, message="Job still running...")

        def delete(self, resource_meta: ResourceMeta, **kwargs):
            meta = resource_meta # type: MyAsyncResourceMeta
            print(f"Deleting job: {meta.job_id}")
            # Simulate deleting external job
            pass
    ```
3.  **Register the Connector**:
    ```python
    from flytekit.extend.backend.base_connector import ConnectorRegistry

    ConnectorRegistry.register(MySyncConnector())
    ConnectorRegistry.register(MyAsyncConnector())
    ```
    This registration typically happens during the Flytekit plugin initialization.

### Use Cases

*   **Querying Databases**: Tasks that execute SQL queries against external databases (e.g., BigQuery, Snowflake, Athena) and return results.
*   **Submitting Long-Running Jobs**: Tasks that submit jobs to external compute platforms (e.g., Spark clusters, Kubernetes jobs, specialized ML training services) and monitor their progress.
*   **Interacting with SaaS APIs**: Tasks that call external APIs for data retrieval, processing, or triggering actions in third-party systems.

## Relationship and Interaction

The Core Plugin System and Connector Framework are complementary. A `PythonCustomizedContainerTask` (from the Core Plugin System) can be designed to leverage a `Connector` (from the Connector Framework) internally.

When a `PythonCustomizedContainerTask` is executed, its `ShimTaskExecutor` can interact with the `ConnectorRegistry` to retrieve the appropriate `Connector` based on the task's `task_type`. The `ShimTaskExecutor` then uses this `Connector` to perform the actual interaction with the external service.

For example, a `PythonCustomizedContainerTask` defined for "BigQuery queries" would have a `ShimTaskExecutor` that, when `execute_from_model` is called, retrieves the `BigQueryConnector` from the `ConnectorRegistry` and invokes its `create`, `get`, or `do` methods to manage the BigQuery job. This separation of concerns allows for flexible and modular integration with diverse external systems.

## Best Practices and Considerations

*   **Task Template Size**: Keep the `custom` field within the `TaskTemplate` concise. It should only contain essential configuration for the `ShimTaskExecutor` or `Connector`. Large data should be passed via inputs or external storage.
*   **Idempotency**: Ensure that `delete` operations in `AsyncConnectorBase` are idempotent. Repeated calls should not cause errors or unintended side effects.
*   **Error Handling**: Implement robust error handling within `ShimTaskExecutor` and `Connector` methods. Propagate meaningful exceptions to provide clear feedback to users.
*   **Local Testing**: Utilize the `SyncConnectorExecutorMixin` and `AsyncConnectorExecutorMixin` for local development and testing. These mixins allow you to run connector-backed tasks directly in your local Python environment, simulating the interaction with the `ConnectorRegistry` and the connector's methods.
*   **Resource Management**: For `AsyncConnectorBase`, ensure the `delete` method properly cleans up any external resources created by the `create` method to prevent resource leaks.
*   **Task Category Uniqueness**: When defining new connectors, ensure the `TaskCategory` (name and version) is unique to avoid conflicts in the `ConnectorRegistry`. This ensures that Flyte can correctly map task types to their corresponding connectors.
*   **Asynchronous Operations**: When implementing `AsyncConnectorBase`, ensure that the `create`, `get`, and `delete` methods are truly asynchronous (e.g., using `asyncio` and `await`) to prevent blocking the main event loop of the connector service. The `mirror_async_methods` utility helps bridge synchronous connector implementations with the asynchronous gRPC service.
<!--
key: summary_core_plugin_system_&_connector_framework_b03d5246-934e-4916-af44-230e845ac247
type: summary_end

-->
<!--
code_unit: flytekit.extend.backend.examples.custom_connector_tutorial
code_unit_type: class
help_text: ''
key: example_3d641ee4-929f-48bb-9d98-b3189d19ca7a
type: example

-->
<!--
code_unit: flytekit.core.task.examples.custom_task_type
code_unit_type: class
help_text: ''
key: example_25fd7cfd-c7c6-47c8-9c01-ee6fe8235313
type: example

-->