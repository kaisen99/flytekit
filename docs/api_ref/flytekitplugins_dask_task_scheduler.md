# Scheduler

This class configures the scheduler pod within a distributed computing environment. It allows customization of the scheduler&#x27;s image, resource requests, and resource limits. These configurations enable users to tailor the scheduler&#x27;s environment to meet specific computational needs.

## Attributes

- **image**: Optional[str] = None
  - Custom image to use. If ``None``, will use the same image the task was registered with. Optional, defaults to ``None``. The image must have ``dask[distributed]`` installed and should have the same Python environment as the rest of the cluster (job runner pod + worker pods).

- **requests**: Optional[[Resources](flytekit_models_task_resources)] = None
  - Resources to request for the scheduler pod. If ``None``, the requests passed into the task will be used. Optional, defaults to ``None``.

- **limits**: Optional[[Resources](flytekit_models_task_resources)] = None
  - Resource limits for the scheduler pod. If ``None``, the limits passed into the task will be used. Optional, defaults to ``None``.

## Constructors
def Scheduler(image: Optional[str] = None, requests: Optional[[Resources](flytekit_models_task_resources)] = None, limits: Optional[[Resources](flytekit_models_task_resources)] = None)
-  Configuration for the scheduler pod
- **Parameters**

  - **image**: Optional[str]
    - Custom image to use. If ``None``, will use the same image the task was registered with. Optional, defaults to ``None``. The image must have ``dask[distributed]`` installed and should have the same Python environment as the rest of the cluster (job runner pod + worker pods).
  - **requests**: Optional[[Resources](flytekit_models_task_resources)]
    - Resources to request for the scheduler pod. If ``None``, the requests passed into the task will be used. Optional, defaults to ``None``.
  - **limits**: Optional[[Resources](flytekit_models_task_resources)]
    - Resource limits for the scheduler pod. If ``None``, the limits passed into the task will be used. Optional, defaults to ``None``.

def Scheduler(image: Optional[str] = None, requests: Optional[[Resources](flytekit_models_task_resources)] = None, limits: Optional[[Resources](flytekit_models_task_resources)] = None)
-  Configuration for the scheduler pod
- **Parameters**

  - **image**: Optional[str]
    - Custom image to use. If ``None``, will use the same image the task was registered with. Optional, defaults to ``None``. The image must have ``dask[distributed]`` installed and should have the same Python environment as the rest of the cluster (job runner pod + worker pods).
  - **requests**: Optional[[Resources](flytekit_models_task_resources)]
    - Resources to request for the scheduler pod. If ``None``, the requests passed into the task will be used. Optional, defaults to ``None``.
  - **limits**: Optional[[Resources](flytekit_models_task_resources)]
    - Resource limits for the scheduler pod. If ``None``, the limits passed into the task will be used. Optional, defaults to ``None``.



