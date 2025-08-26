# RunLevelParams

This class is designed to encapsulate parameters essential for executing workflows, tasks, and launch plans within a system. It provides a structured way to manage settings such as project and domain, along with options for image configuration, service accounts, and execution behavior. The class supports features like environment variables, tags, and labels, and it integrates with remote execution capabilities.

## Attributes

- **project**: str
  - Project to run the workflow in.

- **domain**: str
  - Domain to run the workflow in.

- **destination_dir**: str = &quot;.&quot;
  - Directory inside the image where the tar file containing the code will be copied to

- **copy_all**: bool = False
  - [Deprecated, see --copy] Copy all files in the source root directory to
            the destination directory. You can specify --copy all instead

- **copy**: typing.Optional[[CopyFileDetection](flytekit_constants_copyfiledetection)] = &quot;auto&quot;
  - Specifies how to detect which files to copy into image.
            &#x27;all&#x27; will behave as the deprecated copy-all flag, &#x27;auto&#x27; copies only loaded Python modules

- **image_config**: [ImageConfig](flytekit_configuration_imageconfig) = [DefaultImages.default_image()]
  - Image used to register and run.

- **service_account**: str = &quot;&quot;
  - Service account used when executing this workflow

- **wait_execution**: bool = False
  - Whether to wait for the execution to finish

- **poll_interval**: int = None
  - Poll interval in seconds to check the status of the execution

- **dump_snippet**: bool = False
  - Whether to dump a code snippet instructing how to load the workflow execution using flyteremote

- **overwrite_cache**: bool = False
  - Whether to overwrite the cache if it already exists

- **envvars**: typing.Dict[str, str]
  - Environment variables to set in the container, of the format `ENV_NAME=ENV_VALUE`

- **tags**: typing.List[str]
  - Tags to set for the execution

- **name**: str
  - Name to assign to this execution

- **labels**: typing.Dict[str, str]
  - Labels to be attached to the execution of the format `label_key=label_value`.

- **annotations**: typing.Dict[str, str]
  - Annotations to be attached to the execution of the format `key=value`.

- **raw_output_data_prefix**: str
  - File Path prefix to store raw output data.
            Examples are file://, s3://, gs:// etc as supported by fsspec.
            If not specified, raw data will be stored in default configured location in remote of locally
            to temp file system.
Note, this is not metadata, but only the raw data location used to store Flytefile, Flytedirectory, Structuredataset,
dataframes

- **max_parallelism**: int
  - Number of nodes of a workflow that can be executed in parallel. If not specified,
            project/domain defaults are used. If 0 then it is unlimited.

- **disable_notifications**: bool = False
  - Should notifications be disabled for this execution.

- **remote**: bool = False
  - Whether to register and run the workflow on a Flyte deployment

- **limit**: int = 50
  - Use this to limit number of entities to fetch

- **cluster_pool**: str = &quot;&quot;
  - Assign newly created execution to a given cluster pool

- **execution_cluster_label**: str = &quot;&quot;
  - Assign newly created execution to a given execution cluster label

- **computed_params**: [RunLevelComputedParams](flytekit_clis_sdk_in_container_run_runlevelcomputedparams) = RunLevelComputedParams()
  - Computed parameters for the run level.

## Constructors
def RunLevelParams(project: str = None, domain: str = None, destination_dir: str = &quot;.&quot;, copy_all: bool = False, copy: typing.Optional[[CopyFileDetection](flytekit_constants_copyfiledetection)] = &quot;auto&quot;, image_config: [ImageConfig](flytekit_configuration_imageconfig) = [DefaultImages.default_image()], service_account: str = &quot;&quot;, wait_execution: bool = False, poll_interval: int = None, dump_snippet: bool = False, overwrite_cache: bool = False, envvars: typing.Dict[str, str] = None, tags: typing.List[str] = None, name: str = None, labels: typing.Dict[str, str] = None, annotations: typing.Dict[str, str] = None, raw_output_data_prefix: str = None, max_parallelism: int = None, disable_notifications: bool = False, remote: bool = False, limit: int = 50, cluster_pool: str = &quot;&quot;, execution_cluster_label: str = &quot;&quot;)
-  This class is used to store the parameters that are used to run a workflow / task / launchplan.
- **Parameters**

  - **project**: str
    - Project to run the workflow/task/launchplan in.
  - **domain**: str
    - Domain to run the workflow/task/launchplan in.
  - **destination_dir**: str
    - Directory inside the image where the tar file containing the code will be copied to
  - **copy_all**: bool
    - Deprecated, see --copy Copy all files in the source root directory to the destination directory. You can specify --copy all instead
  - **copy**: typing.Optional[[CopyFileDetection](flytekit_constants_copyfiledetection)]
    - Specifies how to detect which files to copy into image. &#x27;all&#x27; will behave as the deprecated copy-all flag, &#x27;auto&#x27; copies only loaded Python modules
  - **image_config**: [ImageConfig](flytekit_configuration_imageconfig)
    - Image used to register and run.
  - **service_account**: str
    - Service account used when executing this workflow
  - **wait_execution**: bool
    - Whether to wait for the execution to finish
  - **poll_interval**: int
    - Poll interval in seconds to check the status of the execution
  - **dump_snippet**: bool
    - Whether to dump a code snippet instructing how to load the workflow execution using flyteremote
  - **overwrite_cache**: bool
    - Whether to overwrite the cache if it already exists
  - **envvars**: typing.Dict[str, str]
    - Environment variables to set in the container, of the format `ENV_NAME=ENV_VALUE`
  - **tags**: typing.List[str]
    - Tags to set for the execution
  - **name**: str
    - Name to assign to this execution
  - **labels**: typing.Dict[str, str]
    - Labels to be attached to the execution of the format `label_key=label_value`.
  - **annotations**: typing.Dict[str, str]
    - Annotations to be attached to the execution of the format `key=value`.
  - **raw_output_data_prefix**: str
    - File Path prefix to store raw output data. Examples are file://, s3://, gs:// etc as supported by fsspec. If not specified, raw data will be stored in default configured location in remote of locally to temp file system. Note, this is not metadata, but only the raw data location used to store Flytefile, Flytedirectory, Structuredataset, dataframes
  - **max_parallelism**: int
    - Number of nodes of a workflow that can be executed in parallel. If not specified, project/domain defaults are used. If 0 then it is unlimited.
  - **disable_notifications**: bool
    - Should notifications be disabled for this execution.
  - **remote**: bool
    - Whether to register and run the workflow on a Flyte deployment
  - **limit**: int
    - Use this to limit number of entities to fetch
  - **cluster_pool**: str
    - Assign newly created execution to a given cluster pool
  - **execution_cluster_label**: str
    - Assign newly created execution to a given execution cluster label



## Methods
```@classmethod
def remote_instance()
```
-  Returns a FlyteRemote instance. If the instance does not exist, it will be created.

- **Return Value**:
**[FlyteRemote](flytekit_remote_remote_flyteremote)**
  - A FlyteRemote instance.
```@classmethod
def is_remote()
```
-  Returns True if the execution is remote, False otherwise.

- **Return Value**:
**bool**
  - True if the execution is remote, False otherwise.
@classmethod
def from_dict(d: typing.Dict[str, typing.Any]) - > [RunLevelParams](flytekit_clis_sdk_in_container_run_runlevelparams)
-  Creates a RunLevelParams instance from a dictionary.
- **Parameters**

  - **d**: typing.Dict[str, typing.Any]
    - A dictionary containing the parameters for the RunLevelParams instance.

- **Return Value**:
**[RunLevelParams](flytekit_clis_sdk_in_container_run_runlevelparams)**
  - A RunLevelParams instance.
```@classmethod
def options()
```
-  Return the set of base parameters added to every pyflyte run workflow subcommand.

- **Return Value**:
**typing.List[click.Option]**
  - A list of click.Option objects.
