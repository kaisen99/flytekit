
<!--
help_text: ''
key: summary_underlying_data_persistence_layer_63c877c9-70e2-4e20-8922-fe48d5c60aeb
modules:
- flytekit.core.data_persistence
questions_to_answer: []
type: summary

-->
## Underlying Data Persistence Layer

The Underlying Data Persistence Layer provides a robust and flexible mechanism for interacting with various storage systems, both local and remote. It abstracts away the complexities of different file system protocols, enabling seamless data transfer and management within the execution environment.

The primary interface for this layer is the `FileAccessProvider` class. It is available through the `FlyteContext` and serves as the central utility for persisting and retrieving data from durable storage.

### Core Capabilities of FileAccessProvider

The `FileAccessProvider` offers a comprehensive set of functionalities for data interaction:

#### Flexible Storage Access

The `FileAccessProvider` leverages `fsspec` to support a wide range of file system protocols, including local files (`file://`), Amazon S3 (`s3://`), Google Cloud Storage (`gs://`), and FTP (`ftp://`). This abstraction allows developers to interact with different storage backends using a unified API.

*   **Retrieving File Systems:** Use `get_filesystem` or `get_filesystem_for_path` to obtain an `fsspec.AbstractFileSystem` instance for a specific protocol or path. This allows direct interaction with the underlying file system if more granular control is needed.
    ```python
    # Get the default remote file system (e.g., S3 if raw_output_prefix is s3://)
    remote_fs = file_access_provider.raw_output_fs

    # Get a specific file system by protocol
    s3_fs = file_access_provider.get_filesystem("s3")

    # Get a file system based on a path's protocol
    gcs_fs = file_access_provider.get_filesystem_for_path("gs://my-bucket/data")
    ```
*   **Asynchronous Operations:** For non-blocking I/O, `get_async_filesystem_for_path` provides an asynchronous file system interface. Many data transfer operations also offer asynchronous versions.

#### Local Sandbox Management

During execution, a temporary local working directory, referred to as the "local sandbox," is available for storing intermediate or temporary data. This directory is managed by the `FileAccessProvider`.

*   **Accessing the Sandbox:** The `local_sandbox_dir` property provides the path to this temporary directory.
*   **Generating Local Paths:** Use `get_random_local_path` or `get_random_local_directory` to generate unique paths within the local sandbox, useful for temporary files or directories.
    ```python
    local_temp_file = file_access_provider.get_random_local_path("my_data.csv")
    local_temp_dir = file_access_provider.get_random_local_directory()
    ```

#### Remote Output Prefix

The `raw_output_prefix` property defines the default remote durable storage location for outputs. This prefix is typically configured by the execution environment and ensures that task outputs are stored in a consistent and accessible location. The `raw_output_fs` property provides the `fsspec` filesystem for this default prefix.

#### Data Transfer Operations

The `FileAccessProvider` facilitates efficient data transfer between local and remote storage. It provides both synchronous and asynchronous methods for uploading and downloading files and directories.

*   **Downloading Data:**
    *   `download(remote_path, local_path)`: Downloads a single file from a remote location to a local path.
    *   `download_directory(remote_path, local_path)`: Recursively downloads an entire directory from a remote location to a local path.
    *   These methods are synchronous wrappers around `async_get_data`.
    ```python
    # Download a file
    file_access_provider.download("s3://my-bucket/data/input.txt", "/tmp/input.txt")

    # Download a directory
    file_access_provider.download_directory("s3://my-bucket/output_dir/", "/tmp/local_output/")
    ```
*   **Uploading Data:**
    *   `upload(local_path, to_path)`: Uploads a single file from a local path to a remote location.
    *   `upload_directory(local_path, remote_path)`: Recursively uploads an entire directory from a local path to a remote location.
    *   These methods are synchronous wrappers around `async_put_data`.
    ```python
    # Upload a file
    file_access_provider.upload("/tmp/result.json", "s3://my-bucket/results/result.json")

    # Upload a directory
    file_access_provider.upload_directory("/tmp/model_artifacts/", "s3://my-bucket/models/my_model/")
    ```
*   **Raw Data Upload (`put_raw_data` / `async_put_raw_data`):** This method offers more flexibility for uploading various data types (strings, bytes, file-like objects, or paths) directly to the `raw_output_prefix`. It automatically handles file naming and recursive uploads for directories.
    ```python
    import io

    # Upload a string as a file
    file_access_provider.put_raw_data(io.StringIO("Hello, Flyte!"), file_name="greeting.txt")

    # Upload bytes
    file_access_provider.put_raw_data(b"binary data", file_name="image.bin")

    # Upload a local file (similar to upload, but targets raw_output_prefix by default)
    file_access_provider.put_raw_data("/path/to/local/data.csv", upload_prefix="my_data")
    ```
    The `put_raw_data` method is particularly useful when the exact remote path is not critical, and the data should simply be stored under the default output location.

#### Path Utilities

The `FileAccessProvider` includes several static methods and properties to assist with path manipulation, ensuring compatibility across different file systems.

*   `join(*args, fs=None)`: Joins path components using the correct separator for the specified or default file system.
*   `strip_file_header(path)`: Removes the `file://` prefix from a path if present.
*   `get_file_tail(file_path_or_file_name)`: Extracts the base name (file name or last directory name) from a path.
*   `get_random_string()`: Generates a unique hexadecimal string, useful for creating unique file or directory names.
*   `generate_new_custom_path(fs=None, alt=None, stem=None)`: Creates a new remote path, allowing for customization of the bucket/prefix and the final stem. This is useful for writing to specific, non-default locations within a remote storage system.
    ```python
    # Generate a path in a different S3 bucket
    custom_s3_path = file_access_provider.generate_new_custom_path(alt="my-other-s3-bucket", stem="custom_output")
    # Example result: s3://my-other-s3-bucket/default-prefix-part/custom_output
    ```
*   `exists(path)`: Checks if a given path exists on the corresponding file system.

### Integration and Usage

Developers typically access the `FileAccessProvider` instance through the `FlyteContext` within their task code. This ensures that the provider is correctly initialized with the execution-specific local sandbox and remote output prefix.

```python
from flytekit import task, FlyteContext
import pandas as pd

@task
def process_data_and_store(input_csv: str) -> str:
    ctx = FlyteContext.current_context()
    file_access_provider = ctx.file_access

    # 1. Download input data to local sandbox
    local_input_path = file_access_provider.get_random_local_path("input.csv")
    file_access_provider.download(input_csv, local_input_path)

    # 2. Process data locally
    df = pd.read_csv(local_input_path)
    df["new_column"] = df["existing_column"] * 2
    local_output_path = file_access_provider.get_random_local_path("output.csv")
    df.to_csv(local_output_path, index=False)

    # 3. Upload processed data to the default remote output prefix
    # The return value of put_raw_data is the remote path
    remote_output_path = file_access_provider.put_raw_data(local_output_path, upload_prefix="processed_data")

    return remote_output_path
```

### Important Considerations

*   **Windows `file://` Prefix:** On Windows operating systems, using the `file://` prefix with `FileAccessProvider` is not supported. Local paths should be provided without this prefix.
*   **Asynchronous Operations:** While synchronous wrappers are provided for convenience (`get_data`, `put_data`), the underlying implementation often uses asynchronous `fsspec` calls. For advanced use cases or performance-critical paths, consider using the `async_` prefixed methods directly within an `async` context.
*   **Error Handling:** The `FileAccessProvider` raises specific exceptions like `FlyteDataNotFoundException` if a remote path does not exist during a `get` operation, and `FlyteDownloadDataException` or `FlyteUploadDataException` for general transfer failures. Implement appropriate `try-except` blocks to handle these scenarios.
*   **`is_remote` Method:** The `is_remote` static method is deprecated. To determine if a path is remote, extract its protocol and check if it is not "file".
*   **Metadata:** The `FileAccessProvider` can optionally attach execution metadata to uploaded files if configured, which can be useful for tracking provenance or additional context in the remote storage system.
*   **Retry Mechanism:** Data transfer operations (`get`, `_put`) are decorated with a `retry_request` mechanism, enhancing robustness against transient network issues or temporary storage unavailability.
<!--
key: summary_underlying_data_persistence_layer_63c877c9-70e2-4e20-8922-fe48d5c60aeb
type: summary_end

-->
<!--
code_unit: flytekit.core.data_persistence.FileAccessProvider
code_unit_type: class
help_text: ''
key: example_fc15c7ba-4b78-4343-a55a-98d38f0003a4
type: example

-->