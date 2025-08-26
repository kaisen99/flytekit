# ComparisonExpression

This class represents a comparison expression, such as (lhs operator rhs). It facilitates the evaluation of comparisons between operands using various comparison operators. The class supports operands that can be either literal values or Promises, enabling dynamic evaluation. It also provides methods for logical operations like AND and OR, and requires the use of comparison operators.

## Attributes

- **rhs**: [Union](flytekit_models_literals_union)["Promise", _literals_models.Literal]
  - rhs refers to the right hand side operand in a comparison expression.

- **lhs**: [Union](flytekit_models_literals_union)["Promise", _literals_models.Literal]
  - lhs refers to the left hand side operand in a comparison expression.

- **op**: [ComparisonOps](flytekit_core_promise_comparisonops)
  - op refers to the comparison operator used in the comparison expression.

## Constructors
def ComparisonExpression(lhs: [Union](flytekit_models_literals_union)["Promise", Any], op: [ComparisonOps](flytekit_core_promise_comparisonops), rhs: [Union](flytekit_models_literals_union)["Promise", Any])
-  ComparisonExpression refers to an expression of the form (lhs operator rhs), where lhs and rhs are operands and operator can be any comparison expression like &lt; , &gt;, &lt; =, &gt;=, ==, !=
- **Parameters**

  - **lhs**: [Union](flytekit_models_literals_union)["Promise", Any]
    - The left-hand side operand of the comparison.
  - **op**: [ComparisonOps](flytekit_core_promise_comparisonops)
    - The comparison operator (e.g., &lt; , &gt;, ==).
  - **rhs**: [Union](flytekit_models_literals_union)["Promise", Any]
    - The right-hand side operand of the comparison.



## Methods
```@classmethod
def rhs()
```
-  Getter for rhs

- **Return Value**:
**[Union](flytekit_models_literals_union)["Promise", _literals_models.Literal]**
  - The right-hand side operand.
```@classmethod
def lhs()
```
-  Getter for lhs

- **Return Value**:
**[Union](flytekit_models_literals_union)["Promise", _literals_models.Literal]**
  - The left-hand side operand.
```@classmethod
def op()
```
-  Getter for op

- **Return Value**:
**[ComparisonOps](flytekit_core_promise_comparisonops)**
  - The comparison operator.
```@classmethod
def eval()
```
-  Evaluates the comparison expression.

- **Return Value**:
**bool**
  - The boolean result of the comparison.
