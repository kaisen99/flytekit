# BuildParams

This class specializes in defining build parameters, inheriting from RunLevelParams. It provides a &#x27;fast&#x27; option, a boolean flag that controls the serialization method, determining whether the source code is included in the image. The &#x27;fast&#x27; parameter defaults to false, enabling users to optimize image creation.

## Attributes

- **fast**: bool = False
  - Use fast serialization. The image won&#x27;t contain the source code. The value is false by default.

## Constructors
def BuildParams(fast: bool = False)
-  Use fast serialization. The image won&#x27;t contain the source code. The value is false by default.
- **Parameters**

  - **fast**: bool
    - Use fast serialization. The image won&#x27;t contain the source code. The value is false by default.



## Methods
```def fast()
```
-  Use fast serialization. The image won&#x27;t contain the source code. The value is false by default.

- **Return Value**:
**bool**
  - Whether to use fast serialization.
