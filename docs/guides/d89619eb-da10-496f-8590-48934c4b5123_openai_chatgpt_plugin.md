
<!--
help_text: ''
key: summary_openai_chatgpt_plugin_6f0a9bc0-a7be-4267-ac64-8c594b019b0b
modules:
- flytekitplugins.openai.chatgpt.connector
- flytekitplugins.openai.chatgpt.task
questions_to_answer: []
type: summary

-->
## OpenAI ChatGPT Plugin

The OpenAI ChatGPT plugin integrates Large Language Model (LLM) capabilities, specifically OpenAI's chat completions, directly into your workflows. This enables you to define tasks that interact with the ChatGPT API, sending prompts and receiving responses as part of your automated processes.

### Defining a ChatGPT Task

A ChatGPT task is defined using the `ChatGPTTask` class. This class allows you to specify the OpenAI model to use and configure the chat completion parameters.

To define a task, instantiate `ChatGPTTask` with the following parameters:

*   `name`: A unique name for the task within your project.
*   `chatgpt_config`: A dictionary containing the configuration for the ChatGPT API call. This dictionary directly maps to the parameters expected by OpenAI's `chat.completions.create` API. The `model` key is a mandatory field within this configuration.
*   `openai_organization`: (Optional) Your OpenAI organization ID. This is useful if you belong to multiple organizations.

The `ChatGPTTask` is designed to accept a single string input named `message` and produces a single string output named `o0`, which contains the content of the LLM's response.

```python
from flytekitplugins.openai.chatgpt.task import ChatGPTTask
from flytekit import workflow

# Define a ChatGPT task
chat_task = ChatGPTTask(
    name="my_chat_completion_task",
    chatgpt_config={
        "model": "gpt-3.5-turbo",
        "temperature": 0.7,
        "max_tokens": 150,
    },
    # openai_organization="your-openai-organization-id", # Uncomment and set if needed
)

@workflow
def chat_workflow(prompt: str) -> str:
    # Execute the ChatGPT task with the provided prompt
    response = chat_task(message=prompt)
    return response

# Example usage:
# if __name__ == "__main__":
#     print(chat_workflow(prompt="Explain quantum entanglement in simple terms."))
```

### Configuration Details

The `chatgpt_config` parameter is crucial for controlling the behavior of the ChatGPT model. It directly corresponds to the request body for the `chat.completions.create` endpoint in the OpenAI API.

Key configuration options within `chatgpt_config` include:

*   `model`: (Required) Specifies the ID of the model to use (e.g., `"gpt-3.5-turbo"`, `"gpt-4"`). A `ValueError` is raised if this is not provided.
*   `temperature`: Controls the randomness of the output. Higher values (e.g., 0.8) make the output more random, while lower values (e.g., 0.2) make it more focused and deterministic.
*   `max_tokens`: The maximum number of tokens to generate in the chat completion.
*   `top_p`: An alternative to sampling with temperature, where the model considers the tokens with `top_p` probability mass.
*   `n`: How many chat completion choices to generate for each input message.

For a comprehensive list of available parameters and their descriptions, refer to the official OpenAI API documentation for chat completions.

### Task Execution

When a `ChatGPTTask` is executed within a workflow, the underlying `ChatGPTConnector` handles the interaction with the OpenAI API.

1.  **Input Processing**: The `message` input provided to the `ChatGPTTask` is extracted.
2.  **Configuration Merging**: The `chatgpt_config` and `openai_organization` specified during task definition are retrieved. The input `message` is then dynamically inserted into the `messages` array within `chatgpt_config` as a user role message: `{"role": "user", "content": message}`.
3.  **API Client Initialization**: An asynchronous OpenAI client (`openai.AsyncOpenAI`) is initialized using the provided `openai_organization` and an API key. The API key is securely retrieved using `get_connector_secret(secret_key=OPENAI_API_KEY)`. Ensure the `OPENAI_API_KEY` secret is configured in your environment.
4.  **API Call**: The `client.chat.completions.create` method is invoked with the prepared `chatgpt_config`.
5.  **Timeout**: The API call is wrapped with an `asyncio.wait_for` call, enforcing a default timeout of `TIMEOUT_SECONDS` (typically 60 seconds) to prevent indefinite waits.
6.  **Response Extraction**: Upon successful completion, the content of the first message choice (`completion.choices[0].message.content`) is extracted.
7.  **Output**: The extracted content is returned as the `o0` output of the task.

During execution, the `httpx` logger's level is set to `WARNING` to reduce verbose output from the underlying HTTP client.

### Best Practices and Considerations

*   **API Key Management**: Always manage your OpenAI API key securely. The plugin expects the API key to be available via a secret named `OPENAI_API_KEY`. Refer to your platform's documentation on how to configure secrets.
*   **OpenAI Organization**: If your OpenAI account is part of multiple organizations, explicitly setting `openai_organization` ensures that API calls are attributed correctly and use the appropriate billing context.
*   **`chatgpt_config` Validation**: While the plugin enforces the presence of the `model` key, it does not validate the entire structure or values within `chatgpt_config`. Ensure your configuration adheres to OpenAI's API specifications to avoid runtime errors.
*   **Timeouts**: Be aware of the default timeout for API calls. For very long or complex prompts, or if OpenAI's service is experiencing high load, the task might time out. Adjusting the `TIMEOUT_SECONDS` (if exposed or configurable) or handling retries in your workflow logic might be necessary for robust applications.
*   **Cost Management**: Be mindful of the `model` selected and `max_tokens` configured, as these directly impact the cost of your OpenAI API usage.
*   **Error Handling**: Implement robust error handling in your workflows to gracefully manage potential API errors (e.g., rate limits, invalid API keys, model errors) that can occur during the `ChatGPTTask` execution.
<!--
key: summary_openai_chatgpt_plugin_6f0a9bc0-a7be-4267-ac64-8c594b019b0b
type: summary_end

-->
<!--
code_unit: flytekitplugins.openai.chatgpt.task
code_unit_type: class
help_text: ''
key: example_653da582-3898-45be-85b7-6a76ca30c6c5
type: example

-->