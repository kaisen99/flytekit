
<!--
help_text: ''
key: summary_advanced_client_features_79d77580-7a2f-48e8-9898-875eda52fb1c
modules:
- flytekit.clients.friendly.SynchronousFlyteClient
- flytekit.clients.grpc_utils.auth_interceptor.AuthUnaryInterceptor
- flytekit.clients.grpc_utils.default_metadata_interceptor.DefaultMetadataInterceptor
- flytekit.clients.grpc_utils.wrap_exception_interceptor.RetryExceptionWrapperInterceptor
questions_to_answer: []
type: summary

-->
# Advanced Client Features

The client provides a low-level interface for direct gRPC service calls to the Flyte control plane (Admin service). This client is designed for advanced users and programmatic integrations requiring fine-grained control over Flyte entities and executions.

## Client Initialization

To interact with the Flyte Admin service, instantiate the client by providing the service address. For deployments without SSL enabled, set `insecure` to `True`.

```python
from flytekit.clients.synchronous import SynchronousFlyteClient

# Example: Connect to a local Flyte Admin service
client = SynchronousFlyteClient("your.domain:port", insecure=True)
```

The client exposes a `raw` property to access the underlying `_RawSynchronousFlyteClient` for even lower-level gRPC interactions, though the primary client methods are generally preferred.

## Managing Flyte Entities

The client offers comprehensive capabilities for creating, retrieving, and listing various Flyte entities.

### Task Management

Tasks represent the atomic units of computation in Flyte.

*   **Creating Tasks:**
    Use `create_task` to register a new task definition. This operation is idempotent; calling it multiple times with an identical task definition will succeed without error. Overwrites are not supported; any existing task with the same identifier (project, domain, name, version) must match the new definition exactly.

    ```python
    from flytekit.models.core import identifier
    from flytekit.models import task

    task_id = identifier.Identifier(
        resource_type=identifier.ResourceType.TASK,
        project="my_project",
        domain="development",
        name="my_task",
        version="v1"
    )
    task_spec = task.TaskSpec(...) # Construct your TaskSpec
    client.create_task(task_id, task_spec)
    ```

*   **Retrieving Tasks:**
    Retrieve a specific task definition using `get_task` by providing its unique identifier.

    ```python
    retrieved_task = client.get_task(task_id)
    ```

*   **Listing Tasks:**
    The client provides paginated APIs for listing task identifiers and full task metadata.
    *   `list_task_ids_paginated`: Returns a page of task identifiers.
    *   `list_tasks_paginated`: Returns a page of detailed task metadata.

    Both methods support `limit`, `token` for pagination, and `sort_by` for ordering results. `list_tasks_paginated` also accepts `filters` for more specific queries. When iterating through paginated results, use the `token` returned from the previous call to fetch the next page.

    ```python
    from flytekit.models.common import NamedEntityIdentifier

    project = "my_project"
    domain = "development"
    limit = 10
    token = None

    # List task IDs
    task_ids, next_token = client.list_task_ids_paginated(project, domain, limit=limit)
    print(f"Task IDs: {[t.name for t in task_ids]}")

    # List tasks with metadata
    named_entity_id = NamedEntityIdentifier(project=project, domain=domain, name="my_task")
    tasks, next_token = client.list_tasks_paginated(named_entity_id, limit=limit)
    print(f"Tasks: {[t.id.name for t in tasks]}")
    ```

### Workflow Management

Workflows define the directed acyclic graphs (DAGs) of tasks. The client provides similar operations for workflows as for tasks.

*   `create_workflow`: Registers a workflow definition. This operation is also idempotent and does not support overwrites.
*   `get_workflow`: Retrieves a specific workflow by its identifier.
*   `list_workflow_ids_paginated`: Lists workflow identifiers.
*   `list_workflows_paginated`: Lists detailed workflow metadata.

### Launch Plan Management

Launch plans are executable versions of workflows, often used for scheduling or triggering workflows with predefined inputs.

*   `create_launch_plan`: Registers a launch plan definition. This operation is idempotent.
*   `get_launch_plan`: Retrieves a specific launch plan by its identifier.
*   `get_active_launch_plan`: Retrieves the currently active launch plan for a given project, domain, and name. This is useful for fetching the latest or currently scheduled version.
*   `list_launch_plan_ids_paginated`: Lists launch plan identifiers.
*   `list_launch_plans_paginated`: Lists detailed launch plan metadata.
*   `list_active_launch_plans_paginated`: Lists only currently active launch plans.
*   `update_launch_plan`: Modifies a launch plan's state (e.g., `ACTIVE` or `INACTIVE`). When a launch plan is set to `ACTIVE`, any other active launch plan with the same project, domain, and name is automatically switched to `INACTIVE` in a single transaction.

    ```python
    from flytekit.models.launch_plan import LaunchPlanState

    # Assuming launch_plan_id is an Identifier object
    client.update_launch_plan(launch_plan_id, LaunchPlanState.ACTIVE)
    ```

### Named Entity Management

A named entity refers to a resource (task, workflow, or launch plan) identified by its project, domain, and name, spanning all its versions.

*   `update_named_entity`: Updates metadata (e.g., description) associated with a named entity. This metadata applies across all versions of the resource.

    ```python
    from flytekit.models.admin.named_entity import NamedEntityIdentifier, NamedEntityMetadata
    from flytekit.models.core.identifier import ResourceType

    named_entity_id = NamedEntityIdentifier(project="my_project", domain="development", name="my_workflow")
    metadata = NamedEntityMetadata(description="Updated description for my workflow.")
    client.update_named_entity(ResourceType.WORKFLOW, named_entity_id, metadata)
    ```

## Managing Executions

The client provides extensive capabilities for managing workflow, node, and task executions.

### Workflow Execution Management

*   `create_execution`: Initiates a new workflow execution based on an `ExecutionSpec` and `LiteralMap` inputs.
*   `get_execution`: Retrieves the status and details of a specific workflow execution.
*   `get_execution_data`: Fetches signed URLs for accessing the input and output `LiteralMap` blobs of a workflow execution. This is crucial for retrieving large data payloads.
*   `list_executions_paginated`: Lists workflow executions within a given project and domain, with support for pagination, filters, and sorting.
*   `terminate_execution`: Stops a running workflow execution.
*   `recover_execution`: Recreates a previously failed workflow execution, resuming from the last known failure point. This is useful for debugging and resuming long-running processes.
*   `relaunch_execution`: Creates a new workflow execution based on a previous one. This is useful for re-running an entire workflow.
*   `get_execution_metrics`: Retrieves execution metrics, including span information, for a workflow execution.

### Node Execution Management

Node executions represent the execution of individual nodes within a workflow.

*   `get_node_execution`: Retrieves details for a specific node execution.
*   `get_node_execution_data`: Fetches signed URLs for accessing the input and output `LiteralMap` blobs of a node execution.
*   `list_node_executions`: Lists node executions associated with a given workflow execution. It supports filtering by `unique_parent_id` to retrieve child node executions within a parent node (e.g., for dynamic workflows).
*   `list_node_executions_for_task_paginated`: Lists node executions spawned by a specific task execution, typically used for dynamic tasks that launch sub-workflows or other tasks.

### Task Execution Management

Task executions represent the execution of individual tasks within a node.

*   `get_task_execution`: Retrieves details for a specific task execution.
*   `get_task_execution_data`: Fetches signed URLs for accessing the input and output `LiteralMap` blobs of a task execution.
*   `list_task_executions_paginated`: Lists task executions associated with a given node execution.

## Project and Domain Management

The client allows interaction with project and domain entities within Flyte.

### Project Management

*   `register_project`: Registers a new project in Flyte.
*   `update_project`: Updates details of an existing project.
*   `list_projects_paginated`: Lists registered projects with pagination, filters, and sorting.

### Domain Management

*   `get_domains`: Retrieves a list of all configured domains in the Flyte deployment.

## Advanced Configuration and Data Access

### Matching Attributes

Matching attributes allow setting custom configurations at different levels of the Flyte hierarchy (project, project-domain, workflow). These attributes can influence how tasks and workflows are executed.

*   `update_project_domain_attributes`: Sets custom attributes for a specific project and domain combination.
*   `update_workflow_attributes`: Sets custom attributes for a specific project, domain, and workflow combination.
*   `get_project_domain_attributes`: Fetches custom attributes set for a project and domain.
*   `get_workflow_attributes`: Fetches custom attributes set for a project, domain, and workflow.
*   `list_matchable_attributes`: Lists all custom attributes configured for a given resource type.

### Data Proxy Operations

The client provides methods to interact with the Flyte data proxy, enabling secure and efficient data transfer.

*   `get_upload_signed_url`: Obtains a signed URL for uploading data to a storage backend (e.g., S3, GCS). This is commonly used during fast registration to upload serialized Flyte entities.
*   `get_download_signed_url`: Obtains a signed URL for downloading data from a given native URL (e.g., a direct S3 path).
*   `get_data`: Retrieves data directly from a Flyte URI.
*   `get_download_artifact_signed_url`: Obtains a signed URL for downloading specific artifacts associated with a node execution, such as a `deck` (a visualization output).

### Control Plane Version Retrieval

*   `get_control_plane_version`: Retrieves the version string of the Flyte Admin service. This is useful for enabling or disabling client-side features based on the capabilities of the connected Flyte Admin instance.

## Client Interceptors

The client leverages gRPC interceptors to enhance its functionality, providing features like authentication, default metadata injection, and robust error handling with retries.

### Authentication Interceptor (`AuthUnaryInterceptor`)

This interceptor automatically adds authentication metadata to every gRPC call. It supports lazy authentication and automatically refreshes credentials upon encountering `UNAUTHENTICATED` or `UNKNOWN` gRPC status codes, then retries the request. This ensures secure and uninterrupted communication with the Flyte Admin service.

### Default Metadata Interceptor (`DefaultMetadataInterceptor`)

This interceptor injects standard gRPC metadata, such as `("accept", "application/grpc")`, into all outgoing requests. This ensures that the client's requests are properly formatted and understood by the gRPC server.

### Retry Exception Wrapper Interceptor (`RetryExceptionWrapperInterceptor`)

This interceptor wraps generic gRPC errors into more specific and actionable Flyte exceptions, simplifying error handling for developers. It also implements a retry mechanism for transient errors, attempting a request up to a configurable maximum number of retries (defaulting to 3).

The specific exceptions raised include:
*   `FlyteAuthenticationException`: For `UNAUTHENTICATED` errors.
*   `FlyteEntityAlreadyExistsException`: For `ALREADY_EXISTS` errors, indicating an attempt to create an entity that already exists with a different definition.
*   `FlyteEntityNotExistException`: For `NOT_FOUND` errors.
*   `FlyteInvalidInputException`: For `INVALID_ARGUMENT` errors, indicating malformed or invalid request parameters.
*   `FlyteSystemUnavailableException`: For `UNAVAILABLE` errors, suggesting a temporary service disruption.
*   `FlyteSystemException`: A general exception for other system-level errors.

This interceptor significantly improves the robustness of client interactions by automatically handling common transient issues and providing clearer error messages.

## Important Considerations and Best Practices

*   **Pagination Handling:** Always check the `token` returned by paginated list methods. If a `token` is present, it indicates more results are available, and you should use this `token` in the subsequent request to fetch the next page. Be aware that if entries are added to the database between requests for different pages, it is possible to receive duplicate entries.
*   **Idempotency:** Creation methods for tasks, workflows, and launch plans are designed to be idempotent. This means you can safely call them multiple times with the same input without causing unintended side effects, as long as the definition remains identical.
*   **Error Handling:** Implement robust error handling by catching the specific Flyte exceptions raised by the client. This allows for differentiated responses to various error conditions, such as retrying on transient failures or notifying users about invalid inputs.
*   **Resource Identifiers:** Pay close attention to the various identifier models (`Identifier`, `NamedEntityIdentifier`, `WorkflowExecutionIdentifier`, `NodeExecutionIdentifier`, `TaskExecutionIdentifier`) and their appropriate usage for targeting specific Flyte resources.
*   **Performance:** For operations involving large lists, always utilize the pagination parameters (`limit`, `token`) to manage memory usage and network load. The client internally uses `lru_cache` for `get_task` and `get_workflow` to optimize repeated lookups of the same entity.
<!--
key: summary_advanced_client_features_79d77580-7a2f-48e8-9898-875eda52fb1c
type: summary_end

-->
<!--
code_unit: flytekit.clients.friendly.SynchronousFlyteClient.update_named_entity
code_unit_type: class
help_text: ''
key: example_f01cc431-51f8-430a-bffb-1b8ddb67a751
type: example

-->
<!--
code_unit: flytekit.clients.friendly.SynchronousFlyteClient.update_project_domain_attributes
code_unit_type: class
help_text: ''
key: example_4e5209de-4188-48d6-932c-867f694f958a
type: example

-->
<!--
code_unit: flytekit.clients.friendly.SynchronousFlyteClient.update_workflow_attributes
code_unit_type: class
help_text: ''
key: example_12ffc811-7e55-4f3a-bcb0-03fc5ef52c36
type: example

-->
<!--
code_unit: flytekit.clients.friendly.SynchronousFlyteClient.get_project_domain_attributes
code_unit_type: class
help_text: ''
key: example_49ae228a-0b7e-4f8d-af88-a06f4f8496d6
type: example

-->
<!--
code_unit: flytekit.clients.friendly.SynchronousFlyteClient.get_workflow_attributes
code_unit_type: class
help_text: ''
key: example_90fdb937-0fd5-4dd4-9e1d-797271c4a94d
type: example

-->
<!--
code_unit: flytekit.clients.friendly.SynchronousFlyteClient.list_matchable_attributes
code_unit_type: class
help_text: ''
key: example_74af8c43-797d-4258-aea8-ec1e1fa6d869
type: example

-->
<!--
code_unit: flytekit.clients.friendly.SynchronousFlyteClient.get_control_plane_version
code_unit_type: class
help_text: ''
key: example_22eba200-161e-4376-8cb2-25362ae55f54
type: example

-->