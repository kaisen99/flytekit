# Sql

This class represents a SQL task definition within a Flyte workflow. It encapsulates the SQL statement and the dialect to be used for execution. The class provides methods to convert to and from Flyte IDL objects for serialization and deserialization.

## Attributes

- **statement**: string = None
  - This defines a kubernetes pod target. It will build the pod target during task execution

- **dialect**: int = 0
  - This defines a kubernetes pod target. It will build the pod target during task execution

## Constructors
def Sql(statement: string = None, dialect: int = 0)
-  This defines a kubernetes pod target. It will build the pod target during task execution
- **Parameters**

  - **statement**: string
    - 
  - **dialect**: int
    - 



## Methods
```@classmethod
def statement()
```
-  This defines a kubernetes pod target. It will build the pod target during task execution

- **Return Value**:
**str**
  - This defines a kubernetes pod target. It will build the pod target during task execution
```@classmethod
def dialect()
```
-  This defines a kubernetes pod target. It will build the pod target during task execution

- **Return Value**:
**int**
  - This defines a kubernetes pod target. It will build the pod target during task execution
```@classmethod
def to_flyte_idl()
```
-  This defines a kubernetes pod target. It will build the pod target during task execution

- **Return Value**:
**str**
  - This defines a kubernetes pod target. It will build the pod target during task execution
@classmethod
def from_flyte_idl(pb2_object: _core_task.Sql) - > str
-  This defines a kubernetes pod target. It will build the pod target during task execution
- **Parameters**

  - **pb2_object**: _core_task.Sql
    - This defines a kubernetes pod target. It will build the pod target during task execution

- **Return Value**:
**str**
  - This defines a kubernetes pod target. It will build the pod target during task execution
