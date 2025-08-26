
<!--
help_text: ''
key: summary_default_image_management_fd095a43-8ee8-486c-bb76-b97ed031b5b9
modules:
- flytekit.configuration.default_images.PythonVersion
- flytekit.configuration.default_images.DefaultImages
questions_to_answer: []
type: summary

-->
## Default Image Management

Default Image Management provides a robust mechanism for determining the appropriate container image to use for Flytekit workflows. This system ensures that a sensible default image is available, automatically aligning with the current Python environment and Flytekit version, while also offering flexible override options.

### Image Resolution Logic

The system resolves the default image by following a specific lookup order:

1.  **Plugin Override:** The system first attempts to retrieve a default image from an active plugin. This allows for custom image resolution logic to be injected, providing the highest priority override.
2.  **Environment Variable Override:** If no image is provided by a plugin, the system checks for the `FLYTE_INTERNAL_IMAGE_ENV_VAR` environment variable. If this variable is set, its value is used as the default image, bypassing further automatic detection.
3.  **Automatic Detection:** If neither a plugin nor an environment variable specifies an image, the system automatically constructs an image string based on the current Python version and the installed Flytekit version.

### Retrieving Default Images

To retrieve the default image for the current execution environment, use the `default_image` method:

```python
from flytekit.core.constants import DefaultImages

current_default_image = DefaultImages.default_image()
print(f"Default image for current environment: {current_default_image}")
```

This method automatically detects the Python version of the running interpreter and appends the appropriate Flytekit version suffix.

To retrieve a default image for a specific Python version or Flytekit version, use the `find_image_for` method. This is useful when you need to determine an image for an environment different from the current one.

The `PythonVersion` enum defines the supported Python major.minor versions:

```python
from flytekit.core.constants import DefaultImages, PythonVersion

# Get image for Python 3.9
py39_image = DefaultImages.find_image_for(python_version=PythonVersion.PYTHON_3_9)
print(f"Default image for Python 3.9: {py39_image}")

# Get image for Python 3.11 with a specific Flytekit version
py311_specific_image = DefaultImages.find_image_for(
    python_version=PythonVersion.PYTHON_3_11,
    flytekit_version="1.10.0"
)
print(f"Default image for Python 3.11 (Flytekit 1.10.0): {py311_specific_image}")
```

When `flytekit_version` is not provided to `find_image_for`, the system automatically determines the suffix based on the installed Flytekit version. If the installed Flytekit version is a development build (e.g., contains "dev" in its string) or is not found, the suffix defaults to `latest`. Otherwise, the exact installed version is used.

### Configuration and Overrides

The system provides several ways to override the default image resolution:

*   **Environment Variable:** Setting the `FLYTE_INTERNAL_IMAGE_ENV_VAR` environment variable provides a high-priority, system-wide override. This is particularly useful for consistent image usage across different deployments or CI/CD pipelines.

    ```bash
    export FLYTE_INTERNAL_IMAGE_ENV_VAR="my-custom-registry/my-flytekit-image:1.2.3"
    ```

*   **Programmatic Override:** The `find_image_for` method allows explicit specification of the Python version and Flytekit version. This is useful when you need to generate image names for target environments that differ from the current execution environment.

*   **Plugin System:** For advanced scenarios, the plugin system allows developers to register custom logic for default image resolution. This provides the most flexible and highest-priority override mechanism, enabling integration with custom image registries or dynamic image selection based on complex criteria.

### Best Practices

*   **Rely on Automatic Detection:** For most development and testing scenarios, allow the system to automatically determine the default image. This ensures compatibility with the current Python environment and installed Flytekit version.
*   **Use Environment Variables for Deployment:** For production deployments or shared development environments, use the `FLYTE_INTERNAL_IMAGE_ENV_VAR` to enforce a specific, tested image. This ensures consistency and reproducibility across different execution contexts.
*   **Specify Python Versions Explicitly:** When building or testing workflows for different Python environments, explicitly pass the `PythonVersion` to `find_image_for` to ensure the correct base image prefix is used.
*   **Understand Version Suffixes:** Be aware that development versions of Flytekit will result in an image suffixed with `latest`. For stable deployments, ensure a released version of Flytekit is installed to get a version-specific image.
<!--
key: summary_default_image_management_fd095a43-8ee8-486c-bb76-b97ed031b5b9
type: summary_end

-->
<!--
code_unit: flytekit.configuration.default_images
code_unit_type: class
help_text: ''
key: example_a02ca28b-c81f-428a-a414-ba3274ff5b0e
type: example

-->