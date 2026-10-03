# Strings in FPON

## Standard

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

## Interpolation

Curly braces within a string literal is used for **string interpolation.** Whatever value is between the curly braces will get implicitly converted to a string, so explicit conversion isn't necessary. A literal curly brace can be escaped in a string with a backslash like this `\{` which disables string interpolation. 

```fpon
let name = "Bob" in
let score = 95 in
"{name} scored {score} points"
# Result: "Bob scored 95 points."
```

## Word Strings

Maps in FPON are called the same way as functions instead of using dot notation (`.`) like in other languages. Keys can be any type, but the most common type to use is string. Since the period is free for other purposes in FPON, it's used to create **"word string" literals**. These strings only contain valid word characters (`A-Za-z0-9_`) and stop at the first non-word character. This makes them raw strings since other symbols like backslashes `\` for escaping or curly braces `{}` for interpolation can't be used. A word string must not be empty or else it's a syntax error. Word strings are equivalent to standard strings that contain the same characters.

```fpon
.word == "word"
```

This let's you use the familiar dot notation with maps. The following two examples are equivalent.

```fpon

```

```fpon

```

## Doc Strings

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

The main use case for this type of string is for **embedding documents** as strings within a configuration file. Here are some examples of this:

```fpon
{
  "html_template" ->
    \\<!DOCTYPE html>
    \\<html>
    \\  <body>
    \\    <h1>Hello From FPON</h1>
    \\  </body>
    \\</html>
    , # Commas need to be on the next line after the last line of the raw string.
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
