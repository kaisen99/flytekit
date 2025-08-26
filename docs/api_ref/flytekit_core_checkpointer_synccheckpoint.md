# SyncCheckpoint

This class provides synchronous checkpointing and restoration of files or folders. It allows saving checkpoints to a specified destination and restoring from a source location. The class utilizes file access operations for uploading and downloading checkpoint data.

## Attributes

- **SRC_LOCAL_FOLDER**: string = &quot;prev_cp&quot;
  - SRC_LOCAL_FOLDER

- **TMP_DST_PATH**: string = &quot;_dst_cp&quot;
  - TMP_DST_PATH

## Constructors
def SyncCheckpoint(checkpoint_dest: str, checkpoint_src: typing.Optional[str] = None)
-  Args:
            checkpoint_src: If a previous checkpoint should exist, this path should be set to the folder that contains the checkpoint information
            checkpoint_dest: Location where the new checkpoint should be copied to
- **Parameters**

  - **checkpoint_dest**: str
    - Location where the new checkpoint should be copied to
  - **checkpoint_src**: typing.Optional[str]
    - If a previous checkpoint should exist, this path should be set to the folder that contains the checkpoint information



## Methods
```@classmethod
def prev_exists()
```
-  Checks if a previous checkpoint source exists.

- **Return Value**:
**bool**
  - True if a previous checkpoint source exists, False otherwise.
@classmethod
def restore(path: typing.Optional[typing.Union[Path, str]] = None) - > typing.Optional[Path]
-  Restores a previous checkpoint. If no path is provided, it defaults to a temporary directory. The checkpoint is downloaded to the specified or default path.
- **Parameters**

  - **path**: typing.Optional[typing.Union[Path, str]]
    - The optional path to restore the checkpoint to. If None, a temporary directory is used.

- **Return Value**:
**typing.Optional[Path]**
  - The path to the restored checkpoint directory, or None if no checkpoint was restored.
@classmethod
def save(cp: typing.Union[Path, str, io.BufferedReader])
-  Saves a checkpoint. It can accept a file path (directory or file) or a file-like object (reader). If it&#x27;s a directory, the entire directory is uploaded. If it&#x27;s a file, only that file is uploaded. If it&#x27;s a reader, its content is saved to a temporary file and then uploaded.
- **Parameters**

  - **cp**: typing.Union[Path, str, io.BufferedReader]
    - The checkpoint data to save, which can be a Path, a string path, or a file-like object (reader).

```@classmethod
def read()
```
-  Restores the checkpoint and reads its content. It expects exactly one file within the restored checkpoint directory. If no checkpoint is found or multiple files are present, it returns None or raises a ValueError respectively.

- **Return Value**:
**typing.Optional[bytes]**
  - The content of the checkpoint file as bytes, or None if no checkpoint file is found.
@classmethod
def write(b: bytes)
-  Writes bytes to a checkpoint. The bytes are wrapped in a BytesIO object and then saved using the save method.
- **Parameters**

  - **b**: bytes
    - The bytes to write to the checkpoint.

