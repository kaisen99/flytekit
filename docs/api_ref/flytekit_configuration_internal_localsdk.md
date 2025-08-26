# LocalSDK

This class manages the configuration settings for the local SDK environment. It provides access to settings related to workflow packages, local sandbox paths, and logging levels. The class utilizes configuration entries to define and retrieve these settings.

## Attributes

- **SECTION**: string = &quot;sdk&quot;
  - The section of the configuration.

- **WORKFLOW_PACKAGES**: [ConfigEntry](flytekit_configuration_file_configentry) = ConfigEntry(LegacyConfigEntry(SECTION, &quot;workflow_packages&quot;, list))
  - This is a comma-delimited list of packages that SDK tools will use to discover entities for the purpose of registration and execution of entities.

- **LOCAL_SANDBOX**: [ConfigEntry](flytekit_configuration_file_configentry) = ConfigEntry(LegacyConfigEntry(SECTION, &quot;local_sandbox&quot;))
  - This is the path where SDK will place files during local executions and testing. The SDK will not automatically clean up data in these directories.

- **LOGGING_LEVEL**: [ConfigEntry](flytekit_configuration_file_configentry) = ConfigEntry(LegacyConfigEntry(SECTION, &quot;logging_level&quot;, int))
  - This is the default logging level for the Python logging library and will be set before user code runs. Note that this configuration is special in that it is a runtime setting, not a compile time setting. This is the only runtime option in this file. TODO delete the one from internal config

## Constructors
```def LocalSDK()
```
-  Initializes the LocalSDK.



