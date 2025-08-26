# DefaultMetadataInterceptor

This class serves as a gRPC client interceptor, designed to inject default metadata into outgoing unary and stream calls. It modifies client call details to include a default &#x27;accept&#x27; metadata entry. The class implements both UnaryUnaryClientInterceptor and UnaryStreamClientInterceptor interfaces, ensuring compatibility with different gRPC call types.

## Constructors
```def DefaultMetadataInterceptor()
```
-  Initializes the DefaultMetadataInterceptor.



## Methods
@classmethod
def intercept_unary_unary(continuation: typing.Callable, client_call_details: grpc.ClientCallDetails, request: typing.Any) - > None
-  Intercepts unary calls and inject default metadata
- **Parameters**

  - **continuation**: typing.Callable
    - The continuation function to call.
  - **client_call_details**: grpc.ClientCallDetails
    - The client call details.
  - **request**: typing.Any
    - The request object.

- **Return Value**:
**None**
  - The result of the continuation with updated call details.
@classmethod
def intercept_unary_stream(continuation: typing.Callable, client_call_details: grpc.ClientCallDetails, request: typing.Any) - > None
-  Handles a stream call and inject default metadata
- **Parameters**

  - **continuation**: typing.Callable
    - The continuation function to call.
  - **client_call_details**: grpc.ClientCallDetails
    - The client call details.
  - **request**: typing.Any
    - The request object.

- **Return Value**:
**None**
  - The result of the continuation with updated call details.
