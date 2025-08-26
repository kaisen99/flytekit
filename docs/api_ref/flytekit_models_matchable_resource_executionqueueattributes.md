# ExecutionQueueAttributes

This class manages execution queue attributes, specifically tags, for tasks within a Flyte environment. It facilitates the assignment of execution queues based on project, domain, and workflow. The class provides methods for converting to and from Flyte IDL objects, enabling interoperability with the Flyte backend.

## Attributes

- **tags**: list[Text]
  - Tags used for assigning execution queues for tasks matching a project, domain and optionally, workflow.

## Constructors
def ExecutionQueueAttributes(tags: list[Text])
-  Tags used for assigning execution queues for tasks matching a project, domain and optionally, workflow.
- **Parameters**

  - **tags**: list[Text]
    - 

def ExecutionQueueAttributes(tags: list[Text])
-  Tags used for assigning execution queues for tasks matching a project, domain and optionally, workflow.
- **Parameters**

  - **tags**: list[Text]
    - 



## Methods
```@classmethod
def tags()
```
-  

- **Return Value**:
**list[Text]**
```@classmethod
def to_flyte_idl()
```
-  

- **Return Value**:
**flyteidl.admin.matchable_resource_pb2.ExecutionQueueAttributes**
@classmethod
def from_flyte_idl(pb2_object: flyteidl.admin.matchable_resource_pb2.ExecutionQueueAttributes) - > [ExecutionQueueAttributes](flytekit_models_matchable_resource_executionqueueattributes)
-  
- **Parameters**

  - **pb2_object**: flyteidl.admin.matchable_resource_pb2.ExecutionQueueAttributes
    - 

- **Return Value**:
**[ExecutionQueueAttributes](flytekit_models_matchable_resource_executionqueueattributes)**
