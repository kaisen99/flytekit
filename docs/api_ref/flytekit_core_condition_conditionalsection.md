# ConditionalSection

This class represents a conditional section within a workflow, primarily for use in compilation mode. It enables the definition of conditional logic with multiple cases. Key features include methods to define conditions (`if_`), start and end branches (`start_branch`, `end_branch`), and compute outputs. It relies on the FlyteContextManager and interacts with Case, Condition, and Node objects.

## Attributes

- **name**: str
  - The name of the conditional section.

- **cases**: typing.List[[Case](flytekit_core_condition_case)]
  - A list of cases within the conditional section.

## Constructors
def ConditionalSection(name: str)
-  Initializes a new instance of the ConditionalSection class.
- **Parameters**

  - **name**: str
    - The name of the conditional section.



## Methods
```@classmethod
def name()
```
-  Returns the name of the conditional section.

- **Return Value**:
**string**
  - The name of the conditional section.
```@classmethod
def cases()
```
-  Returns the list of cases within the conditional section.

- **Return Value**:
**typing.List[[Case](flytekit_core_condition_case)]**
  - A list of Case objects.
@classmethod
def start_branch(c: [Case](flytekit_core_condition_case), last_case: bool = False) - > [Case](flytekit_core_condition_case)
-  Registers a new case (branch) to the conditional section. This method should be called at the beginning of each branch&#x27;s execution.
- **Parameters**

  - **c**: [Case](flytekit_core_condition_case)
    - The Case object representing this branch.
  - **last_case**: bool
    - A boolean indicating if this is the last branch in the conditional block.

- **Return Value**:
**[Case](flytekit_core_condition_case)**
  - The Case object representing the current branch.
```@classmethod
def end_branch()
```
-  Should be invoked after every branch has been visited. It handles the finalization of the conditional section, returning either a promise or a condition based on whether it&#x27;s the last branch.

- **Return Value**:
**Optional[[Union](flytekit_models_literals_union)[[Condition](flytekit_core_condition_condition), [Promise](flytekit_core_promise_promise), Tuple[[Promise](flytekit_core_promise_promise)], [VoidPromise](flytekit_core_promise_voidpromise)]]**
  - Returns the final promise or condition, or None if it&#x27;s not the last branch.
@classmethod
def if_(expr: [Union](flytekit_models_literals_union)[[ComparisonExpression](flytekit_models_core_condition_comparisonexpression), [ConjunctionExpression](flytekit_models_core_condition_conjunctionexpression)]) - > [Case](flytekit_core_condition_case)
-  Defines the condition for a branch within the conditional section.
- **Parameters**

  - **expr**: [Union](flytekit_models_literals_union)[[ComparisonExpression](flytekit_models_core_condition_comparisonexpression), [ConjunctionExpression](flytekit_models_core_condition_conjunctionexpression)]
    - The expression defining the condition.

- **Return Value**:
**[Case](flytekit_core_condition_case)**
  - A Case object representing the branch with the specified condition.
```@classmethod
def compute_output_vars()
```
-  Computes and returns the minimum set of outputs required for this conditional block, considering all registered cases.

- **Return Value**:
**typing.Optional[typing.List[str]]**
  - A list of output variable names, or None if outputs are not consistently defined across all branches.
