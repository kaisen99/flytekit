# DeviceCodeResponse

This class represents the response received from the device authorization flow endpoint. It encapsulates the device code, user code, verification URI, expiration time, and polling interval. The class provides a method to construct an instance from a JSON response.

## Attributes

- **device_code**: string
  - The device code.

- **user_code**: string
  - The user code.

- **verification_uri**: string
  - The verification URI.

- **expires_in**: integer
  - The expiration time in seconds.

- **interval**: integer
  - The interval in seconds.



## Methods
@classmethod
def from_json_response(j: typing.Dict) - > [DeviceCodeResponse](flytekit_clients_auth_token_client_devicecoderesponse)
-  Creates a DeviceCodeResponse object from a JSON response.
- **Parameters**

  - **j**: typing.Dict
    - The JSON response dictionary.

- **Return Value**:
**[DeviceCodeResponse](flytekit_clients_auth_token_client_devicecoderesponse)**
  - A DeviceCodeResponse object.
