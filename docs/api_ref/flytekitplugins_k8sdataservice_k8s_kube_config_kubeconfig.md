# KubeConfig

This class is designed to manage the loading of Kubernetes configuration. It provides a method to load the Kubernetes configuration, enabling interaction with a Kubernetes cluster. The class attempts to load in-cluster configuration using service account credentials.

## Constructors
```def KubeConfig()
```
-  Initializes the KubeConfig object.



## Methods
@classmethod
def load_kube_config(target_fabric: fabric) - > None
-  Load the kubernetes config based on fabric details prior to K8s client usage
- **Parameters**

  - **target_fabric**: fabric
    - fabric on which we are loading configs

- **Return Value**:
**None**
  - None
