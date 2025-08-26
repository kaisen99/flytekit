# LocalConfig

This class manages configurations specific to local execution environments. It provides settings for caching behavior, including enabling or disabling the cache and controlling cache overwrites. The class utilizes a configuration file to read and set these local configurations.

## Attributes

- **cache_enabled**: bool = True
  - Whether to cache results locally.

- **cache_overwrite**: bool = False
  - Whether to overwrite existing cache entries.



## Methods
@classmethod
def auto(config_file: typing.Union[str, [ConfigFile](flytekit_configuration_file_configfile)] = None) - > [LocalConfig](flytekit_configuration_localconfig)
-  Constructs a LocalConfig object from a configuration file.

Args:
    config_file: The path to the configuration file or a ConfigFile object.

Returns:
    A LocalConfig object with settings loaded from the configuration file.
- **Parameters**

  - **config_file**: typing.Union[str, [ConfigFile](flytekit_configuration_file_configfile)]
    - The path to the configuration file or a ConfigFile object.

- **Return Value**:
**[LocalConfig](flytekit_configuration_localconfig)**
  - A LocalConfig object.
