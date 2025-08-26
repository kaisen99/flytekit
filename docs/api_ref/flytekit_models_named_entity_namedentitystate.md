# NamedEntityState

This class represents the state of a named entity, providing constants for active and archived states. It offers a class method to convert an integer enum value to a human-readable string representation. The class facilitates managing and interpreting the status of named entities within a system.

## Attributes

- **ACTIVE**: int = _common.NAMED_ENTITY_ACTIVE
  - Represents the active state of a named entity.

- **ARCHIVED**: int = _common.NAMED_ENTITY_ARCHIVED
  - Represents the archived state of a named entity.



## Methods
@classmethod
def enum_to_string(val: int) - > string
-  Converts the enum value to its string representation.
- **Parameters**

  - **val**: int
    - The integer value of the enum.

- **Return Value**:
**string**
  - The string representation of the enum value.
