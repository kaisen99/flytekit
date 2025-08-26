# Project

This class represents a project entity within the Flyte platform, serving as a container for organizing tasks, workflows, and executions. It allows for the management of project metadata, including unique identifiers, names, descriptions, and states. Key features include methods for creating active and archived project instances and properties for accessing project attributes.

## Attributes

- **id**: string
  - A globally unique identifier associated with this project

- **name**: string
  - A human-readable name for this project.

- **description**: string
  - A concise description for this project.

- **state**: int = ProjectState.ACTIVE
  - The state of this project.

## Constructors
def Project(id: Text, name: Text, description: Text, state: ProjectState = ProjectState.ACTIVE)
-  A project represents a logical grouping used to organize entities (tasks, workflows, executions) in the Flyte platform.
- **Parameters**

  - **id**: Text
    - A globally unique identifier associated with this project.
  - **name**: Text
    - A human-readable name for this project.
  - **description**: Text
    - A concise description for this project.
  - **state**: ProjectState
    - The state of this project.



## Methods
@classmethod
def archived_project(id: Text) - > [Project](flytekit_models_project_project)
-  Returns a Project object with the state set to ARCHIVED.
- **Parameters**

  - **id**: Text
    - A globally unique identifier associated with this project.

- **Return Value**:
**[Project](flytekit_models_project_project)**
  - A Project object representing an archived project.
@classmethod
def active_project(id: Text) - > [Project](flytekit_models_project_project)
-  Returns a Project object with the state set to ACTIVE.
- **Parameters**

  - **id**: Text
    - A globally unique identifier associated with this project.

- **Return Value**:
**[Project](flytekit_models_project_project)**
  - A Project object representing an active project.
```@classmethod
def id()
```
-  A globally unique identifier associated with this project.

- **Return Value**:
**Text**
  - The unique identifier of the project.
```@classmethod
def name()
```
-  A human-readable name for this project.

- **Return Value**:
**Text**
  - The name of the project.
```@classmethod
def description()
```
-  A concise description for this project.

- **Return Value**:
**Text**
  - The description of the project.
```@classmethod
def state()
```
-  The state of this project.

- **Return Value**:
**int**
  - The state of the project.
```@classmethod
def to_flyte_idl()
```
-  Converts the Project object to its corresponding Flyte IDL protobuf object.

- **Return Value**:
**flyteidl.admin.project_pb2.Project**
  - The Flyte IDL Project protobuf object.
@classmethod
def from_flyte_idl(pb2_object: flyteidl.admin.project_pb2.Project) - > [Project](flytekit_models_project_project)
-  Creates a Project object from a Flyte IDL protobuf object.
- **Parameters**

  - **pb2_object**: flyteidl.admin.project_pb2.Project
    - The Flyte IDL Project protobuf object to convert from.

- **Return Value**:
**[Project](flytekit_models_project_project)**
  - A Project object created from the Flyte IDL protobuf object.
