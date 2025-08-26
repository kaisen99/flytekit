# FlyteBranchNode

This class represents a branch node within a Flyte workflow, enabling conditional execution paths. It extends the base BranchNode class and manages the if-else logic. Key functionality includes promoting from a model and handling sub-workflows, launch plans, tasks, and converted sub-workflows.

## Constructors
def FlyteBranchNode(if_else: _workflow_model.IfElseBlock)
-  Initializes a FlyteBranchNode with an IfElseBlock.
- **Parameters**

  - **if_else**: _workflow_model.IfElseBlock
    - The IfElseBlock that defines the branching logic.



## Methods
@classmethod
def promote_from_model(base_model: _workflow_model.BranchNode, sub_workflows: Dict[id_models.Identifier, _workflow_model.WorkflowTemplate], node_launch_plans: Dict[id_models.Identifier, _launch_plan_model.LaunchPlanSpec], tasks: Dict[id_models.Identifier, [FlyteTask](flytekit_remote_entities_flytetask)], converted_sub_workflows: Dict[id_models.Identifier, [FlyteWorkflow](flytekit_remote_entities_flyteworkflow)]) - > Tuple[[FlyteBranchNode](flytekit_remote_entities_flytebranchnode), Dict[id_models.Identifier, [FlyteWorkflow](flytekit_remote_entities_flyteworkflow)]]
-  Promotes a FlyteBranchNode from a base model, converting sub-workflows and tasks.
- **Parameters**

  - **base_model**: _workflow_model.BranchNode
    - The base BranchNode model to promote from.
  - **sub_workflows**: Dict[id_models.Identifier, _workflow_model.WorkflowTemplate]
    - A dictionary of sub-workflows.
  - **node_launch_plans**: Dict[id_models.Identifier, _launch_plan_model.LaunchPlanSpec]
    - A dictionary of launch plan specifications for nodes.
  - **tasks**: Dict[id_models.Identifier, [FlyteTask](flytekit_remote_entities_flytetask)]
    - A dictionary of Flyte tasks.
  - **converted_sub_workflows**: Dict[id_models.Identifier, [FlyteWorkflow](flytekit_remote_entities_flyteworkflow)]
    - A dictionary of already converted sub-workflows.

- **Return Value**:
**Tuple[[FlyteBranchNode](flytekit_remote_entities_flytebranchnode), Dict[id_models.Identifier, [FlyteWorkflow](flytekit_remote_entities_flyteworkflow)]]**
  - A tuple containing the promoted FlyteBranchNode and a dictionary of converted sub-workflows.
