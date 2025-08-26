
<!--
help_text: ''
key: summary_launch_plan_and_execution_options_ae8cfdfd-02ee-4977-bf8b-5ae79d0af28e
modules:
- flytekit.core.options
- flytekit.models.launch_plan
- flytekit.models.common
questions_to_answer: []
type: summary

-->
## Launch Plan and Execution Options

Launch Plans provide a robust mechanism for defining and managing the execution of workflows. They encapsulate a workflow's identity, default parameters, and a comprehensive set of execution options, enabling repeatable, parameterized, and scheduled runs.

### Configuring Execution Options

The `Options` class defines a set of configurable parameters that can be applied to a Launch Plan during its registration or overridden at the time of execution. These options control various aspects of how a workflow run behaves in the Flyte backend.

Key configurable options include:

*   **Labels**: Custom key-value pairs applied to the execution resource for organizational or filtering purposes.
*   **Annotations**: Custom key-value pairs providing additional, non-identifying metadata for the execution resource.
*   **Security Context**: Specifies the security identity under which the workflow will execute. This can include Kubernetes service accounts or assumable IAM roles.
*   **Raw Output Data Configuration**: Defines the remote storage location (e.g., S3, GCS) where offloaded data (like large files or schemas) will be stored. If not specified, the platform's default location is used.
*   **Max Parallelism**: Controls the maximum number of task nodes that can run concurrently within the entire workflow. This is crucial for managing resource consumption and fairness across executions. Note that MapTasks are treated as a single unit, and their internal parallelism is managed separately.
*   **Notifications**: A list of notification configurations that trigger alerts (email, PagerDuty, Slack) based on specific execution status transitions.
*   **Disable Notifications**: A boolean flag to completely disable all configured notifications for a given execution.
*   **Overwrite Cache**: A boolean flag indicating whether the execution should ignore and overwrite any existing cached results for its tasks.

**Example of using `Options`:**

The `Options` class provides a `default_from` class method for convenient initialization of common security and output configurations:

```python
from flytekit.core.options import Options

# Configure options with a Kubernetes service account and custom output prefix
execution_options = Options.default_from(
    k8s_service_account="my-service-account",
    raw_data_prefix="s3://my-custom-bucket/flyte-outputs"
)

# You can also set other options directly
execution_options.max_parallelism = 10
execution_options.labels = {"team": "data-science", "environment": "production"}
```

These `Options` can then be associated with a Launch Plan during its definition or provided dynamically when triggering an execution.

### Launch Plan Specification

The `LaunchPlanSpec` defines the complete configuration of a Launch Plan. It links to a specific workflow and includes metadata, input definitions, and the execution options described above.

The core components of a `LaunchPlanSpec` are:

*   **Workflow ID**: A unique identifier for the workflow that this Launch Plan targets.
*   **Entity Metadata**: Contains high-level metadata for the Launch Plan, including its schedule and associated notifications.
*   **Default Inputs**: A map of input parameters that provide default values for the workflow. These inputs can be overridden when an execution is launched.
*   **Fixed Inputs**: A map of input literals that are fixed and cannot be overridden during execution. These are immutable parameters for all runs initiated by this Launch Plan.
*   **Labels**: Custom Kubernetes labels applied to workflow execution resources.
*   **Annotations**: Custom Kubernetes annotations applied to workflow execution resources.
*   **Auth Role**: Specifies the authentication method (IAM role or Kubernetes service account) under which the workflow will execute. This is the primary mechanism for defining execution permissions.
*   **Raw Output Data Configuration**: Defines the storage location for offloaded data.
*   **Max Parallelism**: Controls the maximum concurrent task nodes for the workflow.
*   **Security Context**: Provides additional security information for the execution, complementing the `Auth Role`.
*   **Overwrite Cache**: Determines if task caching should be ignored for executions launched by this plan.

### Input Management

Launch Plans offer flexible input management through `default_inputs` and `fixed_inputs`:

*   **Default Inputs**: Use `default_inputs` for parameters that typically have a sensible default but might need to be adjusted for specific runs. For example, a default dataset path that can be changed for a particular experiment.
*   **Fixed Inputs**: Use `fixed_inputs` for parameters that should *never* change for any execution initiated by this Launch Plan. This is useful for hardcoding environment-specific configurations or immutable constants.

### Security and Authentication

The `AuthRole` class defines the identity for workflow execution. It supports:

*   **Assumable IAM Role**: An AWS IAM role that the Flyte execution environment will assume to gain necessary permissions.
*   **Kubernetes Service Account**: A Kubernetes service account that provides an identity for the workflow's pods, with permissions managed by the Kubernetes cluster administrator.

The `security_context` in `Options` and `LaunchPlanSpec` provides a more general mechanism to specify security-related information, including the `run_as` identity, which can reference a Kubernetes service account.

### Output Data Configuration

The `RawOutputDataConfig` specifies the `output_location_prefix` for offloaded data. This is critical for managing large data outputs (e.g., `Blob`, `Schema` types) generated by tasks and workflows. By default, Flyte uses a platform-configured location, but this option allows users to direct outputs to a specific bucket or path, which is useful for data governance, cost management, or integration with external systems.

### Notifications

Notifications are configured using the `Notification` class, which allows specifying a list of `phases` (e.g., `SUCCEEDED`, `FAILED`, `ABORTED`) at which to send alerts. Supported notification channels include:

*   **Email Notification**: Sends alerts to a list of specified email recipients.
*   **PagerDuty Notification**: Integrates with PagerDuty for incident management.
*   **Slack Notification**: Sends messages to specified Slack channels or users.

This enables proactive monitoring and alerting for workflow executions without requiring external monitoring systems.

### Launch Plan Lifecycle

A Launch Plan can exist in one of two states, as defined by `LaunchPlanState`:

*   **INACTIVE**: The Launch Plan is not currently active and cannot be used to trigger new executions.
*   **ACTIVE**: The Launch Plan is active and can be used to trigger new executions, either manually or via its configured schedule.

The `LaunchPlanClosure` provides the current state of the Launch Plan along with its expected inputs and outputs, reflecting its operational status and interface.
<!--
key: summary_launch_plan_and_execution_options_ae8cfdfd-02ee-4977-bf8b-5ae79d0af28e
type: summary_end

-->
<!--
code_unit: flytekit.core.options.Options
code_unit_type: class
help_text: ''
key: example_252967af-ae2e-4e09-bca3-c9e5486d7229
type: example

-->
<!--
code_unit: flytekit.models.launch_plan.LaunchPlanSpec
code_unit_type: class
help_text: ''
key: example_bdc1e99d-b307-47c5-a1d7-be36ecdf7f13
type: example

-->
<!--
code_unit: flytekit.models.common.Labels
code_unit_type: class
help_text: ''
key: example_b5d79f45-d49a-4702-a374-94fb978d0143
type: example

-->