
<!--
help_text: ''
key: summary_core_client_operations_(low-level)_d5e308fd-734a-4456-9f00-e02486604d44
modules:
- flytekit.clients.friendly
- flytekit.clients.raw
- flytekit.clients.grpc_utils.default_metadata_interceptor
- flytekit.clients.grpc_utils.wrap_exception_interceptor
questions_to_answer: []
type: summary

-->
The `SynchronousFlyteClient` provides a direct, low-level interface for interacting with the Flyte Admin service via gRPC. This client enables programmatic management of Flyte entities such as tasks, workflows, launch plans, and executions, offering fine-grained control over the Flyte platform. It is designed for developers who require direct access to the Flyte Admin API, offering a more user-friendly abstraction than the underlying raw gRPC client.

### Client Initialization

Instantiate the `SynchronousFlyteClient` by providing the Flyte Admin service endpoint and specifying whether to use an insecure connection (e.g., for local development or non-SSL deployments).

```python
from flytekit.clients.friendly import SynchronousFlyteClient
from flytekit.configuration import PlatformConfig
import grpc

# For insecure connection (e.g., local Flyte deployment)
client = SynchronousFlyteClient("your.domain:port", insecure=True)

# For secure connection (production deployments)
# client = SynchronousFlyteClient("your.domain:port", insecure=False)

# For secure connections with a custom root certificate:
# client = SynchronousFlyteClient.with_root_certificate(
#     PlatformConfig(endpoint="your.domain:port", insecure=False),
#     root_cert_file="/path/to/your/root_cert.pem"
# )
```

The client automatically injects default gRPC metadata and implements a retry mechanism for transient errors, enhancing robustness.

### Entity Management

The client provides comprehensive operations for managing core Flyte entities. All creation operations are idempotent; repeated calls with identical definitions will succeed without overwriting existing entities.

#### Tasks

Tasks represent the atomic units of computation in Flyte.

*   **Create a Task:** Define and register a task with the Flyte Admin service.
    ```python
    from flytekit.models.core import identifier
    from flytekit.models import task as task_model

    task_id = identifier.Identifier(
        resource_type=identifier.ResourceType.TASK,
        project="my_project",
        domain="development",
        name="my_task",
        version="v1"
    )
    # task_spec would be constructed from a Flyte task definition
    task_spec = task_model.TaskSpec(...)
    client.create_task(task_id, task_spec)
    ```
    A `FlyteEntityAlreadyExistsException` is raised if an identical task version already exists, though this can often be safely ignored due to idempotency.

*   **Retrieve a Task:** Fetch a specific task definition using its unique identifier.
    ```python
    task = client.get_task(task_id)
    print(task.id.name)
    ```
    This method utilizes an LRU cache for performance optimization, caching recently retrieved task definitions.

*   **List Task Identifiers:** Retrieve paginated lists of task identifiers within a specified project and domain.
    ```python
    task_ids, next_token = client.list_task_ids_paginated(
        project="my_project",
        domain="development",
        limit=50
    )
    for tid in task_ids:
        print(f"Task ID: {tid.name} (version: {tid.version})")
    ```
    Pagination is managed via `limit` and `token` parameters. Be aware that new entries added between paginated requests might result in duplicates across pages.

*   **List Tasks:** Retrieve paginated lists of full task metadata.
    ```python
    from flytekit.models.admin import common as admin_common_models

    tasks, next_token = client.list_tasks_paginated(
        identifier=identifier.NamedEntityIdentifier(project="my_project", domain="development"),
        limit=50,
        sort_by=admin_common_models.Sort(key="name", direction=admin_common_models.Sort.Direction.ASCENDING)
    )
    for task_obj in tasks:
        print(f"Task: {task_obj.id.name}, Type: {task_obj.closure.compiled_task.template.type}")
    ```
    Filters and sorting options are available to refine results.

#### Workflows

Workflows define the execution graph of tasks.

*   **Create a Workflow:** Register a workflow definition. Similar to tasks, creation is idempotent.
    ```python
    from flytekit.models.admin import workflow as workflow_model

    workflow_id = identifier.Identifier(
        resource_type=identifier.ResourceType.WORKFLOW,
        project="my_project",
        domain="development",
        name="my_workflow",
        version="v1"
    )
    # workflow_spec would be constructed from a Flyte workflow definition
    workflow_spec = workflow_model.WorkflowSpec(...)
    client.create_workflow(workflow_id, workflow_spec)
    ```

*   **Retrieve a Workflow:** Fetch a specific workflow definition by its identifier. This method also uses an LRU cache.
    ```python
    workflow = client.get_workflow(workflow_id)
    print(workflow.id.name)
    ```

*   **List Workflow Identifiers and Workflows:** Similar paginated listing methods are available for workflow identifiers (`list_workflow_ids_paginated`) and full workflow metadata (`list_workflows_paginated`).

#### Launch Plans

Launch plans are executable versions of workflows, allowing for parameterization and scheduling.

*   **Create a Launch Plan:** Register a launch plan definition. Creation is idempotent.
    ```python
    from flytekit.models import launch_plan as launch_plan_model

    lp_id = identifier.Identifier(
        resource_type=identifier.ResourceType.LAUNCH_PLAN,
        project="my_project",
        domain="development",
        name="my_launch_plan",
        version="v1"
    )
    # lp_spec would be constructed from a Flyte launch plan definition
    lp_spec = launch_plan_model.LaunchPlanSpec(...)
    client.create_launch_plan(lp_id, lp_spec)
    ```

*   **Retrieve a Launch Plan:** Fetch a specific launch plan by its identifier. This method also uses an LRU cache.
    ```python
    launch_plan = client.get_launch_plan(lp_id)
    print(launch_plan.id.name)
    ```

*   **Retrieve Active Launch Plan:** Get the currently active launch plan for a given project, domain, and name.
    ```python
    active_lp = client.get_active_launch_plan(
        identifier.NamedEntityIdentifier(project="my_project", domain="development", name="my_launch_plan")
    )
    print(f"Active Launch Plan: {active_lp.id.version}")
    ```

*   **List Launch Plan Identifiers and Launch Plans:** Paginated listing methods are available for launch plan identifiers (`list_launch_plan_ids_paginated`), full launch plan metadata (`list_launch_plans_paginated`), and specifically active launch plans (`list_active_launch_plans_paginated`).

*   **Update a Launch Plan:** Modify the state of a launch plan (e.g., `ACTIVE` or `INACTIVE`). Setting a launch plan to `ACTIVE` automatically deactivates any other active launch plans with the same project, domain, and name.
    ```python
    from flytekit.models.launch_plan import LaunchPlanState

    client.update_launch_plan(lp_id, LaunchPlanState.ACTIVE)
    ```

#### Named Entities

Named entities represent a logical grouping of versions for a given resource (task, workflow, or launch plan) across a project and domain.

*   **Update Named Entity Metadata:** Modify metadata (e.g., description, archived status) for a named entity.
    ```python
    from flytekit.models.admin import named_entity
    from flytekit.models.core import identifier as core_identifier

    named_entity_id = named_entity.NamedEntityIdentifier(
        project="my_project",
        domain="development",
        name="my_workflow"
    )
    metadata = named_entity.NamedEntityMetadata(description="Updated description", archived=True)
    client.update_named_entity(core_identifier.ResourceType.WORKFLOW, named_entity_id, metadata)
    ```

#### Projects

Projects serve as the top-level organizational unit in Flyte.

*   **Register a Project:** Create a new project.
    ```python
    from flytekit.models import project as project_model

    new_project = project_model.Project(
        id="new_project_id",
        name="New Project Name",
        description="A new project for documentation."
    )
    client.register_project(new_project)
    ```

*   **Update a Project:** Modify an existing project's details.
    ```python
    updated_project = project_model.Project(
        id="new_project_id",
        name="Updated Project Name",
        description="Revised description for the project."
    )
    client.update_project(updated_project)
    ```

*   **List Projects:** Retrieve paginated lists of registered projects.
    ```python
    projects, next_token = client.list_projects_paginated(limit=10)
    for p in projects:
        print(f"Project: {p.name} ({p.id})")
    ```

#### Domains

Domains are logical subdivisions within a project, often used to separate environments (e.g., `development`, `staging`, `production`).

*   **Get Domains:** Retrieve a list of all registered domains.
    ```python
    domains = client.get_domains()
    for d in domains:
        print(f"Domain: {d.id} ({d.name})")
    ```

### Execution Management

The client provides extensive capabilities for managing the lifecycle and data of workflow, node, and task executions.

#### Workflow Executions

*   **Create an Execution:** Initiate a workflow execution from a launch plan.
    ```python
    from flytekit.models import execution as execution_model
    from flytekit.models import literals

    # Assuming 'lp_id' is an Identifier for a Launch Plan
    lp_id = identifier.Identifier(
        resource_type=identifier.ResourceType.LAUNCH_PLAN,
        project="my_project",
        domain="development",
        name="my_launch_plan",
        version="v1"
    )
    execution_spec = execution_model.ExecutionSpec(
        launch_plan=lp_id,
        # ... other execution spec details like inputs, labels, etc.
    )
    inputs = literals.LiteralMap(literals={}) # Provide actual inputs if required by the launch plan
    exec_id = client.create_execution(
        project="my_project",
        domain="development",
        name="my_first_execution", # Optional, client generates if not provided
        execution_spec=execution_spec,
        inputs=inputs
    )
    print(f"Created execution: {exec_id.name}")
    ```

*   **Recover an Execution:** Re-run a failed or terminated execution, resuming from the last known failure point.
    ```python
    # Assuming 'failed_exec_id' is a WorkflowExecutionIdentifier
    failed_exec_id = identifier.WorkflowExecutionIdentifier(
        project="my_project", domain="development", name="failed_execution_name"
    )
    recovered_exec_id = client.recover_execution(failed_exec_id)
    print(f"Recovered execution: {recovered_exec_id.name}")
    ```

*   **Relaunch an Execution:** Start a new execution with the same parameters as a previous one.
    ```python
    # Assuming 'prev_exec_id' is a WorkflowExecutionIdentifier
    prev_exec_id = identifier.WorkflowExecutionIdentifier(
        project="my_project", domain="development", name="previous_execution_name"
    )
    new_exec_id = client.relaunch_execution(prev_exec_id, name="relaunch_of_prev_exec")
    print(f"Relaunched execution: {new_exec_id.name}")
    ```

*   **Terminate an Execution:** Stop a running workflow execution.
    ```python
    # Assuming 'running_exec_id' is a WorkflowExecutionIdentifier
    running_exec_id = identifier.WorkflowExecutionIdentifier(
        project="my_project", domain="development", name="running_execution_name"
    )
    client.terminate_execution(running_exec_id, cause="User requested termination")
    ```

*   **Get Execution Details:** Retrieve the full metadata for a specific workflow execution.
    ```python
    execution = client.get_execution(exec_id)
    print(f"Execution status: {execution.closure.phase}")
    ```

*   **Get Execution Data:** Obtain signed URLs for accessing the inputs and outputs of a workflow execution.
    ```python
    exec_data = client.get_execution_data(exec_id)
    print(f"Input URI: {exec_data.inputs.uri}")
    print(f"Output URI: {exec_data.outputs.uri}")
    ```

*   **Get Execution Metrics:** Retrieve time-series metrics for a workflow execution.
    ```python
    metrics_span = client.get_execution_metrics(exec_id, depth=5)
    # Process metrics_span object, which contains a trace span
    ```

*   **List Executions:** Retrieve paginated lists of workflow executions within a project and domain.
    ```python
    executions, next_token = client.list_executions_paginated(
        project="my_project",
        domain="development",
        limit=20,
        filters=[], # Optional filters from flytekit.models.filters
        sort_by=None # Optional sorting from flytekit.models.admin.common.Sort
    )
    for e in executions:
        print(f"Execution: {e.id.name}, Phase: {e.closure.phase}")
    ```

#### Node Executions

Node executions represent the execution of individual nodes within a workflow.

*   **Get Node Execution Details:** Retrieve metadata for a specific node execution.
    ```python
    from flytekit.models.core import identifier as core_identifier

    node_exec_id = core_identifier.NodeExecutionIdentifier(
        node_id="n0",
        execution_id=exec_id # Use the WorkflowExecutionIdentifier from above
    )
    node_execution = client.get_node_execution(node_exec_id)
    print(f"Node execution phase: {node_execution.closure.phase}")
    ```

*   **Get Node Execution Data:** Obtain signed URLs for inputs and outputs of a node execution.
    ```python
    node_exec_data = client.get_node_execution_data(node_exec_id)
    print(f"Node output URI: {node_exec_data.outputs.uri}")
    ```

*   **List Node Executions:** Retrieve paginated lists of node executions associated with a workflow execution.
    ```python
    # Assuming 'workflow_exec_id' is a WorkflowExecutionIdentifier
    node_executions, next_token = client.list_node_executions(exec_id)
    for ne in node_executions:
        print(f"Node: {ne.id.node_id}, Phase: {ne.closure.phase}")
    ```
    The `unique_parent_id` parameter can be used to filter node executions for a specific parent node (e.g., for sub-workflows).

*   **List Node Executions for Task:** Retrieve node executions spawned by a specific task execution (common for dynamic tasks).
    ```python
    from flytekit.models.core import identifier as core_identifier

    task_exec_id = core_identifier.TaskExecutionIdentifier(...) # Construct TaskExecutionIdentifier
    child_node_executions, next_token = client.list_node_executions_for_task_paginated(task_exec_id)
    ```

#### Task Executions

Task executions represent the execution of individual tasks within a node.

*   **Get Task Execution Details:** Retrieve metadata for a specific task execution.
    ```python
    from flytekit.models.core import identifier as core_identifier

    task_exec_id = core_identifier.TaskExecutionIdentifier(...) # Construct TaskExecutionIdentifier
    task_execution = client.get_task_execution(task_exec_id)
    print(f"Task execution phase: {task_execution.closure.phase}")
    ```

*   **Get Task Execution Data:** Obtain signed URLs for inputs and outputs of a task execution.
    ```python
    task_exec_data = client.get_task_execution_data(task_exec_id)
    print(f"Task output URI: {task_exec_data.outputs.uri}")
    ```

*   **List Task Executions:** Retrieve paginated lists of task executions associated with a node execution.
    ```python
    # Assuming 'node_exec_id' is a NodeExecutionIdentifier
    task_executions, next_token = client.list_task_executions_paginated(node_exec_id)
    for te in task_executions:
        print(f"Task: {te.id.task_id.name}, Phase: {te.closure.phase}")
    ```

### Matching Attributes

The client allows managing custom attributes that can be matched against resources (projects, domains, workflows) to apply specific configurations.

*   **Update Project/Domain Attributes:** Set custom attributes for a project and domain combination.
    ```python
    from flytekit.models import common as common_models
    from flytekit.models import matchable_resource as matchable_resource_models

    # Example: Set a custom execution queue attribute
    matching_attributes = common_models.MatchingAttributes(
        execution_queue_attributes=matchable_resource_models.ExecutionQueueAttributes(
            tags=["high-priority"]
        )
    )
    client.update_project_domain_attributes("my_project", "development", matching_attributes)
    ```

*   **Update Workflow Attributes:** Set custom attributes for a specific workflow within a project and domain.
    ```python
    # Example: Set a custom cluster resource attribute for a workflow
    matching_attributes = common_models.MatchingAttributes(
        cluster_resource_attributes=matchable_resource_models.ClusterResourceAttributes(
            attrs={"cpu": "2", "memory": "4Gi"}
        )
    )
    client.update_workflow_attributes("my_project", "development", "my_workflow", matching_attributes)
    ```

*   **Get Project/Domain Attributes:** Fetch custom attributes for a project and domain.
    ```python
    from flytekit.models import matchable_resource as matchable_resource_models

    attrs = client.get_project_domain_attributes(
        "my_project", "development", matchable_resource_models.MatchableResource.EXECUTION_QUEUE
    )
    # Access attributes via attrs.attributes.execution_queue_attributes
    ```

*   **Get Workflow Attributes:** Fetch custom attributes for a workflow.
    ```python
    attrs = client.get_workflow_attributes(
        "my_project", "development", "my_workflow", matchable_resource_models.MatchableResource.CLUSTER_RESOURCE
    )
    # Access attributes via attrs.attributes.cluster_resource_attributes
    ```

*   **List Matchable Attributes:** Retrieve all custom attributes defined for a specific resource type.
    ```python
    matchable_attrs = client.list_matchable_attributes(matchable_resource_models.MatchableResource.EXECUTION_QUEUE)
    for attr in matchable_attrs.all_attributes:
        print(f"Matchable Attribute: {attr}")
    ```

### Data Proxy Operations

The client provides direct access to the Flyte Data Proxy service for managing data artifacts.

*   **Get Upload Signed URL:** Obtain a pre-signed URL for uploading data to Flyte's configured blob storage. This is useful for fast registration workflows.
    ```python
    import datetime

    upload_response = client.get_upload_signed_url(
        project="my_project",
        domain="development",
        filename="my_data.csv",
        expires_in=datetime.timedelta(minutes=10)
    )
    print(f"Upload URL: {upload_response.signed_url}")
    ```

*   **Get Download Signed URL:** Obtain a pre-signed URL for downloading data from a Flyte-managed URI.
    ```python
    download_response = client.get_download_signed_url(
        native_url="s3://my-bucket/flyte/my_data.csv",
        expires_in=datetime.timedelta(minutes=10)
    )
    print(f"Download URL: {download_response.signed_url}")
    ```

*   **Get Data:** Retrieve data directly from a Flyte URI.
    ```python
    data_response = client.get_data("flyte://my_project/development/my_execution/outputs/output_data")
    # data_response.content contains the raw data as bytes
    ```

*   **Get Download Artifact Signed URL:** Obtain a pre-signed URL for downloading specific artifacts (like decks) associated with a node execution.
    ```python
    from flyteidl.service import dataproxy_pb2
    from flytekit.models.core import identifier as core_identifier

    node_exec_id = core_identifier.NodeExecutionIdentifier(
        node_id="n0",
        execution_id=core_identifier.WorkflowExecutionIdentifier(
            project="my_project", domain="development", name="my_execution_name"
        )
    )
    artifact_url_response = client.get_download_artifact_signed_url(
        node_id=node_exec_id.node_id,
        project=node_exec_id.execution_id.project,
        domain=node_exec_id.execution_id.domain,
        name=node_exec_id.execution_id.name,
        artifact_type=dataproxy_pb2.ARTIFACT_TYPE_DECK, # Example: ARTIFACT_TYPE_DECK for a deck artifact
        expires_in=datetime.timedelta(minutes=5)
    )
    print(f"Deck URL: {artifact_url_response.signed_url}")
    ```

### Control Plane Versioning

*   **Retrieve Control Plane Version:** Get the version string of the Flyte Admin service. This can be useful for feature gating or compatibility checks.
    ```python
    version = client.get_control_plane_version()
    print(f"Flyte Admin Version: {version}")
    ```

### Advanced Considerations

#### Raw Client Access

The `SynchronousFlyteClient` provides a `raw` property to access the underlying `RawSynchronousFlyteClient`. This is generally not necessary for most use cases, as the friendly client wraps the raw client's methods with more Pythonic interfaces and model conversions. However, it can be useful for debugging or when direct interaction with the protobuf request/response objects is preferred.

```python
raw_client = client.raw
# You can now call raw gRPC methods directly, e.g.,
# from flyteidl.admin import task_pb2
# raw_client.CreateTask(task_pb2.TaskCreateRequest(...))
```

#### Error Handling

The client incorporates `RetryExceptionWrapperInterceptor` to automatically retry certain gRPC errors. Specific Flyte-related exceptions are raised for common gRPC status codes:
*   `FlyteAuthenticationException` for `UNAUTHENTICATED`
*   `FlyteEntityAlreadyExistsException` for `ALREADY_EXISTS`
*   `FlyteEntityNotExistException` for `NOT_FOUND`
*   `FlyteInvalidInputException` for `INVALID_ARGUMENT`
*   `FlyteSystemUnavailableException` for `UNAVAILABLE`
Other gRPC errors or unexpected issues will raise a generic `FlyteSystemException`. Implement robust error handling in your application to gracefully manage these exceptions.

#### Pagination

Many listing operations are paginated. Always check the returned `token` to determine if more results are available. If a `token` is returned, pass it in the subsequent request to fetch the next page. Be aware that concurrent modifications to the database might lead to duplicate entries appearing across different pages.
<!--
key: summary_core_client_operations_(low-level)_d5e308fd-734a-4456-9f00-e02486604d44
type: summary_end

-->
<!--
code_unit: flytekit.clients.friendly.SynchronousFlyteClient.create_task
code_unit_type: class
help_text: ''
key: example_e35e740f-18fc-4213-a214-c04f0855ad3f
type: example

-->
<!--
code_unit: flytekit.clients.friendly.SynchronousFlyteClient.get_workflow
code_unit_type: class
help_text: ''
key: example_a03839f2-21fe-4673-b597-31b9494439f1
type: example

-->
<!--
code_unit: flytekit.clients.friendly.SynchronousFlyteClient.list_executions_paginated
code_unit_type: class
help_text: ''
key: example_6aaaf92c-1c8a-4962-a1cd-9030e56467ee
type: example

-->
<!--
code_unit: flytekit.clients.raw.RawSynchronousFlyteClient
code_unit_type: class
help_text: ''
key: example_02cd88a9-d119-40ab-9222-efe1dd5d6ab1
type: example

-->