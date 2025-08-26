# ConfigFile

This class is designed to load and provide access to configuration settings from either YAML or legacy configuration files. It supports reading configurations from different file formats based on the file extension. The class provides methods to retrieve configuration values, abstracting the underlying file format.

## Attributes

- **_location**: str
  - Load the config from this location

- **_legacy_config**: configparser.ConfigParser
  - Load the config from this location

- **_yaml_config**: typing.Dict[str, typing.Any]
  - Load the config from this location

## Constructors
def ConfigFile(location: str)
-  Load the config from this location
- **Parameters**

  - **location**: str
    - The path to the configuration file.

def ConfigFile(location: str)
-  Load the config from this location
- **Parameters**

  - **location**: str
    - Load the config from this location



## Methods
@classmethod
def get(c: typing.Union[[LegacyConfigEntry](flytekit_configuration_file_legacyconfigentry), [YamlConfigEntry](flytekit_configuration_file_yamlconfigentry)]) - > typing.Any
-  Retrieves a configuration value from either legacy or YAML format.
- **Parameters**

  - **c**: typing.Union[[LegacyConfigEntry](flytekit_configuration_file_legacyconfigentry), [YamlConfigEntry](flytekit_configuration_file_yamlconfigentry)]
    - The configuration entry (legacy or YAML) to retrieve.

- **Return Value**:
**typing.Any**
  - The retrieved configuration value.
```@classmethod
def legacy_config()
```
-  Provides access to the legacy configuration object.

- **Return Value**:
**configparser.ConfigParser**
  - The legacy configuration object.
```@classmethod
def yaml_config()
```
-  Provides access to the YAML configuration object.

- **Return Value**:
**typing.Dict[str, typing.Any]**
  - The YAML configuration object.
