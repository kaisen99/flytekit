# BuildCommand

This class serves as a command group within a command-line interface, specifically designed for building images for Flyte workflows and tasks. It extends a base class for running commands and provides functionalities to list and retrieve build-related commands. The class leverages a parameter class for build configurations and integrates with a command for building workflows.



## Methods
@classmethod
def list_commands(ctx: string, args: string, kwargs: string) - > string
-  List all available commands.
- **Parameters**

  - **ctx**: string
    - The click context.
  - **args**: string
    - Additional arguments.
  - **kwargs**: string
    - Additional keyword arguments.

- **Return Value**:
**string**
  - A list of available commands.
@classmethod
def get_command(ctx: string, filename: string) - > string
-  Get a specific command for a given filename.
- **Parameters**

  - **ctx**: string
    - The click context.
  - **filename**: string
    - The name of the file containing the workflow or task.

- **Return Value**:
**string**
  - The BuildWorkflowCommand for the given filename.
