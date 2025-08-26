
<!--
help_text: ''
key: summary_core_extensibility_mechanisms_71fbd8fa-a9bb-4fa2-a99a-ddcf368a00b5
modules:
- flytekit.core.python_auto_container
- flytekit.core.python_customized_container_task
- flytekit.core.shim_task
- flytekit.core.class_based_resolver
- flytekit.core.task
questions_to_answer: []
type: summary

-->
# Core Extensibility Mechanisms

Flytekit provides several core mechanisms to extend and customize how tasks are defined, executed, and resolved within the Flyte platform. These mechanisms enable developers to tailor task behavior, integrate with external systems, and optimize execution environments.

## Task Resolution

Task resolvers are fundamental components that determine how Flytekit locates and loads the executable code for a task at runtime. This allows for flexible deployment and execution strategies.

### Standard Task Resolution

The `DefaultTaskResolver` is the standard mechanism for resolving tasks. It identifies tasks by their Python module path and function name. When a task is executed, this resolver imports the specified module and retrieves the task object by name.

**Usage:** This resolver is automatically used for most Python tasks defined in standard `.py` files. It ensures that the task's code is available and discoverable within the container's Python environment.

### Notebook Task Resolution

For tasks defined within interactive environments like Jupyter notebooks, the `DefaultNotebookTaskResolver` provides a specialized resolution strategy. This resolver serializes task definitions into a pickled file (`.pkl`) during compilation. At execution time, it loads the task directly from this pickled file.

**Considerations:**
*   **Python Version Compatibility:** The `PickledEntity` and `PickledEntityMetadata` classes ensure that the Python version used to create the pickle file matches the runtime environment. Mismatches will raise an error, requiring the use of a compatible container image.
*   **Serialization Overhead:** Pickling can introduce overhead, and the `.pkl` file must be accessible to the task's container.

### Class-Based Task Resolution

The `ClassStorageTaskResolver` offers a unique approach where tasks are stored and resolved from an in-memory list associated with a class. This resolver identifies tasks by their index within this list.

**Use Cases:** This mechanism is particularly useful for advanced scenarios such as:
*   **Testing:** Storing and retrieving tasks programmatically for unit or integration tests.
*   **Dynamic Task Generation:** When tasks are generated at runtime and need to be referenced by an internal identifier rather than a static module path.

**Implementation:** To use this resolver, tasks are explicitly added to its internal mapping. The `loader_args` method returns the index of the task, which `load_task` then uses to retrieve the task instance.

## Customizing Containerized Tasks

The `PythonAutoContainerTask` serves as the base class for most Python tasks that run within a container. It provides extensive capabilities for customizing the container's environment and resource allocation.

**Key Customization Points:**
*   **Container Image:** Specify the Docker image to use for the task's execution. This can be a simple string or an `ImageSpec` for more advanced image management.
*   **Resource Allocation:** Define CPU, memory, and GPU requests and limits using `Resources` or the consolidated `resources` parameter. This ensures tasks receive adequate resources and adhere to cluster policies.
*   **Environment Variables:** Inject custom environment variables into the task's container using the `environment` parameter.
*   **Secrets:** Request and mount secrets into the container using `secret_requests`. Secrets are accessed via the `SecretManager` and are crucial for handling sensitive information securely.
*   **Pod Templates:** For fine-grained control over Kubernetes pod configuration, provide a `PodTemplate` or reference an existing `pod_template_name`. This allows customization of volumes, node selectors, and other pod-level settings.
*   **Accelerators:** Specify GPU accelerators for tasks requiring specialized hardware.
*   **Shared Memory:** Configure shared memory for inter-process communication within the container.

**Execution Command Customization:**
*   **`set_command_fn`:** Override the default `pyflyte-execute` command used to run the task. This is useful for specialized execution patterns, such as map tasks (`pyflyte-map-execute`) or fast-executed tasks.
*   **`reset_command_fn`:** Reverts the command to the default `pyflyte-execute` arguments.

## Building New Task Types (Plugin Authors)

For developers creating new Flytekit plugins or custom task types that require unique execution logic not covered by standard Python functions, the `PythonCustomizedContainerTask` and `ShimTaskExecutor` pattern offers a powerful extensibility point.

This pattern separates the task definition (serialized into a `TaskTemplate`) from its execution logic (implemented in a `ShimTaskExecutor`).

**Implementation Steps:**
1.  **Define the Executor:**
    *   Subclass `ShimTaskExecutor` and override the `execute_from_model` method. This method contains the core business logic for your custom task.
    *   The `execute_from_model` method receives the `TaskTemplate` (which includes the `custom` field) and the task's inputs as Python native values.
    *   **Limitation:** The executor only has access to information serialized within the `TaskTemplate`. Avoid relying on external Python objects or closures that are not part of the template.

    ```python
    from flytekit.core.shim_task import ShimTaskExecutor
    from flytekit.models.task import TaskTemplate
    from typing import Any

    class MyCustomExecutor(ShimTaskExecutor):
        def execute_from_model(self, tt: TaskTemplate, **kwargs) -> Any:
            # Access custom data from the task template
            custom_data = tt.custom.get("my_custom_key")
            print(f"Executing with custom data: {custom_data}")
            # Implement your task's business logic here
            return kwargs["input_arg"] * 2
    ```

2.  **Define the Custom Task:**
    *   Subclass `PythonCustomizedContainerTask`.
    *   In its `__init__` method, pass your custom `ShimTaskExecutor` type to the `executor_type` argument.
    *   Override the `get_custom` method to serialize any necessary information into the `TaskTemplate`'s `custom` field. This data will be available to your `ShimTaskExecutor` at runtime.

    ```python
    from flytekit.core.python_customized_container_task import PythonCustomizedContainerTask
    from flytekit.core.python_auto_container import PythonTask
    from flytekit.core.shim_task import ShimTaskExecutor
    from flytekit.core.base_task import TaskMetadata
    from flytekit.models.resources import Resources
    from flytekit.models.core.identifier import ResourceType
    from flytekit.models import task as _task_model
    from flytekit.models import interface as interface_models
    from flytekit.models import literals as literal_models
    from flytekit.core.type_engine import TypeEngine
    from flytekit.core.context_manager import FlyteContextManager
    from flytekit.core.interface import Interface
    from flytekit.core.task import TaskMetadata
    from flytekit.core.base_task import PythonTask
    from flytekit.core.python_customized_container_task import TaskTemplateResolver
    from flytekit.core.python_customized_container_task import default_task_template_resolver
    from flytekit.configuration import Image, ImageConfig, SerializationSettings
    from typing import Dict, Any, Type, Optional, List

    class MyCustomTask(PythonCustomizedContainerTask[Any]):
        def __init__(self, name: str, container_image: str, my_custom_value: str, **kwargs):
            super().__init__(
                name=name,
                task_config=None, # No specific task config for this example
                container_image=container_image,
                executor_type=MyCustomExecutor,
                task_type="my-custom-task",
                **kwargs,
            )
            self._my_custom_value = my_custom_value

        def get_custom(self, settings: SerializationSettings) -> Dict[str, Any]:
            # This data will be available in tt.custom within MyCustomExecutor
            return {"my_custom_key": self._my_custom_value}

    # Example usage:
    # @task(task_config=MyCustomTask(name="my_task", container_image="my_image", my_custom_value="hello"))
    # def my_workflow_task(input_arg: int) -> int:
    #     ... # The Python function body is not executed; MyCustomExecutor handles it.
    ```

3.  **Task Template Resolution:** The `TaskTemplateResolver` is specifically designed to load `PythonCustomizedContainerTask` instances. It retrieves the `TaskTemplate` and the `ShimTaskExecutor` class path, then instantiates an `ExecutableTemplateShimTask` to handle execution.

**Considerations:**
*   **`TaskTemplate` Size:** The `TaskTemplate` is frequently accessed by the Flyte engine, so keep its size small. Avoid embedding large datasets directly.
*   **IDL Interface:** The interface at execution time is derived from the Flyte IDL, which may result in some loss of Python-specific type information.

## Registering Task Plugins

The `TaskPlugins` factory provides a centralized mechanism for registering and discovering new `PythonFunctionTask` derivatives. This allows developers to extend Flytekit with custom task types that integrate with various services or provide specialized functionality.

**How it Works:**
*   Plugins are registered by mapping a `plugin_config_type` (a configuration object specific to the plugin) to a `plugin_object_type` (a class derived from `PythonFunctionTask`).
*   When a task is defined with a specific `task_config` type, `TaskPlugins` automatically finds and uses the corresponding plugin.

**Common Use Cases:**
*   **Database Query Tasks:** Plugins for Athena, Hive, or other SQL engines, where the `task_config` might specify connection details or query parameters.
*   **Machine Learning Operators:** Integrations with Kubeflow (PyTorch, TensorFlow) or other ML frameworks, where the `task_config` defines training parameters or model configurations.
*   **Generic Resource Tasks:** Plugins like `PodFunctionTask` that allow direct manipulation of Kubernetes resources.

**Example:**
```python
from flytekit.core.task import TaskPlugins, PythonFunctionTask
from dataclasses import dataclass

@dataclass
class MyPluginConfig:
    message: str

class MyPluginTask(PythonFunctionTask):
    def __init__(self, name: str, task_config: MyPluginConfig, **kwargs):
        super().__init__(task_type="my-plugin", name=name, task_config=task_config, **kwargs)

    def execute(self, **kwargs) -> Any:
        print(f"MyPluginTask received config message: {self.task_config.message}")
        print(f"MyPluginTask received inputs: {kwargs}")
        return "Plugin executed successfully"

# Register the plugin
TaskPlugins.register_pythontask_plugin(MyPluginConfig, MyPluginTask)

# Now, when you define a task with MyPluginConfig, MyPluginTask will be used
# @task(task_config=MyPluginConfig(message="Hello from my plugin!"))
# def my_plugin_workflow_task(input_val: str) -> str:
#     return input_val # This function body is replaced by MyPluginTask.execute
```

## Referencing Existing Tasks

The `ReferenceTask` class enables referencing tasks that are already registered on the Flyte platform. This is useful for building workflows that incorporate pre-existing, deployed tasks without needing to redefine their implementation locally.

**Key Aspect:** Only the signature (inputs and outputs) of the `ReferenceTask` is used. The actual Python function body provided during its definition is ignored. It is crucial that the signature of the `ReferenceTask` matches the signature of the remote task to avoid compilation or runtime errors.

**Usage:**
```python
from flytekit.core.task import ReferenceTask
from typing import Dict, Type

# Define a reference to an existing task on the Flyte platform
# The inputs and outputs must match the remote task's signature
my_remote_task = ReferenceTask(
    project="flyteexamples",
    domain="development",
    name="my_pre_existing_task",
    version="v1.0.0",
    inputs={"x": int, "y": str},
    outputs={"result": float},
)

# This reference task can now be used in workflows
# @workflow
# def my_workflow(a: int, b: str) -> float:
#     return my_remote_task(x=a, y=b)
```

## Example: The Echo Task

The `Echo` task serves as a simple, built-in example of a custom task type. It demonstrates how a task can be implemented to directly echo its inputs as outputs. This task is often handled by a specialized plugin within the Flyte propeller, meaning it does not require a separate container to execute, making it highly efficient for simple pass-through operations.

**Configuration:** To enable the `Echo` task, ensure the `echo` plugin is enabled in the Flyte propeller configuration.
<!--
key: summary_core_extensibility_mechanisms_71fbd8fa-a9bb-4fa2-a99a-ddcf368a00b5
type: summary_end

-->
<!--
code_unit: flytekit.core.python_auto_container.DefaultTaskResolver
code_unit_type: class
help_text: ''
key: example_bdb3e570-895d-4312-9179-0dc45bf22dbf
type: example

-->
<!--
code_unit: flytekit.core.python_customized_container_task.PythonCustomizedContainerTask
code_unit_type: class
help_text: ''
key: example_15019dc3-9ab2-40ca-8df2-c9f0f79f09b9
type: example

-->
<!--
code_unit: flytekit.core.shim_task.ShimTaskExecutor
code_unit_type: class
help_text: ''
key: example_ab4ca9ab-b7dc-47d5-b4da-9d1993e5ef74
type: example

-->
<!--
code_unit: flytekit.core.class_based_resolver.ClassStorageTaskResolver
code_unit_type: class
help_text: ''
key: example_76f848d2-43d3-4624-b36e-b7f1e8739cb2
type: example

-->
<!--
code_unit: flytekit.core.task.TaskPlugins
code_unit_type: class
help_text: ''
key: example_4842ac3e-def1-4dc9-9b0e-225faa1f2476
type: example

-->