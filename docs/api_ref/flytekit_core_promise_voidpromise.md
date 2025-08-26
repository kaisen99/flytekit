# VoidPromise

This class represents a promise for tasks that do not return any output. It is designed to handle tasks with an empty declared interface. VoidPromise prevents any interaction or comparison operations, ensuring its intended use for void-returning tasks.

## Attributes

- **task_name**: str
  - The name of the task.

- **ref**: Optional[[NodeOutput](flytekit_core_promise_nodeoutput)]
  - A reference to the node output, if available.

## Constructors
def VoidPromise(task_name: str, ref: Optional[[NodeOutput](flytekit_core_promise_nodeoutput)] = None)
-  Initializes a VoidPromise object.
- **Parameters**

  - **task_name**: str
    - The name of the task associated with this VoidPromise.
  - **ref**: Optional[[NodeOutput](flytekit_core_promise_nodeoutput)]
    - An optional reference to the NodeOutput of the task.

def VoidPromise(task_name: str, ref: Optional[[NodeOutput](flytekit_core_promise_nodeoutput)] = None)
-  This object is returned for tasks that do not return any outputs (declared interface is empty)
    VoidPromise cannot be interacted with and does not allow comparisons or any operations
- **Parameters**

  - **task_name**: str
    - The name of the task.
  - **ref**: Optional[[NodeOutput](flytekit_core_promise_nodeoutput)]
    - A reference to the output node of the task.



## Methods
@classmethod
def runs_before(args: ..., kwargs: ...)
-  This is a placeholder and should do nothing. It is only here to enable local execution of workflows
        where a task returns nothing.
- **Parameters**

  - **args**: ...
    - Placeholder for arguments.
  - **kwargs**: ...
    - Placeholder for keyword arguments.

```@classmethod
def ref()
```
-  A reference to the output node of the task.

- **Return Value**:
**Optional[[NodeOutput](flytekit_core_promise_nodeoutput)]**
  - A reference to the output node of the task.
@classmethod
def with_overrides(args: ..., kwargs: ...) - > [VoidPromise](flytekit_core_promise_voidpromise)
-  Applies overrides to the task. If the task has a reference, it applies the overrides to the referenced node.
- **Parameters**

  - **args**: ...
    - Arguments to pass to the overrides.
  - **kwargs**: ...
    - Keyword arguments to pass to the overrides.

- **Return Value**:
**[VoidPromise](flytekit_core_promise_voidpromise)**
  - The VoidPromise instance itself, allowing for method chaining.
```@classmethod
def task_name()
```
-  Returns the name of the task.

- **Return Value**:
**str**
  - The name of the task.
