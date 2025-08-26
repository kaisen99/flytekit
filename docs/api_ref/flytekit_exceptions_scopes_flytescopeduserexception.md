# FlyteScopedUserException

This class is a specialized exception class designed to handle user-related errors within a Flyte context. It extends FlyteScopedException, providing a structured way to categorize and manage exceptions. The class includes a verbose message property to provide more detailed information about the user error.

## Constructors
def FlyteScopedUserException(exc_type: Any, exc_value: Any, exc_tb: Any, **kwargs: Any)
-  Initializes a FlyteScopedUserException instance.
- **Parameters**

  - **exc_type**: Any
    - The type of the exception.
  - **exc_value**: Any
    - The exception value.
  - **exc_tb**: Any
    - The traceback object.
  - ****kwargs**: Any
    - Additional keyword arguments.



## Methods
```@classmethod
def verbose_message()
```
-  Returns the verbose message for the exception.

- **Return Value**:
**string**
  - The verbose message.
