# PickledEntityMetadata

This class stores metadata for a pickled entity, specifically including the Python version used during pickling. It provides a way to track the Python environment associated with a pickled object. The primary attribute is the Python version string, which is essential for compatibility checks when loading the pickled entity.

## Attributes

- **python_version**: string
  - The Python version string (e.g. &quot;3.12.0&quot;) used to create the pickle

## Constructors
def PickledEntityMetadata(python_version: str)
-  Initializes PickledEntityMetadata with the Python version.
- **Parameters**

  - **python_version**: str
    - The Python version string (e.g. &quot;3.12.0&quot;) used to create the pickle.



