
<!--
help_text: ''
key: summary_nvidia_inference_microservices_(nim)_plugin_0a069c58-fcb6-4328-81a6-7c2a57aefcd7
modules:
- flytekitplugins.inference.nim.serve
- flytekitplugins.inference.sidecar_template
questions_to_answer: []
type: summary

-->
The NVIDIA Inference Microservices (NIM) Plugin facilitates the deployment of NVIDIA Inference Microservices within Flyte workflows. This plugin extends the base model inference capabilities to provide specialized configurations for NIM, including secure credential management, optimized resource allocation, and support for dynamic LoRA adapter loading.

### Core Concepts

NVIDIA Inference Microservices (NIM) are pre-built, optimized microservices designed to simplify the deployment of AI models. The NIM plugin integrates these services directly into Flyte, allowing developers to leverage Flyte's orchestration capabilities for managing the lifecycle of NIM-powered inference endpoints.

The plugin configures a Kubernetes pod template that hosts the NIM server. This template includes specific settings for GPU utilization, shared memory, and secure access to NVIDIA GPU Cloud (NGC) and Hugging Face resources.

### Configuration

The plugin provides two primary classes for configuration: `NIMSecrets` and `NIM`.

#### NIMSecrets

The `NIMSecrets` class manages the credentials required for accessing NVIDIA and Hugging Face resources. It ensures secure handling of sensitive information by referencing Kubernetes secrets.

**Parameters:**

*   `ngc_image_secret` (str): The name of the Kubernetes secret containing credentials for pulling images from NGC. This is essential for accessing NIM Docker images.
*   `ngc_secret_key` (str): The key within the NGC secret that holds the NGC API key.
*   `secrets_prefix` (str): The prefix that Flyte appends to all mounted secrets (e.g., `_UNION_` or `_FSEC_`). This prefix is used to construct the environment variable name for the NGC API key.
*   `ngc_secret_group` (Optional[str]): An optional group name for the NGC API key, used if secrets are organized into groups.
*   `hf_token_group` (Optional[str]): An optional group name for the Hugging Face token.
*   `hf_token_key` (Optional[str]): The key within the Hugging Face secret that holds the Hugging Face token. This is used for downloading LoRA adapters.

**Usage:**

```python
from flytekitplugins.inference.nim.serve import NIMSecrets

nim_secrets = NIMSecrets(
    ngc_image_secret="my-ngc-image-pull-secret",
    ngc_secret_key="ngc_api_key",
    secrets_prefix="_FSEC_",
    hf_token_key="hf_token",
)
```

#### NIM Class

The `NIM` class is the primary interface for defining the configuration of the NIM server pod. It inherits from `ModelInferenceTemplate`, providing a robust foundation for model serving while adding NIM-specific optimizations.

**Parameters:**

*   `secrets` (`NIMSecrets`): An instance of `NIMSecrets` containing the necessary credential configurations. This is a mandatory parameter.
*   `image` (str, optional): The Docker image for the NIM server. Defaults to `nvcr.io/nim/meta/llama3-8b-instruct:1.0.0`.
*   `health_endpoint` (str, optional): The health check endpoint for the NIM server. Defaults to `v1/health/ready`.
*   `port` (int, optional): The port on which the NIM server listens. Defaults to `8000`.
*   `cpu` (int, optional): The number of CPU cores requested. Defaults to `1`.
*   `gpu` (int, optional): The number of GPU cores requested. Defaults to `1`. This is typically `1` for a single GPU.
*   `mem` (str, optional): The amount of memory requested for the NIM server container (e.g., "20Gi"). Defaults to "20Gi".
*   `shm_size` (str, optional): The size of the shared memory volume (`/dev/shm`). This is crucial for large models and inter-process communication. Defaults to "16Gi".
*   `env` (Optional[dict[str, str]]): A dictionary of additional environment variables to set in the NIM server container. Refer to NVIDIA NIM documentation for available environment variables (e.g., `NIM_PEFT_SOURCE`).
*   `hf_repo_ids` (Optional[list[str]]): A list of Hugging Face repository IDs for LoRA adapters to be downloaded. If provided, an init container is added to download these adapters.
*   `lora_adapter_mem` (Optional[str]): The amount of memory requested for the init container responsible for downloading LoRA adapters. This is required if `hf_repo_ids` is specified.

**Key Features and Capabilities:**

*   **Resource Allocation:** Configures CPU, GPU, and memory requests and limits for the NIM server. It also sets up a dedicated shared memory volume (`/dev/shm`) with a configurable size, which is vital for the performance of large language models.
*   **Secure Credential Injection:** Automatically injects the `NGC_API_KEY` into the NIM server container using the provided `NIMSecrets`. It also handles the Hugging Face token for LoRA adapter downloads.
*   **LoRA Adapter Management:** When `hf_repo_ids` are specified, the plugin automatically adds an `init_container` to download the specified LoRA adapters from Hugging Face Hub. These adapters are then mounted into the main NIM server container at the path specified by the `NIM_PEFT_SOURCE` environment variable.
    *   **Important:** If using LoRA adapters, ensure the `NIM_PEFT_SOURCE` environment variable is set in the `env` parameter of the `NIM` class to specify the directory where LoRA adapters are expected by the NIM server.
*   **Cache Volume:** Mounts an `emptyDir` volume at `/opt/nim/.cache` for NIM's internal caching mechanisms.
*   **Inherited Capabilities:** Leverages `ModelInferenceTemplate` for standard model serving features, including:
    *   **Health Checks:** Configures a startup probe to ensure the NIM server is ready before the pod is considered healthy. The `failure_threshold` is increased to accommodate potentially long model initialization times.
    *   **Input Data Handling:** Supports the `download_inputs` feature, which can download Flyte task inputs (especially `FlyteFile`s) to a shared volume, making them accessible to the NIM server.

### Usage Example

This example demonstrates how to configure and use the `NIM` plugin to deploy a NIM server with LoRA adapter support.

```python
from flytekit import task, workflow
from flytekitplugins.inference.nim.serve import NIM, NIMSecrets
from flytekitplugins.inference.sidecar_template import PodTemplate

# 1. Define your NIMSecrets
nim_secrets = NIMSecrets(
    ngc_image_secret="my-ngc-image-pull-secret",  # Replace with your Kubernetes secret name
    ngc_secret_key="ngc_api_key",  # Replace with the key in your secret
    secrets_prefix="_FSEC_",  # Common Flyte secrets prefix
    hf_token_key="hf_token",  # Replace with the key for your HF token
)

# 2. Define the NIM configuration
# This example uses a Llama3 8B Instruct image and downloads a sample LoRA adapter.
nim_config = NIM(
    secrets=nim_secrets,
    image="nvcr.io/nim/meta/llama3-8b-instruct:1.0.0",
    cpu=2,
    gpu=1,
    mem="20Gi",
    shm_size="16Gi",
    env={
        "NIM_PEFT_SOURCE": "/opt/nim/lora_adapters",  # Crucial for LoRA adapters
        "NIM_MODEL_ID": "meta/llama3-8b-instruct", # Example NIM specific env var
    },
    hf_repo_ids=["tloen/alpaca-lora-7b"],  # Example LoRA adapter
    lora_adapter_mem="2Gi",  # Memory for the LoRA download init container
)

# 3. Create a Flyte task that uses the NIM configuration
@task(
    container_image="ghcr.io/flyteorg/flytekit:py3.12-latest", # A base image for the task container
    pod_template=PodTemplate(pod_spec=nim_config.pod_template.pod_spec),
    requests=nim_config.pod_template.pod_spec.containers[0].resources.requests,
    limits=nim_config.pod_template.pod_spec.containers[0].resources.limits,
)
def deploy_nim_inference_service() -> str:
    """
    This task deploys the NIM inference service.
    The actual inference logic would typically be in a separate task
    that interacts with the deployed service at nim_config.base_url.
    """
    # In a real scenario, you might wait for the service to be ready
    # or perform a health check here.
    # The NIM server will be running as a sidecar in this pod.
    print(f"NIM inference service deployed at: {nim_config.base_url}")
    return nim_config.base_url

# 4. Define a Flyte workflow
@workflow
def nim_deployment_workflow() -> str:
    service_url = deploy_nim_inference_service()
    return service_url

# To run this locally (for testing the Flyte task definition, not actual deployment):
if __name__ == "__main__":
    # This will print the generated pod spec and the base URL.
    # For actual deployment, register and execute the workflow on a Flyte cluster.
    print(f"Generated Pod Spec: {nim_config.pod_template.pod_spec.to_dict()}")
    print(f"Base URL for NIM service: {nim_config.base_url}")
    # You would typically run the workflow on a Flyte cluster:
    # nim_deployment_workflow()
```

### Important Considerations and Best Practices

*   **Kubernetes Secrets:** Ensure that the Kubernetes secrets referenced by `NIMSecrets` (`my-ngc-image-pull-secret`, `ngc_api_key`, `hf_token`) exist in the namespace where your Flyte tasks will run and are correctly populated with the necessary credentials.
*   **`NIM_PEFT_SOURCE`:** When using `hf_repo_ids` for LoRA adapters, it is critical to set the `NIM_PEFT_SOURCE` environment variable in the `env` dictionary of the `NIM` class. This variable tells the NIM server where to find the downloaded LoRA adapters. The path specified here must match the `mount_path` for the `lora` volume in the `init_container`.
*   **Memory for LoRA Downloads:** The `lora_adapter_mem` parameter should be set appropriately to ensure the `init_container` has sufficient memory to download the LoRA adapter files. Larger adapters will require more memory.
*   **GPU Resources:** The `gpu` parameter should typically be set to `1` for a single GPU instance. Ensure your Kubernetes cluster has NVIDIA GPU nodes available and configured with the `nvidia.com/gpu` resource type.
*   **Shared Memory (`shm_size`):** The `shm_size` parameter directly impacts the performance of large models. Insufficient shared memory can lead to out-of-memory errors or degraded performance. Adjust this value based on the specific model requirements.
*   **Startup Probe Threshold:** The `startup_probe` `failure_threshold` is intentionally set high (100) to accommodate the potentially long initialization times of large language models. Avoid reducing this value unless you are certain your model initializes quickly.
*   **Base Image for Task:** The `container_image` for the Flyte task itself (e.g., `ghcr.io/flyteorg/flytekit:py3.12-latest`) is separate from the `image` used for the NIM server. The task container is where your Flyte code runs, while the NIM server runs as a sidecar.
<!--
key: summary_nvidia_inference_microservices_(nim)_plugin_0a069c58-fcb6-4328-81a6-7c2a57aefcd7
type: summary_end

-->
<!--
code_unit: flytekitplugins.inference.nim.serve
code_unit_type: class
help_text: ''
key: example_56a58acc-08cb-4b50-8eee-94d23d9ffc45
type: example

-->