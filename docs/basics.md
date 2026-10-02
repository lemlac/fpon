# FPON Basics: Language Design

This document expands on the core FPON concepts from the [README](../README.md), providing deeper dives into language semantics, patterns, edge cases, and design decisions for implementers.

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

FPON uses **lexical scoping with late binding** (lookup at call time). That means `let x = 10 in let f = y -> x + y in let x = 20 in f 5` captures the outer `x` at function definition time or lookup time.

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

Scoping depth is tracked during parsing. Each `let` introduces a new scope level. Variable lookup must search from innermost to outermost scope.

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

`_` is a reserved wildcard symbol. It's used as an explicit catch-all to avoid accidental matches.

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

The keys of each map are patterns. A pattern can either be a literal -- such as strings, numbers, or (as in this example) Booleans -- or be a variable like `x` (or `_` when the variable is unused).

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

### 4.2 Type Errors & Edge Cases

**Missing Keys**:

```fpon
let obj = {
  "a" -> 1,
} in
obj "b"
```

If a pattern doesn't match in a map, then a "Pattern not found" error will throw during interpretation. The attempted key and available keys will also be included in the error message.

**Type Mismatches in Operations**:

```fpon
let result = "hello" + 5
```

If an operation on two types doesn't exist (like in this case `Number + Number` and `String + String` exists but not `String + Number`) then a type error will be thrown. Implicit coercion between incompatible types is not allowed.

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
# Error: "Variable 'undefinedVar' is not defined"
```

All bound variables are tracked during parsing. At interpretation, an error is raised for unbound variable access.

### 6.2 Arity Mismatches

```fpon
let f = x -> y -> x + y in
f 5  # Returns a partial application
# Result: (y -> 5 + y)

f 5 3  # Full application
# Step 1: (y -> 5 + y) 3
# Step 2: 5 + 3
# Result: 8

f 5 3 7  # Extra argument
# Step 1: (y -> 5 + y) 3 7
# Step 2: (5 + 3) 7
# Step 3: 8 7
# Error: Couldn't match expected type ‘a -> b’ with actual type ‘Number’.
```

Functions are chained, so adding too many arguments *usually* results in an error. If result type can't be called like a function (for example primitives such as numbers, strings, and booleans), then having an extra argument will result in a type error. 

### 6.3 Map Pattern Ambiguity

Maps can have duplicate keys since keys are really patterns for each function within the map. The **first matching pattern** will be used.

```fpon
let obj = {
  "a" -> 1,
  "a" -> 2,  # Duplicate key
} in
obj "a"
# Result: 1
```

Potentially, FPON could support a strictness setting so that duplicates trigger a **compile-time warning**, but for now this is legal. 

Recall that maps in FPON are functions. In another language (like JavaScript in this next example) it would be equivalent to this:

```js
function obj(key) {
    if (key === "a") return 1;
    if (key === "a") return 2;  // Unreachable
}
obj("a")
// Result: 1
```

So in FPON, maps check each function that they have and escape at the first pattern match, much like what `return` does in the JavaScript example.

### 6.4 Operations on Incompatible Types

```fpon
true + "string"
# Type error: cannot add boolean and string
```

```fpon
{
  "a" -> 1,
} + 5
# Type error: cannot add map and number
```

Type checking is performed during evaluation. Clear error messages will be provided with the types involved.

## 7. Parsing & Whitespace

### 7.1 Whitespace Insignificance

Whitespace is fully insignificant except within string literals.

**All equivalent:**
- `x -> x + 1`
- `x->x+1`
- `x   ->   x   +   1`
- ```fpon
  # Multi-line
  x -> 
    x + 1
  ```

Indentation is useful for readability but is ultimately ignored.

```fpon
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

Block comments are not supported. For now, there are only line comments. 

### 7.3 String Literals

Strings use double quotes. Escaping is handed with a backslash `\`.

```fpon
"Hello\nWorld"
```

Standard escape sequences (`\n`, `\t`, `\\`, `\"`, `\'`, etc.) are supported.

Strings literals be defined in a single line. Line breaks can be escaped with `\n`, but literal line breaks in a string result in a syntax error.

```fpon
"Hello
world"
# Error: line break literal within string
```

Curly braces within a string literal calls on **string interpolation.** A literal curly brace can be escaped with a backslash like this `\{` which disables string interpolation. Whatever value is between the curly braces will get implicitly converted to a string, so explicit conversion isn't necessary. 

```fpon
let name = "Bob" in
let score = 95 in
"{name} scored {score} points"
# Result: "Bob scored 95 points."
```

Outside of a string literal, the `\\` syntax is used to define **raw multi-line string literals.**

**Core Rules of `\\` Strings:**

- **No Escape Sequences:** Everything inside a multi-line string literal is parsed exactly as written. For example, `\n` or `\t` inside the block are treated as literal backslashes followed by the letters 'n' or 't', rather than a newline or a tab. 
- **Implicit Newlines:** The compiler automatically adds a newline character (`\n`) at the end of each line, except for the very last line.
- **Stripped Leading Whitespace:** Any indentation before the `\\` is ignored by the compiler, allowing you to align your code neatly without introducing unwanted padding into the string itself.
- **Line Termination:** A `\\` string continues until a line that doesn't start with `\\`. Each `\\`-prefixed line becomes a line in the same string, with an automatic newline appended (except after the final line). An empty `\\` with nothing after it produces a blank line in the output.

Basic Usage Example:

```fpon
# The compiler reads this as a single string with embedded newlines
let text =
  \\Line 1: Hello World!
  \\Line 2: Escape sequences like \n don't work here.
  \\Line 3: This behaves similarly to line comments.
in text
```

The main use case for this would be for embedding documents as strings within a configuration file.

A couple more examples:

```fpon
{
  "html_template" ->
    \\<!DOCTYPE html>
    \\<html>
    \\  <body>
    \\    <h1>Hello From FPON</h1>
    \\  </body>
    \\</html>
    , # Comma needs to be on the next line after the last line of the raw string.
  "file_name" -> "index.html",
  "output_directory" -> "./dist/public",
  "file_size_bytes" -> 104,
  "created_at" -> "2026-10-02T18:35:00Z",
  "updated_at" -> "2026-10-02T18:35:00Z",
}
```

```fpon
{
  "name" -> "API Health Check Pipeline",
  "version" -> "1.2.0",

  "metadata" -> {
    "description" -> "Automated script to verify external service availability",
    "author" -> "DevOps Team",
  },

  "config" -> {
    "timeout_minutes" -> 5,
    "retry_attempts" -> 3,
    "allow_failure" -> false,
    "environment" -> {
      "STAGE" -> "production",
      "LOG_LEVEL" -> "DEBUG",
    },
  },

  # Shell script
  "run" ->
    \\CURL="/bin/curl"
    \\JQ="/bin/jq"
    \\
    \\echo "Checking GitHub API Status..."
    \\
    \\# Fetch data and parse it using the interpolated tools
    \\$CURL -s "https://api.github.com"
    \\
    \\echo ""
    \\echo "Script executed successfully!"
}
```

## 8. Operator Precedence & Associativity

### 8.1 Arithmetic Operators

Standard precedence (highest to lowest):
1. `*`, `/` (multiplication, division)
2. `+`, `-` (addition, subtraction)

```fpon
2 + 3 * 4
# Result: 14 (not 20)
```

```fpon
10 - 5 - 2
# Result: 3 (left-associative: (10 - 5) - 2)
```

### 8.2 Comparison Operators

Comparison operators (`==`, `!=`, `<`, `>`, `<=`, `>=`) should return booleans:

```fpon
5 > 3
# Result: true
```

```fpon
10 == 10
# Result: true
```

Arithmetic binds tighter than comparison.

```
let value = 10 in
value + 1 > 10
# Step 1: 10 + 1 > 10
# Step 2: 11 > 10
# Result: true
```

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

---

This guide captures the core semantics, patterns, and design decisions needed to implement FPON's parser and interpreter. Use it as a reference while building the language's systems.
