# AuthorizationCode

This class represents an authorization code, a crucial component in the OAuth 2.0 authorization flow. It encapsulates the code itself and an associated state parameter. The class provides properties to access the code and state values.

## Attributes

- **code**: string
  - The authorization code.

- **state**: string
  - The state parameter to prevent cross-site request forgery.

## Constructors
def AuthorizationCode(code: any, state: any)
-  Initializes an AuthorizationCode object.
- **Parameters**

  - **code**: any
    - The authorization code.
  - **state**: any
    - The state parameter.



## Methods
```@classmethod
def code()
```
-  Returns the authorization code.

- **Return Value**:
**string**
  - The authorization code.
```@classmethod
def state()
```
-  Returns the state parameter.

- **Return Value**:
**string**
  - The state parameter.
