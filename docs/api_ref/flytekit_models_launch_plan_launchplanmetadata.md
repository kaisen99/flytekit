# LaunchPlanMetadata

This class encapsulates metadata for a launch plan, defining its execution schedule and associated notifications. It enables the configuration of automated workflow launches based on specified schedules and event triggers. The class utilizes properties for accessing schedule, notifications, and launch conditions, facilitating the conversion to and from Flyte IDL objects.

## Attributes

- **schedule**: flytekit.models.schedule.Schedule
  - Schedule to execute the Launch Plan

- **notifications**: list[flytekit.models.common.Notification]
  - List of notifications based on Execution status transitions

- **launch_conditions**: any
  - Additional metadata for launching

## Constructors
def LaunchPlanMetadata(schedule: flytekit.models.schedule.Schedule, notifications: list[flytekit.models.common.Notification], launch_conditions: any = None)
-  Schedule to execute the Launch Plan
- **Parameters**

  - **schedule**: flytekit.models.schedule.Schedule
    - Schedule to execute the Launch Plan
  - **notifications**: list[flytekit.models.common.Notification]
    - List of notifications based on execution status transitions
  - **launch_conditions**: any
    - Additional metadata for launching



## Methods
```@classmethod
def schedule()
```
-  Schedule to execute the Launch Plan

- **Return Value**:
**flytekit.models.schedule.Schedule**
  - Schedule to execute the Launch Plan
```@classmethod
def notifications()
```
-  List of notifications based on Execution status transitions

- **Return Value**:
**list[flytekit.models.common.Notification]**
  - List of notifications based on Execution status transitions
```@classmethod
def launch_conditions()
```
-  Additional metadata for launching

- **Return Value**:
**Any**
  - Additional metadata for launching
```@classmethod
def to_flyte_idl()
```
-  List of notifications based on Execution status transitions

- **Return Value**:
**flyteidl.admin.launch_plan_pb2.LaunchPlanMetadata**
  - List of notifications based on Execution status transitions
@classmethod
def from_flyte_idl(pb2_object: flyteidl.admin.launch_plan_pb2.LaunchPlanMetadata) - > [LaunchPlanMetadata](flytekit_models_launch_plan_launchplanmetadata)
-  Converts a protobuf object of LaunchPlanMetadata to a model&#x27;s LaunchPlanMetadata object.
- **Parameters**

  - **pb2_object**: flyteidl.admin.launch_plan_pb2.LaunchPlanMetadata
    - pb2_object

- **Return Value**:
**[LaunchPlanMetadata](flytekit_models_launch_plan_launchplanmetadata)**
  - LaunchPlanMetadata
