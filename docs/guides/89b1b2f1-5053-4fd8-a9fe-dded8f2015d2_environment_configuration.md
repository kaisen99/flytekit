
<!--
help_text: ''
key: summary_environment_configuration_6d24ea79-7e50-4b53-986e-b047955cc6ab
modules:
- flytekit.core.environment
questions_to_answer: []
type: summary

-->
Environment Configuration

Environment configuration centralizes and reuses common settings for tasks and dynamic workflows, reducing boilerplate and promoting consistency across your codebase. This approach allows you to define a set of default or specialized configurations once and apply them to multiple components.

## The `Environment` Class

The `Environment` class, found in `flytekit.core.environment`, serves as a container for key-value overrides. These overrides are typically parameters that can be passed to task or dynamic decorators, such as resource requests (CPU, memory), container images, timeouts, or retry policies.

### Creating an Environment

Initialize an `Environment` instance by passing the desired overrides as keyword arguments.

```python
from flytekit.core.environment import Environment

# Define a common environment for CPU-intensive tasks
cpu_intensive_env = Environment(
    cpu="2",
    memory="4Gi",
    container_image="my_custom_cpu_image:latest"
)

# Define a lightweight environment for quick tasks
lightweight_env = Environment(
    cpu="500m",
    memory="1Gi",
    timeout="1m"
)
```

### Applying Environment to Tasks

An `Environment` instance acts as a decorator factory for `task` functions. You can apply an environment in two primary ways:

1.  **Direct Application:** Apply the environment directly to a task function. All overrides defined in the `Environment` instance are applied to the task.

    ```python
    from flytekit import task
    from flytekit.core.environment import Environment

    # Assume 'common_task_config' is an Environment instance
    common_task_config = Environment(retries=3, timeout="10m")

    @common_task_config
    def my_data_processing_task(data: str) -> str:
        # Task logic here
        return data.upper()
    ```

    This is equivalent to using `@common_task_config.task`.

2.  **Application with Additional Overrides:** Apply the environment and provide additional, task-specific overrides. These new overrides merge with and potentially override the values from the `Environment` instance.

    ```python
    from flytekit import task
    from flytekit.core.environment import Environment

    # Base environment with a default image
    base_image_env = Environment(container_image="default_python_image:3.9")

    @base_image_env(cpu="1", memory="2Gi") # Overrides 'base_image_env' with specific resources
    def my_ml_inference_task(model_input: list) -> list:
        # ML inference logic
        return model_input
    ```

    In this example, `my_ml_inference_task` will use `default_python_image:3.9` as its container image, along with 1 CPU and 2GiB of memory. The `Environment` class handles the merging of these configurations internally.

### Applying Environment to Dynamic Workflows

Similar to tasks, an `Environment` instance can also be used to configure `dynamic` workflows. Use the `dynamic` method of the `Environment` instance.

```python
from flytekit import dynamic
from flytekit.core.environment import Environment

# Environment for dynamic workflows, perhaps with higher timeouts
dynamic_workflow_env = Environment(timeout="30m", retries=1)

@dynamic_workflow_env.dynamic
def my_complex_dynamic_workflow(input_data: int) -> int:
    # This dynamic workflow will inherit the timeout and retries
    # ... call sub-tasks ...
    return input_data * 2
```

### Modifying Environment Configurations

The `Environment` class provides methods to manage its internal overrides:

*   **`update(**overrides: Any)`:** This method modifies the *current* `Environment` instance in place by merging the provided `overrides` with its existing ones. Use `update` when you intend to change the base configuration for subsequent uses of the *same* `Environment` object.

    ```python
    my_env = Environment(cpu="1", memory="2Gi")
    my_env.update(memory="4Gi", gpu="1") # 'my_env' now has cpu="1", memory="4Gi", gpu="1"
    ```

*   **`extend(**overrides: Any) -> "Environment"`:** This method creates and returns a *new* `Environment` instance. The new instance contains a merged set of overrides from the original `Environment` and the `overrides` provided to `extend`. This promotes immutability and is useful when you need a derived configuration without altering the original `Environment` object.

    ```python
    base_env = Environment(container_image="base_image:latest")
    # Create a new environment for GPU tasks, leaving 'base_env' unchanged
    gpu_env = base_env.extend(gpu="1", memory="8Gi")

    # base_env still only has container_image="base_image:latest"
    # gpu_env has container_image="base_image:latest", gpu="1", memory="8Gi"
    ```

    Prefer `extend` when creating variations of an environment to maintain the integrity of your base configurations.

### Inspecting Environment Overrides

To view the current overrides stored within an `Environment` instance, use the `show()` method. This prints a formatted representation of the overrides to the console.

```python
my_env = Environment(cpu="1", memory="2Gi", timeout="5m")
my_env.show()
```

This will output a panel containing the dictionary of overrides, useful for debugging and verification.

## Important Considerations

*   **`_task_function` Override Prevention:** The `Environment` class explicitly prevents overriding the `_task_function` key during initialization or updates. This key is reserved for internal use by the decorator mechanism.
*   **Override Merging Logic:** When applying additional overrides or using `update`/`extend`, the new overrides are merged with existing ones. If a key exists in both the `Environment` instance and the new overrides, the value from the new overrides takes precedence.
*   **Performance:** `Environment` objects are lightweight. The overhead of using them is minimal, primarily occurring during the definition and registration of tasks or dynamic workflows.
*   **Integration:** The `Environment` class seamlessly integrates with the `task` and `dynamic` decorators provided by Flytekit, offering a consistent and powerful way to manage configurations.

## Best Practices

*   **Centralize Common Configurations:** Define `Environment` instances for common resource profiles, container images, or retry policies in a dedicated module. This promotes reusability and consistency across your project.
*   **Use `extend` for Variations:** When creating specialized versions of an environment (e.g., a GPU-enabled version of a base environment), use `extend` to create a new instance. This keeps your base environments immutable and predictable.
*   **Descriptive Naming:** Name your `Environment` instances clearly to reflect their purpose or the type of configuration they encapsulate (e.g., `high_memory_env`, `gpu_training_env`, `short_timeout_env`).
<!--
key: summary_environment_configuration_6d24ea79-7e50-4b53-986e-b047955cc6ab
type: summary_end

-->
<!--
code_unit: flytekit.core.environment.Environment
code_unit_type: class
help_text: ''
key: example_2219d862-21a4-410a-a275-252ed6fe1007
type: example

-->