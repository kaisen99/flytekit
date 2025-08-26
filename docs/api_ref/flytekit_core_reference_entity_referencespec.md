# ReferenceSpec

This class encapsulates a reference specification, associating it with a reference template. It provides access to the underlying template through a property. The class is designed to manage and represent reference-related data.

## Attributes

- **template**: [ReferenceTemplate](flytekit_core_reference_entity_referencetemplate)
  - The template for the reference.

## Constructors
def ReferenceSpec(template: [ReferenceTemplate](flytekit_core_reference_entity_referencetemplate)) - > None
-  Initializes a new instance of the ReferenceSpec class.
- **Parameters**

  - **template**: [ReferenceTemplate](flytekit_core_reference_entity_referencetemplate)
    - The reference template to associate with this instance.

- **Return Value**:
**None**
  - None
def ReferenceSpec(template: [ReferenceTemplate](flytekit_core_reference_entity_referencetemplate))
-  Initializes the ReferenceSpec with a ReferenceTemplate.
- **Parameters**

  - **template**: [ReferenceTemplate](flytekit_core_reference_entity_referencetemplate)
    - The ReferenceTemplate to associate with this spec.



## Methods
```@classmethod
def template()
```
-  Gets the ReferenceTemplate associated with this spec.

- **Return Value**:
**[ReferenceTemplate](flytekit_core_reference_entity_referencetemplate)**
  - The associated ReferenceTemplate.
