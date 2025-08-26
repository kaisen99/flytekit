# WorkflowSpec

This class represents the complete specification of a workflow, encapsulating its template and any associated sub-workflows. It provides methods for converting the workflow specification to and from a Flyte IDL representation. The class includes properties to access the workflow template, sub-workflows, and documentation.

## Attributes

- **template**: flytekit.models.core.workflow.WorkflowTemplate
  - WorkflowTemplate

- **sub_workflows**: list[flytekit.models.core.workflow.WorkflowTemplate]
  - list[flytekit.models.core.workflow.WorkflowTemplate]

- **docs**: typing.Optional[[Documentation](flytekit_models_documentation_documentation)]
  - Documentation entity for the workflow

## Constructors
def WorkflowSpec(template: flytekit.models.core.workflow.WorkflowTemplate, sub_workflows: list[flytekit.models.core.workflow.WorkflowTemplate], docs: typing.Optional[[Documentation](flytekit_models_documentation_documentation)] = None)
-  This object fully encapsulates the specification of a workflow
- **Parameters**

  - **template**: flytekit.models.core.workflow.WorkflowTemplate
    - flytekit.models.core.workflow.WorkflowTemplate template
  - **sub_workflows**: list[flytekit.models.core.workflow.WorkflowTemplate]
    - list[flytekit.models.core.workflow.WorkflowTemplate] sub_workflows
  - **docs**: typing.Optional[[Documentation](flytekit_models_documentation_documentation)]
    - Documentation entity for the workflow

def WorkflowSpec(template: flytekit.models.core.workflow.WorkflowTemplate, sub_workflows: list[flytekit.models.core.workflow.WorkflowTemplate], docs: typing.Optional[[Documentation](flytekit_models_documentation_documentation)] = None)
-  This object fully encapsulates the specification of a workflow
- **Parameters**

  - **template**: flytekit.models.core.workflow.WorkflowTemplate
    - flytekit.models.core.workflow.WorkflowTemplate template
  - **sub_workflows**: list[flytekit.models.core.workflow.WorkflowTemplate]
    - sub_workflows
  - **docs**: typing.Optional[[Documentation](flytekit_models_documentation_documentation)]
    - docs



## Methods
```@classmethod
def template()
```
-  Returns the template of the workflow.

- **Return Value**:
**flytekit.models.core.workflow.WorkflowTemplate**
  - The template of the workflow.
```@classmethod
def sub_workflows()
```
-  Returns the sub-workflows of the workflow.

- **Return Value**:
**list[flytekit.models.core.workflow.WorkflowTemplate]**
  - The sub-workflows of the workflow.
```@classmethod
def docs()
```
-  Returns the documentation of the workflow.

- **Return Value**:
**Description entity for the workflow**
  - The documentation of the workflow.
```@classmethod
def to_flyte_idl()
```
-  Converts the WorkflowSpec to its flyte IDL representation.

- **Return Value**:
**flyteidl.admin.workflow_pb2.WorkflowSpec**
  - The flyte IDL representation of the WorkflowSpec.
@classmethod
def from_flyte_idl(pb2_object: flyteidl.admin.workflow_pb2.WorkflowSpec) - > [WorkflowSpec](flytekit_models_admin_workflow_workflowspec)
-  Creates a WorkflowSpec from its flyte IDL representation.
- **Parameters**

  - **pb2_object**: flyteidl.admin.workflow_pb2.WorkflowSpec
    - The flyte IDL WorkflowSpec object.

- **Return Value**:
**[WorkflowSpec](flytekit_models_admin_workflow_workflowspec)**
  - A WorkflowSpec object.
