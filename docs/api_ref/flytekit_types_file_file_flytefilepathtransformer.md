# FlyteFilePathTransformer

This class transforms FlyteFile objects for use within the Flyte system. It handles the conversion of FlyteFile instances to and from various formats, including local and remote file paths. Key features include file type validation and support for different file formats based on their extensions. It relies on the AsyncTypeTransformer interface and interacts with FlyteContext for file access and management.

## Constructors
```def FlyteFilePathTransformer()
```
-  Initializes a new instance of the FlyteFilePathTransformer class. This constructor does not take any arguments and calls the parent class constructor with specific name and type values.



## Methods
@classmethod
def get_format(t: typing.Union[typing.Type[[FlyteFile](flytekit_types_file_file_flytefile)], os.PathLike]) - > string
-  Returns the file extension of the given type. If the type is os.PathLike, it returns an empty string.
- **Parameters**

  - **t**: typing.Union[typing.Type[[FlyteFile](flytekit_types_file_file_flytefile)], os.PathLike]
    - The type to get the format from.

- **Return Value**:
**string**
  - The file extension.
@classmethod
def assert_type(t: typing.Union[typing.Type[[FlyteFile](flytekit_types_file_file_flytefile)], os.PathLike], v: typing.Union[[FlyteFile](flytekit_types_file_file_flytefile), os.PathLike, str])
-  Asserts that the given value is of the expected type (FlyteFile, os.PathLike, or str).
- **Parameters**

  - **t**: typing.Union[typing.Type[[FlyteFile](flytekit_types_file_file_flytefile)], os.PathLike]
    - The expected type.
  - **v**: typing.Union[[FlyteFile](flytekit_types_file_file_flytefile), os.PathLike, str]
    - The value to assert.

- **Return Value**:
  - None
@classmethod
def get_literal_type(t: typing.Union[typing.Type[[FlyteFile](flytekit_types_file_file_flytefile)], os.PathLike]) - > [LiteralType](flytekit_models_types_literaltype)
-  Returns the LiteralType for a FlyteFile.
- **Parameters**

  - **t**: typing.Union[typing.Type[[FlyteFile](flytekit_types_file_file_flytefile)], os.PathLike]
    - The type to get the literal type for.

- **Return Value**:
**[LiteralType](flytekit_models_types_literaltype)**
  - The LiteralType for the FlyteFile.
@classmethod
def get_mime_type_from_extension(extension: str) - > typing.Union[str, typing.Sequence[str]]
-  Returns the MIME type(s) associated with a given file extension.
- **Parameters**

  - **extension**: str
    - The file extension.

- **Return Value**:
**typing.Union[str, typing.Sequence[str]]**
  - The MIME type or a list of MIME types.
@classmethod
def validate_file_type(python_type: typing.Type[[FlyteFile](flytekit_types_file_file_flytefile)], source_path: typing.Union[str, os.PathLike])
-  Validates the type of a file against the expected Python type using the magic library.
- **Parameters**

  - **python_type**: typing.Type[[FlyteFile](flytekit_types_file_file_flytefile)]
    - The expected type of the file.
  - **source_path**: typing.Union[str, os.PathLike]
    - The path to the file to validate.

- **Return Value**:
  - None
@classmethod
def async_to_literal(ctx: [FlyteContext](flytekit_core_context_manager_flytecontext), python_val: typing.Union[[FlyteFile](flytekit_types_file_file_flytefile), os.PathLike, str], python_type: typing.Type[[FlyteFile](flytekit_types_file_file_flytefile)], expected: [LiteralType](flytekit_models_types_literaltype)) - > [Literal](flytekit_models_literals_literal)
-  Asynchronously converts a Python value (FlyteFile, os.PathLike, or str) to a Flyte Literal.
- **Parameters**

  - **ctx**: [FlyteContext](flytekit_core_context_manager_flytecontext)
    - The Flyte context.
  - **python_val**: typing.Union[[FlyteFile](flytekit_types_file_file_flytefile), os.PathLike, str]
    - The Python value to convert.
  - **python_type**: typing.Type[[FlyteFile](flytekit_types_file_file_flytefile)]
    - The expected Python type.
  - **expected**: [LiteralType](flytekit_models_types_literaltype)
    - The expected LiteralType.

- **Return Value**:
**[Literal](flytekit_models_literals_literal)**
  - The Flyte Literal representation of the input value.
@classmethod
def get_additional_headers(source_path: str | os.PathLike) - > typing.Dict[str, str]
-  Returns additional headers for file uploads, such as ContentEncoding for gzipped files.
- **Parameters**

  - **source_path**: str | os.PathLike
    - The path of the source file.

- **Return Value**:
**typing.Dict[str, str]**
  - A dictionary of additional headers.
@classmethod
def dict_to_flyte_file(dict_obj: typing.Dict[str, str], expected_python_type: typing.Union[typing.Type[[FlyteFile](flytekit_types_file_file_flytefile)], os.PathLike]) - > [FlyteFile](flytekit_types_file_file_flytefile)
-  Converts a dictionary representation of a FlyteFile into a FlyteFile object.
- **Parameters**

  - **dict_obj**: typing.Dict[str, str]
    - The dictionary containing FlyteFile information (path, metadata).
  - **expected_python_type**: typing.Union[typing.Type[[FlyteFile](flytekit_types_file_file_flytefile)], os.PathLike]
    - The expected Python type for the FlyteFile.

- **Return Value**:
**[FlyteFile](flytekit_types_file_file_flytefile)**
  - The FlyteFile object.
@classmethod
def from_binary_idl(binary_idl_object: [Binary](flytekit_models_literals_binary), expected_python_type: typing.Union[typing.Type[[FlyteFile](flytekit_types_file_file_flytefile)], os.PathLike]) - > [FlyteFile](flytekit_types_file_file_flytefile)
-  Deserializes a FlyteFile from a binary IDL object (e.g., MessagePack).
- **Parameters**

  - **binary_idl_object**: [Binary](flytekit_models_literals_binary)
    - The binary IDL object to deserialize.
  - **expected_python_type**: typing.Union[typing.Type[[FlyteFile](flytekit_types_file_file_flytefile)], os.PathLike]
    - The expected Python type for the FlyteFile.

- **Return Value**:
**[FlyteFile](flytekit_types_file_file_flytefile)**
  - The deserialized FlyteFile object.
@classmethod
def from_generic_idl(generic: Struct, expected_python_type: typing.Union[typing.Type[[FlyteFile](flytekit_types_file_file_flytefile)], os.PathLike]) - > [FlyteFile](flytekit_types_file_file_flytefile)
-  Deserializes a FlyteFile from a generic IDL object (e.g., Protobuf Struct from JSON).
- **Parameters**

  - **generic**: Struct
    - The generic IDL object to deserialize.
  - **expected_python_type**: typing.Union[typing.Type[[FlyteFile](flytekit_types_file_file_flytefile)], os.PathLike]
    - The expected Python type for the FlyteFile.

- **Return Value**:
**[FlyteFile](flytekit_types_file_file_flytefile)**
  - The deserialized FlyteFile object.
@classmethod
def async_to_python_value(ctx: [FlyteContext](flytekit_core_context_manager_flytecontext), lv: [Literal](flytekit_models_literals_literal), expected_python_type: typing.Union[typing.Type[[FlyteFile](flytekit_types_file_file_flytefile)], os.PathLike]) - > [FlyteFile](flytekit_types_file_file_flytefile)
-  Asynchronously converts a Flyte Literal to a Python value (FlyteFile or os.PathLike).
- **Parameters**

  - **ctx**: [FlyteContext](flytekit_core_context_manager_flytecontext)
    - The Flyte context.
  - **lv**: [Literal](flytekit_models_literals_literal)
    - The Flyte Literal to convert.
  - **expected_python_type**: typing.Union[typing.Type[[FlyteFile](flytekit_types_file_file_flytefile)], os.PathLike]
    - The expected Python type.

- **Return Value**:
**[FlyteFile](flytekit_types_file_file_flytefile)**
  - The Python value (FlyteFile or os.PathLike).
@classmethod
def guess_python_type(literal_type: [LiteralType](flytekit_models_types_literaltype)) - > typing.Type[[FlyteFile](flytekit_types_file_file_flytefile)[typing.Any]]
-  Guesses the Python type for a given LiteralType, specifically for FlyteFile.
- **Parameters**

  - **literal_type**: [LiteralType](flytekit_models_types_literaltype)
    - The LiteralType to guess the Python type from.

- **Return Value**:
**typing.Type[[FlyteFile](flytekit_types_file_file_flytefile)[typing.Any]]**
  - The guessed Python type.
