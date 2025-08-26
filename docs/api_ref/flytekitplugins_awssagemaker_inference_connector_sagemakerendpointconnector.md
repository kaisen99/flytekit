# SageMakerEndpointConnector

This class serves as a connector for interacting with SageMaker endpoints. It facilitates the creation, retrieval, and deletion of SageMaker endpoints. It leverages asynchronous operations and integrates with Boto3 for AWS service interactions.

## Attributes

- **name**: string = &quot;SageMaker Endpoint Connector&quot;
  - SageMaker Endpoint Connector

## Constructors
```def SageMakerEndpointConnector()
```
-  Initializes the SageMakerEndpointConnector with service, task type, and metadata type.



## Methods
@classmethod
def create(task_template: [TaskTemplate](flytekit_models_task_tasktemplate), inputs: Optional[[LiteralMap](flytekit_models_literals_literalmap)]) - > [SageMakerEndpointMetadata](flytekitplugins_awssagemaker_inference_connector_sagemakerendpointmetadata)
-  Creates a SageMaker endpoint.
- **Parameters**

  - **task_template**: [TaskTemplate](flytekit_models_task_tasktemplate)
    - The task template containing configuration for the endpoint.
  - **inputs**: Optional[[LiteralMap](flytekit_models_literals_literalmap)]
    - Optional inputs for the endpoint creation.

- **Return Value**:
**[SageMakerEndpointMetadata](flytekitplugins_awssagemaker_inference_connector_sagemakerendpointmetadata)**
  - Metadata of the created SageMaker endpoint.
@classmethod
def get(resource_meta: [SageMakerEndpointMetadata](flytekitplugins_awssagemaker_inference_connector_sagemakerendpointmetadata)) - > [Resource](flytekit_extend_backend_base_connector_resource)
-  Retrieves the status and details of a SageMaker endpoint.
- **Parameters**

  - **resource_meta**: [SageMakerEndpointMetadata](flytekitplugins_awssagemaker_inference_connector_sagemakerendpointmetadata)
    - Metadata of the SageMaker endpoint to retrieve.

- **Return Value**:
**[Resource](flytekit_extend_backend_base_connector_resource)**
  - Resource object containing the endpoint&#x27;s phase, outputs, and message.
@classmethod
def delete(resource_meta: [SageMakerEndpointMetadata](flytekitplugins_awssagemaker_inference_connector_sagemakerendpointmetadata))
-  Deletes a SageMaker endpoint.
- **Parameters**

  - **resource_meta**: [SageMakerEndpointMetadata](flytekitplugins_awssagemaker_inference_connector_sagemakerendpointmetadata)
    - Metadata of the SageMaker endpoint to delete.

