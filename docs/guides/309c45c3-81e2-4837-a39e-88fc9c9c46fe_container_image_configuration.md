
<!--
help_text: ''
key: summary_container_image_configuration_38415eaf-5d41-4991-8c18-4e59a62c621a
modules:
- flytekit.configuration.default_images
- flytekit.configuration.internal.Images
questions_to_answer: []
type: summary

-->
Container Image Configuration

The system determines the container image used for tasks and workflows through two primary mechanisms: user-defined custom images and an intelligent default image resolution process. This dual approach provides flexibility for specific project requirements while offering sensible defaults for common use cases.

### Specifying Custom Container Images

Developers can define custom container images within the configuration, allowing for specific image versions or entirely different images to be used. This is particularly useful when tasks require specialized dependencies not present in the default images.

Custom images are specified in the configuration file under an `[images]` section. Each entry in this section maps a friendly name to a fully qualified name (FQN) of the container image. The FQN can optionally include a tag.

**Configuration Example (INI format):**

```ini
[images]
my_custom_image=docker.io/myorg/my-custom-image:v1.0.0
another_image=myregistry.com/path/to/image
```

**Configuration Example (YAML format):**

```yaml
images:
  my_custom_image: docker.io/myorg/my-custom-image:v1.0.0
  another_image: myregistry.com/path/to/image
```

The `Images` class, specifically its `get_specified_images` static method, is responsible for parsing these configurations. It reads the `[images]` section from the provided `ConfigFile` object and returns a dictionary where keys are the friendly names and values are the corresponding image FQNs.

```python
from flytekit.configuration import ConfigFile
from flytekit.configuration.internal import Images

# Assuming cfg is an instance of ConfigFile loaded from your configuration
# For example: cfg = ConfigFile.auto()
# images_config = Images.get_specified_images(cfg)
# print(images_config)
# Output: {'my_custom_image': 'docker.io/myorg/my-custom-image:v1.0.0', 'another_image': 'myregistry.com/path/to/image'}
```

This mechanism allows tasks to reference images by their friendly names, abstracting the underlying FQN and simplifying configuration management across different environments.

### Default Container Image Resolution

When a custom image is not explicitly specified, the system automatically resolves a default container image. This process is handled by the `DefaultImages` class and follows a defined hierarchy to determine the most appropriate image. This ensures that tasks can run without explicit image configuration, leveraging pre-built and optimized images.

The resolution order for the default image is as follows:

1.  **Environment Variable Override**: The system first checks for the `FLYTE_INTERNAL_IMAGE_ENV_VAR` environment variable. If this variable is set, its value is used directly as the default image FQN, bypassing all other resolution steps. This provides a powerful way to globally override the default image for a specific environment or execution.

2.  **Plugin-Provided Default**: If no environment variable is set, the system queries any registered plugins for a default image. Plugins can provide their own default image, allowing for specialized environments or integrations to dictate the base image.

3.  **Python Version-Specific Defaults**: If neither an environment variable nor a plugin provides a default, the system constructs an image FQN based on the Python version of the current execution environment. The `DefaultImages` class maintains a mapping of `PythonVersion` (e.g., `PYTHON_3_8`, `PYTHON_3_9`, etc.) to base image prefixes (e.g., `cr.flyte.org/flyteorg/flytekit:py3.9-`). The `PythonVersion` enum defines the supported Python major and minor versions.

    The `find_image_for` method within `DefaultImages` automatically detects the current Python version if not explicitly provided.

4.  **Flytekit Version Suffix**: Finally, the determined base image prefix is suffixed with the current Flytekit version. The `get_version_suffix` method retrieves the installed Flytekit version. If the version is a development build (e.g., contains "dev"), "latest" is used as the suffix; otherwise, the exact version (e.g., "1.2.3") is appended.

This comprehensive resolution process ensures that a suitable default image is always available, tailored to the execution environment's Python version and the installed Flytekit version.

**Example of Default Image Resolution:**

If running with Python 3.9 and Flytekit version `1.2.3`, and no environment variable or plugin override is present, the resolved default image would be `cr.flyte.org/flyteorg/flytekit:py3.9-1.2.3`.

```python
from flytekit.configuration.default_images import DefaultImages, PythonVersion

# Get the default image for the current environment
# default_image_fqn = DefaultImages.default_image()
# print(default_image_fqn) # e.g., cr.flyte.org/flyteorg/flytekit:py3.9-1.2.3

# You can also explicitly request an image for a specific Python version
# image_for_py310 = DefaultImages.find_image_for(python_version=PythonVersion.PYTHON_3_10)
# print(image_for_py310) # e.g., cr.flyte.org/flyteorg/flytekit:py3.10-1.2.3
```

### Best Practices and Considerations

*   **Consistency**: For production deployments, it is a best practice to explicitly define custom images or use the `FLYTE_INTERNAL_IMAGE_ENV_VAR` to pin the exact image version. This ensures consistent execution environments across different stages (development, staging, production) and prevents unexpected behavior due to automatic version updates.
*   **Custom Dependencies**: When your tasks require specific Python packages or system libraries not included in the default images, define a custom image. Build this image with your required dependencies and reference it by its friendly name in your task definitions.
*   **Performance**: Using pre-pulled or cached images can significantly reduce task startup times. Ensure your execution environment has access to the specified images and that image pull policies are optimized.
*   **Security**: Always use trusted image registries and ensure that the images you use, whether custom or default, are regularly scanned for vulnerabilities.
*   **Debugging**: When debugging image-related issues, first check the `FLYTE_INTERNAL_IMAGE_ENV_VAR` and then inspect the configuration file for custom image definitions. Understanding the default image resolution logic helps in diagnosing why a particular image is being used.
<!--
key: summary_container_image_configuration_38415eaf-5d41-4991-8c18-4e59a62c621a
type: summary_end

-->
<!--
code_unit: flytekit.configuration.default_images.DefaultImages
code_unit_type: class
help_text: ''
key: example_3dc77452-443a-435d-ba44-4add4d821726
type: example

-->
<!--
code_unit: flytekit.configuration.internal.Images
code_unit_type: class
help_text: ''
key: example_dfcae62f-e4ad-401f-a00d-56c09c5bc45a
type: example

-->