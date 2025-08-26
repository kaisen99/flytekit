
<!--
help_text: ''
key: summary_building_docker_images_for_flyte_e5ea5e07-ee63-4e5b-b543-9b4474dad04e
modules:
- flytekit.clis.sdk_in_container.build
- flytekit.clis.sdk_in_container.serialize
questions_to_answer: []
type: summary

-->
# Building Docker Images for Flyte

Building Docker images for Flyte workflows and tasks is a crucial step for deploying and executing your code on a Flyte cluster. This process packages your Python code, its dependencies, and the Flytekit runtime into a self-contained, executable unit.

## Overview

The build process creates a Docker image that encapsulates your Flyte workflows and tasks. This image serves as the execution environment for your code when it runs on the Flyte platform. The build command provides a command-line interface (CLI) to simplify this process, allowing you to specify which workflow or task to package and how its source code is included.

## Core Concepts

### Serialization Modes

Flyte offers different serialization modes when building Docker images, primarily influencing how your source code is included in the final image. This choice impacts image size, build time, and deployment flexibility.

*   **Default Serialization**: This mode, represented by `SerializationMode.DEFAULT`, includes your entire Python source code within the Docker image. This ensures that the image is self-contained and has all necessary code at runtime. This is the default behavior when no specific serialization mode is chosen.
*   **Fast Serialization**: This mode, represented by `SerializationMode.FAST`, optimizes the image by *not* embedding the source code directly into the Docker image. Instead, the image contains only the necessary Flytekit runtime and dependencies. The source code is expected to be available to the Flyte cluster through other means, such as a Git repository or an S3 bucket, and fetched at runtime. This mode is enabled using the `--fast` flag.

## Building Images

To build a Docker image for a Flyte workflow or task, use the build command. This command operates on a specified Python file, allowing you to select a particular entity (workflow or task) within that file.

The `BuildCommand` acts as the primary entry point for the build operations. It dynamically discovers workflows and tasks within a given Python file and presents them as subcommands. The `BuildWorkflowCommand` then handles the specific build logic for the selected entity.

### Basic Usage

To build an image for a specific workflow or task, provide the Python file containing your Flyte code, followed by the name of the workflow or task.

For example, if you have a file named `my_workflow.py` with a workflow called `my_example_workflow`:

```bash
pyflyte build my_workflow.py my_example_workflow
```

This command builds a Docker image for `my_example_workflow` defined in `my_workflow.py` using the default serialization mode. The image will contain the source code of `my_workflow.py`.

### Using Fast Serialization

To build an image using fast serialization, append the `--fast` flag to your build command. This leverages the `BuildParams` configuration to enable `SerializationMode.FAST`.

```bash
pyflyte build --fast my_workflow.py my_example_workflow
```

When using `--fast`, the resulting Docker image will be smaller as it does not embed the source code. This is particularly useful in scenarios where:

*   You manage source code versions externally (e.g., Git, S3).
*   You want to reduce image size and build times.
*   Your deployment strategy involves fetching the source code at runtime.

**Important Consideration**: When using fast serialization, ensure that the Flyte cluster environment where the image will run has access to the source code at the specified location (e.g., a Git repository URL or an S3 path). If the source code is not accessible, the workflow or task execution will fail.

## Best Practices

*   **Choose Serialization Mode Wisely**: For initial development and simpler deployments, default serialization is often sufficient. For production environments, CI/CD pipelines, or when managing large codebases, fast serialization can offer significant advantages in terms of image size and build efficiency.
*   **Dependency Management**: Ensure all Python dependencies required by your workflow or task are specified in your project's `requirements.txt` or equivalent. The build process relies on these to create a functional Docker image.
*   **Image Tagging**: While not explicitly controlled by the `build` command, it is best practice to tag your Docker images with meaningful versions (e.g., Git commit hashes, semantic versions) to ensure reproducibility and traceability.
*   **Environment Consistency**: Strive for consistency between your local development environment and the Docker image's environment to minimize runtime issues.
<!--
key: summary_building_docker_images_for_flyte_e5ea5e07-ee63-4e5b-b543-9b4474dad04e
type: summary_end

-->
<!--
code_unit: pyflyte build
code_unit_type: class
help_text: ''
key: example_bafabca4-0734-4fa9-87ab-b65bd18f6868
type: example

-->
<!--
code_unit: pyflyte build --fast your_file.py your_workflow
code_unit_type: class
help_text: ''
key: example_0c02dbca-c15d-4f5c-a867-91910fe4c49d
type: example

-->