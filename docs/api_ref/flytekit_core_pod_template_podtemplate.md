# PodTemplate

This class defines a custom PodTemplate specification for a Task. It allows users to specify the pod specification, primary container name, labels, and annotations for a pod. The class ensures that a primary container name is defined and provides a default pod specification if one is not provided.

## Attributes

- **pod_spec**: Optional["V1PodSpec"] = None
  - Custom PodTemplate specification for a Task.

- **primary_container_name**: str = PRIMARY_CONTAINER_DEFAULT_NAME
  - Custom PodTemplate specification for a Task.

- **labels**: Optional[Dict[str, str]] = None
  - Custom PodTemplate specification for a Task.

- **annotations**: Optional[Dict[str, str]] = None
  - Custom PodTemplate specification for a Task.



