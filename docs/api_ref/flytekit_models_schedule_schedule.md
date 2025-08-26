# Schedule

This class represents a schedule for a task or workflow execution. It allows defining schedules using either a cron expression or a fixed rate. The class provides methods to convert to and from Flyte IDL objects, enabling serialization and deserialization of schedule configurations.

## Attributes

- **kickoff_time_input_arg**: string
  - kickoff_time_input_arg

- **cron_expression**: string
  - cron_expression

- **rate**: Schedule.FixedRate
  - rate

- **cron_schedule**: Schedule.CronSchedule
  - cron_schedule

## Constructors
def Schedule(kickoff_time_input_arg: Text, cron_expression: Text = None, rate: Schedule.FixedRate = None, cron_schedule: Schedule.CronSchedule = None)
-  One of cron_expression or fixed rate must be specified.
- **Parameters**

  - **kickoff_time_input_arg**: Text
    - 
  - **cron_expression**: Text
    - [Optional]
  - **rate**: Schedule.FixedRate
    - [Optional]
  - **cron_schedule**: Schedule.CronSchedule
    - [Optional]

def Schedule(value: int, unit: int)
-  Initializes a FixedRate object.
- **Parameters**

  - **value**: int
    - The numeric value of the fixed rate.
  - **unit**: int
    - The unit of the fixed rate, an enum value from FixedRateUnit.

def Schedule(schedule: string, offset: string)
-  Initializes a CronSchedule object.
- **Parameters**

  - **schedule**: string
    - The cron expression or aliases.
  - **offset**: string
    - The ISO_8601 Duration offset.

def Schedule(kickoff_time_input_arg: string, cron_expression: string = None, rate: Schedule.FixedRate = None, cron_schedule: Schedule.CronSchedule = None)
-  Initializes a Schedule object. One of cron_expression or fixed rate must be specified.
- **Parameters**

  - **kickoff_time_input_arg**: string
    - The kickoff time input argument.
  - **cron_expression**: string
    - Optional cron expression.
  - **rate**: Schedule.FixedRate
    - Optional fixed rate schedule.
  - **cron_schedule**: Schedule.CronSchedule
    - Optional cron schedule.



## Methods
@classmethod
def enum_to_string(int_value: int) - > string
-  Converts an integer value representing a fixed rate unit to its string representation.
- **Parameters**

  - **int_value**: int
    - The integer value of the fixed rate unit.

- **Return Value**:
**string**
  - The string representation of the fixed rate unit (e.g., &#x27;MINUTE&#x27;, &#x27;HOUR&#x27;, &#x27;DAY&#x27;).
```def value()
```
-  Gets the numeric value of the fixed rate.

- **Return Value**:
**int**
  - The numeric value of the fixed rate.
```def unit()
```
-  Gets the unit of the fixed rate.

- **Return Value**:
**int**
  - The unit of the fixed rate, an enum value from FixedRateUnit.
```@classmethod
def to_flyte_idl()
```
-  Converts the FixedRate object to its corresponding Flyte IDL protobuf object.

- **Return Value**:
**string**
  - The Flyte IDL protobuf object for FixedRate.
@classmethod
def from_flyte_idl(pb2_object: flyteidl.admin.schedule_pb2.FixedRate) - > string
-  Creates a FixedRate object from a Flyte IDL protobuf object.
- **Parameters**

  - **pb2_object**: flyteidl.admin.schedule_pb2.FixedRate
    - The Flyte IDL protobuf object for FixedRate.

- **Return Value**:
**string**
  - A FixedRate object.
```def schedule()
```
-  Gets the cron schedule expression.

- **Return Value**:
**string**
  - The cron schedule expression.
```def offset()
```
-  Gets the ISO_8601 Duration offset.

- **Return Value**:
**string**
  - The ISO_8601 Duration offset.
```@classmethod
def to_flyte_idl()
```
-  Converts the CronSchedule object to its corresponding Flyte IDL protobuf object.

- **Return Value**:
**string**
  - The Flyte IDL protobuf object for CronSchedule.
@classmethod
def from_flyte_idl(pb2_object: flyteidl.admin.schedule_pb2.CronSchedule) - > string
-  Creates a CronSchedule object from a Flyte IDL protobuf object.
- **Parameters**

  - **pb2_object**: flyteidl.admin.schedule_pb2.CronSchedule
    - The Flyte IDL protobuf object for CronSchedule.

- **Return Value**:
**string**
  - A CronSchedule object.
```@classmethod
def kickoff_time_input_arg()
```
-  Gets the kickoff time input argument.

- **Return Value**:
**string**
  - The kickoff time input argument.
```@classmethod
def cron_expression()
```
-  Gets the cron expression.

- **Return Value**:
**string**
  - The cron expression.
```@classmethod
def rate()
```
-  Gets the fixed rate schedule.

- **Return Value**:
**Schedule.FixedRate**
  - The fixed rate schedule.
```@classmethod
def cron_schedule()
```
-  Gets the cron schedule.

- **Return Value**:
**Schedule.CronSchedule**
  - The cron schedule.
```@classmethod
def schedule_expression()
```
-  Gets the schedule expression, which can be a cron expression, a fixed rate, or a cron schedule.

- **Return Value**:
**string**
  - The schedule expression.
```@classmethod
def to_flyte_idl()
```
-  Converts the Schedule object to its corresponding Flyte IDL protobuf object.

- **Return Value**:
**string**
  - The Flyte IDL protobuf object for Schedule.
@classmethod
def from_flyte_idl(pb2_object: flyteidl.admin.schedule_pb2.Schedule) - > [Schedule](flytekit_models_schedule_schedule)
-  Creates a Schedule object from a Flyte IDL protobuf object.
- **Parameters**

  - **pb2_object**: flyteidl.admin.schedule_pb2.Schedule
    - The Flyte IDL protobuf object for Schedule.

- **Return Value**:
**[Schedule](flytekit_models_schedule_schedule)**
  - A Schedule object.
