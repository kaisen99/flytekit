
<!--
help_text: ''
key: summary_system_exceptions_fca0ee3a-49e5-46d2-b62f-bc0bc3c3dec3
modules:
- flytekit.exceptions.system
questions_to_answer: []
type: summary

-->
# System Exceptions

System Exceptions in Flytekit indicate issues originating from the Flyte platform or its underlying infrastructure, rather than from user code logic. These exceptions provide critical insights into the operational health and capabilities of the Flyte environment, enabling developers to diagnose and respond to platform-level failures.

## Core Exception Hierarchy

The foundation for system-related errors is built upon a clear hierarchy, distinguishing between potentially transient issues and critical, unrecoverable failures.

### FlyteSystemException

This is the foundational class for most system-related errors. It inherits from `FlyteRecoverableException`, implying that issues represented by its direct instances or many of its subclasses might be transient or resolvable through retries. The default error code for this base class is `SYSTEM:Unknown`.

### FlyteNonRecoverableSystemException

This exception signifies a critical system failure that cannot be recovered from by retrying the operation. It wraps an underlying exception (`exc_value`) that caused the non-recoverable state. When this exception is raised, the current execution is expected to fail immediately, as further attempts are unlikely to succeed. Although it originates from a system issue, its error code is `USER:NonRecoverableSystemError`, indicating that it is an error reported to the user, signifying a definitive failure of their workflow due to a system problem.

Access the original exception that triggered the non-recoverable state using the `value` property:

```python
try:
    # Some operation that might raise a non-recoverable system exception
    pass
except FlyteNonRecoverableSystemException as e:
    print(f"Non-recoverable system error: {e}")
    print(f"Original exception: {e.value}")
    # Log the error, trigger alerts, etc.
```

## Specific System Exception Types

Flytekit defines several specific system exception types, each corresponding to a distinct category of platform-level issues.

### Resource and Service Availability

These exceptions indicate problems with the availability or configuration of Flyte components or external resources.

*   **FlyteAgentNotFound**: Raised when the system attempts to use an agent that is not configured or available within the Flyte environment. The error code is `SYSTEM:AgentNotFound`.
*   **FlyteConnectorNotFound**: Raised when the system cannot locate a required connector plugin, which might be necessary for interacting with external services or data sources. The error code is `SYSTEM:ConnectorNotFound`.
*   **FlyteSystemUnavailableException**: Indicates that the Flyte cluster or a critical system component is currently unreachable or unresponsive. Operations should typically be retried after a delay. The error code is `SYSTEM:Unavailable`.

    ```python
    try:
        # Attempt to connect to Flyte
        pass
    except FlyteSystemUnavailableException as e:
        print(f"Flyte system is unavailable: {e}")
        # Implement retry logic or alert operations
    ```

### Data Transfer Operations

These exceptions occur during the system's attempts to move data, such as task inputs or outputs.

*   **FlyteDownloadDataException**: Occurs when the system encounters an error while attempting to download data, such as inputs for a task from a configured data store. The error code is `SYSTEM:DownloadDataError`.
*   **FlyteUploadDataException**: Occurs when the system encounters an error while attempting to upload data, such as outputs from a task to a configured data store. The error code is `SYSTEM:UploadDataError`.

### Code Loading and Execution

These exceptions relate to the system's ability to load, discover, or execute user-defined code.

*   **FlyteEntrypointNotLoadable**: Raised when the system fails to load a task's module or locate a specific task within a module. This often points to issues with the task definition, module path, or dependencies within the execution environment. It provides detailed messages including the module and task name. The error code is `SYSTEM:UnloadableCode`.

    ```python
    # Example of how the system might construct the error message
    # (not directly raised by user code)
    error_message_module = FlyteEntrypointNotLoadable._create_verbose_message(
        task_module="my_workflow_module",
        additional_msg="Module not found"
    )
    # Output: "Entrypoint is not loadable! Could not load the module: 'my_workflow_module' due to error: Module not found."

    error_message_task = FlyteEntrypointNotLoadable._create_verbose_message(
        task_module="my_workflow_module",
        task_name="my_task",
        additional_msg="Task definition missing"
    )
    # Output: "Entrypoint is not loadable! Could not find the task: 'my_task' in 'my_workflow_module' due to error: Task definition missing."
    ```

*   **FlyteNotImplementedException**: Indicates that a requested feature or capability is not yet implemented within the Flyte system. The error code is `SYSTEM:NotImplemented`.

### Internal System Assertions

*   **FlyteSystemAssertion**: Raised when an internal system invariant or assumption is violated. This typically indicates a bug within the Flyte system itself. The error code is `SYSTEM:AssertionError`.

## Handling System Exceptions

When integrating with Flytekit, applications should differentiate between system exceptions and user-defined exceptions.

*   For `FlyteSystemException` and its recoverable subclasses, consider implementing retry mechanisms with exponential backoff, as the underlying issue might be transient.
*   For `FlyteNonRecoverableSystemException`, immediate failure and alerting are appropriate. Inspect the `value` property to access the original exception that triggered the non-recoverable state for detailed debugging.
*   Log all system exceptions with sufficient context to aid in debugging and platform monitoring.

## Best Practices

*   Avoid catching `FlyteSystemException` broadly unless specific recovery logic is in place. Prefer catching more specific subclasses when possible to handle distinct failure modes.
*   When developing custom Flyte plugins or extensions, raise appropriate `FlyteSystemException` subclasses to clearly communicate system-level issues to the Flyte platform.
*   Monitor for `SYSTEM:` error codes in logs and metrics to identify recurring infrastructure or platform issues, which can indicate underlying problems with the Flyte deployment or its dependencies.
<!--
key: summary_system_exceptions_fca0ee3a-49e5-46d2-b62f-bc0bc3c3dec3
type: summary_end

-->
<!--
code_unit: flytekit.remote
code_unit_type: class
help_text: ''
key: example_60ee59d5-99f4-43c1-b001-693786d086f9
type: example

-->
<!--
code_unit: flytekit.testing
code_unit_type: class
help_text: ''
key: example_f71ef10e-b7e5-458b-89ef-61ab2590563c
type: example

-->