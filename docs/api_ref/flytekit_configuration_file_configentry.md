# ConfigEntry

This class manages configuration entries, supporting multiple representations like legacy (INI) and YAML formats. It facilitates reading configuration values from environment variables, legacy config files, and YAML files. The class employs a transformation mechanism for data type conversion and prioritizes reading configurations from environment variables, then legacy files, and finally YAML files.

## Attributes

- **legacy**: [LegacyConfigEntry](flytekit_configuration_file_legacyconfigentry)
  - Legacy means the INI style config files.

- **yaml_entry**: typing.Optional[[YamlConfigEntry](flytekit_configuration_file_yamlconfigentry)] = None
  - YAML support is for the flytectl config file, which is there by default when flytectl starts a sandbox

- **transform**: typing.Optional[typing.Callable[[str], typing.Any]] = None
  - A transformation function to apply to the config value.

## Constructors
```def ConfigEntry()
```
-  A top level Config entry holder, that holds multiple different representations of the config. Legacy means the INI style config files. YAML support is for the flytectl config file, which is there by default when flytectl starts a sandbox



## Methods
@classmethod
def read(cfg: Optional[[ConfigFile](flytekit_configuration_file_configfile)] = None) - > Optional[Any]
-  Reads the config entry from various sources in a specific order: environment variable, legacy config file, or YAML file. It prioritizes environment variables, then falls back to the legacy config file if available, and finally to the YAML file if both previous sources are missing. The constructor for ConfigFile does not allow specification of both INI and YAML style formats.
- **Parameters**

  - **cfg**: Optional[[ConfigFile](flytekit_configuration_file_configfile)]
    - An optional ConfigFile object which may contain legacy (INI) or YAML configurations.

- **Return Value**:
**Optional[Any]**
  - The value read from the config sources, or None if not found.
