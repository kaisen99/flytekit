
<!--
help_text: ''
key: summary_core_configuration_concepts_7294b599-10c0-446b-b234-e4b28a24d8c1
modules:
- flytekit.configuration.file.LegacyConfigEntry
- flytekit.configuration.file.ConfigEntry
- flytekit.configuration.file.ConfigFile
- flytekit.configuration.file.YamlConfigEntry
questions_to_answer: []
type: summary

-->
The core configuration system provides a robust and flexible mechanism for managing application settings across various environments. It supports configuration values from environment variables, INI-style files, and YAML files, applying a clear precedence order to ensure predictable behavior.

### Configuration Sources and Precedence

Configuration values are resolved in a specific order:

1.  **Environment Variables:** These have the highest precedence. Values are read from environment variables following the `FLYTE_{SECTION}_{OPTION}` naming convention (e.g., `FLYTE_MYAPP_DEBUG_MODE`).
2.  **INI-style Configuration Files:** These have the next precedence. Values are read from standard INI files, organized by `[section]` and `option=value`.
3.  **YAML Configuration Files:** These have the lowest precedence. Values are read from YAML files, with keys accessed via dot-delimited paths (e.g., `myApp.settings.debugMode`).

The system retrieves the value from the first source in this order that provides a non-null value for a given configuration entry.

### Defining Configuration Entries

Configuration options are defined using the `ConfigEntry` class, which acts as a unified descriptor for a setting. A `ConfigEntry` encapsulates two specific entry types:

*   `LegacyConfigEntry`: Used for options sourced from environment variables or INI files. It requires a `section` (e.g., "my_app"), an `option` name (e.g., "debug_mode"), and an expected `type_` (e.g., `bool`, `str`, `int`, `list`).
*   `YamlConfigEntry`: Used for options sourced from YAML files. It requires a `switch` (a dot-delimited path, e.g., "myApp.settings.debugMode") and an expected `config_value_type`.

**Example of defining configuration entries:**

```python
from dataclasses import dataclass, field
import typing
import os

# Assume LegacyConfigEntry, YamlConfigEntry, ConfigEntry, ConfigFile are available
# (as provided in the context)

@dataclass
class ApplicationConfig:
    """
    Defines all configuration options for the application.
    """
    debug_mode: ConfigEntry = field(
        default_factory=lambda: ConfigEntry(
            legacy=LegacyConfigEntry(section="app", option="debug_mode", type_=bool),
            yaml_entry=YamlConfigEntry(switch="app.settings.debugMode", config_value_type=bool),
        )
    )
    log_level: ConfigEntry = field(
        default_factory=lambda: ConfigEntry(
            legacy=LegacyConfigEntry(section="logging", option="level", type_=str),
            yaml_entry=YamlConfigEntry(switch="logging.level", config_value_type=str),
        )
    )
    feature_flags: ConfigEntry = field(
        default_factory=lambda: ConfigEntry(
            legacy=LegacyConfigEntry(section="features", option="enabled_flags", type_=list),
            yaml_entry=YamlConfigEntry(switch="features.enabledFlags", config_value_type=list),
        )
    )
```

### Loading Configuration Files

The `ConfigFile` class is responsible for parsing and holding the contents of a configuration file. It automatically detects whether the file is INI-style or YAML based on its extension (`.yaml` or `.yml`).

A `ConfigFile` instance provides access to the parsed content, either as a `configparser.ConfigParser` object for INI files or a dictionary for YAML files.

**Example of loading configuration files:**

```python
# Assume 'config.ini' exists with:
# [app]
# debug_mode = True
#
# [logging]
# level = INFO

# Assume 'config.yaml' exists with:
# app:
#   settings:
#     debugMode: false
# logging:
#   level: DEBUG
# features:
#   enabledFlags:
#     - featureA
#     - featureB

# Load an INI file
ini_cfg = ConfigFile("config.ini")

# Load a YAML file
yaml_cfg = ConfigFile("config.yaml")
```

**Important Considerations for `ConfigFile`:**

*   The `ConfigFile` constructor does not support loading both INI and YAML formats from a single instance. It loads one or the other based on the file extension.
*   The `internal` section is reserved and cannot be used in INI configuration files. Attempting to use it will raise a `FlyteAssertion`.

### Retrieving Configuration Values

To retrieve a configuration value, call the `read()` method on a `ConfigEntry` instance, optionally passing a `ConfigFile` object. The `read()` method applies the defined precedence order (Environment &gt; INI &gt; YAML).

**Example of retrieving values:**

```python
# Assuming ApplicationConfig and cfg objects from above examples
app_config_definitions = ApplicationConfig()

# Scenario 1: Value from environment variable (highest precedence)
os.environ["FLYTE_APP_DEBUG_MODE"] = "True"
debug_mode_env = app_config_definitions.debug_mode.read(cfg=ini_cfg) # cfg can be ini_cfg or yaml_cfg, env takes precedence
print(f"Debug mode (from env): {debug_mode_env} (type: {type(debug_mode_env)})")
del os.environ["FLYTE_APP_DEBUG_MODE"] # Clean up env var

# Scenario 2: Value from INI file (if env var not set)
debug_mode_ini = app_config_definitions.debug_mode.read(cfg=ini_cfg)
print(f"Debug mode (from INI): {debug_mode_ini} (type: {type(debug_mode_ini)})")

# Scenario 3: Value from YAML file (if env var not set and INI file not provided or value not found)
# To demonstrate YAML precedence, we don't pass the INI config
debug_mode_yaml = app_config_definitions.debug_mode.read(cfg=yaml_cfg)
print(f"Debug mode (from YAML): {debug_mode_yaml} (type: {type(debug_mode_yaml)})")

# Retrieving a list of feature flags
# If FLYTE_FEATURES_ENABLED_FLAGS is set, it will be used.
# Otherwise, it will look in config.ini, then config.yaml.
# Assuming config.yaml is loaded and has 'features.enabledFlags'
enabled_flags = app_config_definitions.feature_flags.read(cfg=yaml_cfg)
print(f"Enabled flags (from YAML): {enabled_flags} (type: {type(enabled_flags)})")
```

### Type Transformation

The system automatically applies type transformations based on the `type_` specified in `LegacyConfigEntry` or `config_value_type` in `YamlConfigEntry`. Default transformers are provided for `bool`, `list` (comma-separated strings), and `int`.

Custom transformation functions can be provided via the `transform` argument when creating a `ConfigEntry`. This function will be applied to the raw string value read from the source.

**Example of custom transformation:**

```python
def to_uppercase(s: str) -> str:
    """Transforms a string to uppercase."""
    return s.upper()

@dataclass
class CustomConfig:
    my_custom_option: ConfigEntry = field(
        default_factory=lambda: ConfigEntry(
            legacy=LegacyConfigEntry(section="my_app", option="custom_value", type_=str),
            transform=to_uppercase, # Apply custom transformation
        )
    )

# Assuming environment variable FLYTE_MY_APP_CUSTOM_VALUE is set to "hello world"
os.environ["FLYTE_MY_APP_CUSTOM_VALUE"] = "hello world"
custom_val = CustomConfig().my_custom_option.read()
print(f"Custom transformed value: {custom_val}") # Output: HELLO WORLD
del os.environ["FLYTE_MY_APP_CUSTOM_VALUE"]
```

### Best Practices and Considerations

*   **Centralize Definitions:** Define all `ConfigEntry` objects in a single, accessible location (e.g., a dedicated `config.py` module or a dataclass holding all entries) to ensure consistency and ease of management.
*   **Environment Variables for Overrides:** Leverage environment variables for runtime overrides, especially in containerized environments or CI/CD pipelines, due to their highest precedence.
*   **File-based for Defaults:** Use INI or YAML files for application-wide default configurations that are checked into source control.
*   **Choose File Format:** Decide on either INI or YAML for file-based configuration within a project. While the system supports both, `ConfigFile` loads only one type at a time, so mixing them for the *same* configuration value can lead to confusion if not managed carefully.
*   **Clear Naming:** Use clear and consistent naming conventions for sections, options, and YAML paths to improve readability and maintainability.
*   **Type Safety:** Always specify the expected `type_` or `config_value_type` to enable automatic transformations and catch potential issues early.
*   **Error Handling:** The system handles missing values gracefully by returning `None`. Implement appropriate checks in your application code when retrieving values.
*   **Security:** Avoid storing sensitive information directly in configuration files that are checked into source control. Use environment variables or dedicated secrets management systems for such data.
<!--
key: summary_core_configuration_concepts_7294b599-10c0-446b-b234-e4b28a24d8c1
type: summary_end

-->
<!--
code_unit: flytekit.configuration.file
code_unit_type: class
help_text: ''
key: example_b36fe7d9-0ba5-4b52-aac4-fc8e14d6f3bc
type: example

-->