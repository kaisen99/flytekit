
<!--
help_text: ''
key: summary_weights_&_biases_(wandb)_tracking_2cb60dfb-a1c9-4abe-9aa7-59db646a31ea
modules:
- flytekitplugins.wandb
- flytekitplugins.wandb.tracking
questions_to_answer: []
type: summary

-->
Weights & Biases (WandB) Tracking

The Weights & Biases (WandB) tracking plugin provides a seamless way to integrate experiment tracking into Flyte tasks. It automatically handles the lifecycle of a WandB run, ensuring that metrics, configurations, and artifacts generated within a Flyte task are logged to the specified WandB project.

## Using the `wandb_init` Decorator

To enable WandB tracking for a Flyte task, apply the `wandb_init` decorator to the task function. This decorator wraps the task's execution, automatically initializing a WandB run before the task begins and finalizing it when the task completes.

```python
import wandb
from flytekit import task, workflow
from flytekitplugins.wandb.tracking import wandb_init
from flytekit.core.secrets import Secret, SecretString

# Define your WANDB_API_KEY secret in Flyte
# For example, in your project's `secrets.yaml` or via Flyte console
# secrets:
#   - name: wandb-api-key
#     group: wandb
#     key: WANDB_API_KEY

@wandb_init(
    project="my-flyte-experiments",
    entity="my-team",
    secret=Secret(group="wandb", key="WANDB_API_KEY"),
    # Optional: Pass additional wandb.init arguments
    config={"learning_rate": 0.01, "epochs": 10},
    tags=["flyte", "example", "training"],
)
@task
def train_model(epochs: int) -> float:
    """
    A Flyte task that simulates model training and logs metrics to WandB.
    """
    print(f"Starting training for {epochs} epochs...")
    for epoch in range(epochs):
        loss = 1.0 / (epoch + 1)
        accuracy = 0.5 + (epoch * 0.05)
        wandb.log({"loss": loss, "accuracy": accuracy, "epoch": epoch})
        print(f"Epoch {epoch}: Loss = {loss:.4f}, Accuracy = {accuracy:.4f}")
    
    final_accuracy = 0.95
    wandb.log({"final_accuracy": final_accuracy})
    print("Training complete.")
    return final_accuracy

@workflow
def training_workflow(num_epochs: int = 5) -> float:
    final_acc = train_model(epochs=num_epochs)
    return final_acc

```

When this workflow executes, the `train_model` task automatically initializes a WandB run, logs the specified metrics, and then finishes the run.

## Configuration Parameters

The `wandb_init` decorator accepts several parameters to configure the WandB run:

*   **`project`** (`str`, required): The name of the WandB project where the run's data is sent.
*   **`entity`** (`str`, required): The username or team name associated with the WandB project.
*   **`secret`** (`Union[Secret, Callable]`, required): Specifies how to retrieve the WandB API key.
    *   **`flytekit.Secret`**: The recommended method for remote execution. Provide a `Secret` object referencing the key and group where your `WANDB_API_KEY` is stored in Flyte's secret management system.
        ```python
        from flytekit.core.secrets import Secret
        # ...
        secret=Secret(group="wandb", key="WANDB_API_KEY")
        ```
    *   **`Callable`**: A Python callable (function or lambda) that takes no arguments and returns the API key as a string. This is useful for local testing or when the API key is sourced dynamically.
        ```python
        import os
        # ...
        secret=lambda: os.environ.get("WANDB_API_KEY")
        ```
*   **`id`** (`str`, optional): A unique identifier for the WandB run. If provided, WandB uses this ID.
*   **`host`** (`str`, optional, default: `"https://wandb.ai"`): The URL to your WandB service.
*   **`api_host`** (`str`, optional, default: `"https://api.wandb.ai"`): The URL to your WandB API host.
*   **`**init_kwargs`** (`dict`): Any additional keyword arguments are passed directly to the underlying `wandb.init()` function. This allows for extensive customization of the WandB run, such as setting `config`, `tags`, `notes`, `job_type`, etc. Refer to the [WandB `wandb.init` documentation](https://docs.wandb.ai/ref/python/init) for a complete list of available arguments.

## Run ID Management

The `wandb_init` decorator intelligently manages the WandB run ID based on the execution environment:

*   **Local Execution**: If the task runs locally (e.g., using `pyflyte run`), the decorator uses the `id` parameter if provided. If `id` is `None`, WandB generates its own unique ID for the run.
*   **Remote Execution**: When a task executes on a Flyte cluster, if the `id` parameter is `None`, the decorator automatically generates a WandB run ID derived from the Flyte execution context. This ID is typically based on the Flyte execution name, node ID, and retry attempt (e.g., `{.executionName}-{.nodeID}-{.taskRetryAttempt}`). This automatic ID generation ensures traceability between Flyte executions and their corresponding WandB runs. If an `id` is explicitly provided, it takes precedence.

## Linking Flyte Executions

For remote executions, the `wandb_init` decorator automatically injects a link to the Flyte execution URL into the notes section of the corresponding WandB run. This feature provides a direct navigation path from the WandB UI back to the specific Flyte execution that generated the run, enhancing experiment traceability and debugging.

## Best Practices and Considerations

*   **API Key Security**: Always manage your `WANDB_API_KEY` securely using Flyte's secret management system, especially for remote executions. Avoid hardcoding API keys directly in your code.
*   **WandB Library Installation**: Ensure that the `wandb` Python library is included in your task's dependencies (e.g., in your `requirements.txt` or `setup.py`) so it is available in the task's execution environment.
*   **Task Idempotency**: While WandB tracking itself does not directly affect task idempotency, ensure that the underlying task logic remains idempotent if that is a requirement for your workflow.
*   **Resource Usage**: Be mindful of the data being logged to WandB, especially large artifacts, as this can impact network usage and storage.
*   **Customization**: Leverage the `**init_kwargs` parameter to fine-tune your WandB runs, including setting custom configurations, tags, and notes, which are crucial for organizing and analyzing experiments.
<!--
key: summary_weights_&_biases_(wandb)_tracking_2cb60dfb-a1c9-4abe-9aa7-59db646a31ea
type: summary_end

-->
<!--
code_unit: flytekitplugins.wandb.examples.wandb_tracking_example
code_unit_type: class
help_text: ''
key: example_cff8ec40-0f78-4ab5-9a84-f6b94636c626
type: example

-->