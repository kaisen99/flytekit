
<!--
help_text: ''
key: summary_extending_configuration_with_plugins_b51c5364-bbcd-4b7c-81a2-4f5d545c83d0
modules:
- flytekit.configuration.plugin.FlytekitPlugin
- flytekit.configuration.plugin.FlytekitPluginProtocol
questions_to_answer: []
type: summary

-->
# Extending Configuration with Plugins

The system provides a plugin mechanism to customize core behaviors and configurations. This mechanism allows developers to override default settings and introduce custom logic for aspects like remote connection handling, command-line interface (CLI) extensions, default image selection, authentication success pages, and task caching policies.

The `FlytekitPluginProtocol` defines the contract for these extensions. To extend the system, developers implement a custom class that adheres to this protocol, overriding the static methods to provide specific customizations. The `FlytekitPlugin` class provides the default, baseline implementation of this protocol.

### Customizing Remote Connection Behavior

The `get_remote` method within the plugin protocol allows for custom logic when establishing a connection to the remote system. This is particularly useful for scenarios where default configuration file discovery or environment variable parsing is insufficient, or when integrating with custom authentication flows.

**Signature**:
```python
@staticmethod
def get_remote(
    config: Optional[str], project: str, domain: str, data_upload_location: Optional[str] = None
) -> FlyteRemote:
```

**Usage**:
Implement this method to return a `FlyteRemote` object based on custom logic. For instance, a plugin might:
*   Load configuration from a non-standard location.
*   Inject specific authentication headers or tokens.
*   Dynamically determine the endpoint based on environment or runtime conditions.

**Example**:
```python
from flytekit.remote.remote import FlyteRemote
from flytekit.configuration import Config
import os
from typing import Optional

# Assume FlytekitPlugin is available for default behavior fallback
from flytekit.core.plugins import FlytekitPlugin

class MyCustomPlugin:
    @staticmethod
    def get_remote(
        config: Optional[str], project: str, domain: str, data_upload_location: Optional[str] = None
    ) -> FlyteRemote:
        # Example: Always connect to a specific staging environment
        # unless a config file is explicitly provided.
        if config is None and "MY_STAGING_ENDPOINT" in os.environ:
            custom_config = Config.for_endpoint(os.environ["MY_STAGING_ENDPOINT"])
            return FlyteRemote(
                custom_config,
                default_project=project,
                default_domain=domain,
                data_upload_location=data_upload_location
            )
        else:
            # Fallback to default behavior if no custom logic applies
            return FlytekitPlugin.get_remote(config, project, domain, data_upload_location)
```

**Considerations**: The default implementation of `get_remote` makes an assumption about sandbox environments if no configuration file is found and `FLYTE_PLATFORM_URL` is not set. Custom implementations should account for desired fallback behavior.

### Extending the Command-Line Interface (CLI)

The `configure_pyflyte_cli` method provides an extension point for modifying or adding commands to the CLI. This enables developers to integrate custom tools or workflows directly into the existing CLI structure.

**Signature**:
```python
@staticmethod
def configure_pyflyte_cli(main: Group) -> Group:
```

**Usage**:
The method receives the main Click `Group` object for the CLI. Developers can add new commands or sub-groups to this object.

**Example**:
```python
from click import Group, command

class MyCustomPlugin:
    @staticmethod
    def configure_pyflyte_cli(main: Group) -> Group:
        @main.command("my-custom-command")
        def my_custom_command():
            """A custom command added by the plugin."""
            print("Executing my custom CLI command!")

        @main.group("my-group")
        def my_group():
            """A custom command group."""
            pass

        @my_group.command("sub-command")
        def my_group_sub_command():
            """A sub-command within my-group."""
            print("Executing a sub-command!")

        return main
```

### Managing Secret Group Requirements

The `secret_requires_group` method controls whether secrets require a group entry during registration. This offers flexibility in how secrets are managed and organized within the system.

**Signature**:
```python
@staticmethod
def secret_requires_group() -> bool:
```

**Usage**:
Return `True` if secrets must be associated with a group; return `False` otherwise. The default behavior requires a group.

**Example**:
```python
class MyCustomPlugin:
    @staticmethod
    def secret_requires_group() -> bool:
        # Allow secrets to be registered without a group
        return False
```

### Specifying a Default Container Image

The `get_default_image` method allows a plugin to specify a default container image to be used for tasks and workflows. This is useful for standardizing the execution environment across an organization or project without explicitly defining the image for every component.

**Signature**:
```python
@staticmethod
def get_default_image() -> Optional[str]:
```

**Usage**:
Return the full Docker image string (e.g., `"my-registry/my-image:latest"`). Returning `None` defers to the system's default image configuration.

**Example**:
```python
from typing import Optional

class MyCustomPlugin:
    @staticmethod
    def get_default_image() -> Optional[str]:
        return "my-company/flyte-base:v1.2.3"
```

### Customizing Authentication Success Page

The `get_auth_success_html` method provides a way to customize the HTML content displayed to the user after a successful authentication flow. This can be used for branding, providing additional instructions, or redirecting the user.

**Signature**:
```python
@staticmethod
def get_auth_success_html(endpoint: str) -> Optional[str]:
```

**Usage**:
Return a string containing valid HTML. The `endpoint` argument provides the URL of the authentication endpoint, which can be embedded in the custom HTML. Returning `None` uses the system's default success page.

**Example**:
```python
from typing import Optional

class MyCustomPlugin:
    @staticmethod
    def get_auth_success_html(endpoint: str) -> Optional[str]:
        return f"""
        <html>
        <head><title>Authentication Successful!</title></head>
        <body>
            <h1>Welcome to My Company's Platform!</h1>
            <p>You have successfully authenticated with {endpoint}.</p>
            <p>You can now close this window.</p>
        </body>
        </html>
        """
```

### Defining Default Task Cache Policies

The `get_default_cache_policies` method enables plugins to define a list of default cache policies for tasks. These policies determine how task outputs are cached, improving performance by reusing results from previous executions.

**Signature**:
```python
@staticmethod
def get_default_cache_policies() -> List[CachePolicy]:
```

**Usage**:
Return a list of `CachePolicy` objects. Each `CachePolicy` specifies conditions under which caching should apply. Returning an empty list means no default cache policies are applied by the plugin.

**Example**:
```python
from flytekit.core.cache import CachePolicy
from datetime import timedelta
from typing import List

class MyCustomPlugin:
    @staticmethod
    def get_default_cache_policies() -> List[CachePolicy]:
        # Apply a default cache policy for all tasks,
        # caching results for 1 hour if inputs are identical.
        return [
            CachePolicy(
                enable_cache=True,
                cache_version="v1",
                ttl=timedelta(hours=1)
            )
        ]
```
<!--
key: summary_extending_configuration_with_plugins_b51c5364-bbcd-4b7c-81a2-4f5d545c83d0
type: summary_end

-->
<!--
code_unit: flytekit.configuration.plugin
code_unit_type: class
help_text: ''
key: example_d09bac76-f1aa-4f19-8631-1352f3c15c86
type: example

-->