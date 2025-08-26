
<!--
help_text: ''
key: summary_agent-based_task_extensions_(connectors)_d3a007da-29d0-4125-b42b-7dda9f7a0b81
modules:
- flytekit.extend.backend.base_connector
- flytekit.extend.backend.connector_service
questions_to_answer: []
type: summary

-->
Agent-based Task Extensions (Connectors) enable Flyte to integrate with and manage tasks executed on external systems. This mechanism allows Flyte to orchestrate workflows that involve diverse backend services, treating them as first-class tasks.

### Core Concepts

**Task Categories**
A `TaskCategory` uniquely identifies a type of task that a connector supports. It consists of a `name` and an optional `version`. Connectors are registered and looked up based on their associated `TaskCategory`. For example, a BigQuery connector might support a task category named "bigquery_query" with version 1.

**Resource Metadata**
The `ResourceMeta` object serves as a lightweight identifier for an external job or resource managed by a connector. When a connector initiates an external task, it returns an instance of `ResourceMeta`. This metadata, typically containing an external job ID, is then used by Flyte to query the status, retrieve logs, or delete the external task. `ResourceMeta` instances can be encoded to and decoded from bytes, facilitating their transmission between Flyte components.

**Resource Status and Outputs**
The `Resource` object encapsulates the current state and results of an external task. It provides:
*   `phase`: The current execution phase of the external task (e.g., `RUNNING`, `SUCCEEDED`, `FAILED`).
*   `message`: An optional message providing more details about the task's status.
*   `log_links`: A list of links to external logs, such as a link to a BigQuery console job page.
*   `outputs`: The results of the task. This can be a `LiteralMap` (Flyte's internal representation of structured data) or a standard Python dictionary, which the system automatically converts.
*   `custom_info`: An optional dictionary for any additional custom information relevant to the task.

### Connector Types

Connectors extend the `ConnectorBase` abstract class and are categorized into two primary types based on their execution model:

**Asynchronous Connectors**
`AsyncConnectorBase` is designed for long-running tasks that do not return results immediately. These tasks typically involve polling an external system for status updates.
*   `create(task_template, output_prefix, inputs, task_execution_metadata)`: Initiates the external task and returns a `ResourceMeta` object. This method is responsible for submitting the job to the external system.
*   `get(resource_meta)`: Queries the status of the external task using the provided `ResourceMeta` and returns a `Resource` object. This method is called repeatedly by Flyte's backend until the task reaches a terminal phase.
*   `delete(resource_meta)`: Requests the external system to terminate or clean up the task identified by `resource_meta`. This operation should be idempotent.
*   `get_metrics(resource_meta)`: Retrieves metrics related to the external task.
*   `get_logs(resource_meta)`: Retrieves log links for the external task.

**Synchronous Connectors**
`SyncConnectorBase` is suitable for short-running tasks that complete quickly and return their results in a single call.
*   `do(task_template, output_prefix, inputs)`: Executes the task and returns a `Resource` object directly, containing the final status and outputs. This method is invoked once.

### Connector Management

**The Connector Registry**
The `ConnectorRegistry` serves as the central repository for all available connectors. It maps `TaskCategory` instances to their corresponding connector implementations.
*   `register(connector, override=False)`: Adds a connector instance to the registry. Each `TaskCategory` can only have one registered connector. Registering a connector also updates its metadata, including supported task types and whether it's synchronous.
*   `get_connector(task_type_name, task_type_version=0)`: Retrieves a connector instance based on its `TaskCategory`. If no connector is found for the specified category, a `FlyteConnectorNotFound` error occurs.
*   `list_connectors()`: Returns a list of all registered agents (connectors) and their metadata.
*   `get_connector_metadata(name)`: Retrieves metadata for a specific connector by its name.

### Backend Services

Flyte's backend interacts with connectors through gRPC services, which are responsible for handling task lifecycle operations.

**Asynchronous Connector Service**
The `AsyncConnectorService` implements the asynchronous agent service interface. It acts as a bridge between Flyte's control plane (Propeller) and the `AsyncConnectorBase` implementations.
*   `CreateTask`: Receives a request to create a task, looks up the appropriate `AsyncConnectorBase` in the `ConnectorRegistry`, and invokes its `create()` method.
*   `GetTask`: Queries the status of an existing task by calling the `get()` method of the relevant `AsyncConnectorBase`.
*   `DeleteTask`: Requests the deletion of a task by invoking the `delete()` method of the `AsyncConnectorBase`.
*   `GetTaskMetrics`: Retrieves task-specific metrics using the `get_metrics()` method.
*   `GetTaskLogs`: Retrieves task log links using the `get_logs()` method.

**Synchronous Connector Service**
The `SyncConnectorService` implements the synchronous agent service interface. It handles requests for immediate task execution.
*   `ExecuteTaskSync`: Receives a request for synchronous task execution, retrieves the corresponding `SyncConnectorBase` from the `ConnectorRegistry`, and invokes its `do()` method. The result is returned directly in the response.

**Connector Metadata Service**
The `ConnectorMetadataService` provides an interface for discovering registered connectors.
*   `GetAgent`: Retrieves metadata for a specific connector by name.
*   `ListAgents`: Returns a list of all registered connectors and their capabilities.

### Local Execution

Flytekit provides mixin classes that enable local execution of tasks that would otherwise use a connector in a deployed environment. This is crucial for local development and testing.

**Running Synchronous Tasks Locally**
Tasks that inherit from `SyncConnectorExecutorMixin` can be executed locally. The `execute()` method of this mixin retrieves the appropriate `SyncConnectorBase` from the `ConnectorRegistry` and directly calls its `do()` method. This simulates the backend service interaction without requiring a deployed connector service. Timeout settings specified in the task template are ignored during local execution.

**Running Asynchronous Tasks Locally**
Tasks inheriting from `AsyncConnectorExecutorMixin` can also run locally. The `execute()` method simulates the asynchronous lifecycle:
1.  It calls the connector's `create()` method to initiate the task.
2.  It then enters a polling loop, repeatedly calling the connector's `get()` method until the task reaches a terminal phase (succeeded, failed, etc.).
3.  If the task succeeds and the connector does not return outputs directly, the mixin attempts to read outputs from a remote file path specified by `output_prefix`.
Timeout settings are ignored during local execution.

**Cleanup during Local Execution**
For asynchronous tasks run locally, a signal handler is registered. If the local execution is interrupted (e.g., via Ctrl+C), this handler attempts to call the connector's `delete()` method to clean up any external resources created by the task.

### Implementation Guide

To create a custom agent-based task extension:

1.  **Define a Custom Connector:**
    *   Create a new class that inherits from either `AsyncConnectorBase` or `SyncConnectorBase`.
    *   Implement the abstract methods (`create`, `get`, `delete` for async; `do` for sync) to interact with your external system.
    *   Define a `name` property for your connector and specify the `task_type_name` and `task_type_version` in its `__init__` method to set its `TaskCategory`.
    *   For `AsyncConnectorBase`, specify the `metadata_type` to be a subclass of `ResourceMeta` that can encode/decode your external job identifier.

    ```python
    from flytekit.extend.backend.base_connector import AsyncConnectorBase, ResourceMeta, Resource, TaskCategory
    from flytekit.models.core.tasks import TaskTemplate
    from flytekit.models.literals import LiteralMap
    from flytekit.models.core.execution import TaskExecution
    from typing import Optional, Dict, Any

    # Define custom resource metadata for your external system
    class MyExternalJobMeta(ResourceMeta):
        job_id: str

    class MyAsyncConnector(AsyncConnectorBase):
        name = "my_custom_async_connector"

        def __init__(self):
            super().__init__(task_type_name="my_async_task", task_type_version=1, metadata_type=MyExternalJobMeta)

        async def create(
            self,
            task_template: TaskTemplate,
            output_prefix: str,
            inputs: Optional[LiteralMap],
            task_execution_metadata: Optional[Any],
            **kwargs,
        ) -> MyExternalJobMeta:
            # Logic to submit job to external system
            print(f"Creating external job for task: {task_template.id.name}")
            external_job_id = "job-123" # Replace with actual external job submission
            return MyExternalJobMeta(job_id=external_job_id)

        async def get(self, resource_meta: MyExternalJobMeta, **kwargs) -> Resource:
            # Logic to check status of external job
            print(f"Checking status of external job: {resource_meta.job_id}")
            # Simulate job completion
            if resource_meta.job_id == "job-123":
                return Resource(phase=TaskExecution.SUCCEEDED, message="Job completed successfully", outputs={"result": 42})
            return Resource(phase=TaskExecution.RUNNING, message="Job still running")

        async def delete(self, resource_meta: MyExternalJobMeta, **kwargs):
            # Logic to delete external job
            print(f"Deleting external job: {resource_meta.job_id}")

    class MySyncConnector(SyncConnectorBase):
        name = "my_custom_sync_connector"

        def __init__(self):
            super().__init__(task_type_name="my_sync_task", task_type_version=1)

        async def do(
            self, task_template: TaskTemplate, output_prefix: str, inputs: Optional[LiteralMap] = None, **kwargs
        ) -> Resource:
            # Logic to execute synchronous task
            print(f"Executing sync task: {task_template.id.name}")
            input_value = inputs.literals["input_key"].scalar.primitive.integer
            result = input_value * 2
            return Resource(phase=TaskExecution.SUCCEEDED, outputs={"output_key": result})
    ```

2.  **Register Your Connector:**
    Register your connector with the `ConnectorRegistry` at application startup or module import.

    ```python
    from flytekit.extend.backend.base_connector import ConnectorRegistry

    # Assuming MyAsyncConnector and MySyncConnector are defined
    ConnectorRegistry.register(MyAsyncConnector())
    ConnectorRegistry.register(MySyncConnector())
    ```

3.  **Integrate with Flyte Tasks:**
    Define your Flyte task using the `task` decorator, specifying the `task_type` that corresponds to your registered connector. If your task is intended for local execution, inherit from the appropriate `ExecutorMixin`.

    ```python
    from flytekit import task, workflow
    from flytekit.extend.backend.base_connector import AsyncConnectorExecutorMixin, SyncConnectorExecutorMixin
    from flytekit.types.literal import LiteralType

    @task(task_type="my_async_task", task_type_version=1)
    class MyAsyncFlyteTask(AsyncConnectorExecutorMixin):
        def __init__(self):
            super().__init__(
                task_config={},
                task_type="my_async_task",
                task_type_version=1,
                input_type_models={"input_key": LiteralType.from_py_type(int)},
                output_type_models={"result": LiteralType.from_py_type(int)},
            )

    @task(task_type="my_sync_task", task_type_version=1)
    class MySyncFlyteTask(SyncConnectorExecutorMixin):
        def __init__(self):
            super().__init__(
                task_config={},
                task_type="my_sync_task",
                task_type_version=1,
                input_type_models={"input_key": LiteralType.from_py_type(int)},
                output_type_models={"output_key": LiteralType.from_py_type(int)},
            )

    @workflow
    def my_connector_workflow(input_val: int) -> int:
        async_result = MyAsyncFlyteTask(input_key=input_val)
        sync_result = MySyncFlyteTask(input_key=async_result.result)
        return sync_result.output_key

    # To run locally (after registering connectors):
    # if __name__ == "__main__":
    #     result = my_connector_workflow(input_val=10)
    #     print(f"Workflow result: {result}")
    ```

### Best Practices and Considerations

*   **Choosing Between Sync and Async:**
    *   Use `SyncConnectorBase` for operations that complete within seconds and return results immediately (e.g., fetching metadata from a fast API).
    *   Use `AsyncConnectorBase` for long-running operations that require polling for status (e.g., submitting a Spark job, running a large database query).
*   **Idempotency:** Ensure that the `delete()` method in `AsyncConnectorBase` is idempotent. Calling it multiple times should have the same effect as calling it once, preventing errors if cleanup is retried.
*   **Error Handling:** Implement robust error handling within your connector methods. Exceptions raised by the connector are propagated back to Flyte, allowing for proper task failure and retry mechanisms.
*   **Performance Monitoring:** The connector services automatically record metrics for `CreateTask`, `GetTask`, `DeleteTask`, and `ExecuteTaskSync` operations, providing insights into connector performance.
*   **Resource Management:** Connectors are responsible for managing the lifecycle of external resources. This includes creating, monitoring, and cleaning up resources.
*   **Output Handling:** Connectors can return outputs as `LiteralMap` or Python dictionaries. For large outputs, consider writing them to the `output_prefix` location provided by Flyte, allowing Flyte to manage the data transfer.
*   **Security:** Ensure that your connector implementations handle credentials and sensitive information securely when interacting with external systems.
<!--
key: summary_agent-based_task_extensions_(connectors)_d3a007da-29d0-4125-b42b-7dda9f7a0b81
type: summary_end

-->
<!--
code_unit: flytekit.extend.backend.base_connector.ConnectorRegistry
code_unit_type: class
help_text: ''
key: example_60b02c88-adb0-4e38-88cc-30f517289062
type: example

-->
<!--
code_unit: flytekit.extend.backend.base_connector.AsyncConnectorBase
code_unit_type: class
help_text: ''
key: example_91d72b80-e696-4813-9b3f-79da9cd8e888
type: example

-->
<!--
code_unit: flytekit.extend.backend.connector_service.AsyncConnectorService
code_unit_type: class
help_text: ''
key: example_8034a1da-72f1-4d7f-b6bc-7732d8a8526d
type: example

-->