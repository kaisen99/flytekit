# OAuthCallbackHandler

This class extends BaseHTTPRequestHandler to manage a callback URL for receiving authorization tokens. It processes GET requests to a specified redirect path, extracting authorization codes and states from the URL. The class then uses this information to handle the authorization process.

## Constructors
```def OAuthCallbackHandler()
```
-  A simple wrapper around BaseHTTPServer.BaseHTTPRequestHandler that handles a callback URL that accepts an authorization token.



## Methods
```@classmethod
def do_GET()
```
-  Handles GET requests to the callback URL.

@classmethod
def handle_login(data: dict)
-  Processes the authorization code and state received from the OAuth provider.
- **Parameters**

  - **data**: dict
    - A dictionary containing the authorization code and state.

