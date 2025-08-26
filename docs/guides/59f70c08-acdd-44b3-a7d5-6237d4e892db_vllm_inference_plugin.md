
<!--
help_text: ''
key: summary_vllm_inference_plugin_4e718ed6-cf60-4137-8a24-6f9f56a45ece
modules:
- flytekitplugins.inference.vllm.serve
- flytekitplugins.inference.sidecar_template
questions_to_answer: []
type: summary

-->
The VLLM Inference Plugin provides a streamlined way to deploy VLLM (Very Large Language Model) inference servers. It integrates VLLM models into a robust serving environment, handling resource allocation, health checks, and secure access to Hugging Face models.

### VLLM Model Deployment

The `VLLM` class is the primary interface for configuring and deploying a VLLM inference server. It extends a base model inference template, inheriting capabilities for Kubernetes pod management and resource orchestration.

To deploy a VLLM model, instantiate the `VLLM` class, providing essential configuration details:

*   **`hf_secret`**: An instance of `HFSecret` that specifies how to access Hugging Face API tokens. This is crucial for models requiring authentication.
*   **`arg_dict`**: A dictionary of arguments passed directly to the VLLM model server. These arguments control VLLM-specific behaviors, such as the model to load (`model`), tensor parallelism (`tensor_parallel_size`), and other engine configurations. Refer to the VLLM documentation for a comprehensive list of available arguments.
*   **`image`**: The Docker image for the VLLM model server. The default is `vllm/vllm-openai`, which provides an OpenAI-compatible API endpoint.
*   **`health_endpoint`**: The HTTP endpoint used for health checks. The default is `/health`.
*   **`port`**: The port on which the VLLM server listens. The default is `8000`.
*   **`cpu`**, **`gpu`**, **`mem`**: Resource requests for the model server container, specifying CPU cores, GPU units, and memory. VLLM models are typically GPU-intensive, so ensure adequate GPU allocation.

The `VLLM` class automatically constructs the necessary Kubernetes pod specification, including an `init_container` for the model server. It injects the Hugging Face token as an environment variable (`HUGGING_FACE_HUB_TOKEN`) into this container, enabling the VLLM server to download private or gated models from Hugging Face Hub.

**Example:**

```python
from flytekitplugins.inference.vllm.serve import VLLM, HFSecret

# Define Hugging Face secret details
# Assuming 'my-secrets-prefix' is configured in your Flyte environment
# and 'hf_token' is the key for your token within the 'my-hf-group' secret group.
hf_secret_config = HFSecret(
    secrets_prefix="my-secrets-prefix",
    hf_token_group="my-hf-group",
    hf_token_key="hf_token",
)

# Configure VLLM for a specific model
vllm_config = VLLM(
    hf_secret=hf_secret_config,
    arg_dict={
        "model": "mistralai/Mistral-7B-Instruct-v0.2",
        "tensor_parallel_size": 1,
        "max_model_len": 4096,
    },
    image="vllm/vllm-openai:latest", # Use a specific VLLM image version
    cpu=4,
    gpu=1,
    mem="20Gi",
)

# The 'vllm_config' object can now be used to define a Flyte task
# that deploys this VLLM inference server as a sidecar.
```

### Hugging Face Secret Management

The `HFSecret` class facilitates secure access to Hugging Face models by managing the necessary authentication token. It defines the structure for how the Hugging Face token is mounted as a secret within the deployment environment.

When initializing `HFSecret`, provide:

*   **`secrets_prefix`**: The prefix that the underlying platform (e.g., Flyte) appends to all mounted secrets. This is typically configured at the platform level (e.g., `_UNION_` or `_FSEC_`).
*   **`hf_token_key`**: The specific key name for the Hugging Face token within your secret.
*   **`hf_token_group`** (Optional): The group name for the Hugging Face token. If your token is part of a secret group, specify this.

The `VLLM` class uses these parameters to construct the environment variable name (`HUGGING_FACE_HUB_TOKEN`) that the VLLM server expects.

**Example:**

```python
from flytekitplugins.inference.vllm.serve import HFSecret

# Example for a token named 'my_hf_token' in a group 'my_group'
# with a platform prefix '_FSEC_'
hf_secret_grouped = HFSecret(
    secrets_prefix="_FSEC_",
    hf_token_group="my_group",
    hf_token_key="my_hf_token",
)

# Example for a token named 'hf_token' without a group
# with a platform prefix '_UNION_'
hf_secret_ungrouped = HFSecret(
    secrets_prefix="_UNION_",
    hf_token_key="hf_token",
)
```

### Resource Allocation and Health Checks

The VLLM Inference Plugin leverages a base `ModelInferenceTemplate` to define the Kubernetes pod structure. This template handles:

*   **Container Definition**: Sets up a `model-server` container with the specified image, port, and resource requests (CPU, GPU, memory).
*   **Resource Limits**: Configures both requests and limits for CPU, GPU, and memory to ensure predictable performance and resource consumption.
*   **Startup Probe**: Implements an HTTP GET startup probe against the `health_endpoint`. The `failure_threshold` for this probe is set to a high value (100) to accommodate the potentially long initialization time required for large language models to load.
*   **Input Downloader (Optional)**: If `download_inputs` is enabled in the underlying template (not directly exposed in `VLLM` but part of the base), an `input-downloader` `init_container` is added. This container downloads task inputs from the platform's file access system to a shared volume, making them accessible to the model server. This is useful for scenarios where the model server needs to consume input data directly from the task's execution context.

### Best Practices and Considerations

*   **GPU Requirements**: VLLM is designed for GPU acceleration. Ensure your Kubernetes cluster has nodes with sufficient GPU resources and that the `gpu` parameter is set appropriately.
*   **Hugging Face Token Security**: Always manage your Hugging Face tokens securely using the platform's secret management capabilities. The `HFSecret` class is designed to integrate with these mechanisms.
*   **VLLM Arguments**: Carefully review the VLLM documentation for `engine_args` to optimize model loading and inference behavior. Common arguments include `model`, `dtype`, `tensor_parallel_size`, `max_model_len`, and `gpu_memory_utilization`.
*   **Image Versioning**: Pin the `image` to a specific version (e.g., `vllm/vllm-openai:0.3.3`) to ensure reproducibility and avoid unexpected changes from new releases.
*   **Model Loading Time**: Be aware that loading large models can take significant time. The increased `startup_probe` threshold accounts for this, but monitor logs during initial deployments to understand typical startup durations.
*   **Scalability**: For high-throughput scenarios, consider the `tensor_parallel_size` argument to distribute the model across multiple GPUs within a single pod, or deploy multiple VLLM pods for horizontal scaling.
<!--
key: summary_vllm_inference_plugin_4e718ed6-cf60-4137-8a24-6f9f56a45ece
type: summary_end

-->
<!--
code_unit: flytekitplugins.inference.vllm.serve
code_unit_type: class
help_text: ''
key: example_6158eb17-447f-42c9-bddc-069d191971b8
type: example

-->