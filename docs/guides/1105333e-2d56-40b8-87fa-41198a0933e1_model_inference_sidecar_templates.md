
<!--
help_text: ''
key: summary_model_inference_sidecar_templates_b89866c6-8680-438b-85bb-01f1af57bc2b
modules:
- flytekitplugins.inference.sidecar_template
questions_to_answer: []
type: summary

-->
## Model Inference Sidecar Templates

Model Inference Sidecar Templates enable the deployment of dedicated model inference servers as sidecar containers within a Flyte task's Kubernetes pod. This pattern allows for specialized environments and resource allocation for model serving, decoupling it from the main task logic.

### Configuring the Model Inference Sidecar

The `ModelInferenceTemplate` class defines the characteristics of the model inference sidecar. When instantiated, it constructs a Kubernetes `V1PodSpec` that includes the model server container and optionally an input downloading init container.

The `ModelInferenceTemplate` constructor accepts the following parameters to configure the sidecar:

*   **`image`** (`Optional[str]`): The Docker image for the model inference server. This image should contain your model and the serving logic.
*   **`health_endpoint`** (`Optional[str]`): An HTTP path on the model server that can be used for a startup probe. If provided, the system configures a `V1Probe` to check the server's readiness. A higher `failure_threshold` is set to accommodate potential long initialization times for the model server.
*   **`port`** (`int`, default: `8000`): The port on which the model server listens for incoming requests. The sidecar is accessible from other containers in the same pod via `localhost` on this port.
*   **`cpu`** (`int`, default: `1`): The number of CPU cores requested and limited for the model server container.
*   **`gpu`** (`int`, default: `1`): The number of GPUs requested and limited for the model server container. Set to `0` if no GPU is required.
*   **`mem`** (`str`, default: `"1Gi"`): The memory requested and limited for the model server container (e.g., "500Mi", "2Gi").
*   **`env`** (`Optional[dict[str, str]]`): A dictionary of environment variables to set within the model server container.
*   **`download_inputs`** (`bool`, default: `False`): If `True`, an `input-downloader` init container is added to the pod. This init container downloads Flyte task inputs, particularly `FlyteFile` and `BlobType` literals, and makes them available to the model server.
*   **`download_inputs_mem`** (`str`, default: `"500Mi"`): The memory requested and limited for the `input-downloader` init container.
*   **`download_inputs_cpu`** (`int`, default: `2`): The number of CPU cores requested and limited for the `input-downloader` init container.

The generated `V1PodSpec` includes a `model-server` container configured with the specified image, ports, resources, and environment variables. Its `restart_policy` is set to `Always`, treating it as a persistent sidecar.

### Automatic Input Downloading

When `download_inputs` is set to `True`, an `input-downloader` init container is automatically added to the pod. This container executes a Python script to:

1.  Retrieve the Flyte task's input literals.
2.  Download any `FlyteFile` or `BlobType` inputs to the local filesystem, typically within the `/tmp` directory.
3.  Serialize all inputs (including paths to downloaded files) into a JSON file named `inputs.json` and store it in a shared volume mounted at `/shared`.

This mechanism ensures that the model server has local access to necessary input data before it starts serving requests. The model server can then read `/shared/inputs.json` to locate and load the required data.

Both the `model-server` and `input-downloader` containers share two `empty_dir` volumes: `shared-data` (mounted at `/shared`) and `tmp` (mounted at `/tmp`). The `shared-data` volume facilitates communication between the init container and the sidecar, while `tmp` provides a temporary storage location.

### Resource Management

The `cpu`, `gpu`, and `mem` parameters directly control the resource requests and limits for the `model-server` sidecar. Similarly, `download_inputs_cpu` and `download_inputs_mem` manage resources for the `input-downloader` init container.

Properly configuring these resources is crucial for the performance and stability of your model inference service. Over-provisioning can lead to inefficient resource utilization, while under-provisioning can result in performance bottlenecks or out-of-memory errors.

### Accessing the Sidecar

The `ModelInferenceTemplate` exposes a `base_url` property, which returns `http://localhost:{port}`. This URL allows the main container within the same pod to communicate directly with the model inference sidecar.

### Usage Example

To use the model inference sidecar functionality, instantiate `ModelInferenceTemplate` with your desired configuration. The resulting `pod_template` property can then be attached to a Flyte task definition.

```python
from flytekitplugins.inference.sidecar_template import ModelInferenceTemplate

# Configure a model inference sidecar
# This example assumes a custom model server image 'my-model-server:latest'
# that listens on port 8080 and has a health check endpoint at /health.
# It also enables automatic input downloading.
model_sidecar = ModelInferenceTemplate(
    image="my-model-server:latest",
    health_endpoint="/health",
    port=8080,
    cpu=4,
    gpu=1,
    mem="16Gi",
    env={"MODEL_PATH": "/shared/my_model.pt"},
    download_inputs=True,
    download_inputs_mem="1Gi",
    download_inputs_cpu=1,
)

# The pod_template property holds the Kubernetes V1PodSpec
# This pod_template can then be used when defining a Flyte task
# For example:
# @task(pod_template=model_sidecar.pod_template)
# def my_inference_task(input_data: FlyteFile) -> str:
#     # The model server is accessible at model_sidecar.base_url
#     # Input data will be available in /shared/inputs.json
#     # ... logic to interact with the model server ...
#     pass

# You can inspect the generated pod template
print(model_sidecar.pod_template.pod_spec.to_dict())
```

### Considerations and Best Practices

*   **Model Server Image**: Ensure your `image` is optimized for serving, includes all necessary dependencies, and exposes a robust health endpoint if `health_endpoint` is used.
*   **Health Endpoint Reliability**: The `health_endpoint` should accurately reflect the model server's ability to serve requests. A well-implemented health check prevents the main task from starting before the model server is ready.
*   **Resource Tuning**: Carefully tune `cpu`, `gpu`, and `mem` based on your model's requirements and expected inference load. Monitor resource usage during development and testing.
*   **Input Handling**: If `download_inputs` is `True`, design your model server to read the `inputs.json` file from the `/shared` volume to locate and load the necessary input data.
*   **Volume Usage**: The `shared-data` and `tmp` volumes are `empty_dir` volumes, meaning their contents are ephemeral and tied to the pod's lifecycle. Do not rely on them for persistent storage.
*   **Security**: Be mindful of the data downloaded by the `input-downloader` and how it is exposed within the pod. Ensure sensitive information is handled securely.
<!--
key: summary_model_inference_sidecar_templates_b89866c6-8680-438b-85bb-01f1af57bc2b
type: summary_end

-->
<!--
code_unit: flytekitplugins.inference.examples.custom_inference_sidecar
code_unit_type: class
help_text: ''
key: example_ccbf701a-65b5-446f-8cc9-60060ecd7588
type: example

-->