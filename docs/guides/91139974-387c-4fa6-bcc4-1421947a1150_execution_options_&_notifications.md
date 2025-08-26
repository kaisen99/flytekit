
<!--
help_text: ''
key: summary_execution_options_&_notifications_d0039bfd-4177-4aa3-b048-7b3f06edf4d2
modules:
- flytekit.core.options
- flytekit.core.notification
questions_to_answer: []
type: summary

-->
Execution Options & Notifications

Execution options provide a mechanism to customize the behavior and metadata of workflow executions and launch plans. These options allow fine-grained control over aspects like resource allocation, data storage, security contexts, and how execution status is communicated.

### Execution Options

The `Options` class defines a set of configurable parameters that can be applied when registering a launch plan or when initiating an execution. This flexibility enables different users or scenarios to run the same workflow with distinct operational settings without modifying the underlying workflow definition.

Key configurable options include:

*   **Labels and Annotations**:
    *   `labels`: Custom key-value pairs for organizing, filtering, and querying execution resources. Useful for tagging executions by team, project, or environment.
    *   `annotations`: Non-identifying metadata for internal tools or additional context. Unlike labels, annotations are not typically used for querying.

*   **Security Context**:
    *   `security_context`: Specifies the security identity under which the execution runs. This is crucial for managing permissions and access to external resources.
    *   The `default_from` class method provides a convenient way to configure a Kubernetes service account for the execution:

    ```python
    from flytekit.core.options import Options

    # Configure a specific Kubernetes service account for the execution
    execution_options = Options.default_from(k8s_service_account="my-service-account")
    ```

*   **Raw Output Data Configuration**:
    *   `raw_output_data_config`: Defines an optional remote prefix for storing offloaded data, such as large files or complex data structures. This allows users to specify custom storage locations (e.g., `s3://my-bucket/output/` or `gcs://my-gcs-bucket/data/`). If not specified, the platform's default location is used.
    *   The `default_from` class method also supports setting the raw data prefix:

    ```python
    from flytekit.core.options import Options

    # Configure a custom S3 bucket for output data
    execution_options = Options.default_from(raw_data_prefix="s3://my-custom-data-bucket/workflow-outputs/")
    ```

*   **Maximum Parallelism**:
    *   `max_parallelism`: Controls the maximum number of task nodes that can execute concurrently within the entire workflow. This helps manage resource consumption and prevent overwhelming downstream systems. Setting this to `None` (default) allows the platform to determine parallelism.

*   **Cache Overwrite**:
    *   `overwrite_cache`: A boolean flag that, when set to `True`, forces the execution to re-run even if a cached result for the exact inputs exists. This is useful for debugging or when external dependencies have changed.

*   **Notifications**:
    *   `notifications`: A list of notification configurations to be triggered based on the execution's phase changes. This integrates directly with the notification system.
    *   `disable_notifications`: A boolean flag to completely disable all notifications for a specific execution, overriding any configured notifications.

### Notifications

Notifications provide a way to receive alerts about the status of workflow executions. They are configured to trigger only on *terminal* execution phases, ensuring that users are informed when a workflow completes, fails, or is otherwise finalized.

The base `Notification` class defines the common structure for all notification types. It requires a list of `phases` for which the notification should be sent. Valid terminal phases include:

*   `WorkflowExecutionPhase.ABORTED`
*   `WorkflowExecutionPhase.FAILED`
*   `WorkflowExecutionPhase.SUCCEEDED`
*   `WorkflowExecutionPhase.TIMED_OUT`

Attempting to configure notifications for non-terminal phases will result in an error. At least one phase must be specified.

```python
from flytekit.core.notification import Notification
from flytekit.models.core.execution import WorkflowExecutionPhase

# Example of a base notification (not typically used directly, but shows phase validation)
try:
    # This would raise an error as QUEUED is not a terminal phase
    invalid_notification = Notification(phases=[WorkflowExecutionPhase.QUEUED])
except AssertionError as e:
    print(f"Error: {e}")

# Valid notification phases
valid_notification = Notification(phases=[WorkflowExecutionPhase.SUCCEEDED, WorkflowExecutionPhase.FAILED])
```

Specific notification types extend the base `Notification` class, providing integrations with common communication platforms:

*   **Email Notifications**:
    *   The `Email` class sends standard email alerts to a specified list of recipients.

    ```python
    from flytekit.core.notification import Email
    from flytekit.models.core.execution import WorkflowExecutionPhase

    email_notification = Email(
        phases=[WorkflowExecutionPhase.SUCCEEDED, WorkflowExecutionPhase.FAILED],
        recipients_email=["dev-team@example.com", "on-call@example.com"]
    )
    ```

*   **Slack Notifications**:
    *   The `Slack` class sends messages to specified Slack channels or users. The `recipients_email` parameter typically maps to Slack user IDs or channel email addresses configured in the backend.

    ```python
    from flytekit.core.notification import Slack
    from flytekit.models.core.execution import WorkflowExecutionPhase

    slack_notification = Slack(
        phases=[WorkflowExecutionPhase.FAILED, WorkflowExecutionPhase.ABORTED],
        recipients_email=["#alerts-channel", "user-id@example.com"]
    )
    ```

*   **PagerDuty Notifications**:
    *   The `PagerDuty` class integrates with PagerDuty to trigger incidents. The `recipients_email` parameter typically corresponds to PagerDuty service email addresses.

    ```python
    from flytekit.core.notification import PagerDuty
    from flytekit.models.core.execution import WorkflowExecutionPhase

    pagerduty_notification = PagerDuty(
        phases=[WorkflowExecutionPhase.FAILED, WorkflowExecutionPhase.TIMED_OUT],
        recipients_email=["pagerduty-service-key@example.com"]
    )
    ```

### Integrating Options and Notifications

Notifications are integrated into workflow executions by including them in the `notifications` list within the `Options` object. This allows for a comprehensive configuration of execution behavior and alerting in a single object.

```python
from flytekit.core.options import Options
from flytekit.core.notification import Email, Slack
from flytekit.models.core.execution import WorkflowExecutionPhase

# Define notification configurations
success_email = Email(
    phases=[WorkflowExecutionPhase.SUCCEEDED],
    recipients_email=["manager@example.com"]
)

failure_slack = Slack(
    phases=[WorkflowExecutionPhase.FAILED, WorkflowExecutionPhase.ABORTED],
    recipients_email=["#dev-alerts"]
)

# Combine options and notifications for an execution
execution_options = Options(
    labels={"project": "data-pipeline", "environment": "production"},
    max_parallelism=10,
    raw_output_data_config="s3://my-prod-bucket/workflow-data/",
    notifications=[success_email, failure_slack]
)

# This `execution_options` object would then be passed when launching a workflow
# For example:
# my_workflow.launch(options=execution_options)
```

To temporarily disable all notifications for a specific execution, set `disable_notifications=True` in the `Options` object. This overrides any notifications specified in the `notifications` list.

```python
from flytekit.core.options import Options

# Launch an execution with all notifications disabled
disabled_notifications_options = Options(disable_notifications=True)
```

### Best Practices and Considerations

*   **Granularity**: Use labels and annotations effectively for better organization and traceability of executions.
*   **Security**: Always configure `security_context` with the principle of least privilege.
*   **Data Management**: Carefully consider the `raw_output_data_config` for large datasets to optimize storage costs and access patterns.
*   **Notification Scope**: Configure notifications only for the most critical terminal phases to avoid alert fatigue.
*   **Testing**: Thoroughly test notification configurations in a non-production environment to ensure they function as expected.
*   **Performance**: While `max_parallelism` helps manage resource usage, setting it too low can significantly increase workflow execution time. Balance concurrency with available resources.
<!--
key: summary_execution_options_&_notifications_d0039bfd-4177-4aa3-b048-7b3f06edf4d2
type: summary_end

-->
<!--
code_unit: flytekit.core.options.Options
code_unit_type: class
help_text: ''
key: example_30b8b603-eceb-4d44-b208-db63562808f3
type: example

-->
<!--
code_unit: flytekit.core.notification.Email
code_unit_type: class
help_text: ''
key: example_c87f6933-f9d7-4d37-b792-86921e4cf17b
type: example

-->
<!--
code_unit: flytekit.core.notification.Slack
code_unit_type: class
help_text: ''
key: example_f0f62da9-0c8e-4683-bcfe-7ff496da6340
type: example

-->
<!--
code_unit: flytekit.core.notification.PagerDuty
code_unit_type: class
help_text: ''
key: example_41b27f00-4bb9-4a86-8aca-01a2aabd674d
type: example

-->