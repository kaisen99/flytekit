# FlytekitPlugin

This class provides static methods for configuring and interacting with Flyte, a platform for building and operating data and machine learning workflows. It offers methods to retrieve remote Flyte objects, configure the pyflyte CLI, and manage default settings such as images and cache policies. The class facilitates integration with Flyte by providing access to configurations and settings.



## Methods
@classmethod
def get_remote(config: Optional[str], project: str, domain: str, data_upload_location: Optional[str] = None) - > [FlyteRemote](flytekit_remote_remote_flyteremote)
-  Get FlyteRemote object for CLI session.
- **Parameters**

  - **config**: Optional[str]
    - Path to a flyte config file.
  - **project**: str
    - Flyte project
  - **domain**: str
    - Flyte domain
  - **data_upload_location**: Optional[str]
    - Data upload location

- **Return Value**:
**[FlyteRemote](flytekit_remote_remote_flyteremote)**
  - FlyteRemote object
@classmethod
def configure_pyflyte_cli(main: Group) - > Group
-  Configure pyflyte&#x27;s CLI.
- **Parameters**

  - **main**: Group
    - Main CLI group

- **Return Value**:
**Group**
  - Configured CLI group
```@classmethod
def secret_requires_group()
```
-  Return True if secrets require group entry during registration time.

- **Return Value**:
**bool**
  - True if secrets require group entry, False otherwise.
```@classmethod
def get_default_image()
```
-  Get default image. Return None to use the images from flytekit.configuration.DefaultImages

- **Return Value**:
**Optional[str]**
  - Default image name or None
@classmethod
def get_auth_success_html(endpoint: str) - > Optional[str]
-  Get default success html. Return None to use flytekit&#x27;s default success html.
- **Parameters**

  - **endpoint**: str
    - Authentication endpoint

- **Return Value**:
**Optional[str]**
  - Default success HTML content or None
```@classmethod
def get_default_cache_policies()
```
-  Get default cache policies for tasks.

- **Return Value**:
**List[[CachePolicy](flytekit_core_cache_cachepolicy)]**
  - List of default cache policies
