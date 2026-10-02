# FPON (Functional Programming Object Notation)
FPON is an experimental, purely functional data notation language. It explores a unified approach to data representation by treating data structures and functions as the exact same concept. [Permalink to the playground](https://play.rust-lang.org/?version=stable&mode=debug&edition=2024&gist=ab8a2bfc81fc27bb0c67a6ee08223608)

Every FPON file evaluates to exactly one expression, making it a clean configuration and data serialization format similar to JSON or Nix.

## 1. Everything is a Function
The fundamental building block of FPON is the function, written using an arrow (`->`). Every function must have exactly one input and exactly one output.
Line comments are supported using the hash (`#`) symbol.

```fpon
# The identity function: takes x, returns x
x -> x
```

## 2. Variable Definitions & Call Syntax
You can declare local variables using a `let <variable> = <expression> in <body expression>`. Whitespace is insignificant, so the body expression can be optionally placed on the next line. To call a function, simply place the argument immediately after it.

```fpon
# Define a function that increments a number, then call it with 2
let addOne = x -> x + 1 in
let value = 2 in
addOne value
# Result: 3
```

## 3. Maps are Pattern-Matching Functions
In traditional languages, maps (objects or dictionaries) map a key to a value. In FPON, a map is literally a function that accepts a key as an input and returns a value as an output.

Maps are enclosed in curly braces `{}`. Inside, they contain a comma-separated collection of functions that use pattern matching. When called, the map executes the first function whose pattern matches the input.

```fpon
{
  "status" -> "success",
  
  # Nested map configuration
  "data" -> {
    "id"               -> 1234,
    "sentiment"        -> "positive",
    "confidence-score" -> 0.96,
    "summary"          -> "The user praised the faster loading speeds and sleek UI overhaul.",
    "is-locked"        -> false,
    "parent-id"        -> null, # Trailing commas are fully supported
  },
}
```

## 4. Navigating Deep Data
Because maps are just functions, looking up a key is identical to invoking a function. To dig into nested maps, simply pass multiple arguments sequentially.

```fpon
let userProfile = {
  "account" -> {
    "preferences" -> {
      "theme" -> "dark"
    }
  }
} in
# Pass keys sequentially to traverse the structure
userProfile "account" "preferences" "theme"
# Result: "dark"
```

## 5. And more..

See [Basics](#./docs/basics.md) for more information on this and other planned features of FPON. 

## 🤝 Feedback & Contributing
FPON is highly experimental, and your feedback is incredibly valuable!

* Bug Reports & Feature Requests: Please open an [issue](https://github.com/lemlac/fpon/issues) on GitHub.
* Direct Contact: Feel free to reach out to the author via [email](13686726+lemlac@users.noreply.github.com).

Also see [use cases](./docs/use-cases.md).
