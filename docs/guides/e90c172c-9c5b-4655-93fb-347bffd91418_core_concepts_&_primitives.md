
<!--
help_text: ''
key: summary_core_concepts_&_primitives_512e4e5e-391e-400a-8457-1287909b8273
modules:
- flytekit.core.base_task
- flytekit.core.python_function_task
- flytekit.core.python_auto_container
- flytekit.core.python_instance_task
- flytekit.core.task
- flytekit.core.workflow
- flytekit.core.launch_plan
- flytekit.core.node
- flytekit.core.promise
- flytekit.models.core.identifier
- flytekit.models.core.types
- flytekit.models.literals
- flytekit.models.interface
- flytekit.models.common
questions_to_answer: []
type: summary

-->
## Core Concepts & Primitives

Flytekit provides a robust set of core concepts and primitives that form the foundation for defining, orchestrating, and executing data and machine learning pipelines. These primitives enable developers to build reproducible, scalable, and maintainable workflows.

### Tasks: The Atomic Unit of Computation

Tasks represent the smallest, atomic unit of computation in Flyte. They encapsulate a single, well-defined operation.

*   **`Task`**: This is the foundational base class for all tasks, closely mirroring the Flyte IDL's `TaskTemplate`. It captures essential information like `task_type`, `name`, and `interface` (input/output signature). Derived classes extend this to provide Python-native interfaces and specific execution behaviors.
*   **`PythonTask`**: Extends `Task` to provide a Python-native interface. It serves as a base for tasks that have a Python function to be executed.
*   **`PythonAutoContainerTask`**: A specialized `PythonTask` that automatically manages container images and resources for task execution.
    *   **Containerization**: Automatically determines the container image for the task, allowing users to specify a `container_image` or rely on Flytekit's auto-detection.
    *   **Resource Management**: Configures compute `resources` (CPU, memory, GPU) using `requests` and `limits`. It also supports `secret_requests` for securely injecting credentials, `pod_template` for advanced Kubernetes pod configurations, `accelerator` for specialized hardware, and `shared_memory` settings.
    *   **Command Generation**: The `get_command` method defines the entrypoint for the task's container, typically `pyflyte-execute`, along with arguments for input/output handling and task resolution.
    *   **Task Resolution**: Uses a `TaskResolverMixin` to locate and load the task at runtime. The `set_resolver` method allows overriding the default resolver.
*   **`PythonFunctionTask`**: The most common way to define tasks in Flytekit, typically used with the `@task` decorator. It automatically infers the task's `interface` from the Python function's signature.
    *   **Execution Behavior**: Tasks can operate in different `execution_mode`s:
        *   `DEFAULT`: Standard task execution.
        *   `DYNAMIC`: Enables the task function to generate a sub-workflow at runtime, allowing for dynamic graph construction.
        *   `EAGER`: Transforms the Python function into an "eager workflow," where Python directly orchestrates backend executions, creating a stack frame on the Flyte cluster for each task invocation.
    *   **Input Handling**: Supports `ignore_input_vars` to exclude certain inputs from the task's interface, useful for injecting client-side parameters.
    *   **Dependencies**: `node_dependency_hints` can be used for dynamic tasks/workflows where Flyte cannot automatically determine dependencies before runtime.
    *   **Execution Lifecycle**: Defines methods like `pre_execute` (before input conversion), `execute` (the core task logic), and `post_execute` (after execution, for cleanup or output alteration).
*   **`EagerAsyncPythonFunctionTask`**: A specialized `PythonFunctionTask` for eager workflows. It overrides the `async_execute` method to directly run the task function or interact with the Flyte backend for remote execution. The `run_with_backend` method facilitates live execution against a Flyte cluster.
*   **`PythonInstanceTask`**: Used for tasks that do not have a user-defined Python function body but have a platform-defined `execute` method. It ensures the module loader can automatically invoke the correct class.
*   **`ReferenceTask`**: Represents a pointer to an existing task registered on the Flyte platform. Its function body is not used; only its signature is relevant for compilation.
*   **`Echo`**: A special task type that simply echoes its inputs as outputs. It is handled by a Flyte plugin and does not create a separate pod, making it efficient for simple pass-through operations.
*   **`TaskMetadata`**: Configures various operational aspects of a task, including:
    *   `cache`: Enables caching of task outputs. Requires `cache_version`.
    *   `cache_serialize`: Ensures identical task instances execute in serial when caching is enabled.
    *   `cache_ignore_input_vars`: Specifies inputs to exclude when calculating the cache hash.
    *   `retries`: Number of times to retry the task on recoverable failures.
    *   `timeout`: Maximum duration for a single task execution.
    *   `interruptible`: Allows the task to be scheduled on lower QoS nodes that might be preempted.
    *   `is_eager`: Indicates if the task is part of an eager workflow.
*   **`IgnoreOutputs`**: An exception that can be raised within a task to indicate that its outputs should be safely ignored, useful in distributed training or peer-to-peer algorithms.
*   **`TaskPlugins`**: A factory for registering and discovering custom task types that extend `PythonFunctionTask`. This enables Flytekit to support various backend plugins (e.g., for Athena, Hive, PyTorch, TensorFlow).

### Task Resolution and Execution

Flytekit's execution model relies on resolvers to locate and rehydrate task objects at runtime.

*   **`TaskResolverMixin`**: An abstract base class defining the interface for task resolvers. It specifies methods like `location` (the resolver's unique identifier), `load_task` (to rehydrate a task given loader arguments), and `loader_args` (to generate arguments for identifying a task).
*   **`DefaultTaskResolver`**: The standard resolver that locates tasks by their module and name (e.g., `task-module your_module task-name your_task_function`). It's suitable for module-level functions.
*   **`DefaultNotebookTaskResolver`**: A specialized resolver for tasks defined within Jupyter notebooks. It loads tasks from a pickled file (`.pkl`) containing the task definitions.
*   **Execution Flow**:
    *   `local_execute`: Handles local execution of tasks, translating Python native inputs to Flyte literals and managing caching.
    *   `dispatch_execute`: The core method that translates Flyte's type system inputs and invokes the actual task executor. This is called both locally and during remote execution.
    *   `pre_execute` and `post_execute`: Hooks for modifying execution parameters or processing results before and after the main `execute` method.

### Workflows: Orchestrating Tasks

Workflows define the directed acyclic graph (DAG) of tasks and sub-workflows, specifying their dependencies and data flow.

*   **`WorkflowBase`**: The common base class for all workflow types. It manages the workflow's `name`, `workflow_metadata`, `workflow_metadata_defaults`, and `python_interface`. It also supports an `on_failure` handler for custom error handling.
*   **`PythonFunctionWorkflow`**: Workflows defined using Python functions, typically with the `@workflow` decorator.
    *   **Compilation**: The `compile` method analyzes the workflow function to build the underlying Flyte graph (`nodes` and `output_bindings`).
    *   **Dynamic Workflows**: When `execution_mode` is `DYNAMIC`, the `dynamic_execute` method compiles the Python function into a `DynamicJobSpec` at runtime, allowing for flexible, data-dependent graph generation.
*   **`ImperativeWorkflow`**: Provides a programmatic way to define workflows, offering fine-grained control over node creation and dependencies.
    *   **Building Blocks**: Uses `add_workflow_input` to define workflow inputs, `add_entity` to add tasks or sub-workflows as nodes, and `add_workflow_output` to specify workflow outputs.
    *   **Failure Handling**: `add_on_failure_handler` allows attaching a task or sub-workflow to execute upon workflow failure.
    *   **Readiness**: The `ready` method indicates if the workflow definition is complete and valid for compilation.
*   **`ReferenceWorkflow`**: A pointer to an existing workflow on the Flyte platform, similar to `ReferenceTask`.
*   **`WorkflowMetadata`**: Defines metadata for the workflow itself, primarily its `on_failure` policy.
*   **`WorkflowMetadataDefaults`**: Specifies default settings (e.g., `interruptible`) that are inherited by tasks within the workflow.
*   **`WorkflowFailurePolicy`**: Determines how a workflow behaves when a node fails: `FAIL_IMMEDIATELY` (default) or `FAIL_AFTER_EXECUTABLE_NODES_COMPLETE`.

### Launch Plans: Executing Workflows

Launch Plans define how a workflow is executed, including default inputs, schedules, and execution-specific configurations.

*   **`LaunchPlan`**: The primary primitive for triggering workflow executions.
    *   **Creation**: The `create` and `get_or_create` class methods provide flexible ways to instantiate launch plans. `get_or_create` allows retrieving a cached default launch plan for a workflow or creating a new named one.
    *   **Input Configuration**: Supports `default_inputs` (overridable at runtime) and `fixed_inputs` (immutable at runtime).
    *   **Scheduling**: Configures `schedule` for recurring executions and `notifications` for alerts (Email, PagerDuty, Slack).
    *   **Execution Overrides**: Allows specifying `labels`, `annotations`, `raw_output_data_config` (for offloaded data), `max_parallelism` (for workflow-level concurrency), `security_context` (for execution identity), `trigger` (for advanced scheduling), `overwrite_cache` (to force re-execution), and `auto_activate` (for automatic activation on registration).
    *   **Execution**: When a `LaunchPlan` object is called (`__call__`), it either creates a node in a compilation context or directly invokes the underlying workflow for local execution, applying its configured inputs and overrides.
*   **`ReferenceLaunchPlan`**: A pointer to an existing launch plan on the Flyte platform.

### Data Flow and Graph Construction

Flytekit uses a promise-based system to represent data flow and construct the workflow graph.

*   **`Promise`**: Represents a future value that will be produced by a task or node execution.
    *   **Duality**: During workflow compilation, task calls return `Promise` objects that point to the `Node` producing the output. During local execution, `Promise` objects wrap the actual Python native values.
    *   **Attributes**: Contains `var` (variable name), `val` (the literal value if `is_ready`), and `ref` (a `NodeOutput` reference if not ready).
    *   **Overrides**: The `with_overrides` method allows applying execution-specific configurations (e.g., resources, retries, caching) to the underlying node that produces this promise.
    *   **Conditional Logic**: Supports comparison operators (`==`, `!=`, `>`, `>=`, `<`, `<=`) and logical conjunctions (`&`, `|`) for building conditional branches in workflows.
    *   **Attribute Access**: Allows accessing nested attributes or items using `.` (e.g., `promise.attribute`) or `[]` (e.g., `promise["key"]`), which appends to the `attr_path` for deferred resolution.
*   **`NodeOutput`**: A specific reference to a named output variable from a `Node`. It includes the `node_id` and the `var` name.
*   **`VoidPromise`**: A special `Promise` returned by tasks that do not produce any outputs. It cannot be used in comparisons or operations.
*   **`Node`**: Represents an execution unit within a workflow's graph. It encapsulates a `flyte_entity` (a task, sub-workflow, or launch plan), its `metadata`, `bindings` (how its inputs are connected), and `upstream_nodes` (explicit dependencies).
    *   **Dependencies**: The `runs_before` method (aliased by the `>>` operator) explicitly defines execution order between nodes.
    *   **Overrides**: The `with_overrides` method allows applying runtime configurations directly to a specific node in the workflow graph.
*   **`ComparisonExpression`**: Represents a comparison between two operands (e.g., `x > 5`). Used as a building block for conditional statements.
*   **`ConjunctionExpression`**: Represents a logical combination of two expressions using `AND` (`&`) or `OR` (`|`). Used to build complex conditional logic.

### Flyte Type System and Literals

Flyte has a strong type system to ensure data consistency and enable platform optimizations. Literals are the concrete representations of data values.

*   **`Literal`**: The fundamental representation of any data value in Flyte. It can hold a `scalar`, `collection` (list), or `map` (dictionary) of other literals. It also includes `hash` for caching and `metadata`.
*   **`Scalar`**: A wrapper for single, atomic values. It can contain:
    *   **`Primitive`**: Basic data types like `integer`, `float_value`, `string_value`, `boolean`, `datetime`, and `duration`.
    *   **`Blob`**: References to offloaded binary data (e.g., files in S3). Includes `BlobMetadata` for format and dimensionality.
    *   **`Binary`**: Raw binary data with an optional `tag`.
    *   **`Schema`**: References to structured data with a defined schema.
    *   **`StructuredDataset`**: References to tabular datasets, including `StructuredDatasetMetadata` for type information.
    *   **`Union`**: Represents a tagged union type, holding a value and its `stored_type`.
    *   **`Void`**: Represents the absence of a value.
    *   `Error`: Represents an error literal.
    *   `Generic`: For unstructured data represented as a Protobuf `Struct`.
*   **`LiteralCollection`**: A list of `Literal` objects.
*   **`LiteralMap`**: A dictionary mapping string keys to `Literal` objects.
*   **`Binding`**: Connects a variable to a `BindingData` value.
*   **`BindingData`**: Specifies how an input variable is bound, either to a `scalar` value, a `collection` of bindings, a `promise` (output from another node), or a `map` of bindings.
*   **`TypedInterface`**: Defines the complete input and output signature of a task or workflow, mapping variable names to `Variable` objects.
*   **`Variable`**: Describes a single input or output variable, including its `type` (a `LiteralType`) and `description`. It can also include `artifact_partial_id` and `artifact_tag` for artifact management.
*   **`Parameter`**: Declares an input parameter for a launch plan, allowing for `default` values or marking it as `required`. It can also bind to `artifact_query` or `artifact_id`.
*   **`VariableMap`**: A collection of `Variable` objects.
*   **`ParameterMap`**: A collection of `Parameter` objects.

### Identifiers and Common Models

These classes provide universal identification and common metadata structures used across Flyte entities.

*   **`Identifier`**: A universal identifier for Flyte entities (Tasks, Workflows, Launch Plans). It comprises `resource_type`, `project`, `domain`, `name`, and `version`.
*   **`ResourceType`**: An enum defining the different types of Flyte entities that can be identified (TASK, WORKFLOW, LAUNCH_PLAN).
*   **`WorkflowExecutionIdentifier`**: Identifies a specific workflow execution by `project`, `domain`, and `name`.
*   **`NodeExecutionIdentifier`**: Identifies a specific node execution within a workflow execution.
*   **`TaskExecutionIdentifier`**: Identifies a specific task execution within a node execution, including the `retry_attempt`.
*   **`SignalIdentifier`**: Identifies a signal associated with a gate node in a workflow.
*   **`FlyteIdlEntity`**: The base class for all Flyte IDL models, providing common methods for serialization (`to_flyte_idl`), deserialization (`from_flyte_idl`), and string representation.
*   **`FlyteCustomIdlEntity`**: Extends `FlyteIdlEntity` for custom plugin data that can be serialized to/from a dictionary.
*   **`AuthRole`**: (Deprecated, use `SecurityContext`) Specifies IAM roles or Kubernetes service accounts for execution.
*   **`Annotations`**: Key-value pairs for attaching arbitrary metadata to workflow execution resources.
*   **`Labels`**: Key-value pairs for categorizing and organizing workflow execution resources.
*   **`Notification`**: Configures notifications for various execution `phases` (e.g., `EmailNotification`, `PagerDutyNotification`, `SlackNotification`).
*   **`RawOutputDataConfig`**: Specifies the prefix for offloading raw output data (e.g., to S3).
*   **`UrlBlob`**: Represents a URL and byte size for external data.
*   **`Envs`**: A collection of environment variables.
*   **`NamedEntityIdentifier`**: Identifies named entities within a project and domain.
<!--
key: summary_core_concepts_&_primitives_512e4e5e-391e-400a-8457-1287909b8273
type: summary_end

-->
<!--
code_unit: flytekit.core.task.Task
code_unit_type: class
help_text: ''
key: example_c4ba6265-faf6-4668-b765-e5551e2b8134
type: example

-->
<!--
code_unit: flytekit.core.workflow.WorkflowBase
code_unit_type: class
help_text: ''
key: example_001b69bc-d9d6-4a7e-bd26-c0cf7d5c759c
type: example

-->