
<!--
help_text: ''
key: summary_flytekit_configuration_files_842ba87f-4313-4eca-abf3-9b1ce9ddb4c4
modules:
- flytekit.configuration.file
questions_to_answer: []
type: summary

-->
Flytekit uses configuration files to manage various settings, from backend communication details to local execution parameters. This system provides flexibility by allowing settings to be defined in multiple locations with a clear precedence order.

Flytekit supports two primary configuration file formats:
*   **INI-style files**: A legacy format, typically with a `.ini` or `.config` extension.
*   **YAML files**: Aligned with `flytectl` configurations, typically with a `.yaml` or `.yml` extension.

Configuration values are resolved in the following order of precedence:
1.  **Environment Variables**: Values set as environment variables take the highest precedence.
2.  **Legacy INI File**: If an INI-style configuration file is loaded, values from it are considered next.
3.  **YAML File**: If a YAML configuration file is loaded, values from it are considered last.

### Loading Configuration Files

The `ConfigFile` class is central to loading configuration settings from a specified file path.

To load a configuration file:

```python
from flytekit.configuration.file import ConfigFile

# Load an INI-style configuration file
ini_config = ConfigFile("/path/to/my_config.ini")

# Load a YAML-style configuration file
yaml_config = ConfigFile("/path/to/my_config.yaml")
```

The `ConfigFile` constructor automatically determines the file type (INI or YAML) based on its extension (`.yaml` or `.yml`). It then parses the file into either a `configparser.ConfigParser` object (for INI) or a Python dictionary (for YAML).

Access the parsed contents using the `legacy_config` and `yaml_config` properties:

```python
# For an INI file
if ini_config.legacy_config:
    print(ini_config.legacy_config.sections())

# For a YAML file
if yaml_config.yaml_config:
    print(yaml_config.yaml_config.keys())
```

A `ConfigFile` instance can only load one type of configuration file at a given `location`. Attempting to load a YAML file will result in `legacy_config` being `None`, and vice-versa.

### Defining Configuration Entries

Configuration entries define how a specific setting is identified and read from different sources. The `ConfigEntry` class provides a unified interface for this.

A `ConfigEntry` encapsulates references to both legacy INI and YAML representations of a setting, along with optional type transformation logic.

```python
from flytekit.configuration.file import ConfigEntry, LegacyConfigEntry, YamlConfigEntry
import typing

# Define a legacy INI entry
# Corresponds to [section_name] option_name = value
# And environment variable FLYTE_SECTION_NAME_OPTION_NAME
legacy_entry = LegacyConfigEntry(section="section_name", option="option_name", type_=str)

# Define a YAML entry
# Corresponds to a path like root.sub_key.setting_name in the YAML file
yaml_entry = YamlConfigEntry(switch="root.sub_key.setting_name", config_value_type=int)

# Combine them into a ConfigEntry
my_setting = ConfigEntry(legacy=legacy_entry, yaml_entry=yaml_entry)
```

#### Legacy INI Configuration Entries

The `LegacyConfigEntry` class defines settings found in INI-style files.

*   `section`: The section header in the INI file (e.g., `[section_name]`).
*   `option`: The key within that section (e.g., `option_name = value`).
*   `type_`: The expected Python type of the value (e.g., `str`, `int`, `bool`, `list`).

`LegacyConfigEntry` also determines the corresponding environment variable name. The format is `FLYTE_{SECTION_UPPERCASE}_{OPTION_UPPERCASE}`. For example, a `LegacyConfigEntry(section="my_app", option="timeout_seconds")` corresponds to the environment variable `FLYTE_MY_APP_TIMEOUT_SECONDS`.

#### YAML Configuration Entries

The `YamlConfigEntry` class defines settings found in YAML files.

*   `switch`: A dot-delimited string representing the path to the value within the YAML structure (e.g., `"flyte.platform.url"`). This design aligns with `flytectl` command-line arguments.
*   `config_value_type`: The expected Python type of the value.

### Reading Configuration Values

The `read` method of a `ConfigEntry` instance retrieves the configuration value, applying the defined precedence order.

```python
from flytekit.configuration.file import ConfigEntry, LegacyConfigEntry, YamlConfigEntry, ConfigFile
import os

# Example: Define a configuration entry for a service URL
service_url_legacy = LegacyConfigEntry(section="service", option="url", type_=str)
service_url_yaml = YamlConfigEntry(switch="service.url", config_value_type=str)
service_url_config = ConfigEntry(legacy=service_url_legacy, yaml_entry=service_url_yaml)

# --- Scenario 1: Value from Environment Variable ---
os.environ["FLYTE_SERVICE_URL"] = "http://env-service.com"
value = service_url_config.read()
print(f"Value from env: {value}") # Output: Value from env: http://env-service.com
del os.environ["FLYTE_SERVICE_URL"] # Clean up

# --- Scenario 2: Value from Legacy INI File ---
# Create a dummy INI file
with open("config.ini", "w") as f:
    f.write("[service]\nurl = http://ini-service.com\n")
ini_cfg_file = ConfigFile("config.ini")
value = service_url_config.read(cfg=ini_cfg_file)
print(f"Value from INI: {value}") # Output: Value from INI: http://ini-service.com
os.remove("config.ini")

# --- Scenario 3: Value from YAML File ---
# Create a dummy YAML file
with open("config.yaml", "w") as f:
    f.write("service:\n  url: http://yaml-service.com\n")
yaml_cfg_file = ConfigFile("config.yaml")
value = service_url_config.read(cfg=yaml_cfg_file)
print(f"Value from YAML: {value}") # Output: Value from YAML: http://yaml-service.com
os.remove("config.yaml")

# --- Scenario 4: No value found ---
value = service_url_config.read()
print(f"Value when not found: {value}") # Output: Value when not found: None
```

The `read` method first checks for the corresponding environment variable. If not found, it attempts to read from the `ConfigFile` instance provided (if any), prioritizing the legacy INI format over YAML if both are conceptually defined within the `ConfigEntry` and the `ConfigFile` instance supports them.

### Type Transformations

When reading configuration values, Flytekit can automatically transform string values from files or environment variables into the desired Python type.

The `ConfigEntry` class includes `legacy_default_transforms` for common types:
*   `bool`: Transforms string representations (e.g., "true", "false") into boolean values.
*   `list`: Splits comma-separated strings into a list of strings.
*   `int`: Converts string representations to integers.

You can provide a custom `transform` callable to a `ConfigEntry` to handle specific parsing logic:

```python
from flytekit.configuration.file import ConfigEntry, LegacyConfigEntry
import json
import typing
import os

def json_transformer(s: str) -> typing.Dict:
    return json.loads(s)

# Define a legacy entry for a JSON string
json_config_legacy = LegacyConfigEntry(section="data", option="payload", type_=str)

# Create a ConfigEntry with a custom transformer
json_config_entry = ConfigEntry(legacy=json_config_legacy, transform=json_transformer)

# Simulate reading from an environment variable
os.environ["FLYTE_DATA_PAYLOAD"] = '{"key": "value", "count": 123}'
parsed_payload = json_config_entry.read()
print(f"Parsed JSON payload: {parsed_payload}")
print(f"Type of parsed payload: {type(parsed_payload)}")
del os.environ["FLYTE_DATA_PAYLOAD"]
```

If a `transform` function is not explicitly provided, `ConfigEntry` attempts to use a default transformer based on the `type_` specified in the `LegacyConfigEntry`.

### Best Practices and Considerations

*   **Environment Variables for Production**: For production deployments, prioritize setting configuration via environment variables. This provides a robust and secure way to manage sensitive information and allows for easy overrides without modifying files.
*   **YAML for `flytectl` Alignment**: When defining new configurations, especially those that might also be exposed via `flytectl` commands, prefer the YAML format and use the `YamlConfigEntry` with a `switch` that mirrors the `flytectl` argument structure.
*   **Consistency**: While `ConfigEntry` allows defining both legacy and YAML representations for a single setting, strive for consistency. If a new setting is introduced, consider defining it primarily in YAML and relying on environment variables for overrides.
*   **Error Handling**: The `ConfigFile` class includes basic error handling for parsing (e.g., `yaml.YAMLError`, `configparser.Error`). When reading values, `ConfigEntry.read()` returns `None` if a value is not found in any source, requiring the caller to handle this case.
*   **Reserved INI Sections**: Avoid using the section name "internal" in INI configuration files. This section is reserved for Flytekit's internal configurations, and its presence will raise an error.
<!--
key: summary_flytekit_configuration_files_842ba87f-4313-4eca-abf3-9b1ce9ddb4c4
type: summary_end

-->
<!--
code_unit: flytekit.configuration.file.ConfigFile
code_unit_type: class
help_text: ''
key: example_55c66312-d1d2-4bed-a967-c00d7807b17a
type: example

-->
<!--
code_unit: flytekit.configuration.file.ConfigEntry
code_unit_type: class
help_text: ''
key: example_afe04f2b-2947-434a-9446-834cccad4822
type: example

-->