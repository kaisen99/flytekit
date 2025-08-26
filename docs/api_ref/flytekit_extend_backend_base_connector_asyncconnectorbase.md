# AsyncConnectorBase

This class serves as the foundational abstract base for asynchronous connectors. It establishes a standardized interface that all concrete connector implementations must adhere to.  It defines methods for creating, retrieving status, and deleting tasks, ensuring consistent interaction with a connector service.  The class relies on abstract methods that must be implemented by subclasses to handle specific task types.

## Attributes

- **name**: string = &quot;Base Async Connector&quot;
  - Base Async Connector

## Constructors
def AsyncConnectorBase(metadata_type: [ResourceMeta](flytekit_extend_backend_base_connector_resourcemeta), kwargs: dict)
-  Initializes the AsyncConnectorBase with metadata type and other keyword arguments.
- **Parameters**

  - **metadata_type**: [ResourceMeta](flytekit_extend_backend_base_connector_resourcemeta)
    - The metadata type for the resource.
  - **kwargs**: dict
    - Additional keyword arguments to pass to the parent class constructor.



## Methods
```@classmethod
def metadata_type()
```
-  Return a resource meta that can be used to get the status of the task.

- **Return Value**:
**[ResourceMeta](flytekit_extend_backend_base_connector_resourcemeta)**
  - Return a resource meta that can be used to get the status of the task.
@classmethod
def create(task_template: [TaskTemplate](flytekit_models_task_tasktemplate), output_prefix: str, inputs: Optional[[LiteralMap](flytekit_models_literals_literalmap)], task_execution_metadata: Optional[[TaskExecutionMetadata](flytekit_models_task_taskexecutionmetadata)]) - > [ResourceMeta](flytekit_extend_backend_base_connector_resourcemeta)
-  Return a resource meta that can be used to get the status of the task.
- **Parameters**

  - **task_template**: [TaskTemplate](flytekit_models_task_tasktemplate)
    - task_template
  - **output_prefix**: str
    - output_prefix
  - **inputs**: Optional[[LiteralMap](flytekit_models_literals_literalmap)]
    - inputs
  - **task_execution_metadata**: Optional[[TaskExecutionMetadata](flytekit_models_task_taskexecutionmetadata)]
    - task_execution_metadata

- **Return Value**:
**[ResourceMeta](flytekit_extend_backend_base_connector_resourcemeta)**
  - Return a resource meta that can be used to get the status of the task.
@classmethod
def get(resource_meta: [ResourceMeta](flytekit_extend_backend_base_connector_resourcemeta)) - > [Resource](flytekit_extend_backend_base_connector_resource)
-  Return the status of the task, and return the outputs in some cases. For example, bigquery job can&#x27;t write the structured dataset to the output location, so it returns the output literals to the propeller, and the propeller will write the structured dataset to the blob store.
- **Parameters**

  - **resource_meta**: [ResourceMeta](flytekit_extend_backend_base_connector_resourcemeta)
    - resource_meta

- **Return Value**:
**[Resource](flytekit_extend_backend_base_connector_resource)**
  - Return the status of the task, and return the outputs in some cases. For example, bigquery job can&#x27;t write the structured dataset to the output location, so it returns the output literals to the propeller, and the propeller will write the structured dataset to the blob store.
@classmethod
def delete(resource_meta: [ResourceMeta](flytekit_extend_backend_base_connector_resourcemeta)) - > None
-  Delete the task. This call should be idempotent. It should raise an error if fails to delete the task.
- **Parameters**

  - **resource_meta**: [ResourceMeta](flytekit_extend_backend_base_connector_resourcemeta)
    - resource_meta

- **Return Value**:
**None**
  - Delete the task. This call should be idempotent. It should raise an error if fails to delete the task.
@classmethod
def get_metrics(resource_meta: [ResourceMeta](flytekit_extend_backend_base_connector_resourcemeta)) - > GetTaskMetricsResponse
-  Return the metrics for the task.
- **Parameters**

  - **resource_meta**: [ResourceMeta](flytekit_extend_backend_base_connector_resourcemeta)
    - resource_meta

- **Return Value**:
**GetTaskMetricsResponse**
  - Return the metrics for the task.
@classmethod
def get_logs(resource_meta: [ResourceMeta](flytekit_extend_backend_base_connector_resourcemeta)) - > GetTaskLogsResponse
-  Return the metrics for the task.
- **Parameters**

  - **resource_meta**: [ResourceMeta](flytekit_extend_backend_base_connector_resourcemeta)
    - resource_meta

- **Return Value**:
**GetTaskLogsResponse**
  - Return the metrics for the task.
