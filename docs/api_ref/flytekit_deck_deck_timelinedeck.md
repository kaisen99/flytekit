# TimeLineDeck

This class specializes in visualizing the execution time of task components using a timeline. It inherits from a base class and delays HTML rendering until the `html` property is accessed, ensuring a complete dataset for meaningful visualizations. The class utilizes a list to store time information and generates an HTML representation of the timeline.

## Attributes

- **time_info**: list = []
  - Stores information about the execution time of each part of a task.

## Constructors
def TimeLineDeck(name: str, html: Optional[str] = &quot;&quot;, auto_add_to_deck: bool = True)
-  Initializes a new instance of the TimeLineDeck class. This constructor sets up the basic properties of the timeline deck, including its name and an optional HTML representation, and prepares it to store time information.
- **Parameters**

  - **name**: str
    - The name of the timeline deck.
  - **html**: Optional[str]
    - An optional initial HTML string for the deck.
  - **auto_add_to_deck**: bool
    - A boolean indicating whether to automatically add the deck to a parent deck.



## Methods
@classmethod
def append_time_info(info: dict)
-  Appends time information to the internal list. This method is used to record the execution time of different parts of a task.
- **Parameters**

  - **info**: dict
    - A dictionary containing time information. It is asserted to be a dictionary before appending.

