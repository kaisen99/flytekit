
<!--
help_text: ''
key: summary_aws_sagemaker_inference_integration_431e7ebf-25e2-4207-a388-9115e3f455f2
modules:
- flytekitplugins.awssagemaker_inference.boto3_connector
- flytekitplugins.awssagemaker_inference.boto3_mixin
- flytekitplugins.awssagemaker_inference.boto3_task
- flytekitplugins.awssagemaker_inference.connector
- flytekitplugins.awssagemaker_inference.task
questions_to_answer: []
type: summary

-->
AWS SageMaker Inference Integration

The AWS SageMaker Inference Integration provides a robust and flexible way to manage AWS SageMaker inference resources directly within Flyte workflows. This integration offers both a generic mechanism for interacting with any AWS Boto3 service and specialized tasks tailored for common SageMaker inference operations, such as creating models, configuring endpoints, deploying endpoints, and invoking them.

### Generic AWS Boto3 Operations

The core of this integration is the ability to execute arbitrary AWS Boto3 API calls as Flyte tasks. This is achieved through the `BotoTask` and its associated `BotoConfig`.

*   **`BotoTask`**: This is a generic Flyte task that wraps a Boto3 API call. It allows you to specify the AWS service, the method to call, and the configuration parameters for that method.
*   **`BotoConfig`**: This configuration object defines the parameters for a `BotoTask`. It includes:
    *   `service` (str): The AWS service name (e.g., "sagemaker", "sagemaker-runtime", "ec2").
    *   `method` (str): The Boto3 method to invoke (e.g., "create_model", "describe_endpoint", "run_instances").
    *   `config` (Dict[str, Any]): A dictionary of arguments passed directly to the Boto3 method. This dictionary supports dynamic templating using Flyte task inputs and image references.
    *   `region` (Optional[str]): The AWS region for the Boto3 client. If not provided, the region is determined from inputs or environment.
    *   `images` (Optional[Dict[str, Union[str, ImageSpec]]]): A dictionary of Docker images, particularly useful for SageMaker model creation. `ImageSpec` objects are built locally before serialization.

The `BotoTask` returns a `result` dictionary containing the response from the Boto3 call and an `idempotence_token` if applicable.

**Example: Generic Boto3 Task**

To create a generic Boto3 task, instantiate `BotoTask` with the desired `BotoConfig`.

```python
from flytekitplugins.awssagemaker_inference.boto3_task import BotoTask, BotoConfig
from flytekit import task, workflow, kwtypes

# Example: Describe a SageMaker endpoint configuration
describe_endpoint_config_task = BotoTask(
    name="describe_sagemaker_endpoint_config",
    task_config=BotoConfig(
        service="sagemaker",
        method="describe_endpoint_config",
        config={"EndpointConfigName": "{inputs.endpoint_config_name}"}, # Using templating
        region="us-east-1",
    ),
    inputs=kwtypes(endpoint_config_name=str), # Define inputs for templating
)

@workflow
def describe_workflow(endpoint_config_name: str):
    result = describe_endpoint_config_task(endpoint_config_name=endpoint_config_name)
    print(f"Endpoint Config Description: {result['result']}")
    return result
```

### SageMaker Inference Lifecycle Tasks

Building upon the generic Boto3 capabilities, the integration provides specialized tasks for common SageMaker inference operations. These tasks simplify the definition of workflows that manage the full lifecycle of SageMaker models and endpoints.

*   **`SageMakerModelTask`**: Creates a SageMaker model.
    *   **Purpose**: Registers a machine learning model with SageMaker, specifying its artifacts and execution environment.
    *   **Parameters**: `name`, `config` (for `create_model` API), `region`, `images` (for container images).
    *   **Output**: Returns the `ModelArn` upon successful creation.

*   **`SageMakerEndpointConfigTask`**: Creates a SageMaker endpoint configuration.
    *   **Purpose**: Defines the configuration for deploying a model to an endpoint, including instance types, counts, and model variants.
    *   **Parameters**: `name`, `config` (for `create_endpoint_config` API), `region`.
    *   **Output**: Returns the `EndpointConfigArn` upon successful creation.

*   **`SageMakerEndpointTask`**: Creates a SageMaker endpoint.
    *   **Purpose**: Deploys a SageMaker endpoint based on a previously created endpoint configuration. This task is asynchronous, monitoring the endpoint's status until it is `InService` or `Failed`.
    *   **Parameters**: `name`, `config` (for `create_endpoint` API), `region`.
    *   **Output**: Returns the `EndpointArn` once the endpoint is `InService`.

*   **`SageMakerInvokeEndpointTask`**: Invokes a SageMaker endpoint asynchronously.
    *   **Purpose**: Sends inference requests to a deployed SageMaker endpoint. This task uses the `sagemaker-runtime` service.
    *   **Parameters**: `name`, `config` (for `invoke_endpoint_async` API), `region`.
    *   **Output**: Returns the result of the invocation.

*   **Deletion Tasks**:
    *   **`SageMakerDeleteModelTask`**: Deletes a SageMaker model.
    *   **`SageMakerDeleteEndpointConfigTask`**: Deletes a SageMaker endpoint configuration.
    *   **`SageMakerDeleteEndpointTask`**: Deletes a SageMaker endpoint.
    *   **Purpose**: Clean up SageMaker resources. These tasks are crucial for managing costs and resource quotas.
    *   **Parameters**: `name`, `config` (e.g., `{"ModelName": "my-model"}`), `region`.

**Example: Full SageMaker Inference Workflow**

This example demonstrates how to orchestrate the creation, invocation, and deletion of a SageMaker model and endpoint.

```python
from flytekit import task, workflow, kwtypes
from flytekitplugins.awssagemaker_inference.task import (
    SageMakerModelTask,
    SageMakerEndpointConfigTask,
    SageMakerEndpointTask,
    SageMakerInvokeEndpointTask,
    SageMakerDeleteEndpointTask,
    SageMakerDeleteEndpointConfigTask,
    SageMakerDeleteModelTask,
)
from flytekit.image_spec import ImageSpec
from typing import Dict, Any

# Define an ImageSpec for the model container if building locally
# Otherwise, provide a direct ECR image URI
my_model_image = ImageSpec(
    name="my-sagemaker-model-image",
    builder="docker",
    packages=["scikit-learn"], # Example packages
    registry="your-ecr-registry-url", # Replace with your ECR registry
)

# 1. Create SageMaker Model Task
create_model_task = SageMakerModelTask(
    name="create_my_sagemaker_model",
    config={
        "ModelName": "{inputs.model_name}",
        "PrimaryContainer": {
            "Image": "{images.primary_container_image}",
            "ModelDataUrl": "{inputs.model_data_url}",
        },
        "ExecutionRoleArn": "{inputs.execution_role_arn}",
    },
    region="us-east-1",
    images={"primary_container_image": my_model_image},
    inputs=kwtypes(model_name=str, model_data_url=str, execution_role_arn=str),
)

# 2. Create SageMaker Endpoint Config Task
create_endpoint_config_task = SageMakerEndpointConfigTask(
    name="create_my_sagemaker_endpoint_config",
    config={
        "EndpointConfigName": "{inputs.endpoint_config_name}",
        "ProductionVariants": [
            {
                "VariantName": "AllTraffic",
                "ModelName": "{inputs.model_name}",
                "InitialInstanceCount": 1,
                "InstanceType": "ml.m5.large",
            }
        ],
    },
    region="us-east-1",
    inputs=kwtypes(endpoint_config_name=str, model_name=str),
)

# 3. Create SageMaker Endpoint Task
create_endpoint_task = SageMakerEndpointTask(
    name="create_my_sagemaker_endpoint",
    config={
        "EndpointName": "{inputs.endpoint_name}",
        "EndpointConfigName": "{inputs.endpoint_config_name}",
    },
    region="us-east-1",
    inputs=kwtypes(endpoint_name=str, endpoint_config_name=str),
)

# 4. Invoke SageMaker Endpoint Task (asynchronous invocation)
invoke_endpoint_task = SageMakerInvokeEndpointTask(
    name="invoke_my_sagemaker_endpoint",
    config={ # Note: This is 'config' directly, not 'task_config' for specialized tasks
        "EndpointName": "{inputs.endpoint_name}",
        "ContentType": "application/json",
        "InputLocation": "{inputs.input_data_s3_uri}",
        "RequestContentType": "application/json",
        "Accept": "application/json",
    },
    region="us-east-1",
    inputs=kwtypes(endpoint_name=str, input_data_s3_uri=str),
)

# 5. Delete SageMaker Endpoint Task
delete_endpoint_task = SageMakerDeleteEndpointTask(
    name="delete_my_sagemaker_endpoint",
    config={"EndpointName": "{inputs.endpoint_name}"},
    region="us-east-1",
    inputs=kwtypes(endpoint_name=str),
)

# 6. Delete SageMaker Endpoint Config Task
delete_endpoint_config_task = SageMakerDeleteEndpointConfigTask(
    name="delete_my_sagemaker_endpoint_config",
    config={"EndpointConfigName": "{inputs.endpoint_config_name}"},
    region="us-east-1",
    inputs=kwtypes(endpoint_config_name=str),
)

# 7. Delete SageMaker Model Task
delete_model_task = SageMakerDeleteModelTask(
    name="delete_my_sagemaker_model",
    config={"ModelName": "{inputs.model_name}"},
    region="us-east-1",
    inputs=kwtypes(model_name=str),
)

@workflow
def sagemaker_inference_workflow(
    model_name: str,
    endpoint_config_name: str,
    endpoint_name: str,
    model_data_url: str,
    execution_role_arn: str,
    input_data_s3_uri: str,
):
    # Create resources
    model_result = create_model_task(
        model_name=model_name,
        model_data_url=model_data_url,
        execution_role_arn=execution_role_arn,
    )
    endpoint_config_result = create_endpoint_config_task(
        endpoint_config_name=endpoint_config_name,
        model_name=model_name,
    )
    endpoint_result = create_endpoint_task(
        endpoint_name=endpoint_name,
        endpoint_config_name=endpoint_config_name,
    )

    # Invoke endpoint
    invocation_result = invoke_endpoint_task(
        endpoint_name=endpoint_name,
        input_data_s3_uri=input_data_s3_uri,
    )

    # Clean up resources (often done in a final task or separate workflow for robustness)
    delete_endpoint_task(endpoint_name=endpoint_name)
    delete_endpoint_config_task(endpoint_config_name=endpoint_config_name)
    delete_model_task(model_name=model_name)

    return invocation_result
```

### Dynamic Configuration and Idempotency

The `config` parameter in `BotoConfig` and the specialized SageMaker tasks supports string formatting. This allows you to inject values from task inputs or image references directly into the Boto3 API call configuration.

*   **Input Templating**: Use `{inputs.your_input_name}` within the `config` dictionary to reference a task input. For example, `{"EndpointName": "{inputs.endpoint_name}"}`.
*   **Image Templating**: Use `{images.your_image_key}` to reference an image defined in the `images` parameter. This is particularly useful for specifying container images in SageMaker model definitions.

The integration also handles **idempotency** for `create_model` and `create_endpoint_config` calls. If a resource with the specified name already exists, the task will succeed without attempting to re-create it, returning the ARN of the existing resource. This prevents errors in scenarios where a workflow might be retried or re-executed.

### Error Handling and Resource Management

The integration provides specific error handling for common AWS scenarios:

*   **Existing Resources**: If a `create_model` or `create_endpoint_config` call fails because the resource already exists (`ValidationException` with "Cannot create already existing"), the task will gracefully succeed, returning the ARN of the existing resource.
*   **Endpoint Creation Failures**: For `SageMakerEndpointTask`, if endpoint creation fails due to `ResourceLimitExceeded` or if the endpoint cannot be found, specific messages are provided to aid debugging.
*   **Asynchronous Resource Monitoring**: For `SageMakerEndpointTask`, the underlying `SageMakerEndpointConnector` continuously polls the AWS API to monitor the endpoint's status, mapping AWS states (e.g., `Creating`, `InService`, `Failed`) to Flyte task phases. This ensures that the Flyte task accurately reflects the long-running AWS operation's progress.

### Considerations and Best Practices

*   **AWS Permissions**: Ensure the IAM role associated with your Flyte execution environment has the necessary permissions to perform the specified SageMaker (and other AWS) API calls.
*   **Resource Cleanup**: For long-running resources like SageMaker endpoints, it is a best practice to include deletion tasks in your workflows, especially in development or testing environments, to manage costs and avoid hitting AWS service quotas.
*   **Region Consistency**: While `region` can be specified per task, maintaining consistency across related tasks in a workflow is generally recommended.
*   **Input Validation**: Although the plugin handles some AWS-specific error codes, ensure your task inputs conform to AWS API requirements to prevent `ValidationException` errors.
*   **`ImageSpec` Usage**: When using `ImageSpec` for `images`, Flyte will attempt to build the Docker image locally before the task execution. Ensure your Flyte environment has Docker installed and configured if you rely on this feature. For production, it's often better to pre-build and push images to ECR, then reference them directly by URI.
<!--
key: summary_aws_sagemaker_inference_integration_431e7ebf-25e2-4207-a388-9115e3f455f2
type: summary_end

-->
<!--
code_unit: flytekitplugins.awssagemaker_inference.examples.sagemaker_endpoint_deployment
code_unit_type: class
help_text: ''
key: example_dd90d248-36f0-4381-916d-f52dab615cc9
type: example

-->