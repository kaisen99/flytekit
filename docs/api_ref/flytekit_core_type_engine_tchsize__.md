# tchSize:


This class, BatchSize, is designed to annotate FlyteDirectory objects, enabling batch-wise handling of directory contents during download and upload operations. It allows for the specification of batch sizes, controlling the number of files processed in each chunk. This approach optimizes memory usage and improves efficiency when dealing with large directories.

## Attributes

- **val**: int = None
  - This is used to annotate a FlyteDirectory when we want to download/upload the contents of the directory in batches.

## Constructors
def tchSize:
(val: int)
-  This is used to annotate a FlyteDirectory when we want to download/upload the contents of the directory in batches.
- **Parameters**

  - **val**: int
    - The batch size value.

def tchSize:
(val: int)
-  Initializes the BatchSize object with a given integer value.
- **Parameters**

  - **val**: int
    - The integer value representing the batch size.



## Methods
```def val()
```
-  Gets the integer value representing the batch size.

- **Return Value**:
**int**
  - The integer value of the batch size.
