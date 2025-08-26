# FlyteGateNode

This class represents a gate node within a workflow execution. It facilitates the control of workflow execution by introducing signals, sleep durations, and approval mechanisms. The class inherits from a base GateNode model and provides a method to promote from a model.

## Constructors
```def FlyteGateNode()
```
-  Constructs a FlyteGateNode.



## Methods
@classmethod
def promote_from_model(model: _workflow_model.GateNode) - > [FlyteGateNode](flytekit_remote_entities_flytegatenode)
-  Promotes a FlyteGateNode from a _workflow_model.GateNode.
- **Parameters**

  - **model**: _workflow_model.GateNode
    - The _workflow_model.GateNode to promote from.

- **Return Value**:
**[FlyteGateNode](flytekit_remote_entities_flytegatenode)**
  - The promoted FlyteGateNode instance.
