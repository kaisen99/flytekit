# Worker

This class defines the configuration for a worker component. It allows specifying resource requests and limits, the desired number of replicas, and the image to be used. The class also supports defining a restart policy for the worker.

## Attributes

- **image**: Optional[str] = None
  - The container image to use for the worker.

- **requests**: Optional[[Resources](flytekit_models_task_resources)] = None
  - The resource requests for the worker.

- **limits**: Optional[[Resources](flytekit_models_task_resources)] = None
  - The resource limits for the worker.

- **replicas**: Optional[int] = None
  - The number of replicas for the worker.

- **restart_policy**: Optional[[RestartPolicy](flytekitplugins_kftensorflow_task_restartpolicy)] = None
  - The restart policy for the worker.



