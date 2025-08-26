
<!--
help_text: ''
key: summary_connecting_to_flyte_admin_1c2d2436-d833-47d2-a3fd-5be7db070021
modules:
- flytekit.clients.friendly.SynchronousFlyteClient
- flytekit.clients.raw.RawSynchronousFlyteClient
- flytekit.remote.remote.FlyteRemote
questions_to_answer: []
type: summary

-->
Connecting to Flyte Admin

Flyte provides multiple ways to connect to its Admin service, which acts as the control plane for managing tasks, workflows, launch plans, and executions. The choice of client depends on the desired level of abstraction and control.

### Connecting with FlyteRemote

For most programmatic interactions with a Flyte backend, the `FlyteRemote` object is the recommended entry point. It offers a high-level, user-friendly interface that abstracts away the complexities of direct gRPC calls and handles common operations like entity fetching, registration, and execution.

To establish a connection, instantiate `FlyteRemote` by providing a `Config` object. The `Config` object specifies the Flyte Admin endpoint and security settings.

```python
from flytekit.remote.remote import FlyteRemote
from flytekit.configuration import Config

# Option 1: Connect to a specific endpoint (e.g., a production cluster)
remote = FlyteRemote.for_endpoint(
    endpoint="your.flyte.admin:port",
    insecure=True,  # Set to True if your Flyte Admin deployment does not have SSL enabled
    default_project="flytesnacks",
    default_domain="development",
    data_upload_location="s3://my-flyte-data-bucket/", # Important for data offloading
)

# Option 2: Auto-detect configuration (e.g., from environment variables or config files)
remote_auto = FlyteRemote.auto(
    default_project="flytesnacks",
    default_domain="development",
)

# Option 3: Connect to a local Flyte sandbox environment
remote_sandbox = FlyteRemote.for_sandbox(
    default_project="flytesnacks",
    default_domain="development",
)
```

When initializing `FlyteRemote`, you can specify `default_project` and `default_domain` for convenience, which will be used for operations if not explicitly overridden. The `data_upload_location` is crucial for specifying where data (e.g., large inputs/outputs) will be staged.

The `interactive_mode_enabled` flag, when set to `True` (or `None` for auto-detection in Jupyter environments), enables features like automatic pickling and uploading of local Python entities for registration.

`FlyteRemote` exposes a `client` property, which provides access to the underlying `SynchronousFlyteClient` for more granular control when needed.

```python
# Access the underlying SynchronousFlyteClient
admin_client = remote.client
```

### Direct gRPC Interaction with SynchronousFlyteClient

For developers requiring direct, low-level gRPC service calls to the Flyte control plane, the `SynchronousFlyteClient` offers a more direct interface than `FlyteRemote`. This client wraps the raw gRPC stubs, providing a slightly more Pythonic API for interacting with Flyte Admin's various endpoints (e.g., Task, Workflow, Launch Plan, Execution, Project, Domain, and Data Proxy services).

Instantiate `SynchronousFlyteClient` by providing the Flyte Admin endpoint and specifying whether the connection should be insecure (e.g., for local development or non-SSL deployments).

```python
from flytekit.clients.sync_client import SynchronousFlyteClient

# Connect to Flyte Admin directly
client = SynchronousFlyteClient("your.flyte.admin:port", insecure=True)

# Example: Create a task definition
from flytekit.models.core.identifier import Identifier, ResourceType
from flytekit.models.task import TaskSpec
from flytekit.models.core.workflow import TaskTemplate
from flytekit.models.core.types import SimpleType, LiteralType
from flytekit.models.interface import TypedInterface

task_id = Identifier(
    resource_type=ResourceType.TASK,
    project="flytesnacks",
    domain="development",
    name="my_example_task",
    version="1.0.0",
)

task_spec = TaskSpec(
    template=TaskTemplate(
        id=task_id,
        type="python-task",
        metadata={},
        interface=TypedInterface(
            inputs={"x": LiteralType(simple=SimpleType.INTEGER)},
            outputs={"y": LiteralType(simple=SimpleType.INTEGER)},
        ),
        container={
            "image": "ghcr.io/flyteorg/flytekit:py3.9-latest",
            "command": ["python"],
            "args": ["-c", "print('hello world')"],
        },
    )
)

try:
    client.create_task(task_id, task_spec)
    print(f"Task {task_id.name} registered successfully.")
except Exception as e:
    print(f"Failed to register task: {e}")

# Example: List tasks
tasks, token = client.list_tasks_paginated(
    identifier=Identifier(project="flytesnacks", domain="development"),
    limit=10,
)
for task in tasks:
    print(f"Found task: {task.id.name} (version: {task.id.version})")
```

`SynchronousFlyteClient` provides methods for:
*   **Task Endpoints:** `create_task`, `list_task_ids_paginated`, `list_tasks_paginated`, `get_task`.
*   **Workflow Endpoints:** `create_workflow`, `list_workflow_ids_paginated`, `list_workflows_paginated`, `get_workflow`.
*   **Launch Plan Endpoints:** `create_launch_plan`, `get_launch_plan`, `get_active_launch_plan`, `list_launch_plan_ids_paginated`, `list_launch_plans_paginated`, `list_active_launch_plans_paginated`, `update_launch_plan`.
*   **Named Entity Endpoints:** `update_named_entity`.
*   **Execution Endpoints:** `create_execution`, `recover_execution`, `get_execution`, `get_execution_data`, `list_executions_paginated`, `terminate_execution`, `relaunch_execution`, `get_execution_metrics`.
*   **Node Execution Endpoints:** `get_node_execution`, `get_node_execution_data`, `list_node_executions`, `list_node_executions_for_task_paginated`.
*   **Task Execution Endpoints:** `get_task_execution`, `get_task_execution_data`, `list_task_executions_paginated`.
*   **Project Endpoints:** `register_project`, `update_project`, `list_projects_paginated`.
*   **Domain Endpoints:** `get_domains`.
*   **Matching Attributes Endpoints:** `update_project_domain_attributes`, `update_workflow_attributes`, `get_project_domain_attributes`, `get_workflow_attributes`, `list_matchable_attributes`.
*   **Data Proxy Endpoints:** `get_upload_signed_url`, `get_download_signed_url`, `get_data`, `get_download_artifact_signed_url`.
*   **Version Endpoint:** `get_control_plane_version`.

Many listing methods are paginated, returning a list of entities and a `token` for fetching the next page of results.

### Underlying RawSynchronousFlyteClient

The `RawSynchronousFlyteClient` serves as the foundational layer for `SynchronousFlyteClient`. It is a thin, synchronous wrapper directly around the auto-generated gRPC stubs for the Flyte Admin service.

While `RawSynchronousFlyteClient` provides the most direct access to the gRPC API, it is generally not intended for direct use by application developers. Its primary purpose is to provide the core communication mechanism for higher-level clients like `SynchronousFlyteClient` and `FlyteRemote`. It handles the gRPC channel setup, including options for message size limits and authentication.

Developers typically interact with `FlyteRemote` or `SynchronousFlyteClient` for a more convenient and robust experience.
<!--
key: summary_connecting_to_flyte_admin_1c2d2436-d833-47d2-a3fd-5be7db070021
type: summary_end

-->
<!--
code_unit: flytekit.clients.friendly.SynchronousFlyteClient
code_unit_type: class
help_text: ''
key: example_0f3f3ada-5590-40c5-b6fa-10c5b03178b9
type: example

-->
<!--
code_unit: flytekit.remote.remote.FlyteRemote
code_unit_type: class
help_text: ''
key: example_c3b9fc49-2b5e-4bac-86dd-b09817a87154
type: example

-->