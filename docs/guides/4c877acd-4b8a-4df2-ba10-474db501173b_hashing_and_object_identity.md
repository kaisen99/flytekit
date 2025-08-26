
<!--
help_text: ''
key: summary_hashing_and_object_identity_2902e29b-d0e0-409d-a2cf-2514a47412f3
modules:
- flytekit.core.hash
questions_to_answer: []
type: summary

-->
Hashing and Object Identity

Hashing plays a crucial role in identifying and comparing objects, particularly when managing uniqueness, optimizing data structures like dictionaries and sets, or ensuring consistency across distributed systems. This section details the mechanisms for defining object identity through hashing.

### Identity-Based Hashing

For objects whose identity is inherently tied to their unique instance in memory, the `HashOnReferenceMixin` provides a straightforward way to implement hashing. This mixin defines an object's hash based on its memory address, as returned by Python's built-in `id()` function.

**Usage:**
To make an object hashable based on its reference, inherit from `HashOnReferenceMixin`.

```python
from flytekit.core.hash import HashOnReferenceMixin

class MyReferenceObject(HashOnReferenceMixin):
    def __init__(self, value):
        self.value = value

# Instances are unique based on their memory address
obj1 = MyReferenceObject(10)
obj2 = MyReferenceObject(10)
obj3 = obj1

print(f"Hash of obj1: {hash(obj1)}")
print(f"Hash of obj2: {hash(obj2)}")
print(f"Hash of obj3: {hash(obj3)}")

# obj1 and obj3 have the same hash because they are the same object instance
assert hash(obj1) == hash(obj3)
# obj1 and obj2 have different hashes because they are different object instances
assert hash(obj1) != hash(obj2)
```

**Considerations:**
Using `HashOnReferenceMixin` is suitable when:
*   An object's identity is its memory location, not its content.
*   The object is mutable, and content-based hashing would be complex or lead to inconsistent hashes if the object changes while used as a dictionary key or set member.
*   You need a quick, default hash implementation for objects that will not be compared by content.

Be aware that the hash of an object inheriting from `HashOnReferenceMixin` changes if the object is recreated, even if its internal state is identical. This approach is not suitable for scenarios requiring content-based equality or hashing.

### Custom Hashing Strategies

When object identity needs to be determined by its content, specific attributes, or a custom algorithm, the `HashMethod` class provides a flexible and type-safe way to encapsulate and apply custom hashing logic. This approach decouples the hashing algorithm from the object definition, allowing for external, configurable hashing strategies.

A `HashMethod` wraps a callable that takes an object of a specific type `T` and returns a string representation of its hash.

**Usage:**
Define a function that calculates the hash for a given object, then wrap it with `HashMethod`. The `calculate` method then applies this wrapped function.

```python
from typing import NamedTuple
from flytekit.core.hash import HashMethod

# Define a simple data structure
class MyData(NamedTuple):
    name: str
    version: str
    checksum: str

# Define a custom hashing function for MyData
def my_data_hasher(data: MyData) -> str:
    # Example: Hash based on name and version, ignoring checksum
    return f"{data.name}-{data.version}"

# Create a HashMethod instance
data_hash_method = HashMethod[MyData](my_data_hasher)

# Use the HashMethod to calculate hashes
data1 = MyData(name="model_a", version="v1.0", checksum="abc")
data2 = MyData(name="model_a", version="v1.0", checksum="xyz")
data3 = MyData(name="model_b", version="v1.0", checksum="abc")

hash1 = data_hash_method.calculate(data1)
hash2 = data_hash_method.calculate(data2)
hash3 = data_hash_method.calculate(data3)

print(f"Hash of data1: {hash1}")
print(f"Hash of data2: {hash2}")
print(f"Hash of data3: {hash3}")

# data1 and data2 have the same hash because their name and version are identical
assert hash1 == hash2
assert hash1 != hash3
```

**Benefits:**
*   **Decoupling**: Separates hashing logic from the object definition, promoting cleaner code and easier maintenance.
*   **Reusability**: A `HashMethod` instance can be reused across different parts of the codebase where the same hashing strategy is needed for a specific type.
*   **Flexibility**: Allows for complex, context-specific hashing algorithms that might involve external data or specific business rules.
*   **Type Safety**: The generic `HashMethod[T]` ensures that the wrapped function is applied to objects of the expected type.

### Best Practices and Integration

When working with hashing and object identity, consider the following best practices:

*   **Consistency with Equality (`__eq__`)**: If you override `__hash__` for a class, you must also override `__eq__`. Python requires that if two objects are equal (`a == b` is true), then their hashes must also be equal (`hash(a) == hash(b)`). The `HashOnReferenceMixin` inherently satisfies this for identity-based equality. For custom hashing, ensure your `__eq__` implementation aligns with the attributes used in your `HashMethod`'s underlying function.
*   **Immutability for Content-Based Hashing**: For objects that use content-based hashing (e.g., via `HashMethod` or a custom `__hash__` implementation), it is best practice for the object to be immutable. If a mutable object's content changes after it has been hashed and used as a key in a dictionary or a member of a set, its hash will no longer match its current state, leading to incorrect behavior.
*   **Performance**: Hashing operations should be efficient. `id()` is very fast. Custom hashing functions wrapped by `HashMethod` should avoid computationally expensive operations if they are frequently called, especially in performance-critical paths like caching or large data processing.
*   **Use Cases**:
    *   **Caching**: Hashing is fundamental for cache keys, allowing quick lookups of previously computed results based on input object identity or content.
    *   **Deduplication**: Identifying and removing duplicate objects in collections.
    *   **Workflow Uniqueness**: In workflow systems, hashing can ensure that identical tasks or data artifacts are not re-executed or re-stored unnecessarily.
    *   **Configuration Management**: Hashing configuration objects to detect changes or ensure consistent application of settings.

By leveraging these hashing utilities, developers can precisely control how objects are identified and compared, leading to more robust, efficient, and predictable applications.
<!--
key: summary_hashing_and_object_identity_2902e29b-d0e0-409d-a2cf-2514a47412f3
type: summary_end

-->
<!--
code_unit: flytekit.core.hash.HashMethod
code_unit_type: class
help_text: ''
key: example_31e045eb-b87e-44b0-842e-2d2f56b98c11
type: example

-->
<!--
code_unit: flytekit.core.hash.HashOnReferenceMixin
code_unit_type: class
help_text: ''
key: example_968c0e68-ad73-4f17-972d-6e71122516b9
type: example

-->