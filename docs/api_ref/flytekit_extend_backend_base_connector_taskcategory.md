# TaskCategory

This class represents a category for tasks, encapsulating a name and a version. It provides methods for equality comparison and hashing, enabling its use in sets and dictionaries. The class also includes properties to access the category&#x27;s name and version, and a string representation for easy identification.

## Attributes

- **name**: str
  - The name of the task category.

- **version**: int = 0
  - The version of the task category.

## Constructors
def TaskCategory(name: string, version: integer = 0)
-  Initializes a TaskCategory object.
- **Parameters**

  - **name**: string
    - The name of the task category.
  - **version**: integer
    - The version of the task category.



## Methods
```@classmethod
def name()
```
-  Gets the name of the TaskCategory.

- **Return Value**:
**str**
  - The name of the TaskCategory.
```@classmethod
def version()
```
-  Gets the version of the TaskCategory.

- **Return Value**:
**int**
  - The version of the TaskCategory.
