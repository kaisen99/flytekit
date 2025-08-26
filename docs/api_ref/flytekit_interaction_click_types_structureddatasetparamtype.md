# StructuredDatasetParamType

This class defines a custom parameter type for handling structured datasets within a command-line interface. It converts input values, such as strings or ArtifactQuery objects, into a StructuredDataset object. The class supports different input types and returns a StructuredDataset instance.

## Attributes

- **name**: string = structured dataset path (dir/file)
  - structured dataset path (dir/file)



## Methods
@classmethod
def convert(value: typing.Any, param: typing.Optional[click.Parameter], ctx: typing.Optional[click.Context]) - > typing.Any
-  TODO handle column types
- **Parameters**

  - **value**: typing.Any
    - TODO handle column types
  - **param**: typing.Optional[click.Parameter]
    - TODO handle column types
  - **ctx**: typing.Optional[click.Context]
    - TODO handle column types

- **Return Value**:
**typing.Any**
  - TODO handle column types
