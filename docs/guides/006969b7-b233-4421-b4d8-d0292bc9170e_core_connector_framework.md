
<!--
help_text: ''
key: summary_core_connector_framework_65baf3dd-7d35-4480-b543-21a0dc41ebb2
modules:
- flytekit.extend.backend.base_connector
- flytekit.extend.backend.connector_service
questions_to_answer: []
type: summary

-->
The Core Connector Framework provides a standardized and extensible mechanism for Flytekit to interact with external backend systems. It enables Flyte tasks to delegate their execution, monitoring, and resource management to specialized connectors, abstracting away the complexities of diverse external platforms.

### Core Concepts

The framework revolves around several key components that define how external systems integrate with Flyte.

#### Connectors: Asynchronous and Synchronous

Connectors are the primary interface for integrating with external services. They implement the specific logic required to interact with a given backend. The framework distinguishes between two types of connectors based on the nature of the external operation:

*   **Asynchronous Connectors:** These are designed for long-running operations where the task execution is initiated and then monitored for completion.
    *   Implementations derive from `AsyncConnectorBase`.
    *   They define methods for:
        *   `create`: Initiates the external task and returns a `ResourceMeta` object, which is a lightweight identifier for the running task (e.g., a job ID).
        *   `get`: Queries the status of the external task using the `ResourceMeta` and returns a `Resource` object, which includes the current phase, messages, logs, and outputs.
        *   `delete`: Terminates or cleans up the external task. This operation must be idempotent.
        *   `get_metrics`: Retrieves metrics associated with the external task.
        *   `get_logs`: Fetches log links for the external task.
    *   The `metadata_type` property specifies the concrete `ResourceMeta` type used by the connector.

*   **Synchronous Connectors:** These are suitable for quick operations that return results immediately within a single call.
    *   Implementations derive from `SyncConnectorBase`.
    *   They define a single method:
        *   `do`: Executes the external task and returns a `Resource` object directly, containing the final status and outputs.

Both `AsyncConnectorBase` and `SyncConnectorBase` inherit from `ConnectorBase`, which associates each connector with a `TaskCategory`.

#### Task Categories

A `TaskCategory` uniquely identifies a type of task that a connector supports. It consists of a `name` (string) and an optional `version` (integer). This allows for versioning of task types and ensures that each specific task type (name + version) maps to exactly one connector.

#### Resource Management

The framework defines two core data structures for managing external task state:

*   **`ResourceMeta`**: This class represents the minimal metadata required to identify and interact with an external task. For example, it could encapsulate a job ID, a resource URI, or any other unique identifier returned by the external system upon task creation. `ResourceMeta` objects can be encoded to and decoded from bytes for persistence and transmission.
*   **`Resource`**: This class encapsulates the comprehensive status and outputs of an external task. It includes:
    *   `phase`: The current execution phase of the task (e.g., `RUNNING`, `SUCCEEDED`, `FAILED`).
    *   `message`: An optional message providing more details about the task's status.
    *   `log_links`: A list of links to external logs.
    *   `outputs`: The task's outputs, which can be a Flyte `LiteralMap` or a Python dictionary that the framework converts to a `LiteralMap`.
    *   `custom_info`: Any additional custom information from the external system.

#### Connector Registry

The `ConnectorRegistry` is a static component that maintains a mapping of `TaskCategory` to connector instances. It serves as the central lookup mechanism for the framework.

*   `register(connector)`: Adds a connector to the registry. A `TaskCategory` can only have one connector registered.
*   `get_connector(task_type_name, task_type_version)`: Retrieves the appropriate connector instance based on the task category.
*   `list_connectors()`: Returns metadata (`Agent` objects) for all registered connectors.
*   `get_connector_metadata(name)`: Retrieves metadata for a specific connector by its name.

### Architecture and Flow

The Core Connector Framework acts as an intermediary between the Flyte engine (Propeller) and external backend systems.

1.  **Propeller Interaction:** The Flyte Propeller communicates with the Connector Services via gRPC.
    *   For asynchronous tasks, Propeller sends `CreateTaskRequest`, `GetTaskRequest`, `DeleteTaskRequest`, `GetTaskMetricsRequest`, and `GetTaskLogsRequest` to the `AsyncConnectorService`.
    *   For synchronous tasks, Propeller sends `ExecuteTaskSyncRequest` to the `SyncConnectorService`.
    *   Propeller can also query connector metadata via the `ConnectorMetadataService`.

2.  **Connector Service Role:** The `AsyncConnectorService` and `SyncConnectorService` receive requests from Propeller. They use the `ConnectorRegistry` to find the correct connector based on the task type specified in the request.
    *   `AsyncConnectorService` invokes the `create`, `get`, `delete`, `get_metrics`, and `get_logs` methods on the retrieved `AsyncConnectorBase` instance.
    *   `SyncConnectorService` invokes the `do` method on the retrieved `SyncConnectorBase` instance.
    *   These services handle the serialization and deserialization of `ResourceMeta` and `Resource` objects to and from their respective byte representations or Flyte IDL formats.

3.  **Connector Execution:** The chosen connector then executes the requested operation against the external backend. For example, an `AsyncConnectorBase` might make an API call to create a job, then poll its status, and finally delete it. A `SyncConnectorBase` might execute a quick query and return the result.

4.  **Local Execution:** For local development and testing, tasks can leverage `AsyncConnectorExecutorMixin` or `SyncConnectorExecutorMixin`. These mixins simulate the Propeller-connector interaction locally, allowing developers to run connector-backed tasks without deploying to a Flyte cluster.
    *   `AsyncConnectorExecutorMixin` manages the `create` and `get` loop, including handling signals for task cleanup. It also handles reading outputs from a remote location if the connector doesn't return them directly.
    *   `SyncConnectorExecutorMixin` directly calls the `do` method of the synchronous connector.

### Implementing Custom Connectors

To integrate a new external system, implement a custom connector:

1.  **Choose Connector Type:** Decide if the external operations are primarily asynchronous (long-running) or synchronous (quick). This determines whether to inherit from `AsyncConnectorBase` or `SyncConnectorBase`.

2.  **Define `ResourceMeta`:** For `AsyncConnectorBase`, define a `ResourceMeta` subclass (often a dataclass) that holds the necessary information to track the external task. This class must implement `encode()` and `decode()` methods.

3.  **Implement Connector Methods:**
    *   **For `AsyncConnectorBase`:**
        *   Implement `__init__` to set the `task_type_name`, `task_type_version`, and `metadata_type`.
        *   Implement `create(task_template, output_prefix, inputs, task_execution_metadata)`: This method should initiate the external task and return an instance of your custom `ResourceMeta`.
        *   Implement `get(resource_meta)`: This method should query the external system using the provided `resource_meta` and return a `Resource` object reflecting the current status.
        *   Implement `delete(resource_meta)`: This method should clean up the external task. Ensure it is idempotent.
        *   Optionally implement `get_metrics(resource_meta)` and `get_logs(resource_meta)`.
    *   **For `SyncConnectorBase`:**
        *   Implement `__init__` to set the `task_type_name` and `task_type_version`.
        *   Implement `do(task_template, output_prefix, inputs)`: This method should execute the external task and return a `Resource` object with the final status and outputs.

4.  **Register the Connector:** After defining the connector, register it with the `ConnectorRegistry`.

    ```python
    from flytekit.extend.backend.base_connector import ConnectorRegistry, AsyncConnectorBase, ResourceMeta, Resource, TaskCategory
    from flytekit.models.core.tasks import TaskTemplate
    from flytekit.models.literals import LiteralMap
    from flytekit.models.core.execution import TaskExecution
    from dataclasses import dataclass
    from typing import Optional, Dict, Any

    @dataclass
    class MyJobMeta(ResourceMeta):
        job_id: str

    class MyAsyncConnector(AsyncConnectorBase):
        name = "MyCustomAsyncConnector"

        def __init__(self, task_type_name: str, task_type_version: int = 0):
            super().__init__(task_type_name=task_type_name, task_type_version=task_type_version, metadata_type=MyJobMeta)

        def create(
            self,
            task_template: TaskTemplate,
            output_prefix: str,
            inputs: Optional[LiteralMap],
            task_execution_metadata: Optional[Any],
            **kwargs,
        ) -> MyJobMeta:
            # Logic to submit job to external system
            print(f"Creating external job for task type: {task_template.type}")
            print(f"Inputs: {inputs.to_json()}")
            # Simulate job submission and return a job ID
            job_id = "my-external-job-123"
            return MyJobMeta(job_id=job_id)

        def get(self, resource_meta: MyJobMeta, **kwargs) -> Resource:
            # Logic to check job status in external system
            print(f"Getting status for job ID: {resource_meta.job_id}")
            # Simulate job completion
            return Resource(phase=TaskExecution.Phase.SUCCEEDED, message="Job completed successfully")

        def delete(self, resource_meta: MyJobMeta, **kwargs):
            # Logic to delete job in external system
            print(f"Deleting job ID: {resource_meta.job_id}")

    # Register the connector
    ConnectorRegistry.register(MyAsyncConnector(task_type_name="my_custom_task_type", task_type_version=1))
    ```

### Local Development and Testing

To test connector-backed tasks locally, inherit from the appropriate executor mixin:

*   For tasks using `AsyncConnectorBase`, inherit from `AsyncConnectorExecutorMixin`.
*   For tasks using `SyncConnectorBase`, inherit from `SyncConnectorExecutorMixin`.

These mixins provide an `execute` method that simulates the remote execution environment, allowing you to run your task locally and interact with your connector implementation.

```python
from flytekit import task, workflow
from flytekit.extend.backend.base_connector import AsyncConnectorExecutorMixin
from flytekit.types.literal import LiteralType
from flytekit.models.core.types import TaskTemplate
from flytekit.models.interface import TypedInterface
from flytekit.models.literals import LiteralMap, Scalar, Primitive, StringValue
from flytekit.models.types import SimpleType

# Assume MyAsyncConnector is defined and registered as above

@task(task_type="my_custom_task_type", task_type_version=1)
class MyConnectorTask(AsyncConnectorExecutorMixin):
    def __init__(self):
        super().__init__(
            task_type="my_custom_task_type",
            task_type_version=1,
            # Define the task's interface for local execution
            interface=TypedInterface(
                inputs={"input_str": LiteralType(simple=SimpleType.STRING)},
                outputs={"output_str": LiteralType(simple=SimpleType.STRING)},
            ),
        )

    def _get_full_task_template(self) -> TaskTemplate:
        # This method is usually handled by Flytekit's serialization,
        # but for a minimal local example, we might need to mock it.
        # In a real scenario, this task would be decorated with @task
        # and Flytekit would generate the template.
        return TaskTemplate(
            id=None,
            type="my_custom_task_type",
            task_type_version=1,
            interface=self.interface,
            metadata=None,
            custom={},
        )

    def execute(self, input_str: str) -> str:
        # The actual execution logic for the task, which will internally
        # call the connector's methods via the mixin.
        # The mixin's execute method will handle the interaction with the connector.
        # We need to call the mixin's execute method with the inputs as a dictionary.
        outputs_literal_map = super().execute(input_str=input_str)
        # Extract the output from the LiteralMap
        return outputs_literal_map.literals["output_str"].scalar.primitive.string_value

@workflow
def my_connector_workflow(input_val: str) -> str:
    return MyConnectorTask()(input_str=input_val)

if __name__ == "__main__":
    # This will run the task locally using MyAsyncConnector
    result = my_connector_workflow(input_val="hello world")
    print(f"Workflow result: {result}")
```

### Key Considerations

*   **Idempotency:** The `delete` method in `AsyncConnectorBase` must be idempotent. Calling it multiple times with the same `ResourceMeta` should have the same effect as calling it once, without raising errors if the resource is already gone.
*   **Timeout Handling:** When running tasks locally using the executor mixins, any timeouts specified in the `TaskTemplate.metadata.timeout` are ignored. Timeouts are enforced by the Flyte engine in a deployed environment.
*   **Output Handling:** For `AsyncConnectorBase`, if the external system does not return outputs directly in the `Resource` object, the connector should ensure that outputs are written to the `output_prefix` location provided during the `create` call. Flytekit's local execution mixin will then read these outputs from the remote file. Synchronous connectors can return outputs directly within the `Resource` object.
*   **One Connector Per Task Category:** The `ConnectorRegistry` enforces that only one connector can be registered for a given `TaskCategory` (task type name and version). This ensures deterministic behavior.
*   **Error Handling:** Connectors should implement robust error handling for interactions with external systems. Exceptions raised within connector methods will be propagated by the Connector Services.

### Best Practices

*   **Clear Naming:** Use descriptive names for your connectors and task types to clearly indicate their purpose and the external system they integrate with.
*   **Granular `ResourceMeta`:** Design your `ResourceMeta` to contain only the essential information needed to track and manage the external task. Avoid including large or unnecessary data.
*   **Comprehensive Logging:** Implement detailed logging within your connector methods to aid in debugging and monitoring, especially for long-running asynchronous tasks.
*   **Resource Cleanup:** Ensure your `delete` method correctly cleans up all associated resources in the external system to prevent resource leaks and unnecessary costs.
*   **Version Control:** Utilize the `task_type_version` in `TaskCategory` to manage changes and updates to your connector implementations without breaking existing workflows.
<!--
key: summary_core_connector_framework_65baf3dd-7d35-4480-b543-21a0dc41ebb2
type: summary_end

-->
<!--
code_unit: flytekit.extend.backend.base_connector
code_unit_type: class
help_text: ''
key: example_1062c1ae-f457-46f5-985d-441b0ebf68e5
type: example

-->
<!--
code_unit: flytekit.extend.backend.connector_service
code_unit_type: class
help_text: ''
key: example_e398441e-2e5c-40f4-a482-98c9a1af81fa
type: example

-->