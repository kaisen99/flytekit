
<!--
help_text: ''
key: summary_notifications_configuration_883ab978-b718-431e-b4c7-fdfd0696ece5
modules:
- flytekit.core.notification
- flytekit.models.common
questions_to_answer: []
type: summary

-->
# Notifications Configuration

Notifications configuration enables users to receive alerts about the status of their workflow executions. By integrating notifications, teams can stay informed about critical events such as successful completions, failures, or timeouts, facilitating timely responses and monitoring.

## Defining Notifications

Notifications are configured by specifying the execution phases that trigger an alert and the recipients for that alert. The core `Notification` class, along with its specialized types, provides the interface for this configuration.

### Common Parameters: Phases and Recipients

All notification types require two primary parameters:

*   **`phases`**: A list of integer values representing the workflow execution phases that should trigger the notification. Notifications are only supported for *terminal* phases of a workflow execution.
*   **`recipients_email`**: A required non-empty list of email addresses or channel identifiers to which the notification will be sent.

### Supported Execution Phases

Notifications can only be configured for the following terminal workflow execution phases, as defined in `flytekit.models.core.execution.WorkflowExecutionPhase`:

*   `WorkflowExecutionPhase.ABORTED`: The workflow execution was aborted.
*   `WorkflowExecutionPhase.FAILED`: The workflow execution failed.
*   `WorkflowExecutionPhase.SUCCEEDED`: The workflow execution completed successfully.
*   `WorkflowExecutionPhase.TIMED_OUT`: The workflow execution timed out.

Attempting to configure a notification for a non-terminal or invalid phase will result in an error. Additionally, at least one phase must be specified for any notification.

## Notification Channels

The system supports various notification channels, each tailored for a specific communication method.

### Email Notifications

The `Email` notification type sends standard email alerts to the specified recipients.

```python
from flytekit.models.core.execution import WorkflowExecutionPhase
from flytekit.core.notification import Email

# Configure an email notification for successful workflow executions
email_notification = Email(
    phases=[WorkflowExecutionPhase.SUCCEEDED],
    recipients_email=["team-leads@example.com"]
)

# Example: Configure for both failure and success
email_notification_multi_phase = Email(
    phases=[WorkflowExecutionPhase.FAILED, WorkflowExecutionPhase.SUCCEEDED],
    recipients_email=["dev-ops@example.com", "on-call@example.com"]
)
```

### Slack Notifications

The `Slack` notification type sends alerts to Slack channels or users via email addresses associated with their Slack accounts.

```python
from flytekit.models.core.execution import WorkflowExecutionPhase
from flytekit.core.notification import Slack

# Configure a Slack notification for failed workflow executions
slack_notification = Slack(
    phases=[WorkflowExecutionPhase.FAILED],
    recipients_email=["#data-alerts", "john.doe@example.com"] # Use channel names or user emails
)
```

### PagerDuty Notifications

The `PagerDuty` notification type integrates with PagerDuty to trigger incidents or send alerts. The `recipients_email` parameter should correspond to the email addresses configured for PagerDuty services or users.

```python
from flytekit.models.core.execution import WorkflowExecutionPhase
from flytekit.core.notification import PagerDuty

# Configure a PagerDuty notification for timed-out workflow executions
pagerduty_notification = PagerDuty(
    phases=[WorkflowExecutionPhase.TIMED_OUT],
    recipients_email=["pagerduty-service-email@example.com"]
)
```

## Important Considerations

*   **Terminal Phases Only**: Ensure that only `WorkflowExecutionPhase` values representing terminal states are used in the `phases` list. Non-terminal phases are not supported for notifications.
*   **Recipient Configuration**: The `recipients_email` list must not be empty. For Slack and PagerDuty, ensure the provided email addresses or channel identifiers are correctly configured within your respective services to receive alerts.
*   **Platform Integration**: While `flytekit` defines the notification configuration, the actual sending and routing of these notifications are handled by the underlying Flyte platform. Proper setup of notification services (e.g., SMTP for email, Slack webhooks, PagerDuty integrations) on the Flyte deployment is required for notifications to function.
*   **Error Handling**: The `Notification` class includes validation to ensure that at least one phase is specified and that all specified phases are valid terminal states. Invalid configurations will raise an `AssertionError` during definition.
<!--
key: summary_notifications_configuration_883ab978-b718-431e-b4c7-fdfd0696ece5
type: summary_end

-->
<!--
code_unit: flytekit.core.notification.Notification
code_unit_type: class
help_text: ''
key: example_2996061b-2a27-4e1b-84ca-9c7efe93b601
type: example

-->
<!--
code_unit: flytekit.core.notification.Email
code_unit_type: class
help_text: ''
key: example_d483b807-0212-4a8c-9b72-9f9afc103488
type: example

-->
<!--
code_unit: flytekit.core.notification.Slack
code_unit_type: class
help_text: ''
key: example_89611436-a021-436e-989d-ab8b0c992845
type: example

-->