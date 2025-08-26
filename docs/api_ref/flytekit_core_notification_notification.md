# Notification

This class manages notifications triggered by workflow execution phases. It allows configuration for email, PagerDuty, and Slack notifications based on specified terminal workflow phases. The class validates the provided phases to ensure they are valid terminal states.

## Attributes

- **VALID_PHASES**: set = {_execution_model.WorkflowExecutionPhase.ABORTED, _execution_model.WorkflowExecutionPhase.FAILED, _execution_model.WorkflowExecutionPhase.SUCCEEDED, _execution_model.WorkflowExecutionPhase.TIMED_OUT}
  - A set of valid phases for which to fire the event. Events can only be fired for terminal phases.

## Constructors
def Notification(phases: List[int], email: _common_model.EmailNotification = None, pager_duty: _common_model.PagerDutyNotification = None, slack: _common_model.SlackNotification = None)
-  Initializes a Notification object.

Args:
    phases (List[int]): A required list of phases for which to fire the event. Events can only be fired for terminal phases. Phases should be as defined in: flytekit.models.core.execution.WorkflowExecutionPhase
    email (_common_model.EmailNotification, optional): Email notification settings. Defaults to None.
    pager_duty (_common_model.PagerDutyNotification, optional): PagerDuty notification settings. Defaults to None.
    slack (_common_model.SlackNotification, optional): Slack notification settings. Defaults to None.
- **Parameters**

  - **phases**: List[int]
    - A required list of phases for which to fire the event. Events can only be fired for terminal phases. Phases should be as defined in: flytekit.models.core.execution.WorkflowExecutionPhase
  - **email**: _common_model.EmailNotification
    - Email notification settings.
  - **pager_duty**: _common_model.PagerDutyNotification
    - PagerDuty notification settings.
  - **slack**: _common_model.SlackNotification
    - Slack notification settings.



