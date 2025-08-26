# RawSynchronousFlyteClient

This class provides a synchronous client for interacting with the Flyte Admin service using gRPC. It offers methods to manage tasks, workflows, launch plans, and executions. The client utilizes auto-generated gRPC stubs and requires a PlatformConfig for configuration.

## Constructors
def RawSynchronousFlyteClient(cfg: [PlatformConfig](flytekit_configuration_platformconfig), **kwargs: dict)
-  Initializes a gRPC channel to the given Flyte Admin service.
- **Parameters**

  - **cfg**: [PlatformConfig](flytekit_configuration_platformconfig)
    - Platform configuration object containing endpoint and security settings.
  - ****kwargs**: dict
    - Additional keyword arguments.



## Methods
@classmethod
def with_root_certificate(cfg: [PlatformConfig](flytekit_configuration_platformconfig), root_cert_file: str) - > [RawSynchronousFlyteClient](flytekit_clients_raw_rawsynchronousflyteclient)
-  Class method to create a RawSynchronousFlyteClient with a root certificate.
- **Parameters**

  - **cfg**: [PlatformConfig](flytekit_configuration_platformconfig)
    - The platform configuration.
  - **root_cert_file**: str
    - The path to the root certificate file.

- **Return Value**:
**[RawSynchronousFlyteClient](flytekit_clients_raw_rawsynchronousflyteclient)**
  - A new instance of RawSynchronousFlyteClient.
```@classmethod
def url()
```
-  Returns the endpoint URL of the Flyte Admin service.

- **Return Value**:
**str**
  - The endpoint URL.
@classmethod
def create_task(task_create_request: flyteidl.admin.task_pb2.TaskCreateRequest) - > flyteidl.admin.task_pb2.TaskCreateResponse
-  Creates a task definition in the Admin database. Overwrites are not supported.
- **Parameters**

  - **task_create_request**: flyteidl.admin.task_pb2.TaskCreateRequest
    - The request protobuf object for creating a task.

- **Return Value**:
**flyteidl.admin.task_pb2.TaskCreateResponse**
  - The response from creating the task.
@classmethod
def list_task_ids_paginated(identifier_list_request: flyteidl.admin.common_pb2.NamedEntityIdentifierListRequest) - > flyteidl.admin.common_pb2.NamedEntityIdentifierList
-  Returns a page of identifiers for tasks for a given project and domain. Filters can be specified. The name field in the request is ignored. This is a paginated API.
- **Parameters**

  - **identifier_list_request**: flyteidl.admin.common_pb2.NamedEntityIdentifierListRequest
    - The request protobuf object for listing task identifiers.

- **Return Value**:
**flyteidl.admin.common_pb2.NamedEntityIdentifierList**
  - A paginated list of task identifiers.
@classmethod
def list_tasks_paginated(resource_list_request: flyteidl.admin.common_pb2.ResourceListRequest) - > flyteidl.admin.task_pb2.TaskList
-  Returns a page of task metadata for tasks in a given project and domain. Optionally, specifying a name will limit the results. This is a paginated API.
- **Parameters**

  - **resource_list_request**: flyteidl.admin.common_pb2.ResourceListRequest
    - The request protobuf object for listing resources.

- **Return Value**:
**flyteidl.admin.task_pb2.TaskList**
  - A paginated list of task metadata.
@classmethod
def get_task(get_object_request: flyteidl.admin.common_pb2.ObjectGetRequest) - > flyteidl.admin.task_pb2.Task
-  Returns a single task for a given identifier.
- **Parameters**

  - **get_object_request**: flyteidl.admin.common_pb2.ObjectGetRequest
    - The request protobuf object for getting an object.

- **Return Value**:
**flyteidl.admin.task_pb2.Task**
  - The requested task object.
@classmethod
def set_signal(signal_set_request: SignalSetRequest) - > SignalSetResponse
-  Sets a signal.
- **Parameters**

  - **signal_set_request**: SignalSetRequest
    - The request protobuf object for setting a signal.

- **Return Value**:
**SignalSetResponse**
  - The response from setting the signal.
@classmethod
def list_signals(signal_list_request: SignalListRequest) - > SignalList
-  Lists signals.
- **Parameters**

  - **signal_list_request**: SignalListRequest
    - The request protobuf object for listing signals.

- **Return Value**:
**SignalList**
  - A list of signals.
@classmethod
def create_workflow(workflow_create_request: flyteidl.admin.workflow_pb2.WorkflowCreateRequest) - > flyteidl.admin.workflow_pb2.WorkflowCreateResponse
-  Creates a workflow definition in the Admin database. Overwrites are not supported.
- **Parameters**

  - **workflow_create_request**: flyteidl.admin.workflow_pb2.WorkflowCreateRequest
    - The request protobuf object for creating a workflow.

- **Return Value**:
**flyteidl.admin.workflow_pb2.WorkflowCreateResponse**
  - The response from creating the workflow.
@classmethod
def list_workflow_ids_paginated(identifier_list_request: flyteidl.admin.common_pb2.NamedEntityIdentifierListRequest) - > flyteidl.admin.common_pb2.NamedEntityIdentifierList
-  Returns a page of identifiers for workflows for a given project and domain. Filters can be specified. The name field in the request is ignored. This is a paginated API.
- **Parameters**

  - **identifier_list_request**: flyteidl.admin.common_pb2.NamedEntityIdentifierListRequest
    - The request protobuf object for listing workflow identifiers.

- **Return Value**:
**flyteidl.admin.common_pb2.NamedEntityIdentifierList**
  - A paginated list of workflow identifiers.
@classmethod
def list_workflows_paginated(resource_list_request: flyteidl.admin.common_pb2.ResourceListRequest) - > flyteidl.admin.workflow_pb2.WorkflowList
-  Returns a page of workflow meta-information for workflows in a given project and domain. Optionally, specifying a name will limit the results. This is a paginated API.
- **Parameters**

  - **resource_list_request**: flyteidl.admin.common_pb2.ResourceListRequest
    - The request protobuf object for listing resources.

- **Return Value**:
**flyteidl.admin.workflow_pb2.WorkflowList**
  - A paginated list of workflow metadata.
@classmethod
def get_workflow(get_object_request: flyteidl.admin.common_pb2.ObjectGetRequest) - > flyteidl.admin.workflow_pb2.Workflow
-  Returns a single workflow for a given identifier.
- **Parameters**

  - **get_object_request**: flyteidl.admin.common_pb2.ObjectGetRequest
    - The request protobuf object for getting an object.

- **Return Value**:
**flyteidl.admin.workflow_pb2.Workflow**
  - The requested workflow object.
@classmethod
def create_launch_plan(launch_plan_create_request: flyteidl.admin.launch_plan_pb2.LaunchPlanCreateRequest) - > flyteidl.admin.launch_plan_pb2.LaunchPlanCreateResponse
-  Creates a launch plan definition in the Admin database. Overwrites are not supported.
- **Parameters**

  - **launch_plan_create_request**: flyteidl.admin.launch_plan_pb2.LaunchPlanCreateRequest
    - The request protobuf object for creating a launch plan.

- **Return Value**:
**flyteidl.admin.launch_plan_pb2.LaunchPlanCreateResponse**
  - The response from creating the launch plan.
@classmethod
def get_launch_plan(object_get_request: flyteidl.admin.common_pb2.ObjectGetRequest) - > flyteidl.admin.launch_plan_pb2.LaunchPlan
-  Retrieves a launch plan entity.
- **Parameters**

  - **object_get_request**: flyteidl.admin.common_pb2.ObjectGetRequest
    - The request protobuf object for getting an object.

- **Return Value**:
**flyteidl.admin.launch_plan_pb2.LaunchPlan**
  - The requested launch plan object.
@classmethod
def get_active_launch_plan(active_launch_plan_request: flyteidl.admin.common_pb2.ActiveLaunchPlanRequest) - > flyteidl.admin.launch_plan_pb2.LaunchPlan
-  Retrieves an active launch plan entity.
- **Parameters**

  - **active_launch_plan_request**: flyteidl.admin.common_pb2.ActiveLaunchPlanRequest
    - The request protobuf object for getting an active launch plan.

- **Return Value**:
**flyteidl.admin.launch_plan_pb2.LaunchPlan**
  - The requested active launch plan object.
@classmethod
def update_launch_plan(update_request: flyteidl.admin.launch_plan_pb2.LaunchPlanUpdateRequest) - > flyteidl.admin.launch_plan_pb2.LaunchPlanUpdateResponse
-  Allows updates to a launch plan, currently only state switching between ACTIVE and INACTIVE.
- **Parameters**

  - **update_request**: flyteidl.admin.launch_plan_pb2.LaunchPlanUpdateRequest
    - The request protobuf object for updating a launch plan.

- **Return Value**:
**flyteidl.admin.launch_plan_pb2.LaunchPlanUpdateResponse**
  - The response from updating the launch plan.
@classmethod
def list_launch_plan_ids_paginated(identifier_list_request: flyteidl.admin.common_pb2.NamedEntityIdentifierListRequest) - > flyteidl.admin.common_pb2.NamedEntityIdentifierList
-  Lists launch plan named identifiers for a given project and domain.
- **Parameters**

  - **identifier_list_request**: flyteidl.admin.common_pb2.NamedEntityIdentifierListRequest
    - The request protobuf object for listing named entity identifiers.

- **Return Value**:
**flyteidl.admin.common_pb2.NamedEntityIdentifierList**
  - A paginated list of launch plan identifiers.
@classmethod
def list_launch_plans_paginated(resource_list_request: flyteidl.admin.common_pb2.ResourceListRequest) - > flyteidl.admin.launch_plan_pb2.LaunchPlanList
-  Lists Launch Plans for a given Identifier (project, domain, name).
- **Parameters**

  - **resource_list_request**: flyteidl.admin.common_pb2.ResourceListRequest
    - The request protobuf object for listing resources.

- **Return Value**:
**flyteidl.admin.launch_plan_pb2.LaunchPlanList**
  - A paginated list of launch plans.
@classmethod
def list_active_launch_plans_paginated(active_launch_plan_list_request: flyteidl.admin.common_pb2.ActiveLaunchPlanListRequest) - > flyteidl.admin.launch_plan_pb2.LaunchPlanList
-  Lists Active Launch Plans for a given (project, domain).
- **Parameters**

  - **active_launch_plan_list_request**: flyteidl.admin.common_pb2.ActiveLaunchPlanListRequest
    - The request protobuf object for listing active launch plans.

- **Return Value**:
**flyteidl.admin.launch_plan_pb2.LaunchPlanList**
  - A paginated list of active launch plans.
@classmethod
def update_named_entity(update_named_entity_request: flyteidl.admin.common_pb2.NamedEntityUpdateRequest) - > flyteidl.admin.common_pb2.NamedEntityUpdateResponse
-  Updates a named entity.
- **Parameters**

  - **update_named_entity_request**: flyteidl.admin.common_pb2.NamedEntityUpdateRequest
    - The request protobuf object for updating a named entity.

- **Return Value**:
**flyteidl.admin.common_pb2.NamedEntityUpdateResponse**
  - The response from updating the named entity.
@classmethod
def create_execution(create_execution_request: flyteidl.admin.execution_pb2.ExecutionCreateRequest) - > flyteidl.admin.execution_pb2.ExecutionCreateResponse
-  Creates an execution for the given execution spec.
- **Parameters**

  - **create_execution_request**: flyteidl.admin.execution_pb2.ExecutionCreateRequest
    - The request protobuf object for creating an execution.

- **Return Value**:
**flyteidl.admin.execution_pb2.ExecutionCreateResponse**
  - The response from creating the execution.
@classmethod
def recover_execution(recover_execution_request: flyteidl.admin.execution_pb2.ExecutionRecoverRequest) - > flyteidl.admin.execution_pb2.ExecutionRecoverResponse
-  Recreates an execution with the same spec as the one belonging to the given execution identifier.
- **Parameters**

  - **recover_execution_request**: flyteidl.admin.execution_pb2.ExecutionRecoverRequest
    - The request protobuf object for recovering an execution.

- **Return Value**:
**flyteidl.admin.execution_pb2.ExecutionRecoverResponse**
  - The response from recovering the execution.
@classmethod
def get_execution(get_object_request: flyteidl.admin.execution_pb2.WorkflowExecutionGetRequest) - > flyteidl.admin.execution_pb2.Execution
-  Returns an execution of a workflow entity.
- **Parameters**

  - **get_object_request**: flyteidl.admin.execution_pb2.WorkflowExecutionGetRequest
    - The request protobuf object for getting a workflow execution.

- **Return Value**:
**flyteidl.admin.execution_pb2.Execution**
  - The requested execution object.
@classmethod
def get_execution_data(get_execution_data_request: flyteidl.admin.execution_pb2.WorkflowExecutionGetRequest) - > flyteidl.admin.execution_pb2.WorkflowExecutionGetDataResponse
-  Returns signed URLs to LiteralMap blobs for an execution&#x27;s inputs and outputs (when available).
- **Parameters**

  - **get_execution_data_request**: flyteidl.admin.execution_pb2.WorkflowExecutionGetRequest
    - The request protobuf object for getting execution data.

- **Return Value**:
**flyteidl.admin.execution_pb2.WorkflowExecutionGetDataResponse**
  - The response containing signed URLs for execution data.
@classmethod
def get_execution_metrics(get_execution_metrics_request: flyteidl.admin.execution_pb2.WorkflowExecutionGetMetricsRequest) - > flyteidl.admin.execution_pb2.WorkflowExecutionGetMetricsResponse
-  Returns metrics partitioning and categorizing the workflow execution time-series.
- **Parameters**

  - **get_execution_metrics_request**: flyteidl.admin.execution_pb2.WorkflowExecutionGetMetricsRequest
    - The request protobuf object for getting execution metrics.

- **Return Value**:
**flyteidl.admin.execution_pb2.WorkflowExecutionGetMetricsResponse**
  - The response containing execution metrics.
@classmethod
def list_executions_paginated(resource_list_request: flyteidl.admin.common_pb2.ResourceListRequest) - > flyteidl.admin.execution_pb2.ExecutionList
-  Lists the executions for a given identifier. This is a paginated API.
- **Parameters**

  - **resource_list_request**: flyteidl.admin.common_pb2.ResourceListRequest
    - The request protobuf object for listing resources.

- **Return Value**:
**flyteidl.admin.execution_pb2.ExecutionList**
  - A paginated list of executions.
@classmethod
def terminate_execution(terminate_execution_request: flyteidl.admin.execution_pb2.TerminateExecutionRequest) - > flyteidl.admin.execution_pb2.TerminateExecutionResponse
-  Terminates an execution.
- **Parameters**

  - **terminate_execution_request**: flyteidl.admin.execution_pb2.TerminateExecutionRequest
    - The request protobuf object for terminating an execution.

- **Return Value**:
**flyteidl.admin.execution_pb2.TerminateExecutionResponse**
  - The response from terminating the execution.
@classmethod
def relaunch_execution(relaunch_execution_request: flyteidl.admin.execution_pb2.ExecutionRelaunchRequest) - > flyteidl.admin.execution_pb2.ExecutionCreateResponse
-  Relaunches an execution.
- **Parameters**

  - **relaunch_execution_request**: flyteidl.admin.execution_pb2.ExecutionRelaunchRequest
    - The request protobuf object for relaunching an execution.

- **Return Value**:
**flyteidl.admin.execution_pb2.ExecutionCreateResponse**
  - The response from relaunching the execution.
@classmethod
def get_node_execution(node_execution_request: flyteidl.admin.node_execution_pb2.NodeExecutionGetRequest) - > flyteidl.admin.node_execution_pb2.NodeExecution
-  Retrieves a node execution.
- **Parameters**

  - **node_execution_request**: flyteidl.admin.node_execution_pb2.NodeExecutionGetRequest
    - The request protobuf object for getting a node execution.

- **Return Value**:
**flyteidl.admin.node_execution_pb2.NodeExecution**
  - The requested node execution object.
@classmethod
def get_node_execution_data(get_node_execution_data_request: flyteidl.admin.node_execution_pb2.NodeExecutionGetDataRequest) - > flyteidl.admin.node_execution_pb2.NodeExecutionGetDataResponse
-  Returns signed URLs to LiteralMap blobs for a node execution&#x27;s inputs and outputs (when available).
- **Parameters**

  - **get_node_execution_data_request**: flyteidl.admin.node_execution_pb2.NodeExecutionGetDataRequest
    - The request protobuf object for getting node execution data.

- **Return Value**:
**flyteidl.admin.node_execution_pb2.NodeExecutionGetDataResponse**
  - The response containing signed URLs for node execution data.
@classmethod
def list_node_executions_paginated(node_execution_list_request: flyteidl.admin.node_execution_pb2.NodeExecutionListRequest) - > flyteidl.admin.node_execution_pb2.NodeExecutionList
-  Lists node executions. This is a paginated API.
- **Parameters**

  - **node_execution_list_request**: flyteidl.admin.node_execution_pb2.NodeExecutionListRequest
    - The request protobuf object for listing node executions.

- **Return Value**:
**flyteidl.admin.node_execution_pb2.NodeExecutionList**
  - A paginated list of node executions.
@classmethod
def list_node_executions_for_task_paginated(node_execution_for_task_list_request: flyteidl.admin.node_execution_pb2.NodeExecutionListRequest) - > flyteidl.admin.node_execution_pb2.NodeExecutionList
-  Lists node executions for a specific task. This is a paginated API.
- **Parameters**

  - **node_execution_for_task_list_request**: flyteidl.admin.node_execution_pb2.NodeExecutionListRequest
    - The request protobuf object for listing node executions for a task.

- **Return Value**:
**flyteidl.admin.node_execution_pb2.NodeExecutionList**
  - A paginated list of node executions for a task.
@classmethod
def get_task_execution(task_execution_request: flyteidl.admin.task_execution_pb2.TaskExecutionGetRequest) - > flyteidl.admin.task_execution_pb2.TaskExecution
-  Retrieves a task execution.
- **Parameters**

  - **task_execution_request**: flyteidl.admin.task_execution_pb2.TaskExecutionGetRequest
    - The request protobuf object for getting a task execution.

- **Return Value**:
**flyteidl.admin.task_execution_pb2.TaskExecution**
  - The requested task execution object.
@classmethod
def get_task_execution_data(get_task_execution_data_request: flyteidl.admin.task_execution_pb2.TaskExecutionGetDataRequest) - > flyteidl.admin.task_execution_pb2.TaskExecutionGetDataResponse
-  Returns signed URLs to LiteralMap blobs for a task execution&#x27;s inputs and outputs (when available).
- **Parameters**

  - **get_task_execution_data_request**: flyteidl.admin.task_execution_pb2.TaskExecutionGetDataRequest
    - The request protobuf object for getting task execution data.

- **Return Value**:
**flyteidl.admin.task_execution_pb2.TaskExecutionGetDataResponse**
  - The response containing signed URLs for task execution data.
@classmethod
def list_task_executions_paginated(task_execution_list_request: flyteidl.admin.task_execution_pb2.TaskExecutionListRequest) - > flyteidl.admin.task_execution_pb2.TaskExecutionList
-  Lists task executions. This is a paginated API.
- **Parameters**

  - **task_execution_list_request**: flyteidl.admin.task_execution_pb2.TaskExecutionListRequest
    - The request protobuf object for listing task executions.

- **Return Value**:
**flyteidl.admin.task_execution_pb2.TaskExecutionList**
  - A paginated list of task executions.
@classmethod
def list_projects(project_list_request: typing.Optional[ProjectListRequest] = None) - > flyteidl.admin.project_pb2.Projects
-  Returns a list of the projects registered with the Flyte Admin Service.
- **Parameters**

  - **project_list_request**: typing.Optional[ProjectListRequest]
    - The request protobuf object for listing projects.

- **Return Value**:
**flyteidl.admin.project_pb2.Projects**
  - A list of projects.
@classmethod
def register_project(project_register_request: flyteidl.admin.project_pb2.ProjectRegisterRequest) - > flyteidl.admin.project_pb2.ProjectRegisterResponse
-  Registers a project along with a set of domains.
- **Parameters**

  - **project_register_request**: flyteidl.admin.project_pb2.ProjectRegisterRequest
    - The request protobuf object for registering a project.

- **Return Value**:
**flyteidl.admin.project_pb2.ProjectRegisterResponse**
  - The response from registering the project.
@classmethod
def update_project(project: flyteidl.admin.project_pb2.Project) - > flyteidl.admin.project_pb2.ProjectUpdateResponse
-  Updates an existing project specified by id.
- **Parameters**

  - **project**: flyteidl.admin.project_pb2.Project
    - The project object to update.

- **Return Value**:
**flyteidl.admin.project_pb2.ProjectUpdateResponse**
  - The response from updating the project.
```@classmethod
def get_domains()
```
-  Returns a list of domains registered with the Flyte Admin Service.

- **Return Value**:
**flyteidl.admin.project_pb2.GetDomainsResponse**
  - A list of domains.
@classmethod
def update_project_domain_attributes(project_domain_attributes_update_request: flyteidl.admin.ProjectDomainAttributesUpdateRequest) - > flyteidl.admin.ProjectDomainAttributesUpdateResponse
-  Updates the attributes for a project and domain registered with the Flyte Admin Service.
- **Parameters**

  - **project_domain_attributes_update_request**: flyteidl.admin.ProjectDomainAttributesUpdateRequest
    - The request protobuf object for updating project domain attributes.

- **Return Value**:
**flyteidl.admin.ProjectDomainAttributesUpdateResponse**
  - The response from updating project domain attributes.
@classmethod
def update_workflow_attributes(workflow_attributes_update_request: flyteidl.admin.UpdateWorkflowAttributesRequest) - > flyteidl.admin.WorkflowAttributesUpdateResponse
-  Updates the attributes for a project, domain, and workflow registered with the Flyte Admin Service.
- **Parameters**

  - **workflow_attributes_update_request**: flyteidl.admin.UpdateWorkflowAttributesRequest
    - The request protobuf object for updating workflow attributes.

- **Return Value**:
**flyteidl.admin.WorkflowAttributesUpdateResponse**
  - The response from updating workflow attributes.
@classmethod
def get_project_domain_attributes(project_domain_attributes_get_request: flyteidl.admin.ProjectDomainAttributesGetRequest) - > flyteidl.admin.ProjectDomainAttributesGetResponse
-  Fetches the attributes for a project and domain registered with the Flyte Admin Service.
- **Parameters**

  - **project_domain_attributes_get_request**: flyteidl.admin.ProjectDomainAttributesGetRequest
    - The request protobuf object for getting project domain attributes.

- **Return Value**:
**flyteidl.admin.ProjectDomainAttributesGetResponse**
  - The response containing project domain attributes.
@classmethod
def get_workflow_attributes(workflow_attributes_get_request: flyteidl.admin.GetWorkflowAttributesAttributesRequest) - > flyteidl.admin.WorkflowAttributesGetResponse
-  Fetches the attributes for a project, domain, and workflow registered with the Flyte Admin Service.
- **Parameters**

  - **workflow_attributes_get_request**: flyteidl.admin.GetWorkflowAttributesAttributesRequest
    - The request protobuf object for getting workflow attributes.

- **Return Value**:
**flyteidl.admin.WorkflowAttributesGetResponse**
  - The response containing workflow attributes.
@classmethod
def list_matchable_attributes(matchable_attributes_list_request: flyteidl.admin.ListMatchableAttributesRequest) - > flyteidl.admin.ListMatchableAttributesResponse
-  Fetches the attributes for a specific resource type registered with the Flyte Admin Service.
- **Parameters**

  - **matchable_attributes_list_request**: flyteidl.admin.ListMatchableAttributesRequest
    - The request protobuf object for listing matchable attributes.

- **Return Value**:
**flyteidl.admin.ListMatchableAttributesResponse**
  - The response containing matchable attributes.
@classmethod
def create_upload_location(create_upload_location_request: _dataproxy_pb2.CreateUploadLocationRequest) - > _dataproxy_pb2.CreateUploadLocationResponse
-  Gets a signed URL to be used during fast registration.
- **Parameters**

  - **create_upload_location_request**: _dataproxy_pb2.CreateUploadLocationRequest
    - The request protobuf object for creating an upload location.

- **Return Value**:
**_dataproxy_pb2.CreateUploadLocationResponse**
  - The response containing the upload location URL.
@classmethod
def create_download_location(create_download_location_request: _dataproxy_pb2.CreateDownloadLocationRequest) - > _dataproxy_pb2.CreateDownloadLocationResponse
-  Gets a signed URL for downloading data.
- **Parameters**

  - **create_download_location_request**: _dataproxy_pb2.CreateDownloadLocationRequest
    - The request protobuf object for creating a download location.

- **Return Value**:
**_dataproxy_pb2.CreateDownloadLocationResponse**
  - The response containing the download location URL.
@classmethod
def create_download_link(create_download_link_request: _dataproxy_pb2.CreateDownloadLinkRequest) - > _dataproxy_pb2.CreateDownloadLinkResponse
-  Creates a signed link for downloading data.
- **Parameters**

  - **create_download_link_request**: _dataproxy_pb2.CreateDownloadLinkRequest
    - The request protobuf object for creating a download link.

- **Return Value**:
**_dataproxy_pb2.CreateDownloadLinkResponse**
  - The response containing the download link.
