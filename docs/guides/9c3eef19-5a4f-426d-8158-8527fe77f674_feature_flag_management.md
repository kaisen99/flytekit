
<!--
help_text: ''
key: summary_feature_flag_management_cf7f592e-835e-4788-8715-b65f647724d9
modules:
- flytekit.configuration.feature_flags
questions_to_answer: []
type: summary

-->
Feature Flag Management

Feature Flag Management in Flytekit primarily involves configuring internal settings that influence core behaviors, particularly how the system discovers and interprets Python package structures. These flags are not for enabling or disabling user-facing features but for fine-tuning Flytekit's operational characteristics related to code packaging and execution.

### Configuring Python Package Root Discovery

The `FeatureFlags` class provides settings to control how Flytekit identifies the root of your Python project when packaging code for execution. This is crucial for correctly resolving module imports during serialization and remote execution.

The `FLYTE_PYTHON_PACKAGE_ROOT` setting within the `FeatureFlags` class dictates how Flytekit determines the top-level directory of your Python package. This is essential for correctly resolving module imports when Flytekit serializes and executes your code.

#### `FLYTE_PYTHON_PACKAGE_ROOT` Setting

The `FLYTE_PYTHON_PACKAGE_ROOT` attribute accepts specific values to define the Python package root:

*   **`"auto"`**: Flytekit automatically attempts to locate the fully qualified module name. This assumes a standard Python package structure where every directory intended as a Python package contains an `__init__.py` file. This is the default behavior and is generally recommended for most standard projects.

    *   **Example:** If your project structure is `my_project/src/workflows/my_workflow.py`, and `src` is a Python package (i.e., `src/__init__.py` exists), Flytekit attempts to identify `src` as a potential root.

*   **`.`**: Designates the current working directory (where the Flytekit command is executed) as the Python package root. Use this when your project's root directly corresponds to your execution context.

    *   **Example:** If you execute `pyflyte build` from the `my_project/` directory, and your workflows are located in `my_project/workflows/`, setting `FLYTE_PYTHON_PACKAGE_ROOT=.` makes `my_project` the root for packaging.

*   **`"path/to/root"`**: Specifies an explicit absolute or relative path to the desired Python package root. This provides precise control and is useful for complex monorepos or non-standard project layouts.

    *   **Example:** For a project structure like `repo/python_src/my_project/workflows/`, you might set `FLYTE_PYTHON_PACKAGE_ROOT=python_src/my_project` (relative to `repo/`) or provide the full absolute path to `python_src/my_project`.

#### Usage and Best Practices

This setting is typically configured via environment variables or Flytekit's configuration files. For instance, you can set it as an environment variable:

```bash
export FLYTE_PYTHON_PACKAGE_ROOT="path/to/your/project/root"
```

*   **Default Behavior (`"auto"`):** For most standard Python projects, the `"auto"` setting works effectively by leveraging Python's module resolution rules. It simplifies setup by inferring the root, reducing the need for manual configuration.
*   **Explicit Control (`.` or `path`):** Use `.` when your build context is consistently the project root. Employ an explicit path when `"auto"` fails to correctly identify the root, or when you need to define a specific sub-directory within a larger repository as the package root. This is common in monorepos where multiple Flytekit projects might reside within a single repository.
*   **Troubleshooting:** Incorrectly setting this value can lead to `ModuleNotFoundError` during serialization or execution, as Flytekit will fail to correctly package your code or resolve internal imports. Ensure the specified root allows all your workflow and task modules to be imported correctly relative to it.

#### Limitations and Considerations

*   The `FLYTE_PYTHON_PACKAGE_ROOT` setting primarily affects how Flytekit *packages* your code for remote execution. It does not alter Python's standard `sys.path` or module import behavior during local development.
*   When using `"auto"`, ensure that directories intended as Python packages contain `__init__.py` files to facilitate correct discovery by Flytekit.
*   For complex monorepos, carefully consider the implications of the chosen root path on all included Flytekit projects. Consistency across projects within the same repository is crucial for reliable packaging.
<!--
key: summary_feature_flag_management_cf7f592e-835e-4788-8715-b65f647724d9
type: summary_end

-->
<!--
code_unit: flytekit.configuration.feature_flags.FeatureFlags
code_unit_type: class
help_text: ''
key: example_0f1927b6-e27e-4637-9aa4-e6f025abe20a
type: example

-->