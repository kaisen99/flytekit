# ConnectorMetadataService

This class provides a gRPC service for retrieving metadata about connectors. It allows clients to fetch a specific connector&#x27;s metadata by name and list all available connectors. The class leverages a ConnectorRegistry to access and manage connector metadata.



## Methods
@classmethod
def GetAgent(request: GetAgentRequest, context: grpc.ServicerContext) - > GetAgentResponse
-  Retrieves agent information based on the provided name.
- **Parameters**

  - **request**: GetAgentRequest
    - The request object containing the name of the agent to retrieve.
  - **context**: grpc.ServicerContext
    - The gRPC service context.

- **Return Value**:
**GetAgentResponse**
  - The response object containing agent details.
@classmethod
def ListAgents(request: ListAgentsRequest, context: grpc.ServicerContext) - > ListAgentsResponse
-  Retrieves a list of all available agents.
- **Parameters**

  - **request**: ListAgentsRequest
    - The request object for listing agents.
  - **context**: grpc.ServicerContext
    - The gRPC service context.

- **Return Value**:
**ListAgentsResponse**
  - The response object containing a list of agents.
