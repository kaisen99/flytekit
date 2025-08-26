
<!--
help_text: ''
key: summary_experiment_tracking_integrations_ba6c9831-41f2-4143-90c7-a092b03493f3
modules:
- flytekitplugins.comet_ml.tracking
- flytekitplugins.wandb
- flytekitplugins.wandb.tracking
questions_to_answer: []
type: summary

-->
Experiment tracking is crucial for machine learning workflows, enabling users to log metrics, parameters, and artifacts, and to visualize and compare experiment runs. Flyte provides seamless integrations with popular experiment tracking platforms, allowing you to automatically initialize and manage experiment runs directly from your Flyte tasks.

These integrations leverage decorators to wrap your Flyte tasks, handling the necessary setup and teardown for each experiment run. This includes managing API keys securely and linking runs back to their corresponding Flyte executions.

### Core Concepts

**Decorator-Based Integration**
Experiment tracking integrations are implemented as Python decorators. Apply these decorators directly to your Flyte tasks to enable automatic experiment initialization when the task executes.

**API Key Management**
For secure access to tracking platforms, API keys are managed through Flyte's secret management system or by providing a callable that returns the API key.
*   **Flyte Secrets**: Recommended for remote executions. Define your API key as a Flyte `Secret` and reference it in the decorator. Flyte automatically injects the secret into the task's environment.
*   **Callable**: For more dynamic scenarios or local testing, provide a Python callable (a function) that returns the API key string. This callable takes no arguments.

**Experiment/Run ID Generation**
Each integration supports both automatic and custom experiment/run ID generation:
*   **Automatic IDs**: If you do not provide an explicit ID, the integration automatically generates one based on the Flyte execution context. For remote executions, this ID often incorporates the Flyte execution name, node ID, and retry attempt, ensuring uniqueness and traceability.
*   **Custom IDs**: You can specify a custom ID for your experiment or run. This is useful for resuming previous runs or maintaining specific naming conventions.

**Local vs. Remote Execution**
The behavior of the decorators adapts based on the execution environment:
*   **Local Execution**: When running tasks locally (e.g., using `pyflyte run`), the decorators use the provided or auto-generated ID directly. API keys are typically expected to be available in the local environment (e.g., via environment variables or the callable).
*   **Remote Execution**: When tasks are executed on a Flyte cluster, API keys are securely retrieved from Flyte secrets. Experiment/run IDs are automatically generated based on the Flyte execution context if not explicitly provided, ensuring unique tracking for each remote run.

### Comet ML Integration

The Comet ML integration allows you to log and track your machine learning experiments using the Comet ML platform. Apply the `comet_ml_login` decorator to your Flyte tasks.

**Usage**

To integrate Comet ML, decorate your task with `comet_ml_login`. You must provide the `project_name`, `workspace`, and a `secret` containing your Comet API key.

```python
import os
from flytekit import task, workflow, Secret
from flytekitplugins.comet_ml.tracking import _comet_ml_login_class as comet_ml_login

# Define a secret for your Comet ML API key
# In your Flyte project, configure a secret named 'comet-api-key' in group 'comet'
COMET_API_KEY_SECRET = Secret(group="comet", key="comet-api-key")

@comet_ml_login(
    project_name="my-flyte-project",
    workspace="my-workspace",
    secret=COMET_API_KEY_SECRET,
    # Optional: provide a custom experiment key
    # experiment_key="my-custom-experiment-id",
    # Optional: pass additional arguments to comet_ml.login
    # tags=["flyte", "example"],
)
@task
def train_model_comet(epochs: int) -> float:
    import comet_ml
    # The experiment is automatically initialized by the decorator
    # You can now log metrics, parameters, etc.
    experiment = comet_ml.get_experiment()
    if experiment:
        experiment.log_parameter("epochs", epochs)
        for i in range(epochs):
            loss = 1.0 / (i + 1)
            experiment.log_metric("loss", loss, step=i)
            print(f"Epoch {i+1}, Loss: {loss}")
        experiment.log_metric("final_loss", loss)
    else:
        print("Comet ML experiment not initialized (e.g., in local test without API key setup).")
    return loss

@workflow
def comet_workflow(epochs: int = 5):
    train_model_comet(epochs=epochs)
```

**Parameters**

*   `project_name` (str, required): The name of the Comet ML project where the experiment will be sent.
*   `workspace` (str, required): The workspace associated with the project.
*   `secret` (Secret or Callable, required): Your Comet ML API key. This can be a `flytekit.Secret` object referencing a configured secret in Flyte, or a callable that returns the API key string.
*   `experiment_key` (str, optional): A unique identifier for the Comet ML experiment. If omitted, an ID is automatically generated based on the Flyte execution.
*   `host` (str, optional): The URL to your Comet ML service. Defaults to `"https://www.comet.com"`.
*   `**login_kwargs` (dict, optional): Additional keyword arguments passed directly to `comet_ml.login` (or `comet_ml.init` for older Comet ML versions). Refer to the Comet ML documentation for available options.

### Weights & Biases (W&B) Integration

The Weights & Biases (W&B) integration allows you to track, visualize, and manage your machine learning experiments using the W&B platform. Apply the `wandb_init` decorator to your Flyte tasks.

**Usage**

To integrate W&B, decorate your task with `wandb_init`. You must provide the `project`, `entity`, and a `secret` containing your W&B API key.

```python
import os
from flytekit import task, workflow, Secret
from flytekitplugins.wandb.tracking import wandb_init

# Define a secret for your W&B API key
# In your Flyte project, configure a secret named 'wandb-api-key' in group 'wandb'
WANDB_API_KEY_SECRET = Secret(group="wandb", key="wandb-api-key")

@wandb_init(
    project="my-flyte-wandb-project",
    entity="my-wandb-entity",
    secret=WANDB_API_KEY_SECRET,
    # Optional: provide a custom run ID
    # id="my-custom-wandb-run-id",
    # Optional: pass additional arguments to wandb.init
    # tags=["flyte", "example"],
    # config={"learning_rate": 0.01},
)
@task
def train_model_wandb(epochs: int) -> float:
    import wandb
    # The run is automatically initialized by the decorator
    # You can now log metrics, parameters, etc.
    wandb.log({"epochs": epochs})
    for i in range(epochs):
        loss = 1.0 / (i + 1)
        wandb.log({"loss": loss}, step=i)
        print(f"Epoch {i+1}, Loss: {loss}")
    wandb.log({"final_loss": loss})
    # wandb.finish() is automatically called by the decorator
    return loss

@workflow
def wandb_workflow(epochs: int = 5):
    train_model_wandb(epochs=epochs)
```

**Parameters**

*   `project` (str, required): The name of the W&B project where the run will be sent.
*   `entity` (str, required): The W&B username or team name (entity) where the run will be sent.
*   `secret` (Secret or Callable, required): Your W&B API key. This can be a `flytekit.Secret` object referencing a configured secret in Flyte, or a callable that returns the API key string.
*   `id` (str, optional): A unique identifier for the W&B run. If omitted, an ID is automatically generated based on the Flyte execution.
*   `host` (str, optional): The URL to your W&B service. Defaults to `"https://wandb.ai"`.
*   `api_host` (str, optional): The URL to your W&B API host. Defaults to `"https://api.wandb.ai"`.
*   `**init_kwargs` (dict, optional): Additional keyword arguments passed directly to `wandb.init`. Refer to the W&B documentation for available options.

**Special Considerations for W&B**

*   **Flyte Execution URL Linking**: The W&B integration automatically injects the URL of the current Flyte execution into the W&B run's notes. This provides a direct link from your W&B dashboard back to the corresponding Flyte workflow execution, enhancing traceability.
*   **Run Finalization**: The `wandb_init` decorator ensures that `wandb.finish()` is called automatically after your task completes, properly finalizing the W&B run.

### Best Practices and Considerations

*   **Dependency Management**: Ensure that the `comet_ml` or `wandb` Python packages are included in your task's `requirements.txt` or environment definition.
*   **Secret Configuration**: For remote Flyte executions, always configure your API keys as Flyte secrets. This is the most secure way to manage sensitive credentials.
*   **Idempotency and Retries**: When using automatically generated IDs, each retry of a task will typically result in a new experiment/run being created. If you need to resume or update a specific experiment across retries, provide a custom `experiment_key` (Comet ML) or `id` (W&B).
*   **Logging within Tasks**: Once the decorator initializes the experiment/run, you can use the respective library's functions (e.g., `comet_ml.log_metric`, `wandb.log`) within your task function to log data.
*   **Observability**: The Flyte UI may provide direct links to the corresponding experiment runs in Comet ML or W&B, depending on the Flyte console configuration.
<!--
key: summary_experiment_tracking_integrations_ba6c9831-41f2-4143-90c7-a092b03493f3
type: summary_end

-->
<!--
code_unit: flytekitplugins.comet_ml.tracking._comet_ml_login_class
code_unit_type: class
help_text: ''
key: example_cb547f2b-c6b6-401e-93f3-28ef86168bf1
type: example

-->
<!--
code_unit: flytekitplugins.wandb.wandb_init
code_unit_type: class
help_text: ''
key: example_11031896-93d7-429b-b489-e21feab5612b
type: example

-->
<!--
code_unit: flytekitplugins.wandb.tracking.wandb_init
code_unit_type: class
help_text: ''
key: example_537d8386-d9fa-4994-b422-e25c7a13d7cf
type: example

-->