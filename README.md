# Pydantic RPN

A delightful Python library for Reverse Polish Notation with Pydantic integration.

## Features

- 🚀 **Intuitive API**: Multiple ways to create and evaluate RPN expressions
- 🔧 **Method Chaining**: Fluent interface for building complex expressions
- 🔒 **Type Safety**: Full Pydantic integration with validation
- 📊 **Rich Operations**: Arithmetic, trigonometry, stack manipulation, and more
- 🔄 **Flexible I/O**: JSON serialization, infix conversion, template support
- 🎯 **Variable Support**: Dynamic variable substitution and partial evaluation
- 🧪 **Comprehensive Testing**: Extensive test coverage with property-based testing

## Quick Start

```bash
pip install pydantic-rpn
```

```python
from pydantic_rpn import RPN, rpn

# Basic usage
result = RPN("3 4 +").eval()  # 7
result = rpn("3 4 +")()       # 7 (shorthand)

# Builder pattern for chaining
result = (RPNBuilder()
    .push(3).push(4).add()
    .push(2).mul().eval())  # 14

# Variables
expr = RPN("x 2 * y +")
result = expr.eval(x=5, y=3)  # 13

# Builder pattern
result = (RPNBuilder()
    .push(10)
    .push(20)
    .add()
    .push(2)
    .div()
    .eval())  # 15
```

## Expression Creation

```python
# Multiple construction methods
expr = RPN("3 4 +")        # String
expr = RPN([3, 4, "+"])    # List
expr = rpn("3 4 +")        # Convenience function

# Template expressions
template = RPN.template("${price} ${tax} +")
result = template.eval(price=100, tax=10)  # 110
```

## Operations

### Arithmetic
```python
rpn("10 5 +").eval()   # 15 (addition)
rpn("10 5 -").eval()   # 5  (subtraction)  
rpn("10 5 *").eval()   # 50 (multiplication)
rpn("10 5 /").eval()   # 2  (division)
rpn("2 3 **").eval()   # 8  (power)
```

### Mathematical Functions
```python
rpn("25 sqrt").eval()  # 5
rpn("-5 abs").eval()   # 5
rpn("pi sin").eval()   # ~0
rpn("e ln").eval()     # 1
```

### Stack Operations
```python
rpn("5 dup *").eval()      # 25 (duplicate and multiply)
rpn("3 4 swap -").eval()   # 1  (swap and subtract)
rpn("3 4 5 drop +").eval() # 7  (drop top, then add)
```

## Method Chaining

```python
# Build expressions fluently with RPNBuilder
from pydantic_rpn import RPNBuilder

expr = (RPNBuilder()
    .push(3).push(4)  # 3 4
    .add()           # 3 4 +
    .push(2)         # 3 4 + 2
    .mul()           # 3 4 + 2 *
    .sqrt())         # 3 4 + 2 * sqrt

result = expr.eval()  # sqrt(14) ≈ 3.74
```

## Variables and Templates

```python
# Variables with defaults
expr = RPN("x y +", defaults={"x": 10})
result = expr.eval(y=5)  # 15

# Partial evaluation
expr = RPN("x 2 * y +")
partial = expr.partial(x=5)  # Fix x=5
result = partial.eval(y=3)   # 13

# Template placeholders
template = RPN.template("${a} ${b} + ${c} *")
result = template.eval(a=3, b=4, c=2)  # 14
```

## Expression Composition

```python
# Combine expressions
expr1 = RPN("3 4 +")
expr2 = RPN("2 *") 
combined = expr1 + expr2  # "3 4 + 2 *"

# Pipe results
result = RPN("3 4 +") | RPN("ans 2 *")  # 14
```

## Validation and Safety

```python
# Strict validation (default)
try:
    RPN("3 +", strict=True)  # ValidationError
except ValidationError as e:
    print(f"Invalid: {e}")

# Manual validation
expr = RPN("3 4 +", strict=False)
errors = expr.validate_expression()  # []

# Type constraints and safety
from pydantic_rpn import Config

with Config(max_stack_size=10):
    result = expr.eval()  # Limited stack depth
```

## Serialization

```python
# JSON serialization
expr = RPN("3 4 +", metadata={"author": "alice"})
json_str = expr.to_json()
restored = RPN.from_json(json_str)

# Notation conversion
expr = RPN("3 4 + 2 *")
print(expr.to_infix())   # "((3 + 4) * 2)"
print(expr.to_prefix())  # "+ 3 4"
```

## Interactive REPL

```python
from pydantic_rpn import REPL

repl = REPL()
repl.run()
```

```
rpn> 3 4 +
7
rpn> ans 2 *  
14
rpn> x = 10
x = 10
rpn> x 2 /
5.0
```

## Command Line Interface

```bash
# Evaluate expressions
rpn "3 4 +"                    # 7

# Use variables  
rpn "x 2 *" --var x=5         # 10

# Interactive mode
rpn --repl

# Show infix notation
rpn "3 4 + 2 *" --infix       # ((3 + 4) * 2)

# Validate only
rpn "3 4 +" --validate        # Expression is valid
```

## Builder Pattern

```python
from pydantic_rpn import RPNBuilder

# Complex expressions
quadratic = (RPNBuilder()
    .var("b").neg()                    # -b
    .var("b").push(2).pow()            # b²  
    .push(4).var("a").mul().var("c").mul()  # 4ac
    .sub().sqrt()                      # sqrt(b² - 4ac)
    .add()                             # -b + sqrt(...)
    .push(2).var("a").mul()            # 2a
    .div()                             # final division
    .build())

root = quadratic.eval(a=1, b=-5, c=6)  # Quadratic formula
```

## Integration Examples

### Pydantic Models
```python
from pydantic import BaseModel

class Calculation(BaseModel):
    name: str
    formula: RPN
    
calc = Calculation(name="pythagorean", formula="a 2 ** b 2 ** + sqrt")
result = calc.formula.eval(a=3, b=4)  # 5.0
```

### Pandas Integration
```python
import pandas as pd

df = pd.DataFrame({"x": [1, 2, 3], "y": [4, 5, 6]})
expr = RPN("x 2 ** y +")  # x² + y

# Apply to dataframe
df["result"] = [expr.eval(x=row.x, y=row.y) for _, row in df.iterrows()]
# [5, 9, 15]
```

## Advanced Features

### Custom Constants and Variables
```python
# Use variables for custom behavior
expr = RPN("x y +")
avg_result = expr.eval(x=10, y=20) / 2  # 15

# Or use defaults for custom constants
expr = RPN("pi 2 *", defaults={"pi": 3.14159})
result = expr.eval()  # 6.28318
```

### Error Handling
```python
from pydantic_rpn import RPNError, EvaluationError

try:
    result = RPN("5 0 /").eval()
except EvaluationError as e:
    print(f"Math error: {e}")
```

### Configuration
```python
from pydantic_rpn import Config

# Global settings
Config.set_defaults(
    precision=4,
    angle_unit="degrees",
    strict=True
)

# Temporary settings
with Config(precision=2):
    result = RPN("1 3 /").eval()  # 0.33
```

## Performance

```python
# Reuse expressions for better performance
expr = RPN("x 2 ** y 2 ** +")  # x² + y²

# Evaluate multiple times efficiently
results = []
for x, y in [(0, 0), (3, 4), (5, 12)]:
    results.append(expr.eval(x=x, y=y))
# [0.0, 25.0, 169.0]
```

## Contributing

Contributions welcome! Please see [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

## License

MIT License - see [LICENSE](LICENSE) for details.