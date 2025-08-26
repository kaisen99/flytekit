
<!--
help_text: ''
key: summary_nvidia_nim_inference_b20cc3d3-8fa8-4a32-b630-cadf8e3e3884
modules:
- flytekitplugins.inference.nim.serve
questions_to_answer: []
type: summary

-->
NVIDIA NIM Inference provides a robust framework for deploying NVIDIA Inference Microservices (NIMs) within a Kubernetes environment. It streamlines the configuration of Kubernetes pods, containers, volumes, and secrets, enabling efficient and secure serving of AI models, including large language models (LLMs) with LoRA adapters.

### Managing Secrets for NIM Deployment

Deploying NVIDIA Inference Microservices requires secure access to various credentials, such as image pull secrets for NVIDIA GPU Cloud (NGC) and API keys for services like Hugging Face. The `NIMSecrets` class centralizes the management of these critical secrets.

The `NIMSecrets` class defines the following parameters:

*   `ngc_image_secret`: The name of the Kubernetes secret that contains credentials for pulling images from NGC. This secret must exist in your Kubernetes cluster.
*   `ngc_secret_key`: The key within the Kubernetes secret that holds the NGC API key.
*   `ngc_secret_group` (Optional): An optional group name for the NGC API key, used when secrets are organized into groups.
*   `secrets_prefix`: A prefix that Flyte appends to all mounted secrets (e.g., `_UNION_` or `_FSEC_`). This is crucial for correctly referencing secrets within the container environment.
*   `hf_token_key` (Optional): The key within the Kubernetes secret that holds the Hugging Face token, necessary for downloading models or LoRA adapters from Hugging Face Hub.
*   `hf_token_group` (Optional): An optional group name for the Hugging Face token.

**Example:**

```python
from flytekitplugins.inference.nim.serve import NIMSecrets

nim_secrets = NIMSecrets(
    ngc_image_secret="my-ngc-image-pull-secret",
    ngc_secret_key="ngc_api_key",
    secrets_prefix="_FSEC_",
    hf_token_key="hf_token",
    hf_token_group="huggingface",
)
```

### Configuring NVIDIA NIM Deployments

The `NIM` class extends `ModelInferenceTemplate` to define the Kubernetes pod template for your NVIDIA Inference Microservice. It encapsulates the necessary configurations for the model server container, resource allocation, and specialized requirements like shared memory and LoRA adapter management.

When initializing the `NIM` class, you configure the following parameters:

*   `secrets`: An instance of `NIMSecrets`, providing the necessary credentials for the deployment. This parameter is mandatory.
*   `image`: The Docker image for the model server container. The default is `nvcr.io/nim/meta/llama3-8b-instruct:1.0.0`.
*   `health_endpoint`: The health check endpoint for the model server. The default is `v1/health/ready`.
*   `port`: The port number on which the model server listens. The default is `8000`.
*   `cpu`: The number of CPU cores requested for the container. The default is `1`.
*   `gpu`: The number of GPU cores requested. The default is `1`.
*   `mem`: The amount of memory requested for the container (e.g., "20Gi"). The default is `20Gi`.
*   `shm_size`: The size of the shared memory volume (`/dev/shm`). This is critical for GPU-accelerated applications. The default is `16Gi`.
*   `env` (Optional): A dictionary of environment variables to set within the model server container. These can include NIM-specific configurations.
*   `hf_repo_ids` (Optional): A list of Hugging Face repository IDs for LoRA adapters to be downloaded. If provided, an init container is automatically configured to download these adapters.
*   `lora_adapter_mem` (Optional): The amount of memory requested for the init container responsible for downloading LoRA adapters. This is required if `hf_repo_ids` are specified.

**Example:**

```python
from flytekitplugins.inference.nim.serve import NIM, NIMSecrets

nim_secrets = NIMSecrets(
    ngc_image_secret="my-ngc-image-pull-secret",
    ngc_secret_key="ngc_api_key",
    secrets_prefix="_FSEC_",
    hf_token_key="hf_token",
    hf_token_group="huggingface",
)

nim_config = NIM(
    secrets=nim_secrets,
    image="nvcr.io/nim/meta/llama3-8b-instruct:1.0.0",
    cpu=2,
    gpu=1,
    mem="32Gi",
    shm_size="24Gi",
    env={
        "NIM_PEFT_SOURCE": "/opt/nim/peft_adapters",  # Required for LoRA adapters
        "NIM_MODEL_MAX_BATCH_SIZE": "128",
    },
    hf_repo_ids=["my-org/my-lora-adapter-1", "my-org/my-lora-adapter-2"],
    lora_adapter_mem="2Gi",
)

# The nim_config object can now be used to deploy the NIM via Flyte.
```

### Architectural Components and Capabilities

The `NIM` class configures several key Kubernetes components to ensure optimal performance and functionality for your inference service:

*   **Shared Memory (`/dev/shm`)**: A `dshm` volume is mounted to `/dev/shm` within the model server container. This volume uses `Memory` as its medium and is sized according to the `shm_size` parameter. Adequate shared memory is crucial for GPU-accelerated applications to prevent performance bottlenecks.
*   **Cache Volume**: A `cache` volume is mounted to `/opt/nim/.cache`, providing a persistent location for NIM's internal caching mechanisms.
*   **Image Pull Secrets**: The `ngc_image_secret` specified in `NIMSecrets` is automatically configured as an `imagePullSecret` for the pod, allowing the cluster to pull the NIM Docker image from NGC.
*   **NGC API Key Injection**: The NGC API key, retrieved from the Kubernetes secret using the `ngc_secret_key` and `ngc_secret_group` (if provided) along with the `secrets_prefix`, is injected into the model server container as the `NGC_API_KEY` environment variable.
*   **Security Context**: The model server container runs with a `securityContext` setting `run_as_user=1000`, enhancing security by running the process as a non-root user.

#### LoRA Adapter Management

A significant capability of the `NIM` class is its integrated support for downloading and managing LoRA (Low-Rank Adaptation) adapters from Hugging Face Hub. When `hf_repo_ids` are provided, the following occurs:

1.  **LoRA Volume**: A dedicated `lora` volume is created and mounted into the model server container at the path specified by the `NIM_PEFT_SOURCE` environment variable.
2.  **Init Container for Downloads**: An `init_container` named `download-loras` is added to the pod. This container executes before the main model server container starts.
    *   It uses a `python:3.12-alpine` image.
    *   It installs `huggingface_hub[cli]`.
    *   It logs into Hugging Face CLI using the token retrieved from the Kubernetes secret (if `hf_token_key` is provided).
    *   It downloads each specified LoRA adapter from `hf_repo_ids` into a subdirectory within the `NIM_PEFT_SOURCE` path on the `lora` volume.
    *   It sets appropriate permissions (`chmod -R 777`) on the downloaded files.
    *   Resource requests for this init container are set using `lora_adapter_mem`.

**Important Consideration for LoRA Adapters:**
For the LoRA adapters to be correctly mounted and utilized by the NIM, you **must** set the `NIM_PEFT_SOURCE` environment variable within the `env` dictionary passed to the `NIM` constructor. This variable specifies the directory where the LoRA adapters will be downloaded and from which the NIM will load them. If `NIM_PEFT_SOURCE` is not set, the `NIM` class will raise a `ValueError` when `hf_repo_ids` are provided.

### Best Practices and Considerations

*   **Secrets Management**: Always store sensitive information like NGC API keys and Hugging Face tokens in Kubernetes secrets. Ensure the `NIMSecrets` parameters correctly reference these secrets and their keys.
*   **Resource Sizing**: Carefully determine the `cpu`, `gpu`, `mem`, and `shm_size` based on the specific model being served and the expected workload. Under-provisioning can lead to performance issues, while over-provisioning wastes resources.
*   **LoRA Adapter Workflow**: When using LoRA adapters, ensure that:
    *   `hf_repo_ids` lists the correct Hugging Face repository IDs.
    *   `lora_adapter_mem` is set to provide sufficient memory for the download process.
    *   The `NIM_PEFT_SOURCE` environment variable is explicitly defined in the `env` parameter of the `NIM` class, matching the path where the NIM expects to find the adapters.
*   **Image Compatibility**: Verify that the chosen `image` is compatible with the specific NVIDIA Inference Microservice requirements and the model you intend to serve.
*   **Health Checks**: The default `health_endpoint` (`v1/health/ready`) is standard for NIMs. Ensure your NIM is configured to expose this endpoint correctly.
<!--
key: summary_nvidia_nim_inference_b20cc3d3-8fa8-4a32-b630-cadf8e3e3884
type: summary_end

-->
<!--
code_unit: flytekitplugins.inference.nim.examples.nim_model_serving
code_unit_type: class
help_text: ''
key: example_70e2827d-9f10-4228-b971-3565947a466b
type: example

-->