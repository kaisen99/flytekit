
<!--
help_text: ''
key: summary_platform_&_admin_connection_settings_e377fc07-cf59-4798-8ae3-f52fec49c13f
modules:
- flytekit.configuration.internal.Platform
questions_to_answer: []
type: summary

-->
Platform & Admin Connection Settings centralize the configuration for establishing and managing connections to the core platform and its administrative interfaces. These settings are crucial for defining network endpoints, securing communications, and routing traffic through proxies.

### Configuration Management

Connection settings are managed through a robust configuration system that supports multiple sources, including legacy configurations and modern YAML-based definitions. Each setting is represented by a `ConfigEntry`, which intelligently resolves the effective value from these sources. This design ensures backward compatibility while promoting a clear, declarative configuration approach, typically favoring YAML definitions when present.

### Core Connection Endpoints

The following settings define the primary network locations for platform services:

*   **`URL`**: Specifies the main endpoint for the platform. This is the fundamental address used for most API interactions and service communications.
    *   **Type**: String
    *   **Example YAML Configuration**:
        ```yaml
        admin:
          endpoint: "https://api.example.com"
        ```

*   **`CONSOLE_ENDPOINT`**: Defines the URL for the administrative console or management UI. This endpoint is distinct from the primary API URL and provides access to graphical management tools.
    *   **Type**: String
    *   **Example YAML Configuration**:
        ```yaml
        console:
          endpoint: "https://console.example.com"
        ```

### Security Configuration

Secure communication is paramount. These settings control how the system handles TLS/SSL connections and certificate validation.

*   **`INSECURE`**: When set to `true`, this flag disables TLS/SSL for connections to the platform.
    *   **Type**: Boolean
    *   **Considerations**: Using this setting is highly discouraged in production environments as it exposes communications to significant security risks, including eavesdropping and man-in-the-middle attacks. It is primarily intended for development or testing scenarios where TLS setup is complex or unnecessary.
    *   **Example YAML Configuration**:
        ```yaml
        admin:
          insecure: true
        ```

*   **`INSECURE_SKIP_VERIFY`**: When set to `true`, this flag instructs the system to skip verification of the server's TLS certificate. Connections still use TLS, but the authenticity of the server is not validated.
    *   **Type**: Boolean
    *   **Considerations**: Similar to `INSECURE`, this setting compromises security by allowing connections to servers with invalid, expired, or self-signed certificates without warning. Use with extreme caution and only in controlled, non-production environments. Prefer `CA_CERT_FILE_PATH` for trusted self-signed certificates.
    *   **Example YAML Configuration**:
        ```yaml
        admin:
          insecureSkipVerify: true
        ```

*   **`CA_CERT_FILE_PATH`**: Specifies the file path to a custom Certificate Authority (CA) certificate bundle. This is essential when connecting to a platform that uses self-signed certificates or certificates issued by a private CA not trusted by default system roots.
    *   **Type**: String (file path)
    *   **Best Practice**: For secure connections to environments with custom CAs, always provide the `CA_CERT_FILE_PATH` instead of using `INSECURE_SKIP_VERIFY`.
    *   **Example YAML Configuration**:
        ```yaml
        admin:
          caCertFilePath: "/etc/ssl/certs/my_custom_ca.pem"
        ```

### Proxy Configuration

*   **`HTTP_PROXY_URL`**: Configures an HTTP proxy for all outbound connections made by the platform client. This is useful in environments where direct internet access is restricted or where all traffic must be routed through a corporate proxy.
    *   **Type**: String (URL)
    *   **Format**: `http://[user:password@]host:port`
    *   **Example YAML Configuration**:
        ```yaml
        admin:
          httpProxyURL: "http://proxy.example.com:8080"
        ```

### Usage and Best Practices

Developers configure these settings typically through a YAML configuration file or environment variables, which the system then loads and applies.

*   **Prioritize Security**: Always strive to use secure connections. Avoid `INSECURE` and `INSECURE_SKIP_VERIFY` in production.
*   **Custom CAs**: When dealing with self-signed certificates or private CAs, use `CA_CERT_FILE_PATH` to establish trust.
*   **Environment Variables**: For dynamic deployments or sensitive information, consider using environment variables to override YAML settings, especially for proxy credentials or specific endpoint URLs. The underlying `ConfigEntry` mechanism supports this flexibility.
*   **Validation**: Ensure that provided URLs are well-formed and file paths for certificates are accessible. The system performs basic validation, but incorrect paths can lead to connection failures.
<!--
key: summary_platform_&_admin_connection_settings_e377fc07-cf59-4798-8ae3-f52fec49c13f
type: summary_end

-->
<!--
code_unit: flytekit.configuration.internal.Platform
code_unit_type: class
help_text: ''
key: example_11325a64-78e1-4467-a63a-864f0144091c
type: example

-->