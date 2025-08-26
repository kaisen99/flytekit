# ExecutionSpec

This class encapsulates the specification for an execution, including launch plan details, metadata, and various configuration options. It allows for the configuration of notifications, labels, annotations, and authorization roles. The class supports serialization and deserialization to and from Flyte IDL representations.

## Attributes

- **launch_plan**: flytekit.models.core.identifier.Identifier
  - Launch plan unique identifier to execute

- **metadata**: [ExecutionMetadata](flytekit_models_execution_executionmetadata)
  - The metadata to be associated with this execution

- **notifications**: Optional[[NotificationList](flytekit_models_execution_notificationlist)] = None
  - List of notifications for this execution.

- **disable_all**: Optional[bool] = None
  - If true, all notifications should be disabled.

- **labels**: flytekit.models.common.Labels = None
  - Labels to apply to the execution.

- **annotations**: flytekit.models.common.Annotations = None
  - Annotations to apply to the execution

- **auth_role**: flytekit.models.common.AuthRole = None
  - The authorization method with which to execute the workflow.

- **raw_output_data_config**: flytekit.models.common.RawOutputDataConfig = None
  - Optional location of offloaded data for things like S3, etc.

- **max_parallelism**: Optional[int] = None
  - Controls the maximum number of tasknodes that can be run in parallel for the entire
            workflow. This is useful to achieve fairness. Note: MapTasks are regarded as one unit, and
            parallelism/concurrency of MapTasks is independent from this.

- **security_context**: typing.Optional[security.SecurityContext] = None
  - Optional security context to use for this execution.

- **overwrite_cache**: Optional[bool] = None
  - Optional flag to overwrite the cache for this execution.

- **interruptible**: Optional[bool] = None
  - Optional flag to override the default interruptible flag of the executed entity.

- **envs**: Optional[_common_models.Envs] = None
  - flytekit.models.common.Envs environment variables to set for this execution.

- **tags**: Optional[typing.List[str]] = None
  - Optional list of tags to apply to the execution.

- **cluster_assignment**: Optional[[ClusterAssignment](flytekit_models_execution_clusterassignment)] = None
  - None

- **execution_cluster_label**: Optional[[ExecutionClusterLabel](flytekit_models_matchable_resource_executionclusterlabel)] = None
  - None

## Constructors
def ExecutionSpec(launch_plan: flytekit.models.core.identifier.Identifier, metadata: [ExecutionMetadata](flytekit_models_execution_executionmetadata), notifications: [NotificationList](flytekit_models_execution_notificationlist) = None, disable_all: bool = None, labels: flytekit.models.common.Labels = None, annotations: flytekit.models.common.Annotations = None, auth_role: flytekit.models.common.AuthRole = None, raw_output_data_config: flytekit.models.common.RawOutputDataConfig = None, max_parallelism: Optional[int] = None, security_context: Optional[security.SecurityContext] = None, overwrite_cache: Optional[bool] = None, interruptible: Optional[bool] = None, envs: Optional[_common_models.Envs] = None, tags: Optional[typing.List[str]] = None, cluster_assignment: Optional[[ClusterAssignment](flytekit_models_execution_clusterassignment)] = None, execution_cluster_label: Optional[[ExecutionClusterLabel](flytekit_models_matchable_resource_executionclusterlabel)] = None)
-  Initializes an ExecutionSpec object.
- **Parameters**

  - **launch_plan**: flytekit.models.core.identifier.Identifier
    - Launch plan unique identifier to execute
  - **metadata**: [ExecutionMetadata](flytekit_models_execution_executionmetadata)
    - The metadata to be associated with this execution
  - **notifications**: [NotificationList](flytekit_models_execution_notificationlist)
    - List of notifications for this execution.
  - **disable_all**: bool
    - If true, all notifications should be disabled.
  - **labels**: flytekit.models.common.Labels
    - Labels to apply to the execution.
  - **annotations**: flytekit.models.common.Annotations
    - Annotations to apply to the execution
  - **auth_role**: flytekit.models.common.AuthRole
    - The authorization method with which to execute the workflow.
  - **raw_output_data_config**: flytekit.models.common.RawOutputDataConfig
    - Optional location of offloaded data for things like S3, etc.
  - **max_parallelism**: Optional[int]
    - Controls the maximum number of tasknodes that can be run in parallel for the entire workflow. This is useful to achieve fairness. Note: MapTasks are regarded as one unit, and parallelism/concurrency of MapTasks is independent from this.
  - **security_context**: Optional[security.SecurityContext]
    - Optional security context to use for this execution.
  - **overwrite_cache**: Optional[bool]
    - Optional flag to overwrite the cache for this execution.
  - **interruptible**: Optional[bool]
    - Optional flag to override the default interruptible flag of the executed entity.
  - **envs**: Optional[_common_models.Envs]
    - flytekit.models.common.Envs environment variables to set for this execution.
  - **tags**: Optional[typing.List[str]]
    - Optional list of tags to apply to the execution.
  - **cluster_assignment**: Optional[[ClusterAssignment](flytekit_models_execution_clusterassignment)]
    - Optional cluster assignment for this execution.
  - **execution_cluster_label**: Optional[[ExecutionClusterLabel](flytekit_models_matchable_resource_executionclusterlabel)]
    - Optional execution cluster label to use for this execution.

def ExecutionSpec(launch_plan: flytekit.models.core.identifier.Identifier, metadata: [ExecutionMetadata](flytekit_models_execution_executionmetadata), notifications: [NotificationList](flytekit_models_execution_notificationlist) = None, disable_all: bool = None, labels: flytekit.models.common.Labels = None, annotations: flytekit.models.common.Annotations = None, auth_role: flytekit.models.common.AuthRole = None, raw_output_data_config: flytekit.models.common.RawOutputDataConfig = None, max_parallelism: Optional[int] = None, security_context: Optional[security.SecurityContext] = None, overwrite_cache: Optional[bool] = None, interruptible: Optional[bool] = None, envs: Optional[_common_models.Envs] = None, tags: Optional[typing.List[str]] = None, cluster_assignment: Optional[[ClusterAssignment](flytekit_models_execution_clusterassignment)] = None, execution_cluster_label: Optional[[ExecutionClusterLabel](flytekit_models_matchable_resource_executionclusterlabel)] = None)
-  Initializes an ExecutionSpec object.

        :param flytekit.models.core.identifier.Identifier launch_plan: Launch plan unique identifier to execute
        :param ExecutionMetadata metadata: The metadata to be associated with this execution
        :param NotificationList notifications: List of notifications for this execution.
        :param bool disable_all: If true, all notifications should be disabled.
        :param flytekit.models.common.Labels labels: Labels to apply to the execution.
        :param flytekit.models.common.Annotations annotations: Annotations to apply to the execution
        :param flytekit.models.common.AuthRole auth_role: The authorization method with which to execute the workflow.
        :param raw_output_data_config: Optional location of offloaded data for things like S3, etc.
        :param max_parallelism int: Controls the maximum number of tasknodes that can be run in parallel for the entire
            workflow. This is useful to achieve fairness. Note: MapTasks are regarded as one unit, and
            parallelism/concurrency of MapTasks is independent from this.
        :param security_context: Optional security context to use for this execution.
        :param overwrite_cache: Optional flag to overwrite the cache for this execution.
        :param interruptible: Optional flag to override the default interruptible flag of the executed entity.
        :param envs: flytekit.models.common.Envs environment variables to set for this execution.
        :param tags: Optional list of tags to apply to the execution.
        :param execution_cluster_label: Optional execution cluster label to use for this execution.
- **Parameters**

  - **launch_plan**: flytekit.models.core.identifier.Identifier
    - Launch plan unique identifier to execute
  - **metadata**: [ExecutionMetadata](flytekit_models_execution_executionmetadata)
    - The metadata to be associated with this execution
  - **notifications**: [NotificationList](flytekit_models_execution_notificationlist)
    - List of notifications for this execution.
  - **disable_all**: bool
    - If true, all notifications should be disabled.
  - **labels**: flytekit.models.common.Labels
    - Labels to apply to the execution.
  - **annotations**: flytekit.models.common.Annotations
    - Annotations to apply to the execution
  - **auth_role**: flytekit.models.common.AuthRole
    - The authorization method with which to execute the workflow.
  - **raw_output_data_config**: flytekit.models.common.RawOutputDataConfig
    - Optional location of offloaded data for things like S3, etc.
  - **max_parallelism**: Optional[int]
    - Controls the maximum number of tasknodes that can be run in parallel for the entire
            workflow. This is useful to achieve fairness. Note: MapTasks are regarded as one unit, and
            parallelism/concurrency of MapTasks is independent from this.
  - **security_context**: Optional[security.SecurityContext]
    - Optional security context to use for this execution.
  - **overwrite_cache**: Optional[bool]
    - Optional flag to overwrite the cache for this execution.
  - **interruptible**: Optional[bool]
    - Optional flag to override the default interruptible flag of the executed entity.
  - **envs**: Optional[_common_models.Envs]
    - flytekit.models.common.Envs environment variables to set for this execution.
  - **tags**: Optional[typing.List[str]]
    - Optional list of tags to apply to the execution.
  - **cluster_assignment**: Optional[[ClusterAssignment](flytekit_models_execution_clusterassignment)]
    - Optional execution cluster label to use for this execution.
  - **execution_cluster_label**: Optional[[ExecutionClusterLabel](flytekit_models_matchable_resource_executionclusterlabel)]
    - Optional execution cluster label to use for this execution.



## Methods
```@classmethod
def launch_plan()
```
-  If the values were too large, this is the URI where the values were offloaded.
        :rtype: flytekit.models.core.identifier.Identifier

- **Return Value**:
**flytekit.models.core.identifier.Identifier**
  - If the values were too large, this is the URI where the values were offloaded.
```@classmethod
def metadata()
```
-  :rtype: ExecutionMetadata

- **Return Value**:
**[ExecutionMetadata](flytekit_models_execution_executionmetadata)**
```@classmethod
def notifications()
```
-  :rtype: Optional[NotificationList]

- **Return Value**:
**Optional[[NotificationList](flytekit_models_execution_notificationlist)]**
```@classmethod
def disable_all()
```
-  :rtype: Optional[bool]

- **Return Value**:
**Optional[bool]**
```@classmethod
def labels()
```
-  :rtype: flytekit.models.common.Labels

- **Return Value**:
**flytekit.models.common.Labels**
```@classmethod
def annotations()
```
-  :rtype: flytekit.models.common.Annotations

- **Return Value**:
**flytekit.models.common.Annotations**
```@classmethod
def auth_role()
```
-  :rtype: flytekit.models.common.AuthRole

- **Return Value**:
**flytekit.models.common.AuthRole**
```@classmethod
def raw_output_data_config()
```
-  :rtype: flytekit.models.common.RawOutputDataConfig

- **Return Value**:
**flytekit.models.common.RawOutputDataConfig**
```@classmethod
def max_parallelism()
```
-  Returns the maximum parallelism for the execution.

- **Return Value**:
**int**
  - The maximum parallelism for the execution.
```@classmethod
def security_context()
```
-  Returns the security context for the execution.

- **Return Value**:
**typing.Optional[security.SecurityContext]**
  - The security context for the execution.
```@classmethod
def overwrite_cache()
```
-  Returns the overwrite cache flag for the execution.

- **Return Value**:
**Optional[bool]**
  - The overwrite cache flag for the execution.
```@classmethod
def interruptible()
```
-  Returns the interruptible flag for the execution.

- **Return Value**:
**Optional[bool]**
  - The interruptible flag for the execution.
```@classmethod
def envs()
```
-  Returns the environment variables for the execution.

- **Return Value**:
**Optional[_common_models.Envs]**
  - The environment variables for the execution.
```@classmethod
def tags()
```
-  Returns the tags for the execution.

- **Return Value**:
**Optional[typing.List[str]]**
  - The tags for the execution.
```@classmethod
def cluster_assignment()
```
-  Returns the cluster assignment for the execution.

- **Return Value**:
**Optional[[ClusterAssignment](flytekit_models_execution_clusterassignment)]**
  - The cluster assignment for the execution.
```@classmethod
def execution_cluster_label()
```
-  Returns the execution cluster label for the execution.

- **Return Value**:
**Optional[[ExecutionClusterLabel](flytekit_models_matchable_resource_executionclusterlabel)]**
  - The execution cluster label for the execution.
```@classmethod
def to_flyte_idl()
```
-  Converts the ExecutionSpec object to its Flyte IDL representation.
        :rtype: flyteidl.admin.execution_pb2.ExecutionSpec

- **Return Value**:
**flyteidl.admin.execution_pb2.ExecutionSpec**
  - The Flyte IDL representation of the ExecutionSpec object.
@classmethod
def from_flyte_idl(p: flyteidl.admin.execution_pb2.ExecutionSpec) - > [ExecutionSpec](flytekit_models_execution_executionspec)
-  Creates an ExecutionSpec object from its Flyte IDL representation.
        :param flyteidl.admin.execution_pb2.ExecutionSpec p:
        :return: ExecutionSpec
- **Parameters**

  - **p**: flyteidl.admin.execution_pb2.ExecutionSpec
    - The Flyte IDL representation of an ExecutionSpec.

- **Return Value**:
**[ExecutionSpec](flytekit_models_execution_executionspec)**
  - An ExecutionSpec object created from the Flyte IDL representation.
