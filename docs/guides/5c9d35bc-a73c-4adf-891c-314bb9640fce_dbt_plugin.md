
<!--
help_text: ''
key: summary_dbt_plugin_ec30e8fb-60bf-46c6-b53b-6bf10626c0eb
modules:
- flytekitplugins.dbt.error
- flytekitplugins.dbt.schema
- flytekitplugins.dbt.task
questions_to_answer: []
type: summary

-->
# DBT Plugin

The DBT Plugin integrates dbt (data build tool) operations directly into Flyte workflows, allowing you to define, execute, and monitor your data transformations as part of your broader data pipelines. This plugin provides dedicated tasks that wrap common dbt CLI commands, enabling seamless execution of dbt runs, tests, and freshness checks within a Flyte environment.

## Core Capabilities

The plugin offers three primary tasks, each corresponding to a fundamental dbt CLI command:

### Running dbt Models

The `DBTRun` task executes the `dbt run` command. This is used to compile and run your dbt models against your data warehouse.

*   **Input:** The `DBTRunInput` object configures the `dbt run` command. It extends `BaseDBTInput`, providing common parameters like `project_dir`, `profiles_dir`, and `profile`. Additionally, it supports `select` and `exclude` lists to specify which models to run or exclude.
*   **Output:** The `DBTRunOutput` object provides the results of the `dbt run` command. It includes the `command` that was executed, the `exit_code` from the dbt CLI, and the raw contents of `run_results.json` and `manifest.json` as `raw_run_result` and `raw_manifest` respectively.

**Example:**

```python
from flytekit import workflow
from flytekitplugins.dbt.schema import DBTRunInput, DBTRunOutput
from flytekitplugins.dbt.task import DBTRun

dbt_run_task = DBTRun(name="my-dbt-run-task")

@workflow
def dbt_run_workflow() -> DBTRunOutput:
    return dbt_run_task(
        input=DBTRunInput(
            project_dir="tests/jaffle_shop",
            profiles_dir="tests/jaffle_shop/profiles",
            profile="jaffle_shop",
            select=["my_model"], # Optional: run only specific models
            flags={"full-refresh": True}, # Optional: pass custom CLI flags
        )
    )
```

### Testing dbt Models

The `DBTTest` task executes the `dbt test` command. This is used to run tests defined in your dbt project to ensure data quality and integrity.

*   **Input:** The `DBTTestInput` object configures the `dbt test` command. It extends `BaseDBTInput` and supports `select` and `exclude` lists to filter which tests to execute.
*   **Output:** The `DBTTestOutput` object provides the results of the `dbt test` command. Similar to `DBTRunOutput`, it includes the `command`, `exit_code`, `raw_run_result` (from `run_results.json`), and `raw_manifest` (from `manifest.json`).

**Example:**

```python
from flytekit import workflow
from flytekitplugins.dbt.schema import DBTTestInput, DBTTestOutput
from flytekitplugins.dbt.task import DBTTest

dbt_test_task = DBTTest(name="my-dbt-test-task")

@workflow
def dbt_test_workflow() -> DBTTestOutput:
    # Run all tests
    all_tests_output = dbt_test_task(
        input=DBTTestInput(
            project_dir="tests/jaffle_shop",
            profiles_dir="tests/jaffle_shop/profiles",
            profile="jaffle_shop",
        )
    )

    # Run only singular tests
    singular_tests_output = dbt_test_task(
        input=DBTTestInput(
            project_dir="tests/jaffle_shop",
            profiles_dir="tests/jaffle_shop/profiles",
            profile="jaffle_shop",
            select=["test_type:singular"],
        )
    )
    return singular_tests_output
```

### Checking dbt Source Freshness

The `DBTFreshness` task executes the `dbt source freshness` command. This command checks the freshness of your declared dbt sources, ensuring that your upstream data is up-to-date.

*   **Input:** The `DBTFreshnessInput` object configures the `dbt source freshness` command. It extends `BaseDBTInput` and supports `select` and `exclude` lists to filter which sources to check.
*   **Output:** The `DBTFreshnessOutput` object provides the results of the `dbt source freshness` command. It includes the `command`, `exit_code`, and the raw contents of `sources.json` as `raw_sources`.

**Example:**

```python
from flytekit import workflow
from flytekitplugins.dbt.schema import DBTFreshnessInput, DBTFreshnessOutput
from flytekitplugins.dbt.task import DBTFreshness

dbt_freshness_task = DBTFreshness(name="my-dbt-freshness-task")

@workflow
def dbt_freshness_workflow() -> DBTFreshnessOutput:
    return dbt_freshness_task(
        input=DBTFreshnessInput(
            project_dir="tests/jaffle_shop",
            profiles_dir="tests/jaffle_shop/profiles",
            profile="jaffle_shop",
        )
    )
```

## Input and Output Structures

The plugin defines a clear structure for passing arguments to dbt commands and receiving their results.

### Base Input (`BaseDBTInput`)

All dbt task inputs (`DBTRunInput`, `DBTTestInput`, `DBTFreshnessInput`) inherit from `BaseDBTInput`. This class defines common parameters for interacting with a dbt project:

*   `project_dir` (str): The path to the directory containing your `dbt_project.yml` file. This is crucial for dbt to locate your project.
*   `profiles_dir` (str): The path to the directory containing your `profiles.yml` file. This specifies how dbt connects to your data warehouse.
*   `profile` (str): The name of the profile to use from your `profiles.yml`. This overrides any profile specified in `dbt_project.yml`.
*   `target` (str, optional): The target to load for the given profile.
*   `output_path` (str, default="target"): The path within the `project_dir` where dbt writes compiled files and run artifacts (e.g., `run_results.json`, `manifest.json`).
*   `ignore_handled_error` (bool, default=False): If `True`, the task will not raise a `DBTHandledError` when dbt returns an exit code of `1`. This is useful if you want to treat certain dbt failures (like test failures) as warnings rather than workflow failures.
*   `flags` (dict, optional): A dictionary of additional CLI flags to pass to the dbt command. Keys are flag names (e.g., `"full-refresh"`, `"vars"`), and values are their corresponding arguments. Boolean flags should have `True` as their value.

The `to_args()` method converts these input parameters into a list of command-line arguments for the dbt CLI.

### Base Output (`BaseDBTOutput`)

All dbt task outputs (`DBTRunOutput`, `DBTTestOutput`, `DBTFreshnessOutput`) inherit from `BaseDBTOutput`. This class provides common information about the executed dbt command:

*   `command` (str): The complete dbt CLI command that was executed, including all flags.
*   `exit_code` (int): The exit code returned by the dbt CLI process.

Specific output classes add attributes to capture dbt's generated JSON artifacts:

*   `DBTRunOutput` and `DBTTestOutput` include `raw_run_result` and `raw_manifest`.
*   `DBTFreshnessOutput` includes `raw_sources`.

## Error Handling

The DBT Plugin provides specific exceptions to differentiate between types of dbt command failures:

*   `DBTHandledError`: Raised when the dbt CLI command returns an exit code of `1`. According to dbt's exit codes, this typically indicates a "handled error," such as a test failure or a model compilation warning that dbt explicitly reports. By default, this error will cause the Flyte task to fail. You can suppress this behavior by setting `ignore_handled_error=True` in the task's input.
*   `DBTUnhandledError`: Raised when the dbt CLI command returns an exit code of `2`. This signifies an "unhandled error," indicating a more severe issue like a syntax error, an invalid configuration, or an unexpected internal dbt error. This error always causes the Flyte task to fail.

Both error classes include the `message` and `logs` from the dbt command execution, aiding in debugging.

## Best Practices and Considerations

*   **DBT Project and Profiles:** Ensure your dbt project (`dbt_project.yml`) and profiles (`profiles.yml`) are correctly configured and accessible within the task's execution environment. The `project_dir` and `profiles_dir` inputs must point to these locations.
*   **Execution Environment:** The Flyte task's container image must have `dbt` installed and any necessary dbt adapters (e.g., `dbt-bigquery`, `dbt-snowflake`) for your data warehouse.
*   **Artifacts:** The plugin reads dbt's output JSON files (`run_results.json`, `manifest.json`, `sources.json`) from the `output_path` within the `project_dir`. Ensure dbt has write permissions to this directory.
*   **Custom CLI Flags:** Use the `flags` dictionary in the input objects to pass any additional dbt CLI flags not explicitly exposed as input parameters. This provides flexibility for advanced dbt usage.
*   **Idempotency:** Design your dbt models and tests to be idempotent, meaning they can be run multiple times without causing different results or side effects. This is a general dbt best practice that is particularly important in automated workflows.
*   **Secrets Management:** Handle database credentials and other sensitive information in your `profiles.yml` securely, typically by referencing environment variables or a secrets management system accessible within your Flyte execution environment.
<!--
key: summary_dbt_plugin_ec30e8fb-60bf-46c6-b53b-6bf10626c0eb
type: summary_end

-->
<!--
code_unit: flytekitplugins.dbt.task
code_unit_type: class
help_text: ''
key: example_7b5cc198-b645-440c-b485-fc6682e17940
type: example

-->