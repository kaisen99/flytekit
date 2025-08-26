# CompilationState

This class manages the state of a workflow or task during compilation. It stores and tracks the nodes created while traversing the workflow graph. It supports nested workflows through the use of prefixes and allows for the management of execution modes and task resolution.

## Attributes

- **prefix**: string
  - This is because we may one day want to be able to have subworkflows inside other workflows. If users choose to not specify their node names, then we can end up with multiple &quot;n0&quot;s. This prefix allows us to give those nested nodes a distinct name, as well as properly identify them in the workflow.

- **mode**: int = 1
  - refer to `flytekit.extend.ExecutionState.Mode`

- **task_resolver**: Optional[[TaskResolverMixin](flytekit_core_base_task_taskresolvermixin)] = None
  - Please see `flytekit.extend.TaskResolverMixin`

- **nodes**: List = field(default_factory=list)
  - Stores currently compiled nodes so far.

## Constructors
def CompilationState(prefix: str, mode: Optional[int] = 1, task_resolver: Optional[[TaskResolverMixin](flytekit_core_base_task_taskresolvermixin)] = None, nodes: List = [])
-  Initializes the CompilationState. This constructor is not a pydantic constructor.
- **Parameters**

  - **prefix**: str
    - Prefix for node names, useful for nested workflows.
  - **mode**: Optional[int]
    - Execution mode, referring to `flytekit.extend.ExecutionState.Mode`.
  - **task_resolver**: Optional[[TaskResolverMixin](flytekit_core_base_task_taskresolvermixin)]
    - Task resolver for resolving tasks.
  - **nodes**: List
    - List of compiled nodes so far.



## Methods
@classmethod
def add_node(n: [Node](flytekit_models_core_workflow_node)) - > None
-  Appends a node to the list of nodes in the compilation state.
- **Parameters**

  - **n**: [Node](flytekit_models_core_workflow_node)
    - The node to add to the compilation state.

- **Return Value**:
**None**
  - This method does not return anything.
@classmethod
def with_params(prefix: str, mode: Optional[int], resolver: Optional[[TaskResolverMixin](flytekit_core_base_task_taskresolvermixin)], nodes: Optional[List]) - > [CompilationState](flytekit_core_context_manager_compilationstate)
-  Creates a new CompilationState with updated parameters. It allows overriding the prefix, mode, task resolver, and nodes. If a parameter is not provided, it defaults to the current CompilationState&#x27;s values.
- **Parameters**

  - **prefix**: str
    - The prefix to set for the new CompilationState.
  - **mode**: Optional[int]
    - The mode to set for the new CompilationState. Defaults to the current state&#x27;s mode if not provided.
  - **resolver**: Optional[[TaskResolverMixin](flytekit_core_base_task_taskresolvermixin)]
    - The task resolver to set for the new CompilationState. Defaults to the current state&#x27;s resolver if not provided.
  - **nodes**: Optional[List]
    - The list of nodes to set for the new CompilationState. Defaults to an empty list if not provided.

- **Return Value**:
**[CompilationState](flytekit_core_context_manager_compilationstate)**
  - A new CompilationState object with the specified parameters.
