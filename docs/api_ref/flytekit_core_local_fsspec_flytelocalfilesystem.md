# FlyteLocalFileSystem

This class specializes in file system operations within a local environment. It primarily overrides the separator attribute to ensure compatibility across different operating systems, particularly Windows. This ensures consistent path handling when interacting with the local file system.

## Attributes

- **sep**: string = os.sep
  - This class doesn&#x27;t do anything except override the separator so that it works on windows



