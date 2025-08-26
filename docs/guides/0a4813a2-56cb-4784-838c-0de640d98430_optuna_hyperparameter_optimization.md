
<!--
help_text: ''
key: summary_optuna_hyperparameter_optimization_9d3f909d-5d13-4754-9071-85ee9317f555
modules:
- flytekitplugins.optuna
- flytekitplugins.optuna.optimizer
questions_to_answer: []
type: summary

-->
# Optuna Hyperparameter Optimization

Hyperparameter optimization is a critical step in machine learning workflows, enabling the discovery of optimal model configurations. This plugin integrates Optuna, a popular hyperparameter optimization framework, with Flytekit, allowing for distributed and scalable optimization of Flyte tasks.

## The Optimizer

The core component for performing hyperparameter optimization is the `Optimizer` class. It orchestrates the execution of an objective function across multiple trials, leveraging Optuna's intelligent search algorithms.

An `Optimizer` instance is configured with the following parameters:

*   **`objective`**: The function to be optimized. This can be either a standard Python callable or a Flytekit `AsyncPythonFunctionTask`. The `objective` function must return a single `float` for single-objective optimization or a `tuple` of `float`s for multi-objective optimization.
    *   If the `objective` is a standard callable, it must accept an `optuna.Trial` object as its first argument, named `trial`.
    *   If the `objective` is an `AsyncPythonFunctionTask`, the `optuna.Trial` object is implicitly processed and its suggested parameters are passed as keyword arguments to the task.
*   **`concurrency`**: An integer specifying the maximum number of trials to run in parallel. This directly controls the parallelism of your optimization process.
*   **`n_trials`**: The total number of optimization trials to execute.
*   **`study`**: An optional `optuna.Study` object. If provided, the `Optimizer` continues an existing study, which is useful for resuming optimizations or sharing study state. If `None`, a new `optuna.Study` is created automatically.
*   **`delay`**: An optional integer representing the delay in seconds between starting each trial. This can be useful for managing resource contention or staggering trial starts.

### Objective Function Requirements

The `Optimizer` enforces strict requirements on the `objective` function's return type and signature to ensure compatibility with Optuna's study directions:

*   **Single-Objective Optimization**: If the `objective` returns a single `float`, the associated `optuna.Study` must be configured for a single objective (e.g., `optuna.create_study(direction="minimize")`).
*   **Multi-Objective Optimization**: If the `objective` returns a `tuple` of `float`s, the number of elements in the tuple must match the number of directions defined in the `optuna.Study` (e.g., `optuna.create_study(directions=["minimize", "maximize"])`). All elements in the tuple must be `float`s.

### Distributed Execution

When an `Optimizer` instance is called, it asynchronously executes the specified number of trials (`n_trials`). It uses an `asyncio.Semaphore` to manage the `concurrency` limit, ensuring that only a specified number of trials run simultaneously. Each trial involves:

1.  **Asking for a new trial**: `self.study.ask()` retrieves a new set of hyperparameters from the Optuna study.
2.  **Executing the objective**: The `objective` function is invoked with the suggested parameters.
    *   For `AsyncPythonFunctionTask` objectives, the trial parameters are automatically processed and passed as inputs to the remote task.
    *   For callable objectives, the `optuna.Trial` object is passed directly, allowing the objective to explicitly suggest parameters using `trial.suggest_float`, `trial.suggest_int`, `trial.suggest_categorical`, etc.
3.  **Telling the study the result**: The result (a `float` or `tuple` of `float`s) is reported back to the Optuna study, along with the trial's state (e.g., `COMPLETE` or `FAIL`). If the objective execution raises an `EagerException`, the trial is marked as `FAIL`.

### Example: Using a Callable Objective

This example demonstrates optimizing a simple function using a standard Python callable as the objective.

```python
import optuna
import asyncio
from flytekitplugins.optuna.optimizer import Optimizer

# Define the objective function
async def my_objective(trial: optuna.Trial, x_offset: float) -> float:
    x = trial.suggest_float("x", -10, 10)
    y = trial.suggest_float("y", -5, 5)
    # Simulate some computation
    await asyncio.sleep(0.1)
    return (x - x_offset)**2 + y**2

async def run_optimization():
    # Create an Optuna study
    study = optuna.create_study(direction="minimize")

    # Initialize the Optimizer
    optimizer = Optimizer(
        objective=my_objective,
        concurrency=2,  # Run 2 trials concurrently
        n_trials=10,    # Run a total of 10 trials
        study=study,
        delay=0.1       # 0.1 second delay between starting trials
    )

    # Run the optimization
    await optimizer(x_offset=2.0)

    print(f"Best trial: {study.best_trial.value}")
    print(f"Best parameters: {study.best_trial.params}")

# To run this in a Flyte workflow, you would typically wrap it in a task
# @task
# async def optuna_workflow_task():
#     await run_optimization()
```

### Example: Using a Flytekit Task as Objective

When the objective is a Flytekit `AsyncPythonFunctionTask`, the `Optimizer` handles the remote execution and parameter passing. The task's inputs are automatically populated from the Optuna trial's suggestions.

```python
import optuna
import asyncio
from flytekit import task, workflow
from flytekitplugins.optuna.optimizer import Optimizer

# Define a Flytekit task as the objective
@task
async def flyte_objective(x: float, y: float, x_offset: float) -> float:
    # This task would run remotely on Flyte
    print(f"Running trial with x={x}, y={y}, x_offset={x_offset}")
    await asyncio.sleep(0.1) # Simulate work
    return (x - x_offset)**2 + y**2

@workflow
async def optuna_workflow(x_offset: float = 2.0) -> float:
    # Create an Optuna study.
    # Note: For Flytekit tasks, the search space definition (e.g., trial.suggest_float)
    # is typically handled by a separate mechanism or implicitly by the Optimizer's
    # integration with Flytekit's parameter processing.
    # For this example, we assume the Optimizer's internal 'process' function
    # maps trial suggestions to task inputs.
    study = optuna.create_study(direction="minimize")

    # Initialize the Optimizer with the Flytekit task
    optimizer = Optimizer(
        objective=flyte_objective,
        concurrency=3,
        n_trials=15,
        study=study,
    )

    # Run the optimization. The 'x_offset' input is passed to each trial.
    await optimizer(x_offset=x_offset)

    # Return the best value found by the study
    return study.best_trial.value

# To run this workflow:
# pyflyte run your_script.py optuna_workflow --x-offset 2.0
```

In the Flytekit task example, the `Optimizer`'s internal `process` function (not explicitly shown in the provided code) is responsible for mapping the `optuna.Trial` object's suggested parameters (e.g., `trial.suggest_float("x", ...)`) to the input arguments of the `flyte_objective` task (`x`, `y`). This allows the task to remain clean and focused on its computation, while the `Optimizer` handles the Optuna integration.

## Defining Search Spaces

While the `Optimizer` class itself orchestrates trials, the actual definition of the hyperparameter search space occurs within the `objective` function using Optuna's `trial.suggest_` methods. The plugin provides helper classes like `Suggestion`, `Number`, `Float`, `Integer`, and `Category` which conceptually represent different types of hyperparameter suggestions.

*   **`Float`**: Represents a floating-point hyperparameter with a defined `low` and `high` bound, an optional `step`, and a `log` scale option.
*   **`Integer`**: Represents an integer hyperparameter with `low`, `high`, `step`, and `log` scale options.
*   **`Category`**: Represents a categorical hyperparameter with a list of `choices`.

These classes are primarily for defining the structure of the search space. When using a callable `objective`, you would directly use `trial.suggest_float`, `trial.suggest_int`, `trial.suggest_categorical` within your function. For `AsyncPythonFunctionTask` objectives, the mapping of these suggestion types to task inputs is handled by the plugin's internal mechanisms.

## Important Considerations

*   **Study Persistence**: To persist the optimization results and resume studies, configure your `optuna.Study` with a persistent storage backend (e.g., a database). The `Optimizer` accepts an existing `optuna.Study` object, allowing you to manage its lifecycle externally.
*   **Error Handling**: If an objective trial fails (e.g., due to an unhandled exception within the objective function), the `Optimizer` reports the trial as `FAIL` to the Optuna study, allowing Optuna to adjust its search strategy accordingly.
*   **Resource Management**: The `concurrency` parameter is crucial for managing the computational resources consumed by your optimization. Set it appropriately based on the available capacity in your Flyte cluster.
*   **Multi-Objective Optimization**: When optimizing for multiple objectives, ensure the `optuna.Study` is initialized with the correct `directions` (e.g., `["minimize", "maximize"]`) and that your `objective` function returns a tuple of floats matching these directions.
<!--
key: summary_optuna_hyperparameter_optimization_9d3f909d-5d13-4754-9071-85ee9317f555
type: summary_end

-->
<!--
code_unit: flytekitplugins.optuna.examples.optuna_optimization_workflow
code_unit_type: class
help_text: ''
key: example_caaf3638-3c87-4845-a625-9c0744bd83f7
type: example

-->