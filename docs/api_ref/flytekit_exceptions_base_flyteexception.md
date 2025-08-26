# FlyteException

This class serves as the base exception class for Flyte, extending the built-in Exception class. It provides a structured way to represent and handle errors within the Flyte ecosystem. Key features include the ability to store a timestamp and a standardized string representation for error messages.

## Constructors
def FlyteException(args: Any, timestamp: Optional[float] = None)
-  Initializes a FlyteException with optional arguments and a timestamp.
- **Parameters**

  - **args**: Any
    - Positional arguments for the exception.
  - **timestamp**: Optional[float]
    - The timestamp as fractional seconds since epoch.



## Methods
```@classmethod
def timestamp()
```
-  The timestamp as fractional seconds since epoch

- **Return Value**:
**Optional[float]**
  - The timestamp as fractional seconds since epoch
