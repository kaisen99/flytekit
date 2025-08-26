# Case

This class represents a conditional case within a conditional section. It encapsulates an expression and the actions to be taken if the expression evaluates to true. The class supports comparison and conjunction expressions. It interacts with a ConditionalSection to manage the control flow of conditional logic.

## Attributes

- **cs**: [ConditionalSection](flytekit_core_condition_conditionalsection)
  - The conditional section this case belongs to.

- **expr**: Optional[[Union](flytekit_models_literals_union)[[ComparisonExpression](flytekit_models_core_condition_comparisonexpression), [ConjunctionExpression](flytekit_models_core_condition_conjunctionexpression)]]
  - The expression to evaluate for this case. Can be a ComparisonExpression or ConjunctionExpression, or None.

- **stmt**: str = &#x27;elif&#x27;
  - The statement type, defaults to &#x27;elif&#x27;.

- **_cs**: [ConditionalSection](flytekit_core_condition_conditionalsection)
  - Internal reference to the conditional section.

- **_expr**: Optional[[Union](flytekit_models_literals_union)[[ComparisonExpression](flytekit_models_core_condition_comparisonexpression), [ConjunctionExpression](flytekit_models_core_condition_conjunctionexpression)]]
  - Internal storage for the expression.

- **_output_promise**: Optional[[Union](flytekit_models_literals_union)[Tuple[[Promise](flytekit_core_promise_promise)], [Promise](flytekit_core_promise_promise)]]
  - Internal storage for the output promise.

- **_err**: Optional[str]
  - Internal storage for any error message.

- **_stmt**: str
  - Internal storage for the statement type.

- **_output_node**: Optional[[Node](flytekit_models_core_workflow_node)]
  - Internal pointer to the node that created this case.

## Constructors
def Case(cs: [ConditionalSection](flytekit_core_condition_conditionalsection), expr: Optional[[Union](flytekit_models_literals_union)[[ComparisonExpression](flytekit_models_core_condition_comparisonexpression), [ConjunctionExpression](flytekit_models_core_condition_conjunctionexpression)]], stmt: str = elif)
-  Initializes a Case object with a conditional section, an optional expression, and a statement string.
- **Parameters**

  - **cs**: [ConditionalSection](flytekit_core_condition_conditionalsection)
    - The conditional section this case belongs to.
  - **expr**: Optional[[Union](flytekit_models_literals_union)[[ComparisonExpression](flytekit_models_core_condition_comparisonexpression), [ConjunctionExpression](flytekit_models_core_condition_conjunctionexpression)]]
    - The expression to evaluate for this case. Supports Comparison or Conjunction expressions. Raises AssertionError for logical operations or evaluated expressions.
  - **stmt**: str
    - The statement type, defaults to &#x27;elif&#x27;.



## Methods
```@classmethod
def output_node()
```
-  This is supposed to hold a pointer to the node that created this case. It is set in the then() call. but the value will not be set if it&#x27;s a VoidPromise or None was returned.

- **Return Value**:
**Optional[[Node](flytekit_models_core_workflow_node)]**
  - The node that created this case.
```@classmethod
def expr()
```
-  Returns the expression associated with this case.

- **Return Value**:
**Optional[[Union](flytekit_models_literals_union)[[ComparisonExpression](flytekit_models_core_condition_comparisonexpression), [ConjunctionExpression](flytekit_models_core_condition_conjunctionexpression)]]**
  - The expression for this case.
```@classmethod
def output_promise()
```
-  Returns the output promise of this case.

- **Return Value**:
**Optional[[Union](flytekit_models_literals_union)[Tuple[[Promise](flytekit_core_promise_promise)], [Promise](flytekit_core_promise_promise)]]**
  - The output promise of this case.
```@classmethod
def err()
```
-  Returns the error message associated with this case, if any.

- **Return Value**:
**Optional[str]**
  - The error message.
@classmethod
def then(p: [Union](flytekit_models_literals_union)[[Promise](flytekit_core_promise_promise), Tuple[[Promise](flytekit_core_promise_promise)]]) - > Optional[[Union](flytekit_models_literals_union)[[Condition](flytekit_core_condition_condition), [Promise](flytekit_core_promise_promise), Tuple[[Promise](flytekit_core_promise_promise)], [VoidPromise](flytekit_core_promise_voidpromise)]]
-  Sets the output promise for this case and returns the end of the current branch.
- **Parameters**

  - **p**: [Union](flytekit_models_literals_union)[[Promise](flytekit_core_promise_promise), Tuple[[Promise](flytekit_core_promise_promise)]]
    - The promise representing the output of this case.

- **Return Value**:
**Optional[[Union](flytekit_models_literals_union)[[Condition](flytekit_core_condition_condition), [Promise](flytekit_core_promise_promise), Tuple[[Promise](flytekit_core_promise_promise)], [VoidPromise](flytekit_core_promise_voidpromise)]]**
  - The result of ending the current branch.
@classmethod
def fail(err: str) - > [Promise](flytekit_core_promise_promise)
-  Sets the error message for this case and returns the end of the current branch.
- **Parameters**

  - **err**: str
    - The error message.

- **Return Value**:
**[Promise](flytekit_core_promise_promise)**
  - The result of ending the current branch.
