
<!--
help_text: ''
key: summary_registering_flyte_entities_148e5fda-31e3-4b2a-a9b3-38268a749f7b
modules:
- flytekit.remote.remote
- flytekit.remote.remote_fs
questions_to_answer: []
type: summary

-->
# Registering Flyte Entities

Flyte entities, such as tasks, workflows, and launch plans, are defined in Python code. To make these entities executable on a Flyte cluster, their definitions must be registered with the Flyte Admin service. This process involves serializing the Python objects into a format understood by Flyte Admin, packaging any necessary code, uploading artifacts to a data plane, and then making an API call to the Admin service.

The `FlyteRemote` object provides the primary interface for interacting with a remote Flyte backend, including methods for entity registration.

## Initializing the `FlyteRemote` Object

Before registering entities, initialize a `FlyteRemote` object. This object manages the connection to your Flyte cluster and provides access to various remote operations.

You can initialize `FlyteRemote` in several ways:

*   **`FlyteRemote.auto()`**: Automatically detects configuration from environment variables or a `config.yaml` file. This is often the most convenient method for local development.
*   **`FlyteRemote.for_endpoint()`**: Explicitly specifies the Flyte Admin endpoint.
*   **`FlyteRemote.for_sandbox()`**: Configures the remote object for a local Flyte sandbox environment.

When initializing, you can specify `default_project` and `default_domain` to avoid repeatedly providing them for each operation.

```python
from flytekit.remote import FlyteRemote

# Initialize FlyteRemote using auto-detection
remote = FlyteRemote.auto(
    default_project="flytesnacks",
    default_domain="development",
    data_upload_location="s3://my-s3-bucket/data" # Override for non-sandbox environments
)

# Or for a specific endpoint
# remote = FlyteRemote.for_endpoint(
#     endpoint="flyte.mycompany.com:80",
#     insecure=False,
#     default_project="my_project",
#     default_domain="production"
# )
```

## Core Registration Methods

The `FlyteRemote` object offers dedicated methods for registering different Flyte entity types. These methods handle the underlying serialization, versioning, and API interactions.

### Registering Tasks

Use the `register_task` method to register a Python task (a function decorated with `@task` or an instance of `PythonTask`).

```python
from flytekit import task
from flytekit.remote import FlyteRemote

remote = FlyteRemote.auto(default_project="flytesnacks", default_domain="development")

@task
def my_addition_task(a: int, b: int) -> int:
    """A simple task that adds two numbers."""
    return a + b

# Register the task
registered_task = remote.register_task(my_addition_task, version="v1.0.0")
print(f"Registered task: {registered_task.id.name} with version {registered_task.id.version}")

# If the task with the same project, domain, name, and version already exists,
# registration will be skipped without error.
```

### Registering Workflows

The `register_workflow` method registers a Python workflow (a function decorated with `@workflow` or an instance of `WorkflowBase`). By default, it also creates a default launch plan for the registered workflow.

```python
from flytekit import task, workflow
from flytekit.remote import FlyteRemote

remote = FlyteRemote.auto(default_project="flytesnacks", default_domain="development")

@task
def my_subtraction_task(a: int, b: int) -> int:
    """A simple task that subtracts two numbers."""
    return a - b

@workflow
def my_calculation_workflow(x: int, y: int) -> int:
    """A workflow that uses the subtraction task."""
    return my_subtraction_task(a=x, b=y)

# Register the workflow and its default launch plan
registered_workflow = remote.register_workflow(my_calculation_workflow, version="v1.0.0")
print(f"Registered workflow: {registered_workflow.id.name} with version {registered_workflow.id.version}")

# You can choose not to create a default launch plan
# remote.register_workflow(my_calculation_workflow, version="v1.0.1", default_launch_plan=False)
```

### Registering Launch Plans

Use `register_launch_plan` to register a `LaunchPlan` object. A launch plan is an executable version of a workflow, often with fixed inputs or specific execution options.

When registering a launch plan, the system first checks if its associated workflow is already registered. If not, it automatically registers the workflow and any of its dependencies before registering the launch plan itself.

```python
from flytekit import LaunchPlan
from flytekit.remote import FlyteRemote

# Assuming my_calculation_workflow and my_subtraction_task are defined as above
remote = FlyteRemote.auto(default_project="flytesnacks", default_domain="development")

# Create a launch plan for my_calculation_workflow with a default input
lp_with_fixed_input = LaunchPlan.create(
    "my_fixed_input_lp",
    my_calculation_workflow,
    default_inputs={"y": 5}
)

# Register the launch plan. If my_calculation_workflow isn't registered, it will be.
registered_lp = remote.register_launch_plan(lp_with_fixed_input, version="v1.0.0")
print(f"Registered launch plan: {registered_lp.id.name} with version {registered_lp.id.version}")

# You can also activate a launch plan after registration
remote.activate_launchplan(registered_lp.id)
print(f"Activated launch plan: {registered_lp.id.name}")
```

## Script Mode Registration (Fast Serialization)

For larger codebases or when not operating in an interactive environment (like a Jupyter notebook), `register_script` (or `fast_register_workflow` for workflows) offers a more efficient way to register entities. Instead of pickling individual Python objects, this method packages your entire project's source code into a zip file and uploads it to the Flyte data plane. This zip file is then referenced by the registered entities, allowing the Flyte engine to reconstruct the execution environment.

This approach is generally recommended for production deployments due to its efficiency and robustness.

```python
import os
import pathlib
from flytekit import task, workflow
from flytekit.remote import FlyteRemote
from flytekit.configuration import FastPackageOptions, CopyFileDetection

# Assume your project structure is:
# my_project/
# ├── __init__.py
# └── workflows/
#     ├── __init__.py
#     └── my_example.py
#         @task
#         def my_script_task(x: int) -> int: ...
#         @workflow
#         def my_script_workflow(x: int) -> int: ...

# Create dummy files for demonstration if they don't exist
project_root = "./my_project_for_script_mode"
os.makedirs(os.path.join(project_root, "workflows"), exist_ok=True)
with open(os.path.join(project_root, "__init__.py"), "w") as f: f.write("")
with open(os.path.join(project_root, "workflows", "__init__.py"), "w") as f: f.write("")
with open(os.path.join(project_root, "workflows", "my_example.py"), "w") as f:
    f.write("""
from flytekit import task, workflow

@task
def my_script_task(x: int) -> int:
    return x * 2

@workflow
def my_script_workflow(x: int) -> int:
    return my_script_task(x=x)
""")

# Import the workflow from the dummy file
import sys
sys.path.insert(0, project_root)
from workflows.my_example import my_script_workflow

remote = FlyteRemote.auto(default_project="flytesnacks", default_domain="development")

# Register the workflow using script mode
# source_path: The root directory of your project
# module_name: The Python module path to your workflow/task
registered_wf_script = remote.register_script(
    my_script_workflow,
    source_path=project_root,
    module_name="workflows.my_example",
    version="script-v1.0.0",
    fast_package_options=FastPackageOptions(copy_style=CopyFileDetection.ALL) # Copy all relevant files
)
print(f"Registered workflow via script mode: {registered_wf_script.id.name} version {registered_wf_script.id.version}")

# Clean up dummy files
import shutil
shutil.rmtree(project_root)
```

The `fast_package_options` parameter in `register_script` allows fine-grained control over which files are included in the uploaded package. `CopyFileDetection.ALL` ensures all relevant Python files are copied.

## Versioning and Identifiers

Every registered Flyte entity is uniquely identified by its `(project, domain, name, version)`.

*   **Explicit Versioning**: You can provide a `version` string directly to the registration methods. This is recommended for production environments to ensure reproducibility and control over deployments.
*   **Auto-generated Versioning**: If no `version` is provided, especially when `interactive_mode_enabled` is `True` for the `FlyteRemote` object, Flytekit automatically generates a version. This version is a hash derived from the entity's serialized content, `SerializationSettings`, Flytekit version, and any additional context (like image names or default inputs). This ensures that identical code registered under the same project/domain will have the same version.

The `_resolve_version` and `_version_from_hash` internal methods handle this auto-generation logic. The `_resolve_identifier` method ensures that all components of the `Identifier` (project, domain, name, version) are present before registration.

## Serialization Settings

`SerializationSettings` is a crucial object that dictates how entities are serialized and registered. It encapsulates configuration such as:

*   `image_config`: Specifies the Docker image to be used for tasks and workflows. `ImageConfig.auto_default_image()` is a convenient way to use the default image configured for your environment.
*   `project`, `domain`, `version`: Overrides the default project, domain, or version for the entities being serialized.
*   `source_root`: The root directory of your project, used by script mode for packaging.
*   `env`: Environment variables to be set during serialization.
*   `fast_serialization_settings`: Specific settings for script mode, including the `distribution_location` (where the packaged code is uploaded) and `destination_dir` (where it's extracted on the remote cluster).

When you call `register_task`, `register_workflow`, or `register_launch_plan`, if `serialization_settings` are not explicitly provided, a default object is created using the `FlyteRemote` object's `default_project`, `default_domain`, and an auto-detected image.

## File System Integration and Data Handling

Flytekit leverages `fsspec` internally to handle file system operations, particularly for uploading packaged code in script mode or large inputs/outputs.

*   **`FlyteFS`**: This custom `fsspec` filesystem handles `flyte://` URIs. It extends `HTTPFileSystem` and integrates with the `FlyteRemote` client to manage data uploads and downloads to the Flyte data plane (e.g., S3, GCS, Azure Blob Storage).
*   **`HttpFileWriter`**: An internal component used by `FlyteFS` for writing data to remote locations via HTTP PUT requests, typically to pre-signed URLs provided by Flyte Admin.
*   **`FlytePathResolver`**: Manages a mapping between local `flyte://` URIs and their corresponding remote storage paths (e.g., `s3://bucket/path`). This is primarily for internal tracking during interactive sessions.

When you use `register_script` or when interactive mode is enabled, methods like `upload_file` and `fast_package` are invoked. `upload_file` handles the hashing and uploading of local files (like the packaged source code) to the configured data plane.

## Important Considerations

*   **Interactive Mode**: When `interactive_mode_enabled` is `True` (often detected automatically in Jupyter notebooks), `FlyteRemote` attempts to pickle your Python entities and upload them directly. This is convenient for rapid iteration but can be less robust for complex dependencies or large codebases. For production, `register_script` is preferred.
*   **`RegistrationSkipped` Exception**: The `RegistrationSkipped` exception is raised internally when attempting to register an entity that is not registrable (e.g., a `ReferenceSpec` or a `RemoteEntity` that is explicitly marked as not registrable). This is usually handled gracefully by the public `register_*` methods, resulting in a debug log message rather than an error.
*   **Idempotency**: Registration methods are designed to be idempotent. If you attempt to register an entity with the exact same `(project, domain, name, version)` as an already existing entity, the operation will typically succeed without re-uploading or re-registering, and a debug message will be logged.
*   **Dependencies**: Ensure that all Python dependencies required by your tasks and workflows are included in the Docker image specified in your `ImageConfig` or available in the environment where the tasks will run.
*   **`execute` Method and Registration**: The `execute` method on `FlyteRemote` can implicitly trigger registration for local Python entities. If you call `remote.execute(my_local_workflow, ...)` and `my_local_workflow` is not yet registered on the cluster, `execute` will first attempt to register it (using default `SerializationSettings` if not provided) before initiating the execution. This provides a seamless development experience.
*   **`raw_register`**: This is an internal method primarily used by the public `register_*` methods. It takes a `FlyteControlPlaneEntity` (the serialized protobuf representation) and directly calls the Admin API. Most users should use the higher-level `register_task`, `register_workflow`, or `register_launch_plan` methods.
<!--
key: summary_registering_flyte_entities_148e5fda-31e3-4b2a-a9b3-38268a749f7b
type: summary_end

-->