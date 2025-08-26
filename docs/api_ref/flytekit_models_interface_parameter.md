# Parameter

This class defines an input parameter for a launch plan, enabling the specification of a name, type, and optional default value. It supports marking parameters as required and associating them with artifact queries or IDs. The class facilitates the conversion to and from Flyte IDL representations for interoperability.

## Attributes

- **var**: [Variable](flytekit_models_interface_variable)
  - The variable definition for this input parameter.

- **default**: flytekit.models.literals.Literal
  - This is the default literal value that will be applied for this parameter if not user specified.

- **required**: bool
  - If True, this parameter must be specified.  There cannot be a default value.

- **behavior**: T
  - None

- **artifact_query**: typing.Optional[art_id.ArtifactQuery]
  - None

- **artifact_id**: typing.Optional[art_id.ArtifactID]
  - None

## Constructors
def Parameter(var: [Variable](flytekit_models_interface_variable), default: flytekit.models.literals.Literal = None, required: bool = None, artifact_query: typing.Optional[art_id.ArtifactQuery] = None, artifact_id: typing.Optional[art_id.ArtifactID] = None)
-  Declares an input parameter. A parameter is used as input to a launch plan and has the special ability to have a default value or mark itself as required.
- **Parameters**

  - **var**: [Variable](flytekit_models_interface_variable)
    - Defines a name and a type to reference/compare through out the system.
  - **default**: flytekit.models.literals.Literal
    - Defines a default value that has to match the variable type defined.
  - **required**: bool
    - is this value required to be filled in?
  - **artifact_query**: typing.Optional[art_id.ArtifactQuery]
    - Specify this to bind to a query instead of a constant.
  - **artifact_id**: typing.Optional[art_id.ArtifactID]
    - When you want to bind to a known artifact pointer.

def Parameter(var: [Variable](flytekit_models_interface_variable), default: flytekit.models.literals.Literal = None, required: bool, artifact_query: typing.Optional[art_id.ArtifactQuery], artifact_id: typing.Optional[art_id.ArtifactID])
-  Declares an input parameter. A parameter is used as input to a launch plan and has the special ability to have a default value or mark itself as required.
- **Parameters**

  - **var**: [Variable](flytekit_models_interface_variable)
    - Defines a name and a type to reference/compare through out the system.
  - **default**: flytekit.models.literals.Literal
    - Optional Defines a default value that has to match the variable type defined.
  - **required**: bool
    - Optional is this value required to be filled in?
  - **artifact_query**: typing.Optional[art_id.ArtifactQuery]
    - Specify this to bind to a query instead of a constant.
  - **artifact_id**: typing.Optional[art_id.ArtifactID]
    - When you want to bind to a known artifact pointer.



## Methods
```@classmethod
def var()
```
-  The variable definition for this input parameter.

- **Return Value**:
**[Variable](flytekit_models_interface_variable)**
  - The variable definition for this input parameter.
```@classmethod
def default()
```
-  This is the default literal value that will be applied for this parameter if not user specified.

- **Return Value**:
**flytekit.models.literals.Literal**
  - This is the default literal value that will be applied for this parameter if not user specified.
```@classmethod
def required()
```
-  If True, this parameter must be specified. There cannot be a default value.

- **Return Value**:
**bool**
  - If True, this parameter must be specified. There cannot be a default value.
```@classmethod
def behavior()
```
-  Returns the default or required or artifact_query.

- **Return Value**:
**T**
  - Returns the default or required or artifact_query.
```@classmethod
def artifact_query()
```
-  Returns the artifact query.

- **Return Value**:
**typing.Optional[art_id.ArtifactQuery]**
  - Returns the artifact query.
```@classmethod
def artifact_id()
```
-  Returns the artifact id.

- **Return Value**:
**typing.Optional[art_id.ArtifactID]**
  - Returns the artifact id.
```@classmethod
def to_flyte_idl()
```
-  Converts the Parameter object to its flyte IDL representation.

- **Return Value**:
**flyteidl.core.interface_pb2.Parameter**
  - The flyte IDL representation of the Parameter.
@classmethod
def from_flyte_idl(pb2_object: flyteidl.core.interface_pb2.Parameter) - > [Parameter](flytekit_models_interface_parameter)
-  Creates a Parameter object from its flyte IDL representation.
- **Parameters**

  - **pb2_object**: flyteidl.core.interface_pb2.Parameter
    - The flyte IDL representation of a Parameter.

- **Return Value**:
**[Parameter](flytekit_models_interface_parameter)**
  - A Parameter object created from the flyte IDL representation.
