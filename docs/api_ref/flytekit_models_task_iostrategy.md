# IOStrategy

This class facilitates data management within a raw container, utilizing various download modes. It enables control over data transfer operations, including eager and stream-based downloads, and upload behaviors. The class is designed to work in conjunction with DataLoadingConfig.

## Attributes

- **DOWNLOAD_MODE_EAGER**: int = None
  - Eager download mode.

- **DOWNLOAD_MODE_STREAM**: int = None
  - Stream download mode.

- **DOWNLOAD_MODE_NO_DOWNLOAD**: int = None
  - No download mode.

- **UPLOAD_MODE_EAGER**: int = None
  - Eager upload mode.

- **UPLOAD_MODE_ON_EXIT**: int = None
  - Upload on exit mode.

- **UPLOAD_MODE_NO_UPLOAD**: int = None
  - No upload mode.

## Constructors
def IOStrategy(download_mode: _core_task.IOStrategy.DownloadMode = DOWNLOAD_MODE_EAGER, upload_mode: _core_task.IOStrategy.UploadMode = UPLOAD_MODE_ON_EXIT)
-  Provides methods to manage data in and out of the Raw container using Download Modes. This can only be used if DataLoadingConfig is enabled.
- **Parameters**

  - **download_mode**: _core_task.IOStrategy.DownloadMode
    - Specifies how data should be downloaded. Defaults to eager download.
  - **upload_mode**: _core_task.IOStrategy.UploadMode
    - Specifies how data should be uploaded. Defaults to upload on exit.



## Methods
```@classmethod
def to_flyte_idl()
```
-  Converts the IOStrategy object to its Flyte IDL representation.

- **Return Value**:
**_core_task.IOStrategy**
  - The Flyte IDL representation of the IOStrategy object.
@classmethod
def from_flyte_idl(pb2_object: _core_task.IOStrategy) - > [IOStrategy](flytekit_models_task_iostrategy)
-  Creates an IOStrategy object from its Flyte IDL representation.
- **Parameters**

  - **pb2_object**: _core_task.IOStrategy
    - The Flyte IDL object to convert from.

- **Return Value**:
**[IOStrategy](flytekit_models_task_iostrategy)**
  - An IOStrategy object created from the Flyte IDL representation.
