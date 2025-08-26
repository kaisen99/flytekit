# RetryExceptionWrapperInterceptor

This class serves as a gRPC interceptor, designed to handle and retry unary-unary and unary-stream client calls. It intercepts gRPC calls to catch specific exceptions and implement a retry mechanism. The interceptor retries failed calls up to a predefined maximum number of times.

## Attributes

- **_max_retries**: int = 3
  - The maximum number of retries allowed for a request.

## Constructors
def RetryExceptionWrapperInterceptor(max_retries: int = 3)
-  Initializes the RetryExceptionWrapperInterceptor with a maximum number of retries.
- **Parameters**

  - **max_retries**: int
    - The maximum number of times to retry an operation before raising an exception.



## Methods
@classmethod
def intercept_unary_unary(continuation: callable, client_call_details: grpc.ClientCallDetails, request: typing.Any) - > grpc.Future
-  Intercepts unary-unary gRPC calls, retrying them up to a maximum number of times if specific exceptions occur. It uses `_raise_if_exc` to handle and raise appropriate Flyte exceptions.
- **Parameters**

  - **continuation**: callable
    - The continuation function to call the next interceptor or the actual RPC method.
  - **client_call_details**: grpc.ClientCallDetails
    - Details about the client call.
  - **request**: typing.Any
    - The request message for the RPC.

- **Return Value**:
**grpc.Future**
  - The future object representing the result of the gRPC call.
@classmethod
def intercept_unary_stream(continuation: callable, client_call_details: grpc.ClientCallDetails, request: typing.Any) - > grpc.Call
-  Intercepts unary-stream gRPC calls. This implementation simply calls the continuation and returns the resulting gRPC call object without any retry logic.
- **Parameters**

  - **continuation**: callable
    - The continuation function to call the next interceptor or the actual RPC method.
  - **client_call_details**: grpc.ClientCallDetails
    - Details about the client call.
  - **request**: typing.Any
    - The request message for the RPC.

- **Return Value**:
**grpc.Call**
  - The gRPC call object for the unary-stream operation.
