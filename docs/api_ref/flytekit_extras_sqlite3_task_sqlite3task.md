# SQLite3Task

This class executes client-side SQLite3 queries, optionally returning a FlyteSchema object. It leverages a pre-built container task, utilizing a predefined image for execution. The class integrates with SQLTask and PythonCustomizedContainerTask, enabling the execution of SQL queries against SQLite3 databases.

## Constructors
def SQLite3Task(name: str, query_template: str, inputs: typing.Optional[typing.Dict[str, typing.Type]] = None, task_config: typing.Optional[[SQLite3Config](flytekit_extras_sqlite3_task_sqlite3config)] = None, output_schema_type: typing.Optional[typing.Type["FlyteSchema"]] = None, container_image: typing.Optional[str] = None)
-  Run client side SQLite3 queries that optionally return a FlyteSchema object.
- **Parameters**

  - **name**: str
    - The name of the task.
  - **query_template**: str
    - The SQL query template to execute.
  - **inputs**: typing.Optional[typing.Dict[str, typing.Type]]
    - A dictionary of input parameter names and their types.
  - **task_config**: typing.Optional[[SQLite3Config](flytekit_extras_sqlite3_task_sqlite3config)]
    - Configuration for the SQLite3 task, including the database URI.
  - **output_schema_type**: typing.Optional[typing.Type["FlyteSchema"]]
    - The FlyteSchema type for the output.
  - **container_image**: typing.Optional[str]
    - The container image to use for the task.



## Methods
```@classmethod
def output_columns()
```
-  Returns the output columns of the task.

- **Return Value**:
**Optional[List[str]]**
  - A list of output column names, or None if not available.
@classmethod
def get_custom(settings: [SerializationSettings](flytekit_configuration_serializationsettings)) - > Dict[str, Any]
-  Returns custom settings for the task serialization.
- **Parameters**

  - **settings**: [SerializationSettings](flytekit_configuration_serializationsettings)
    - Serialization settings.

- **Return Value**:
**Dict[str, Any]**
  - A dictionary containing custom settings like query_template, uri, and compressed.
