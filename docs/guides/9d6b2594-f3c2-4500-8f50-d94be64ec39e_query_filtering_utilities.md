
<!--
help_text: ''
key: summary_query_filtering_utilities_c302b705-f875-4eb8-a391-be8b867660e8
modules:
- flytekit.models.filters
questions_to_answer: []
type: summary

-->
Query Filtering Utilities

The Query Filtering Utilities provide a robust and extensible mechanism for programmatically constructing and serializing query conditions. These utilities are primarily designed to generate string-based filter expressions suitable for use as parameters in REST API calls, enabling dynamic and flexible data retrieval.

## Core Concepts

At the heart of the filtering system are two main components: individual `Filter` objects, which represent a single comparison, and `FilterList` objects, which combine multiple filters.

### Filter Objects

A `Filter` object represents a single comparison operation against a specific field. All concrete filter types, such as `Equal` or `GreaterThan`, inherit from the base `Filter` class. Each filter is defined by:

*   **Key**: The name of the field to compare against (e.g., `name`, `creation_time`).
*   **Value**: The textual value used in the comparison.
*   **Comparator**: An internal string (`_comparator`) that defines the type of comparison (e.g., `eq` for equality, `gt` for greater than).

For filters that operate on a set of values, such as `Contains` or `ValueIn`, the `SetFilter` class extends the base `Filter`. `SetFilter` automatically handles the serialization of a list of values into a single semicolon-separated string and the deserialization back into a list.

### Filter List

The `FilterList` utility allows for combining multiple individual `Filter` objects. When a `FilterList` is serialized, it joins the string representations of its contained filters using a `+` operator, effectively creating a logical AND condition between them. This enables building complex queries by chaining multiple criteria.

## Available Filter Types

The filtering system supports a comprehensive set of comparison operators:

*   **`Equal`**: Checks for exact equality.
    *   Comparator: `eq`
    *   Example: `Equal("name", "my_workflow")` -&gt; `eq(name,my_workflow)`
*   **`NotEqual`**: Checks for inequality.
    *   Comparator: `ne`
    *   Example: `NotEqual("status", "SUCCEEDED")` -&gt; `ne(status,SUCCEEDED)`
*   **`GreaterThan`**: Checks if a field's value is greater than the specified value.
    *   Comparator: `gt`
    *   Example: `GreaterThan("creation_time", "2023-01-01T00:00:00Z")` -&gt; `gt(creation_time,2023-01-01T00:00:00Z)`
*   **`GreaterThanOrEqual`**: Checks if a field's value is greater than or equal to the specified value.
    *   Comparator: `gte`
    *   Example: `GreaterThanOrEqual("version", "1.0.0")` -&gt; `gte(version,1.0.0)`
*   **`LessThan`**: Checks if a field's value is less than the specified value.
    *   Comparator: `lt`
    *   Example: `LessThan("execution_count", "10")` -&gt; `lt(execution_count,10)`
*   **`LessThanOrEqual`**: Checks if a field's value is less than or equal to the specified value.
    *   Comparator: `lte`
    *   Example: `LessThanOrEqual("duration_seconds", "300")` -&gt; `lte(duration_seconds,300)`
*   **`Contains`**: Checks if a field's value contains any of the specified values. This is a `SetFilter`.
    *   Comparator: `contains`
    *   Example: `Contains("tags", ["urgent", "production"])` -&gt; `contains(tags,urgent;production)`
*   **`ValueIn`**: Checks if a field's value is present in the specified list of values. This is a `SetFilter`.
    *   Comparator: `value_in`
    *   Example: `ValueIn("domain", ["development", "staging"])` -&gt; `value_in(domain,development;staging)`
*   **`ValueNotIn`**: Checks if a field's value is not present in the specified list of values. This is a `SetFilter`.
    *   Comparator: `value_not_in`
    *   Example: `ValueNotIn("project", ["internal", "test"])` -&gt; `value_not_in(project,internal;test)`

## Constructing Filters

To construct filters, instantiate the desired filter class with the `key` (field name) and `value` (comparison value). For set-based filters (`Contains`, `ValueIn`, `ValueNotIn`), the `values` parameter expects a list of strings.

```python
from flytekit.models.filters import Equal, GreaterThan, Contains, FilterList

# Create individual filters
name_filter = Equal("name", "my_workflow")
time_filter = GreaterThan("creation_time", "2023-01-01T00:00:00Z")
tag_filter = Contains("tags", ["data_science", "ml_model"])

# Combine filters using FilterList (logical AND)
combined_filters = FilterList([name_filter, time_filter, tag_filter])
```

## Serialization and Deserialization

The primary utility of these filters lies in their ability to be serialized into a string format for API requests and deserialized back into objects.

### Serialization to API String

Each `Filter` object and `FilterList` object provides a `to_flyte_idl()` method. This method converts the filter object into a compact string representation, which is ideal for use as a query parameter in REST API calls.

```python
from flytekit.models.filters import Equal, GreaterThan, Contains, FilterList

name_filter = Equal("name", "my_workflow")
time_filter = GreaterThan("creation_time", "2023-01-01T00:00:00Z")
tag_filter = Contains("tags", ["data_science", "ml_model"])

# Serialize individual filters
print(name_filter.to_flyte_idl())
# Output: eq(name,my_workflow)

print(time_filter.to_flyte_idl())
# Output: gt(creation_time,2023-01-01T00:00:00Z)

print(tag_filter.to_flyte_idl())
# Output: contains(tags,data_science;ml_model)

# Serialize a FilterList
combined_filters = FilterList([name_filter, time_filter, tag_filter])
print(combined_filters.to_flyte_idl())
# Output: eq(name,my_workflow)+gt(creation_time,2023-01-01T00:00:00Z)+contains(tags,data_science;ml_model)

# This string can then be used in a URL, e.g.:
# /api/v1/workflows?filters=eq(name,my_workflow)+gt(creation_time,2023-01-01T00:00:00Z)+contains(tags,data_science;ml_model)
```

### Deserialization from String

The `Filter` class provides a static factory method, `from_python_std(string)`, which can parse a filter string back into the appropriate `Filter` object. This is useful for processing filter strings received from external sources.

```python
from flytekit.models.filters import Filter

filter_string_eq = "eq(name,my_workflow)"
filter_string_contains = "contains(tags,data_science;ml_model)"

# Parse individual filter strings
parsed_eq_filter = Filter.from_python_std(filter_string_eq)
print(type(parsed_eq_filter).__name__)
# Output: Equal
print(parsed_eq_filter._key, parsed_eq_filter._value)
# Output: name my_workflow

parsed_contains_filter = Filter.from_python_std(filter_string_contains)
print(type(parsed_contains_filter).__name__)
# Output: Contains
print(parsed_contains_filter._key, parsed_contains_filter._value)
# Output: tags ['data_science', 'ml_model'] # Note: _value is parsed back to a list for SetFilter
```

## Usage Patterns and Best Practices

*   **Dynamic Query Construction**: Use these utilities to build dynamic queries based on user input or application logic, abstracting away the complexities of string formatting.
*   **API Integration**: The `to_flyte_idl()` method is the primary integration point for generating query parameters for RESTful APIs.
*   **Logical AND**: Remember that `FilterList` combines filters using a logical AND. If OR logic is required, it must be handled at a higher level or by the API endpoint itself.
*   **Value Types**: All filter values are treated as text internally. Ensure that numerical or datetime values are formatted as strings appropriately before being passed to the filter constructors. For example, timestamps should be in a consistent, sortable string format (e.g., ISO 8601).

## Important Considerations

*   **Serialization Focus**: The utilities are primarily designed for serializing Python objects into a specific string format for API consumption.
*   **No Protobuf Recovery**: The `from_flyte_idl()` method is not implemented for filters, meaning filter objects cannot be directly recovered from a protobuf representation. The `from_python_std()` method serves the purpose of parsing the string representation.
*   **Error Handling**: The `from_python_std()` method raises `ValueError` for malformed filter strings, ensuring robust parsing.
<!--
key: summary_query_filtering_utilities_c302b705-f875-4eb8-a391-be8b867660e8
type: summary_end

-->
<!--
code_unit: flytekit.models.filters.Filter
code_unit_type: class
help_text: ''
key: example_f534839e-a038-4a42-93ee-4d51c01c0f19
type: example

-->
<!--
code_unit: flytekit.models.filters.FilterList
code_unit_type: class
help_text: ''
key: example_e7fb17cf-09a8-4cd0-934a-312a466e2d8c
type: example

-->
<!--
code_unit: flytekit.models.filters.Equal
code_unit_type: class
help_text: ''
key: example_3fa3afae-ed7c-4da3-a681-04941533d28b
type: example

-->
<!--
code_unit: flytekit.models.filters.LessThan
code_unit_type: class
help_text: ''
key: example_e9d5d71d-30de-4120-95b7-c7bda04b53e8
type: example

-->