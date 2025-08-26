# PythonFunctionTask

This class serves as the foundation for tasks defined by Python functions, automatically handling interface detection and execution within the Flyte platform. It supports various execution modes, including default, dynamic, and eager, and integrates with the `@task` decorator for streamlined task creation. Key features include automatic interface inference and the ability to manage dependencies for dynamic workflows.

## Attributes

- **task_config**: T
  - Configuration object for Task. Should be a unique type for that specific Task

- **task_function**: Callable
  - Python function that has type annotations and works for the task

- **task_type**: str = &quot;python-task&quot;
  - String task type to be associated with this Task

- **ignore_input_vars**: Optional[List[str]]
  - When supplied, these input variables will be removed from the interface. This can be used to inject some client side variables only. Prefer using ExecutionParams

- **execution_mode**: ExecutionBehavior = ExecutionBehavior.DEFAULT
  - Defines how the execution should behave, for example executing normally or specially handling a dynamic case.

- **task_resolver**: Optional[[TaskResolverMixin](flytekit_core_base_task_taskresolvermixin)]
  - None

- **node_dependency_hints**: Optional[Iterable[[Union](flytekit_models_literals_union)["PythonFunctionTask", "_annotated_launch_plan.LaunchPlan", [WorkflowBase](flytekit_core_workflow_workflowbase)]]]
  - A list of tasks, launchplans, or workflows that this task depends on. This is only for dynamic tasks/workflows, where flyte cannot automatically determine the dependencies prior to runtime.

- **pickle_untyped**: bool = false
  - If set to True, the task will pickle untyped outputs. This is just a convenience flag to avoid having to specify the output types in the interface. This is not recommended for production use.

## Constructors
def PythonFunctionTask(task_config: T, task_function: Callable, task_type: str = python-task, ignore_input_vars: Optional[List[str]], execution_mode: ExecutionBehavior = ExecutionBehavior.DEFAULT, task_resolver: Optional[[TaskResolverMixin](flytekit_core_base_task_taskresolvermixin)], node_dependency_hints: Optional[Iterable[[Union](flytekit_models_literals_union)["PythonFunctionTask", "_annotated_launch_plan.LaunchPlan", [WorkflowBase](flytekit_core_workflow_workflowbase)]]], pickle_untyped: bool = False)
-  Initializes a PythonFunctionTask.
        :param T task_config: Configuration object for Task. Should be a unique type for that specific Task
        :param Callable task_function: Python function that has type annotations and works for the task
        :param Optional[List[str]] ignore_input_vars: When supplied, these input variables will be removed from the
        interface. This
                                  can be used to inject some client side variables only. Prefer using ExecutionParams
        :param Optional[ExecutionBehavior] execution_mode: Defines how the execution should behave, for example
            executing normally or specially handling a dynamic case.
        :param str task_type: String task type to be associated with this Task
        :param Optional[Iterable[Union[&quot;PythonFunctionTask&quot;, &quot;_annotated_launch_plan.LaunchPlan&quot;, WorkflowBase]]]
        node_dependency_hints:
            A list of tasks, launchplans, or workflows that this task depends on. This is only
            for dynamic tasks/workflows, where flyte cannot automatically determine the dependencies prior to runtime.
        :param bool pickle_untyped: If set to True, the task will pickle untyped outputs. This is just a convenience
            flag to avoid having to specify the output types in the interface. This is not recommended for production
            use.
- **Parameters**

  - **task_config**: T
    - Configuration object for Task. Should be a unique type for that specific Task
  - **task_function**: Callable
    - Python function that has type annotations and works for the task
  - **task_type**: str
    - String task type to be associated with this Task
  - **ignore_input_vars**: Optional[List[str]]
    - When supplied, these input variables will be removed from the
        interface. This
                                  can be used to inject some client side variables only. Prefer using ExecutionParams
  - **execution_mode**: ExecutionBehavior
    - Defines how the execution should behave, for example
            executing normally or specially handling a dynamic case.
  - **task_resolver**: Optional[[TaskResolverMixin](flytekit_core_base_task_taskresolvermixin)]
    - None
  - **node_dependency_hints**: Optional[Iterable[[Union](flytekit_models_literals_union)["PythonFunctionTask", "_annotated_launch_plan.LaunchPlan", [WorkflowBase](flytekit_core_workflow_workflowbase)]]]
    - A list of tasks, launchplans, or workflows that this task depends on. This is only
            for dynamic tasks/workflows, where flyte cannot automatically determine the dependencies prior to runtime.
  - **pickle_untyped**: bool
    - If set to True, the task will pickle untyped outputs. This is just a convenience
            flag to avoid having to specify the output types in the interface. This is not recommended for production
            use.

def PythonFunctionTask(task_config: T, task_function: Callable, task_type: str = &quot;python-task&quot;, ignore_input_vars: Optional[List[str]], execution_mode: Optional[ExecutionBehavior] = ExecutionBehavior.DEFAULT, task_resolver: Optional[[TaskResolverMixin](flytekit_core_base_task_taskresolvermixin)], node_dependency_hints: Optional[Iterable[[Union](flytekit_models_literals_union)["PythonFunctionTask", "_annotated_launch_plan.LaunchPlan", [WorkflowBase](flytekit_core_workflow_workflowbase)]]], pickle_untyped: bool = False, kwargs: Any)
-  Initializes a PythonFunctionTask.

        :param T task_config: Configuration object for Task. Should be a unique type for that specific Task
        :param Callable task_function: Python function that has type annotations and works for the task
        :param Optional[List[str]] ignore_input_vars: When supplied, these input variables will be removed from the
        interface. This
                                  can be used to inject some client side variables only. Prefer using ExecutionParams
        :param Optional[ExecutionBehavior] execution_mode: Defines how the execution should behave, for example
            executing normally or specially handling a dynamic case.
        :param str task_type: String task type to be associated with this Task
        :param Optional[Iterable[Union[&quot;PythonFunctionTask&quot;, &quot;_annotated_launch_plan.LaunchPlan&quot;, WorkflowBase]]]
        node_dependency_hints:
            A list of tasks, launchplans, or workflows that this task depends on. This is only
            for dynamic tasks/workflows, where flyte cannot automatically determine the dependencies prior to runtime.
        :param bool pickle_untyped: If set to True, the task will pickle untyped outputs. This is just a convenience
            flag to avoid having to specify the output types in the interface. This is not recommended for production
            use.
- **Parameters**

  - **task_config**: T
    - Configuration object for Task. Should be a unique type for that specific Task
  - **task_function**: Callable
    - Python function that has type annotations and works for the task
  - **task_type**: str
    - String task type to be associated with this Task
  - **ignore_input_vars**: Optional[List[str]]
    - When supplied, these input variables will be removed from the
        interface. This
                                  can be used to inject some client side variables only. Prefer using ExecutionParams
  - **execution_mode**: Optional[ExecutionBehavior]
    - Defines how the execution should behave, for example
            executing normally or specially handling a dynamic case.
  - **task_resolver**: Optional[[TaskResolverMixin](flytekit_core_base_task_taskresolvermixin)]
    - 
  - **node_dependency_hints**: Optional[Iterable[[Union](flytekit_models_literals_union)["PythonFunctionTask", "_annotated_launch_plan.LaunchPlan", [WorkflowBase](flytekit_core_workflow_workflowbase)]]]
    - A list of tasks, launchplans, or workflows that this task depends on. This is only
            for dynamic tasks/workflows, where flyte cannot automatically determine the dependencies prior to runtime.
  - **pickle_untyped**: bool
    - If set to True, the task will pickle untyped outputs. This is just a convenience
            flag to avoid having to specify the output types in the interface. This is not recommended for production
            use.
  - **kwargs**: Any
    - 



## Methods
```@classmethod
def execution_mode()
```
-  Returns the execution mode of the task.

- **Return Value**:
**ExecutionBehavior**
  - The execution mode.
```@classmethod
def node_dependency_hints()
```
-  Returns the node dependency hints for the task.

- **Return Value**:
**Optional[Iterable[[Union](flytekit_models_literals_union)["PythonFunctionTask", "_annotated_launch_plan.LaunchPlan", [WorkflowBase](flytekit_core_workflow_workflowbase)]]]**
  - The node dependency hints.
```@classmethod
def task_function()
```
-  Returns the task function.

- **Return Value**:
**Any**
  - The task function.
```@classmethod
def name()
```
-  Returns the name of the task.

- **Return Value**:
**str**
  - The name of the task.
@classmethod
def execute(kwargs: Any) - > Any
-  This method will be invoked to execute the task. If you do decide to override this method you must also
        handle dynamic tasks or you will no longer be able to use the task as a dynamic task generator.
- **Parameters**

  - **kwargs**: Any
    - 

- **Return Value**:
**Any**
@classmethod
def compile_into_workflow(ctx: [FlyteContext](flytekit_core_context_manager_flytecontext), task_function: Callable, kwargs: Any) - > [Union](flytekit_models_literals_union)[_dynamic_job.DynamicJobSpec, _literal_models.LiteralMap]
-  In the case of dynamic workflows, this function will produce a workflow definition at execution time which will
        then proceed to be executed.
- **Parameters**

  - **ctx**: [FlyteContext](flytekit_core_context_manager_flytecontext)
    - 
  - **task_function**: Callable
    - 
  - **kwargs**: Any
    - 

- **Return Value**:
**[Union](flytekit_models_literals_union)[_dynamic_job.DynamicJobSpec, _literal_models.LiteralMap]**
@classmethod
def dynamic_execute(task_function: Callable, kwargs: Any) - > Any
-  By the time this function is invoked, the local_execute function should have unwrapped the Promises and Flyte
        literal wrappers so that the kwargs we are working with here are now Python native literal values. This
        function is also expected to return Python native literal values.

        Since the user code within a dynamic task constitute a workflow, we have to first compile the workflow, and
        then execute that workflow.

        When running for real in production, the task would stop after the compilation step, and then create a file
        representing that newly generated workflow, instead of executing it.
- **Parameters**

  - **task_function**: Callable
    - 
  - **kwargs**: Any
    - 

- **Return Value**:
**Any**
