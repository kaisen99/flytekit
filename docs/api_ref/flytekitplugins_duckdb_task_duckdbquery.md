# DuckDBQuery

This class facilitates the execution of DuckDB queries within a workflow. It allows users to define and execute SQL queries against various data sources, supporting parameterized queries and different DuckDB providers. The class integrates with structured datasets and handles connection management, query execution, and result retrieval.

## Constructors
def DuckDBQuery(name: str, query: Optional[[Union](flytekit_models_literals_union)[str, List[str]]] = None, inputs: Optional[Dict[str, [Union](flytekit_models_literals_union)[[StructuredDataset](flytekit_types_structured_structured_dataset_structureddataset), list]]] = None, provider: [Union](flytekit_models_literals_union)[[DuckDBProvider](flytekitplugins_duckdb_task_duckdbprovider), Callable] = DuckDBProvider.LOCAL)
-  This method initializes the DuckDBQuery.

Note that the provider can be one of the default providers listed in DuckDBProvider or a custom callable like the following:

def custom_connect_motherduck(token: str):
    return duckdb.connect(&quot;md:&quot;, config={&quot;motherduck_token&quot;: token, &quot;another_config&quot;: &quot;hello&quot;})

DuckDBQuery(..., provider=custom_connect_motherduck)

Also note that a query can be provided at runtime if query=None is provided.

duckdb_query = DuckDBQuery(
    name=&quot;my_duckdb_query&quot;,
    inputs=kwtypes(query=str)
)

@workflow
def wf(user_query: str) - &gt; pd.DataFrame:
    return duckdb_query(query=user_query)

Args:
    name: Name of the task
    query: DuckDB query to execute
    inputs: The query parameters to be used while executing the query
    provider: DuckDB provider
- **Parameters**

  - **name**: str
    - Name of the task
  - **query**: Optional[[Union](flytekit_models_literals_union)[str, List[str]]]
    - DuckDB query to execute
  - **inputs**: Optional[Dict[str, [Union](flytekit_models_literals_union)[[StructuredDataset](flytekit_types_structured_structured_dataset_structureddataset), list]]]
    - The query parameters to be used while executing the query
  - **provider**: [Union](flytekit_models_literals_union)[[DuckDBProvider](flytekitplugins_duckdb_task_duckdbprovider), Callable]
    - DuckDB provider



## Methods
```@classmethod
def execute()
```
-  This method executes the DuckDB query and returns the result as a StructuredDataset.

- **Return Value**:
**[StructuredDataset](flytekit_types_structured_structured_dataset_structureddataset)**
  - The result of the DuckDB query as a StructuredDataset.
