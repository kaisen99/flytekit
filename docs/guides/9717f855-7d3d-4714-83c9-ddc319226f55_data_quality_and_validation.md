
<!--
help_text: ''
key: summary_data_quality_and_validation_d17151a1-0083-4898-9ba8-792ac9aed61e
modules:
- flytekitplugins.whylogs
- flytekitplugins.whylogs.schema
- flytekitplugins.whylogs.renderer
- flytekitplugins.pandera.pandas_renderer
questions_to_answer: []
type: summary

-->
## Data Quality and Validation

Ensuring data quality is critical for reliable machine learning models and robust data pipelines. This section describes the capabilities for validating data, detecting anomalies, and monitoring data drift, leveraging integrations with `whylogs` for data profiling and `pandera` for schema-based validation.

### Data Profiling and Drift Detection

The system integrates with `whylogs` to provide comprehensive data profiling, enable the definition of custom data constraints, and facilitate the detection of data drift. These capabilities are essential for understanding data characteristics and identifying changes over time.

#### Persisting and Visualizing Data Profiles

Data profiles, represented by `whylogs.DatasetProfileView` objects, capture statistical summaries and other metadata about a dataset. The `WhylogsDatasetProfileTransformer` enables these profiles to be seamlessly passed as inputs and outputs between tasks. This allows for the persistence, sharing, and subsequent analysis of data characteristics across different stages of a data pipeline.

To visualize a `DatasetProfileView` as an HTML report, use the `to_html` method provided by the transformer:

```python
from flytekitplugins.whylogs.schema import WhylogsDatasetProfileTransformer
from whylogs.core.view import DatasetProfileView
# Assume 'profile_view' is an instance of DatasetProfileView obtained from a task output
# or loaded from storage.
# For example:
# import whylogs as why
# import pandas as pd
# df = pd.DataFrame({"col1": [1, 2, 3], "col2": ["A", "B", "C"]})
# profile_view = why.log(df).view()

transformer = WhylogsDatasetProfileTransformer()
html_report = transformer.to_html(ctx, profile_view, DatasetProfileView)
# 'html_report' now contains the HTML string for the profile summary report.
```

This HTML report provides a summary of the dataset's characteristics, including column-level statistics, data types, and value distributions.

#### Enforcing Data Constraints

Custom data quality rules can be defined as constraints against a profiled dataset. The `WhylogsConstraintsRenderer` generates an HTML report that visualizes the pass/fail status of these defined constraints. This allows for immediate feedback on whether the data adheres to expected quality standards.

To use this capability, first profile your data to create a `DatasetProfileView`, then define your constraints using `whylogs.core.constraints.ConstraintsBuilder` and `whylogs.core.constraints.MetricConstraint`. Finally, pass the built `Constraints` object to the renderer:

```python
import whylogs as why
import pandas as pd
from whylogs.core.constraints import MetricConstraint, ConstraintsBuilder
from whylogs.core.metrics.selectors import MetricsSelector
from flytekitplugins.whylogs.renderer import WhylogsConstraintsRenderer

df = pd.DataFrame({
    "sepal_length": [5.1, 4.9, 6.3, 5.0, 5.5],
    "sepal_width": [3.5, 3.0, 3.3, 3.6, 2.8]
})

# 1. Profile the DataFrame
profile_view = why.log(df).view()

# 2. Define constraints
builder = ConstraintsBuilder(profile_view)
min_value = 4.0
max_value = 7.0
num_constraint = MetricConstraint(
    name=f'sepal_length between {min_value} and {max_value} only',
    condition=lambda x: x.min > min_value and x.max < max_value,
    metric_selector=MetricsSelector(
        metric_name='distribution',
        column_name='sepal_length'
    )
)
builder.add_constraint(num_constraint)
constraints = builder.build()

# 3. Generate the HTML report
html_report = WhylogsConstraintsRenderer.to_html(constraints)
# 'html_report' now contains the HTML string visualizing constraint adherence.
```

The generated report highlights which constraints passed or failed, providing a clear overview of data quality against predefined rules.

#### Monitoring Data Drift

Data drift occurs when the statistical properties of a target dataset change relative to a reference dataset. The `WhylogsSummaryDriftRenderer` facilitates the detection and visualization of such changes by comparing two `pandas.DataFrame` objects. It generates an HTML report detailing the drift calculations for all columns.

This is particularly useful for monitoring production data against training data or comparing data across different time periods.

```python
import pandas as pd
from flytekitplugins.whylogs.renderer import WhylogsSummaryDriftRenderer

# Assume 'reference_df' is your baseline data (e.g., training data)
reference_df = pd.DataFrame({
    "feature_a": [1, 2, 3, 4, 5],
    "feature_b": [10, 11, 12, 13, 14]
})

# Assume 'target_df' is the new data to be compared (e.g., production data)
target_df = pd.DataFrame({
    "feature_a": [1.1, 2.2, 3.3, 4.4, 5.5],
    "feature_b": [100, 110, 120, 130, 140]
})

# Generate the Summary Drift report
html_report = WhylogsSummaryDriftRenderer.to_html(reference_data=reference_df, target_data=target_df)
# 'html_report' now contains the HTML string for the drift report.
```

The report provides insights into how each column's distribution has shifted, helping identify potential issues that could impact downstream processes or models.

### Schema-Based Data Validation

For strict enforcement of data structure and types, the system integrates with `pandera` to provide schema-based validation for `pandas.DataFrame` objects. This ensures that data conforms to a predefined schema, catching issues like incorrect data types, missing columns, or out-of-range values.

#### Validating DataFrames with Schemas

The `PandasReportRenderer` is responsible for generating detailed HTML reports from `pandera` validation results. It can produce a success report when data passes validation or a comprehensive error report when validation fails, detailing schema-level and data-level errors.

To perform schema validation and generate a report:

1.  Define a `pandera.DataFrameSchema` that specifies the expected structure and constraints of your DataFrame.
2.  Attempt to validate your DataFrame against this schema.
3.  If validation fails, a `pandera.errors.SchemaErrors` exception is raised, which contains detailed error information.
4.  Pass the DataFrame, schema, and optionally the error object to the `PandasReportRenderer.to_html` method.

```python
import pandas as pd
import pandera as pa
from pandera.errors import SchemaErrors
from flytekitplugins.pandera.pandas_renderer import PandasReportRenderer

# 1. Define a Pandera DataFrameSchema
schema = pa.DataFrameSchema(
    columns={
        "id": pa.Column(int, unique=True),
        "value": pa.Column(float, pa.Check.in_range(0, 100)),
        "category": pa.Column(str, pa.Check.isin(["A", "B", "C"])),
    },
    strict=True, # Ensure no extra columns
    name="MyDataFrameSchema"
)

# Example 1: Valid data
valid_df = pd.DataFrame({
    "id": [1, 2, 3],
    "value": [10.5, 20.0, 30.1],
    "category": ["A", "B", "C"]
})

# Example 2: Invalid data
invalid_df = pd.DataFrame({
    "id": [1, 1, 4], # Duplicate ID
    "value": [5.0, 105.0, 25.0], # Out of range value
    "category": ["A", "D", "B"], # Invalid category
    "extra_col": [1, 2, 3] # Extra column due to strict=True
})

renderer = PandasReportRenderer()

# Generate report for valid data
html_success_report = renderer.to_html(data=valid_df, schema=schema)
# 'html_success_report' will indicate successful validation.

# Generate report for invalid data
html_error_report = ""
try:
    schema.validate(invalid_df, lazy=True) # lazy=True collects all errors
except SchemaErrors as err:
    html_error_report = renderer.to_html(data=invalid_df, schema=schema, error=err)
# 'html_error_report' will contain detailed information about validation failures.
```

The `PandasReportRenderer` generates an HTML page that includes:
*   A summary of the validation outcome (success or failure).
*   Metadata about the schema and data shape.
*   A preview of the input DataFrame.
*   Detailed tables for schema-level errors (e.g., wrong column names, incorrect dtypes) and data-level errors (e.g., invalid values, out-of-range data), including failure cases and affected rows.

This detailed reporting is invaluable for debugging data quality issues and providing clear feedback on data pipeline health.
<!--
key: summary_data_quality_and_validation_d17151a1-0083-4898-9ba8-792ac9aed61e
type: summary_end

-->
<!--
code_unit: flytekitplugins.whylogs.WhylogsConstraintsRenderer
code_unit_type: class
help_text: ''
key: example_20eba6b3-bddd-474c-a2bd-700acd14cc84
type: example

-->
<!--
code_unit: flytekitplugins.whylogs.WhylogsSummaryDriftRenderer
code_unit_type: class
help_text: ''
key: example_9beac597-4602-4a12-ba32-d74b4ce11188
type: example

-->
<!--
code_unit: flytekitplugins.whylogs.WhylogsDatasetProfileTransformer
code_unit_type: class
help_text: ''
key: example_313e78b5-f3fd-4b31-aacf-9e77f6f2b3c8
type: example

-->
<!--
code_unit: flytekitplugins.whylogs.schema.WhylogsDatasetProfileTransformer
code_unit_type: class
help_text: ''
key: example_2b600077-69f7-4ee5-b914-1aec420a0979
type: example

-->
<!--
code_unit: flytekitplugins.whylogs.renderer.WhylogsSummaryDriftRenderer
code_unit_type: class
help_text: ''
key: example_7d92ad53-8782-460c-adf6-340e13732665
type: example

-->
<!--
code_unit: flytekitplugins.whylogs.renderer.WhylogsConstraintsRenderer
code_unit_type: class
help_text: ''
key: example_15a417e0-9528-4196-9c56-d3f5933c6906
type: example

-->
<!--
code_unit: flytekitplugins.pandera.pandas_renderer.PandasReportRenderer
code_unit_type: class
help_text: ''
key: example_2c4255fd-03b9-4b00-8619-926c02ea4911
type: example

-->
<!--
code_unit: flytekitplugins.pandera.pandas_renderer.PandasReport
code_unit_type: class
help_text: ''
key: example_92bab107-2408-4885-9870-7d0fdc7001dc
type: example

-->