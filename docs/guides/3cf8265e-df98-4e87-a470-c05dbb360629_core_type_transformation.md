
<!--
help_text: ''
key: summary_core_type_transformation_11a8b2cb-6cc2-4e81-bfa3-36989c66631d
modules:
- flytekit.core.type_engine.TypeEngine
- flytekit.core.type_engine.TypeTransformer
- flytekit.core.type_engine.AsyncTypeTransformer
- flytekit.core.type_engine.RestrictedTypeTransformer
- flytekit.core.type_engine.SimpleTransformer
- flytekit.core.type_engine.ProtobufTransformer
- flytekit.core.type_engine.EnumTransformer
- flytekit.core.type_engine.BinaryIOTransformer
- flytekit.core.type_engine.TextIOTransformer
questions_to_answer: []
type: summary

-->
Core Type Transformation provides a robust and flexible mechanism for converting data between disparate types and structures within an application. This capability is essential for integrating with external systems, persisting data, or adapting internal data models to different contexts. It focuses on declarative and programmatic approaches to define how source data maps to a target structure, ensuring data integrity and consistency across various representations.

## Core Concepts

The transformation system relies on several key components to define and execute type conversions:

*   **Transformation Engine:** The central orchestrator that processes transformation requests. It resolves appropriate conversion rules and applies them to source data.
*   **Type Mappers:** Define the explicit mapping rules between a source type and a target type. A type mapper specifies how individual fields or nested structures are converted, including renaming, reformatting, or applying custom logic.
*   **Conversion Rules:** Granular definitions within a type mapper that dictate how a specific source field or value transforms into a target field or value. These can range from simple direct assignments to complex custom functions.
*   **Type Adapters:** Provide a way to handle common conversions for specific data types (e.g., converting a string to a `datetime` object, or an integer to a boolean). These are often pre-registered with the transformation engine.

## Basic Usage

To perform a core type transformation, define a `TypeMapper` and then use the `TransformationEngine` to apply it.

Consider a scenario where an external API returns user data in a flat dictionary format, and the application requires a structured `User` object.

```python
from typing import Dict, Any, Optional
from datetime import datetime

# Assume these are your application's internal models
class UserProfile:
    email: str
    full_name: str
    last_login: Optional[datetime]

class User:
    user_id: str
    username: str
    profile: UserProfile

# The external API data might look like this:
api_data = {
    "id": "usr_123",
    "user_name": "johndoe",
    "email_address": "john.doe@example.com",
    "first_name": "John",
    "last_name": "Doe",
    "last_access_timestamp": 1678886400 # Unix timestamp
}

# Define a TypeMapper
class ApiUserToUserMapper:
    def map(self, source: Dict[str, Any]) -> User:
        profile_data = {
            "email": source["email_address"],
            "full_name": f"{source['first_name']} {source['last_name']}",
            "last_login": datetime.fromtimestamp(source["last_access_timestamp"]) if "last_access_timestamp" in source else None
        }
        profile = self.transform_profile(profile_data) # Delegate nested transformation

        return User(
            user_id=source["id"],
            username=source["user_name"],
            profile=profile
        )

    def transform_profile(self, source: Dict[str, Any]) -> UserProfile:
        # This could also be a separate TypeMapper registered with the engine
        return UserProfile(
            email=source["email"],
            full_name=source["full_name"],
            last_login=source["last_login"]
        )

# Instantiate and use the Transformation Engine
# In a real system, the engine would be configured with registered mappers and adapters
class TransformationEngine:
    def transform(self, source_data: Any, mapper_instance: Any) -> Any:
        # Simplified for example: In reality, it would look up the mapper based on types
        return mapper_instance.map(source_data)

engine = TransformationEngine()
mapper = ApiUserToUserMapper()
transformed_user = engine.transform(api_data, mapper)

print(f"Transformed User ID: {transformed_user.user_id}")
print(f"Transformed User Profile Email: {transformed_user.profile.email}")
print(f"Transformed User Profile Full Name: {transformed_user.profile.full_name}")
print(f"Transformed User Profile Last Login: {transformed_user.profile.last_login}")
```

This example demonstrates a manual `TypeMapper`. In a more sophisticated implementation, the `TransformationEngine` automatically discovers and applies mappers based on the source and target types, often leveraging decorators or a registration API.

## Advanced Mappings and Customization

The transformation system supports complex mapping scenarios beyond direct field-to-field assignments.

### Custom Conversion Rules

Define custom logic for specific field transformations using functions or methods. This is particularly useful for data enrichment, formatting, or conditional logic during conversion.

```python
# Extending the previous example
class EnhancedApiUserToUserMapper(ApiUserToUserMapper):
    def map(self, source: Dict[str, Any]) -> User:
        # ... (previous mapping logic) ...
        profile_data = {
            "email": self._normalize_email(source["email_address"]), # Apply custom rule
            "full_name": f"{source['first_name']} {source['last_name']}",
            "last_login": datetime.fromtimestamp(source["last_access_timestamp"]) if "last_access_timestamp" in source else None
        }
        profile = self.transform_profile(profile_data)

        return User(
            user_id=source["id"],
            username=source["user_name"],
            profile=profile
        )

    def _normalize_email(self, email: str) -> str:
        """Converts email to lowercase and strips whitespace."""
        return email.strip().lower()

# Usage remains similar
enhanced_mapper = EnhancedApiUserToUserMapper()
transformed_user_enhanced = engine.transform(api_data, enhanced_mapper)
print(f"Normalized Email: {transformed_user_enhanced.profile.email}")
```

### Nested Object Transformation

Handle complex object graphs by chaining or nesting `TypeMapper` instances. The `TransformationEngine` can automatically resolve and apply mappers for nested objects if they are registered.

For instance, if `UserProfile` was a separate, complex object, a dedicated `UserProfileMapper` could be created and registered, allowing the `ApiUserToUserMapper` to simply delegate the `profile` field transformation to the engine.

```python
# Assuming a dedicated UserProfileMapper exists and is registered with the engine
class UserProfileMapper:
    def map(self, source: Dict[str, Any]) -> UserProfile:
        return UserProfile(
            email=source["email"],
            full_name=source["full_name"],
            last_login=source["last_login"]
        )

# The TransformationEngine would have a registration mechanism
class AdvancedTransformationEngine:
    def __init__(self):
        self._mappers = {}

    def register_mapper(self, source_type: type, target_type: type, mapper_instance: Any):
        self._mappers[(source_type, target_type)] = mapper_instance

    def transform(self, source_data: Any, target_type: type) -> Any:
        # In a real system, source_type would be inferred or passed
        source_type = type(source_data) if isinstance(source_data, dict) else source_data.__class__
        mapper = self._mappers.get((source_type, target_type))
        if not mapper:
            raise ValueError(f"No mapper registered for {source_type} to {target_type}")
        return mapper.map(source_data)

# Registering mappers
adv_engine = AdvancedTransformationEngine()
adv_engine.register_mapper(Dict, UserProfile, UserProfileMapper())
adv_engine.register_mapper(Dict, User, ApiUserToUserMapper()) # This mapper would now call adv_engine.transform for profile

# The ApiUserToUserMapper would be updated to use the engine for nested objects:
class ApiUserToUserMapperWithEngine:
    def __init__(self, engine: AdvancedTransformationEngine):
        self.engine = engine

    def map(self, source: Dict[str, Any]) -> User:
        profile_data = {
            "email": source["email_address"],
            "full_name": f"{source['first_name']} {source['last_name']}",
            "last_login": datetime.fromtimestamp(source["last_access_timestamp"]) if "last_access_timestamp" in source else None
        }
        # Delegate profile transformation to the engine
        profile = self.engine.transform(profile_data, UserProfile)

        return User(
            user_id=source["id"],
            username=source["user_name"],
            profile=profile
        )

adv_engine.register_mapper(Dict, User, ApiUserToUserMapperWithEngine(adv_engine))
transformed_user_nested = adv_engine.transform(api_data, User)
print(f"Transformed User (nested): {transformed_user_nested.profile.full_name}")
```

## Integration Patterns

Core Type Transformation integrates seamlessly into various application architectures:

*   **Data Layer Mapping:** Convert database records (e.g., SQLAlchemy ORM objects, raw dictionaries from a NoSQL store) into domain-specific business objects. This decouples the application logic from the persistence layer's data representation.
*   **API Gateway/Serialization:** Transform incoming request payloads into internal data models and outgoing response models back into API-specific formats (e.g., JSON, XML). This ensures consistent data contracts with external consumers.
*   **Event-Driven Architectures:** Normalize event payloads from different sources into a canonical internal event format. This is crucial for microservices communicating via message queues.
*   **Configuration Management:** Convert raw configuration data (e.g., from YAML, TOML files) into strongly typed configuration objects, providing validation and easier access.

## Performance Considerations

While providing flexibility, complex transformations can impact performance.

*   **Pre-compilation/Caching:** For frequently used mappers, consider pre-compiling or caching the transformation logic to avoid repeated setup overhead.
*   **Lazy Transformation:** For very large or deeply nested structures, transform only the necessary parts of the data on demand, rather than the entire object graph upfront.
*   **Batch Processing:** When transforming collections of objects, process them in batches to leverage potential optimizations in the underlying transformation engine.
*   **Avoid Redundant Operations:** Design mappers to perform each transformation step only once. For example, if a value is needed in multiple places, transform it once and reuse the result.

## Limitations and Best Practices

*   **Circular References:** Be mindful of circular references in object graphs when defining mappers, as this can lead to infinite recursion during transformation. Design mappers to break such cycles or handle them explicitly.
*   **Error Handling:** Implement robust error handling within custom conversion rules. If a source value cannot be transformed into the target type, decide whether to raise an error, return a default value, or log the issue.
*   **Schema Evolution:** When source or target schemas change, update the corresponding `TypeMapper` definitions. Versioning mappers can help manage schema evolution in large systems.
*   **Test Thoroughly:** Write comprehensive unit and integration tests for all `TypeMapper` implementations to ensure correct data transformation under various conditions, including edge cases and invalid inputs.
*   **Keep Mappers Focused:** Each `TypeMapper` should ideally be responsible for transforming one specific source type to one specific target type. Avoid creating monolithic mappers that handle too many different transformations.
*   **Declarative Over Imperative:** Where possible, prefer declarative mapping configurations (e.g., using field annotations or configuration objects) over purely imperative code within `map` methods. This improves readability and maintainability.
<!--
key: summary_core_type_transformation_11a8b2cb-6cc2-4e81-bfa3-36989c66631d
type: summary_end

-->
<!--
code_unit: flytekit.core.type_engine.SimpleTransformer
code_unit_type: class
help_text: ''
key: example_e551ec7a-48ee-4c4c-b7d3-0ec29294c982
type: example

-->
<!--
code_unit: flytekit.core.type_engine.ProtobufTransformer
code_unit_type: class
help_text: ''
key: example_830c26d6-3331-4743-802d-41cbc42a0c5d
type: example

-->
<!--
code_unit: flytekit.core.type_engine.EnumTransformer
code_unit_type: class
help_text: ''
key: example_811800d0-50cb-4aa5-8a49-1ac512b93295
type: example

-->