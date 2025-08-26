# K8sPod

This class represents a Kubernetes pod configuration within a Flyte context. It facilitates the creation of pod specifications, including metadata and container definitions. The class supports conversion to and from Flyte IDL and Kubernetes PodTemplate objects, enabling seamless integration with the Flyte platform.

## Attributes

- **metadata**: [K8sObjectMetadata](flytekit_models_task_k8sobjectmetadata) = None
  - This defines a kubernetes pod target.  It will build the pod target during task execution

- **pod_spec**: typing.Dict[str, typing.Any] = None
  - This defines a kubernetes pod target.  It will build the pod target during task execution

- **data_config**: typing.Optional[[DataLoadingConfig](flytekit_models_task_dataloadingconfig)] = None
  - This defines a kubernetes pod target.  It will build the pod target during task execution

- **primary_container_name**: typing.Optional[str] = None
  - This defines a kubernetes pod target.  It will build the pod target during task execution

## Constructors
def K8sPod(metadata: [K8sObjectMetadata](flytekit_models_task_k8sobjectmetadata) = None, pod_spec: typing.Dict[str, typing.Any] = None, data_config: typing.Optional[[DataLoadingConfig](flytekit_models_task_dataloadingconfig)] = None, primary_container_name: typing.Optional[str] = None)
-  This defines a kubernetes pod target.  It will build the pod target during task execution
- **Parameters**

  - **metadata**: [K8sObjectMetadata](flytekit_models_task_k8sobjectmetadata)
    - The metadata for the Kubernetes pod.
  - **pod_spec**: typing.Dict[str, typing.Any]
    - The specification for the Kubernetes pod.
  - **data_config**: typing.Optional[[DataLoadingConfig](flytekit_models_task_dataloadingconfig)]
    - Configuration for data loading.
  - **primary_container_name**: typing.Optional[str]
    - The name of the primary container in the pod.



## Methods
```@classmethod
def metadata()
```
-  Returns the metadata of the K8sPod.

- **Return Value**:
**[K8sObjectMetadata](flytekit_models_task_k8sobjectmetadata)**
  - The metadata associated with the Kubernetes pod.
```@classmethod
def pod_spec()
```
-  Returns the pod specification of the K8sPod.

- **Return Value**:
**typing.Dict[str, typing.Any]**
  - The pod specification as a dictionary.
```@classmethod
def data_config()
```
-  Returns the data configuration of the K8sPod.

- **Return Value**:
**typing.Optional[[DataLoadingConfig](flytekit_models_task_dataloadingconfig)]**
  - The data loading configuration.
```@classmethod
def primary_container_name()
```
-  Returns the name of the primary container in the K8sPod.

- **Return Value**:
**typing.Optional[str]**
  - The name of the primary container.
```@classmethod
def to_flyte_idl()
```
-  Converts the K8sPod object to its Flyte IDL representation.

- **Return Value**:
**_core_task.K8sPod**
  - The Flyte IDL representation of the K8sPod.
@classmethod
def from_flyte_idl(pb2_object: _core_task.K8sPod) - > [K8sPod](flytekit_models_task_k8spod)
-  Creates a K8sPod object from its Flyte IDL representation.
- **Parameters**

  - **pb2_object**: _core_task.K8sPod
    - The Flyte IDL object to convert from.

- **Return Value**:
**[K8sPod](flytekit_models_task_k8spod)**
  - A K8sPod object.
```@classmethod
def to_pod_template()
```
-  Converts the K8sPod object to a PodTemplate object.

- **Return Value**:
**[PodTemplate](flytekit_core_pod_template_podtemplate)**
  - A PodTemplate object.
@classmethod
def from_pod_template(pod_template: [PodTemplate](flytekit_core_pod_template_podtemplate)) - > [K8sPod](flytekit_models_task_k8spod)
-  Creates a K8sPod object from a PodTemplate object.
- **Parameters**

  - **pod_template**: [PodTemplate](flytekit_core_pod_template_podtemplate)
    - The PodTemplate object to convert from.

- **Return Value**:
**[K8sPod](flytekit_models_task_k8spod)**
  - A K8sPod object.
