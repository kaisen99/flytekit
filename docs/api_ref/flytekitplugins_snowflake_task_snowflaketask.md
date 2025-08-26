# SnowflakeTask

This class represents a Snowflake Task, designed for executing SQL queries against Snowflake databases. It leverages Flyte&#x27;s templating format for query construction and supports specifying input and output schemas. The class extends SQLTask and utilizes a Snowflake configuration object for database connection details.

## Constructors
def SnowflakeTask(name: string, query_template: string, task_config: [SnowflakeConfig](flytekitplugins_snowflake_task_snowflakeconfig), inputs: Optional[Dict[str, Type]] = None, output_schema_type: Optional[Type[[StructuredDataset](flytekit_types_structured_structured_dataset_structureddataset)]] = None, kwargs: **kwargs)
-  To be used to query Snowflake databases.
- **Parameters**

  - **name**: string
    - Name of this task, should be unique in the project
  - **query_template**: string
    - The actual query to run. We use Flyte&#x27;s Golang templating format for Query templating.
          Refer to the templating documentation
  - **task_config**: [SnowflakeConfig](flytekitplugins_snowflake_task_snowflakeconfig)
    - SnowflakeConfig object
  - **inputs**: Optional[Dict[str, Type]]
    - Name and type of inputs specified as an ordered dictionary
  - **output_schema_type**: Optional[Type[[StructuredDataset](flytekit_types_structured_structured_dataset_structureddataset)]]
    - If some data is produced by this query, then you can specify the output schema type
  - **kwargs**: **kwargs
    - All other args required by Parent type - SQLTask



## Methods
@classmethod
def get_config(settings: [SerializationSettings](flytekit_configuration_serializationsettings)) - > Dict[str, str]
-  Returns the configuration for the Snowflake task.
- **Parameters**

  - **settings**: [SerializationSettings](flytekit_configuration_serializationsettings)
    - Serialization settings.

- **Return Value**:
**Dict[str, str]**
  - A dictionary containing the Snowflake connection details.
@classmethod
def get_sql(settings: [SerializationSettings](flytekit_configuration_serializationsettings)) - > Optional[_task_model.Sql]
-  Returns the SQL statement for the Snowflake task.
- **Parameters**

  - **settings**: [SerializationSettings](flytekit_configuration_serializationsettings)
    - Serialization settings.

- **Return Value**:
**Optional[_task_model.Sql]**
  - An object representing the SQL statement, or None if no SQL is defined.
