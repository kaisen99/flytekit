# ArrayNode

This class represents an array node within a workflow, enabling the execution of a target entity (LaunchPlan or ReferenceTask) over a collection of inputs. It supports features like concurrency control, minimum success thresholds, and input binding. The class implements interfaces for node creation and provides methods for local execution and interface management.

## Attributes

- **target**: [Union](flytekit_models_literals_union)[[LaunchPlan](flytekit_models_launch_plan_launchplan), [ReferenceTask](flytekit_core_task_referencetask), "FlyteLaunchPlan"]
  - The target Flyte entity to map over

- **bindings**: Optional[List[_literal_models.Binding]] = None
  - Optional list of bindings.

- **concurrency**: Optional[int] = None
  - If specified, this limits the number of mapped tasks than can run in parallel to the given batch size. If the size of the input exceeds the concurrency value, then multiple batches will be run serially until all inputs are processed. If set to 0, this means unbounded concurrency. If left unspecified, this means the array node will inherit parallelism from the workflow

- **min_successes**: Optional[int] = None
  - The minimum number of successful executions. If set, this takes precedence over min_success_ratio

- **min_success_ratio**: Optional[float] = None
  - The minimum ratio of successful executions.

- **metadata**: Optional[_workflow_model.NodeMetadata] = None
  - The metadata for the underlying node

## Constructors
def ArrayNode(target: [Union](flytekit_models_literals_union)[[LaunchPlan](flytekit_models_launch_plan_launchplan), [ReferenceTask](flytekit_core_task_referencetask), "FlyteLaunchPlan"], bindings: Optional[List[_literal_models.Binding]] = None, concurrency: Optional[int] = None, min_successes: Optional[int] = None, min_success_ratio: Optional[float] = None, metadata: Optional[_workflow_model.NodeMetadata] = None)
-  Initializes an ArrayNode instance.

        :param target: The target Flyte entity to map over.
        :param concurrency: Limits the number of mapped tasks that can run in parallel. If set to 0, concurrency is unbounded. If unspecified, it inherits parallelism from the workflow.
        :param min_successes: The minimum number of successful executions. Takes precedence over min_success_ratio.
        :param min_success_ratio: The minimum ratio of successful executions.
        :param metadata: The metadata for the underlying node.
- **Parameters**

  - **target**: [Union](flytekit_models_literals_union)[[LaunchPlan](flytekit_models_launch_plan_launchplan), [ReferenceTask](flytekit_core_task_referencetask), "FlyteLaunchPlan"]
    - The target Flyte entity to map over
  - **bindings**: Optional[List[_literal_models.Binding]]
    - Optional list of bindings.
  - **concurrency**: Optional[int]
    - If specified, this limits the number of mapped tasks than can run in parallel to the given batch
            size. If the size of the input exceeds the concurrency value, then multiple batches will be run serially until
            all inputs are processed. If set to 0, this means unbounded concurrency. If left unspecified, this means the
            array node will inherit parallelism from the workflow
  - **min_successes**: Optional[int]
    - The minimum number of successful executions. If set, this takes precedence over
            min_success_ratio
  - **min_success_ratio**: Optional[float]
    - The minimum ratio of successful executions.
  - **metadata**: Optional[_workflow_model.NodeMetadata]
    - The metadata for the underlying node



## Methods
```@classmethod
def construct_node_metadata()
```
-  Constructs node metadata for the ArrayNode.

- **Return Value**:
**object**
  - The NodeMetadata object.
```@classmethod
def name()
```
-  Returns the name of the ArrayNode, which is derived from its target&#x27;s name.

- **Return Value**:
**string**
  - The name of the ArrayNode.
```@classmethod
def python_interface()
```
-  Returns the Python interface of the ArrayNode, which is transformed to handle list inputs/outputs.

- **Return Value**:
**object**
  - The transformed Python interface.
```@classmethod
def interface()
```
-  Returns the Flyte interface of the ArrayNode, used for serialization.

- **Return Value**:
**object**
  - The Flyte TypedInterface object.
```@classmethod
def bindings()
```
-  Returns the bindings associated with the ArrayNode, required for serialization.

- **Return Value**:
**array**
  - A list of Binding objects.
```@classmethod
def upstream_nodes()
```
-  Returns an empty list as ArrayNodes do not have upstream nodes in this context, required for serialization.

- **Return Value**:
**array**
  - An empty list of nodes.
```@classmethod
def flyte_entity()
```
-  Returns the target Flyte entity (e.g., LaunchPlan or Task) that the ArrayNode maps over.

- **Return Value**:
**any**
  - The target Flyte entity.
```@classmethod
def data_mode()
```
-  Returns the data mode of the ArrayNode, indicating how data is processed (e.g., SINGLE_INPUT_FILE or INDIVIDUAL_INPUT_FILES).

- **Return Value**:
**enum**
  - The DataMode enum value.
@classmethod
def local_execute(ctx: [FlyteContext](flytekit_core_context_manager_flytecontext), kwargs: dict) - > [Union](flytekit_models_literals_union)[Tuple[[Promise](flytekit_core_promise_promise)], [Promise](flytekit_core_promise_promise), [VoidPromise](flytekit_core_promise_voidpromise)]
-  Executes the ArrayNode locally, iterating over the input list and calling the target entity for each item.
- **Parameters**

  - **ctx**: [FlyteContext](flytekit_core_context_manager_flytecontext)
    - The Flyte context for execution.
  - **kwargs**: dict
    - Keyword arguments for the execution, including the list to map over.

- **Return Value**:
**[Union](flytekit_models_literals_union)[Tuple[[Promise](flytekit_core_promise_promise)], [Promise](flytekit_core_promise_promise), [VoidPromise](flytekit_core_promise_voidpromise)]**
  - The result of the local execution.
```@classmethod
def local_execution_mode()
```
-  Returns the execution mode for local execution, which is LOCAL_TASK_EXECUTION.

- **Return Value**:
**enum**
  - The ExecutionMode enum value.
```@classmethod
def min_success_ratio()
```
-  Returns the minimum success ratio required for the mapped tasks.

- **Return Value**:
**Optional[float]**
  - The minimum success ratio.
```@classmethod
def min_successes()
```
-  Returns the minimum number of successful executions required for the mapped tasks.

- **Return Value**:
**Optional[int]**
  - The minimum number of successes.
```@classmethod
def concurrency()
```
-  Returns the concurrency limit for parallel execution of mapped tasks.

- **Return Value**:
**Optional[int]**
  - The concurrency limit.
```@classmethod
def execution_mode()
```
-  Returns the execution mode of the ArrayNode, indicating how it should be executed.

- **Return Value**:
**enum**
  - The ExecutionMode enum value.
```@classmethod
def is_original_sub_node_interface()
```
-  Indicates whether the ArrayNode uses its original sub-node interface.

- **Return Value**:
**bool**
  - True if the original sub-node interface is used, False otherwise.
```@classmethod
def bound_inputs()
```
-  Returns an empty set, as bound inputs are not supported for ArrayNodes at the moment.

- **Return Value**:
**Set[str]**
  - An empty set of strings.
