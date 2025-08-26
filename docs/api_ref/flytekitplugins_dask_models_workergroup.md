# WorkerGroup

This class configures a group of Dask workers. It allows specifying the number of workers, an optional image for worker pods, and resource allocation. The class implements the FlyteIdlEntity interface for serialization.

## Attributes

- **number_of_workers**: int
  - Number of workers in the group

- **image**: Optional[str]
  - Optional image to use for the pods of the worker group

- **resources**: Optional[task.Resources]
  - Optional resources to use for the pods of the worker group

## Constructors
def WorkerGroup(number_of_workers: int, image: Optional[str] = None, resources: Optional[task.Resources] = None)
-  Configuration for a dask worker group
- **Parameters**

  - **number_of_workers**: int
    - Number of workers in the group
  - **image**: Optional[str]
    - Optional image to use for the pods of the worker group
  - **resources**: Optional[task.Resources]
    - Optional resources to use for the pods of the worker group

def WorkerGroup(number_of_workers: int, image: Optional[str] = None, resources: Optional[task.Resources] = None)
-  Configuration for a dask worker group
- **Parameters**

  - **number_of_workers**: int
    - Number of workers in the group
  - **image**: Optional[str]
    - Optional image to use for the pods of the worker group
  - **resources**: Optional[task.Resources]
    - Optional resources to use for the pods of the worker group



## Methods
```@classmethod
def number_of_workers()
```
-  Number of workers in the group

- **Return Value**:
**Optional[int]**
  - Optional number of workers for the worker group
```@classmethod
def image()
```
-  Optional image to use for the pods of the worker group

- **Return Value**:
**Optional[str]**
  - The optional image to use for the worker pods
```@classmethod
def resources()
```
-  Optional resources to use for the pods of the worker group

- **Return Value**:
**Optional[task.Resources]**
  - Optional resources to use for the worker pods
```@classmethod
def to_flyte_idl()
```
-  Configuration for a dask worker group

- **Return Value**:
**dask_task.DaskWorkerGroup**
  - The dask cluster serialized to protobuf
