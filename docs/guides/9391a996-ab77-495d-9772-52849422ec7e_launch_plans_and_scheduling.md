
<!--
help_text: ''
key: summary_launch_plans_and_scheduling_b0ea1781-be73-4182-b10d-4a02457b5be7
modules:
- flytekit.core.launch_plan
- flytekit.models.launch_plan
questions_to_answer: []
type: summary

-->
# Launch Plans and Scheduling

Launch Plans are a fundamental construct for managing and executing workflows. While a workflow defines the computational graph, a Launch Plan specifies *how* and *when* that workflow should be executed. This includes defining default input values, fixing certain inputs, configuring schedules, setting up notifications, and specifying execution-level overrides.

Every workflow implicitly has a default Launch Plan. This default plan uses the workflow's inherent input defaults (if any) and standard execution settings. For more advanced use cases, you can create custom, named Launch Plans.

## Creating and Managing Launch Plans

The primary way to create or retrieve a Launch Plan is using the `LaunchPlan.get_or_create` class method. This method intelligently handles both default and named Launch Plans, and caches them for efficient access.

### Default Launch Plans

A default Launch Plan is automatically associated with a workflow and does not require an explicit name. It inherits all default input values defined in the workflow's signature and uses the default execution settings.

To obtain the default Launch Plan for a workflow:

```python
from flytekit import workflow, LaunchPlan

@workflow
def my_wf(a: int, c: str) -> str:
    # ... workflow logic ...
    pass

default_lp = LaunchPlan.get_or_create(workflow=my_wf)
```

When creating a default Launch Plan, you cannot specify additional parameters like schedules, fixed inputs, or notifications. Attempting to do so will raise a `ValueError`.

### Named Launch Plans

Named Launch Plans provide extensive customization options for workflow execution. They require a unique name and allow you to override or augment the workflow's default behavior.

To create a named Launch Plan:

```python
from flytekit import workflow, LaunchPlan
from flytekit.models.common import Annotations, Labels, RawOutputDataConfig, AuthRole
from flytekit.models.schedule import Schedule
from flytekit.models.security import SecurityContext, Identity

@workflow
def my_wf(a: int, b: float = 1.0, c: str = "hello") -> str:
    # ... workflow logic ...
    pass

# Example: A named Launch Plan with custom inputs and a schedule
scheduled_lp = LaunchPlan.get_or_create(
    name="daily_run_lp",
    workflow=my_wf,
    default_inputs={"a": 10},  # Default for 'a', can be overridden at execution
    fixed_inputs={"c": "fixed_value"}, # 'c' is fixed and cannot be changed at execution
    schedule=Schedule(cron_expression="0 0 * * *"), # Daily at midnight UTC
    labels=Labels({"team": "data-science"}),
    annotations=Annotations({"owner": "john.doe"}),
    max_parallelism=5,
    security_context=SecurityContext(run_as=Identity(iam_role="arn:aws:iam::123456789012:role/my-execution-role")),
    auto_activate=True, # Automatically activate on registration
)
```

The `LaunchPlan.create` method is an alternative for explicit creation, but `get_or_create` is generally preferred as it handles caching and prevents duplicate definitions for the same named Launch Plan. If you attempt to create two named Launch Plans with the same name but different properties, an `AssertionError` will be raised.

## Configuring Launch Plan Properties

Launch Plans offer a rich set of properties to control workflow executions:

### Input Management

Launch Plans allow for flexible input management, distinguishing between default and fixed inputs:

*   **Default Inputs (`default_inputs`):** These are values provided to the workflow if no explicit input is given at execution time. They can be overridden when triggering an execution. The `parameters` property of a Launch Plan reflects these default inputs.
*   **Fixed Inputs (`fixed_inputs`):** These inputs are set at the Launch Plan definition and cannot be changed at execution time. They are useful for parameters that should always have a specific value for a given Launch Plan. The `fixed_inputs` property stores these values as a `LiteralMap`.

When a Launch Plan is initialized, any fixed inputs are removed from the `parameters` map to ensure they are not mistakenly treated as overridable defaults. The `_saved_inputs` attribute internally stores both default and fixed inputs for convenience during local execution.

### Scheduling

Workflows can be scheduled to run automatically using a Launch Plan. The `schedule` parameter accepts a `Schedule` object, which can define cron expressions or other scheduling mechanisms.

```python
from flytekit.models.schedule import Schedule

# Schedule to run every Monday at 9 AM UTC
weekly_lp = LaunchPlan.get_or_create(
    name="weekly_report",
    workflow=my_wf,
    schedule=Schedule(cron_expression="0 9 * * MON"),
)
```

Additionally, an alpha feature `trigger` (of type `LaunchPlanTriggerBase`) provides a new syntax for specifying schedules.

### Notifications

Launch Plans can be configured to send notifications based on the execution status (e.g., success, failure). The `notifications` parameter accepts a list of `Notification` objects.

```python
from flytekit.models.common import Notification, NotificationType

# Send an email on workflow completion
notify_lp = LaunchPlan.get_or_create(
    name="notify_on_completion",
    workflow=my_wf,
    notifications=[
        Notification(
            phases=[NotificationType.SUCCEEDED, NotificationType.FAILED],
            recipients_email=["dev-team@example.com"]
        )
    ],
)
```

### Execution Overrides

Several parameters allow you to customize the execution environment and behavior:

*   **Labels (`labels`):** Custom Kubernetes labels applied to workflow executions launched by this plan. Useful for organization and filtering.
*   **Annotations (`annotations`):** Custom Kubernetes annotations applied to workflow executions. Useful for adding arbitrary non-identifying metadata.
*   **Raw Output Data Configuration (`raw_output_data_config`):** Specifies the storage location for offloaded data (e.g., large blobs, schemas).
*   **Maximum Parallelism (`max_parallelism`):** Controls the maximum number of task nodes that can run concurrently within the workflow execution. This helps manage resource consumption and fairness. Note that MapTasks are treated as a single unit, and their internal parallelism is separate.
*   **Security Context (`security_context`):** Defines the identity and permissions under which the workflow execution will run. This is the preferred method for specifying execution roles.
    *   *Deprecated:* The `auth_role` parameter is deprecated. Use `security_context` instead, which provides a more comprehensive `Identity` object (e.g., `iam_role`, `k8s_service_account`).
*   **Overwrite Cache (`overwrite_cache`):** If set to `True`, the execution will ignore any existing cache and re-run all tasks.
*   **Auto Activate (`auto_activate`):** If `True`, the Launch Plan will be automatically activated upon registration. This is a client-side setting and does not affect the Launch Plan's state on the Flyte backend until registration.

## Referencing Existing Launch Plans

The `ReferenceLaunchPlan` class allows you to create a local Python object that points to an existing Launch Plan on the Flyte platform without needing to re-register it. This is useful for building dependencies on pre-existing Launch Plans in different projects or domains.

When creating a `ReferenceLaunchPlan`, you must provide its unique identifier (project, domain, name, version) and its expected input and output interfaces. Flyte will validate this interface against the remote Launch Plan during compilation.

```python
from flytekit import ReferenceLaunchPlan
from typing import Dict, Type

# Reference a Launch Plan named 'my_prod_lp' in the 'production' domain
prod_lp_ref = ReferenceLaunchPlan(
    project="my_project",
    domain="production",
    name="my_prod_lp",
    version="v1.0.0",
    inputs={"a": int, "b": float},
    outputs={"result": str},
)

# You can then use this reference in other workflows or for local testing
# prod_lp_ref(a=5, b=2.5)
```

## Executing Launch Plans

A Launch Plan can be executed by calling it like a function, similar to how you would call a workflow. Any keyword arguments provided during the call will override the `default_inputs` defined in the Launch Plan. `fixed_inputs` cannot be overridden.

```python
# Assuming 'scheduled_lp' was created as in the example above
# This will trigger an immediate execution of the workflow with the LP's settings
# and override 'a' to 20
execution = scheduled_lp(a=20)
```

During compilation, calling a Launch Plan creates and links a node in the workflow graph. In local execution contexts, the call is forwarded directly to the underlying workflow, incorporating the Launch Plan's saved inputs.

## Internal Structure and State

Under the hood, Launch Plans are represented by several model classes that define their specification and state:

*   **`LaunchPlanSpec`**: This model defines the desired configuration of a Launch Plan. It includes the `workflow_id` (the unique identifier of the associated workflow), `entity_metadata` (which contains scheduling and notification details), `default_inputs`, `fixed_inputs`, `labels`, `annotations`, `auth_role` (deprecated, use `security_context`), `raw_output_data_config`, `max_parallelism`, `security_context`, and `overwrite_cache`.
*   **`LaunchPlanMetadata`**: Encapsulates the `schedule` and `notifications` associated with the Launch Plan.
*   **`LaunchPlanClosure`**: Represents the current state of a Launch Plan on the Flyte platform. It includes the `state` (e.g., `ACTIVE`, `INACTIVE`), `expected_inputs`, and `expected_outputs` of the Launch Plan.
*   **`LaunchPlanState`**: An enum defining the possible states of a Launch Plan, such as `INACTIVE` and `ACTIVE`.

These model classes are primarily used for serialization and deserialization when interacting with the Flyte Admin service. Developers typically interact with the higher-level `LaunchPlan` class in `flytekit.core.launch_plan` for defining and managing Launch Plans in their Python code.
<!--
key: summary_launch_plans_and_scheduling_b0ea1781-be73-4182-b10d-4a02457b5be7
type: summary_end

-->
<!--
code_unit: flytekit.core.launch_plan
code_unit_type: class
help_text: ''
key: example_f3f8d080-eeb7-48c6-bd4e-6bd826551625
type: example

-->