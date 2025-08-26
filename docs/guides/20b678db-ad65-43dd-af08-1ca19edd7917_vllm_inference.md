
<!--
help_text: ''
key: summary_vllm_inference_15464b2a-01b4-4941-a8bd-d5c18790cc65
modules:
- flytekitplugins.inference.vllm.serve
questions_to_answer: []
type: summary

-->
## vLLM Inference

vLLM Inference provides a robust and flexible framework for deploying vLLM-powered large language model (LLM) inference servers. This capability streamlines the process of serving LLMs by abstracting the underlying infrastructure setup and securely managing model access credentials.

### Core Components

The vLLM Inference capability is primarily managed through two key components:

*   **`VLLM` Class:** This class serves as the primary interface for configuring and deploying a vLLM inference server. It extends a base `ModelInferenceTemplate`, enabling the definition of a Kubernetes pod template for the inference service.
*   **`HFSecret` Class:** This class facilitates the secure management and injection of HuggingFace API tokens, which are often required for accessing private or gated models from the HuggingFace Hub.

### Configuring a vLLM Inference Server

To deploy a vLLM inference server, instantiate the `VLLM` class and provide the necessary configuration parameters.

#### Resource Allocation

Specify the computational resources required for the inference server using the `cpu`, `gpu`, and `mem` parameters during initialization.

*   `cpu`: The number of CPU cores requested.
*   `gpu`: The number of GPU cores requested. For vLLM, a GPU is typically essential for optimal performance.
*   `mem`: The amount of memory requested, specified as a string (e.g., "10Gi").

```python
from flytekitplugins.inference.vllm.serve import VLLM, HFSecret

# Example: Request 2 CPUs, 1 GPU, and 10Gi memory
vllm_server = VLLM(
    hf_secret=HFSecret(secrets_prefix="_FSEC_", hf_token_key="HF_TOKEN"),
    cpu=2,
    gpu=1,
    mem="10Gi",
)
```

#### HuggingFace Secret Management

Accessing private or gated models on the HuggingFace Hub requires authentication. The `HFSecret` class securely handles the injection of your HuggingFace token into the inference server's environment.

When initializing `HFSecret`:

*   `secrets_prefix`: The prefix that the underlying platform appends to all mounted secrets (e.g., `_UNION_` or `_FSEC_`). This prefix is crucial for correctly resolving the secret's path.
*   `hf_token_key`: The specific key name under which your HuggingFace token is stored in the secret.
*   `hf_token_group` (Optional): If your token is part of a specific secret group, provide its name.

The `VLLM` class automatically configures the `HUGGING_FACE_HUB_TOKEN` environment variable within the inference container using the provided `HFSecret` details. This ensures that the vLLM server can authenticate with HuggingFace to download models.

```python
from flytekitplugins.inference.vllm.serve import HFSecret

# Assuming your HuggingFace token is stored under the key "HF_TOKEN"
# and the platform uses "_FSEC_" as a secret prefix.
hf_secret_config = HFSecret(
    secrets_prefix="_FSEC_",
    hf_token_key="HF_TOKEN",
    hf_token_group="my_hf_group" # Optional: if your token is part of a group
)

# Pass this configuration to the VLLM instance
vllm_server = VLLM(hf_secret=hf_secret_config, ...)
```

#### VLLM Engine Arguments

The `VLLM` class allows you to pass a dictionary of arguments directly to the vLLM model server using the `arg_dict` parameter. This enables fine-grained control over the vLLM engine's behavior, such as specifying the model path, tensor parallelism, or quantization settings.

Refer to the official vLLM documentation for a comprehensive list of available engine arguments. The `build_vllm_args` method within the `VLLM` class converts this dictionary into the command-line arguments expected by the vLLM server.

```python
from flytekitplugins.inference.vllm.serve import VLLM, HFSecret

# Example: Specify a model and enable GPU memory utilization
vllm_args = {
    "model": "mistralai/Mistral-7B-Instruct-v0.2",
    "gpu-memory-utilization": 0.9,
    "max-model-len": 4096,
    "dtype": "bfloat16",
}

vllm_server = VLLM(
    hf_secret=HFSecret(secrets_prefix="_FSEC_", hf_token_key="HF_TOKEN"),
    arg_dict=vllm_args,
    gpu=1, # Ensure GPU is requested if using GPU-specific args
)
```

#### Image and Network Configuration

You can customize the Docker image used for the vLLM server, as well as its network settings:

*   `image`: The Docker image to use. The default is `vllm/vllm-openai`, which provides an OpenAI-compatible API endpoint.
*   `health_endpoint`: The HTTP endpoint for health checks (default: `/health`).
*   `port`: The port number on which the model server listens (default: `8000`).

```python
from flytekitplugins.inference.vllm.serve import VLLM, HFSecret

vllm_server = VLLM(
    hf_secret=HFSecret(secrets_prefix="_FSEC_", hf_token_key="HF_TOKEN"),
    image="my_custom_vllm_image:latest", # Use a custom image
    health_endpoint="/vllm-health",      # Custom health check path
    port=8080,                           # Listen on a different port
)
```

### Deployment and Integration

The `VLLM` class, once configured, sets up a Kubernetes pod template. This template defines the container, environment variables (including the HuggingFace token), and command-line arguments necessary to run the vLLM server. The underlying platform then uses this template to deploy and manage the inference service.

### Best Practices and Considerations

*   **GPU Requirements:** vLLM is highly optimized for GPU inference. Ensure that you request sufficient `gpu` resources and that your environment supports GPU-enabled containers.
*   **HuggingFace Token Security:** Always manage your HuggingFace tokens securely using the platform's secret management capabilities. Avoid hardcoding tokens directly in your code.
*   **VLLM Argument Validation:** While the `arg_dict` provides flexibility, ensure that the arguments passed are valid for your chosen vLLM version and model. Incorrect arguments can lead to server startup failures.
*   **Model Compatibility:** Verify that the vLLM version in your chosen Docker image is compatible with the LLM you intend to serve.
*   **Resource Sizing:** Accurately estimate the CPU, GPU, and memory requirements for your specific LLM to prevent out-of-memory errors or performance bottlenecks. Larger models and higher batch sizes typically require more resources.
<!--
key: summary_vllm_inference_15464b2a-01b4-4941-a8bd-d5c18790cc65
type: summary_end

-->
<!--
code_unit: flytekitplugins.inference.vllm.examples.vllm_llm_serving
code_unit_type: class
help_text: ''
key: example_422c71e1-3591-40a0-964f-12a2c74ce934
type: example

-->