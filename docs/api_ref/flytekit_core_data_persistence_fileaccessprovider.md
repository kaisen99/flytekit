# FileAccessProvider

This class provides access to a remote durable store for persisting data within the Flyte execution environment. It offers methods for uploading and downloading data, supporting both single files and directories. The class leverages fsspec for interacting with various storage backends and provides utilities for path manipulation and file system operations.

## Attributes

- **put_raw_data**: typing.Callable
  - This is a more flexible version of put that accepts a file-like object or a string path. Writes to the raw output prefix only. If you want to write to another fs use put_data or get the fsspec file system directly. FYI: Currently the raw output prefix set by propeller is already unique per retry and looks like s3://my-s3-bucket/data/o4/feda4e266c748463a97d-n0-0 If lpath is a folder, then recursive will be set. If lpath is a streamable, then it can only be a single file. Writes to: {raw output prefix}/{upload_prefix}/{file_name} :param lpath: A file-like object or a string path :param upload_prefix: A prefix to add to the path, see above for usage, can be an &quot;. If None then a random string will be generated :param file_name: A file name to add to the path. If None, then the file name will be the tail of the path if lpath is a file, or a random string if lpath is a buffer :param read_chunk_size_bytes: If lpath is a buffer, this is the chunk size to read from it :param encoding: If lpath is a io.StringIO, this is the encoding to use to encode it to binary. :param skip_raw_data_prefix: If True, the raw data prefix will not be prepended to the upload_prefix :param kwargs: Additional kwargs are passed into the fsspec put() call or the open() call :return: Returns the final path data was written to.

- **get_data**: typing.Callable
  - :param remote_path: :param local_path: :param is_multipart: 

- **put_data**: typing.Callable
  - :param local_path: :param remote_path: :param is_multipart: 

## Constructors
def FileAccessProvider(local_sandbox_dir: [Union](flytekit_models_literals_union)[str, os.PathLike], raw_output_prefix: str, data_config: typing.Optional[[DataConfig](flytekit_configuration_dataconfig)] = None, execution_metadata: typing.Optional[dict] = None)
-  Args:
            local_sandbox_dir: A local temporary working directory, that should be used to store data
            raw_output_prefix:
            data_config:
        
- **Parameters**

  - **local_sandbox_dir**: [Union](flytekit_models_literals_union)[str, os.PathLike]
    - A local temporary working directory, that should be used to store data
  - **raw_output_prefix**: str
    - 
  - **data_config**: typing.Optional[[DataConfig](flytekit_configuration_dataconfig)]
    - 
  - **execution_metadata**: typing.Optional[dict]
    - 



