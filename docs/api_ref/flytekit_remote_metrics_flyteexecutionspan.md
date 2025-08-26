# FlyteExecutionSpan

This class represents a Flyte execution span, encapsulating information about a specific operation&#x27;s lifecycle. It provides methods to explain and dump the span data, facilitating debugging and analysis. The class utilizes a protobuf-based span object and offers conversion methods to and from Flyte IDL representations.

## Constructors
def FlyteExecutionSpan(span: Span)
-  Initializes a FlyteExecutionSpan object.
- **Parameters**

  - **span**: Span
    - A Span object from the Flyte IDL.



## Methods
```@classmethod
def explain()
```
-  Prints a formatted table of span information, including operation, start and end timestamps, duration, and entity.

- **Return Value**:
**None**
  - None
```@classmethod
def dump()
```
-  Dumps the aggregated span information to YAML format with an indentation of 2 spaces.

- **Return Value**:
**None**
  - None
```@classmethod
def to_flyte_idl()
```
-  Returns the internal _span object, which represents the span in Flyte IDL format.

- **Return Value**:
**Span**
  - The internal _span object.
@classmethod
def from_flyte_idl(pb: Span) - > [FlyteExecutionSpan](flytekit_remote_metrics_flyteexecutionspan)
-  Creates a FlyteExecutionSpan instance from a given Flyte IDL Span object.
- **Parameters**

  - **pb**: Span
    - The Flyte IDL Span object to create the instance from.

- **Return Value**:
**[FlyteExecutionSpan](flytekit_remote_metrics_flyteexecutionspan)**
  - A new FlyteExecutionSpan instance.
