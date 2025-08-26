# DatabricksV2

This class configures and executes tasks natively on the Databricks platform using Spark. It leverages Databricks Job API version 2.1, supporting configurations for distributed Spark execution. The class requires a Databricks job configuration and the Databricks instance domain name for deployment.

## Attributes

- **databricks_conf**: Optional[Dict[str, [Union](flytekit_models_literals_union)[str, dict]]] = None
  - Databricks job configuration compliant with API version 2.1, supporting 2.0 use cases. For the configuration structure, visit here.https://docs.databricks.com/dev-tools/api/2.0/jobs.html#request-structure For updates in API 2.1, refer to: https://docs.databricks.com/en/workflows/jobs/jobs-api-updates.html

- **databricks_instance**: Optional[str] = None
  - Domain name of your deployment. Use the form &lt; account &gt;.cloud.databricks.com.

## Constructors
def DatabricksV2(databricks_conf: Optional[Dict[str, [Union](flytekit_models_literals_union)[str, dict]]] = None, databricks_instance: Optional[str] = None)
-  Use this to configure a Databricks task. Task&#x27;s marked with this will automatically execute natively onto databricks platform as a distributed execution of spark
- **Parameters**

  - **databricks_conf**: Optional[Dict[str, [Union](flytekit_models_literals_union)[str, dict]]]
    - Databricks job configuration compliant with API version 2.1, supporting 2.0 use cases. For the configuration structure, visit here.https://docs.databricks.com/dev-tools/api/2.0/jobs.html#request-structure For updates in API 2.1, refer to: https://docs.databricks.com/en/workflows/jobs/jobs-api-updates.html
  - **databricks_instance**: Optional[str]
    - Domain name of your deployment. Use the form &lt; account &gt;.cloud.databricks.com.



