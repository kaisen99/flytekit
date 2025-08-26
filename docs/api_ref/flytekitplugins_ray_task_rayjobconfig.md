# RayJobConfig

This class configures the settings for a Ray job. It allows specification of worker and head node configurations, enabling autoscaling, and defining the runtime environment. Additional configurations include job termination behavior and time-to-live settings.

## Attributes

- **worker_node_config**: typing.List[[WorkerNodeConfig](flytekitplugins_ray_task_workernodeconfig)]
  - worker_node_config

- **head_node_config**: typing.Optional[[HeadNodeConfig](flytekitplugins_ray_task_headnodeconfig)] = None
  - head_node_config

- **enable_autoscaling**: bool = False
  - enable_autoscaling

- **runtime_env**: typing.Optional[dict] = None
  - runtime_env

- **address**: typing.Optional[str] = None
  - address

- **shutdown_after_job_finishes**: bool = False
  - shutdown_after_job_finishes

- **ttl_seconds_after_finished**: typing.Optional[int] = None
  - ttl_seconds_after_finished

## Constructors
def RayJobConfig(worker_node_config: typing.List[[WorkerNodeConfig](flytekitplugins_ray_task_workernodeconfig)], head_node_config: typing.Optional[[HeadNodeConfig](flytekitplugins_ray_task_headnodeconfig)] = None, enable_autoscaling: bool = False, runtime_env: typing.Optional[dict] = None, address: typing.Optional[str] = None, shutdown_after_job_finishes: bool = False, ttl_seconds_after_finished: typing.Optional[int] = None)
-  Initializes a RayJobConfig object.
- **Parameters**

  - **worker_node_config**: typing.List[[WorkerNodeConfig](flytekitplugins_ray_task_workernodeconfig)]
    - Configuration for worker nodes.
  - **head_node_config**: typing.Optional[[HeadNodeConfig](flytekitplugins_ray_task_headnodeconfig)]
    - Configuration for the head node. Defaults to None.
  - **enable_autoscaling**: bool
    - Whether to enable autoscaling. Defaults to False.
  - **runtime_env**: typing.Optional[dict]
    - The runtime environment configuration. Defaults to None.
  - **address**: typing.Optional[str]
    - The address of the Ray cluster. Defaults to None.
  - **shutdown_after_job_finishes**: bool
    - Whether to shut down the cluster after the job finishes. Defaults to False.
  - **ttl_seconds_after_finished**: typing.Optional[int]
    - Time-to-live in seconds after the job finishes. Defaults to None.



