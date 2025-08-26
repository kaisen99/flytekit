
<!--
help_text: ''
key: summary_referencing_existing_flyte_entities_ac9c6310-883d-430b-bbac-97a25a9d29ac
modules:
- flytekit.core.reference_entity
- flytekit.core.task
- flytekit.core.workflow
- flytekit.core.launch_plan
questions_to_answer: []
type: summary

-->
Referencing Existing Flyte Entities

Flyte enables the reuse of pre-existing tasks, workflows, and launch plans that are already registered on a Flyte cluster. This capability is essential for building modular, composable, and scalable data and ML pipelines, allowing you to integrate components without redefining their underlying logic.

### Core Concepts

At the heart of referencing existing entities is the concept of a **Reference Entity**. A reference entity acts as a pointer to a remote Flyte object (a task, workflow, or launch plan) identified by its unique `project`, `domain`, `name`, and `version`. Unlike locally defined entities, reference entities do not contain the actual code logic; they only hold the metadata necessary to interact with the remote object.

The `ReferenceEntity` class serves as the base for all remote entity types. It encapsulates a `Reference` object, which can be a `TaskReference`, `WorkflowReference`, or `LaunchPlanReference`. These specific reference types define the `resource_type` (e.g., `ResourceType.TASK`, `ResourceType.WORKFLOW`) to identify the kind of remote entity being referenced.

A critical aspect of defining a reference entity is providing its **interface** (inputs and outputs). Since the actual code is remote, Flytekit relies on the user to declare the expected input and output types. This interface must precisely match the interface of the remote entity on the Flyte cluster; any mismatch will result in a compilation error during registration.

### Referencing Tasks

To reference an existing task on the Flyte cluster, use the `ReferenceTask` class. This class inherits from `ReferenceEntity` and `PythonTask`, allowing it to be used seamlessly within new workflows as if it were a locally defined task.

When instantiating a `ReferenceTask`, you must provide:
*   `project`: The project name where the remote task is registered.
*   `domain`: The domain name where the remote task is registered.
*   `name`: The unique name of the remote task.
*   `version`: The specific version of the remote task.
*   `inputs`: A dictionary mapping input names to their Python types (e.g., `{"x": int, "y": str}`).
*   `outputs`: A dictionary mapping output names to their Python types (e.g., `{"result": float}`).

**Example:**
Assume a task named `my_remote_task` exists in `flyte-project` / `development` with version `v1` that takes an integer `a` and returns a string `b`.

```python
from flytekit import workflow, task
from flytekit.core.task import ReferenceTask
from typing import Dict, Type

# Define the reference to the remote task
remote_task = ReferenceTask(
    project="flyte-project",
    domain="development",
    name="my_remote_task",
    version="v1",
    inputs={"a": int},
    outputs={"b": str},
)

@task
def process_output(output_str: str) -> str:
    return f"Processed: {output_str}"

@workflow
def my_workflow_using_remote_task(input_val: int) -> str:
    # Use the remote task as if it were a local task
    remote_output = remote_task(a=input_val)
    final_result = process_output(output_str=remote_output.b)
    return final_result
```

In this example, `remote_task` is a placeholder that tells Flyte to link to the specified remote task during compilation.

### Referencing Workflows

Similarly, you can reference an existing workflow using the `ReferenceWorkflow` class. This is useful for creating hierarchical workflows or for building new launch plans that execute a pre-existing workflow.

The `ReferenceWorkflow` class inherits from `ReferenceEntity` and `PythonFunctionWorkflow`. Its constructor requires the same `project`, `domain`, `name`, `version`, `inputs`, and `outputs` parameters as `ReferenceTask`.

**Example:**
Assume a workflow named `my_remote_workflow` exists in `flyte-project` / `development` with version `v1` that takes a string `input_str` and returns an integer `output_int`.

```python
from flytekit import workflow
from flytekit.core.workflow import ReferenceWorkflow
from typing import Dict, Type

# Define the reference to the remote workflow
remote_workflow = ReferenceWorkflow(
    project="flyte-project",
    domain="development",
    name="my_remote_workflow",
    version="v1",
    inputs={"input_str": str},
    outputs={"output_int": int},
)

@task
def analyze_result(num: int) -> str:
    return f"Analysis complete for {num}"

@workflow
def parent_workflow(data: str) -> str:
    # Call the remote workflow
    result = remote_workflow(input_str=data)
    final_analysis = analyze_result(num=result.output_int)
    return final_analysis
```

### Referencing Launch Plans

The `ReferenceLaunchPlan` class allows you to reference an existing launch plan. This is particularly useful for triggering specific configurations of a workflow that are already defined and managed on the Flyte cluster.

The `ReferenceLaunchPlan` class inherits from `ReferenceEntity` and `LaunchPlan`. Its constructor also requires `project`, `domain`, `name`, `version`, `inputs`, and `outputs`.

**Example:**
Assume a launch plan named `my_remote_lp` exists in `flyte-project` / `development` with version `v1` that takes a boolean `flag` and returns a string `status`.

```python
from flytekit import workflow
from flytekit.core.launch_plan import ReferenceLaunchPlan
from typing import Dict, Type

# Define the reference to the remote launch plan
remote_launch_plan = ReferenceLaunchPlan(
    project="flyte-project",
    domain="development",
    name="my_remote_lp",
    version="v1",
    inputs={"flag": bool},
    outputs={"status": str},
)

@task
def log_status(message: str):
    print(f"Status received: {message}")

@workflow
def trigger_remote_lp(enable_feature: bool):
    # Trigger the remote launch plan
    lp_result = remote_launch_plan(flag=enable_feature)
    log_status(message=lp_result.status)
```

### Local Execution and Testing

Reference entities (`ReferenceTask`, `ReferenceWorkflow`, `ReferenceLaunchPlan`) are designed to link to remote components. Consequently, their `execute` method, which would normally run the entity's logic locally, raises a `NotImplementedError`. This is because the actual code for these entities resides on the Flyte cluster, not in your local environment.

For local testing and development, you must **mock** the behavior of reference entities. The `local_execute` method is available for this purpose, allowing you to simulate the outputs of the remote entity without requiring a connection to the Flyte cluster. When running workflows locally, Flytekit's execution context will automatically invoke `local_execute` for reference entities.

**Important:** When defining a reference entity, the `inputs` and `outputs` dictionaries are crucial. They define the contract for the remote entity. If these interfaces do not precisely match the registered entity on the Flyte cluster, compilation and execution will fail. Always ensure the provided interface accurately reflects the remote entity's signature.
<!--
key: summary_referencing_existing_flyte_entities_ac9c6310-883d-430b-bbac-97a25a9d29ac
type: summary_end

-->
<!--
code_unit: flytekit.core.reference_entity.ReferenceEntity
code_unit_type: class
help_text: ''
key: example_e81f0f7e-4561-492a-92cd-e7e770aae9e0
type: example

-->
<!--
code_unit: flytekit.core.task.ReferenceTask
code_unit_type: class
help_text: ''
key: example_6157843e-451a-4aa4-9468-3bc1e1abd6f3
type: example

-->
<!--
code_unit: flytekit.core.workflow.ReferenceWorkflow
code_unit_type: class
help_text: ''
key: example_b3cf0819-eef9-4448-88bb-d9731663f795
type: example

-->
<!--
code_unit: flytekit.core.launch_plan.ReferenceLaunchPlan
code_unit_type: class
help_text: ''
key: example_f6fbab48-ff20-4d6c-9a87-a02fde66acac
type: example

-->