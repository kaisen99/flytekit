# FlyteRemote

This class serves as the primary interface for interacting with a remote Flyte backend. It facilitates the registration, execution, and management of Flyte entities such as tasks, workflows, and launch plans. Key features include methods for fetching entities, executing workflows, and monitoring execution status, with dependencies on configuration and client libraries.

## Attributes

- **config**: [Config](flytekit_configuration_config)
  - Image config.

- **default_project**: typing.Optional[str]
  - default project to use when fetching or executing flyte entities.

- **default_domain**: typing.Optional[str]
  - default domain to use when fetching or executing flyte entities.

- **data_upload_location**: str = &quot;flyte://my-s3-bucket/&quot;
  - this is where all the default data will be uploaded when providing inputs.
The default location - `s3://my-s3-bucket/data` works for sandbox/demo environment. Please override this for non-sandbox cases.

- **interactive_mode_enabled**: typing.Optional[bool]
  - If set to True, the FlyteRemote will pickle the task/workflow, if False, it will not. If set to None, then it will automatically detect if it is running in an interactive environment like a Jupyter notebook and enable interactive mode.

## Constructors
def FlyteRemote(config: [Config](flytekit_configuration_config), default_project: typing.Optional[str], default_domain: typing.Optional[str], data_upload_location: str = flyte://my-s3-bucket/, interactive_mode_enabled: typing.Optional[bool], kwargs: typing.Any)
-  Initialize a FlyteRemote object.

        :type kwargs: All arguments that can be passed to create the SynchronousFlyteClient. These are usually grpc
            parameters, if you want to customize credentials, ssl handling etc.
        :param default_project: default project to use when fetching or executing flyte entities.
        :param default_domain: default domain to use when fetching or executing flyte entities.
        :param data_upload_location: this is where all the default data will be uploaded when providing inputs.
            The default location - `s3://my-s3-bucket/data` works for sandbox/demo environment. Please override this for non-sandbox cases.
        :param interactive_mode_enabled: If set to True, the FlyteRemote will pickle the task/workflow, if False, 
         it will not. If set to None, then it will automatically detect if it is running in an interactive environment
         like a Jupyter notebook and enable interactive mode.
- **Parameters**

  - **config**: [Config](flytekit_configuration_config)
    - null
  - **default_project**: typing.Optional[str]
    - default project to use when fetching or executing flyte entities.
  - **default_domain**: typing.Optional[str]
    - default domain to use when fetching or executing flyte entities.
  - **data_upload_location**: str
    - this is where all the default data will be uploaded when providing inputs.
            The default location - `s3://my-s3-bucket/data` works for sandbox/demo environment. Please override this for non-sandbox cases.
  - **interactive_mode_enabled**: typing.Optional[bool]
    - If set to True, the FlyteRemote will pickle the task/workflow, if False, 
         it will not. If set to None, then it will automatically detect if it is running in an interactive environment
         like a Jupyter notebook and enable interactive mode.
  - **kwargs**: typing.Any
    - All arguments that can be passed to create the SynchronousFlyteClient. These are usually grpc
            parameters, if you want to customize credentials, ssl handling etc.



