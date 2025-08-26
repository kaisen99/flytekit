
<!--
help_text: ''
key: summary_building_custom_docker_images_815f6bfc-5549-4767-b115-82f35ed99d68
modules:
- flytekit.clis.sdk_in_container.build
- flytekit.image_spec.image_spec
- flytekit.image_spec.default_builder
- flytekit.image_spec.noop_builder
- flytekit.clis.sdk_in_container.serialize
questions_to_answer: []
type: summary

-->
Building Custom Docker Images

Flytekit provides a robust mechanism for defining and building custom Docker images, ensuring reproducible and isolated execution environments for your tasks and workflows. This capability allows you to specify dependencies, source code, and runtime configurations directly within your Python code, streamlining the development and deployment process.

## Defining Image Specifications

The core of custom image building is the `ImageSpec` class. It offers a declarative way to define the characteristics of your Docker image.

An `ImageSpec` instance encapsulates various parameters that dictate how your image is constructed:

*   **`name`**: The name of your image (e.g., `my-custom-image`).
*   **`registry`**: An optional container registry where the image will be pushed (e.g., `ghcr.io/username`).
*   **`tag_format`**: A custom format string for the image tag, allowing for dynamic tagging based on the image specification's hash.
*   **`python_version`**: The Python version to use within the image. If `None`, the default Python version of the base image is used.
*   **`packages`**: A list of Python packages to install (e.g., `["pandas", "scikit-learn"]`).
*   **`requirements`**: A path to a `requirements.txt` file for Python dependencies.
*   **`conda_packages`**, **`conda_channels`**: Lists for specifying Conda packages and channels.
*   **`apt_packages`**: A list of APT packages to install.
*   **`base_image`**: The foundational image for your custom build. This can be a string (e.g., `"ubuntu:22.04"`) or another `ImageSpec` instance, allowing for layered image definitions.
*   **`builder`**: Specifies the image builder to use (e.g., `"default"`, `"noop"`). If `None`, the builder with the highest priority is automatically selected.
*   **`source_root`**: The root directory containing your source code. When set, Flytekit can automatically copy relevant files into the image.
*   **`source_copy_mode`**: Controls which source files are copied from `source_root`. Options include `CopyFileDetection.LOADED_MODULES` (default when `source_root` is set and fast registration is not used) to copy only loaded Python modules, `CopyFileDetection.ALL` to copy everything, or `CopyFileDetection.NO_COPY`.
*   **`copy`**: A list of specific files or directories to copy into the `/root` directory of the image.
*   **`env`**: A dictionary of environment variables to set within the image.
*   **`commands`**: A list of shell commands to execute during the image building process.
*   **`entrypoint`**: Overwrites the base image's entrypoint. Set to `[]` to remove it.
*   **`cuda`**, **`cudnn`**: Versions of CUDA and cuDNN to install for GPU support.
*   **`platform`**: Specifies the target platform for the build output (e.g., `linux/amd64`).
*   **`pip_index`**, **`pip_extra_index_url`**: Custom PyPI index URLs for pip.
*   **`pip_secret_mounts`**: (Experimental) A list of tuples `(secret_file_path, mount_path)` to mount secrets for pip installation.
*   **`pip_extra_args`**: Additional arguments for `pip install`.
*   **`runtime_packages`**: Packages to be installed at runtime. This requires `pip` to be present in your base image.
*   **`builder_options`**: A dictionary of builder-specific options.
*   **`builder_config`**: Custom builder images configuration (e.g., for `uv` or `micromamba`).

### Image Identification and State

The `ImageSpec` provides properties and methods to manage image identification and check its existence:

*   **`id`**: A unique hash derived from the `ImageSpec`'s content. This ID is used to identify the image in the serialization context and to check if the current container matches the spec at runtime.
*   **`tag`**: A hash-based tag for the image, incorporating changes in dependencies, source code (if `source_root` is used), and other parameters. This ensures that a new image is built when its definition changes.
*   **`image_name()`**: Returns the fully qualified image name, including the registry and tag.
*   **`is_container()`**: Checks if the currently running container was built from this specific `ImageSpec`. This is useful for conditional logic within your tasks.
*   **`exist()`**: Attempts to verify if the image already exists in the configured registry or locally. It returns `True` if found, `False` if not, or `None` if the check fails (e.g., due to permissions).

### Modifying Image Specifications

`ImageSpec` instances are immutable. To modify an `ImageSpec`, use its `with_` methods, which return a new `ImageSpec` instance with the updated attributes:

*   **`with_commands(commands)`**: Adds commands to be executed during the build.
*   **`with_packages(packages)`**: Adds Python packages.
*   **`with_apt_packages(apt_packages)`**: Adds APT packages.
*   **`with_copy(src)`**: Adds files/directories to copy.
*   **`force_push()`**: Returns a new `ImageSpec` that will force push the image, overwriting any existing image with the same tag.
*   **`with_runtime_packages(runtime_packages)`**: Adds packages to be installed at runtime.
*   **`with_builder_options(builder_options)`**: Adds builder-specific options.

### Pinning Environment Dependencies

The `ImageSpec.from_env()` class method allows you to create an `ImageSpec` that automatically infers the Python version and pins specified packages from your current environment. This is useful for ensuring consistency between your local development environment and the Docker image.

```python
from flytekit.image_spec import ImageSpec

# Create a basic image spec
my_image_spec = ImageSpec(
    name="my-flyte-app",
    python_version="3.9",
    packages=["pandas==1.5.3", "numpy"],
    apt_packages=["git"],
    commands=["RUN echo 'Hello from Dockerfile'"],
    registry="my-registry.com/my-org",
)

# Add more packages to an existing spec
my_image_spec_with_scipy = my_image_spec.with_packages("scipy")

# Define an image with a custom base image
custom_base_image_spec = ImageSpec(
    name="my-custom-base",
    python_version="3.9",
    packages=["tensorflow"],
)
my_app_image_spec = ImageSpec(
    name="my-app",
    base_image=custom_base_image_spec,
    packages=["keras"],
)

# Define an image that uses runtime packages
# Ensure 'pip' is in your base image's packages if using this feature
runtime_image_spec = ImageSpec(
    name="my-runtime-deps",
    base_image="python:3.9-slim",
    packages=["pip"],
).with_runtime_packages(["torch"])

# Create an ImageSpec from the current environment, pinning flytekit
env_image_spec = ImageSpec.from_env(pinned_packages=["flytekit"])

# Force push an image
force_push_image_spec = my_image_spec.force_push()
```

## Image Building Orchestration

The `ImageBuildEngine` class is responsible for managing and orchestrating the image building process. It maintains a registry of available `ImageSpecBuilder` implementations and selects the appropriate builder based on the `ImageSpec`.

*   **`register(builder_type, image_spec_builder, priority)`**: Registers a new image builder with a given type and priority.
*   **`build(image_spec)`**: The primary method to trigger an image build. This method orchestrates the build by:
    *   Recursively building any `ImageSpec` defined as a `base_image`.
    *   Selecting the appropriate `ImageSpecBuilder` based on the `builder` attribute of the `ImageSpec`.
    *   Calling the selected builder's `build_image` method.
    *   **Important**: Image building is skipped if the current execution mode is not a local workflow execution. This prevents unnecessary builds during actual Flyte task or workflow runs.

## Available Image Builders

Flytekit provides several built-in image builders, each serving a specific purpose. All builders implement the `ImageSpecBuilder` abstract base class, which defines the `build_image` and `should_build` methods.

The `should_build` method determines whether a build is necessary. It checks if the image already exists in the registry and respects the `_is_force_push` flag on the `ImageSpec`.

### Default Image Builder

The `DefaultImageBuilder` (`builder_type = "default"`) is the standard builder that leverages Docker and BuildKit to construct images.

*   **`build_image(image_spec)`**: This method creates a temporary Docker context (including a Dockerfile generated from the `ImageSpec`) and executes a `docker build` command. If a `registry` is specified in the `ImageSpec` and the `FLYTE_PUSH_IMAGE_SPEC` environment variable is set to `True` (default), the image is also pushed to the registry.
*   **Requirements**: A running Docker daemon is required for this builder to function.
*   **Limitations**: The `DefaultImageBuilder` supports a specific set of `ImageSpec` parameters. If unsupported parameters are provided, a warning will be issued, and they will be ignored.

### No-Op Image Builder

The `NoOpBuilder` (`builder_type = "noop"`) is a special builder that does not perform any actual image building.

*   **`build_image(image_spec)`**: This method simply returns the `base_image` string directly.
*   **Use Case**: This builder is useful when you already have a pre-existing Docker image that you want to use as your execution environment, and you do not need Flytekit to build or manage it. It assumes the `base_image` is a string representing a fully qualified image name.
*   **Limitation**: The `base_image` attribute of the `ImageSpec` must be a string, not another `ImageSpec`, when using the `noop` builder.

## Command-Line Interface (CLI)

Flytekit provides CLI commands to trigger image builds directly from your local development environment.

The `flytekit build` command group (`BuildCommand`) allows you to build images for your workflows and tasks.

*   **`flytekit build <filename>`**: Builds images for all workflows and tasks defined within the specified Python file.
*   **`flytekit build <filename> <entity_name>`**: Builds an image specifically for a named workflow or task within the file.

### Fast Serialization

The `BuildParams` class introduces the `--fast` flag for the `build` command:

*   **`--fast`**: When this flag is enabled, Flytekit uses "fast serialization." In this mode, the source code of your workflows and tasks is *not* copied into the Docker image. Instead, it's expected to be provided separately, typically via Flyte's fast registration mechanism (e.g., as a tarball uploaded to a blob store). This significantly reduces image size and build times, especially for large codebases.

```bash
# Build images for all workflows/tasks in 'my_project/workflows.py'
flytekit build my_project/workflows.py

# Build an image specifically for the 'my_workflow' workflow in 'my_project/workflows.py'
flytekit build my_project/workflows.py my_workflow

# Build images using fast serialization (source code not included in image)
flytekit build --fast my_project/workflows.py
```

## Practical Implementation and Best Practices

*   **Start Simple**: For most projects, begin with a basic `ImageSpec` defining your Python version and primary packages.
    ```python
    from flytekit.image_spec import ImageSpec

    my_task_image = ImageSpec(
        name="my-data-processing-image",
        python_version="3.10",
        packages=["pandas", "scikit-learn", "matplotlib"],
        registry="your-docker-registry.com/your-org",
    )
    ```
*   **Include Local Source Code**: If your tasks depend on local utility modules or files, use `source_root` and `source_copy_mode`.
    ```python
    from flytekit.image_spec import ImageSpec, CopyFileDetection

    # Assuming your project root is the current directory
    my_app_image = ImageSpec(
        name="my-ml-app",
        source_root=".",
        source_copy_mode=CopyFileDetection.LOADED_MODULES, # Copies only imported Python files
        packages=["torch"],
    )
    ```
*   **Layer Images for Efficiency**: Define a base image with common dependencies and then build application-specific images on top of it. This can speed up builds and reduce image sizes.
    ```python
    from flytekit.image_spec import ImageSpec

    # Define a common base image for all ML projects
    ml_base_image = ImageSpec(
        name="flyte-ml-base",
        python_version="3.9",
        packages=["numpy", "scipy", "pandas"],
        registry="your-registry.com/common",
    )

    # Define an application-specific image using the base
    my_model_image = ImageSpec(
        name="my-fraud-detection-model",
        base_image=ml_base_image,
        packages=["xgboost", "lightgbm"],
        source_root="./src", # Your model code
    )
    ```
*   **Use `from_env` for Development Consistency**: Leverage `ImageSpec.from_env()` to ensure that the packages installed in your Docker image match the versions in your local development environment.
    ```python
    from flytekit.image_spec import ImageSpec

    # Create an image spec that pins flytekit and other key packages
    # to the versions currently installed in your environment.
    dev_image_spec = ImageSpec.from_env(
        name="my-dev-image",
        pinned_packages=["flytekit", "pandas"],
        registry="your-dev-registry.com/your-username",
    )
    ```
*   **Consider Runtime Packages for Large Dependencies**: For very large or infrequently used dependencies, `runtime_packages` can be beneficial. This keeps the initial image smaller, and the package is installed only when the task runs. Ensure `pip` is available in your base image.
*   **Force Pushing for Development**: During rapid iteration, `force_push()` can be useful to ensure your latest image is always deployed, even if a cached version exists. Avoid this in production unless absolutely necessary.

## Important Considerations

*   **Docker Daemon Requirement**: The `DefaultImageBuilder` relies on a running Docker daemon. Ensure Docker is installed and running on the machine where you execute `flytekit build`.
*   **`--fast` Serialization Implications**: When using the `--fast` flag, remember that your source code is *not* bundled into the Docker image. Your deployment strategy must account for providing the code to the execution environment (e.g., via Flyte's fast registration).
*   **ImageSpec Hashing and Immutability**: The `id` and `tag` of an `ImageSpec` are derived from its content. Any change to the `ImageSpec` (e.g., adding a package, changing a command) will result in a new hash and thus a new image tag. This promotes immutability and ensures that changes to your environment definition lead to new, distinct images.
*   **`envd` Builder Compatibility**: If you configure `envd` as your builder, ensure your `envd` version is `0.3.39` or higher for compatibility with Flytekit versions greater than `v1.10.2`. Older `envd` versions may cause permission issues due to `WorkDir` overwrites.
*   **Experimental Features**: Features like `pip_secret_mounts` are experimental and their interface may change in future releases.
*   **Unsupported Parameters**: Be aware that the `DefaultImageBuilder` may not support all parameters available in `ImageSpec`. If you use unsupported parameters, the builder will issue a warning and ignore them.
<!--
key: summary_building_custom_docker_images_815f6bfc-5549-4767-b115-82f35ed99d68
type: summary_end

-->
<!--
code_unit: flytekit.clis.sdk_in_container.build.BuildCommand
code_unit_type: class
help_text: ''
key: example_fe1c1ceb-d917-49e0-98d8-92963ea25b60
type: example

-->
<!--
code_unit: flytekit.image_spec.image_spec.ImageSpec
code_unit_type: class
help_text: ''
key: example_3ade2893-55cb-4a6c-a38a-3a3f7d64ff71
type: example

-->
<!--
code_unit: flytekit.image_spec.default_builder.DefaultImageBuilder
code_unit_type: class
help_text: ''
key: example_d4016871-4a01-4a75-ab48-613ff9cdbe4c
type: example

-->