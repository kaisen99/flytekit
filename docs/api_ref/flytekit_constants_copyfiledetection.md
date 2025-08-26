# CopyFileDetection

This class defines an enumeration used to specify the file copying behavior during certain operations. It provides options to copy all files, copy files from loaded modules, or skip file copying entirely. The `NO_COPY` option indicates that no files should be copied, which is useful for scenarios where fast registration is not desired.

## Attributes

- **LOADED_MODULES**: Enum = 1
  - Represents loaded modules.

- **ALL**: Enum = 2
  - Represents all files.

- **NO_COPY**: Enum = 3
  - Represents no files should be copied. In the future this will mean that no files should be copied (i.e. no fast registration is used). For now, both this value and setting this Enum to Python None are both valid to distinguish between users explicitly setting --copy none and not setting the flag. Currently, this is only used for register, not for package or run because run doesn&#x27;t have a no-fast-register option and package is by default non-fast.



