
<!--
help_text: ''
key: summary_user-defined_exceptions_8d3743a1-17d7-493d-b217-40db175de860
modules:
- flytekit.exceptions.user
- flytekit.exceptions.eager
questions_to_answer: []
type: summary

-->
## User-Defined Exceptions

Flytekit provides a comprehensive set of user-defined exceptions to handle various error conditions that can arise during the compilation, execution, and interaction with Flyte entities. These exceptions extend standard Python exceptions, offering more specific context and enabling robust error handling within your Flyte tasks and workflows.

### Core User Exception Types

All user-defined exceptions in Flytekit inherit from `FlyteUserException`, which itself derives from an internal `_FlyteException` base. This hierarchy ensures consistent error reporting and allows for broad exception handling.

*   **`FlyteUserException`**: This is the base class for all user-facing exceptions. It provides a common `_ERROR_CODE` attribute, which helps categorize the error. When a general, uncategorized user error occurs, this exception can be raised.

*   **`FlyteUserRuntimeException`**: This exception wraps any arbitrary exception raised directly from user code within a task or workflow. It captures the original exception value and provides a timestamp. This is particularly useful for Flyte to report user-generated errors without losing the original context.

    ```python
    from flytekit.exceptions.user import FlyteUserRuntimeException

    def my_task_logic():
        try:
            # Some user code that might raise an exception
            result = 1 / 0
        except Exception as e:
            raise FlyteUserRuntimeException(e)

    # In a Flyte task, this would typically be caught by the Flyte engine
    # and reported as a FlyteUserRuntimeException.
    ```

*   **`FlyteRecoverableException`**: This specialized exception indicates an error that is transient and can potentially be resolved by retrying the operation. It inherits from `FlyteUserException` and an internal `_Recoverable` mixin, signaling to the Flyte platform that a retry mechanism might be appropriate.

    ```python
    from flytekit.exceptions.user import FlyteRecoverableException

    def fetch_data_from_api():
        if network_is_down():
            raise FlyteRecoverableException("Network temporarily unavailable, please retry.")
        # ... fetch data
    ```

### Compilation and Definition Exceptions

These exceptions are raised when issues are detected during the compilation or definition phase of Flyte tasks and workflows, often related to incorrect function signatures or missing annotations.

*   **`FlyteCompilationException`**: The base class for errors encountered during the compilation of Flyte entities. It stores the function (`fn`) and an optional parameter name (`param_name`) where the compilation error occurred.

*   **`FlyteMissingReturnValueException`**: Raised when a Flyte task or workflow function is expected to return a value but lacks a return statement.

    ```python
    from flytekit import task
    from flytekit.exceptions.user import FlyteMissingReturnValueException

    @task
    def my_task_without_return(x: int) -> int:
        # This will raise FlyteMissingReturnValueException during compilation
        # because it declares a return type but has no return statement.
        print(f"Processing {x}")

    # To fix:
    @task
    def my_task_with_return(x: int) -> int:
        return x + 1
    ```

*   **`FlyteMissingTypeException`**: Raised when an input parameter to a Flyte task or workflow function is missing a type annotation. Flyte requires type hints for all inputs and outputs to correctly infer data types for serialization and deserialization.

    ```python
    from flytekit import task
    from flytekit.exceptions.user import FlyteMissingTypeException

    @task
    def my_task_missing_type(x) -> int: # 'x' is missing a type annotation
        # This will raise FlyteMissingTypeException during compilation
        return x + 1

    # To fix:
    @task
    def my_task_with_type(x: int) -> int:
        return x + 1
    ```

### Assertion and Validation Exceptions

These exceptions indicate that an assertion failed or a validation rule was violated, often due to incorrect assumptions or invalid states.

*   **`FlyteAssertion`**: The base class for assertion-related failures. It extends `FlyteUserException` and Python's `AssertionError`.

*   **`FlyteEntityNotExistException`**: Indicates that a required Flyte entity (e.g., a task, workflow, or launch plan) does not exist.

*   **`FlyteDisapprovalException`**: Raised when a result or state is not approved, typically in scenarios requiring explicit confirmation or validation.

*   **`FlyteEntityAlreadyExistsException`**: Indicates an attempt to create a Flyte entity that already exists, violating a uniqueness constraint.

*   **`FlyteFailureNodeInputMismatchException`**: Raised when the inputs defined for a failure node within a workflow do not align with the expected inputs of the overall workflow. This ensures that error handling branches receive the correct context.

    ```python
    # Example scenario (conceptual, actual implementation depends on workflow definition)
    # If a workflow 'main_wf' expects input 'x' and its failure node 'error_handler'
    # is configured to receive 'y', this exception would be raised.
    ```

*   **`FlyteTimeout`**: Signifies that an operation or execution exceeded its allotted time limit.

*   **`FlyteAuthenticationException`**: Raised when an authentication failure occurs, such as invalid credentials or insufficient permissions.

*   **`FlytePromiseAttributeResolveException`**: Occurs when attempting to resolve an attribute of a `Promise` object (a deferred output from a task or node) that cannot be resolved at the current stage of execution.

*   **`FlyteValidationException`**: A general exception for when data or configuration fails a validation check.

### Value and Type Exceptions

These exceptions are specific to issues with data values or type mismatches.

*   **`FlyteValueException`**: The base class for errors related to incorrect or invalid values. It extends `FlyteUserException` and Python's `ValueError`. It provides a verbose message format including the received value and an error message.

*   **`FlyteDataNotFoundException`**: Raised when expected data, typically at a specified path, cannot be found.

    ```python
    from flytekit.exceptions.user import FlyteDataNotFoundException

    def load_config(path: str):
        if not file_exists(path):
            raise FlyteDataNotFoundException(path)
        # ... load config
    ```

*   **`FlyteEntityNotFoundException`**: A specialized `FlyteValueException` indicating that a specific task or workflow entity was not found within a given module.

    ```python
    from flytekit.exceptions.user import FlyteEntityNotFoundException

    # When attempting to load a task/workflow dynamically
    # If 'my_task' is not found in 'my_module'
    raise FlyteEntityNotFoundException(module_name="my_module", entity_name="my_task")
    ```

*   **`FlyteTypeException`**: Raised when a type mismatch occurs. It extends `FlyteUserException` and Python's `TypeError`. It provides detailed messages, including the received type, expected type(s), and optionally the received value.

    ```python
    from flytekit.exceptions.user import FlyteTypeException

    def process_number(value: int):
        if not isinstance(value, int):
            raise FlyteTypeException(received_type=type(value), expected_type=int, received_value=value)
        # ... process
    ```

### API Input Exceptions

*   **`FlyteInvalidInputException`**: Raised when an API call receives an invalid or malformed request. It stores the problematic request object for debugging.

### Eager Workflow Exceptions

*   **`EagerException`**: This exception is specific to `eager` workflows. When a task or sub-workflow within an `eager` workflow raises an exception, Flytekit wraps that exception in an `EagerException`. This allows developers to catch errors originating from nested Flyte calls within an `eager` context.

    ```python
    from flytekit import task, eager
    from flytekit.exceptions.eager import EagerException

    @task
    def divide(numerator: int, denominator: int) -> float:
        if denominator == 0:
            raise ValueError("Cannot divide by zero")
        return numerator / denominator

    @eager
    async def eager_division_workflow(a: int, b: int) -> float:
        try:
            result = await divide(numerator=a, denominator=b)
        except EagerException as e:
            # Catches the ValueError wrapped by EagerException
            print(f"Caught an error in eager workflow: {e}")
            raise # Re-raise or handle as needed
        return result

    # Example usage:
    # await eager_division_workflow(10, 0) # This will raise EagerException
    ```

### Handling User-Defined Exceptions

When developing Flyte tasks and workflows, you can catch these specific exceptions to implement fine-grained error handling logic. This allows you to differentiate between various failure modes and respond appropriately, such as retrying, logging, or propagating specific error messages.

### Best Practices

*   **Prefer specific exceptions**: When an error condition aligns with an existing `FlyteUserException` subclass (e.g., `FlyteTypeException` for type mismatches, `FlyteDataNotFoundException` for missing files), use that specific exception. This provides clearer context to the Flyte platform and to other developers.
*   **Wrap external exceptions**: If your task interacts with external libraries or APIs that raise their own exceptions, consider wrapping them in a `FlyteUserRuntimeException` or a more specific `FlyteUserException` if applicable. This ensures that Flyte's error reporting mechanisms can properly capture and display the issue.
*   **Use `FlyteRecoverableException` for transient errors**: If an error is temporary (e.g., network glitch, temporary service unavailability), raising `FlyteRecoverableException` signals to Flyte that a retry might succeed, leveraging Flyte's built-in retry mechanisms.
*   **Catch `EagerException` in `eager` workflows**: When composing `eager` workflows, explicitly catch `EagerException` to handle errors originating from nested task or workflow calls. This allows for graceful degradation or custom error handling within the `eager` context.
<!--
key: summary_user-defined_exceptions_8d3743a1-17d7-493d-b217-40db175de860
type: summary_end

-->
<!--
code_unit: flytekit.exceptions.eager
code_unit_type: class
help_text: ''
key: example_51c48148-19d3-4604-92fa-2b4bc7fac192
type: example

-->
<!--
code_unit: flytekit.core.task
code_unit_type: class
help_text: ''
key: example_be8133a8-92dd-4186-a707-f624acda420d
type: example

-->
<!--
code_unit: flytekit.core.workflow
code_unit_type: class
help_text: ''
key: example_389544f1-7539-463a-8da8-a5970890705b
type: example

-->