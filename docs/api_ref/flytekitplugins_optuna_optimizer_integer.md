# Integer

This class represents an integer number with a defined range and step. It provides functionalities for iterating through a sequence of integers within the specified bounds. The class also includes an option to enable logging during operations.

## Attributes

- **low**: int
  - low

- **high**: int
  - high

- **step**: int = 1
  - step

- **log**: bool = False
  - log

## Constructors
def Integer(low: int, high: int, step: int = 1, log: bool = False)
-  Initializes an Integer object.
- **Parameters**

  - **low**: int
    - The lower bound of the integer range.
  - **high**: int
    - The upper bound of the integer range.
  - **step**: int
    - The step value for the integer range.
  - **log**: bool
    - A boolean indicating whether to use logarithmic spacing.



