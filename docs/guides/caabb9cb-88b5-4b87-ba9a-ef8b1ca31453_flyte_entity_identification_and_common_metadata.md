
<!--
help_text: ''
key: summary_flyte_entity_identification_and_common_metadata_a0dfd041-4203-4194-b447-d9a50b8d44ea
modules:
- flytekit.models.core.identifier
- flytekit.models.common
questions_to_answer: []
type: summary

-->
Flyte provides a robust system for uniquely identifying its various components and associating common metadata with them. This system is fundamental to how Flyte tracks, manages, and interacts with tasks, workflows, launch plans, and their executions across different projects and domains.

### Core Entity Identifiers

Flyte distinguishes between identifiers for *registered definitions* (like a specific version of a task or workflow) and identifiers for *runtime executions* (like a particular run of a workflow). All core identifier classes inherit from `FlyteIdlEntity`, enabling seamless serialization and deserialization to and from Flyte's internal IDL (Interface Definition Language) protobufs.

#### Identifier

The `Identifier` class serves as the primary mechanism for uniquely identifying registered Flyte entities. These entities are typically versioned and represent the blueprint for execution.

An `Identifier` is composed of:
*   `resource_type`: Specifies the type of Flyte entity. This is an integer value corresponding to the `ResourceType` enum.
*   `project`: The project name under which the entity is registered.
*   `domain`: The domain within the project where the entity resides.
*   `name`: The unique name of the entity within its project, domain, and resource type.
*   `version`: The specific version of the entity.

The `ResourceType` enum defines the supported entity types:
*   `UNSPECIFIED`: Default or unknown type.
*   `TASK`: Identifies a task definition.
*   `WORKFLOW`: Identifies a workflow definition.
*   `LAUNCH_PLAN`: Identifies a launch plan definition.

**Usage Example:**
To identify a specific version of a task:

```python
from flytekit.models.core.identifier import Identifier, ResourceType

task_id = Identifier(
    resource_type=ResourceType.TASK,
    project="my_project",
    domain="development",
    name="my_data_processing_task",
    version="v1.0.0"
)
print(f"Task Identifier: {task_id}")
# Output: Flyte Serialized object (Identifier):
#   resource_type: TASK
#   project: my_project
#   domain: development
#   name: my_data_processing_task
#   version: v1.0.0
```

#### NamedEntityIdentifier

The `NamedEntityIdentifier` provides a more general way to identify entities that are named within a project and domain, but may not have a specific version or a strict resource type like those managed by the `Identifier` class. It is often used for administrative or conceptual grouping.

A `NamedEntityIdentifier` includes:
*   `project`: The project name.
*   `domain`: The domain name.
*   `name`: The name of the entity.

**Usage Example:**
```python
from flytekit.models.common import NamedEntityIdentifier

named_entity = NamedEntityIdentifier(
    project="my_project",
    domain="development",
    name="my_named_group"
)
print(f"Named Entity Identifier: {named_entity}")
```

#### WorkflowExecutionIdentifier

The `WorkflowExecutionIdentifier` uniquely identifies a specific *execution* (run) of a workflow. Unlike `Identifier`, it does not include a `version` as it refers to an instance, not a definition.

A `WorkflowExecutionIdentifier` is composed of:
*   `project`: The project where the workflow execution occurred.
*   `domain`: The domain within the project.
*   `name`: The unique name (ID) of the workflow execution.

**Usage Example:**
```python
from flytekit.models.core.identifier import WorkflowExecutionIdentifier

wf_exec_id = WorkflowExecutionIdentifier(
    project="my_project",
    domain="development",
    name="wf_run_20231027_abc123"
)
print(f"Workflow Execution Identifier: {wf_exec_id}")
```

#### NodeExecutionIdentifier

The `NodeExecutionIdentifier` uniquely identifies a specific *execution* of a node within a workflow execution. Nodes represent individual steps or sub-workflows within a larger workflow.

A `NodeExecutionIdentifier` includes:
*   `node_id`: The unique ID of the node within its parent workflow.
*   `execution_id`: The `WorkflowExecutionIdentifier` of the workflow that owns this node execution.

**Usage Example:**
```python
from flytekit.models.core.identifier import NodeExecutionIdentifier, WorkflowExecutionIdentifier

wf_exec_id = WorkflowExecutionIdentifier(project="my_project", domain="development", name="wf_run_20231027_abc123")
node_exec_id = NodeExecutionIdentifier(
    node_id="my_task_node",
    execution_id=wf_exec_id
)
print(f"Node Execution Identifier: {node_exec_id}")
```

#### TaskExecutionIdentifier

The `TaskExecutionIdentifier` uniquely identifies a specific *execution* of a task, which occurs within a node execution. This identifier also accounts for retry attempts.

A `TaskExecutionIdentifier` is composed of:
*   `task_id`: The `Identifier` of the task definition that is being executed.
*   `node_execution_id`: The `NodeExecutionIdentifier` of the node that owns this task execution.
*   `retry_attempt`: An integer indicating the retry attempt number (0 for the first attempt).

**Usage Example:**
```python
from flytekit.models.core.identifier import TaskExecutionIdentifier, NodeExecutionIdentifier, WorkflowExecutionIdentifier, Identifier, ResourceType

task_def_id = Identifier(ResourceType.TASK, "my_project", "development", "my_task", "v1")
wf_exec_id = WorkflowExecutionIdentifier("my_project", "development", "wf_run_20231027_abc123")
node_exec_id = NodeExecutionIdentifier("my_task_node", wf_exec_id)

task_exec_id = TaskExecutionIdentifier(
    task_id=task_def_id,
    node_execution_id=node_exec_id,
    retry_attempt=0
)
print(f"Task Execution Identifier: {task_exec_id}")
```

#### SignalIdentifier

The `SignalIdentifier` is used to uniquely identify a signal within a workflow execution. Signals are used for external interactions, such as waiting for user input or an external event.

A `SignalIdentifier` includes:
*   `signal_id`: A user-provided name for the signal.
*   `execution_id`: The `WorkflowExecutionIdentifier` of the workflow execution to which this signal belongs.

**Usage Example:**
```python
from flytekit.models.core.identifier import SignalIdentifier, WorkflowExecutionIdentifier

wf_exec_id = WorkflowExecutionIdentifier("my_project", "development", "wf_run_20231027_abc123")
signal_id = SignalIdentifier(
    signal_id="user_approval_signal",
    execution_id=wf_exec_id
)
print(f"Signal Identifier: {signal_id}")
```

### Common Metadata and Configuration

Flyte entities can be enriched with various types of metadata and configuration, providing additional context, control, and integration capabilities. These common models are typically found in the `flytekit.models.common` package.

#### FlyteIdlEntity

The `FlyteIdlEntity` class serves as the abstract base class for almost all Flyte models. It provides foundational capabilities for:
*   **Serialization and Deserialization**: The `to_flyte_idl()` and `from_flyte_idl()` methods are crucial for converting Python objects to and from their Protobuf representations, enabling communication with the Flyte backend.
*   **Equality and Hashing**: Implements `__eq__`, `__ne__`, and `__hash__` based on the underlying Protobuf representation, ensuring consistent object comparison.
*   **String Representation**: Provides `short_string()` and `verbose_string()` for human-readable output, and `_repr_html_()` for rich display in environments like Jupyter notebooks.

Developers should leverage `to_flyte_idl()` when interacting with Flyte's gRPC APIs and `from_flyte_idl()` when processing responses from the Flyte backend.

#### Labels and Annotations

`Labels` and `Annotations` allow attaching arbitrary key-value metadata to Flyte resources, particularly workflow executions.
*   **Labels**: Intended for identifying and grouping resources. They are typically short, descriptive, and used for querying and filtering.
*   **Annotations**: Intended for non-identifying, descriptive metadata. They can be longer and more detailed, providing additional context for human consumption or external tools.

Both classes store their data in a `values` dictionary.

**Usage Example:**
```python
from flytekit.models.common import Labels, Annotations

labels = Labels({"team": "data_science", "environment": "production"})
annotations = Annotations({"description": "Daily ETL job for sales data", "owner_email": "team@example.com"})

# These can be attached to workflow execution configurations, for instance.
```

#### AuthRole

The `AuthRole` class specifies the identity under which a Flyte execution (e.g., a workflow or task) will run. This is critical for managing permissions and access to external resources.

It supports two primary mechanisms:
*   `assumable_iam_role`: An AWS IAM role ARN that Flyte will assume for the execution.
*   `kubernetes_service_account`: A Kubernetes service account name that will be used for the execution's pods.

Either or both can be specified.

**Usage Example:**
```python
from flytekit.models.common import AuthRole

auth_role = AuthRole(
    assumable_iam_role="arn:aws:iam::123456789012:role/FlyteExecutionRole",
    kubernetes_service_account="flyte-executor-sa"
)
```

#### Notifications

Flyte supports various notification types to alert users about the status of workflow executions. The `Notification` class aggregates different notification channels, allowing users to configure alerts for specific execution phases.

*   `EmailNotification`: Sends notifications to a list of email recipients.
*   `PagerDutyNotification`: Sends notifications to PagerDuty.
*   `SlackNotification`: Sends notifications to Slack.

The `Notification` class takes a list of `phases` (e.g., `ExecutionPhase.SUCCEEDED`, `ExecutionPhase.FAILED`) to trigger the configured alerts.

**Usage Example:**
```python
from flytekit.models.common import Notification, EmailNotification, SlackNotification
# Assuming ExecutionPhase is imported from flytekit.models.core.execution
from flytekit.models.core.execution import ExecutionPhase

email_notif = EmailNotification(recipients_email=["devs@example.com"])
slack_notif = SlackNotification(recipients_email=["#flyte-alerts"]) # Slack uses email for channel mapping

notification_config = Notification(
    phases=[ExecutionPhase.SUCCEEDED, ExecutionPhase.FAILED],
    email=email_notif,
    slack=slack_notif
)
```

#### Envs

The `Envs` class allows defining environment variables that will be set for the containers running tasks or workflows. This is useful for passing configuration or runtime parameters.

**Usage Example:**
```python
from flytekit.models.common import Envs

env_vars = Envs({"MY_API_KEY": "some_secret_value", "DEBUG_MODE": "true"})
```

#### RawOutputDataConfig

`RawOutputDataConfig` specifies the location where raw output data (e.g., large files, datasets) should be stored. This is particularly relevant for offloading data to external storage systems like S3.

*   `output_location_prefix`: The URI prefix for the storage location (e.g., `s3://my-bucket/flyte-outputs/`).

**Usage Example:**
```python
from flytekit.models.common import RawOutputDataConfig

output_config = RawOutputDataConfig(output_location_prefix="s3://my-data-lake/flyte-results/")
```

#### UrlBlob

The `UrlBlob` class represents a piece of data accessible via a URL, along with its size in bytes. This is a simple model for referencing external data.

**Usage Example:**
```python
from flytekit.models.common import UrlBlob

data_blob = UrlBlob(url="https://example.com/my_data.csv", bytes=102400)
```

### Practical Considerations and Best Practices

*   **Immutability of Identifiers**: Once an `Identifier` or an execution identifier is created, it represents a fixed reference. The properties of these identifiers should not be changed after creation.
*   **Serialization and Interoperability**: The `to_flyte_idl()` and `from_flyte_idl()` methods are the primary integration points for interacting with the Flyte Admin service. Understanding their role is crucial for building custom clients or extending Flyte's capabilities.
*   **Consistent Naming**: Adhere to consistent naming conventions for `project`, `domain`, `name`, and `version` across your Flyte entities. This improves discoverability, organization, and simplifies management.
*   **Execution Context**: When working with execution identifiers (`WorkflowExecutionIdentifier`, `NodeExecutionIdentifier`, `TaskExecutionIdentifier`), remember that they form a hierarchical structure. A `TaskExecutionIdentifier` depends on a `NodeExecutionIdentifier`, which in turn depends on a `WorkflowExecutionIdentifier`.
*   **Metadata for Observability**: Leverage `Labels` and `Annotations` to enrich your workflow executions with business-specific metadata. This can be invaluable for monitoring, cost allocation, and debugging in production environments.
*   **Security with `AuthRole`**: Always configure `AuthRole` appropriately to ensure that your Flyte executions run with the necessary, and only the necessary, permissions. This is a critical security best practice.
<!--
key: summary_flyte_entity_identification_and_common_metadata_a0dfd041-4203-4194-b447-d9a50b8d44ea
type: summary_end

-->
<!--
code_unit: flytekit.models.core.identifier.Identifier
code_unit_type: class
help_text: ''
key: example_0bc8dbb6-8c98-4cc1-b918-1a3bb1da4cf7
type: example

-->
<!--
code_unit: flytekit.models.core.identifier.WorkflowExecutionIdentifier
code_unit_type: class
help_text: ''
key: example_5053b8ab-d198-4f4a-869a-2ff360e56348
type: example

-->
<!--
code_unit: flytekit.models.common.Labels
code_unit_type: class
help_text: ''
key: example_111ee0bb-c0b4-4335-b5bf-9f0930c9eebc
type: example

-->
<!--
code_unit: flytekit.models.common.AuthRole
code_unit_type: class
help_text: ''
key: example_34aeb6e2-f899-455d-ad3e-2a5d83c59e33
type: example

-->