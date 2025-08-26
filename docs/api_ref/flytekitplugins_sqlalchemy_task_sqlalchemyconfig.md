# SQLAlchemyConfig

This class configures database connections for tasks using SQLAlchemy. It allows specifying the database URI and connection arguments.  It supports loading secrets for secure database access.

## Attributes

- **uri**: str
  - default sqlalchemy connector

- **connect_args**: typing.Optional[typing.Dict[str, typing.Any]] = None
  - sqlalchemy kwarg overrides -- ex: host

- **secret_connect_args**: typing.Optional[typing.Dict[str, [Secret](flytekit_models_security_secret)]] = None
  - flyte secrets loaded into sqlalchemy connect args -- ex: {&quot;password&quot;: flytekit.models.security.Secret(name=SECRET_NAME, group=SECRET_GROUP)}

## Constructors
def SQLAlchemyConfig(uri: str, connect_args: typing.Optional[typing.Dict[str, typing.Any]] = None, secret_connect_args: typing.Optional[typing.Dict[str, [Secret](flytekit_models_security_secret)]] = None)
-  Use this configuration to configure task. String should be standard
sqlalchemy connector format
(https://docs.sqlalchemy.org/en/14/core/engines.html#database-urls).
Database can be found:
- within the container
- or from a publicly accessible source
- **Parameters**

  - **uri**: str
    - default sqlalchemy connector
  - **connect_args**: typing.Optional[typing.Dict[str, typing.Any]]
    - sqlalchemy kwarg overrides -- ex: host
  - **secret_connect_args**: typing.Optional[typing.Dict[str, [Secret](flytekit_models_security_secret)]]
    - flyte secrets loaded into sqlalchemy connect args
-- ex: {&quot;password&quot;: flytekit.models.security.Secret(name=SECRET_NAME, group=SECRET_GROUP)}



## Methods
```@classmethod
def secret_connect_args_to_dicts()
```
-  Converts the secret_connect_args dictionary to a dictionary of dictionaries, where each inner dictionary represents a Secret object.

- **Return Value**:
**Optional[Dict[str, Dict[str, Optional[str]]]]**
  - A dictionary of dictionaries representing the secret_connect_args, or None if secret_connect_args is not set.
