# DataConfig

This class encapsulates data storage configuration settings for various cloud providers. It provides configurations for S3, GCS, Azure Blob Storage, and generic persistence. The class utilizes auto-configuration methods to load settings from a configuration file.

## Attributes

- **s3**: [S3Config](flytekit_configuration_s3config) = S3Config()
  - Any data storage specific configuration. Please do not use this to store secrets, in S3 case, as it is used in
    Flyte sandbox environment we store the access key id and secret.
    All DataPersistence plugins are passed all DataConfig and the plugin should correctly use the right config

- **gcs**: [GCSConfig](flytekit_configuration_gcsconfig) = GCSConfig()
  - Any data storage specific configuration. Please do not use this to store secrets, in S3 case, as it is used in
    Flyte sandbox environment we store the access key id and secret.
    All DataPersistence plugins are passed all DataConfig and the plugin should correctly use the right config

- **azure**: [AzureBlobStorageConfig](flytekit_configuration_azureblobstorageconfig) = AzureBlobStorageConfig()
  - Any data storage specific configuration. Please do not use this to store secrets, in S3 case, as it is used in
    Flyte sandbox environment we store the access key id and secret.
    All DataPersistence plugins are passed all DataConfig and the plugin should correctly use the right config

- **generic**: [GenericPersistenceConfig](flytekit_configuration_genericpersistenceconfig) = GenericPersistenceConfig()
  - Any data storage specific configuration. Please do not use this to store secrets, in S3 case, as it is used in
    Flyte sandbox environment we store the access key id and secret.
    All DataPersistence plugins are passed all DataConfig and the plugin should correctly use the right config

## Constructors
def DataConfig(s3: [S3Config](flytekit_configuration_s3config) = S3Config(), gcs: [GCSConfig](flytekit_configuration_gcsconfig) = GCSConfig(), azure: [AzureBlobStorageConfig](flytekit_configuration_azureblobstorageconfig) = AzureBlobStorageConfig(), generic: [GenericPersistenceConfig](flytekit_configuration_genericpersistenceconfig) = GenericPersistenceConfig())
-  Any data storage specific configuration. Please do not use this to store secrets, in S3 case, as it is used in
    Flyte sandbox environment we store the access key id and secret.
    All DataPersistence plugins are passed all DataConfig and the plugin should correctly use the right config
- **Parameters**

  - **s3**: [S3Config](flytekit_configuration_s3config)
    - S3 configuration
  - **gcs**: [GCSConfig](flytekit_configuration_gcsconfig)
    - GCS configuration
  - **azure**: [AzureBlobStorageConfig](flytekit_configuration_azureblobstorageconfig)
    - Azure blob storage configuration
  - **generic**: [GenericPersistenceConfig](flytekit_configuration_genericpersistenceconfig)
    - Generic persistence configuration



## Methods
@classmethod
def auto(config_file: typing.Union[str, [ConfigFile](flytekit_configuration_file_configfile)] = None) - > [DataConfig](flytekit_configuration_dataconfig)
-  auto
- **Parameters**

  - **config_file**: typing.Union[str, [ConfigFile](flytekit_configuration_file_configfile)]
    - auto

- **Return Value**:
**[DataConfig](flytekit_configuration_dataconfig)**
  - auto
