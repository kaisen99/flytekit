# FlyteContextManager

This class manages the execution context within Flytekit, handling global state for compilation and execution. It provides a singleton stack for managing contexts, including compilation and execution states. The class supports context initialization, pushing, and popping, and it utilizes a context manager for managing the context lifecycle.

## Attributes

- **signal_handlers**: typing.List[typing.Callable[[int, FrameType], typing.Any]] = []
  - List of signal handlers.

## Constructors
```def FlyteContextManager()
```
-  Re-initializes the context and erases the entire context



## Methods
@classmethod
def add_signal_handler(handler: typing.Callable[[int, FrameType], typing.Any])
-  Adds a signal handler to the list of handlers that will be called when a SIGINT signal is received.
- **Parameters**

  - **handler**: typing.Callable[[int, FrameType], typing.Any]
    - The signal handler function to add.

@classmethod
def get_origin_stackframe(limit: int = 2) - > traceback.FrameSummary
-  Retrieves the stack frame of the caller. This is useful for debugging and understanding the execution flow.
- **Parameters**

  - **limit**: int
    - The number of stack frames to go up. Defaults to 2.

- **Return Value**:
**traceback.FrameSummary**
  - The stack frame summary of the origin.
```@classmethod
def current_context()
```
-  Returns the current Flyte context. If no context is found, it initializes a new one.

- **Return Value**:
**[FlyteContext](flytekit_core_context_manager_flytecontext)**
  - The current Flyte context.
@classmethod
def push_context(ctx: [FlyteContext](flytekit_core_context_manager_flytecontext), f: Optional[traceback.FrameSummary]) - > [FlyteContext](flytekit_core_context_manager_flytecontext)
-  Pushes a new Flyte context onto the stack. This is typically used when entering a new execution scope.
- **Parameters**

  - **ctx**: [FlyteContext](flytekit_core_context_manager_flytecontext)
    - The Flyte context to push.
  - **f**: Optional[traceback.FrameSummary]
    - The stack frame to associate with the context. Defaults to the origin stack frame.

- **Return Value**:
**[FlyteContext](flytekit_core_context_manager_flytecontext)**
  - The context that was pushed.
```@classmethod
def pop_context()
```
-  Pops the current Flyte context from the stack. This is typically used when exiting an execution scope.

- **Return Value**:
**[FlyteContext](flytekit_core_context_manager_flytecontext)**
  - The context that was popped.
@classmethod
def with_context(b: FlyteContext.Builder) - > Generator[[FlyteContext](flytekit_core_context_manager_flytecontext), None, None]
-  A context manager that pushes a new Flyte context onto the stack upon entry and pops it upon exit. It also handles potential context leaks from conditional execution.
- **Parameters**

  - **b**: FlyteContext.Builder
    - The builder used to create the Flyte context.

- **Return Value**:
**Generator[[FlyteContext](flytekit_core_context_manager_flytecontext), None, None]**
  - A generator that yields the pushed context.
```@classmethod
def size()
```
-  Returns the current number of contexts on the stack.

- **Return Value**:
**int**
  - The number of contexts on the stack.
```@classmethod
def initialize()
```
-  Re-initializes the Flyte context and erases the entire context stack. It sets up default configurations for local execution, including signal handling for SIGINT.

