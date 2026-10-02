Pipe operator `|>` example:

```fpon
let append_space = num -> str -> str + (" " * num) in
"prefix:" |> append_space 3
# Rwsult: "prefix:   "
```

Fallback operator `<|>` example:

```fpon
let defaults = { "theme" -> "light", "lang" -> "en" } in
let overrides = { "theme" -> "dark" } in
let config = overrides <|> defaults in
{
  "theme" -> config "theme",   # "dark"  (left wins)
  "lang"  -> config "lang",    # "en"    (falls through to defaults)
  "other" -> config "other",   # no match in either, so a miss
}
```

Since maps are also functions, `|>` can be applied to a map literal to effectively get a switch/match statement.

```fpon
let fruit = "apple" in
fruit |> {
  "apple" | "banana" | "orange" -> "This is a fruit.",
  "broccoli" | "carrot" -> "This is a vegetable.",
  _ -> "Unknown food item.",
}
# Result: "This is a fruit."
```

Ternary pipe operator `<value> ?> <function> : <default value>` 

Here's the previous example rewritten with this operator:

```fpon
let fruit = "apple" in
fruit ?> {
  "apple" | "banana" | "orange" -> "This is a fruit.",
  "broccoli" | "carrot" -> "This is a vegetable.",
} : "Unknown food item."
# Result: "This is a fruit."
```

This can also be useful for when getting a key from a map that you're not sure exists.

```fpon
let data = {
    "id"               -> 1234,
    "sentiment"        -> "positive",
    "confidence-score" -> 0.96,
    "summary"          -> "The user praised the faster loading speeds and sleek UI overhaul.",
    "is-locked"        -> false,
    "parent-id"        -> null,
} in
"non-existant" ?> data : "default value"
# Result: "default value"
# because "non-existant" is not in data
```

`true` and `false` are also values that can be pattern matched, so we can also get ternary conditional operator out of this.

```fpon
a ?> true -> "a is true" :
b ?> true -> "b is true" :
"neither a nor b is true"
```

Since this is super common, we can drop the `>` to show that it's not being piped into a function like before. The result looks like the ternary conditional operator in other C-like languages.

```fpon
a ? "a is true" :
b ? "b is true" :
"neither a nor b is true"
```
