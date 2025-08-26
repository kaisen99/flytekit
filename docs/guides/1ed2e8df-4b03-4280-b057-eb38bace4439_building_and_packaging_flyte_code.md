
<!--
help_text: ''
key: summary_building_and_packaging_flyte_code_f023b80e-c599-4f31-a38a-45bb8c149f6c
modules:
- flytekit.clis.sdk_in_container.build
- flytekit.clis.sdk_in_container.serialize
- flytekit.tools.fast_registration
questions_to_answer: []
type: summary

-->
Building and Packaging Flyte Code

Flyte code, comprising tasks and workflows, requires packaging into a container image for execution within a Flyte environment. This process transforms your Python source code and its dependencies into a deployable artifact.

### Standard Build Process

The standard build process packages your entire Flyte project's source code into a Docker image. This image includes all necessary Python files, allowing the Flyte engine to execute your tasks and workflows directly from the container.

To build an image for a specific workflow or task defined in a Python file, use the `pyflyte build` command. This command leverages the `BuildCommand` and `BuildWorkflowCommand` components to discover and prepare your code for containerization.

**Example:**

```bash
pyflyte build my_workflow_file.py my_workflow_name
```

This command generates a Docker image containing `my_workflow_file.py` and its dependencies, ready for registration and execution on a Flyte cluster.

### Fast Serialization

Fast serialization, also known as fast registration, offers an optimized approach to packaging Flyte code. Instead of embedding the entire source code into the Docker image, fast serialization extracts and registers the Flyte entities (tasks and workflows) directly with the Flyte backend. The resulting Docker image then only needs to contain the runtime dependencies required to execute the *serialized* entities, not the original source code.

This approach is particularly beneficial for:
*   **Faster Image Builds:** Reduces the amount of data copied into the image, leading to quicker build times.
*   **Smaller Image Sizes:** Images are leaner as they do not contain the full source code.
*   **Quicker Registration:** The serialized entities can be registered more rapidly.

To enable fast serialization, use the `--fast` flag with the `pyflyte build` command. This flag is managed by the `BuildParams` configuration, which sets the `SerializationMode` to `FAST`.

**Example:**

```bash
pyflyte build --fast my_workflow_file.py my_workflow_name
```

When using fast serialization, the generated image will not contain your original Python source code. This is an important consideration for debugging or introspection, as you cannot directly inspect the source code within the running container.

### Controlling Package Contents

When packaging Flyte code, especially in standard build mode, you might need to control which files are included or excluded from the final container image. The `FastPackageOptions` component provides mechanisms to configure this behavior. While primarily designed for fast registration scenarios, its principles apply to how source code is handled during packaging.

`FastPackageOptions` allows you to specify:
*   `ignores`: A list of patterns to exclude files or directories from being copied into the image. This is useful for development-specific files, large datasets, or sensitive information that should not be part of the deployed artifact.
*   `keep_default_ignores`: A boolean flag to determine whether default ignore patterns (e.g., `.git`, `__pycache__`) should still apply.
*   `copy_style`: Defines how files are copied, offering flexibility in handling symlinks or specific file types.
*   `show_files`: A debugging option to list the files that are being packaged.

These options are typically configured through project-level settings (e.g., `pyproject.toml` or similar configuration files) rather than direct CLI parameters, influencing the underlying packaging logic.

### Best Practices and Considerations

*   **When to use Fast Serialization:** Opt for fast serialization (`--fast`) in production environments where build speed, image size, and rapid deployment are critical. It's also beneficial for CI/CD pipelines.
*   **Development Workflow:** During active development and debugging, the standard build process (without `--fast`) might be more convenient, as it includes the source code in the image, allowing for easier inspection and debugging within the container.
*   **Dependency Management:** Regardless of the build mode, ensure your project's dependencies are correctly specified (e.g., in `requirements.txt`) so they are installed in the container image.
*   **Code Organization:** Maintain a clean and modular codebase. Use `FastPackageOptions` to explicitly exclude unnecessary files, reducing image bloat and potential security risks.
*   **Reproducibility:** Always pin your dependencies to specific versions to ensure consistent builds across different environments.
<!--
key: summary_building_and_packaging_flyte_code_f023b80e-c599-4f31-a38a-45bb8c149f6c
type: summary_end

-->
<!--
code_unit: flytekit.clis.sdk_in_container.build.BuildCommand
code_unit_type: class
help_text: ''
key: example_4fab25f2-d504-4520-a06b-d716b2456be3
type: example

-->
<!--
code_unit: flytekit.clis.sdk_in_container.build.BuildWorkflowCommand
code_unit_type: class
help_text: ''
key: example_361bb27c-0f96-44ca-a3ba-e6483c241d97
type: example

-->
<!--
code_unit: flytekit.clis.sdk_in_container.serialize.SerializationMode
code_unit_type: class
help_text: ''
key: example_4d7a0754-a805-480e-9ff4-2ef84300ee00
type: example

-->
<!--
code_unit: flytekit.tools.fast_registration.FastPackageOptions
code_unit_type: class
help_text: ''
key: example_d0ab3212-ae50-475d-9f0d-e9cf40d2735c
type: example

-->