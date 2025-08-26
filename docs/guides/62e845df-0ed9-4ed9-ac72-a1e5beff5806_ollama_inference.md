
<!--
help_text: ''
key: summary_ollama_inference_024a567a-6e45-49f7-b501-a143f82b43b9
modules:
- flytekitplugins.inference.ollama.serve
questions_to_answer: []
type: summary

-->
## Ollama Inference

Ollama Inference enables the deployment and management of Ollama models within a Kubernetes environment. It automates the provisioning of Ollama servers, handling model pulling, custom model creation from Modelfiles, and resource allocation for efficient inference.

### Model Configuration

The `Model` class defines the specific Ollama model to be served. It encapsulates the model's identity and resource requirements for its initial setup.

*   **`name`**: A string representing the name of the model. This is the identifier Ollama uses (e.g., "llama2", "my-custom-model").
*   **`mem`**: A string specifying the memory allocated for the model during its initial setup (pulling or creation). Default is "500Mi". This resource is for the `init_container` that prepares the model.
*   **`cpu`**: An integer representing the number of CPU cores allocated for the model during its initial setup. Default is 1. This resource is for the `init_container` that prepares the model.
*   **`modelfile`**: An optional JSON-serializable string containing the content of a custom Modelfile. If provided, Ollama creates a model from this definition instead of pulling a pre-existing one. This allows for highly customized models, including those based on local weights or specific configurations.

**Example:**

```python
from flytekitplugins.inference.ollama.serve import Model

# To pull a pre-trained model
llama2_model = Model(name="llama2")

# To create a custom model from a Modelfile
custom_modelfile_content = """
FROM llama2
PARAMETER temperature 0.7
SYSTEM "You are a helpful assistant."
"""
custom_model = Model(name="my-custom-model", modelfile=custom_modelfile_content)
```

### Ollama Deployment Configuration

The `Ollama` class orchestrates the deployment of an Ollama server and the specified model within a Kubernetes pod. It extends a base `ModelInferenceTemplate` to manage the container image, ports, and resource requests for both the main Ollama serving container and the model provisioning step.

*   **`model`**: An instance of the `Model` class, defining the Ollama model to be deployed.
*   **`image`**: The Docker image for the Ollama server container. Defaults to `"ollama/ollama"`.
*   **`port`**: The port number on which the Ollama server exposes its service within the container. Defaults to `11434`.
*   **`cpu`**: The number of CPU cores requested for the *main* Ollama serving container. Defaults to `1`.
*   **`gpu`**: The number of GPUs requested for the *main* Ollama serving container. Defaults to `1`.
*   **`mem`**: The amount of memory requested for the *main* Ollama serving container. Defaults to `"15Gi"`.
*   **`download_inputs_mem`**: The memory requested for a potential input download step, if the `modelfile` requires external inputs. Defaults to `"500Mi"`.
*   **`download_inputs_cpu`**: The CPU cores requested for a potential input download step. Defaults to `2`.

The `Ollama` class automatically configures an `init_container` within the Kubernetes pod. This `init_container` is responsible for preparing the Ollama model before the main Ollama server starts. It either pulls a pre-trained model or creates a custom model based on the provided `modelfile`.

### Model Provisioning

Model provisioning is handled by an `init_container` that runs before the main Ollama server starts. This ensures the model is ready for inference as soon as the server is operational. The `init_container` uses a `python:3.11-slim` image and installs necessary dependencies (`requests`, `ollama==0.3.3`).

The provisioning process includes a readiness check that polls the Ollama service endpoint to ensure it is available before attempting to pull or create a model.

#### Pulling Pre-trained Models

When the `Model` instance is initialized without a `modelfile` (i.e., `modelfile=None`), the `init_container` executes an `ollama pull` command. This downloads the specified model from the Ollama registry.

**Example:**

```python
from flytekitplugins.inference.ollama.serve import Model, Ollama

# Define the model to pull
llama2_model = Model(name="llama2")

# Configure Ollama deployment to pull llama2
ollama_deployment = Ollama(
    model=llama2_model,
    cpu=2,
    mem="8Gi",
    gpu=0, # No GPU needed for this example
)

# The ollama_deployment object now contains the Kubernetes pod template
# configured to pull 'llama2' on startup.
```

#### Creating Models from a Modelfile

If a `modelfile` string is provided in the `Model` instance, the `init_container` creates a local Modelfile and then executes an `ollama create` command. The `modelfile` content is base64 encoded for safe transmission and then decoded within the `init_container`.

**Example:**

```python
from flytekitplugins.inference.ollama.serve import Model, Ollama

# Define a custom Modelfile
custom_modelfile_content = """
FROM llama2
PARAMETER temperature 0.7
SYSTEM "You are a helpful assistant. Always respond concisely."
"""

# Define the model using the custom Modelfile
my_custom_model = Model(name="my-concise-llama", modelfile=custom_modelfile_content)

# Configure Ollama deployment to create the custom model
ollama_custom_deployment = Ollama(
    model=my_custom_model,
    cpu=2,
    mem="8Gi",
    gpu=0,
)
```

#### Dynamic Modelfiles with External Inputs

The `modelfile` supports dynamic content through the `"{inputs"` placeholder. If the `modelfile` string contains this placeholder, the system expects an `inputs.json` file to be available in the `/shared` volume within the `init_container`. This `inputs.json` file is loaded, and its contents are used to format the `modelfile` string dynamically. This is particularly useful for scenarios where model parameters or configurations are determined at runtime or depend on external data.

The `download_inputs` parameter in the `Ollama` constructor (derived from the presence of `"{inputs"` in the `modelfile`) signals the underlying system to prepare these inputs.

**Example:**

Assume an `inputs.json` file with content like `{"model_version": "llama2", "temp": 0.5}` is made available in `/shared`.

```python
from flytekitplugins.inference.ollama.serve import Model, Ollama

# Define a Modelfile that uses dynamic inputs
dynamic_modelfile_content = """
FROM {inputs.model_version}
PARAMETER temperature {inputs.temp}
SYSTEM "You are a helpful assistant."
"""

# Define the model with the dynamic Modelfile
dynamic_model = Model(name="my-dynamic-model", modelfile=dynamic_modelfile_content)

# Configure Ollama deployment for the dynamic model
# The 'download_inputs' flag will be set to True automatically
# because '{inputs' is found in the modelfile.
ollama_dynamic_deployment = Ollama(
    model=dynamic_model,
    cpu=2,
    mem="8Gi",
    gpu=0,
    download_inputs_mem="1Gi", # Allocate more memory for input download if needed
)

# When this deployment runs, the 'init_container' will:
# 1. Download inputs into /shared/inputs.json (handled by the base template).
# 2. Read inputs.json.
# 3. Format the modelfile string using the inputs.
# 4. Create the model using the formatted Modelfile.
```

### Resource Management

Ollama Inference allows fine-grained control over resource allocation for both the model provisioning step and the main Ollama serving container:

*   **Model Provisioning Resources (`Model` class):**
    *   `Model.cpu` and `Model.mem` define the CPU and memory limits for the `init_container` that pulls or creates the model. These resources are temporary and released after the model is ready.
*   **Ollama Server Resources (`Ollama` class):**
    *   `Ollama.cpu`, `Ollama.gpu`, and `Ollama.mem` define the CPU, GPU, and memory requests/limits for the *main* Ollama serving container. These resources are persistent for the lifetime of the inference pod.
*   **Input Download Resources (`Ollama` class):**
    *   `Ollama.download_inputs_cpu` and `Ollama.download_inputs_mem` are allocated for a separate step (managed by the base template) that downloads external inputs if the `modelfile` is dynamic.

### Considerations and Best Practices

*   **Ollama Service Readiness:** The `init_container` includes a robust retry mechanism to wait for the Ollama service to become ready before attempting model operations. This ensures stability during startup.
*   **Volume Mounts:** The `init_container` utilizes `shared-data` and `tmp` volume mounts. `shared-data` is crucial for passing `inputs.json` when using dynamic Modelfiles.
*   **Security Context:** The `init_container` runs with `run_as_user=0` (root) to ensure necessary permissions for file operations and package installations.
*   **Ollama Client Version:** The `init_container` explicitly installs `ollama==0.3.3`. Ensure compatibility if using a different Ollama server version.
*   **Modelfile Escaping:** When providing a `modelfile` string, if it contains literal `{{` or `}}` that are *not* intended for dynamic input formatting, they are automatically escaped to `{{{{` and `}}}}` to prevent misinterpretation.
*   **Resource Sizing:** Carefully size the `cpu`, `gpu`, and `mem` parameters for the main Ollama container based on the chosen model's requirements and expected inference load. Over-provisioning can lead to wasted resources, while under-provisioning can cause performance issues or out-of-memory errors.
<!--
key: summary_ollama_inference_024a567a-6e45-49f7-b501-a143f82b43b9
type: summary_end

-->
<!--
code_unit: flytekitplugins.inference.ollama.examples.ollama_llm_serving
code_unit_type: class
help_text: ''
key: example_7ae839e3-8f4a-4dc8-b7a0-e51a95f95d90
type: example

-->