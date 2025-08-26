# StandardIgnore

This class extends the base Ignore class to provide standard file and directory exclusion functionality. It utilizes a list of patterns to determine which paths should be ignored. The class can be initialized with custom ignore patterns or defaults to a set of standard patterns.

## Attributes

- **patterns**: Optional[List[str]] = STANDARD_IGNORE_PATTERNS
  - List of ignore patterns.

## Constructors
def StandardIgnore(root: Path, patterns: Optional[List[str]] = None)
-  Retains the standard ignore functionality that previously existed. Could in theory by fed with custom ignore patterns from cli.
- **Parameters**

  - **root**: Path
    - The root directory to consider for ignoring files.
  - **patterns**: Optional[List[str]]
    - A list of ignore patterns. If not provided, defaults to STANDARD_IGNORE_PATTERNS.



