
<!--
help_text: ''
key: summary_aws_batch_integration_58daccaf-b5ad-4b52-9308-e36bab05630d
modules:
- flytekitplugins.awsbatch.task
questions_to_answer: []
type: summary

-->
AWS Batch Integration

AWS Batch Integration enables Flyte tasks to execute natively on the AWS Batch service. This integration allows leveraging AWS Batch's robust capabilities for managing compute environments, job queues, and job execution, providing a scalable and efficient way to run compute-intensive Flyte workflows.

### Configuring AWS Batch Job Parameters

The `AWSBatchConfig` class defines the specific parameters for an AWS Batch job. Tasks marked with this configuration automatically execute on the AWS Batch service. These parameters directly influence the `SubmitJobInput` API call to AWS Batch.

The `AWSBatchConfig` class includes the following fields:

*   `parameters` (Optional[Dict[str, str]]): A dictionary of key-value pairs passed to the container as environment variables or command-line arguments, depending on the job definition. These are typically used to customize the job's behavior.
*   `schedulingPriority` (Optional[int]): The priority for the job. Jobs with a higher priority are scheduled before jobs with a lower priority.
*   `platformCapabilities` (str): The platform capabilities for the job. This defaults to `"EC2"`. Other possible values include `"FARGATE"`. This setting determines the compute environment type used for the job.
*   `propagateTags` (Optional[bool]): Specifies whether to propagate the tags from the job definition to the Amazon ECS task.
*   `tags` (Optional[Dict[str, str]]): A dictionary of key-value pairs to associate with the job. These tags can be used for cost allocation, access control, and other purposes.

When an `AWSBatchConfig` instance is provided to an AWS Batch task, its properties are serialized and sent to FlytePropeller. The `platformCapabilities` field, in particular, is used to configure the AWS Job Definition.

### Defining AWS Batch Tasks

The `AWSBatchFunctionTask` class is the core plugin that transforms a standard Python function into a task executable within an AWS Batch job. It extends `PythonFunctionTask`, allowing it to wrap any Python callable.

To define an AWS Batch task, instantiate `AWSBatchFunctionTask`, providing an `AWSBatchConfig` instance and the Python function to be executed.

```python
from flytekitplugins.awsbatch.task import AWSBatchConfig, AWSBatchFunctionTask
from flytekit import task

# Define an AWS Batch configuration
aws_batch_config = AWSBatchConfig(
    parameters={"MY_PARAM": "value"},
    schedulingPriority=10,
    platformCapabilities="EC2",
    tags={"project": "my-project"}
)

# Define a Python function that will run on AWS Batch
@task
def my_batch_job(input_data: str) -> str:
    """
    This function will execute as an AWS Batch job.
    """
    print(f"Processing input: {input_data}")
    return f"Processed: {input_data}"

# Create an AWS Batch task instance
aws_batch_task = AWSBatchFunctionTask(
    task_config=aws_batch_config,
    task_function=my_batch_job
)

# This task can now be used within a Flyte workflow
# @workflow
# def my_workflow(data: str) -> str:
#     result = aws_batch_task(input_data=data)
#     return result
```

When an `AWSBatchFunctionTask` is serialized, its `get_custom` method returns the `AWSBatchConfig` as a dictionary, which FlytePropeller uses to construct the `SubmitJobInput`. The `get_config` method adds `platformCapabilities` to the task template configuration, influencing the AWS Job Definition.

The `get_command` method constructs the command that the AWS Batch container executes. This command typically involves `pyflyte-execute` to run the wrapped Python function. For single jobs, FlytePropeller expects outputs to be written to a specific path: `{{.outputPrefix}}/0`. The `AWSBatchFunctionTask` automatically configures the `pyflyte-execute` command to use this output path, ensuring proper output handling for single AWS Batch jobs.

### Important Considerations

*   **Output Path Convention:** For single AWS Batch jobs, FlytePropeller expects task outputs to be located under `{{.outputPrefix}}/0`. The `AWSBatchFunctionTask` automatically handles this convention by setting the `--output-prefix` argument for `pyflyte-execute`.
*   **Job Definition Parameters:** Parameters specified in `AWSBatchConfig` (e.g., `platformCapabilities`) are used to create or select an appropriate AWS Job Definition. Ensure your AWS Batch environment has job definitions compatible with your task configurations.
*   **Container Image:** The container image used by the AWS Batch job definition must include `flytekit` and all necessary dependencies for your `task_function` to execute successfully.
*   **Single Job Execution:** The current implementation of the AWS Batch plugin in FlytePropeller primarily supports running tasks as single AWS Batch jobs. While AWS Batch supports array jobs, this integration focuses on individual task execution.
*   **AWS Batch Service Quotas:** Be mindful of AWS Batch service quotas (e.g., number of active jobs, job definitions) when designing and scaling your workflows.
<!--
key: summary_aws_batch_integration_58daccaf-b5ad-4b52-9308-e36bab05630d
type: summary_end

-->
<!--
code_unit: flytekitplugins.awsbatch.examples.simple_batch_task
code_unit_type: class
help_text: ''
key: example_1669d6bd-1505-4df2-b84e-45ef65fbf774
type: example

-->