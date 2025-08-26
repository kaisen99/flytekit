# nCommand(c

This class, RunCommand, is a click command group designed for managing and executing Flyte workflows and tasks defined in Python files. It enables the registration and execution of Flyte entities. The class utilizes a parameter object for configuration and supports remote entity commands.

## Attributes

- **un_params**: typing.Type[[RunLevelParams](flytekit_clis_sdk_in_container_run_runlevelparams)]
  - un_params

## Constructors
def nCommand(c(args: tuple, kwargs: dict)
-  Initializes the RunCommand with optional parameters, setting default run parameters if not provided.
- **Parameters**

  - **args**: tuple
    - Positional arguments passed to the parent class.
  - **kwargs**: dict
    - Keyword arguments passed to the parent class. If &#x27;params&#x27; is not in kwargs, it will be populated with default run parameters.

def nCommand(c(args: typing.Any, kwargs: typing.Any)
-  Initialize the RunCommand with optional parameters.
- **Parameters**

  - **args**: typing.Any
    - Positional arguments.
  - **kwargs**: typing.Any
    - Keyword arguments.



## Methods
def list_commands(ctx: click.Context, add_remote: bool = True) - > list[str]
-  List available commands, optionally including remote entities.
- **Parameters**

  - **ctx**: click.Context
    - The Click context object.
  - **add_remote**: bool
    - Whether to include remote entities in the list.

- **Return Value**:
**list[str]**
  - A list of command names.
def get_command(ctx: click.Context, filename: str) - > click.Command
-  Get a specific command based on the filename.
- **Parameters**

  - **ctx**: click.Context
    - The Click context object.
  - **filename**: str
    - The name of the file or remote entity.

- **Return Value**:
**click.Command**
  - The requested command object.
