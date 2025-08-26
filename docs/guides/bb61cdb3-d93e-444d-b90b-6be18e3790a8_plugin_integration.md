
<!--
help_text: ''
key: summary_plugin_integration_ad17883b-975b-4fd4-9d94-c85837c9bf6c
modules:
- flytekit.configuration.plugin
questions_to_answer: []
type: summary

-->
# Plugin Integration

Plugin integration provides a powerful mechanism to customize and extend Flytekit's core behaviors, particularly around remote connection management, command-line interface (CLI) interactions, and default settings for tasks and workflows. This allows developers to tailor Flytekit to specific organizational requirements, integrate with custom infrastructure, or enforce consistent configurations across projects.

## Plugin Architecture

The plugin system is built around a protocol-based design, defining a clear contract for custom implementations.

### The Plugin Protocol

The `FlytekitPluginProtocol` defines the interface that any custom plugin must adhere to. It specifies a set of static methods that Flytekit expects a plugin to implement. This protocol acts as a blueprint, ensuring that any plugin provides the necessary capabilities for Flytekit to interact with it.

Key methods defined in the protocol include:

*   `get_remote`: Responsible for providing a `FlyteRemote` object, which manages connections to the Flyte backend.
*   `configure_pyflyte_cli`: Allows for extending or modifying the `pyflyte` CLI's command structure.
*   `secret_requires_group`: Indicates whether secrets require a group entry during registration.
*   `get_default_image`: Specifies a default Docker image to be used for tasks.
*   `get_auth_success_html`: Provides custom HTML content for authentication success pages.
*   `get_default_cache_policies`: Defines default caching policies for tasks.

### The Default Plugin Implementation

The `FlytekitPlugin` class provides a default implementation of the `FlytekitPluginProtocol`. This class serves as the baseline behavior for Flytekit when no custom plugin is explicitly configured or discovered. Developers can use this as a reference or extend it to create their own specialized plugins.

The default behaviors provided by `FlytekitPlugin` are:

*   **`get_remote`**:
    *   Attempts to load configuration from a specified file.
    *   If no configuration file is found, it checks for the `FLYTE_PLATFORM_URL` environment variable to auto-configure.
    *   If neither is present, it defaults to a sandbox configuration, simplifying initial setup for local development.
    *   It constructs and returns a `FlyteRemote` instance based on the determined configuration.
*   **`configure_pyflyte_cli`**: Returns the `main` CLI group unchanged, meaning no additional commands or modifications are added to the `pyflyte` CLI by default.
*   **`secret_requires_group`**: Returns `True`, indicating that secrets, by default, require a group entry during registration.
*   **`get_default_image`**: Returns `None`, which instructs Flytekit to use its internal default image resolution logic.
*   **`get_auth_success_html`**: Returns `None`, causing Flytekit to use its built-in default HTML for authentication success.
*   **`get_default_cache_policies`**: Returns an empty list, meaning no default cache policies are applied to tasks.

## Key Plugin Capabilities

Custom plugins can override these default behaviors to introduce specific functionalities or enforce organizational standards.

### Customizing Remote Connections

The `get_remote` method is a critical integration point. By implementing this method, a plugin can control how Flytekit connects to the Flyte backend. This is useful for:

*   **Centralized Configuration**: Managing connection details (endpoint, credentials) from a single, custom source rather than relying solely on environment variables or local configuration files.
*   **Dynamic Endpoint Resolution**: Implementing logic to determine the Flyte endpoint based on environment, user, or other dynamic factors.
*   **Custom Authentication Flows**: Integrating with proprietary authentication systems by injecting custom `AuthType` or `AuthClient` configurations into the `FlyteRemote` object.

**Example:**
A custom plugin might always connect to a specific staging environment unless an explicit configuration is provided.

```python
from flytekit.configuration.plugin import FlytekitPluginProtocol
from flytekit.remote.remote import FlyteRemote
from flytekit.configuration import Config
from typing import Optional

class MyCustomRemotePlugin(FlytekitPluginProtocol):
    @staticmethod
    def get_remote(
        config: Optional[str], project: str, domain: str, data_upload_location: Optional[str] = None
    ) -> FlyteRemote:
        # Always use a specific staging endpoint unless a config file is provided
        if config is None:
            cfg_obj = Config.for_endpoint("https://flyte.staging.example.com")
            print("Using custom staging endpoint for remote connection.")
        else:
            cfg_obj = Config.auto(config)
        return FlyteRemote(
            cfg_obj, default_project=project, default_domain=domain, data_upload_location=data_upload_location
        )

    # Other methods would also need to be implemented or inherited from FlytekitPlugin
    # For brevity, only get_remote is shown here.
    @staticmethod
    def configure_pyflyte_cli(main):
        return main

    @staticmethod
    def secret_requires_group() -> bool:
        return True

    @staticmethod
    def get_default_image() -> Optional[str]:
        return None

    @staticmethod
    def get_auth_success_html(endpoint: str) -> Optional[str]:
        return None

    @staticmethod
    def get_default_cache_policies():
        return []
```

### Extending the `pyflyte` CLI

The `configure_pyflyte_cli` method allows plugins to inject custom commands or modify existing ones within the `pyflyte` CLI. This is invaluable for:

*   **Domain-Specific Commands**: Adding commands relevant to a particular team or project, such as `pyflyte my-project deploy-model`.
*   **Automating Workflows**: Providing shortcuts for common operations that involve multiple Flytekit commands or external tools.
*   **Custom Setup/Teardown**: Adding commands for environment setup, data preparation, or resource cleanup.

### Managing Secrets Configuration

The `secret_requires_group` method controls whether secrets registered with Flyte require a group entry. By default, this is `True`. A plugin can override this to `False` if the environment or security policy dictates that secrets do not need to be associated with a specific group.

### Setting Default Task Images

The `get_default_image` method enables a plugin to specify a default Docker image that Flytekit tasks will use if no image is explicitly defined. This is crucial for:

*   **Standardization**: Ensuring all tasks within an organization or project use approved base images.
*   **Environment Consistency**: Automatically pointing tasks to images pre-configured with necessary dependencies for a specific execution environment.

### Customizing Authentication Success Pages

The `get_auth_success_html` method allows a plugin to provide custom HTML content for the authentication success page displayed after a successful login flow. This can be used for:

*   **Branding**: Displaying company logos or specific messages.
*   **User Guidance**: Providing instructions or links to next steps after authentication.

### Defining Default Cache Policies

The `get_default_cache_policies` method enables a plugin to define a list of `CachePolicy` objects that will be applied by default to tasks. This is useful for:

*   **Performance Optimization**: Automatically enabling caching for certain types of tasks to reduce execution time and resource consumption.
*   **Cost Management**: Implementing policies that prevent re-execution of expensive tasks.

## Implementing a Custom Plugin

To implement a custom plugin, create a Python class that adheres to the `FlytekitPluginProtocol`. This means implementing all the static methods defined in the protocol. While you can start from scratch, it's often practical to inherit from `FlytekitPlugin` and override only the methods you need to customize.

**Example of a comprehensive custom plugin:**

```python
from flytekit.configuration.plugin import FlytekitPluginProtocol, FlytekitPlugin
from flytekit.remote.remote import FlyteRemote
from flytekit.configuration import Config
from flytekit.core.cache import CachePolicy
from click import Group
from typing import Optional, List

class MyOrgFlytekitPlugin(FlytekitPlugin):
    """
    A custom plugin for MyOrg that enforces specific defaults and adds CLI commands.
    """

    @staticmethod
    def get_remote(
        config: Optional[str], project: str, domain: str, data_upload_location: Optional[str] = None
    ) -> FlyteRemote:
        # Always use a specific production endpoint unless overridden by config
        if config is None:
            cfg_obj = Config.for_endpoint("https://flyte.prod.myorg.com")
            print("MyOrg Plugin: Using production endpoint by default.")
        else:
            cfg_obj = Config.auto(config)
        return FlyteRemote(
            cfg_obj, default_project=project, default_domain=domain, data_upload_location=data_upload_location
        )

    @staticmethod
    def configure_pyflyte_cli(main: Group) -> Group:
        # Add a custom command to the pyflyte CLI
        @main.command("my-org-status")
        def my_org_status():
            """
            Checks the status of MyOrg's Flyte services.
            """
            print("MyOrg Flyte services are operational.")
            # In a real scenario, this would call an internal API or check logs.

        return main

    @staticmethod
    def secret_requires_group() -> bool:
        # MyOrg policy: secrets do not require a group
        return False

    @staticmethod
    def get_default_image() -> Optional[str]:
        # Enforce a specific base image for all tasks
        return "myorg/flytekit-base:1.2.3"

    @staticmethod
    def get_auth_success_html(endpoint: str) -> Optional[str]:
        # Provide a custom success page with MyOrg branding
        return f"""
        <html>
            <head><title>MyOrg Flyte Auth Success</title></head>
            <body>
                <h1>Welcome to MyOrg Flyte!</h1>
                <p>You have successfully authenticated to {endpoint}.</p>
                <p>You can now close this window.</p>
                <img src="https://myorg.com/logo.png" alt="MyOrg Logo" width="100">
            </body>
        </html>
        """

    @staticmethod
    def get_default_cache_policies() -> List[CachePolicy]:
        # Apply a default cache policy for all tasks
        return [CachePolicy(enable_cache=True, cache_version="v1")]
```

For Flytekit to utilize a custom plugin, the plugin class must be discoverable within the Python environment. The exact mechanism for plugin discovery and activation depends on the Flytekit setup and environment variables (e.g., `FLYTEKIT_PLUGIN`). Refer to the Flytekit deployment and configuration guides for details on how to register and activate custom plugins.

## Common Use Cases

*   **Enterprise Standardization**: Enforce consistent configurations, such as default Docker images, cache policies, and secret handling, across all projects within an organization.
*   **Custom CLI Tools**: Develop specialized `pyflyte` commands that automate common development or deployment tasks specific to a team's workflow.
*   **Infrastructure Integration**: Seamlessly integrate Flytekit with existing internal systems for authentication, logging, or resource management by customizing remote connection logic.
*   **Environment-Specific Defaults**: Automatically configure Flytekit to connect to different Flyte clusters (e.g., development, staging, production) based on the current environment.

## Considerations and Best Practices

*   **Modularity**: Keep plugin implementations focused on specific concerns. If a plugin becomes too large, consider breaking it down.
*   **Backward Compatibility**: When updating a plugin, ensure that changes do not break existing workflows or CLI scripts that rely on its behavior.
*   **Testing**: Thoroughly test custom plugin implementations, especially those that modify core behaviors like `get_remote` or `configure_pyflyte_cli`.
*   **Documentation**: Document the specific behaviors and configurations introduced by your custom plugin for other developers.
*   **Performance**: While the plugin methods are generally lightweight, be mindful of any complex logic within `get_remote` or `configure_pyflyte_cli` that could impact CLI startup time.
*   **Overriding Defaults**: Understand that a custom plugin's settings will override Flytekit's internal defaults. Ensure that these overrides are intentional and well-understood.
<!--
key: summary_plugin_integration_ad17883b-975b-4fd4-9d94-c85837c9bf6c
type: summary_end

-->
<!--
code_unit: flytekit.configuration.plugin.FlytekitPlugin
code_unit_type: class
help_text: ''
key: example_7a4d45e3-f277-4d56-9698-4e58df5e19f0
type: example

-->
<!--
code_unit: flytekit.configuration.plugin.FlytekitPluginProtocol
code_unit_type: class
help_text: ''
key: example_98f9c1b1-b601-4bb2-a935-bacfe6616f89
type: example

-->