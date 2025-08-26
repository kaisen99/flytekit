
<!--
help_text: ''
key: summary_gate_nodes_(manual_approval_and_sleep)_688fea64-9fe3-44b1-a75d-be46d835a7d1
modules:
- flytekit.core.gate
questions_to_answer: []
type: summary

-->
Gate nodes provide a mechanism to introduce pauses or manual intervention points within a workflow execution. Unlike typical tasks that execute code, gate nodes halt workflow progression, awaiting either a specified duration to elapse or explicit user input to proceed. This capability is essential for implementing human-in-the-loop processes, staged deployments, or time-based delays.

### Gate Node Types

The `Gate` class supports distinct modes of operation, primarily categorized by their waiting mechanism:

#### Manual Approval Gates

Manual Approval Gates pause workflow execution until a user explicitly approves or disapproves a specific value or state. This is particularly useful for critical checkpoints where human oversight is required before proceeding.

*   **Functionality:** An Approval Gate takes an upstream node's output as input and presents it to a user for confirmation. If approved, the workflow continues, and the approved value is passed downstream. If disapproved, the workflow raises an exception, halting further execution.
*   **Definition:** To create an Approval Gate, pass the output of an upstream node to the `Gate` constructor. The `upstream_item` parameter links the gate to the value requiring approval.

    ```python
    from flytekit import workflow
    from flytekit.core.gate import Gate

    @workflow
    def approval_workflow(data: str) -> str:
        # Assume 'data' is the value that needs approval
        # The 'approve' method (not shown in class contents but implied by usage)
        # would typically wrap the Gate instantiation to simplify usage.
        # Conceptually, it looks like this:
        approval_gate = Gate(name="approve_data", upstream_item=data)
        # In a real workflow, you'd chain this:
        # approved_data = approval_gate.approve()
        # next_task(approved_data)
        return approval_gate # Simplified representation for illustration
    ```

    During execution, the system prompts the user to approve the value passed to the gate.

#### Sleep Gates

Sleep Gates introduce a controlled delay in the workflow execution. The workflow pauses for a predefined duration before automatically resuming. This is useful for rate limiting, waiting for external systems to stabilize, or scheduling staggered operations.

*   **Functionality:** A Sleep Gate pauses the workflow for the duration specified by `sleep_duration`. No user interaction is required; the workflow automatically proceeds once the time elapses.
*   **Definition:** Specify the `sleep_duration` parameter in the `Gate` constructor using a `datetime.timedelta` object.

    ```python
    import datetime
    from flytekit import workflow
    from flytekit.core.gate import Gate

    @workflow
    def delayed_workflow():
        # Pause for 5 minutes
        Gate(name="wait_five_minutes", sleep_duration=datetime.timedelta(minutes=5))
        # Subsequent tasks will run after the delay
        # ...
    ```

#### Input Gates

Input Gates pause workflow execution, awaiting user input of a specified type. This allows for dynamic data injection or configuration during a running workflow.

*   **Functionality:** An Input Gate prompts the user to provide data of a defined type. Once the input is provided, the workflow continues, and the user-provided data becomes the output of the gate node, available for downstream tasks.
*   **Definition:** Specify the `input_type` parameter in the `Gate` constructor. The output of the gate will be of this type.

    ```python
    from flytekit import workflow
    from flytekit.core.gate import Gate

    @workflow
    def configure_workflow() -> int:
        # Wait for user to input an integer configuration value
        config_value = Gate(name="get_config", input_type=int)
        # The 'config_value' can then be used by subsequent tasks
        # ...
        return config_value
    ```

### Defining Gate Nodes

All gate nodes share common parameters and integrate seamlessly into workflows.

*   **Common Parameters:**
    *   `name` (str): A unique name for the gate node within the workflow. This name is used for identification and logging.
    *   `timeout` (datetime.timedelta, optional): The maximum duration the gate will wait. If the timeout is reached before the condition (sleep duration elapsed, user approval/input) is met, the gate node fails. Defaults to a system-defined timeout if not specified.

*   **Workflow Integration:** Gate nodes are instantiated directly within a workflow definition. They behave like any other node, allowing for chaining with upstream and downstream tasks.

    ```python
    from flytekit import workflow, task
    from flytekit.core.gate import Gate
    import datetime

    @task
    def process_data(data: str) -> str:
        return f"Processed: {data}"

    @task
    def deploy_model(model_id: str):
        print(f"Deploying model {model_id}")

    @workflow
    def staged_deployment_workflow(model_id: str) -> str:
        # Step 1: Process data (example task)
        processed_output = process_data(data=model_id)

        # Step 2: Manual Approval Gate for deployment
        # The 'approve' method is a common pattern for creating approval gates
        # and would internally use Gate(upstream_item=...)
        # For this example, we'll show the direct Gate usage.
        approval_gate = Gate(name="deploy_approval", upstream_item=processed_output)

        # Step 3: Sleep Gate after approval, before final deployment
        Gate(name="stagger_deployment", sleep_duration=datetime.timedelta(minutes=10))

        # Step 4: Final deployment task
        deploy_model(model_id=approval_gate) # approval_gate passes its approved value

        return "Deployment workflow completed"
    ```

### Execution Behavior

The behavior of gate nodes differs between local and remote execution environments.

#### Local Execution

During local execution, gate nodes simulate their waiting behavior using command-line prompts.

*   **Sleep Gates:** Print a message indicating the simulated sleep duration and immediately proceed. No actual time delay occurs.
*   **Input Gates:** Prompt the user via the command line to enter the required input value. The workflow pauses until valid input is provided.
*   **Approval Gates:** Prompt the user via the command line to confirm approval of the upstream value. The workflow pauses until the user responds. If the user declines, a `FlyteDisapprovalException` is raised, halting the local execution.

#### Remote Execution and User Interaction

In a remote execution environment (e.g., on a Flyte cluster), gate nodes genuinely pause the workflow.

*   **Sleep Gates:** The workflow pauses for the specified `sleep_duration`. The execution state changes to "waiting" and automatically resumes after the duration.
*   **Manual Approval/Input Gates:** The workflow pauses, and the execution state changes to "waiting for input" or "waiting for approval." Users interact with these gates through the platform's UI or API, providing input or approval. The workflow resumes only after the required interaction occurs.

### Practical Applications

Gate nodes enable powerful control flow patterns:

*   **Staged Deployments:** Implement a manual approval step before deploying to production environments, ensuring human review of artifacts or metrics.
*   **Data Validation Checkpoints:** Pause a data pipeline to allow data quality engineers to review intermediate results before expensive downstream computations proceed.
*   **Scheduled Delays:** Introduce a fixed delay between workflow steps, for example, to allow external systems to synchronize or to stagger resource-intensive operations.
*   **A/B Testing Rollouts:** Use a sleep gate to gradually roll out a new feature, allowing for monitoring and potential rollback before full deployment.
*   **Dynamic Configuration:** Allow users to provide runtime configuration values for a workflow, such as the number of parallel workers or a specific data filter.

### Considerations and Limitations

*   **Timeouts:** Always consider setting an appropriate `timeout` for gate nodes, especially for manual gates. This prevents workflows from indefinitely waiting for user input, which could lead to resource waste or deadlocks.
*   **Disapproval Handling:** For Approval Gates, understand that user disapproval results in a `FlyteDisapprovalException`. Design your workflows to handle this exception gracefully if specific recovery or notification actions are required.
*   **Local vs. Remote Differences:** Be aware that local execution of sleep gates is simulated. For accurate testing of time-based delays, remote execution is necessary. Manual gates require actual user interaction in both environments, but the interface differs (CLI locally, UI/API remotely).
*   **User Interface:** The user experience for interacting with manual gates in a remote environment depends on the specific platform's UI/API capabilities. Ensure users are aware of how to approve or provide input.
<!--
key: summary_gate_nodes_(manual_approval_and_sleep)_688fea64-9fe3-44b1-a75d-be46d835a7d1
type: summary_end

-->
<!--
code_unit: flytekit.core.gate.Gate.sleep_duration
code_unit_type: class
help_text: ''
key: example_34eec7be-95ac-4459-a788-bb54bbb9a2ad
type: example

-->
<!--
code_unit: flytekit.core.gate.Gate.input_type
code_unit_type: class
help_text: ''
key: example_115e5176-7457-4d20-b4b9-c8cd925fcfec
type: example

-->
<!--
code_unit: flytekit.core.gate.Gate.local_execute
code_unit_type: class
help_text: ''
key: example_e355243e-b05e-4e93-ab69-84674774e6d2
type: example

-->