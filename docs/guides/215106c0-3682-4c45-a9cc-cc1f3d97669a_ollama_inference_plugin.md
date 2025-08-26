
<!--
help_text: ''
key: summary_ollama_inference_plugin_b5f44a22-2ab6-4d42-8c56-033e99366b81
modules:
- flytekitplugins.inference.ollama.serve
- flytekitplugins.inference.sidecar_template
questions_to_answer: []
type: summary

-->
The Ollama Inference Plugin facilitates deploying Ollama models as sidecar containers within a Kubernetes pod, enabling efficient local inference alongside your main application. It manages the lifecycle of the Ollama server, including model pulling or creation, and resource allocation.

### Model Configuration

The `Model` class defines the specific Ollama model to be deployed. It encapsulates essential configuration parameters:

*   `name`: The identifier for the Ollama model (e.g., "llama2", "my-custom-model").
*   `mem`: The memory allocated for the model within the Ollama container (default: "500Mi").
*   `cpu`: The CPU cores allocated for the model (default: 1).
*   `modelfile`: An optional string representing the content of an Ollama Modelfile. If provided, the plugin creates a custom model; otherwise, it pulls a pre-existing model by `name`. This parameter supports dynamic templating using Flyte task inputs.

**Example `Model` Configuration:**

```python
from flytekitplugins.inference.ollama.serve import Model

# To pull a pre-existing model
llama2_model = Model(name="llama2")

# To create a model from a custom Modelfile
custom_model_file_content = """
FROM llama2
PARAMETER temperature 0.7
"""
custom_model = Model(name="my-custom-llama", modelfile=custom_model_file_content)
```

### Ollama Deployment and Lifecycle

The `Ollama` class extends the base `ModelInferenceTemplate` to provide Ollama-specific deployment capabilities. It configures a Kubernetes pod template that includes:

1.  **An Ollama server container:** This is the main sidecar container running the `ollama/ollama` image, exposing the Ollama API.
2.  **An initialization container:** This container is responsible for preparing the Ollama model before the main server starts. Its behavior depends on whether a `modelfile` is provided in the `Model` configuration.

**Initialization Parameters for `Ollama`:**

*   `model`: An instance of the `Model` class, defining the target Ollama model.
*   `image`: The Docker image for the Ollama server (default: "ollama/ollama").
*   `port`: The port on which the Ollama server listens (default: 11434).
*   `cpu`, `gpu`, `mem`: Resource requests and limits for the main Ollama server container (defaults: `cpu=1`, `gpu=1`, `mem="15Gi"`).
*   `download_inputs_mem`, `download_inputs_cpu`: Resources for the optional input downloading container (defaults: `mem="500Mi"`, `cpu=2`).

**Example `Ollama` Initialization:**

```python
from flytekitplugins.inference.ollama.serve import Model, Ollama

# Deploying a pre-existing 'llama2' model
ollama_sidecar_config = Ollama(
    model=Model(name="llama2"),
    cpu=2,
    mem="20Gi",
    gpu=1,
)

# Deploying a custom model from a Modelfile
custom_modelfile = """
FROM llama2
PARAMETER temperature 0.7
"""
custom_ollama_sidecar_config = Ollama(
    model=Model(name="my-custom-model", modelfile=custom_modelfile),
    cpu=2,
    mem="20Gi",
)
```

### Model Pulling and Creation

The plugin handles two primary scenarios for model availability:

#### Pulling a Pre-existing Model

When the `modelfile` attribute in the `Model` configuration is `None` (the default), the initialization container executes an `ollama pull` command for the specified `model.name`. This is suitable for models available in the public Ollama registry or a configured private registry.

#### Creating a Model from a Modelfile

If a `modelfile` string is provided in the `Model` configuration, the initialization container writes this content to a `Modelfile` and then executes an `ollama create` command. This allows for highly customized models, including:

*   **Base Model Customization:** Modifying parameters of an existing base model (e.g., `FROM llama2 PARAMETER temperature 0.7`).
*   **Local Model Files:** Incorporating local files (e.g., `FROM ./model.gguf`). For local files, these must be made available to the container, typically via a volume mount or by embedding them directly if small enough.

#### Dynamic Modelfile Generation with Flyte Inputs

A powerful feature is the ability to dynamically generate the `modelfile` content using Flyte task inputs. If the `modelfile` string contains the placeholder `{inputs}`, the plugin automatically sets up an additional initialization container (`input-downloader`). This container downloads the Flyte task's inputs and makes them available as a JSON object at `/shared/inputs.json`. The `modelfile` string is then formatted using these inputs before the `ollama create` command is executed.

This enables scenarios where model parameters, base models, or even paths to model weights can be dynamically determined by the workflow's inputs.

**Example: Dynamic Modelfile with Flyte Inputs**

Assume a Flyte task receives an input `temperature_value: float`.

```python
from flytekitplugins.inference.ollama.serve import Model, Ollama

# The Modelfile string includes a placeholder for 'inputs.temperature_value'
dynamic_modelfile_content = """
FROM llama2
PARAMETER temperature {inputs.temperature_value}
"""

dynamic_ollama_sidecar_config = Ollama(
    model=Model(name="dynamic-llama", modelfile=dynamic_modelfile_content),
    cpu=2,
    mem="20Gi",
)

# When this is used in a Flyte task, the 'temperature_value' input
# will be used to format the Modelfile before model creation.
# For example, if temperature_value = 0.5, the Modelfile will become:
# FROM llama2
# PARAMETER temperature 0.5
```

### Resource Management

The plugin allows granular control over resource allocation for both the main Ollama server and the model initialization process:

*   **Ollama Server Resources:** Configured via `cpu`, `gpu`, and `mem` parameters of the `Ollama` class. These apply to the container running the `ollama/ollama` image.
*   **Model Initialization Resources:** Configured via `cpu` and `mem` parameters of the `Model` class. These apply to the `init_container` that pulls or creates the model. This separation ensures that the potentially resource-intensive model download/creation phase has dedicated resources, preventing it from starving the main application or the Ollama server itself.
*   **Input Downloader Resources:** Configured via `download_inputs_cpu` and `download_inputs_mem` parameters of the `Ollama` class. These apply to the `input-downloader` init container, which is only active when dynamic Modelfile generation is used.

### Integration and Best Practices

*   **Sidecar Pattern:** The Ollama server runs as a sidecar, meaning it shares the pod's network and volumes with the main application container. This allows the main application to communicate with Ollama via `http://localhost:11434`.
*   **Startup Probe:** The plugin configures a startup probe for the Ollama server container, ensuring that the main application container does not start until the Ollama service is ready to accept requests. The `failure_threshold` is set high (100) to accommodate potentially long model loading times.
*   **Shared Data Volume:** A `shared-data` volume is mounted at `/shared` in both the initialization containers and the main Ollama container. This volume is used to pass the generated `Modelfile` and downloaded `inputs.json` between containers.
*   **Error Handling:** The initialization container includes retry logic to wait for the Ollama service to become ready before attempting to pull or create a model. If the service does not become ready, the container exits with an error, failing the pod.
*   **GPU Allocation:** When `gpu` is set to a value greater than 0, the plugin requests `nvidia.com/gpu` resources for the Ollama server container, assuming a Kubernetes cluster with NVIDIA GPU support and the NVIDIA device plugin installed.
*   **Security Context:** The initialization container runs as `root` (`run_as_user=0`) to ensure necessary permissions for file operations and package installations (`pip install`).

### Limitations and Considerations

*   **Ollama Version:** The `ollama` Python client library is explicitly pinned to `0.3.3` within the initialization container. Ensure compatibility with the `ollama/ollama` Docker image version being used.
*   **Modelfile Paths:** When using `FROM ./model.gguf` in a Modelfile, ensure that `model.gguf` is present in the container's working directory or a mounted volume accessible to the Ollama server. The plugin's `input-downloader` can help with downloading Flyte `FlyteFile` inputs to `/tmp` which can then be referenced.
*   **Resource Overheads:** Be mindful of the combined resource requests (CPU, memory, GPU) for all containers within the pod (main application, Ollama server, init containers). Over-provisioning can lead to inefficient cluster utilization, while under-provisioning can cause performance issues or pod failures.
*   **Network Access:** The Ollama server runs locally within the pod. If external access to the Ollama API is required, additional Kubernetes services (e.g., `ClusterIP`, `NodePort`, `LoadBalancer`) would need to be configured separately. The plugin focuses on the sidecar deployment pattern.
<!--
key: summary_ollama_inference_plugin_b5f44a22-2ab6-4d42-8c56-033e99366b81
type: summary_end

-->
<!--
code_unit: flytekitplugins.inference.ollama.serve
code_unit_type: class
help_text: ''
key: example_988c5d6f-7189-4996-ac9f-adb08ee00516
type: example

-->