
<!--
help_text: ''
key: summary_envd_image_builder_424d2a35-e423-40fc-b911-f0097481a4e5
modules:
- flytekitplugins.envd.image_builder
questions_to_answer: []
type: summary

-->
The Envd Image Builder facilitates building Docker images using `envd`. It abstracts the complexities of `envd` commands, providing a programmatic interface for creating custom build environments. This builder is particularly useful for scenarios requiring specific dependencies or configurations for Flyte tasks.

### Core Component: EnvdImageSpecBuilder

The `EnvdImageSpecBuilder` class is the primary interface for initiating image builds. It extends the base `ImageSpecBuilder` and implements `envd`-specific logic.

The `build_image` method is central to the builder's functionality. It accepts an `ImageSpec` object, which encapsulates all necessary details for the image to be built.

```python
class EnvdImageSpecBuilder(ImageSpecBuilder):
    """
    This class is used to build a docker image using envd.
    """

    def build_image(self, image_spec: ImageSpec):
        # ... implementation details ...
```

### Image Specification (`ImageSpec`)

The `ImageSpec` object defines the desired characteristics of the Docker image. This includes attributes such as the image's name (accessed via `image_spec.image_name()`), the target `platform` (e.g., `linux/amd64`, `linux/arm64`), and `registry_config` for authentication details. The builder translates this specification into an `envd` configuration file, which dictates the build environment, dependencies, and other `envd` settings.

### Image Building Process

The `build_image` method orchestrates the following steps:

1.  **Configuration Generation**: An `envd` configuration file is generated based on the provided `ImageSpec`. This file serves as the blueprint for `envd` to construct the image.
2.  **Registry Authentication**: If the `ImageSpec` includes `registry_config`, the builder executes `envd bootstrap --registry-config <config_path>` to configure authentication with the specified image registry. This step ensures that `envd` can pull base images or push the final image to private registries.
3.  **Build Execution**: The `envd build` command is invoked.
    *   The command targets the directory containing the generated `envd` configuration file.
    *   It supports multi-platform builds by utilizing the `platform` attribute from the `ImageSpec`.
    *   **Image Pushing**: The builder conditionally pushes the image to a remote registry. If `image_spec.registry` is specified and the `FLYTE_PUSH_IMAGE_SPEC` environment variable is set to "true" (case-insensitive, default behavior), the image is built and pushed directly to the registry. Otherwise, the image is only tagged locally without being pushed.
4.  **Context Management**: The builder handles `envd` context switching, which can be relevant when interacting with different registries or build environments.

### Error Handling

The `build_image` method incorporates robust error handling. If the `envd build` command fails for any reason, the builder prints a detailed, formatted representation of the `ImageSpec` that caused the failure. This output aids in debugging by providing a clear view of the input specification. After printing the `ImageSpec`, the original exception is re-raised, allowing upstream callers to handle the build failure appropriately.

### Usage Considerations

*   **`envd` Installation**: Ensure `envd` is installed and properly configured in the environment where the `EnvdImageSpecBuilder` operates. The builder relies on the `envd` CLI being accessible.
*   **`FLYTE_PUSH_IMAGE_SPEC` Environment Variable**: This environment variable controls whether the built image is automatically pushed to a remote registry. By default, it is set to "True". Setting it to "False" or "0" prevents automatic pushing, even if a registry is specified in the `ImageSpec`. This can be useful for local development or testing scenarios where only a local image is required.
*   **`ImageSpec` Accuracy**: The success of the image build heavily depends on a correctly formed `ImageSpec`. Ensure that the image name, platform, and any required registry credentials are accurately provided.
<!--
key: summary_envd_image_builder_424d2a35-e423-40fc-b911-f0097481a4e5
type: summary_end

-->
<!--
code_unit: flytekitplugins.envd.examples.envd_image_spec
code_unit_type: class
help_text: ''
key: example_ee2c6a58-b756-4ecf-9935-7156781441ac
type: example

-->