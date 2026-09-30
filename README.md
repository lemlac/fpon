FPON (Functional Programming Object Notation) is a purely functional form of object notation similar to JSON. This is an experimental language to test a new way of looking at data. 

The basic element of FPON is the function, notated using an arrow (`->`, *hyphen + greater-than sign*). Functions in FPON must have exactly 1 input and 1 output. Comments can be written using a hash (`#`) which comments out text to the end of the line.

```fpon
# Identity function:
x -> x
```

Every FPON file must only contain one expression, similar to JSON or Nix. Variables can be declared with the `let _ = _ in` expression. Placing an argument after a function will call it.

```fpon
let addOne = x -> x + 1 in addOne 2   # Result: 2 + 1 which is 3
```

Most languages have some kind of a map type, sometimes called "objects" or "dictionaries". This type takes 1 input (a key) and returns 1 output (a value). In a way, a map is a kind of function. FPON takes this idea to heart. **Maps** in FPON are a collection of functions which use pattern matching to call the first matching function. One type of pattern is a string literal. Maps are marked with curly braces `{}` and each function in it is separated by commas (`,`).

```fpon
{
  "status" -> "success",
  # Nested map
  "data" -> {
    "id" -> 1234,
    "sentiment" -> "positive",
    "confidence-score" -> 0.96,
    "summary" -> "The user is highly satisfied with the new update, specifically praising the faster loading speeds and sleek UI overhaul.",
    "locked" -> false,
    "parent" -> null,
  },
}
```

Accessing a map is the same as calling a functions. Use multiple arguments to get from a nested map.

```fpon
let obj = {
  "a" -> {
    "b" -> {
      "c" -> "value"
    }
  }
} in o "a" "b" "c"     # Result: "value"
```

This is a rough outline of the language so far. Feedback is welcomed: either through the [issues](https://github.com/lemlac/fpon/issues) page or contact me directly via [email](mailto:13686726+lemlac@users.noreply.github.com).
