
<!--
help_text: ''
key: summary_rate_limiting_utilities_a390f692-24e5-49a9-abc9-839b3486f51b
modules:
- flytekit.utils.rate_limiter
questions_to_answer: []
type: summary

-->
# Rate Limiting Utilities

Rate limiting is a crucial mechanism for controlling the rate at which operations, such as API calls or resource access, are performed. It prevents resource exhaustion, ensures fair usage, and protects services from abuse. The Rate Limiting Utilities provide a robust and easy-to-use class for implementing per-minute rate limits within applications.

## The `RateLimiter` Class

The `RateLimiter` class is the core component for managing request rates. It allows applications to enforce a maximum number of operations within a 60-second window.

### Initialization

Instantiate the `RateLimiter` by specifying the desired maximum requests per minute (RPM).

```python
from flytekit.utils.rate_limiter import RateLimiter

# Initialize a rate limiter allowing up to 10 requests per minute
limiter = RateLimiter(rpm=10)
```

The `rpm` parameter must be an integer between 1 and 100, inclusive. Providing a value outside this range raises a `ValueError`.

### Acquiring a Permit

Before performing a rate-limited operation, an application must acquire a permit from the `RateLimiter` instance. If the rate limit has been reached, the `RateLimiter` automatically pauses execution until a permit becomes available.

#### Asynchronous Usage

For asynchronous applications, use the `acquire` method. This method is an awaitable coroutine that suspends execution until a permit is granted.

```python
import asyncio
from flytekit.utils.rate_limiter import RateLimiter

async def perform_api_call(limiter: RateLimiter, request_id: int):
    """Simulates an API call that needs to be rate-limited."""
    await limiter.acquire()
    print(f"[{asyncio.get_event_loop().time():.2f}] Performing API call {request_id}...")
    # Simulate work
    await asyncio.sleep(0.1)

async def main():
    limiter = RateLimiter(rpm=5) # 5 requests per minute
    tasks = [perform_api_call(limiter, i) for i in range(10)]
    await asyncio.gather(*tasks)

if __name__ == "__main__":
    asyncio.run(main())
```

In this example, even though 10 API calls are initiated almost simultaneously, the `RateLimiter` ensures that no more than 5 calls are "performed" within any given minute, by pausing subsequent calls.

#### Synchronous Usage

For synchronous codebases or when integrating with existing synchronous patterns, use the `sync_acquire` method. This is a blocking call that internally runs the asynchronous `acquire` method in a new event loop or an existing one if available.

```python
from flytekit.utils.rate_limiter import RateLimiter
import time

def perform_sync_operation(limiter: RateLimiter, operation_id: int):
    """Simulates a synchronous operation that needs to be rate-limited."""
    limiter.sync_acquire()
    print(f"[{time.time():.2f}] Performing synchronous operation {operation_id}...")
    # Simulate work
    time.sleep(0.1)

if __name__ == "__main__":
    limiter = RateLimiter(rpm=3) # 3 requests per minute
    for i in range(5):
        perform_sync_operation(limiter, i)
```

The `sync_acquire` method is convenient for quick integration but can introduce overhead due to managing an internal event loop. For performance-critical asynchronous applications, prefer `acquire`.

### How it Works

The `RateLimiter` implements a sliding window rate limiting mechanism using a `deque` to store timestamps of recent requests and an `asyncio.Semaphore` to control concurrent access.

When `acquire` is called:
1.  It first clears any timestamps from the internal queue that are older than 60 seconds. This ensures the window always reflects the last minute of activity.
2.  If the number of requests in the queue (after clearing old ones) is already at or above the configured RPM, the method calculates the necessary delay until the earliest request in the queue "expires" (i.e., becomes older than 60 seconds). It then pauses execution for that duration.
3.  Once a slot is available, the current timestamp is added to the queue, and the permit is granted.

This approach effectively limits the rate to `rpm` requests per minute, ensuring a smooth distribution of requests over time rather than bursty behavior.

### Important Considerations

*   **RPM Range:** The `RateLimiter` supports an RPM range of 1 to 100. This range is suitable for many common rate limiting scenarios, but it is not designed for extremely high-throughput or very low-frequency requirements.
*   **Instance Scope:** Each `RateLimiter` instance manages its own independent rate limit. To enforce a global rate limit across multiple parts of an application or different concurrent tasks, ensure all operations share the *same* `RateLimiter` instance.
*   **Asynchronous by Design:** The core `acquire` method is asynchronous. While `sync_acquire` provides a synchronous wrapper, it is generally more efficient and idiomatic to use `acquire` directly within an `asyncio` application.
*   **Precision:** The timing of delays relies on `asyncio.sleep` and system clock precision. While generally accurate, minor deviations can occur.

### Best Practices

*   **Single Instance:** For a given rate-limited resource, create a single `RateLimiter` instance and pass it around or make it globally accessible (e.g., via dependency injection or a singleton pattern). Avoid creating new `RateLimiter` instances for each operation.
*   **Error Handling:** Wrap rate-limited operations in `try...except` blocks to handle potential exceptions that might occur during the operation itself, independent of the rate limiting.
*   **Logging:** Utilize the `developer_logger` (as seen in the internal implementation) for debugging rate limiter behavior, especially when observing unexpected delays or throughput.
<!--
key: summary_rate_limiting_utilities_a390f692-24e5-49a9-abc9-839b3486f51b
type: summary_end

-->
<!--
code_unit: flytekit.utils.rate_limiter.RateLimiter
code_unit_type: class
help_text: ''
key: example_9e48c9a4-dacb-4b42-853a-eedd3b3d8c2f
type: example

-->