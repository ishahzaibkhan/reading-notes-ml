# Regular Expressions (Regex) in Python - Complete Guide

## Introduction to Regular Expressions

Regular expressions (regex) provide a powerful method for searching and matching text patterns in large text files, such as emails, domain names, and other structured data.

## The `re` Module

Python's built-in `re` module must be imported to work with regular expressions:
```python
import re
```

### Compiling Patterns

Patterns can be compiled using `re.compile()` for reusability, improved readability, and flexibility:
```python
pattern = re.compile(r"ABC")
```

**Raw Strings (`r` prefix):** The `r` prefix indicates a raw string, ensuring backslashes are treated as literal characters (e.g., `\t` prints `\t` instead of a tab).

**Case Sensitivity:** Regex is case-sensitive by default (e.g., "ABC" will not match "abc").

---

## 1. Methods to Search for Matches

### `finditer()`
Returns an iterator of match objects. Each match object contains details like span (start and end positions) and the matched string.
```python
matches = pattern.finditer(test_string)
for match in matches:
    print(match.span(), match.group())
```

### `findall()`
Returns a list of all non-overlapping matches as strings.

### `match()`
Determines if the regex matches at the **beginning** of the string. Returns a match object only if the pattern is found at the very start; otherwise, returns `None`.

### `search()`
Scans through the entire string and returns a match object for the **first occurrence** where the regex matches; otherwise, returns `None`.

**Direct Use:** Methods like `re.finditer()` can be used directly on the `re` module without explicit compilation, though compilation is preferred for readability and flexibility.

---

## 2. Methods on Match Objects

Once a match object is obtained, several methods extract information:

### `span()`
Returns a tuple containing the start and end indices of the match.

### `start()`
Returns the start index of the match.

### `end()`
Returns the end index of the match.

### `group()`
- `match.group()` or `match.group(0)`: Returns the entire matched string
- `match.group(n)` (where n > 0): Returns the string corresponding to a specific capturing group

---

## 3. Meta Characters

Meta characters are special characters with specific meanings in regex:

| Character | Meaning |
|-----------|---------|
| `.` (Dot) | Matches any character except a newline. To match a literal dot, escape it: `\.` |
| `^` (Caret) | Matches the beginning of the string |
| `$` (Dollar Sign) | Matches the end of the string |
| `*` (Asterisk) | Zero or more occurrences (quantifier) |
| `+` (Plus) | One or more occurrences (quantifier) |
| `?` (Question Mark) | Zero or one occurrence - makes element optional (quantifier) |
| `{}` (Curly Braces) | Exact number or range of occurrences (quantifier) |
| `[]` (Square Brackets) | Defines a set of characters to match |
| `|` (Pipe) | OR condition - either/or |
| `()` (Parentheses) | Grouping and capturing |
| `\` (Backslash) | Escapes meta characters to match them literally, or introduces special sequences |

---

## 4. Special Sequences

Special sequences start with a backslash and provide shortcuts for common character sets:

| Sequence | Matches |
|----------|---------|
| `\d` | Any digit (0-9) |
| `\D` | Any non-digit character |
| `\s` | Any whitespace character (space, tab, newline) |
| `\S` | Any non-whitespace character |
| `\w` | Any word character (alphanumeric a-z, A-Z, 0-9, and underscore `_`) |
| `\W` | Any non-word character (non-alphanumeric) |
| `\b` | Word boundary (position between word and non-word character, or at beginning/end of string) |
| `\B` | Non-word boundary |

---

## 5. Sets

Sets are defined using square brackets `[]` and match any single character within the set:

### Individual Characters
`[aLo]` matches 'a', 'L', or 'o'

### Ranges
- `[a-z]`: Any lowercase letter
- `[A-Z]`: Any uppercase letter
- `[0-9]`: Any digit (same as `\d`)

### Combining Ranges
`[a-zA-Z0-9]` matches any alphanumeric character

### Literal Dash
If `-` is at the beginning or end of the set, or escaped, it matches a literal dash: `[a-z-]`

---

## 6. Quantifiers

Quantifiers specify the number of occurrences of the preceding character or group:

| Quantifier | Meaning |
|------------|---------|
| `*` | Zero or more occurrences |
| `+` | One or more occurrences |
| `?` | Zero or one occurrence (optional) |
| `{n}` | Exactly n occurrences |
| `{min,}` | At least min occurrences |
| `{min,max}` | Between min and max occurrences (inclusive) |

### Practical Example: Date Extraction

To extract dates in formats like "YYYY-MM-DD" or "YYYY/MM/DD":
- `\d{4}`: Four digits for year
- `\d{2}`: Two digits for month/day
- `[-/]`: Set for separators (dash or slash)
- `0[5-7]`: Specific month ranges (e.g., May, June, July)

---

## 7. Conditions

### `|` (Pipe/OR)
Allows specifying alternative patterns. Used within parentheses to group alternatives.

**Example:** `(Mr\.|Mrs\.|Miss)` matches "Mr.", "Mrs.", or "Miss" with optional dots after "Mr".

---

## 8. Grouping

### `()` (Parentheses)
Creates capturing groups, allowing parts of the match to be extracted separately.

### Accessing Groups
- `match.group(0)`: The entire match
- `match.group(1)`: The first capturing group
- `match.group(2)`: The second capturing group, etc.

**Example - Email Extraction:**
Build a complex regex for email addresses and use groups to separate username, domain, and top-level domain.

---

## 9. Modification

Regular expressions can modify strings:

### `split()`
Splits a string into a list of substrings wherever the regex pattern matches. The matching pattern itself is not included in the resulting list.

### `sub()`
Finds all substrings where the regex matches and replaces them with a specified replacement string.

**Example:** Replacing "world" with "planet"

### Backreferences
In the replacement string, `\1`, `\2`, etc., reference the content of capturing groups from the original match, enabling dynamic replacements based on matched content.

**Example - URL Modification:**
Use `sub()` with backreferences to remove the protocol (http/https) and www. from URLs, leaving only the domain name and its ending.

---

## 10. Compilation Flags

Flags modify the behavior of the regex engine during compilation. They are passed as arguments to `re.compile()`:
```python
pattern = re.compile(r"pattern", re.IGNORECASE)
```

### Common Flags

| Flag | Description |
|------|-------------|
| `re.IGNORECASE` (or `re.I`) | Makes regex matching case-insensitive |
| `re.ASCII` | Makes `\w`, `\W`, `\b`, `\B`, `\d`, `\D`, `\s`, `\S` perform ASCII-only matching |
| `re.DOTALL` | Makes `.` match any character including newline |
| `re.LOCALE` | Makes `\w`, `\W`, `\b`, `\B` dependent on current locale |
| `re.MULTILINE` | Makes `^` and `$` match at line boundaries |
| `re.VERBOSE` | Allows writing more readable regex by ignoring whitespace and comments |

**Note:** Check the official Python documentation for detailed explanations of all flags.

---

## Summary

This guide covers the complete functionality of Python's `re` module for regular expressions:
- Compiling and using patterns
- Search methods and match object manipulation
- Meta characters and special sequences
- Sets and quantifiers for flexible matching
- Conditions and grouping for complex patterns
- String modification with split and substitution
- Compilation flags for customized behavior

With these tools, you can understand and create any regex expression for text pattern matching and manipulation in Python.

[Regex Crash Course (Python)](https://www.youtube.com/watch?v=AEE9ecgLgdQ)
