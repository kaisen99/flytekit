
<!--
help_text: ''
key: summary_custom_container_tasks_c174235e-7a92-46db-a5f8-01ebc77f157d
modules:
- flytekit.core.python_customized_container_task
questions_to_answer: []
type: summary

-->
Custom Container Tasks enable you to define Flyte tasks where the core execution logic runs within a custom container image, distinct from the standard Flytekit Python environment. This capability is particularly useful for integrating non-Python code, leveraging specialized runtime environments, or managing complex dependencies that are difficult to package with typical Python tasks.

### Core Components

Custom Container Tasks rely on two primary components:

*   **`PythonCustomizedContainerTask`**: This is the base class you extend to define your custom container task. It bridges your Python task definition with the external container execution.
*   **`TaskTemplateResolver`**: This specialized resolver operates at runtime within the execution container. It is responsible for loading the task's serialized template and invoking the specific executor that contains your business logic.

#### `PythonCustomizedContainerTask`

The `PythonCustomizedContainerTask` class allows you to specify the container image where your task's logic will run and to pass custom configuration to that logic.

When defining a custom container task, you subclass `PythonCustomizedContainerTask` and provide the following:

*   **`name`**: A unique identifier for the task.
*   **`task_config`**: An optional configuration object specific to your task type.
*   **`container_image`**: The Docker image where the task's execution logic resides. This image must contain the necessary runtime environment and your `ShimTaskExecutor` implementation.
*   **`executor_type`**: A reference to a class that subclasses `ShimTaskExecutor`. This executor class encapsulates the actual business logic that will run inside your `container_image`.
*   **`requests` and `limits`**: Define the compute resources (CPU, memory, GPU) required by the container.
*   **`environment`**: A dictionary of environment variables to set within the container.
*   **`secret_requests`**: A list of `Secret` objects specifying secrets to be mounted into the container.

A critical method to override in your `PythonCustomizedContainerTask` subclass is `get_custom`. This method allows you to serialize any custom data or configuration from your task definition into the `TaskTemplate`. This data is then accessible by your `ShimTaskExecutor` at runtime.

```python
from flytekit import Resources, Secret
from flytekit.core.python_customized_container_task import PythonCustomizedContainerTask
from flytekit.extend import ShimTaskExecutor, FlyteContext, TaskTemplate
from typing import Dict, Any, Type

# Assume MyCustomExecutor is defined elsewhere, inheriting from ShimTaskExecutor
# and implementing execute_from_model

class MyCustomTask(PythonCustomizedContainerTask):
    def __init__(self, name: str, my_custom_setting: str, container_image: str):
        super().__init__(
            name=name,
            task_config={}, # Or a specific task config object
            container_image=container_image,
            executor_type=MyCustomExecutor, # Your ShimTaskExecutor subclass
            requests=Resources(cpu="1", mem="500Mi"),
            environment={"MY_ENV_VAR": "some_value"},
            secret_requests=[Secret(group_name="my-secrets", key="api_key")]
        )
        self._my_custom_setting = my_custom_setting

    def get_custom(self, settings) -> Dict[str, Any]:
        """
        Override this method to pass custom data to your ShimTaskExecutor.
        This data will be serialized into the TaskTemplate.
        """
        return {"my_custom_setting": self._my_custom_setting}

# Example ShimTaskExecutor (must be in the container image)
class MyCustomExecutor(ShimTaskExecutor):
    def execute_from_model(self, context: FlyteContext, task_template: TaskTemplate, inputs: Dict[str, Any]) -> Dict[str, Any]:
        # Access custom data passed from MyCustomTask
        custom_data = task_template.custom
        my_setting = custom_data.get("my_custom_setting")
        print(f"Executing with custom setting: {my_setting}")

        # Your actual business logic goes here
        # Process inputs and return outputs
        return {"output_key": "processed_data"}
```

The `get_command` method within `PythonCustomizedContainerTask` defines how the `pyflyte-execute` command is constructed for the container. This command includes arguments for the `TaskTemplateResolver`, ensuring it can locate and load the necessary task template and executor at runtime.

#### `TaskTemplateResolver`

The `TaskTemplateResolver` is a specialized component that facilitates the execution of custom container tasks. Unlike other resolvers that might restore the exact Python object, this resolver always loads an `ExecutableTemplateShimTask`. This `ExecutableTemplateShimTask` is constructed solely from the `TaskTemplate`, meaning it only has access to information that was serialized into the template.

At runtime, within the custom container:

1.  The `pyflyte-execute` command is invoked.
2.  The `TaskTemplateResolver` is used to resolve the task.
3.  Its `load_task` method is called, which:
    *   Downloads the `TaskTemplate` protobuf file (identified by `{{.taskTemplatePath}}`).
    *   Dynamically loads the `ShimTaskExecutor` class (identified by its fully qualified name, e.g., `my_module.MyCustomExecutor`).
    *   Instantiates an `ExecutableTemplateShimTask` using the loaded template and executor.
4.  The `execute_from_model` method of your `ShimTaskExecutor` is then invoked, receiving the `TaskTemplate` (including your custom data) and the task inputs.

The `loader_args` method of the `TaskTemplateResolver` is crucial for this process. It generates the arguments that `pyflyte-execute` uses to tell the resolver where to find the task template and which executor class to load. For `PythonCustomizedContainerTask`, these arguments are typically `["{{.taskTemplatePath}}", "path.to.your.executor"]`.

### Implementation Guide

To implement a Custom Container Task:

1.  **Define Your `ShimTaskExecutor`**:
    Create a Python class that inherits from `flytekit.extend.ShimTaskExecutor`. This class will contain the actual business logic of your task. You must override the `execute_from_model` method. This method receives the `FlyteContext`, the `TaskTemplate` (which includes any custom data you serialized), and the task inputs.

    ```python
    # my_executor_module.py
    from flytekit.extend import ShimTaskExecutor, FlyteContext, TaskTemplate
    from typing import Dict, Any

    class MyCustomExecutor(ShimTaskExecutor):
        def execute_from_model(self, context: FlyteContext, task_template: TaskTemplate, inputs: Dict[str, Any]) -> Dict[str, Any]:
            # Access custom data from the task template
            custom_data = task_template.custom
            my_setting = custom_data.get("my_custom_setting", "default_value")
            print(f"Executor received custom setting: {my_setting}")

            # Access inputs
            input_value = inputs.get("input_name")
            print(f"Executor received input: {input_value}")

            # Perform your core logic here
            result = f"Processed {input_value} with setting {my_setting}"

            # Return outputs as a dictionary
            return {"output_name": result}
    ```

2.  **Define Your `PythonCustomizedContainerTask`**:
    Create a Python class that inherits from `flytekit.core.python_customized_container_task.PythonCustomizedContainerTask`. In its `__init__` method, pass your `ShimTaskExecutor` class as the `executor_type` and specify the `container_image`. Crucially, override the `get_custom` method to return a dictionary of any data your `ShimTaskExecutor` needs at runtime. This data will be embedded in the `TaskTemplate`.

    ```python
    # my_task_definition.py
    from flytekit import task
    from flytekit.core.python_customized_container_task import PythonCustomizedContainerTask
    from typing import Dict, Any

    # Assuming my_executor_module.MyCustomExecutor is available in the container
    # and its path is known for the executor_type argument.
    # For simplicity, we'll import it directly here, but in a real scenario,
    # the executor code would be part of the container image.
    from my_executor_module import MyCustomExecutor

    # Define your custom container image
    # This image must contain my_executor_module.py and its dependencies
    CUSTOM_IMAGE = "your_registry/your_custom_image:latest"

    class MyCustomContainerTask(PythonCustomizedContainerTask):
        def __init__(self, name: str, config_param: str):
            super().__init__(
                name=name,
                container_image=CUSTOM_IMAGE,
                executor_type=MyCustomExecutor, # Reference to your ShimTaskExecutor
                # You can also specify resources, environment variables, secrets here
            )
            self._config_param = config_param

        def get_custom(self, settings) -> Dict[str, Any]:
            """
            Pass custom configuration to the executor via the TaskTemplate.
            """
            return {"config_param_for_executor": self._config_param}

    # Instantiate and use your custom task
    @task
    def my_workflow_task(input_data: str) -> str:
        # The name should be unique across your project/domain
        custom_task_instance = MyCustomContainerTask(
            name="my_unique_custom_task_name",
            config_param="special_value"
        )
        # Invoke the custom task instance
        return custom_task_instance(input_name=input_data)

    # Example usage in a workflow
    from flytekit import workflow

    @workflow
    def my_custom_workflow(initial_data: str) -> str:
        return my_workflow_task(input_data=initial_data)

    # To run locally:
    # if __name__ == "__main__":
    #     print(my_custom_workflow(initial_data="hello world"))
    ```

3.  **Build Your Container Image**:
    Ensure your `container_image` includes your `ShimTaskExecutor` class and all its dependencies. The `pyflyte-execute` command, which is part of the Flytekit installation, must also be available in this image. A common practice is to build on top of a Flytekit base image.

4.  **Register and Execute**:
    Register your Flyte project containing the `PythonCustomizedContainerTask` definition. When the task executes on the Flyte platform, Flyte will launch your specified `container_image`, and the `pyflyte-execute` command within it will use the `TaskTemplateResolver` to load and run your `ShimTaskExecutor`.

### Execution Flow

The execution of a Custom Container Task follows these steps:

1.  **Task Definition**: You define your `PythonCustomizedContainerTask` subclass, specifying the `container_image` and the `executor_type`. You also implement `get_custom` to pass any necessary runtime configuration.
2.  **Serialization**: During `pyflyte register` or local execution, the `PythonCustomizedContainerTask` is serialized into a `TaskTemplate` protobuf. The data returned by your `get_custom` method is embedded within this `TaskTemplate`. The `TaskTemplateResolver.loader_args` also determines the arguments needed to load your executor at runtime.
3.  **Registration**: The generated `TaskTemplate` is registered with the Flyte backend.
4.  **Execution**: When the task is triggered on the Flyte platform:
    *   Flyte provisions a pod and launches the `container_image` you specified.
    *   Inside the container, the `pyflyte-execute` command is run. This command includes arguments generated by the `TaskTemplateResolver.loader_args`, pointing to the `TaskTemplate` file and the fully qualified name of your `ShimTaskExecutor` class.
    *   The `TaskTemplateResolver`'s `load_task` method is invoked. It retrieves the `TaskTemplate` from the provided path and dynamically loads your `ShimTaskExecutor` class.
    *   Finally, the `execute_from_model` method of your `ShimTaskExecutor` is called, receiving the loaded `TaskTemplate` (with your custom data) and the task's inputs. Your business logic then executes.

### Important Considerations

*   **`TaskTemplate` Size**: The `TaskTemplate`, including the data returned by your `get_custom` method, should remain small. It is frequently accessed by the Flyte engine, and large templates can impact performance. Avoid embedding large datasets or binaries directly.
*   **Data Transfer**: All information required by your `ShimTaskExecutor` at runtime must be either:
    *   Serialized into the `TaskTemplate` via `get_custom`.
    *   Pre-packaged within the `container_image` itself (e.g., pre-installed libraries, configuration files).
    *   Fetched at runtime (e.g., from external storage like S3, a database, or passed as task inputs).
*   **Container Image Contents**: Ensure your `container_image` is self-contained and includes all necessary dependencies for your `ShimTaskExecutor` to run successfully. This includes Python interpreters, libraries, and any non-Python binaries.
*   **Resource Management**: Accurately specify `requests` and `limits` for CPU, memory, and GPU to ensure your task receives adequate resources and does not over-consume cluster capacity.
*   **Debugging**: Debugging custom container tasks can be more challenging than standard Python tasks due to the isolated container environment. Rely heavily on logging within your `ShimTaskExecutor` to understand its behavior.
*   **`TaskTemplateResolver` Limitations**: The `TaskTemplateResolver`'s `get_all_tasks` method currently returns an empty list. This means it is not designed for discovering tasks at definition time but rather for loading a specific task at execution time.
<!--
key: summary_custom_container_tasks_c174235e-7a92-46db-a5f8-01ebc77f157d
type: summary_end

-->
<!--
code_unit: flytekit.core.python_customized_container_task
code_unit_type: class
help_text: ''
key: example_c10255a6-7880-4ed0-abb1-80acf6988525
type: example

-->