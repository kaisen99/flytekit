# Config

This class serves as the central configuration object, aggregating various sub-configurations. It holds settings necessary for interacting with a Flyte backend, serialization, and task runtime. Key features include automatic configuration loading and methods for creating configurations tailored to specific environments, such as sandboxes and endpoints.

## Attributes

- **platform**: [PlatformConfig](flytekit_configuration_platformconfig) = PlatformConfig()
  - Platform configuration for the Flyte backend.

- **secrets**: [SecretsConfig](flytekit_configuration_secretsconfig) = SecretsConfig()
  - Secrets configuration for accessing Flyte resources.

- **stats**: [StatsConfig](flytekit_configuration_statsconfig) = StatsConfig()
  - Statistics configuration for monitoring Flyte tasks.

- **data_config**: [DataConfig](flytekit_configuration_dataconfig) = DataConfig()
  - Data configuration for handling data inputs and outputs.

- **local_sandbox_path**: str = tempfile.mkdtemp(prefix=&quot;flyte&quot;)
  - Path to the local sandbox directory for Flyte operations.

## Constructors
def Config(entrypoint_settings: [EntrypointSettings](flytekit_configuration_entrypointsettings))
-  This the parent configuration object and holds all the underlying configuration object types. An instance of
    this object holds all the config necessary to

    1. Interactive session with Flyte backend
    2. Some parts are required for Serialization, for example Platform Config is not required
    3. Runtime of a task
- **Parameters**

  - **entrypoint_settings**: [EntrypointSettings](flytekit_configuration_entrypointsettings)
    - EntrypointSettings object for use with Spark tasks. If supplied, this will be
          used when serializing Spark tasks, which need to know the path to the flytekit entrypoint.py file,
          inside the container.



## Methods
@classmethod
def with_params(platform: [PlatformConfig](flytekit_configuration_platformconfig) = None, secrets: [SecretsConfig](flytekit_configuration_secretsconfig) = None, stats: [StatsConfig](flytekit_configuration_statsconfig) = None, data_config: [DataConfig](flytekit_configuration_dataconfig) = None, local_sandbox_path: str = None) - > [Config](flytekit_configuration_config)
-  This method is used to create a new Config object with updated parameters. It allows for partial updates by accepting optional arguments for platform, secrets, stats, data_config, and local_sandbox_path. If an argument is not provided, the existing value from the current Config object is retained.
- **Parameters**

  - **platform**: [PlatformConfig](flytekit_configuration_platformconfig)
    - Optional. The new PlatformConfig object to use.
  - **secrets**: [SecretsConfig](flytekit_configuration_secretsconfig)
    - Optional. The new SecretsConfig object to use.
  - **stats**: [StatsConfig](flytekit_configuration_statsconfig)
    - Optional. The new StatsConfig object to use.
  - **data_config**: [DataConfig](flytekit_configuration_dataconfig)
    - Optional. The new DataConfig object to use.
  - **local_sandbox_path**: str
    - Optional. The new local sandbox path to use.

- **Return Value**:
**[Config](flytekit_configuration_config)**
  - A new Config object with the updated parameters.
@classmethod
def auto(config_file: typing.Union[str, [ConfigFile](flytekit_configuration_file_configfile), None] = None) - > [Config](flytekit_configuration_config)
-  Automatically constructs the Config Object. The order of precedence is as follows: 1. first try to find any env vars that match the config vars specified in the FLYTE_CONFIG format. 2. If not found in environment then values ar read from the config file 3. If not found in the file, then the default values are used.
- **Parameters**

  - **config_file**: typing.Union[str, [ConfigFile](flytekit_configuration_file_configfile), None]
    - file path to read the config from, if not specified default locations are searched

- **Return Value**:
**[Config](flytekit_configuration_config)**
  - A Config object constructed automatically.
```@classmethod
def for_sandbox()
```
-  Constructs a new Config object specifically to connect to the Flyte Sandbox. If you are using a hosted Sandbox like environment, then you may need to use port-forward or ingress urls.

- **Return Value**:
**[Config](flytekit_configuration_config)**
  - A Config object configured for the sandbox environment.
@classmethod
def for_endpoint(endpoint: str, insecure: bool = False, data_config: typing.Optional[[DataConfig](flytekit_configuration_dataconfig)] = None, config_file: typing.Union[str, [ConfigFile](flytekit_configuration_file_configfile)]) - > [Config](flytekit_configuration_config)
-  Creates an automatic config for the given endpoint and uses the config_file or environment variable for default. Refer to `Config.auto()` to understand the default bootstrap behavior. data_config can be used to configure how data is downloaded or uploaded to a specific Blob storage like S3 / GCS etc. But, for permissions to a specific backend just use Cloud providers recommendation. If using fsspec, then refer to fsspec documentation.
- **Parameters**

  - **endpoint**: str
    - Endpoint where Flyte admin is available
  - **insecure**: bool
    - if the connection should be insecure, default is secure (SSL ON)
  - **data_config**: typing.Optional[[DataConfig](flytekit_configuration_dataconfig)]
    - Data config, if using specialized connection params like minio etc
  - **config_file**: typing.Union[str, [ConfigFile](flytekit_configuration_file_configfile)]
    - Optional config file in the flytekit config format.

- **Return Value**:
**[Config](flytekit_configuration_config)**
  - A Config object configured for the specified endpoint.
