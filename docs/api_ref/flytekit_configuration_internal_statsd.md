# StatsD

This class manages the configuration for StatsD, a metrics-gathering daemon. It provides access to configuration settings such as host, port, and flags to enable or disable StatsD functionality. The class utilizes ConfigEntry to retrieve configuration values, allowing for flexible configuration management.

## Attributes

- **SECTION**: string = &quot;secrets&quot;
  - SECTION

- **HOST**: [ConfigEntry](flytekit_configuration_file_configentry) = ConfigEntry(LegacyConfigEntry(SECTION, &quot;host&quot;))
  - HOST

- **PORT**: [ConfigEntry](flytekit_configuration_file_configentry) = ConfigEntry(LegacyConfigEntry(SECTION, &quot;port&quot;, int))
  - PORT

- **DISABLED**: [ConfigEntry](flytekit_configuration_file_configentry) = ConfigEntry(LegacyConfigEntry(SECTION, &quot;disabled&quot;, bool))
  - DISABLED

- **DISABLE_TAGS**: [ConfigEntry](flytekit_configuration_file_configentry) = ConfigEntry(LegacyConfigEntry(SECTION, &quot;disable_tags&quot;, bool))
  - DISABLE_TAGS



