
<!--
help_text: ''
key: summary_artifacts_for_data_versioning_and_querying_0927c951-1916-4ca8-b9e8-f63c7f6277c9
modules:
- flytekit.core.artifact
questions_to_answer: []
type: summary

-->
Artifacts provide a robust mechanism for versioning and querying data within the platform, enabling discoverability, reproducibility, and clear data lineage. An Artifact acts as a metadata layer over data, allowing users to define how data is structured, partitioned, and subsequently queried.

### Defining Artifacts

An `Artifact` object defines the metadata schema for a dataset. This includes its name, whether it is time-partitioned, and any additional partition keys.

To define an artifact:

```python
from flytekit.core.artifact import Artifact, Granularity
from datetime import datetime, timedelta

# A simple artifact with a name
MySimpleData = Artifact(name="my.simple.data")

# An artifact with static partitions
PricingData = Artifact(name="pricing.data", partitions={"region": "us-east-1"})

# An artifact with dynamic partition keys
SalesData = Artifact(name="sales.data", partition_keys=["region", "product_category"])

# An artifact that is time-partitioned
DailyMetrics = Artifact(name="daily.metrics", time_partitioned=True, time_partition_granularity=Granularity.DAY)

# An artifact with both partition keys and time partitioning
HourlyLogs = Artifact(name="hourly.logs", partition_keys=["service_name"], time_partitioned=True, time_partition_granularity=Granularity.HOUR)
```

*   `name`: A required string that uniquely identifies the artifact.
*   `partition_keys`: A list of strings defining the keys for dynamic partitions. These keys will be used to segment the data.
*   `time_partitioned`: A boolean indicating if the artifact is partitioned by time.
*   `time_partition_granularity`: Specifies the granularity (e.g., `Granularity.DAY`, `Granularity.HOUR`) for time-partitioned artifacts. If not specified, `DAY` is the default.
*   A hard limit of 10 partition keys per artifact is currently enforced.

### Producing Artifacts

Artifacts are typically produced as outputs of tasks or workflows. There are two primary ways to associate data with an artifact: at definition time (compile-time) or dynamically at runtime.

#### Declaring Artifact Outputs in Tasks

Use `Annotated` to declare that a task or workflow output produces an artifact. You can bind partition values directly in the annotation.

```python
from flytekit import task, workflow
from flytekit.core.artifact import Artifact, Inputs, Granularity
from typing import Annotated
import pandas as pd
import datetime

# Define artifacts
DailySales = Artifact(name="daily.sales", partition_keys=["region"], time_partitioned=True, time_partition_granularity=Granularity.DAY)
ProductCatalog = Artifact(name="product.catalog", partition_keys=["category"])

@task
def generate_sales_report(
    region: str,
    report_date: datetime.datetime,
) -> Annotated[pd.DataFrame, DailySales(region=Inputs.region, time_partition=Inputs.report_date)]:
    """
    Generates a daily sales report for a given region and date.
    The region and report_date are dynamically bound from task inputs.
    """
    df = pd.DataFrame({"sales": [100, 200], "product": ["A", "B"]})
    # Simulate data generation
    return df

@task
def update_product_catalog(
    category: str,
) -> Annotated[pd.DataFrame, ProductCatalog(category=category)]:
    """
    Updates the product catalog for a specific category.
    The category partition is statically bound at task definition.
    """
    df = pd.DataFrame({"product_id": [1, 2], "name": ["Laptop", "Mouse"]})
    return df

@workflow
def sales_workflow(region: str, report_date: datetime.datetime):
    generate_sales_report(region=region, report_date=report_date)
    update_product_catalog(category="electronics")

# Example execution (conceptual)
# sales_workflow(region="us-west-1", report_date=datetime.datetime(2023, 10, 26))
```

In this example:
*   `DailySales(region=Inputs.region, time_partition=Inputs.report_date)`: The `region` and `time_partition` for `DailySales` are bound to the `region` and `report_date` inputs of the `generate_sales_report` task. `Inputs` is a special object (`InputsBase`) that allows referencing task/workflow inputs for dynamic binding.
*   `ProductCatalog(category=category)`: The `category` partition for `ProductCatalog` is bound to a static string value provided at the task definition.

The `ArtifactIDSpecification` class internally handles the binding of these partition values to create a partial artifact ID for the output.

#### Dynamically Creating Artifacts at Runtime

For scenarios where partition values are determined within the task's execution logic, use the `Artifact.create_from` method. This allows you to set partition values based on runtime variables.

```python
from flytekit import task
from flytekit.core.artifact import Artifact
from typing import Annotated
import pandas as pd
import datetime

# Define artifacts
Pricing = Artifact(name="pricing", partition_keys=["region"])
EstError = Artifact(name="estimation_error", partition_keys=["dataset"], time_partitioned=True)

@task
def analyze_data() -> (Annotated[pd.DataFrame, Pricing], Annotated[float, EstError]):
    """
    Analyzes data and produces two artifacts with dynamically determined partitions.
    """
    # Simulate data processing
    pricing_df = pd.DataFrame({"item": ["A"], "price": [10.0]})
    estimation_error_value = 0.05
    current_time = datetime.datetime.now()

    # Create artifacts with runtime-determined partition values
    return (
        Pricing.create_from(pricing_df, region="dubai"),
        EstError.create_from(estimation_error_value, dataset="train", time_partition=current_time)
    )

# Example execution (conceptual)
# df, error = analyze_data()
```

`Artifact.create_from` takes the actual data object as its first argument, followed by keyword arguments for partition keys and `time_partition`. This method integrates with the platform's output metadata tracking to correctly associate the data with the specified artifact and its partitions.

### Querying Artifacts

Once artifacts are produced, they can be queried to retrieve specific versions or subsets of data. The `Artifact.query()` method is used for this purpose.

```python
from flytekit.core.artifact import Artifact, Inputs
import datetime

# Assume these artifacts have been defined and produced data
DailySales = Artifact(name="daily.sales", partition_keys=["region"], time_partitioned=True)
ProductCatalog = Artifact(name="product.catalog", partition_keys=["category"])

# Query for specific static partitions
sales_query_us_east = DailySales.query(region="us-east-1", time_partition=datetime.datetime(2023, 10, 25))
# This query will retrieve the DailySales artifact for 'us-east-1' on 2023-10-25.

# Query for a specific product category
electronics_catalog_query = ProductCatalog.query(category="electronics")

# Query for a time partition relative to another time partition
# This is useful in triggers or when chaining queries
# For example, to get sales data from yesterday relative to an input date
yesterday_sales_query = DailySales.query(
    region=Inputs.region,
    time_partition=Inputs.report_date - datetime.timedelta(days=1)
)

# Querying based on another artifact's partitions
# This allows creating dependencies between artifacts in queries
# For example, if you have a 'RegionConfig' artifact with a 'region' partition
RegionConfig = Artifact(name="region.config", partition_keys=["region"])
sales_query_from_config = DailySales.query(
    region=RegionConfig.partitions.region, # Binds to the 'region' partition of the RegionConfig artifact
    time_partition=datetime.datetime(2023, 10, 25)
)
```

The `ArtifactQuery` object returned by `query()` encapsulates the artifact's name, project, domain, and specified partitions.
*   `project` and `domain`: Optional parameters to specify the project and domain of the artifact. If not provided, they are typically inferred from the current execution context or the artifact's definition.
*   `time_partition`: Can be a `datetime.datetime` object, a `TimePartition` object, or an `Inputs` binding.
*   `partitions` or `kwargs`: Used to specify values for the defined `partition_keys`. You can use a dictionary for `partitions` or pass key-value pairs directly as `kwargs`. Do not use both.

The `ArtifactQuery.bound` property indicates whether all required partitions (both time and regular) for the artifact have been specified in the query.

### Partitioning Details

The system supports two main types of partitions: regular key-value partitions and time-based partitions.

#### Regular Partitions (`Partitions`, `Partition`)

`Partitions` represents a collection of key-value pairs that segment the data. Each individual key-value pair is represented by a `Partition` object.

*   When defining an `Artifact` with `partition_keys`, these keys are expected to be provided when producing or querying the artifact.
*   Partition values can be static strings or dynamically bound to task/workflow inputs using `Inputs`.

#### Time Partitions (`TimePartition`)

`TimePartition` represents a time-based segmentation of data. It includes a `value` (a timestamp or input binding) and a `granularity` (e.g., `DAY`, `HOUR`).

*   **Granularity**: When defining a `time_partitioned` artifact, you can specify `time_partition_granularity` (e.g., `Granularity.DAY`). This helps in organizing and querying time-series data.
*   **Time Operations**: `TimePartition` objects support arithmetic operations with `timedelta` objects, allowing for relative time queries. For example, `Inputs.report_date - datetime.timedelta(days=1)` queries data from the day before `report_date`.

### Serialization and Internal Mechanisms

The `Serializer` class, along with its default implementation `DefaultArtifactSerializationHandler`, is responsible for converting Python artifact objects (`Partitions`, `TimePartition`, `ArtifactQuery`) into their platform-specific IDL (Interface Definition Language) representations. This conversion is crucial for communicating artifact metadata to the underlying platform services. While these classes are internal to the serialization process, understanding their role helps in comprehending how artifact definitions are translated for execution.

### Important Considerations

*   **Project and Domain**: While `project` and `domain` can be specified when defining or querying an `Artifact`, they are often automatically inferred from the execution context when producing artifacts. When querying, explicitly specifying them can narrow down the search.
*   **`embed_as_query`**: This method on the `Artifact` class is specifically designed for use in trigger definitions, allowing a trigger to reference properties of the artifact that initiated it (e.g., its time partition or a specific partition key). It is not intended for general data querying within tasks or workflows.
*   **Consistency Checking**: While the system provides mechanisms for defining and binding partitions, comprehensive consistency checking (e.g., ensuring all required partitions are bound) is limited, especially during the alpha phase. Developers should ensure that all necessary partition values are provided when producing or querying artifacts.
<!--
key: summary_artifacts_for_data_versioning_and_querying_0927c951-1916-4ca8-b9e8-f63c7f6277c9
type: summary_end

-->