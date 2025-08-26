# FlyteEntities

This class serves as a central registry for tracking tasks and workflows declared during the registration process within a virtual machine. It maintains a collection of launch plans, tasks, and workflow definitions. This class facilitates the management and organization of Flyte entities within the system.

## Attributes

- **entities**: List[[Union](flytekit_models_literals_union)["LaunchPlan", [Task](flytekit_models_task_task), "WorkflowBase"]] = []
  - This is a global Object that tracks various tasks and workflows that are declared within a VM during the registration process



