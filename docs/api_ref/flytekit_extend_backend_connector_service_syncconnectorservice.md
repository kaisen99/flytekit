# SyncConnectorService

This class serves as a gRPC service for executing synchronous tasks. It receives task execution requests, retrieves the appropriate connector, and executes the task. The class handles input and output serialization, and manages error handling and metrics collection during task execution.



## Methods
@classmethod
def ExecuteTaskSync(request_iterator: typing.AsyncIterator[ExecuteTaskSyncRequest], context: grpc.ServicerContext) - > typing.AsyncIterator[ExecuteTaskSyncResponse]
-  Executes a task synchronously, handling requests and responses through an asynchronous iterator. It retrieves a connector based on the task type, processes inputs, executes the task using the connector, and yields the results. Exceptions during execution are handled and logged.
- **Parameters**

  - **request_iterator**: typing.AsyncIterator[ExecuteTaskSyncRequest]
    - An asynchronous iterator providing ExecuteTaskSyncRequest objects.
  - **context**: grpc.ServicerContext
    - The gRPC servicer context.

- **Return Value**:
**typing.AsyncIterator[ExecuteTaskSyncResponse]**
  - An asynchronous iterator yielding ExecuteTaskSyncResponse objects containing the task execution results.
