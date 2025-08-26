
<!--
help_text: ''
key: summary_cli_input_and_output_data_types_26a06d64-ecee-4e70-8a6b-39a85d72f775
modules:
- flytekit.interaction.click_types
- flytekit.clis.sdk_in_container.run.YamlFileReadingCommand
questions_to_answer: []
type: summary

-->
Custom CLI parameter types extend the standard command-line interface (CLI) capabilities to support complex and Flyte-specific data types. These types enable developers to define robust CLI arguments that seamlessly integrate with Python objects, structured data, and Flyte's data models, facilitating both local execution and remote workflow launches.

### Handling Complex Python Objects

When CLI arguments need to represent complex Python objects or structured data, specialized parameter types provide the necessary conversion logic.

#### Dynamic Python Object Loading

The `PickleParamType` allows CLI arguments to specify Python objects by their module and variable name. This is particularly useful for passing references to functions, classes, or pre-configured instances without requiring serialization.

**Usage:**
Provide the argument as a string in the format `<MODULE>:<VAR>`. The system dynamically imports the specified module and retrieves the attribute.

**Example:**
To pass a reference to a function `my_func` located in `my_module.py`:
```python
# my_module.py
def my_func(x):
    return x * 2

# CLI usage
# Assuming 'my_module.py' is in the current working directory
# pyflyte run --my-param "my_module:my_func"
```

**Considerations:**
The specified module must be discoverable and importable from the environment where the CLI command executes. Typically, this means the module file should be in the current working directory or on the Python path.

#### Structured Data Input

The `JsonParamType` facilitates the input of structured data, accepting JSON strings, JSON file paths, or YAML file paths. It automatically parses the content into Python dictionaries or lists, and can further convert them into Python dataclasses or Pydantic models.

**Usage:**
Input can be a direct JSON string, a path to a `.json` file, or a path to a `.yaml` file.

**Key Capabilities:**
*   **Direct Content or File Paths:** Accepts `{"key": "value"}` directly or a path like `config.json`.
*   **Dataclass Conversion:** Automatically converts the parsed JSON/YAML into an instance of a specified Python dataclass, leveraging `dataclass_json` for deserialization. It supports nested dataclasses within lists or dictionaries.
*   **Pydantic Model Support:** Integrates with Pydantic `BaseModel` (both v1 and v2) to validate and deserialize structured input into Pydantic model instances.

**Example:**
```python
# my_config.py
import dataclasses

@dataclasses.dataclass
class MyConfig:
    name: str
    value: int

# CLI usage with direct JSON string
# pyflyte run --config '{"name": "test", "value": 123}'

# CLI usage with a JSON file
# config.json: {"name": "test", "value": 123}
# pyflyte run --config config.json

# CLI usage with a YAML file
# config.yaml:
#   name: test
#   value: 123
# pyflyte run --config config.yaml
```

**Considerations:**
While `JsonParamType` handles conversion to dataclasses, it specifically excludes Flyte-native types such as `FlyteFile`, `FlyteDirectory`, `StructuredDataset`, and `FlyteSchema` from this dataclass conversion process, as these types have their own dedicated handling mechanisms.

### Time and Duration Inputs

Specialized types simplify the input of date, time, and duration values, supporting both standard and human-readable formats.

#### Flexible Date and Time Parsing

The `DateTimeType` extends standard date/time parsing to include convenient keywords and relative time calculations.

**Usage:**
*   **Standard Formats:** Accepts ISO 8601 formatted strings (e.g., `2023-10-27T10:00:00`).
*   **Keywords:**
    *   `now`: Represents the current timestamp.
    *   `today`: Represents the current date at midnight (00:00:00).
*   **Relative Time:** Supports expressions like `<FORMAT> - <ISO8601 duration>` or `<FORMAT> + <ISO8601 duration>` to calculate a date/time relative to a base. For example, `2023-10-27 - P1D` (one day before October 27th, 2023) or `now + PT1H` (one hour from now).

**Example:**
```bash
# Set a timestamp to now
# pyflyte run --start-time now

# Set a timestamp to today at midnight
# pyflyte run --start-time today

# Set a timestamp to 24 hours from now
# pyflyte run --start-time "now + PT24H"

# Set a timestamp to 7 days before a specific date
# pyflyte run --start-time "2023-10-27T10:00:00 - P7D"
```

#### Human-Readable Duration Parsing

The `DurationParamType` converts human-readable duration strings into `datetime.timedelta` objects.

**Usage:**
Provide durations in formats like `1h`, `30m`, `1 day`, `1:24` (1 minute 24 seconds), or `PT1H30M` (ISO 8601 duration).

**Example:**
```bash
# Set a timeout of 5 minutes
# pyflyte run --timeout "5m"

# Set a schedule interval of 1 day and 12 hours
# pyflyte run --interval "1 day 12h"
```

### File System and Structured Data Inputs

These types provide robust handling for file paths, directory paths, and Flyte's structured dataset abstraction.

#### File Path Handling

The `FileParamType` processes file paths, validating local file existence and converting them into `FlyteFile` objects.

**Usage:**
Provide a local file path or a remote URI (e.g., `s3://my-bucket/data.csv`).

**Output:**
A `FlyteFile` object, which abstracts file access regardless of whether the file is local or remote.

**Considerations:**
When running locally, the system ensures the provided path exists and points to a file. For remote execution, the path is treated as a URI. The `remote_path` attribute of the `FlyteFile` object is automatically adjusted based on the execution context.

#### Directory Path Handling

Similar to `FileParamType`, the `DirParamType` handles directory paths, validating local directory existence and converting them into `FlyteDirectory` objects.

**Usage:**
Provide a local directory path or a remote URI (e.g., `s3://my-bucket/my-dir/`).

**Output:**
A `FlyteDirectory` object, providing a unified interface for directory access.

**Considerations:**
Local execution validates the existence of the directory. For remote execution, the path is treated as a URI. The `remote_directory` attribute of the `FlyteDirectory` object is automatically adjusted based on the execution context.

#### Structured Dataset Input

The `StructuredDatasetParamType` facilitates passing inputs that represent Flyte's `StructuredDataset` type.

**Usage:**
Input can be a URI string (e.g., `s3://my-bucket/my-dataset`), an existing `StructuredDataset` object, or even a dataframe directly (though typically used for URIs in CLI).

**Output:**
A `StructuredDataset` object, which encapsulates the location and schema of structured data.

**Example:**
```bash
# Pass a URI to a structured dataset
# pyflyte run --dataset-uri "s3://my-data-lake/sales_data"
```

### Flexible and Enumerated Inputs

These types enhance CLI flexibility by allowing arguments to accept multiple data types or to be constrained to a predefined set of choices.

#### Union Type Arguments

The `UnionParamType` enables a single CLI argument to accept values that conform to one of several specified types. This is useful for arguments where the input format might vary.

**Usage:**
Define a union of `click.ParamType` instances. The system attempts to convert the input value using each type in a defined precedence order.

**Behavior:**
The type conversion attempts proceed in a specific order: non-string types are prioritized over string types to prevent ambiguous conversions (e.g., an integer string being interpreted as a generic string instead of an integer). The first successful conversion determines the output type.

**Example:**
Consider an argument that could be either an integer or a file path:
```python
# In your CLI definition
# my_param = UnionParamType([click.INT, FileParamType()])

# CLI usage
# pyflyte run --my-param 123          # Interpreted as an integer
# pyflyte run --my-param "data.txt"   # Interpreted as a FlyteFile
```

#### Enumerated Choices

The `EnumParamType` constrains CLI input to a predefined set of values, corresponding to members of a Python `enum.Enum`. This ensures type safety and provides clear options to the user.

**Usage:**
Initialize with a Python `enum.Enum` class. The CLI will automatically provide choices based on the enum's values.

**Example:**
```python
import enum

class Status(enum.Enum):
    PENDING = "pending"
    RUNNING = "running"
    COMPLETED = "completed"

# In your CLI definition
# status_param = EnumParamType(Status)

# CLI usage
# pyflyte run --status pending
# pyflyte run --status RUNNING # Case-insensitive matching might depend on click.Choice behavior
```

### Internal Conversion Mechanisms

Beyond user-facing parameter types, an internal converter plays a crucial role in bridging CLI inputs with Flyte's core type system.

#### Flyte Literal Conversion

The `FlyteLiteralConverter` acts as a bridge, converting CLI input values into Flyte `Literal` objects for remote execution or retaining them as native Python types for local execution. This is a key integration point with Flyte's `TypeEngine`.

**Role:**
*   **Remote Execution:** When a workflow is launched remotely, CLI arguments must be converted into Flyte `Literal` objects, which are the canonical representation of data within the Flyte platform. `FlyteLiteralConverter` performs this serialization using Flyte's `TypeEngine`.
*   **Local Execution:** For local `pyflyte run` commands, the values are typically kept as native Python types, avoiding unnecessary serialization/deserialization overhead.
*   **Type Adaptation:** Handles specific type adaptations, such as converting a `datetime.datetime` object (often produced by `click.DateTime`) to a `datetime.date` object if the target Python type is `date`.
*   **Default Value Handling:** If a CLI input matches the default value defined in the launch plan, the conversion can be optimized or skipped.

**Key Integration:**
This converter works in conjunction with Flyte's `TypeEngine` to ensure that the Python type of the CLI argument is correctly mapped to a Flyte `LiteralType` and then converted into a `Literal` value.

**Considerations:**
The `is_remote` flag passed to the converter dictates whether the output is a Flyte `Literal` or a native Python type. The converter also handles `ArtifactQuery` instances by passing them through directly, indicating a mechanism for deferred resolution of inputs.

#### JSON Iterator Placeholder

The `JSONIteratorParamType` serves as a placeholder for scenarios involving JSON iterators. Its `convert` method simply returns the input value as-is, implying that the actual iteration and processing logic occurs elsewhere in the system. This type is typically used for advanced streaming or batch processing patterns where the CLI argument represents a source of JSON data that will be consumed iteratively.
<!--
key: summary_cli_input_and_output_data_types_26a06d64-ecee-4e70-8a6b-39a85d72f775
type: summary_end

-->
<!--
code_unit: flytekit.interaction.click_types.FileParamType
code_unit_type: class
help_text: ''
key: example_00ec232f-bdd4-46c0-87b4-3e15c73fb865
type: example

-->
<!--
code_unit: flytekit.interaction.click_types.DirParamType
code_unit_type: class
help_text: ''
key: example_0687af76-e6f0-4f8d-9cb8-d8abf62a2bf2
type: example

-->
<!--
code_unit: flytekit.interaction.click_types.JsonParamType
code_unit_type: class
help_text: ''
key: example_42d5e1a9-5bef-4948-be54-2c733b3551c2
type: example

-->
<!--
code_unit: flytekit.clis.sdk_in_container.run.YamlFileReadingCommand
code_unit_type: class
help_text: ''
key: example_c85c2ae6-6c93-4097-bc37-52c17d1218e8
type: example

-->