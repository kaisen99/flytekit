
<!--
help_text: ''
key: summary_airflow_integration_541a95f9-5a5b-4ba4-bb6c-cb439d2063a6
modules:
- flytekitplugins.airflow.connector
- flytekitplugins.airflow.task
questions_to_answer: []
type: summary

-->
Airflow Integration enables the execution and management of Apache Airflow tasks directly within Flyte workflows. This integration leverages Flyte's asynchronous execution model and containerization capabilities to provide robust and scalable orchestration for Airflow components.

## Core Components

The integration relies on several key components that facilitate the definition, execution, and state management of Airflow tasks within Flyte.

### Airflow Task Definition

The `AirflowObj` class serves as a serialized representation of an Airflow task. It captures the essential details required to instantiate an Airflow operator, sensor, or hook at runtime.

*   **`module`**: The Python module path where the Airflow task class is defined (e.g., `airflow.sensors.filesystem`).
*   **`name`**: The class name of the Airflow task (e.g., `FileSensor`).
*   **`parameters`**: A dictionary of keyword arguments used to initialize the Airflow task instance (e.g., `{"task_id": "my_sensor", "filepath": "/path/to/file"}`).

This object is crucial for passing Airflow task configurations between Flyte components.

### Flyte Task Wrappers

Two primary Flyte task types are provided to wrap Airflow tasks, each catering to different execution models.

#### AirflowTask

The `AirflowTask` is a Python task designed for Airflow components that can be executed asynchronously, primarily leveraging Airflow's deferrable operators and sensors. It uses an `AirflowObj` to define the underlying Airflow task.

When an `AirflowTask` is executed, its configuration (the `AirflowObj`) is serialized and passed to the `AirflowConnector` for remote execution and status polling. This allows Flyte to manage the Airflow task's lifecycle without blocking the Flyte worker.

#### AirflowContainerTask

The `AirflowContainerTask` is a specialized Python container task for Airflow operators that are *not* deferrable. These operators typically block until completion and do not provide an asynchronous mechanism for status updates.

For such tasks, the `AirflowContainerTask` ensures they are executed directly within a Flyte pod's container. This means the Flyte worker will wait for the Airflow operator to complete before proceeding. Examples include `BeamRunJavaPipelineOperator` or `BeamRunPythonPipelineOperator`, which manage external long-running processes without built-in deferral.

### Airflow Connector

The `AirflowConnector` is the central component responsible for orchestrating the execution of Airflow tasks that are defined using `AirflowTask`. It is registered within Flyte's Connector Registry and handles the asynchronous lifecycle of Airflow operators, sensors, and hooks.

*   **Task Creation (`create` method)**:
    *   For **deferrable Airflow operators**: The connector attempts to execute the operator. If the operator is deferrable, it raises a `TaskDeferred` exception. The connector captures the associated Airflow trigger and its callback method, storing this information in `AirflowMetadata`.
    *   For **Airflow sensors and hooks**: These are typically designed to complete quickly or are polled directly. The `create` method might execute them if they are not deferrable and complete immediately, or prepare them for polling in the `get` method.

*   **Status Polling (`get` method)**:
    *   For **Airflow sensors**: The connector repeatedly calls the sensor's `poke()` method. The task remains in a `RUNNING` state until `poke()` returns `True`, at which point it transitions to `SUCCEEDED`.
    *   For **deferrable Airflow operators**: The connector interacts with the captured Airflow trigger. It awaits events from the trigger and, upon receiving an event, invokes the specified callback method on the original Airflow operator. This callback determines the final status (succeeded or failed) based on the trigger event payload. If no event is received within a timeout, the task remains `RUNNING`.
    *   For **non-deferrable Airflow operators or hooks that completed in `create`**: If no trigger information is present in `AirflowMetadata`, it implies the task completed synchronously during the `create` phase. In this scenario, the connector immediately returns a `SUCCEEDED` status.

*   **Task Deletion (`delete` method)**: Currently, this method is a no-op, indicating that the connector does not perform explicit cleanup actions for Airflow tasks.

### Airflow Metadata

The `AirflowMetadata` class is used by the `AirflowConnector` to persist the state and configuration of an Airflow task across different execution phases (e.g., between the `create` and `get` calls). This metadata is serialized and returned to the FlytePropeller.

It stores:
*   `airflow_operator`: The `AirflowObj` representing the original Airflow task.
*   `airflow_trigger`: If the task is a deferrable operator, this stores the `AirflowObj` representing the Airflow trigger.
*   `airflow_trigger_callback`: The name of the method on the Airflow operator that should be invoked when the trigger fires.
*   `job_id`: An optional identifier for the Airflow job.

### Airflow Task Resolver

The `AirflowTaskResolver` is a Flytekit component that dynamically loads and instantiates Airflow tasks (or triggers) within a container at runtime. This resolver is primarily used by `AirflowContainerTask` to reconstruct the Airflow operator from its serialized `AirflowObj` configuration before execution.

## Execution Models and Task Types

The integration distinguishes between different types of Airflow components and their execution patterns within Flyte.

### Airflow Operators

*   **Deferrable Operators**: These operators are designed to yield control after submitting a job and rely on a separate trigger to monitor its completion. When an `AirflowTask` wraps a deferrable operator, the `AirflowConnector` initiates the task and captures the trigger. Flyte then asynchronously polls this trigger via the `get` method until the underlying Airflow job completes or fails. This model is efficient for long-running external processes.
*   **Non-Deferrable Operators**: These operators block until their execution is complete. Examples include operators that perform local computations or interact with systems that do not offer asynchronous polling. When an `AirflowTask` wraps a non-deferrable operator, it is automatically converted to an `AirflowContainerTask` and executed directly within a Flyte pod. The Flyte worker will wait for the operator to finish.

### Airflow Sensors

Sensors are used to wait for a certain condition to be met (e.g., a file to appear, a database record to exist). When an `AirflowTask` wraps an Airflow sensor, the `AirflowConnector`'s `get` method repeatedly calls the sensor's `poke()` method. The task remains in a `RUNNING` state until `poke()` returns `True`, indicating the condition is met, at which point the task succeeds.

### Airflow Hooks

Hooks provide a high-level interface to external platforms. They are typically used for quick, synchronous interactions (e.g., sending a Slack message). When an `AirflowTask` wraps an Airflow hook, it is generally treated like a non-deferrable operator. The hook's action is performed during the `create` phase of the `AirflowConnector`. If successful, the task immediately transitions to `SUCCEEDED` in the subsequent `get` call. If an error occurs during the hook's execution, it will fail at the creation step.

## Defining Airflow Tasks in Flyte

To integrate an Airflow task into a Flyte workflow, you define it using either `AirflowTask` or `AirflowContainerTask`.

### Example: Using a Deferrable Airflow Operator

```python
from flytekitplugins.airflow.task import AirflowObj, AirflowTask
from airflow.providers.cncf.kubernetes.operators.kubernetes_pod import KubernetesPodOperator

# Define the Airflow KubernetesPodOperator
k8s_pod_operator_config = AirflowObj(
    module="airflow.providers.cncf.kubernetes.operators.kubernetes_pod",
    name="KubernetesPodOperator",
    parameters={
        "task_id": "my_k8s_pod_task",
        "namespace": "default",
        "image": "ubuntu:latest",
        "cmds": ["bash", "-cx"],
        "arguments": ["echo hello world"],
        "do_xcom_push": False,
        "is_delete_operator_pod": True,
        "deferrable": True, # Important for asynchronous execution
    },
)

# Wrap it in an AirflowTask for Flyte
k8s_pod_flyte_task = AirflowTask(
    name="run_k8s_pod_async",
    task_config=k8s_pod_operator_config,
)

# You can then use k8s_pod_flyte_task in your Flyte workflow
# @workflow
# def my_airflow_workflow():
#     k8s_pod_flyte_task()
```

### Example: Using an Airflow Sensor

```python
from flytekitplugins.airflow.task import AirflowObj, AirflowTask
from airflow.sensors.filesystem import FileSensor

# Define the Airflow FileSensor
file_sensor_config = AirflowObj(
    module="airflow.sensors.filesystem",
    name="FileSensor",
    parameters={
        "task_id": "wait_for_file",
        "filepath": "/tmp/my_data.csv",
        "poke_interval": 5, # How often to check
    },
)

# Wrap it in an AirflowTask for Flyte
file_sensor_flyte_task = AirflowTask(
    name="wait_for_data_file",
    task_config=file_sensor_config,
)

# @workflow
# def data_processing_workflow():
#     file_sensor_flyte_task()
#     # ... proceed with data processing once file exists
```

### Example: Using a Non-Deferrable Airflow Operator

```python
from flytekitplugins.airflow.task import AirflowObj, AirflowContainerTask
# Assuming a hypothetical non-deferrable operator
# from airflow.operators.dummy import DummyOperator # DummyOperator is not deferrable

# Define a non-deferrable Airflow operator
# For demonstration, let's use DummyOperator as an example of a non-deferrable task
dummy_operator_config = AirflowObj(
    module="airflow.operators.dummy",
    name="DummyOperator",
    parameters={
        "task_id": "my_dummy_task",
    },
)

# Wrap it in an AirflowContainerTask for Flyte
dummy_flyte_task = AirflowContainerTask(
    name="run_dummy_operator_in_container",
    task_config=dummy_operator_config,
)

# @workflow
# def simple_workflow():
#     dummy_flyte_task()
```

## Error Handling and Limitations

*   **AirflowException**: If an Airflow task or trigger encounters an error, the `AirflowConnector` captures `AirflowException` and transitions the Flyte task to a `FAILED` state, including the error message.
*   **Timeout Handling**: For deferrable operators, the `AirflowConnector` includes a timeout when awaiting events from the Airflow trigger. If no event is received within this period, the task remains in a `RUNNING` state, indicating that the underlying Airflow job is still in progress.
*   **Unsupported Task Types**: The integration currently supports Airflow operators, sensors, and hooks. Other Airflow components might not be directly compatible.
*   **No Explicit Deletion**: The `delete` method in `AirflowConnector` is a no-op. This means Flyte does not actively terminate or clean up Airflow tasks once they are completed or failed. Cleanup of the underlying Airflow resources (e.g., Kubernetes pods launched by `KubernetesPodOperator`) is managed by Airflow itself or the external system.

## Best Practices

*   **Choose the Right Wrapper**: Carefully determine if your Airflow operator is deferrable. Use `AirflowTask` for deferrable operators, sensors, and hooks that complete quickly. Use `AirflowContainerTask` for non-deferrable operators that block execution.
*   **Parameterization**: Leverage Flyte's input/output capabilities to pass dynamic parameters to your Airflow tasks. This allows for flexible and reusable Airflow task definitions.
*   **Observability**: Utilize Flyte's logging and monitoring features to observe the execution of your Airflow tasks. The `AirflowConnector` provides log links to help debug issues within the Airflow context.
*   **Dependency Management**: Ensure that the Flyte execution environment (especially for `AirflowContainerTask`) has all necessary Airflow providers and dependencies installed to run your specific Airflow operators, sensors, or hooks.
<!--
key: summary_airflow_integration_541a95f9-5a5b-4ba4-bb6c-cb439d2063a6
type: summary_end

-->
<!--
code_unit: flytekitplugins.airflow.examples.basic_airflow_task
code_unit_type: class
help_text: ''
key: example_4220649f-9a85-46bc-bce9-0fe53988474d
type: example

-->