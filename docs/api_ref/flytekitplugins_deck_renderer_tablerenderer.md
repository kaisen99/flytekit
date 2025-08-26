# TableRenderer

This class transforms a pandas DataFrame into an HTML table format. It provides the functionality to customize the table&#x27;s appearance, including header labels and table width. The class utilizes the pandas DataFrame&#x27;s built-in `to_html` method and incorporates CSS styling for enhanced presentation.



## Methods
@classmethod
def to_html(df: pd.DataFrame, header_labels: Optional[List], table_width: Optional[int]) - > string
-  Convert a pandas DataFrame into an HTML table.
- **Parameters**

  - **df**: pd.DataFrame
    - The pandas DataFrame to convert.
  - **header_labels**: Optional[List]
    - Optional list of custom header labels for the table columns.
  - **table_width**: Optional[int]
    - Optional width of the table in pixels.

- **Return Value**:
**string**
  - An HTML string representing the table.
