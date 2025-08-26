# LaunchPlan

This class represents a Flyte Launch Plan, which defines how a workflow is executed. It allows for the configuration of default and fixed inputs, schedules, and notifications. Launch Plans can be created, retrieved, and customized with various parameters for execution control.

## Constructors
def LaunchPlan(name: string, workflow: _annotated_workflow.WorkflowBase, parameters: _interface_models.ParameterMap, fixed_inputs: _literal_models.LiteralMap, schedule: Optional[_schedule_model.Schedule] = None, notifications: Optional[List[_common_models.Notification]] = [], labels: Optional[_common_models.Labels] = None, annotations: Optional[_common_models.Annotations] = None, raw_output_data_config: Optional[_common_models.RawOutputDataConfig] = None, max_parallelism: Optional[int] = None, security_context: Optional[security.SecurityContext] = None, trigger: Optional[[LaunchPlanTriggerBase](flytekit_core_schedule_launchplantriggerbase)] = None, overwrite_cache: Optional[bool] = None, auto_activate: bool = False)
-  Initializes a LaunchPlan object.

        Args:
            name (str): The name of the launch plan.
            workflow (_annotated_workflow.WorkflowBase): The workflow this launch plan is associated with.
            parameters (_interface_models.ParameterMap): A map of parameters for the workflow.
            fixed_inputs (_literal_models.LiteralMap): A map of fixed inputs that cannot be changed at launch time.
            schedule (Optional[_schedule_model.Schedule]): An optional schedule for the launch plan.
            notifications (Optional[List[_common_models.Notification]]): A list of optional notifications.
            labels (Optional[_common_models.Labels]): Optional labels to attach to executions.
            annotations (Optional[_common_models.Annotations]): Optional annotations to attach to executions.
            raw_output_data_config (Optional[_common_models.RawOutputDataConfig]): Configuration for raw output data.
            max_parallelism (Optional[int]): Maximum number of tasks that can run in parallel.
            security_context (Optional[security.SecurityContext]): Security context for execution.
            trigger (Optional[LaunchPlanTriggerBase]): An optional trigger for the launch plan.
            overwrite_cache (Optional[bool]): Whether to overwrite the cache.
            auto_activate (bool): Whether to automatically activate the launch plan on registration.
- **Parameters**

  - **name**: string
    - The name of the launch plan.
  - **workflow**: _annotated_workflow.WorkflowBase
    - The workflow this launch plan is associated with.
  - **parameters**: _interface_models.ParameterMap
    - A map of parameters for the workflow.
  - **fixed_inputs**: _literal_models.LiteralMap
    - A map of fixed inputs that cannot be changed at launch time.
  - **schedule**: Optional[_schedule_model.Schedule]
    - An optional schedule for the launch plan.
  - **notifications**: Optional[List[_common_models.Notification]]
    - A list of optional notifications.
  - **labels**: Optional[_common_models.Labels]
    - Optional labels to attach to executions.
  - **annotations**: Optional[_common_models.Annotations]
    - Optional annotations to attach to executions.
  - **raw_output_data_config**: Optional[_common_models.RawOutputDataConfig]
    - Configuration for raw output data.
  - **max_parallelism**: Optional[int]
    - Maximum number of tasks that can run in parallel.
  - **security_context**: Optional[security.SecurityContext]
    - Security context for execution.
  - **trigger**: Optional[[LaunchPlanTriggerBase](flytekit_core_schedule_launchplantriggerbase)]
    - An optional trigger for the launch plan.
  - **overwrite_cache**: Optional[bool]
    - Whether to overwrite the cache.
  - **auto_activate**: bool
    - Whether to automatically activate the launch plan on registration.



