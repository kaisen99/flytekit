# Slack

This class facilitates sending notifications to Slack channels. It allows users to specify the workflow execution phases that trigger notifications and the recipients. The class leverages a SlackNotification model to manage the notification details.

## Attributes

- **phases**: List[int]
  - A required list of phases for which to fire the event. Events can only be fired for terminal phases. Phases should be as defined in: flytekit.models.core.execution.WorkflowExecutionPhase

- **recipients_email**: List[str]
  - A required non-empty list of recipients for the notification.

## Constructors
def Slack(phases: List[int], recipients_email: List[str])
-  This notification should be used when sending emails to the Slack.
- **Parameters**

  - **phases**: List[int]
    - A required list of phases for which to fire the event. Events can only be fired for terminal phases. Phases should be as defined in: flytekit.models.core.execution.WorkflowExecutionPhase
  - **recipients_email**: List[str]
    - A required non-empty list of recipients for the notification.

def Slack(phases: List[int], recipients_email: List[str])
-  This notification should be used when sending emails to the Slack.
- **Parameters**

  - **phases**: List[int]
    - A required list of phases for which to fire the event. Events can only be fired for terminal phases. Phases should be as defined in: flytekit.models.core.execution.WorkflowExecutionPhase
  - **recipients_email**: List[str]
    - A required non-empty list of recipients for the notification.



