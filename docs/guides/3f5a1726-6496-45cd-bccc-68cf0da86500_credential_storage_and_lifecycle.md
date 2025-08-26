
<!--
help_text: ''
key: summary_credential_storage_and_lifecycle_52c6a526-b507-4fa6-b8b5-154ff10ee1dd
modules:
- flytekit.clients.auth.keyring.KeyringStore
- flytekit.clients.auth.keyring.Credentials
questions_to_answer: []
type: summary

-->
Credential Storage and Lifecycle

The system manages authentication tokens securely and persistently across sessions. This involves encapsulating various token types and interacting with the underlying operating system's secure storage mechanisms.

### Credential Representation

The `Credentials` object encapsulates all necessary authentication tokens and associated metadata. It serves as a unified container for handling user or service authentication details.

A `Credentials` object includes:
*   `access_token` (str): The primary token used for authenticating API requests.
*   `refresh_token` (Optional[str]): A token used to obtain new access tokens when the current one expires, avoiding re-authentication.
*   `id_token` (Optional[str]): An OpenID Connect ID Token, typically a JWT, containing identity information about the user.
*   `for_endpoint` (str): A unique identifier, defaulting to "flyte-default", that specifies the target service or endpoint for which these credentials are valid. This value is crucial for storing and retrieving credentials correctly.
*   `expires_in` (Optional[int]): The lifetime in seconds of the access token. This field is part of the `Credentials` object but is not directly persisted by the storage mechanism; it is typically used by higher-level logic for token expiration and refresh.

### Secure Credential Storage

The `KeyringStore` component provides methods for securely storing, retrieving, and deleting authentication tokens using the operating system's native keyring service. This approach leverages platform-specific security features to protect sensitive credential data.

The `KeyringStore` interacts with the `keyring` library, which abstracts away the complexities of different OS keyrings (e.g., macOS Keychain, Windows Credential Manager, Linux Secret Service).

#### Storing Credentials

To persist credentials securely, use the `KeyringStore.store()` method. This method takes a `Credentials` object and stores its `access_token`, `refresh_token`, and `id_token` in the keyring, associated with the `for_endpoint` identifier.

If a `refresh_token` or `id_token` is not present in the `Credentials` object, it is not stored.

```python
from your_module import KeyringStore, Credentials

# Example: Storing new credentials
new_credentials = Credentials(
    access_token="your_access_token_here",
    refresh_token="your_refresh_token_here",
    for_endpoint="my-service-endpoint",
    id_token="your_id_token_here"
)
stored_credentials = KeyringStore.store(new_credentials)
print(f"Credentials stored for endpoint: {stored_credentials.for_endpoint}")
```

**Important Consideration:** If the underlying keyring service is unavailable (e.g., the `keyring` library cannot find a suitable backend), a `NoKeyringError` occurs. In such cases, the tokens are not cached persistently, and a debug log message indicates the failure. The `store` operation still returns the `Credentials` object, but the tokens will not be available in subsequent sessions.

#### Retrieving Credentials

To retrieve previously stored credentials, use the `KeyringStore.retrieve()` method, providing the `for_endpoint` identifier.

```python
from your_module import KeyringStore

# Example: Retrieving credentials
endpoint_id = "my-service-endpoint"
retrieved_credentials = KeyringStore.retrieve(endpoint_id)

if retrieved_credentials:
    print(f"Access Token: {retrieved_credentials.access_token}")
    print(f"Refresh Token: {retrieved_credentials.refresh_token}")
    print(f"ID Token: {retrieved_credentials.id_token}")
else:
    print(f"No credentials found for endpoint: {endpoint_id}")
```

The `retrieve` method returns a `Credentials` object if an `access_token` or `id_token` is found for the specified endpoint. If neither is found, or if the keyring service is unavailable, it returns `None`.

#### Deleting Credentials

To remove credentials from the secure storage, use the `KeyringStore.delete()` method, specifying the `for_endpoint` identifier. This action removes the `access_token`, `refresh_token`, and `id_token` associated with that endpoint.

```python
from your_module import KeyringStore

# Example: Deleting credentials
endpoint_to_delete = "my-service-endpoint"
KeyringStore.delete(endpoint_to_delete)
print(f"Credentials deleted for endpoint: {endpoint_to_delete}")
```

If a specific token (e.g., `refresh_token`) is not found during deletion, a `PasswordDeleteError` is caught and logged at debug level, but the deletion process continues for other tokens. Similar to storage and retrieval, `NoKeyringError` prevents deletion, and a debug log message indicates the issue.

### Integration and Best Practices

*   **Endpoint Identification:** Always use a consistent and unique `for_endpoint` string for each set of credentials to ensure correct storage and retrieval. This string acts as the key for your credentials in the keyring.
*   **Error Handling:** Implement robust error handling around `KeyringStore` operations, especially considering the `NoKeyringError`. If the keyring is unavailable, your application should gracefully fall back to alternative authentication methods (e.g., prompting the user for credentials) or inform the user about the lack of persistent storage.
*   **Token Refresh Logic:** While `KeyringStore` handles persistence, the responsibility for checking token expiration (`expires_in`) and initiating token refresh using the `refresh_token` lies with higher-level authentication logic within your application. After a successful refresh, the new `access_token` and potentially new `refresh_token` should be stored again using `KeyringStore.store()`.
*   **Security:** The use of the system keyring provides a strong security posture by leveraging OS-level protection. Avoid storing sensitive credentials directly in plain text files or environment variables for long-term persistence.
<!--
key: summary_credential_storage_and_lifecycle_52c6a526-b507-4fa6-b8b5-154ff10ee1dd
type: summary_end

-->
<!--
code_unit: flytekit.examples.authentication.credential_management
code_unit_type: class
help_text: ''
key: example_34edc50d-49a9-4759-8839-3686b72ac6c8
type: example

-->