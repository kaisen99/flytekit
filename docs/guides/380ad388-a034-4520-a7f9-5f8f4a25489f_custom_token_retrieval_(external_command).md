
<!--
help_text: ''
key: summary_custom_token_retrieval_(external_command)_d7122be7-fff7-41e0-af50-5a897acc40e5
modules:
- flytekit.clients.auth.authenticator.CommandAuthenticator
questions_to_answer: []
type: summary

-->
# Custom Token Retrieval (External Command)

Custom Token Retrieval (External Command) provides a flexible mechanism for obtaining access tokens by executing an external process. This approach is particularly useful when token generation involves complex logic, interaction with external systems, or requires credentials that should not be directly managed by the application.

## How it Works

The core component for this functionality is the `CommandAuthenticator` class. This authenticator operates by:

1.  **Executing a Command:** It runs a specified external command or script.
2.  **Capturing Output:** It captures the standard output (stdout) of the executed command.
3.  **Token Extraction:** It treats the captured stdout as the access token.
4.  **Error Handling:** It raises an `AuthenticationError` if the command fails to execute or returns a non-zero exit code.

This design allows developers to integrate with any external tool or script capable of printing an access token to its standard output.

## Implementation with `CommandAuthenticator`

The `CommandAuthenticator` class is initialized with the command to execute and an optional header key for the token.

### Initialization

To instantiate `CommandAuthenticator`, provide a list of strings representing the command and its arguments.

```python
from your_package.auth import CommandAuthenticator # Assuming the class is in 'your_package.auth'

# Example: A simple command that echoes a token
authenticator = CommandAuthenticator(command=["echo", "my_secret_token_123"])

# Example: A more complex command, e.g., calling a shell script
authenticator = CommandAuthenticator(command=["/usr/local/bin/get_auth_token.sh", "--scope", "api"])

# Example: Specifying a custom header key (default is 'Authorization')
authenticator = CommandAuthenticator(
    command=["python", "generate_token.py"],
    header_key="X-Custom-Auth-Token"
)
```

The `command` parameter must be a non-empty list of strings. An `AuthenticationError` is raised if an empty list is provided during initialization.

### Token Retrieval

The `refresh_credentials` method within `CommandAuthenticator` handles the execution of the external command and the retrieval of the token. This method is invoked automatically when a token is needed or requires refreshing.

During execution, `CommandAuthenticator` uses `subprocess.run` with `capture_output=True`, `text=True`, and `check=True`.
*   `capture_output=True` ensures that stdout and stderr are captured.
*   `text=True` decodes stdout and stderr as text using the default encoding.
*   `check=True` causes a `subprocess.CalledProcessError` to be raised if the command returns a non-zero exit code, indicating failure.

The `refresh_credentials` method expects the external command to print the raw access token to its standard output. Any leading or trailing whitespace from the output is automatically stripped.

```python
# Example of how refresh_credentials is conceptually used (it's typically called internally)
try:
    authenticator.refresh_credentials()
    token = authenticator._creds.access_token # Access the retrieved token
    print(f"Retrieved token: {token}")
except AuthenticationError as e:
    print(f"Failed to retrieve token: {e}")
```

If the external command fails (e.g., returns a non-zero exit code), an `AuthenticationError` is raised, providing a message that includes the command executed, aiding in debugging.

## Usage Example

Consider a scenario where you have an external Python script, `generate_token.py`, that fetches a token from an identity provider and prints it to stdout.

**`generate_token.py`:**
```python
import sys
import os

# Simulate fetching a token from an external service or cache
# In a real scenario, this would involve API calls, credential handling, etc.
def get_token():
    # For demonstration, return a static token or one based on environment
    return os.getenv("MY_APP_TOKEN", "generated_token_from_script_12345")

if __name__ == "__main__":
    token = get_token()
    sys.stdout.write(token)
    sys.exit(0) # Indicate success
```

**Integrating with `CommandAuthenticator`:**
```python
import os
from your_package.auth import CommandAuthenticator # Assuming the class is in 'your_package.auth'

# Ensure the script is executable and in a known path, or provide full path
# For this example, assume generate_token.py is in the current directory
token_script_path = os.path.join(os.getcwd(), "generate_token.py")

try:
    # Initialize the authenticator with the command to run the script
    authenticator = CommandAuthenticator(command=["python", token_script_path])

    # The system will call refresh_credentials when a token is needed
    # For demonstration, we'll manually trigger it
    authenticator.refresh_credentials()
    
    # Access the credentials object to get the token
    current_token = authenticator._creds.access_token
    print(f"Successfully retrieved token: {current_token}")

except Exception as e:
    print(f"An error occurred during token retrieval: {e}")

```

## Considerations and Best Practices

*   **Security:**
    *   **Command Injection:** Be extremely cautious when constructing the `command` list, especially if any parts of the command originate from untrusted user input. Always use a list of arguments (e.g., `["python", "script.py", user_input]`) rather than a single string that could be vulnerable to shell injection.
    *   **Permissions:** Ensure the external command has only the necessary permissions to perform its task and nothing more.
    *   **Path Security:** Specify full paths to executables (e.g., `/usr/bin/python` instead of `python`) to prevent execution of malicious binaries found earlier in the system's PATH.
*   **Performance:** Spawning a new process for each token refresh introduces overhead. For high-frequency token requirements, consider if the external command can cache tokens internally or if a different authentication mechanism is more suitable.
*   **Error Handling:**
    *   **External Command Exit Code:** The `CommandAuthenticator` relies on the external command's exit code to determine success or failure. Ensure your external scripts return `0` for success and a non-zero value for any error.
    *   **Standard Error (stderr):** While `CommandAuthenticator` captures stdout for the token, it does not process stderr. Your external command should write any debugging or error messages to stderr, which can then be inspected if the command fails.
    *   **Debugging:** If token retrieval fails, execute the `command` directly in your terminal to debug its behavior and output. This helps identify issues with the script itself, its environment, or its output format.
*   **Command Output Format:** The external command *must* print only the raw access token to standard output. Any additional text, logs, or prompts on stdout will be treated as part of the token, leading to authentication failures.
*   **Token Expiration:** The `refresh_credentials` method is called when the current token is deemed invalid or expired. The external command itself should handle the logic of determining if it needs to generate a new token or return a cached, still-valid one.
*   **Environment Variables:** External commands inherit the environment variables of the process that launched them. This can be a convenient way to pass configuration or sensitive information (e.g., API keys) to the external script, but ensure sensitive data is handled securely.
<!--
key: summary_custom_token_retrieval_(external_command)_d7122be7-fff7-41e0-af50-5a897acc40e5
type: summary_end

-->
<!--
code_unit: flytekit.examples.authentication.external_command
code_unit_type: class
help_text: ''
key: example_90e7879f-100c-4901-ae7b-f7b1297b0dd4
type: example

-->