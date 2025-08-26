# TaskExecutionGetDataResponse

This class represents the response data for a task execution&#x27;s get data operation. It encapsulates the inputs, outputs, and full input/output literal maps associated with a task execution. The class provides methods for converting between its internal representation and the Flyte IDL protocol buffer format.

## Attributes

- **inputs**: [UrlBlob](flytekit_models_common_urlblob) = None
  - The inputs of the task execution.

- **outputs**: [UrlBlob](flytekit_models_common_urlblob) = None
  - The outputs of the task execution.

- **full_inputs**: [LiteralMap](flytekit_models_literals_literalmap) = None
  - The full inputs of the task execution.

- **full_outputs**: [LiteralMap](flytekit_models_literals_literalmap) = None
  - The full outputs of the task execution.



## Methods
@classmethod
def from_flyte_idl(cls: type, pb2_object: _task_execution_pb2.TaskExecutionGetDataResponse) - > [TaskExecutionGetDataResponse](flytekit_models_execution_taskexecutiongetdataresponse)
-  Converts a protobuf object to a TaskExecutionGetDataResponse object.
- **Parameters**

  - **cls**: type
    - The class itself.
  - **pb2_object**: _task_execution_pb2.TaskExecutionGetDataResponse
    - The protobuf object to convert.

- **Return Value**:
**[TaskExecutionGetDataResponse](flytekit_models_execution_taskexecutiongetdataresponse)**
  - A TaskExecutionGetDataResponse object.
```@classmethod
def to_flyte_idl()
```
-  Converts the TaskExecutionGetDataResponse object to a protobuf object.

- **Return Value**:
**_task_execution_pb2.TaskExecutionGetDataResponse**
  - A protobuf object representing the TaskExecutionGetDataResponse.
