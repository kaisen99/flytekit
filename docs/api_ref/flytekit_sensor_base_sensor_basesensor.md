# BaseSensor

This class serves as the foundational abstract class for all sensors within the system. Sensors are designed to continuously monitor for specific conditions, triggering completion upon fulfillment. It leverages the AsyncConnectorExecutorMixin and PythonTask, providing a framework for asynchronous execution and integration with the task management system.

## Constructors
def BaseSensor(name: str, timeout: Optional[[Union](flytekit_models_literals_union)[datetime.timedelta, int]] = None, sensor_config: Optional[T] = None, task_type: str = &quot;sensor&quot;, kwargs: **kwargs)
-  Base class for all sensors. Sensors are tasks that are designed to run forever and periodically check for some condition to be met. When the condition is met, the sensor will complete. Sensors are designed to be run by the connector and not by the Flyte engine.
- **Parameters**

  - **name**: str
    - The name of the sensor.
  - **timeout**: Optional[[Union](flytekit_models_literals_union)[datetime.timedelta, int]]
    - The timeout for the sensor.
  - **sensor_config**: Optional[T]
    - The configuration for the sensor.
  - **task_type**: str
    - The type of the task.
  - **kwargs**: **kwargs
    - Additional keyword arguments.



## Methods
@classmethod
def poke(kwargs: dict) - > boolean
-  This method should be overridden by the user to implement the actual sensor logic. This method should return ``True`` if the sensor condition is met, else ``False``.
- **Parameters**

  - **kwargs**: dict
    - Arbitrary keyword arguments.

- **Return Value**:
**boolean**
  - True if the sensor condition is met, else False.
@classmethod
def get_custom(settings: [SerializationSettings](flytekit_configuration_serializationsettings)) - > dict
-  Returns a dictionary containing custom metadata for the sensor, including its module, name, and configuration.
- **Parameters**

  - **settings**: [SerializationSettings](flytekit_configuration_serializationsettings)
    - Serialization settings.

- **Return Value**:
**dict**
  - A dictionary containing the sensor&#x27;s custom metadata.
