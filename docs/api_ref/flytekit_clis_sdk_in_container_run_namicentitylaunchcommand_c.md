# namicEntityLaunchCommand(c

This class dynamically creates a command for each launch plan, enabling the execution of launch plans. It retrieves launch plan details from a remote source and generates parameters based on the launch plan&#x27;s inputs. The class leverages the click.RichCommand interface for command-line interaction.

## Attributes

- **LP_LAUNCHER**: string = &quot;lp&quot;
  - This is a dynamic command that is created for each launch plan. This is used to execute a launch plan. It will fetch the launch plan from remote and create parameters from all the inputs of the launch plan.

- **TASK_LAUNCHER**: string = &quot;task&quot;
  - This is a dynamic command that is created for each launch plan. This is used to execute a launch plan. It will fetch the launch plan from remote and create parameters from all the inputs of the launch plan.

## Constructors
def namicEntityLaunchCommand(c(name: string, h: string, entity_name: string, launcher: string, **kwargs: dict)
-  This is a dynamic command that is created for each launch plan. This is used to execute a launch plan. It will fetch the launch plan from remote and create parameters from all the inputs of the launch plan.
- **Parameters**

  - **name**: string
    - The name of the command.
  - **h**: string
    - The help string for the command.
  - **entity_name**: string
    - The name of the entity (launch plan or task) to execute.
  - **launcher**: string
    - The type of launcher to use (e.g., &#x27;lp&#x27; for launch plan, &#x27;task&#x27; for task).
  - ****kwargs**: dict
    - Additional keyword arguments.

def namicEntityLaunchCommand(c(name: string, h: string, entity_name: string, launcher: string, kwargs: dict)
-  This is a dynamic command that is created for each launch plan. This is used to execute a launch plan. It will fetch the launch plan from remote and create parameters from all the inputs of the launch plan.
- **Parameters**

  - **name**: string
    - The name of the command.
  - **h**: string
    - The help string for the command.
  - **entity_name**: string
    - The name of the entity (launch plan or task) to fetch.
  - **launcher**: string
    - The type of launcher (&#x27;lp&#x27; for launch plan, &#x27;task&#x27; for task).
  - **kwargs**: dict
    - Additional keyword arguments.



## Methods
def _fetch_entity(ctx: click.Context) - > [Union](flytekit_models_literals_union)[[FlyteLaunchPlan](flytekit_remote_entities_flytelaunchplan), [FlyteTask](flytekit_remote_entities_flytetask)]
-  Fetches the launch plan or task entity from the remote instance.
- **Parameters**

  - **ctx**: click.Context
    - The Click context object.

- **Return Value**:
**[Union](flytekit_models_literals_union)[[FlyteLaunchPlan](flytekit_remote_entities_flytelaunchplan), [FlyteTask](flytekit_remote_entities_flytetask)]**
  - The fetched FlyteLaunchPlan or FlyteTask object.
def _get_params(ctx: click.Context, inputs: Dict[str, [Variable](flytekit_models_interface_variable)], native_inputs: Dict[str, type], fixed: Optional[Dict[str, [Literal](flytekit_models_literals_literal)]] = None, defaults: Optional[Dict[str, [Parameter](flytekit_models_interface_parameter)]] = None) - > List[click.Parameter]
-  Generates a list of Click parameters from the entity&#x27;s inputs.
- **Parameters**

  - **ctx**: click.Context
    - The Click context object.
  - **inputs**: Dict[str, [Variable](flytekit_models_interface_variable)]
    - A dictionary of input variables.
  - **native_inputs**: Dict[str, type]
    - A dictionary of native Python types for the inputs.
  - **fixed**: Optional[Dict[str, [Literal](flytekit_models_literals_literal)]]
    - A dictionary of fixed input values.
  - **defaults**: Optional[Dict[str, [Parameter](flytekit_models_interface_parameter)]]
    - A dictionary of default parameter values.

- **Return Value**:
**List[click.Parameter]**
  - A list of Click parameters.
def get_params(ctx: click.Context) - > List[click.Parameter]
-  Gets the parameters for the command, fetching the entity and its inputs if necessary.
- **Parameters**

  - **ctx**: click.Context
    - The Click context object.

- **Return Value**:
**List[click.Parameter]**
  - A list of Click parameters.
def invoke(ctx: click.Context) - > Any
-  Invokes the command, executing the launch plan or task remotely.
- **Parameters**

  - **ctx**: click.Context
    - The Click context object.

- **Return Value**:
**Any**
  - The result of the remote execution.
