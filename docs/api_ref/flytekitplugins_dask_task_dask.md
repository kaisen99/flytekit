# Dask

This class configures and manages Dask tasks within a distributed computing environment. It allows users to specify configurations for the Dask scheduler and worker groups. The class utilizes Scheduler and WorkerGroup objects for detailed configuration of the respective components.

## Attributes

- **scheduler**: [Scheduler](flytekitplugins_dask_task_scheduler) = Scheduler()
  - Configuration for the scheduler pod. Optional, defaults to ``Scheduler()``.

- **workers**: [WorkerGroup](flytekitplugins_dask_task_workergroup) = WorkerGroup()
  - Configuration for the pods of the default worker group. Optional, defaults to ``WorkerGroup()``.

## Constructors
def Dask(scheduler: [Scheduler](flytekitplugins_dask_task_scheduler) = Scheduler(), workers: [WorkerGroup](flytekitplugins_dask_task_workergroup) = WorkerGroup())
-  Configuration for the dask task
- **Parameters**

  - **scheduler**: [Scheduler](flytekitplugins_dask_task_scheduler)
    - Configuration for the scheduler pod. Optional, defaults to ``Scheduler()``.
  - **workers**: [WorkerGroup](flytekitplugins_dask_task_workergroup)
    - Configuration for the pods of the default worker group. Optional, defaults to ``WorkerGroup()``.



