# Model

This class defines the configuration for a model within a Kubernetes pod template. It encapsulates the model&#x27;s name, memory allocation, CPU allocation, and the model file itself. The class provides a structured way to represent and manage model-specific resource requirements within a Kubernetes environment.

## Attributes

- **name**: string
  - The name of the model.

- **mem**: string = 500Mi
  - The amount of memory allocated for the model, specified as a string.

- **cpu**: integer = read_only
  - The number of CPU cores allocated for the model.

- **modelfile**: Optional[string]
  - The actual model file as a JSON-serializable string. This represents the file content.

## Constructors
def Model(name: str, mem: str = 500Mi, cpu: int = 1, modelfile: Optional[str] = None)
-  Represents the configuration for a model used in a Kubernetes pod template.
- **Parameters**

  - **name**: str
    - The name of the model.
  - **mem**: str
    - The amount of memory allocated for the model, specified as a string.
  - **cpu**: int
    - The number of CPU cores allocated for the model.
  - **modelfile**: Optional[str]
    - The actual model file as a JSON-serializable string. This represents the file content.

def Model(name: str, mem: str = 500Mi, cpu: int = 1, modelfile: Optional[str] = None)
-  Initializes the Model configuration.
- **Parameters**

  - **name**: str
    - The name of the model.
  - **mem**: str
    - The amount of memory allocated for the model, specified as a string.
  - **cpu**: int
    - The number of CPU cores allocated for the model.
  - **modelfile**: Optional[str]
    - The actual model file as a JSON-serializable string. This represents the file content.



