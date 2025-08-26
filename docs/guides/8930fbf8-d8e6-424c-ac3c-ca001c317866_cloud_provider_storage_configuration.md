
<!--
help_text: ''
key: summary_cloud_provider_storage_configuration_4d1c8b35-07e3-4c8b-a312-3a1881d9f11b
modules:
- flytekit.configuration.internal.AWS
- flytekit.configuration.internal.AZURE
- flytekit.configuration.internal.GCP
questions_to_answer: []
type: summary

-->
Cloud Provider Storage Configuration

This section details the configuration options for integrating with various cloud provider storage services. These configurations enable secure and efficient data operations with Amazon S3, Azure Blob Storage, and Google Cloud Storage.

### AWS Storage Configuration

The AWS configuration section defines parameters for connecting to Amazon S3. These settings are crucial for authentication, endpoint customization, and operational resilience.

**Key Configuration Parameters:**

*   **`S3_ENDPOINT`**: Specifies a custom S3 endpoint. This is useful for connecting to S3-compatible storage solutions or specific AWS regions that require an explicit endpoint.
    *   **Type**: String
    *   **Example**: `storage.connection.endpoint: https://s3.us-west-1.amazonaws.com`
*   **`S3_ACCESS_KEY_ID`**: Your AWS access key ID. This credential is used for authenticating requests to S3.
    *   **Type**: String
    *   **Example**: `storage.connection.access-key: AKIAIOSFODNN7EXAMPLE`
*   **`S3_SECRET_ACCESS_KEY`**: Your AWS secret access key. This credential, used in conjunction with the access key ID, signs your S3 requests.
    *   **Type**: String
    *   **Example**: `storage.connection.secret-key: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY`
*   **`ENABLE_DEBUG`**: A boolean flag to enable detailed debug logging for S3 operations. Enabling this can provide verbose output useful for troubleshooting connection or operation issues.
    *   **Type**: Boolean
    *   **Example**: `aws.enable_debug: true`
*   **`RETRIES`**: The maximum number of times to retry a failed S3 operation. Configuring retries enhances the robustness of operations against transient network issues or service unavailability.
    *   **Type**: Integer
    *   **Example**: `aws.retries: 5`
*   **`BACKOFF_SECONDS`**: The delay in seconds between retry attempts for S3 operations. This value is transformed into a `datetime.timedelta` object internally, allowing for exponential or fixed backoff strategies.
    *   **Type**: Integer (seconds)
    *   **Example**: `aws.backoff_seconds: 10`

**Implementation Notes:**

These parameters are typically configured via a YAML configuration file under the `aws` section or `storage.connection` for S3-specific connection details. For example:

```yaml
aws:
  enable_debug: false
  retries: 3
  backoff_seconds: 5
storage:
  connection:
    endpoint: https://s3.us-east-1.amazonaws.com
    access-key: YOUR_ACCESS_KEY_ID
    secret-key: YOUR_SECRET_ACCESS_KEY
```

**Best Practices:**

*   Avoid hardcoding AWS credentials directly in your application code. Utilize environment variables or a secure configuration management system.
*   For production environments, prefer using AWS Identity and Access Management (IAM) roles with temporary credentials over long-lived access keys, especially when running on EC2 instances or within AWS Lambda.
*   Tune `RETRIES` and `BACKOFF_SECONDS` based on the expected network reliability and the nature of your S3 operations. Aggressive retries with short backoffs can exacerbate issues during widespread outages.

### Azure Storage Configuration

The Azure configuration section provides parameters for authenticating and interacting with Azure Blob Storage. It supports both traditional storage account key authentication and more secure Azure Active Directory (AAD) based authentication using service principals.

**Key Configuration Parameters:**

*   **`STORAGE_ACCOUNT_NAME`**: The name of your Azure Storage Account. This is a mandatory parameter for all Azure storage operations.
    *   **Type**: String
    *   **Example**: `azure.storage_account_name: mystorageaccount`
*   **`STORAGE_ACCOUNT_KEY`**: The access key for your Azure Storage Account. This provides full access to the storage account and its contents.
    *   **Type**: String
    *   **Example**: `azure.storage_account_key: YOUR_STORAGE_ACCOUNT_KEY`
*   **`TENANT_ID`**: The Azure Active Directory (AAD) tenant ID (directory ID) for service principal authentication.
    *   **Type**: String
    *   **Example**: `azure.tenant_id: 12345678-abcd-1234-efgh-1234567890ab`
*   **`CLIENT_ID`**: The application (client) ID of your service principal for AAD authentication.
    *   **Type**: String
    *   **Example**: `azure.client_id: abcdef12-3456-7890-abcd-ef1234567890`
*   **`CLIENT_SECRET`**: The client secret (application password) for your service principal.
    *   **Type**: String
    *   **Example**: `azure.client_secret: YOUR_CLIENT_SECRET`

**Authentication Methods:**

The Azure configuration supports two primary authentication methods:

1.  **Storage Account Key**: By providing `STORAGE_ACCOUNT_NAME` and `STORAGE_ACCOUNT_KEY`. This method offers direct access but is less secure for long-term credentials.
2.  **Azure Active Directory (AAD) Service Principal**: By providing `STORAGE_ACCOUNT_NAME`, `TENANT_ID`, `CLIENT_ID`, and `CLIENT_SECRET`. This is the recommended approach for production environments as it allows for fine-grained access control via Azure RBAC (Role-Based Access Control) and better credential management.

**Implementation Notes:**

Configure these parameters in your YAML file under the `azure` section:

```yaml
azure:
  storage_account_name: myprodstorage
  # Option 1: Storage Account Key (less secure for production)
  # storage_account_key: YOUR_STORAGE_ACCOUNT_KEY

  # Option 2: Azure Active Directory Service Principal (recommended)
  tenant_id: YOUR_TENANT_ID
  client_id: YOUR_CLIENT_ID
  client_secret: YOUR_CLIENT_SECRET
```

**Best Practices:**

*   Prioritize AAD service principal authentication over storage account keys for production deployments. Service principals offer enhanced security, auditability, and integration with Azure's identity management.
*   Ensure that the service principal has only the necessary permissions (least privilege) to the Azure Blob Storage containers it needs to access.
*   Regularly rotate storage account keys and client secrets to minimize the risk of compromise.

### GCP Storage Configuration

The GCP configuration section provides specific settings for interacting with Google Cloud Storage (GCS), particularly concerning performance optimizations for data transfer operations.

**Key Configuration Parameters:**

*   **`GSUTIL_PARALLELISM`**: A boolean flag that controls whether `gsutil` operations (e.g., uploads, downloads) should utilize parallel processing. Enabling parallelism can significantly improve performance for large files or numerous small files by leveraging multiple concurrent connections.
    *   **Type**: Boolean
    *   **Example**: `gcp.gsutil_parallelism: true`

**Implementation Notes:**

This parameter is configured in your YAML file under the `gcp` section:

```yaml
gcp:
  gsutil_parallelism: true
```

**Performance Considerations:**

Enabling `GSUTIL_PARALLELISM` is generally recommended for optimal performance when transferring data to or from Google Cloud Storage. However, in environments with strict network bandwidth limitations or when dealing with a very high number of extremely small files, disabling parallelism might sometimes reduce overhead, though this is rare. Test with and without parallelism to determine the best setting for your specific use case and network conditions.

**Authentication for GCP:**

Authentication for GCP storage operations typically relies on the standard Google Cloud authentication mechanisms, such as `gcloud` CLI configuration, environment variables (`GOOGLE_APPLICATION_CREDENTIALS`), or service account JSON files. These are handled externally and are not directly configured through the provided parameters. Ensure your environment is correctly authenticated with GCP before performing storage operations.
<!--
key: summary_cloud_provider_storage_configuration_4d1c8b35-07e3-4c8b-a312-3a1881d9f11b
type: summary_end

-->
<!--
code_unit: flytekit.configuration.internal.AWS
code_unit_type: class
help_text: ''
key: example_be2d750f-11fa-4886-8cdc-2466d7951bd4
type: example

-->
<!--
code_unit: flytekit.configuration.internal.AZURE
code_unit_type: class
help_text: ''
key: example_36f65d66-0691-470b-a402-dbd834ea3b49
type: example

-->
<!--
code_unit: flytekit.configuration.internal.GCP
code_unit_type: class
help_text: ''
key: example_e1d22c1e-6d2a-45a0-a71c-5aeea0cf86df
type: example

-->