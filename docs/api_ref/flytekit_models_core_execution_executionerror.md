# ExecutionError

This class represents an execution error within a system. It encapsulates error details such as a code, message, and URI. The class provides methods to convert to and from a Flyte IDL representation and exposes the error kind as an enum.

## Attributes

- **code**: string
  - The error code.

- **message**: string
  - The error message.

- **error_uri**: string
  - A URI that points to more information about the error.

- **kind**: int
  - Enum value from ErrorKind

## Constructors
def ExecutionError(code: string, message: string, error_uri: string, kind: int)
-  Initializes an ExecutionError object.
- **Parameters**

  - **code**: string
    - The error code.
  - **message**: string
    - The error message.
  - **error_uri**: string
    - The URI associated with the error.
  - **kind**: int
    - The kind of error, represented by an enum value from ErrorKind.

def ExecutionError(code: str, message: str, error_uri: str, kind: int)
-  Initialize an ExecutionError instance.
- **Parameters**

  - **code**: str
    - The error code.
  - **message**: str
    - The error message.
  - **error_uri**: str
    - The URI associated with the error.
  - **kind**: int
    - The kind of error, an enum value from ErrorKind.



## Methods
```@classmethod
def code()
```
-  Get the error code.

- **Return Value**:
**string**
  - The error code.
```@classmethod
def message()
```
-  Get the error message.

- **Return Value**:
**string**
  - The error message.
```@classmethod
def error_uri()
```
-  Get the URI associated with the error.

- **Return Value**:
**string**
  - The URI associated with the error.
```@classmethod
def kind()
```
-  Get the kind of error.

- **Return Value**:
**int**
  - Enum value from ErrorKind
```@classmethod
def to_flyte_idl()
```
-  Convert the ExecutionError instance to its Flyte IDL representation.

- **Return Value**:
**string**
  - flyteidl.core.execution_pb2.ExecutionError
@classmethod
def from_flyte_idl(p: string) - > string
-  Create an ExecutionError instance from its Flyte IDL representation.
- **Parameters**

  - **p**: string
    - flyteidl.core.execution_pb2.ExecutionError

- **Return Value**:
**string**
  - ExecutionError
