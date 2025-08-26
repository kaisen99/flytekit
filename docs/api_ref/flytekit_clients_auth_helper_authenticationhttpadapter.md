# AuthenticationHTTPAdapter

This class extends the HTTPAdapter to inject authentication headers into HTTP requests. It utilizes an authenticator to obtain credentials and adds them to the request headers before sending. The class also handles 401 Unauthorized responses by refreshing credentials and retrying the request.

## Attributes

- **authenticator**: object
  - The authenticator object used to provide authentication credentials.

## Constructors
def AuthenticationHTTPAdapter(authenticator: object, *args: tuple, **kwargs: dict)
-  Initializes the AuthenticationHTTPAdapter with an authenticator and optional arguments.
- **Parameters**

  - **authenticator**: object
    - The authenticator object to use for authentication.
  - ***args**: tuple
    - Positional arguments to pass to the parent class constructor.
  - ****kwargs**: dict
    - Keyword arguments to pass to the parent class constructor.



## Methods
@classmethod
def add_auth_header(request: request)
-  Adds authentication headers to the request.
        :param request: The request object to add headers to.
- **Parameters**

  - **request**: request
    - The request object to add headers to.

@classmethod
def send(request: request) - > response
-  Sends the request with added authentication headers.
        If the response returns a 401 status code, refreshes the credentials and retries the request.
        :param request: The request object to send.
        :return: The response object.
- **Parameters**

  - **request**: request
    - The request object to send.

- **Return Value**:
**response**
  - The response object.
