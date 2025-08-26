
<!--
help_text: ''
key: summary_asynchronous_execution_management_34a62602-43de-4192-bd2f-7b2aaa3aad0b
modules:
- flytekit.utils.asyn
questions_to_answer: []
type: summary

-->
## Asynchronous Execution Management

Asynchronous Execution Management provides a robust mechanism for seamlessly integrating asynchronous Python code within synchronous execution contexts. This utility is crucial when an application's primary flow is synchronous, but it needs to interact with asynchronous libraries, APIs, or I/O operations without blocking the main thread or requiring a complete rewrite to an asynchronous framework.

### Bridging Synchronous and Asynchronous Code

Python's `asyncio` library enables efficient concurrent operations, but directly calling an `async def` function from a regular `def` function is not possible without an active event loop. Asynchronous Execution Management solves this by managing dedicated `asyncio` event loops on background threads, allowing synchronous code to invoke and await asynchronous operations as if they were synchronous calls.

### Core Capabilities

The utility offers two primary ways to execute asynchronous functions from synchronous code:

#### Executing Asynchronous Functions Synchronously

Use the `run_sync` capability to execute an asynchronous coroutine and wait for its result directly within a synchronous function. This is ideal for one-off calls to asynchronous APIs.

**Usage:**

```python
import asyncio
import time
from typing import Awaitable, TypeVar, Callable, Any

# Assume _AsyncLoopManager is instantiated and available, e.g., as a singleton
# from flytekit.utils.asyn import _AsyncLoopManager
# async_loop_manager = _AsyncLoopManager()

T = TypeVar("T")
P = TypeVar("P", bound=tuple[Any, ...])

class _AsyncLoopManager:
    # ... (simplified for example)
    def run_sync(self, coro_func: Callable[..., Awaitable[T]], *args, **kwargs) -> T:
        # Internal implementation details omitted for clarity
        print(f"Running async function synchronously: {coro_func.__name__}")
        # This would internally use _TaskRunner to run the coroutine
        # For demonstration, we'll simulate a direct run if possible,
        # but in reality, it's dispatched to a background thread.
        # In a real scenario, this would block until the async function completes
        # on the background thread.
        if asyncio.iscoroutinefunction(coro_func):
            # This is a simplification; the actual implementation uses a background thread
            # and asyncio.run_coroutine_threadsafe
            loop = asyncio.get_event_loop() if asyncio.get_event_loop_policy().get_event_loop() else asyncio.new_event_loop()
            return loop.run_until_complete(coro_func(*args, **kwargs))
        else:
            raise TypeError("Expected a coroutine function")

async_loop_manager = _AsyncLoopManager() # Instantiate for example

async def fetch_data(url: str) -> str:
    """Simulates an asynchronous network request."""
    print(f"  [Async] Fetching data from {url}...")
    await asyncio.sleep(1) # Simulate I/O
    print(f"  [Async] Data from {url} fetched.")
    return f"Data from {url}"

def process_request(url: str) -> str:
    """A synchronous function that needs to call an async API."""
    print(f"[Sync] Starting processing for {url}")
    # Call the async function using run_sync
    data = async_loop_manager.run_sync(fetch_data, url)
    print(f"[Sync] Finished processing for {url}, received: {data}")
    return data

# Example usage
if __name__ == "__main__":
    start_time = time.time()
    result = process_request("https://example.com/api/data")
    end_time = time.time()
    print(f"\nTotal execution time: {end_time - start_time:.2f} seconds")
    print(f"Result: {result}")
```

In this example, `process_request` is a standard synchronous function. By using `async_loop_manager.run_sync(fetch_data, url)`, it can call the `fetch_data` coroutine and wait for its completion without needing to be an `async def` itself.

#### Decorating Synchronous Wrappers for Asynchronous Functions

The `synced` decorator transforms an asynchronous function into a synchronous callable. This is particularly useful when integrating with frameworks or libraries that expect synchronous function signatures, but the underlying implementation is asynchronous.

**Usage:**

```python
import asyncio
import time
from typing import Awaitable, TypeVar, Callable, Any

# Assume _AsyncLoopManager is instantiated and available
# from flytekit.utils.asyn import _AsyncLoopManager
# async_loop_manager = _AsyncLoopManager()

T = TypeVar("T")
P = TypeVar("P", bound=tuple[Any, ...])

class _AsyncLoopManager:
    # ... (simplified for example)
    def run_sync(self, coro_func: Callable[..., Awaitable[T]], *args, **kwargs) -> T:
        # Internal implementation details omitted for clarity
        print(f"Running async function synchronously via decorator: {coro_func.__name__}")
        # This would internally use _TaskRunner to run the coroutine
        # For demonstration, we'll simulate a direct run if possible,
        # but in reality, it's dispatched to a background thread.
        if asyncio.iscoroutinefunction(coro_func):
            loop = asyncio.get_event_loop() if asyncio.get_event_loop_policy().get_event_loop() else asyncio.new_event_loop()
            return loop.run_until_complete(coro_func(*args, **kwargs))
        else:
            raise TypeError("Expected a coroutine function")

    def synced(self, coro_func: Callable[P, Awaitable[T]]) -> Callable[P, T]:
        """Make loop run coroutine until it returns. Runs in other thread"""
        import functools

        @functools.wraps(coro_func)
        def wrapped(*args: Any, **kwargs: Any) -> T:
            return self.run_sync(coro_func, *args, **kwargs)
        return wrapped

async_loop_manager = _AsyncLoopManager() # Instantiate for example

async def send_notification(message: str, recipient: str) -> bool:
    """Simulates sending an asynchronous notification."""
    print(f"  [Async] Sending '{message}' to {recipient}...")
    await asyncio.sleep(0.5) # Simulate I/O
    print(f"  [Async] Notification sent.")
    return True

# Apply the synced decorator
@async_loop_manager.synced
async def send_notification_sync(message: str, recipient: str) -> bool:
    """Synchronous wrapper for send_notification."""
    return await send_notification(message, recipient)

def handle_user_event(user_id: str, event_details: str):
    """A synchronous event handler that uses the 'synced' async function."""
    print(f"[Sync] Handling event for user {user_id}: {event_details}")
    # Call the decorated function as if it were synchronous
    success = send_notification_sync(f"Event: {event_details}", f"user_{user_id}")
    print(f"[Sync] Notification status: {success}")

# Example usage
if __name__ == "__main__":
    start_time = time.time()
    handle_user_event("alice", "Login successful")
    end_time = time.time()
    print(f"\nTotal execution time: {end_time - start_time:.2f} seconds")
```

Here, `send_notification_sync` is an `async def` function decorated with `@async_loop_manager.synced`. This makes `send_notification_sync` callable directly from `handle_user_event` (a synchronous function) as if it were a regular synchronous function, abstracting away the asynchronous execution details.

### Architectural Overview

The Asynchronous Execution Management system relies on two core components:

*   **`_TaskRunner`**: This internal class is responsible for managing a single `asyncio` event loop on a dedicated background `threading.Thread`. When a coroutine needs to be executed, `_TaskRunner` uses `asyncio.run_coroutine_threadsafe` to submit the coroutine to its event loop and then blocks the calling thread until the coroutine completes, returning its result. Each `_TaskRunner` instance ensures its event loop runs indefinitely (`loop.run_forever()`) until explicitly stopped or the program exits.
*   **`_AsyncLoopManager`**: This class acts as a manager for `_TaskRunner` instances. It maintains a map (`_runner_map`) of `_TaskRunner` instances, typically one per unique combination of the current thread's name and process ID (`threading.current_thread().name + f"PID:{os.getpid()}"`). This design ensures that each synchronous thread that initiates an asynchronous call gets its own dedicated `_TaskRunner` and event loop, preventing contention and simplifying resource management.

When `run_sync` or a function wrapped by `synced` is called, the `_AsyncLoopManager` checks if a `_TaskRunner` already exists for the current thread. If not, it creates a new `_TaskRunner`, starts its background thread, and then uses it to execute the provided coroutine.

### Exception Handling

The `_TaskRunner` sets a custom exception handler for its `asyncio` event loop. Any unhandled exceptions that occur within coroutines running on this background loop are caught by this handler and logged as errors, preventing the background thread from silently crashing and providing visibility into issues.

### Considerations and Best Practices

*   **Thread Safety**: The system inherently handles the complexities of running an `asyncio` event loop on a separate thread and safely communicating results back to the calling thread. Developers do not need to manage thread synchronization for the event loop itself.
*   **Resource Management**: Each `_TaskRunner` registers an `atexit` hook to ensure its `asyncio` event loop is properly stopped when the program exits, preventing resource leaks.
*   **Performance Overhead**: While efficient for bridging sync/async, there is a slight overhead associated with thread creation and inter-thread communication. For applications that are entirely asynchronous, it is generally more performant to use `asyncio` directly. This utility shines when incremental adoption of async features is required in an existing synchronous codebase.
*   **Many Runners Warning**: The `_AsyncLoopManager` logs a warning if more than 500 `_TaskRunner` instances are created. This can indicate a potential issue, such as runaway recursion or an unexpected pattern of thread creation that leads to an excessive number of background event loops. While not necessarily an error, it's a signal to investigate the application's thread usage.
*   **Daemon Threads**: The background threads created by `_TaskRunner` are daemon threads. This means they will automatically terminate when the main program exits, even if they are still running.
*   **Avoid Deep Synchronous Recursion into Async**: While the system handles bridging, repeatedly calling `run_sync` or `synced` in a deeply recursive synchronous call stack that then calls back into async functions can lead to complex debugging scenarios and potentially hit the "many runners" warning. Design your application to minimize such patterns.
<!--
key: summary_asynchronous_execution_management_34a62602-43de-4192-bd2f-7b2aaa3aad0b
type: summary_end

-->
<!--
code_unit: flytekit.utils.asyn._TaskRunner
code_unit_type: class
help_text: ''
key: example_85f38ec5-c43e-4405-a672-384d29cc6b64
type: example

-->
<!--
code_unit: flytekit.utils.asyn._AsyncLoopManager
code_unit_type: class
help_text: ''
key: example_be53721d-ad3a-4014-93c4-c31f3339fe0d
type: example

-->