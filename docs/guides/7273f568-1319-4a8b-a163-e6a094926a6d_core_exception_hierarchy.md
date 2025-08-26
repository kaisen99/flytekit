
<!--
help_text: ''
key: summary_core_exception_hierarchy_f7622d15-43be-470a-97ab-08292ad8e785
modules:
- flytekit.exceptions.base
questions_to_answer: []
type: summary

-->
# Core Exception Hierarchy

The core exception hierarchy provides a robust and standardized mechanism for error handling within the system. This structure ensures that errors are not only descriptive for human understanding but also programmatically identifiable, facilitating automated error recovery and detailed diagnostics.

## The `FlyteException` Base Class

All custom exceptions within the system derive from `FlyteException`. This base class extends Python's built-in `Exception`, incorporating additional capabilities crucial for operational visibility and control.

Each `FlyteException` instance exposes an `error_code` property. This property provides a consistent, machine-readable identifier for the specific type of error that occurred. The `error_code` is defined by the `_ERROR_CODE` class attribute. By default, `FlyteException` instances report an `error_code` of "UnknownFlyteException".

```python
# Example: Accessing the error_code
try:
    raise FlyteException("Something went wrong")
except FlyteException as e:
    print(f"Caught error with code: {e.error_code}")
    # Output: Caught error with code: UnknownFlyteException
```

Additionally, `FlyteException` includes a `timestamp` property, which records the time the exception was instantiated. This timestamp, represented as fractional seconds since the epoch, is invaluable for correlating errors across logs and tracing execution flows.

The string representation of `FlyteException` instances is enhanced to include the `error_code`, the exception arguments, and, if present, the underlying cause (`__cause__`). This provides a comprehensive summary for debugging purposes.

```python
import time

# Example: Raising and inspecting FlyteException
try:
    raise FlyteException("Failed to process data", timestamp=time.time())
except FlyteException as e:
    print(e)
    # Output example: UnknownFlyteException: error=Failed to process data
    print(f"Error occurred at: {e.timestamp}")
```

When raising exceptions, consider chaining them using `raise ... from ...` to preserve the original cause, which significantly aids in debugging complex issues.

```python
# Example: Chaining exceptions
class DataProcessingError(FlyteException):
    _ERROR_CODE = "DataProcessingFailed"

def process_data():
    try:
        # Simulate an underlying error
        1 / 0
    except ZeroDivisionError as e:
        raise DataProcessingError("Failed to calculate metrics") from e

try:
    process_data()
except DataProcessingError as e:
    print(e)
    # Output example: DataProcessingFailed: error=Failed to calculate metrics, cause=division by zero
```

## Distinguishing Recoverable Errors

The hierarchy includes `FlyteRecoverableException`, a specialized exception type that inherits from `FlyteException`. This class is specifically designed to signal errors from which a system or workflow can potentially recover, often through retries or alternative execution paths. `FlyteRecoverableException` instances have an `error_code` of "RecoverableFlyteException".

Identifying an error as recoverable is critical for building resilient systems. For example, a task might encounter a transient network issue, which is a recoverable error. In contrast, a fundamental logic error is typically not recoverable.

```python
# Example: Handling recoverable exceptions
class TransientNetworkError(FlyteRecoverableException):
    _ERROR_CODE = "NetworkTransient"

def make_api_call():
    # Simulate a transient network issue
    raise TransientNetworkError("API call failed due to network glitch")

try:
    make_api_call()
except TransientNetworkError as e:
    print(f"Caught recoverable error: {e.error_code}. Retrying...")
    # Implement retry logic here
except FlyteException as e:
    print(f"Caught unrecoverable error: {e.error_code}. Aborting.")
    # Handle unrecoverable error
```

## Custom Exception Development

When extending the exception hierarchy for specific application domains or components, always derive new exception classes from `FlyteException` or one of its specialized subclasses like `FlyteRecoverableException`.

Define a unique `_ERROR_CODE` for each custom exception. This ensures that downstream systems can reliably identify and react to specific error conditions programmatically. Choose descriptive and concise error codes that reflect the nature of the error.

```python
# Example: Defining a custom exception
class InvalidConfigurationError(FlyteException):
    """
    Raised when a critical configuration parameter is missing or invalid.
    """
    _ERROR_CODE = "InvalidConfiguration"

class ResourceUnavailableError(FlyteRecoverableException):
    """
    Raised when a required external resource is temporarily unavailable.
    """
    _ERROR_CODE = "ResourceTemporarilyUnavailable"

# Usage
try:
    raise InvalidConfigurationError("Database connection string is missing")
except InvalidConfigurationError as e:
    print(f"Configuration error: {e.error_code} - {e.args[0]}")

try:
    raise ResourceUnavailableError("External service is down")
except ResourceUnavailableError as e:
    print(f"Recoverable resource error: {e.error_code} - {e.args[0]}")
```

## Error Handling Best Practices

*   **Specificity in Catching:** Catch the most specific exception types first, then broader `FlyteException` types, and finally generic `Exception` if necessary. This allows for fine-grained error handling.
*   **Leverage `error_code`:** Use the `error_code` property for programmatic decision-making, such as triggering retries, sending specific alerts, or routing errors to different handlers. Avoid parsing error messages for this purpose.
*   **Utilize `timestamp`:** Integrate the `timestamp` into logging and monitoring systems to accurately track when errors occur, aiding in performance analysis and incident response.
*   **Chain Exceptions:** Always chain exceptions using `raise ... from ...` when re-raising an exception with more context. This preserves the original stack trace and error information, which is invaluable for debugging.
*   **Clear Messages:** Provide clear, concise, and actionable messages when raising exceptions. The message should help a developer understand what went wrong and, ideally, how to resolve it.
<!--
key: summary_core_exception_hierarchy_f7622d15-43be-470a-97ab-08292ad8e785
type: summary_end

-->
<!--
code_unit: flytekit.core.task
code_unit_type: class
help_text: ''
key: example_eba83a32-ccdb-4720-be59-2138d024ad43
type: example

-->
<!--
code_unit: flytekit.core.workflow
code_unit_type: class
help_text: ''
key: example_c82ff54d-6d9c-4a65-9d35-f2ecf08561ae
type: example

-->