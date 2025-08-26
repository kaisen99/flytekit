# FilterList

This class represents a list of filters that are combined using a logical AND operation. It provides a method to convert the filter list into a Flyte IDL representation, suitable for use in API requests. The class relies on individual Filter objects and utilizes their `to_flyte_idl` methods for serialization.

## Attributes

- **filter_list**: list[[Filter](flytekit_models_filters_filter)] = None
  - List of filters to AND together

## Constructors
def FilterList(filter_list: list[[Filter](flytekit_models_filters_filter)])
-  Initializes a FilterList object.
- **Parameters**

  - **filter_list**: list[[Filter](flytekit_models_filters_filter)]
    - List of filters to AND together.



## Methods
```@classmethod
def to_flyte_idl()
```
-  For supporting the auto-generated REST API, filters must be dumped to a string for representation as GET params.

- **Return Value**:
**string**
  - Text
```@classmethod
def from_flyte_idl()
```
-  Filters are never recovered from a protobuf.

- **Return Value**:
**string**
  - NotImplementedError
