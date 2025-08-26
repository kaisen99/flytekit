# SerializationSettings

This class encapsulates settings required for serializing workflows and tasks before registration. It provides runtime information and default values necessary for the serialization process. Key features include configuration for project, domain, version, image, and environment variables.

## Attributes

- **image_config**: [ImageConfig](flytekit_configuration_imageconfig)
  - The image config used to define task container images.

- **project**: typing.Optional[str] = None
  - The project (if any) with which to register entities under.

- **domain**: typing.Optional[str] = None
  - The domain (if any) with which to register entities under.

- **version**: typing.Optional[str] = None
  - The version (if any) with which to register entities under.

- **env**: Optional[Dict[str, str]] = None
  - Environment variables injected into task container definitions.

- **git_repo**: Optional[str] = None
  - None

- **python_interpreter**: str = DEFAULT_RUNTIME_PYTHON_INTERPRETER
  - The python executable to use. This is used for spark tasks in out of
            container execution.

- **flytekit_virtualenv_root**: Optional[str] = None
  - During out of container serialize the absolute path of the flytekit
            virtualenv at serialization time won&#x27;t match the in-container value at execution time. This optional value
            is used to provide the in-container virtualenv path

- **fast_serialization_settings**: Optional[[FastSerializationSettings](flytekit_configuration_fastserializationsettings)] = None
  - If the code is being serialized so that it can be fast registered (and thus omit building a Docker image) this object contains additional parameters
            for serialization.

- **source_root**: Optional[str] = None
  - The root directory of the source code.

## Constructors
def SerializationSettings(image_config: [ImageConfig](flytekit_configuration_imageconfig), project: typing.Optional[str] = None, domain: typing.Optional[str] = None, version: typing.Optional[str] = None, env: Optional[Dict[str, str]] = None, git_repo: Optional[str] = None, python_interpreter: str = DEFAULT_RUNTIME_PYTHON_INTERPRETER, flytekit_virtualenv_root: Optional[str] = None, fast_serialization_settings: Optional[[FastSerializationSettings](flytekit_configuration_fastserializationsettings)] = None, source_root: Optional[str] = None)
-  These settings are provided while serializing a workflow and task, before registration. This is required to get
    runtime information at serialization time, as well as some defaults.
- **Parameters**

  - **image_config**: [ImageConfig](flytekit_configuration_imageconfig)
    - The image config used to define task container images.
  - **project**: typing.Optional[str]
    - The project (if any) with which to register entities under.
  - **domain**: typing.Optional[str]
    - The domain (if any) with which to register entities under.
  - **version**: typing.Optional[str]
    - The version (if any) with which to register entities under.
  - **env**: Optional[Dict[str, str]]
    - Environment variables injected into task container definitions.
  - **git_repo**: Optional[str]
    - None
  - **python_interpreter**: str
    - The python executable to use. This is used for spark tasks in out of
            container execution.
  - **flytekit_virtualenv_root**: Optional[str]
    - During out of container serialize the absolute path of the flytekit
            virtualenv at serialization time won&#x27;t match the in-container value at execution time. This optional value
            is used to provide the in-container virtualenv path
  - **fast_serialization_settings**: Optional[[FastSerializationSettings](flytekit_configuration_fastserializationsettings)]
    - If the code is being serialized so that it
            can be fast registered (and thus omit building a Docker image) this object contains additional parameters
            for serialization.
  - **source_root**: Optional[str]
    - The root directory of the source code.



## Methods
```@classmethod
def entrypoint_settings()
```
-  Returns the EntrypointSettings for the serialization settings. It constructs the path to the entrypoint file using the python_interpreter and a default file location.

- **Return Value**:
**[EntrypointSettings](flytekit_configuration_entrypointsettings)**
  - The entrypoint settings.
@classmethod
def from_transport(s: str) - > [SerializationSettings](flytekit_configuration_serializationsettings)
-  Deserializes a SerializationSettings object from a transport string. The string is expected to be base64 encoded and gzip compressed JSON.
- **Parameters**

  - **s**: str
    - The base64 encoded and gzip compressed JSON string representing the SerializationSettings.

- **Return Value**:
**[SerializationSettings](flytekit_configuration_serializationsettings)**
  - The deserialized SerializationSettings object.
@classmethod
def for_image(image: str, version: str, project: str = &quot;&quot;, domain: str = &quot;&quot;, python_interpreter_path: str = DEFAULT_RUNTIME_PYTHON_INTERPRETER) - > [SerializationSettings](flytekit_configuration_serializationsettings)
-  Creates a SerializationSettings object configured for a specific container image. It sets up the image configuration, project, domain, version, and python interpreter path.
- **Parameters**

  - **image**: str
    - The name or identifier of the container image.
  - **version**: str
    - The version of the container image.
  - **project**: str
    - The project name.
  - **domain**: str
    - The domain name.
  - **python_interpreter_path**: str
    - The path to the Python interpreter.

- **Return Value**:
**[SerializationSettings](flytekit_configuration_serializationsettings)**
  - A new SerializationSettings object configured for the specified image.
@classmethod
def venv_root_from_interpreter(interpreter_path: str) - > str
-  Computes the path of the virtual environment root based on the provided Python interpreter path. For example, it converts &#x27;/opt/venv/bin/python3&#x27; to &#x27;/opt/venv&#x27;.
- **Parameters**

  - **interpreter_path**: str
    - The path to the Python interpreter.

- **Return Value**:
**str**
  - The computed virtual environment root path.
@classmethod
def default_entrypoint_settings(interpreter_path: str) - > [EntrypointSettings](flytekit_configuration_entrypointsettings)
-  Assumes the entrypoint is installed in a virtual environment where the interpreter is located and returns the default EntrypointSettings.
- **Parameters**

  - **interpreter_path**: str
    - The path to the Python interpreter.

- **Return Value**:
**[EntrypointSettings](flytekit_configuration_entrypointsettings)**
  - The default entrypoint settings.
```@classmethod
def new_builder()
```
-  Creates a SerializationSettings.Builder that copies the existing serialization settings parameters and allows for customization.

- **Return Value**:
**Builder**
  - A new instance of SerializationSettings.Builder initialized with current settings.
```@classmethod
def should_fast_serialize()
```
-  Determines whether the serialization settings specify that entities should be serialized for fast registration.

- **Return Value**:
**bool**
  - True if fast serialization is enabled, False otherwise.
```@classmethod
def serialized_context()
```
-  Returns the serialization context as a base64 encoded, gzip compressed JSON string. If the context is already in the environment, it returns that value. Otherwise, it serializes the current settings, compresses, encodes, and returns it.

- **Return Value**:
**str**
  - The serialized context as a string.
```@classmethod
def with_serialized_context()
```
-  Creates a new SerializationSettings object with the serialized context added to the environment variables. This is useful for transporting serialization context to serialized and registered tasks. If the context is already present, it returns the current object.

- **Return Value**:
**[SerializationSettings](flytekit_configuration_serializationsettings)**
  - A new SerializationSettings object with the serialized context, or the current object if it already has the context.
