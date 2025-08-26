
<!--
help_text: ''
key: summary_mock_statistics_client_6db8099e-42b2-4142-8b71-efdb3ff4de5e
modules:
- flytekit.core.mock_stats
questions_to_answer: []
type: summary

-->
The Mock Statistics Client provides an in-memory implementation for collecting and inspecting application metrics. This utility is designed primarily for testing and development environments where a full-fledged statistics client (like DataDog or Prometheus) is not available or desired. It allows developers to verify that metrics are being emitted correctly without requiring external dependencies or network calls.

### Core Functionality

The `MockStats` class serves as the central component for interacting with mock statistics. It captures metric names, values, and associated tags in memory, making them accessible for assertion and inspection.

#### Initialization

Initialize a `MockStats` instance with an optional `scope` and default `tags`. The `scope` prefixes all metrics recorded by this instance, providing a way to namespace metrics.

```python
from flytekit.core.mock_stats import MockStats

# Initialize with a global scope
stats_client = MockStats(scope="my_app.service_a")

# Initialize with default tags
stats_client_with_tags = MockStats(tags={"env": "test"})
```

#### Counters: Incrementing and Decrementing Metrics

Use `incr` to increment a counter metric and `decr` to decrement it. These methods are suitable for tracking events or counts that accumulate over time.

*   `incr(metric, count=1, tags=None)`: Increases the value of `metric` by `count`.
*   `decr(metric, count=1, tags=None)`: Decreases the value of `metric` by `count`.

Both methods accept optional `tags` to provide additional context for the metric.

```python
stats = MockStats(scope="my_workflow")

# Increment a simple counter
stats.incr("task_executions")
assert stats.current_value("task_executions") == 1

# Increment by a specific count with tags
stats.incr("api_calls", count=5, tags={"endpoint": "/data", "status": "success"})
assert stats.current_value("api_calls") == 5
assert stats.current_tags("api_calls") == {"endpoint": "/data", "status": "success"}

# Decrement a counter
stats.decr("pending_items", count=2)
assert stats.current_value("pending_items") == -2
```

#### Gauges: Setting Metric Values

Use `gauge` to record an instantaneous value for a metric. This is useful for tracking values that can go up or down, such as queue sizes, memory usage, or current temperatures.

*   `gauge(metric, value, tags=None)`: Sets the value of `metric` to `value`.

```python
stats = MockStats()

# Record a gauge metric
stats.gauge("queue_size", 10)
assert stats.current_value("queue_size") == 10

# Update the gauge value
stats.gauge("queue_size", 5)
assert stats.current_value("queue_size") == 5

# Record with tags
stats.gauge("cpu_utilization", 75.5, tags={"host": "server-1"})
assert stats.current_value("cpu_utilization") == 75.5
assert stats.current_tags("cpu_utilization") == {"host": "server-1"}
```

#### Timers: Measuring Durations

The `timer` method provides a convenient way to measure the duration of an operation using a context manager. It records the elapsed time as a gauge metric.

*   `timer(metric, tags=None)`: Returns a context manager that, upon exiting, records the duration of the enclosed block as a gauge metric.

```python
import time
from flytekit.core.mock_stats import MockStats

stats = MockStats(scope="my_component")

# Measure the execution time of a block
with stats.timer("data_processing_time"):
    time.sleep(0.1) # Simulate some work

# The recorded value is a timedelta object
processing_time = stats.current_value("data_processing_time")
assert processing_time is not None
assert processing_time.total_seconds() >= 0.1

# Measure with tags
with stats.timer("db_query_latency", tags={"query_type": "read"}):
    time.sleep(0.05)

db_latency = stats.current_value("db_query_latency")
assert db_latency.total_seconds() >= 0.05
assert stats.current_tags("db_query_latency") == {"query_type": "read"}
```

Note that the `timing` method is currently not implemented and will issue a warning if called. Use `timer` for measuring durations.

#### Retrieving Current Metric Values and Tags

After recording metrics, retrieve their current values and associated tags using `current_value` and `current_tags`.

*   `current_value(metric)`: Returns the current value of the specified `metric`. Returns `None` if the metric has not been recorded.
*   `current_tags(metric)`: Returns the dictionary of tags associated with the specified `metric`. Returns `None` if the metric has not been recorded or has no tags.

```python
stats = MockStats()
stats.incr("requests_total", tags={"status": "200"})
stats.gauge("memory_usage_mb", 512)

# Retrieve values
assert stats.current_value("requests_total") == 1
assert stats.current_value("memory_usage_mb") == 512
assert stats.current_value("non_existent_metric") is None

# Retrieve tags
assert stats.current_tags("requests_total") == {"status": "200"}
assert stats.current_tags("memory_usage_mb") == {} # Gauge was set without explicit tags
assert stats.current_tags("non_existent_metric") is None
```

### Usage Patterns and Best Practices

The Mock Statistics Client is invaluable for unit and integration testing.

*   **Testing Metric Emission:** In tests, instantiate `MockStats` and pass it to the component under test. After the component executes, assert on the values and tags collected by the `MockStats` instance.

    ```python
    from flytekit.core.mock_stats import MockStats

    class MyService:
        def __init__(self, stats_client):
            self._stats = stats_client

        def process_data(self, data):
            with self._stats.timer("data_processing_duration"):
                # Simulate data processing
                if data:
                    self._stats.incr("data_processed_count")
                    self._stats.gauge("last_data_size", len(data))
                else:
                    self._stats.incr("empty_data_count", tags={"reason": "no_input"})

    # In a test file:
    def test_my_service_metrics():
        mock_stats = MockStats(scope="my_service")
        service = MyService(mock_stats)

        service.process_data("some_data")
        assert mock_stats.current_value("my_service.data_processed_count") == 1
        assert mock_stats.current_value("my_service.last_data_size") == 9
        assert mock_stats.current_value("my_service.data_processing_duration") is not None

        service.process_data("")
        assert mock_stats.current_value("my_service.empty_data_count") == 1
        assert mock_stats.current_tags("my_service.empty_data_count") == {"reason": "no_input"}
    ```

*   **Isolation:** Each `MockStats` instance maintains its own independent set of records. This allows for isolated testing of different components or scenarios without metric pollution.

### Limitations

*   **In-Memory Only:** All metrics are stored in memory and are lost when the `MockStats` instance is garbage collected or the application exits. This client is not suitable for production monitoring or persistent metric storage.
*   **No Aggregation or Reporting:** The client does not perform any aggregation (e.g., sum of all `incr` calls across multiple instances) or reporting to external systems. It simply stores the last recorded value for gauges and the cumulative sum for counters.
*   **`timing` Method:** The `timing` method is a placeholder and does not implement any functionality. Use the `timer` context manager for measuring durations.
<!--
key: summary_mock_statistics_client_6db8099e-42b2-4142-8b71-efdb3ff4de5e
type: summary_end

-->
<!--
code_unit: flytekit.core.mock_stats.MockStats
code_unit_type: class
help_text: ''
key: example_2d77371e-12ac-47ba-9aa5-b449cc14c41a
type: example

-->
<!--
code_unit: flytekit.core.mock_stats._Timer
code_unit_type: class
help_text: ''
key: example_add496fa-2f67-4dd7-b773-bcebf8246383
type: example

-->