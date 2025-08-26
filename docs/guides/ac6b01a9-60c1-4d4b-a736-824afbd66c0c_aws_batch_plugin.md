
<!--
help_text: ''
key: summary_aws_batch_plugin_2aaae14e-9002-4fc4-b22b-5304d3d0e58d
modules:
- flytekitplugins.awsbatch.task
questions_to_answer: []
type: summary

-->
The AWS Batch Plugin enables Flyte tasks to execute as native AWS Batch jobs, leveraging AWS's managed compute environment for scalable and robust job processing. This integration allows developers to offload compute-intensive or long-running tasks to AWS Batch, benefiting from its job scheduling, resource management, and cost optimization features.

### AWSBatchFunctionTask

The `AWSBatchFunctionTask` class is the core component for defining Flyte tasks that run on AWS Batch. It extends `PythonFunctionTask`, transforming a standard Python function into a task executable within the AWS Batch ecosystem.

When defining a task, specify `AWSBatchFunctionTask` as the task type and provide an `AWSBatchConfig` object to configure the underlying AWS Batch job.

```python
from flytekit import task
from flytekitplugins.awsbatch.task import AWSBatchConfig

@task(task_config=AWSBatchConfig(
    parameters={"my_param": "value"},
    schedulingPriority=100,
    platformCapabilities="EC2", # Default, but can be explicitly set
    tags={"project": "my-project"}
))
def my_aws_batch_task(input_data: str) -> str:
    # Your Python logic here, which will run inside an AWS Batch job
    print(f"Processing {input_data} in AWS Batch.")
    return f"Processed: {input_data}"
```

The plugin handles the serialization of task configurations and the generation of the necessary command for the container that runs within AWS Batch.

*   **`get_custom`**: This method serializes the `AWSBatchConfig` object into a dictionary, which FlytePropeller uses to construct the `SubmitJobInput` for the AWS Batch service.
*   **`get_config`**: This method includes the `platformCapabilities` from the `AWSBatchConfig` in the task template's configuration, which is crucial for defining the AWS Batch job definition. The default `platformCapabilities` is `EC2`.
*   **`get_command`**: This method constructs the `pyflyte-execute` command that the AWS Batch container will run. It ensures that inputs, outputs, and resolver details are correctly passed. Notably, for single jobs, the output is directed to `{{.outputPrefix}}/0`. This is due to how FlytePropeller's array batch plugin currently handles single job outputs, always expecting them in a subdirectory `0`.

### AWSBatchConfig

The `AWSBatchConfig` class provides a structured way to define the parameters for an AWS Batch job. These parameters directly map to the `SubmitJobInput` structure used by the AWS Batch service.

Use `AWSBatchConfig` to customize the behavior and environment of your AWS Batch jobs:

*   **`parameters`** (`Optional[Dict[str, str]]`): A dictionary of key-value pairs that are passed to the container as environment variables or command-line arguments, depending on the job definition.
*   **`schedulingPriority`** (`Optional[int]`): The priority of the job. Jobs with a higher priority are more likely to be scheduled sooner.
*   **`platformCapabilities`** (`str`): Specifies the compute environment type for the job. The default and currently supported value is `"EC2"`.
*   **`propagateTags`** (`Optional[bool]`): When set to `True`, tags associated with the job definition are propagated to the resources launched for the job (e.g., EC2 instances).
*   **`tags`** (`Optional[Dict[str, str]]`): A dictionary of key-value pairs to apply as tags to the AWS Batch job. These tags can be used for cost allocation, access control, and resource organization.

**Example Usage:**

```python
from flytekitplugins.awsbatch.task import AWSBatchConfig

# Configure an AWS Batch job with specific parameters and tags
batch_config = AWSBatchConfig(
    parameters={"DATA_SOURCE": "s3://my-bucket/data.csv"},
    schedulingPriority=50,
    tags={"environment": "production", "owner": "data-team"}
)

# This config can then be passed to an AWSBatchFunctionTask
# @task(task_config=batch_config)
# def process_data_batch(...):
#     pass
```

### Execution Flow

When a task defined with the AWS Batch Plugin is triggered:

1.  FlytePropeller receives the task execution request.
2.  It uses the `AWSBatchConfig` provided with the task to construct and submit a job to the configured AWS Batch job queue.
3.  AWS Batch provisions the necessary compute resources (e.g., EC2 instances) based on the job definition and compute environment.
4.  A container image (specified in the Flyte task's container definition) is launched on the AWS Batch compute resource. This container includes `flytekit` and your task's dependencies.
5.  Inside the container, the `pyflyte-execute` command runs, which then invokes your Python function with the provided inputs.
6.  Upon completion, the task's outputs are written to the designated output prefix in S3, and the AWS Batch job status is updated. FlytePropeller monitors this status and retrieves the outputs.

### Considerations and Best Practices

*   **AWS Batch Setup**: Ensure your AWS environment has AWS Batch configured with appropriate compute environments and job queues that the Flyte deployment can access. The IAM role associated with FlytePropeller must have permissions to submit and manage AWS Batch jobs.
*   **Container Images**: Your task's container image must include `flytekit` and all necessary Python dependencies for your task function. Optimize your container images for size and startup time.
*   **Resource Allocation**: Define appropriate CPU and memory limits in your Flyte task's container definition. These limits guide AWS Batch in selecting suitable instances and scheduling jobs efficiently.
*   **Output Handling**: Be aware that for single jobs, the plugin directs outputs to a `0` subdirectory within the specified output prefix. This is an internal detail handled by the plugin, but it's useful to understand for debugging or direct S3 access.
*   **Monitoring**: Leverage AWS CloudWatch logs and the AWS Batch console for detailed monitoring and debugging of your jobs.
*   **Cost Management**: Utilize `tags` in `AWSBatchConfig` for better cost allocation and tracking of resources consumed by your Flyte tasks on AWS Batch.
<!--
key: summary_aws_batch_plugin_2aaae14e-9002-4fc4-b22b-5304d3d0e58d
type: summary_end

-->
<!--
code_unit: flytekitplugins.awsbatch.task
code_unit_type: class
help_text: ''
key: example_49fe3671-1b49-423e-a612-67a6707a1f8e
type: example

-->