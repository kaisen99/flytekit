
<!--
help_text: ''
key: summary_kubernetes_data_service_8d5370d0-49c9-4625-bfce-a28dcc0503b1
modules:
- flytekitplugins.k8sdataservice.connector
- flytekitplugins.k8sdataservice.k8s.kube_config
- flytekitplugins.k8sdataservice.k8s.manager
- flytekitplugins.k8sdataservice.sensor
- flytekitplugins.k8sdataservice.task
questions_to_answer: []
type: summary

-->
The Kubernetes Data Service provides a robust mechanism for deploying, managing, and interacting with stateful data services directly within a Kubernetes cluster. It simplifies the lifecycle management of such services, integrating their deployment and monitoring seamlessly into workflow orchestration.

### Core Components

*   **DataServiceTask**: This is the primary interface for defining and orchestrating a data service within a workflow. It encapsulates the configuration necessary to either deploy a new Kubernetes-native data service or connect to an existing one.
*   **DataServiceConfig**: A configuration object that specifies the desired state and operational parameters of the data service. This includes essential details such as the container `Image`, `Command` to execute, number of `Replicas`, `Port` exposure, and Kubernetes `Requests` and `Limits` for resources. It also supports referencing an `ExistingReleaseName` for managing pre-existing services.
*   **DataServiceMetadata**: An internal object used to store and pass metadata about the deployed data service, including its configuration and the dynamically generated or specified release name.
*   **DataServiceConnector**: An asynchronous connector responsible for orchestrating the actual Kubernetes API calls. It implements the core operations for the data service lifecycle: `create` (deploys the service), `get` (retrieves its status), and `delete` (cleans up resources).
*   **K8sManager**: An internal utility that abstracts direct Kubernetes API interactions. It handles the creation, status checking, and deletion of Kubernetes `StatefulSet` and `Service` resources. It relies on in-cluster Kubernetes configuration for authentication.
*   **CleanupSensor**: A dedicated sensor designed to ensure the proper cleanup of Kubernetes data service resources. Unlike the connector's `delete` method (which is typically called on task abortion or failure), the sensor provides a robust mechanism for post-workflow cleanup, especially for long-running services that need explicit termination.

### Capabilities

*   **Declarative Service Deployment**: Define the desired state of your data service using `DataServiceConfig`. The system automatically handles the deployment of corresponding Kubernetes `StatefulSet` and `Service` resources.
*   **Existing Service Integration**: Connect to and manage the lifecycle of an already deployed Kubernetes data service by specifying its `ExistingReleaseName`. This capability is crucial for persistent services that outlive individual workflow executions.
*   **Resource Management**: Allocate CPU and memory resources to your data service pods using `Requests` and `Limits` within `DataServiceConfig`, ensuring efficient resource utilization and performance.
*   **Automated Status Monitoring**: The connector continuously monitors the health and readiness of the deployed `StatefulSet`, reporting the service's status (e.g., pending, running, succeeded, failed) back to the workflow.
*   **Controlled Resource Cleanup**: Resources (StatefulSet and Service) can be automatically deleted upon task failure or abortion. For explicit cleanup after successful workflow completion or for independent management, the `CleanupSensor` provides a dedicated mechanism.

### Usage

#### Defining a Data Service Task

To deploy a new data service, define a `DataServiceTask` and provide a `DataServiceConfig` instance.

```python
from flytekitplugins.k8sdataservice.task import DataServiceTask, DataServiceConfig
from flytekit.types.resources import Resources
from flytekit import workflow, task

# Define your data service configuration
my_data_service_config = DataServiceConfig(
    Name="my-custom-data-service",
    Image="my-registry/my-data-service:latest",
    Command=["/usr/bin/my-service", "--port", "40000"],
    Port=40000,
    Replicas=1,
    Requests=Resources(cpu="500m", mem="1Gi"),
    Limits=Resources(cpu="1", mem="2Gi"),
    Cluster="default-cluster", # Optional, for context/logging
)

# Define the DataServiceTask
data_service_task = DataServiceTask(
    name="deploy-my-data-service",
    task_config=my_data_service_config,
)

@workflow
def my_workflow():
    # Deploy the data service. The task returns the name of the deployed service.
    service_name = data_service_task()
    # ... subsequent tasks can interact with the data service using 'service_name' ...
```

#### Connecting to an Existing Data Service

To manage an already deployed data service without creating a new one, specify its `ExistingReleaseName` in the `DataServiceConfig`. The `Image` and `Command` fields are still required by the configuration object but will not trigger a new deployment if `ExistingReleaseName` is set.

```python
from flytekitplugins.k8sdataservice.task import DataServiceTask, DataServiceConfig
from flytekit import workflow

# Configure to use an existing service
existing_service_config = DataServiceConfig(
    ExistingReleaseName="my-pre-existing-service",
    Image="placeholder/image:latest", # Required by DataServiceConfig, but ignored for existing services
    Command=["/bin/true"], # Required by DataServiceConfig, but ignored for existing services
    Cluster="default-cluster",
)

existing_data_service_task = DataServiceTask(
    name="connect-to-existing-service",
    task_config=existing_service_config,
)

@workflow
def connect_workflow():
    service_name = existing_data_service_task()
    # The workflow now operates with the existing service identified by 'service_name'
```

#### Cleaning Up Resources

The `DataServiceConnector` automatically attempts to delete resources if the task fails or is aborted. For explicit cleanup after a successful workflow, or for services that need to be torn down independently, use the `CleanupSensor`.

```python
from flytekitplugins.k8sdataservice.sensor import CleanupSensor
from flytekit import task, workflow
import asyncio

# Assume 'data_service_name' is the output from a DataServiceTask
# or a known release name.
@task
def cleanup_task(release_name: str, cluster: str):
    sensor = CleanupSensor(name="data-service-cleanup-sensor")
    # Set cleanup_data_service to True to trigger deletion
    # The poke method is async and should be awaited in an async context.
    # For a simple task, asyncio.run can be used.
    asyncio.run(sensor.poke(release_name=release_name, cleanup_data_service=True, cluster=cluster))
    print(f"Cleanup initiated for {release_name}")

@workflow
def full_lifecycle_workflow():
    # ... deploy data service (e.g., using data_service_task from above) ...
    service_name = data_service_task()
    # ... tasks that use the data service ...
    cleanup_task(release_name=service_name, cluster="default-cluster")
```

### Important Considerations

*   **Kubernetes Access**: The system relies on in-cluster Kubernetes configuration for API access. Ensure the service account associated with the workflow execution environment has the necessary RBAC permissions to create, get, and delete `StatefulSet` and `Service` resources in the target namespace.
*   **Namespace**: The `K8sManager` and `CleanupSensor` currently default to the `flyte` namespace for resource management.
*   **Resource Naming**: If `Name` is not provided in `DataServiceConfig`, a unique name is generated using a UUID prefix (`k8s-dataservice-`).
*   **Cleanup Behavior**: The `DataServiceConnector`'s `delete` method is invoked on task failure or abortion. For guaranteed cleanup after successful runs or for independent management, explicitly use the `CleanupSensor`. Setting `cleanup_data_service=False` in the sensor's `poke` method prevents deletion.
*   **Pod Security Context**: Deployed pods run with a specific security context (`fs_group=1001`, `run_as_group=1001`, `run_as_non_root=True`, `run_as_user=1001`). Ensure your container image is compatible with these settings.

### Best Practices

*   **Idempotency**: When using `ExistingReleaseName`, the system will attempt to use the existing service rather than creating a new one, making workflows more robust to retries and re-executions.
*   **Resource Allocation**: Always specify `Requests` and `Limits` in `DataServiceConfig` to ensure your data service pods receive adequate resources and do not overconsume cluster capacity. This is critical for stable and performant deployments.
*   **Explicit Cleanup**: For production deployments, integrate the `CleanupSensor` into your workflow or a separate cleanup process to manage the lifecycle of data services effectively, especially if they are long-lived or consume significant resources. This ensures resources are properly de-provisioned.
<!--
key: summary_kubernetes_data_service_8d5370d0-49c9-4625-bfce-a28dcc0503b1
type: summary_end

-->
<!--
code_unit: flytekitplugins.k8sdataservice.examples.data_service_task
code_unit_type: class
help_text: ''
key: example_99843066-ec6d-485b-b48a-97c13f268e6b
type: example

-->