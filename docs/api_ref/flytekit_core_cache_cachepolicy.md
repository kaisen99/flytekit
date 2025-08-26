# CachePolicy

This class defines the interface for cache policy implementations. It provides a contract for determining cache versions based on a salt and version parameters. Concrete implementations of this interface dictate the specific caching behavior.



## Methods
@classmethod
def get_version(salt: string, params: [VersionParameters](flytekit_core_cache_versionparameters)) - > string
-  Returns the version string for the cache policy.
- **Parameters**

  - **salt**: string
    - A salt string to ensure version uniqueness.
  - **params**: [VersionParameters](flytekit_core_cache_versionparameters)
    - The parameters to use for generating the version.

- **Return Value**:
**string**
  - The version string.
