# YamlConfigEntry

This class represents a configuration entry, designed to read and manage settings from a configuration file. It uses a dot-delimited string to match flytectl arguments. The class supports reading configuration values from a file and optionally transforming them using a provided function.

## Attributes

- **switch**: string
  - dot-delimited string that should match flytectl args. Leaving it as dot-delimited instead of a list of strings because it&#x27;s easier to maintain alignment with flytectl.

- **config_value_type**: typing.Type = str
  - Expected type of the value



## Methods
@classmethod
def read_from_file(cfg: [ConfigFile](flytekit_configuration_file_configfile), transform: typing.Optional[typing.Callable]) - > typing.Optional[typing.Any]
-  Reads the config value from the given config file. Optionally applies a transformation to the value before returning it.
- **Parameters**

  - **cfg**: [ConfigFile](flytekit_configuration_file_configfile)
    - The configuration file object to read from.
  - **transform**: typing.Optional[typing.Callable]
    - An optional callable function to transform the read value.

- **Return Value**:
**typing.Optional[typing.Any]**
  - The transformed config value, or None if the value doesn&#x27;t exist or an error occurs.
