# FPON Basics: Language Design & Implementation Guide

This document expands on the core FPON concepts from the README, providing deeper dives into language semantics, patterns, edge cases, and design decisions for implementers.

## 1. Functions: The Fundamental Unit

### 1.1 Function Definition & Semantics

All functions in FPON follow the lambda calculus model: `x -> body`. The arrow operator is **right-associative**, meaning nested functions are curried by default.

```fpon
# Simple identity
x -> x

# Function that returns a function
x -> (y -> x + y)

# This is equivalent to:
x -> y -> x + y

# When called, this becomes:
let add = x -> y -> x + y in
add 5 3
# Result: 8
```

### 1.2 Currying & Partial Application

Because functions naturally curry in FPON, partial application is inherent:

```fpon
# Define a generic operation
let multiply = x -> y -> x * y in

# Partial application: create a doubler
let double = multiply 2 in

# Use the partial function
double 5
# Result: 10
```

This enables **function factories** and **composable operations**:

```fpon
let operation = op -> x -> y -> op x y in
let add = x -> y -> x + y in
let subtract = x -> y -> x - y in

let addThen = operation add in
addThen 10 3
# Result: 13
```

### 1.3 Higher-Order Functions

Functions that accept or return functions unlock powerful abstractions:

```fpon
# Map-like operation: apply a function to multiple values
let apply = f -> x -> y -> z -> {
  "result1" -> f x,
  "result2" -> f y,
  "result3" -> f z,
} in

let increment = x -> x + 1 in
apply increment 1 2 3
# Result: {
#   "result1" -> 2,
#   "result2" -> 3,
#   "result3" -> 4,
# }
```

### 1.4 Function Composition Patterns

While FPON doesn't provide explicit composition operators, you can build them:

```fpon
# Basic function composition
let compose = f -> g -> x -> f (g x) in

let double = x -> x * 2 in
let increment = x -> x + 1 in

let doubleThenIncrement = compose increment double in
doubleThenIncrement 5
# Step 1: double(5) = 10
# Step 2: increment(10) = 11
# Result: 11
```

### 1.5 Scoping & Variable Capture

Variables in FPON follow lexical scoping. Inner bindings shadow outer ones:

```fpon
let x = 10 in
let f = y -> x + y in
let x = 20 in  # This x shadows the outer x
f 5
# Result: 15 (uses the inner x = 20)
```

**Design Note**: The scoping precedence must be clarified during parsing. Does `let x = 10 in let f = y -> x + y in let x = 20 in f 5` capture the outer `x` at function definition time or lookup time? FPON should use **lexical scoping with late binding** (lookup at call time).

## 2. Advanced Variable Binding

### 2.1 Nested Let Expressions

Multiple `let` bindings can be chained:

```fpon
let a = 5 in
let b = 10 in
let c = a + b in
c * 2
# Result: 30
```

### 2.2 Let in Let Patterns

Combining nested functions with nested bindings:

```fpon
let outer = x -> (
  let inner = y -> x * y in
  let z = inner 2 in
  z + 1
) in
outer 5
# Step 1: inner = y -> 5 * y
# Step 2: z = inner 2 = 10
# Step 3: z + 1 = 11
# Result: 11
```

### 2.3 Shadowing & Re-binding

```fpon
let value = 10 in
let result = (
  let value = 20 in
  value + 5
) in
result
# Result: 25 (the inner binding is used)

# The outer value is only accessible outside the inner scope
```

**Implementation Note**: Track scoping depth during parsing. Each `let` introduces a new scope level. Variable lookup must search from innermost to outermost scope.

## 3. Maps: More Than Just Data

### 3.1 Maps as Pattern-Matching Functions

Maps are the primary composite data structure. Conceptually, a map is a function that accepts a key and returns a value:

```fpon
let config = {
  "host"     -> "localhost",
  "port"     -> 8080,
  "debug"    -> true,
  "timeout"  -> 30,
} in
config "host"
# Result: "localhost"
```

### 3.2 Pattern Matching & Wildcards

Maps execute the first matching pattern. A catch-all pattern should be supported for default behavior:

```fpon
let response = {
  200 -> "OK",
  404 -> "Not Found",
  500 -> "Server Error",
  # Default case (catch-all)
  _ -> "Unknown Status Code",
} in
response 404
# Result: "Not Found"

response 999
# Result: "Unknown Status Code"
```

**Design Note**: Should `_` be a reserved wildcard symbol? Or should any unbound variable act as a catch-all? Recommend treating `_` as an explicit catch-all to avoid accidental matches.

### 3.3 Nested Maps & Deep Navigation

```fpon
let database = {
  "users" -> {
    "john" -> {
      "email" -> "john@example.com",
      "age"   -> 30,
      "roles" -> {
        "admin"  -> true,
        "editor" -> false,
      },
    },
    "jane" -> {
      "email" -> "jane@example.com",
      "age"   -> 28,
      "roles" -> {
        "admin"  -> false,
        "editor" -> true,
      },
    },
  },
} in
database "users" "john" "roles" "admin"
# Result: true
```

### 3.4 Maps with Function Values

Maps can store functions as values, enabling method-like behavior:

```fpon
let calculator = {
  "add"      -> (x -> y -> x + y),
  "subtract" -> (x -> y -> x - y),
  "multiply" -> (x -> y -> x * y),
  "divide"   -> (x -> y -> x / y),
} in

let add = calculator "add" in
add 5 3
# Result: 8
```

Combining operations:

```fpon
let mathLib = {
  "operations" -> {
    "add"      -> (x -> y -> x + y),
    "multiply" -> (x -> y -> x * y),
  },
  "compose" -> (f -> g -> x -> f (g x)),
} in

let add = mathLib "operations" "add" in
let multiply = mathLib "operations" "multiply" in
let compose = mathLib "compose" in

let addThenMultiply = compose (multiply 2) (add 5) in
addThenMultiply 10
# Step 1: add 5 10 = 15
# Step 2: multiply 2 15 = 30
# Result: 30
```

### 3.5 Maps with Computed Values

Maps can use `let` expressions to build complex structures:

```fpon
let config = 
  let baseURL = "https://api.example.com" in
  let timeout = 5000 in
  let retries = 3 in
  {
    "baseURL"   -> baseURL,
    "timeout"   -> timeout,
    "retries"   -> retries,
    "fullURL"   -> baseURL + "/v1/data",
    "endpoints" -> {
      "users"   -> baseURL + "/users",
      "posts"   -> baseURL + "/posts",
      "comments" -> baseURL + "/comments",
    },
  }
in
config "endpoints" "users"
# Result: "https://api.example.com/users"
```

### 3.6 Conditional Maps (If-Then-Else Pattern)

While FPON doesn't have explicit if-then-else, maps can simulate it:

```fpon
let abs = x -> (
  {
    true  -> x,
    false -> x * -1,
  } (x >= 0)
) in
abs -5
# Result: 5
```

**Design Note**: This pattern relies on boolean values being used as keys. Should booleans automatically convert to string keys, or should maps support multiple key types? Consider implementing type coercion or requiring explicit string conversion.

## 4. Type System Considerations

### 4.1 Primitive Types

FPON should support the following primitives:

- **Numbers**: integers and floats (`42`, `3.14`)
- **Strings**: enclosed in quotes (`"hello"`)
- **Booleans**: `true`, `false`
- **Null**: `null`

```fpon
let data = {
  "number"  -> 42,
  "float"   -> 3.14,
  "string"  -> "hello",
  "bool"    -> true,
  "nothing" -> null,
} in
data "string"
# Result: "hello"
```

### 4.2 Type Coercion & Conversion

How should the language handle type mismatches?

```fpon
# Should this work? Does the key auto-convert to a string?
let obj = {
  "name" -> "Alice",
} in
obj 123  # or obj "123"?
```

**Recommendation**: Require explicit string keys for maps. Allow implicit number/boolean/string conversions in arithmetic operations.

### 4.3 Type Errors & Edge Cases

**Missing Keys**:
```fpon
let obj = {
  "a" -> 1,
} in
obj "b"
# Should this throw an error or return null?
```

**Recommendation**: Throw a clear "Key not found" error during interpretation. Include the attempted key and available keys in the error message.

**Type Mismatches in Operations**:
```fpon
let result = "hello" + 5
# Should this error or coerce?
```

**Recommendation**: Throw a type error. Don't allow implicit coercion between incompatible types.

## 5. Common Patterns & Idioms

### 5.1 Configuration Objects

FPON excels at representing nested configuration:

```fpon
let appConfig = {
  "name"    -> "MyApp",
  "version" -> "1.0.0",
  "server" -> {
    "host"     -> "0.0.0.0",
    "port"     -> 8080,
    "ssl"      -> true,
    "certPath" -> "/etc/ssl/certs",
  },
  "database" -> {
    "driver"   -> "postgres",
    "host"     -> "localhost",
    "port"     -> 5432,
    "name"     -> "appdb",
    "poolSize" -> 20,
  },
  "logging" -> {
    "level"    -> "info",
    "format"   -> "json",
    "outputs" -> {
      "console" -> true,
      "file"    -> "/var/log/app.log",
    },
  },
} in
appConfig "database" "host"
# Result: "localhost"
```

### 5.2 Data Transformation Pipelines

Chain functions to transform data:

```fpon
let pipeline = input -> (
  let step1 = input * 2 in
  let step2 = step1 + 10 in
  let step3 = step2 / 5 in
  step3
) in
pipeline 15
# Step 1: 15 * 2 = 30
# Step 2: 30 + 10 = 40
# Step 3: 40 / 5 = 8
# Result: 8
```

### 5.3 Builder Pattern

Accumulate values using nested functions:

```fpon
let builder = (
  let init = x -> (
    let addX = y -> x + y in
    let multiplyBy = z -> x * z in
    {
      "value"      -> x,
      "addX"       -> addX,
      "multiplyBy" -> multiplyBy,
      "build"      -> x,
    }
  ) in
  init
) in

let result = builder 5 in
result "addX" 3
# Result: 8
```

### 5.4 Enumeration-Like Maps

Use maps to represent enums with associated behavior:

```fpon
let status = {
  "pending" -> {
    "code"        -> 0,
    "description" -> "Awaiting processing",
    "retriable"   -> true,
  },
  "success" -> {
    "code"        -> 1,
    "description" -> "Operation completed",
    "retriable"   -> false,
  },
  "failed" -> {
    "code"        -> 2,
    "description" -> "Operation failed",
    "retriable"   -> true,
  },
} in
status "success" "description"
# Result: "Operation completed"
```

## 6. Error Handling & Edge Cases

### 6.1 Undefined Variables

```fpon
let result = undefinedVar + 5
# Should throw: "Variable 'undefinedVar' is not defined"
```

**Implementation**: Track all bound variables during parsing. At interpretation, raise an error for unbound variable access.

### 6.2 Arity Mismatches

```fpon
let f = x -> y -> x + y in
f 5  # Returns a partial application
# Result: (y -> 5 + y)

f 5 3  # Full application
# Result: 8

f 5 3 7  # Extra argument
# What happens here?
```

**Design Decision**: Should extra arguments be ignored, or should this error? Recommendation: **Allow** extra arguments but only pass what the function accepts (or implement function chaining where the result becomes the input to the next argument).

### 6.3 Circular References

```fpon
let x = x + 1 in
x
# Infinite recursion during evaluation
```

**Implementation**: Detect during evaluation. Set a maximum recursion depth and throw a stack overflow error.

### 6.4 Map Pattern Ambiguity

What if multiple patterns could match?

```fpon
let obj = {
  "a" -> 1,
  "a" -> 2,  # Duplicate key
} in
obj "a"
# Which value is returned?
```

**Recommendation**: The **first matching pattern** is used. Duplicates should trigger a **compile-time warning** or error, depending on strictness settings.

### 6.5 Operations on Incompatible Types

```fpon
let result = true + "string"
# Type error: cannot add boolean and string

let result = {
  "a" -> 1,
} + 5
# Type error: cannot add map and number
```

**Implementation**: Perform type checking during evaluation. Provide clear error messages with the types involved.

## 7. Parsing & Whitespace

### 7.1 Whitespace Insignificance

Whitespace is fully insignificant except within string literals:

```fpon
# All equivalent:
x -> x + 1
x->x+1
x   ->   x   +   1

# Multi-line
x -> 
  x + 1

# Indentation for readability (ignored)
let config = {
  "a" -> 1,
  "b" -> 2,
} in
config "a"
```

### 7.2 Comments

Line comments use `#`:

```fpon
# This is a comment
let x = 5 in  # Inline comment
x + 1
# Result: 6
```

**Design Note**: Should block comments (/* */) be supported? For now, recommend only line comments for simplicity.

### 7.3 String Literals

Strings use double quotes. How should escape sequences be handled?

```fpon
let greeting = "Hello\nWorld" in
greeting
# Does this support \n, \t, \\, \", etc.?
```

**Recommendation**: Support standard escape sequences (`\n`, `\t`, `\\`, `\"`, `\'`).

## 8. Operator Precedence & Associativity

### 8.1 Arithmetic Operators

Standard precedence (highest to lowest):
1. `*`, `/` (multiplication, division)
2. `+`, `-` (addition, subtraction)

```fpon
2 + 3 * 4
# Result: 14 (not 20)

10 - 5 - 2
# Result: 3 (left-associative: (10 - 5) - 2)
```

### 8.2 Comparison Operators

Comparison operators (`==`, `!=`, `<`, `>`, `<=`, `>=`) should return booleans:

```fpon
5 > 3
# Result: true

10 == 10
# Result: true
```

**Design Note**: Clarify precedence relative to arithmetic. Recommend: arithmetic binds tighter than comparison.

### 8.3 Arrow Operator (Right-Associative)

The arrow operator is **right-associative**:

```fpon
x -> y -> z -> x + y + z

# Equivalent to:
x -> (y -> (z -> x + y + z))
```

This is crucial for currying.

### 8.4 Function Application (Left-Associative)

Function application associates left:

```fpon
f x y z

# Equivalent to:
((f x) y) z
```

## 9. Implementation Strategy

### 9.1 Parser Output

The parser should produce an Abstract Syntax Tree (AST) with nodes for:
- **Literals**: numbers, strings, booleans, null
- **Variables**: identifiers
- **Functions**: `Fn { param, body }`
- **Applications**: `App { func, arg }`
- **Let-bindings**: `Let { var, value, body }`
- **Maps**: `Map { patterns }` where patterns are key-value pairs
- **Binary operations**: `BinOp { op, left, right }`

### 9.2 Interpreter Evaluation

Use an environment/scope map during evaluation:

```
eval(env, expr):
  match expr:
    Literal(v) -> v
    Variable(name) -> lookup(env, name)
    Fn(param, body) -> closure(param, body, env)
    App(func, arg) -> 
      f = eval(env, func)
      a = eval(env, arg)
      call(f, a)
    Let(var, value, body) ->
      v = eval(env, value)
      new_env = extend(env, var, v)
      eval(new_env, body)
    Map(patterns) -> 
      // Create a function that pattern-matches on keys
    BinOp(op, left, right) ->
      l = eval(env, left)
      r = eval(env, right)
      apply_op(op, l, r)
```

### 9.3 Error Reporting

Maintain source location info in the AST:

```
struct Node {
  expr: Expr,
  line: usize,
  column: usize,
}

struct Error {
  message: String,
  location: (usize, usize),
  context: String,  // Line of source code
}
```

## 10. Future Considerations

### 10.1 Pattern Matching in Functions

Could FPON support pattern matching in function parameters?

```fpon
# Hypothetical: destructure on input
{x, y} -> x + y

# Or match on shape:
{name: n, age: a} -> n
```

### 10.2 Type Annotations

Should type annotations be optional or required?

```fpon
# Hypothetical:
let add = (x: Number) -> (y: Number) -> Number = x -> y -> x + y
```

### 10.3 Recursion

How should recursion be handled? Does FPON support named recursion?

```fpon
# Hypothetical: Y-combinator pattern
let factorial = (f -> n -> (n == 0) -> {
  true -> 1,
  false -> n * f (n - 1),
}) in
factorial factorial 5
```

### 10.4 Standard Library

What built-in functions should FPON provide? (length, contains, map, filter, etc.)

---

This guide captures the core semantics, patterns, and design decisions needed to implement FPON's parser and interpreter. Use it as a reference while building the language's systems.
