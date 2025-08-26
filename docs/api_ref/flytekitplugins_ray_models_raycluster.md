# RayCluster

This class defines the specification for a Ray cluster, used for launching and managing Ray clusters within a Kubernetes environment. It allows configuration of head and worker groups, and enables autoscaling. The class utilizes `HeadGroupSpec` and `WorkerGroupSpec` for detailed configuration and interacts with a protocol buffer for serialization and deserialization.

## Attributes

- **head_group_spec**: [HeadGroupSpec](flytekitplugins_ray_models_headgroupspec) = None
  - The head group configuration.

- **worker_group_spec**: typing.List[[WorkerGroupSpec](flytekitplugins_ray_models_workergroupspec)] = None
  - The worker group configurations.

- **enable_autoscaling**: bool = False
  - Whether to enable autoscaling.

## Constructors
def RayCluster(worker_group_spec: typing.List[[WorkerGroupSpec](flytekitplugins_ray_models_workergroupspec)], head_group_spec: typing.Optional[[HeadGroupSpec](flytekitplugins_ray_models_headgroupspec)] = None, enable_autoscaling: bool = False)
-  Define RayCluster spec that will be used by KubeRay to launch the cluster.
- **Parameters**

  - **worker_group_spec**: typing.List[[WorkerGroupSpec](flytekitplugins_ray_models_workergroupspec)]
    - The worker group configurations.
  - **head_group_spec**: typing.Optional[[HeadGroupSpec](flytekitplugins_ray_models_headgroupspec)]
    - The head group configuration.
  - **enable_autoscaling**: bool
    - Whether to enable autoscaling.



## Methods
```@classmethod
def head_group_spec()
```
-  The head group configuration.

- **Return Value**:
**[HeadGroupSpec](flytekitplugins_ray_models_headgroupspec)**
  - The head group configuration.
```@classmethod
def worker_group_spec()
```
-  The worker group configurations.

- **Return Value**:
**typing.List[[WorkerGroupSpec](flytekitplugins_ray_models_workergroupspec)]**
  - The worker group configurations.
```@classmethod
def enable_autoscaling()
```
-  Whether to enable autoscaling.

- **Return Value**:
**bool**
  - Whether to enable autoscaling.
```@classmethod
def to_flyte_idl()
```
-  Converts the RayCluster object to its corresponding Flyte IDL representation.

- **Return Value**:
**_ray_pb2.RayCluster**
  - The Flyte IDL representation of the RayCluster.
@classmethod
def from_flyte_idl(proto: flyteidl.plugins._ray_pb2.RayCluster) - > [RayCluster](flytekitplugins_ray_models_raycluster)
-  Creates a RayCluster object from its Flyte IDL representation.
- **Parameters**

  - **proto**: flyteidl.plugins._ray_pb2.RayCluster
    - The Flyte IDL representation of the RayCluster.

- **Return Value**:
**[RayCluster](flytekitplugins_ray_models_raycluster)**
  - A RayCluster object.
