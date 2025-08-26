# AzureBlobStorageConfig

This class encapsulates the configuration settings required to interact with Azure Blob Storage. It provides a structured way to manage account credentials and authentication parameters. The class supports automatic configuration loading from a configuration file.

## Attributes

- **account_name**: Optional[str] = None
  - The name of the storage account.

- **account_key**: Optional[str] = None
  - The account key for the storage account.

- **tenant_id**: Optional[str] = None
  - The tenant ID for Azure authentication.

- **client_id**: Optional[str] = None
  - The client ID for Azure authentication.

- **client_secret**: Optional[str] = None
  - The client secret for Azure authentication.



## Methods
@classmethod
def auto(config_file: typing.Union[str, [ConfigFile](flytekit_configuration_file_configfile)] = None) - > [AzureBlobStorageConfig](flytekit_configuration_azureblobstorageconfig)
-  Automatically configures Azure Blob Storage settings from a configuration file.
- **Parameters**

  - **config_file**: typing.Union[str, [ConfigFile](flytekit_configuration_file_configfile)]
    - The path to the configuration file or a ConfigFile object. If not provided, it defaults to the system&#x27;s default configuration file.

- **Return Value**:
**[AzureBlobStorageConfig](flytekit_configuration_azureblobstorageconfig)**
  - An instance of AzureBlobStorageConfig with settings populated from the configuration file.
