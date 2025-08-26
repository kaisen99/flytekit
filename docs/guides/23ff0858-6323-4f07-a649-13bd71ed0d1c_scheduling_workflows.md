
<!--
help_text: ''
key: summary_scheduling_workflows_1c18cf86-11ba-48f7-b230-f903013eee40
modules:
- flytekit.core.schedule
- flytekit.models.schedule
- flytekit.models.launch_plan
questions_to_answer: []
type: summary

-->
Scheduling Workflows

Workflows can be configured to run automatically at specified intervals or times using schedules. This capability is managed through `LaunchPlan` objects, which encapsulate the workflow definition and its execution parameters, including scheduling.

### Fixed Rate Schedules

Fixed rate schedules execute a workflow at regular, recurring intervals. This is suitable for tasks that need to run every few minutes, hours, or days.

To define a fixed rate schedule, use the `FixedRate` class. This class requires a `duration` parameter, specified as a `datetime.timedelta` object. The smallest supported granularity for fixed rate schedules is one minute.

```python
from datetime import timedelta
from flytekit.core.schedule import FixedRate

# Schedule a workflow to run every 10 minutes
fixed_rate_schedule = FixedRate(duration=timedelta(minutes=10))

# Schedule a workflow to run every 2 hours
hourly_schedule = FixedRate(duration=timedelta(hours=2))

# Schedule a workflow to run every day
daily_schedule = FixedRate(duration=timedelta(days=1))
```

Attempts to set a duration with granularity less than a minute (e.g., seconds or microseconds) will result in an error.

### Cron Schedules

Cron schedules provide a flexible way to define execution times using standard cron expressions or predefined aliases. This is ideal for workflows that need to run at specific times of day, on particular days of the week, or on certain dates.

To define a cron schedule, use the `CronSchedule` class. The primary parameter is `schedule`, which accepts either a cron expression or a recognized alias.

```python
from flytekit.core.schedule import CronSchedule

# Schedule a workflow to run every minute
every_minute_schedule = CronSchedule(schedule="*/1 * * * *")

# Schedule a workflow to run at 3:00 AM UTC every day
daily_3am_schedule = CronSchedule(schedule="0 3 * * *")

# Schedule a workflow to run every Monday at 9:00 AM UTC
monday_9am_schedule = CronSchedule(schedule="0 9 * * MON")
```

The `schedule` parameter supports standard 5-field cron expressions (minute, hour, day-of-month, month, day-of-week). It also supports convenient aliases:

*   `@hourly` or `hourly` or `hours`
*   `@daily` or `daily` or `days`
*   `@weekly` or `weekly` or `weeks`
*   `@monthly` or `monthly` or `months`
*   `@annually` or `@yearly` or `annually` or `yearly` or `years`

An optional `offset` parameter can be provided as an ISO 8601 duration string. This adjusts the effective start time of the schedule.

```python
# Schedule a workflow to run every day at 3:00 AM UTC, but offset by 1 hour
# This means it will effectively run at 4:00 AM UTC
offset_schedule = CronSchedule(schedule="0 3 * * *", offset="PT1H")
```

The `cron_expression` parameter is deprecated and should not be used. Always use the `schedule` parameter for defining cron-based schedules.

### Accessing Kickoff Time in Workflows

Workflows can be designed to receive the exact time they were scheduled to kick off as an input argument. This is useful for time-sensitive logic within the workflow, such as fetching data up to the kickoff time or generating time-stamped outputs.

To enable this, specify the name of the workflow's `datetime` input argument using the `kickoff_time_input_arg` parameter in both `FixedRate` and `CronSchedule`.

```python
from datetime import datetime, timedelta
from flytekit import workflow, task
from flytekit.core.schedule import FixedRate, CronSchedule

@task
def process_data(kickoff_time: datetime):
    """
    A task that uses the workflow's kickoff time.
    """
    print(f"Workflow kicked off at: {kickoff_time}")
    # Your data processing logic using kickoff_time

@workflow
def my_scheduled_workflow(kickoff_time: datetime):
    process_data(kickoff_time=kickoff_time)

# Define a fixed rate schedule that passes the kickoff time
fixed_rate_lp = my_scheduled_workflow.create_launch_plan(
    "fixed_rate_lp",
    schedule=FixedRate(duration=timedelta(hours=1), kickoff_time_input_arg="kickoff_time")
)

# Define a cron schedule that passes the kickoff time
cron_lp = my_scheduled_workflow.create_launch_plan(
    "cron_lp",
    schedule=CronSchedule(schedule="0 0 * * *", kickoff_time_input_arg="kickoff_time")
)
```

While `kickoff_time_input_arg` provides the scheduled time, minor discrepancies of a few seconds may occur due to system clock synchronization and scheduling overhead.

### Associating Schedules with Launch Plans

Schedules are applied to workflows by associating them with a `LaunchPlan`. A `LaunchPlan` represents a deployable and executable version of a workflow. The scheduling information is embedded within the `LaunchPlanMetadata` of a `LaunchPlanSpec`.

When you create a `LaunchPlan` using `workflow.create_launch_plan()`, you can pass the `FixedRate` or `CronSchedule` object directly to the `schedule` argument. This automatically configures the `LaunchPlanMetadata` with the specified schedule.

Once a `LaunchPlan` with a schedule is registered, the Flyte system activates it, and the workflow begins executing according to the defined schedule. A `LaunchPlan` can be in an `ACTIVE` or `INACTIVE` state, controlling whether its schedule is currently running.
<!--
key: summary_scheduling_workflows_1c18cf86-11ba-48f7-b230-f903013eee40
type: summary_end

-->
<!--
code_unit: flytekit.core.schedule.CronSchedule
code_unit_type: class
help_text: ''
key: example_743bd404-5276-42a8-88c6-595ef2c5a77e
type: example

-->
<!--
code_unit: flytekit.core.schedule.FixedRate
code_unit_type: class
help_text: ''
key: example_24445dfb-3fd8-4a96-871b-9996eee77eb6
type: example

-->
<!--
code_unit: flytekit.models.launch_plan.LaunchPlanMetadata
code_unit_type: class
help_text: ''
key: example_701958b0-1a1b-41fa-b477-4e03a0aa1e29
type: example

-->