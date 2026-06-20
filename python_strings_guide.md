# The Complete Python Strings Guide

*From fundamentals to advanced usage, formatting, regular expressions, real-world processing, performance, interviews, and practice exercises.*

## Table of Contents

1. [Introduction to Strings](#1-introduction-to-strings)
2. [Creating Strings](#2-creating-strings)
3. [String Representation](#3-string-representation)
4. [Accessing String Data](#4-accessing-string-data)
5. [String Operations](#5-string-operations)
6. [Built-in Functions for Strings](#6-built-in-functions-for-strings)
7. [String Methods](#7-string-methods)
8. [String Formatting](#8-string-formatting)
9. [Unicode and Internationalization](#9-unicode-and-internationalization)
10. [String Iteration](#10-string-iteration)
11. [String Copying and Immutability](#11-string-copying-and-immutability)
12. [String Searching Techniques](#12-string-searching-techniques)
13. [String Manipulation Techniques](#13-string-manipulation-techniques)
14. [Strings and Regular Expressions](#14-strings-and-regular-expressions)
15. [Real-World String Processing](#15-real-world-string-processing)
16. [Performance Considerations](#16-performance-considerations)
17. [Common Mistakes](#17-common-mistakes)
18. [Best Practices](#18-best-practices)
19. [Python String Interview Questions](#19-python-string-interview-questions)
20. [Practice Exercises](#20-practice-exercises)
21. [Cheat Sheet](#21-cheat-sheet)
22. [Summary](#22-summary)

---

## 1. Introduction to Strings

### 1.1 What is a String

A string is a sequence of Unicode characters used to represent text. In Python, strings are created using the built-in `str` type.

```python
name = "Python"
print(type(name))
```

**Output:**
```
<class 'str'>
```

### 1.2 Characteristics of Strings

* Strings are **ordered** sequences — each character has a fixed position.
* Strings are **immutable** — once created, they cannot be changed in place.
* Strings are **iterable** — you can loop over each character.
* Strings support **indexing and slicing**.
* Strings can hold **any Unicode character**, including letters, digits, symbols, and emojis.

### 1.3 String as a Sequence of Characters

Every string behaves like a sequence (similar to a list of characters), supporting `len()`, indexing, and slicing.

```python
word = "Hello"
print(word[0])     # H
print(len(word))   # 5
for ch in word:
    print(ch, end=" ")
```

**Output:**
```
H
5
H e l l o
```

### 1.4 String Immutability

Once a string object is created, its contents cannot be changed. Any "modification" actually creates a brand-new string object.

```python
s = "cat"
# s[0] = "b"  # TypeError: 'str' object does not support item assignment
s = "b" + s[1:]
print(s)
```

**Output:**
```
bat
```

### 1.5 String Use Cases

* Storing and displaying text (names, messages, UI text)
* Parsing and processing files, logs, and CSV data
* Building dynamic output (reports, emails, templates)
* Validating user input (emails, phone numbers, identifiers)
* Communicating with APIs/web data (JSON, URLs, HTML)
* Working with natural language and internationalized text

---

## 2. Creating Strings

### 2.1 Single Quotes

**Definition:** Strings enclosed in `'...'`.

```python
s = 'Hello, World!'
print(s)
```
**Output:** `Hello, World!`

### 2.2 Double Quotes

**Definition:** Strings enclosed in `"..."`. Functionally identical to single quotes; useful when the string itself contains a single quote.

```python
s = "It's a sunny day"
print(s)
```
**Output:** `It's a sunny day`

### 2.3 Triple Quotes

**Definition:** Strings enclosed in `'''...'''` or `"""..."""`. Allow embedded quotes and multi-line text without escaping.

```python
s = '''She said, "Python is great!"'''
print(s)
```
**Output:** `She said, "Python is great!"`

### 2.4 Multi-line Strings

```python
text = """Line One
Line Two
Line Three"""
print(text)
```
**Output:**
```
Line One
Line Two
Line Three
```

**Note:** Triple-quoted strings are also commonly used as docstrings for modules, classes, and functions.

### 2.5 Empty Strings

```python
empty = ""
print(len(empty))      # 0
print(bool(empty))     # False
```

### 2.6 String Constructors — `str()`

**Syntax:** `str(object='', encoding=..., errors=...)`

**Parameters:**
* `object` – the value to convert to a string.
* `encoding`, `errors` – used only when decoding bytes.

**Return Value:** A new `str` object.

```python
print(str(123))        # '123'
print(str(3.14))       # '3.14'
print(str([1, 2, 3]))  # '[1, 2, 3]'
print(str(None))       # 'None'
print(str(True))       # 'True'
```

**Common Mistakes:** Forgetting that `str(None)` returns the text `"None"`, not an empty string.

**Best Practice:** Use `str()` to safely convert non-string values before concatenation, e.g. `"Age: " + str(age)`.

---

## 3. String Representation

### 3.1 Characters

In Python there is no separate `char` type — a single character is simply a string of length 1.

```python
c = "A"
print(type(c), len(c))
```
**Output:** `<class 'str'> 1`

### 3.2 Unicode

Python 3 strings are sequences of Unicode code points, so they can represent virtually any character from any language.

```python
print("Café", "日本語", "Привет")
```

### 3.3 Escape Sequences

| Escape | Meaning |
|---|---|
| `\\` | Backslash |
| `\'` | Single quote |
| `\"` | Double quote |
| `\n` | Newline |
| `\t` | Tab |
| `\r` | Carriage return |
| `\b` | Backspace |
| `\f` | Form feed |
| `\v` | Vertical tab |
| `\0` | Null character |
| `\a` | Bell/alert |
| `\xhh` | Character with hex value `hh` |
| `\uxxxx` | Unicode character (16-bit) |
| `\Uxxxxxxxx` | Unicode character (32-bit) |
| `\N{NAME}` | Unicode character by name |

```python
print("Tab:\tNewline:\nBackslash:\\")
print("\x41\x42\x43")          # ABC
print("\u00e9")                  # é
print("\N{HEAVY BLACK HEART}")  # ❤
```

### 3.4 Raw Strings

**Definition:** Prefixing a string with `r` disables escape-sequence processing — useful for file paths and regex patterns.

```python
path = r"C:\Users\new_folder"
print(path)
```
**Output:** `C:\Users\new_folder`

**Common Mistake:** A raw string cannot end with an odd number of backslashes (`r"abc\"` is invalid).

### 3.5 String Literals

* **Plain:** `"text"`
* **Raw:** `r"text"`
* **Bytes:** `b"text"`
* **Formatted (f-string):** `f"text {var}"`
* **Raw + formatted:** `rf"text\path {var}"`

```python
name = "Tom"
print(f"Hello, {name}!")
print(b"bytes literal")
```

---

## 4. Accessing String Data

### 4.1 Indexing

**Definition:** Each character in a string has a numeric position (index), starting at 0.

```
 P   y   t   h   o   n
 0   1   2   3   4   5
-6  -5  -4  -3  -2  -1
```

### 4.2 Positive Indexing

```python
s = "Python"
print(s[0])  # P
print(s[3])  # h
```

### 4.3 Negative Indexing

Negative indices count from the end, starting at `-1`.

```python
s = "Python"
print(s[-1])  # n
print(s[-6])  # P
```

**Common Mistake:** `s[len(s)]` or `s[6]` on a 6-character string raises `IndexError`.

### 4.4 Slicing

**Syntax:** `s[start:stop:step]` — `start` inclusive, `stop` exclusive.

```python
s = "Python Programming"
print(s[0:6])    # Python
print(s[7:])     # Programming
print(s[:6])     # Python
```

### 4.5 Slice Steps

```python
s = "abcdefgh"
print(s[::2])    # acegh -> a c e g
print(s[1::2])   # bdfh
```

### 4.6 Reverse Slicing

```python
s = "Python"
print(s[::-1])   # nohtyP
```

**Notes:** Slicing never raises `IndexError` even with out-of-range indices; it simply returns as much as is available (or an empty string).

**Best Practice:** Prefer slicing over manual loops for substring extraction and reversal — it's faster and more readable.


---

## 5. String Operations

### 5.1 Concatenation

**Operator:** `+`

```python
a = "Hello"
b = "World"
print(a + " " + b)
```
**Output:** `Hello World`

Adjacent string literals are also concatenated automatically:
```python
s = "Hello " "World"
print(s)   # Hello World
```

### 5.2 Repetition

**Operator:** `*`

```python
print("ab" * 3)   # ababab
```

### 5.3 Membership Operators

**Operators:** `in`, `not in`

```python
s = "Python Programming"
print("Pro" in s)       # True
print("Java" not in s)  # True
```

### 5.4 Comparison Operators

`==`, `!=`, `<`, `>`, `<=`, `>=` compare strings **lexicographically** based on Unicode code points.

```python
print("apple" == "apple")   # True
print("apple" < "banana")   # True
print("Apple" < "apple")    # True  ('A' = 65, 'a' = 97)
```

### 5.5 Lexicographical Comparison

Comparison happens character by character using Unicode code point values until a difference is found or one string ends.

```python
print("car" < "cat")    # True  (r < t)
print("car" < "carpet") # True  (shorter prefix is "less")
```

**Common Mistake:** Assuming string comparison is case-insensitive — `"Zebra" < "apple"` is `True` because uppercase letters have smaller code points.

**Best Practice:** Use `.lower()` or `.casefold()` before comparing user input when case shouldn't matter.

---

## 6. Built-in Functions for Strings

### `len()`
**Syntax:** `len(s)` → returns the number of characters.
```python
print(len("Python"))   # 6
```

### `str()`
**Syntax:** `str(obj)` → converts `obj` to its string representation.
```python
print(str(42))   # '42'
```

### `ord()`
**Syntax:** `ord(c)` → returns the Unicode code point of a single character.
```python
print(ord("A"))   # 65
```

### `chr()`
**Syntax:** `chr(i)` → returns the character for a Unicode code point (inverse of `ord`).
```python
print(chr(65))    # A
```

### `ascii()`
**Syntax:** `ascii(obj)` → returns a string with non-ASCII characters escaped.
```python
print(ascii("Café"))   # 'Caf\xe9'
```

### `repr()`
**Syntax:** `repr(obj)` → returns the official, unambiguous string representation (useful for debugging).
```python
print(repr("Hi\n"))   # 'Hi\n'
```

### `format()`
**Syntax:** `format(value, format_spec)`
```python
print(format(3.14159, ".2f"))   # 3.14
```

### `sorted()`
**Syntax:** `sorted(iterable)` → returns a sorted list of characters.
```python
print(sorted("python"))   # ['h', 'n', 'o', 'p', 't', 'y']
```

### `min()` / `max()`
```python
print(min("python"))   # h  (smallest code point)
print(max("python"))   # y
```

### `any()` / `all()`
```python
print(any(c.isdigit() for c in "abc123"))   # True
print(all(c.isalpha() for c in "abc123"))   # False
```

### `enumerate()`
**Syntax:** `enumerate(iterable, start=0)` → yields `(index, character)` pairs.
```python
for i, ch in enumerate("abc"):
    print(i, ch)
```
**Output:**
```
0 a
1 b
2 c
```

**Common Mistake:** Using `ord()`/`chr()` on multi-character strings — both require exactly one character.

**Best Practice:** Use `repr()` (not `print`) when debugging strings that may contain hidden whitespace or special characters.

---

## 7. String Methods

> **Format for every method:** Syntax → Parameters → Return Value → Example.

### 7.1 Case Conversion

**`lower()`** — Syntax: `s.lower()` → Returns a lowercase copy.
```python
print("PYTHON".lower())   # python
```

**`upper()`** — Syntax: `s.upper()` → Returns an uppercase copy.
```python
print("python".upper())   # PYTHON
```

**`capitalize()`** — Syntax: `s.capitalize()` → First character uppercase, rest lowercase.
```python
print("hello world".capitalize())   # Hello world
```

**`title()`** — Syntax: `s.title()` → First letter of each word uppercase.
```python
print("hello world".title())   # Hello World
```

**`swapcase()`** — Syntax: `s.swapcase()` → Swaps case of every character.
```python
print("Hello World".swapcase())   # hELLO wORLD
```

**`casefold()`** — Syntax: `s.casefold()` → Aggressive lowercasing for caseless matching (handles special Unicode cases like German `ß`).
```python
print("Straße".casefold())   # strasse
```

**Common Mistake:** Using `.lower()` for caseless comparisons across languages — use `.casefold()` instead.

### 7.2 Alignment

**`center(width, fillchar=' ')`**
```python
print("hi".center(10, "*"))   # ****hi****
```

**`ljust(width, fillchar=' ')`**
```python
print("hi".ljust(6, "-"))   # hi----
```

**`rjust(width, fillchar=' ')`**
```python
print("hi".rjust(6, "-"))   # ----hi
```

**`zfill(width)`** — pads a numeric string with leading zeros.
```python
print("42".zfill(5))   # 00042
print("-42".zfill(5))  # -0042
```

### 7.3 Searching

**`find(sub, start=0, end=len(s))`** → index of first occurrence, or `-1` if not found.
```python
print("hello world".find("o"))    # 4
print("hello world".find("z"))    # -1
```

**`rfind(sub)`** → index of last occurrence, or `-1`.
```python
print("hello world".rfind("o"))   # 7
```

**`index(sub)`** → like `find()`, but raises `ValueError` if not found.
```python
print("hello".index("e"))   # 1
```

**`rindex(sub)`** → like `rfind()`, raises `ValueError` if not found.

**`count(sub, start=0, end=len(s))`** → number of non-overlapping occurrences.
```python
print("banana".count("a"))   # 3
```

**Common Mistake:** Using `index()` on text that might not contain the substring — wrap in `try/except` or use `find()`.

### 7.4 Validation

| Method | Returns `True` if... | Example |
|---|---|---|
| `isalpha()` | all characters are alphabetic | `"abc".isalpha()` → True |
| `isdigit()` | all characters are digits | `"123".isdigit()` → True |
| `isnumeric()` | all characters are numeric (incl. fractions, Unicode numerals) | `"½".isnumeric()` → True |
| `isdecimal()` | all characters are decimal digits | `"123".isdecimal()` → True |
| `isalnum()` | all characters are alphanumeric | `"abc123".isalnum()` → True |
| `islower()` | all cased characters are lowercase | `"abc".islower()` → True |
| `isupper()` | all cased characters are uppercase | `"ABC".isupper()` → True |
| `istitle()` | string follows title-case rules | `"Hello World".istitle()` → True |
| `isspace()` | all characters are whitespace | `"   ".isspace()` → True |
| `isidentifier()` | string is a valid Python identifier | `"my_var".isidentifier()` → True |
| `isprintable()` | all characters are printable | `"abc\n".isprintable()` → False |

```python
print("123".isdigit(), "½".isnumeric(), "123".isdecimal())
# True True True
```

**Common Mistake:** Confusing `isdigit()`, `isdecimal()`, and `isnumeric()` — they overlap for plain ASCII digits but differ for superscripts, fractions, and Unicode numerals.

### 7.5 Modification

**`replace(old, new, count=-1)`**
```python
print("foo bar foo".replace("foo", "baz"))         # baz bar baz
print("foo bar foo".replace("foo", "baz", 1))       # baz bar foo
```

**`strip(chars=None)`** — removes leading/trailing whitespace (or given characters).
```python
print("  hello  ".strip())     # 'hello'
print("xxhelloxx".strip("x"))  # 'hello'
```

**`lstrip(chars=None)`** / **`rstrip(chars=None)`**
```python
print("  hello  ".lstrip())   # 'hello  '
print("  hello  ".rstrip())   # '  hello'
```

**`removeprefix(prefix)`** (Python 3.9+)
```python
print("HelloWorld".removeprefix("Hello"))   # World
```

**`removesuffix(suffix)`** (Python 3.9+)
```python
print("file.txt".removesuffix(".txt"))   # file
```

**Common Mistake:** Using `replace()` to "strip" characters only at the edges — it replaces everywhere, not just the start/end.

### 7.6 Splitting

**`split(sep=None, maxsplit=-1)`**
```python
print("a,b,c".split(","))        # ['a', 'b', 'c']
print("a b   c".split())          # ['a', 'b', 'c']  (splits on any whitespace)
```

**`rsplit(sep=None, maxsplit=-1)`** — splits from the right.
```python
print("a,b,c".rsplit(",", 1))    # ['a,b', 'c']
```

**`splitlines(keepends=False)`**
```python
print("line1\nline2\nline3".splitlines())   # ['line1', 'line2', 'line3']
```

**`partition(sep)`** — splits into a 3-tuple at the first occurrence.
```python
print("key=value".partition("="))   # ('key', '=', 'value')
```

**`rpartition(sep)`** — splits at the last occurrence.
```python
print("a=b=c".rpartition("="))   # ('a=b', '=', 'c')
```

### 7.7 Joining

**`join(iterable)`**
```python
print(",".join(["a", "b", "c"]))   # a,b,c
print(" ".join("abc"))              # a b c
```

**Common Mistake:** Calling `join()` on a list containing non-string elements raises `TypeError` — convert with `str()` first, e.g. `",".join(str(x) for x in [1,2,3])`.

### 7.8 Translation

**`str.maketrans(x, y=None, z=None)`** — builds a translation table.

**`translate(table)`** — applies the table.

```python
table = str.maketrans("abc", "xyz", "0123456789")
print("a1b2c3".translate(table))   # xyz
```

**Best Practice:** Use `translate()` with `maketrans()` for fast bulk character replacement/removal — it's much faster than chained `replace()` calls.

---

## 8. String Formatting

### 8.1 `%` Formatting (legacy, printf-style)

```python
name = "Alice"
age = 30
print("Name: %s, Age: %d" % (name, age))
print("Pi: %.2f" % 3.14159)
```
**Output:**
```
Name: Alice, Age: 30
Pi: 3.14
```

### 8.2 `str.format()`

```python
print("Name: {}, Age: {}".format("Bob", 25))
print("{0} is {1}, {0} likes Python".format("Carl", 22))
print("{name} scored {score}".format(name="Dee", score=95))
```

**Width / Alignment / Precision:**
```python
print("{:<10}|".format("left"))     # left      |
print("{:>10}|".format("right"))    #      right|
print("{:^10}|".format("mid"))      #    mid    |
print("{:.3f}".format(3.14159))     # 3.142
print("{:10.3f}".format(3.14159))   #      3.142
```

### 8.3 f-Strings (Python 3.6+, recommended)

```python
name = "Eve"
score = 95.456
print(f"{name} scored {score:.2f}")          # Eve scored 95.46
print(f"{name!r}")                            # 'Eve'
print(f"{2 + 3}")                             # 5
```

**Width, Alignment, Fill:**
```python
print(f"{'hi':*^10}")     # ***hi****
print(f"{42:>6}")         #     42
print(f"{42:<6}|")        # 42    |
```

**Number Formatting:**
```python
print(f"{1234567:,}")          # 1,234,567
print(f"{0.4567:.1%}")          # 45.7%
print(f"{255:#x}")              # 0xff
print(f"{255:#o}")              # 0o377
print(f"{255:#b}")              # 0b11111111
print(f"{1234.5:+.2f}")         # +1234.50
```

**Currency Formatting:**
```python
amount = 1234.5
print(f"${amount:,.2f}")        # $1,234.50
```

**Date Formatting (with f-strings + `datetime`):**
```python
from datetime import datetime
now = datetime(2026, 6, 20, 14, 30)
print(f"{now:%Y-%m-%d %H:%M}")   # 2026-06-20 14:30
print(f"{now:%B %d, %Y}")         # June 20, 2026
```

**Common Mistakes:**
* Mixing `%` placeholders and `.format()`/f-string syntax in the same string.
* Forgetting the `!r`/`!s` conversion when debugging — use `f"{value=}"` (3.8+) for quick debug printing.
* Forgetting that f-strings cannot contain a backslash directly inside the expression part (pre-3.12).

**Best Practice:** Prefer f-strings for readability and performance in modern Python (3.6+); reserve `.format()` for cases needing reusable templates.

---

## 9. Unicode and Internationalization

### 9.1 Unicode Basics

Unicode assigns every character a unique code point, allowing Python strings to represent virtually any writing system.

```python
print(ord("A"), ord("€"), ord("😀"))
```

### 9.2 UTF-8

UTF-8 is the most common encoding used to convert Unicode strings into bytes for storage/transmission.

```python
text = "café"
encoded = text.encode("utf-8")
print(encoded)                      # b'caf\xc3\xa9'
print(encoded.decode("utf-8"))      # café
```

### 9.3 Unicode Characters & Emojis

```python
print("\N{SMILING FACE WITH SMILING EYES}")   # 😊
print(len("👍"))                                # 1 (single code point)
```

### 9.4 Non-English Text Handling

```python
text = "Bonjour, 世界, Привет"
print(text.upper())
for ch in "café":
    print(ch, ord(ch))
```

**Common Mistake:** Assuming `len()` always matches the visual character count — some emoji and combined characters are made of multiple code points.

**Best Practice:** Always specify `encoding="utf-8"` explicitly when reading/writing text files to avoid platform-dependent defaults.

---

## 10. String Iteration

### 10.1 `for` Loops

```python
for ch in "abc":
    print(ch)
```

### 10.2 `enumerate()`

```python
for i, ch in enumerate("abc"):
    print(i, ch)
```

### 10.3 Manual Iteration with an Index

```python
s = "abc"
i = 0
while i < len(s):
    print(s[i])
    i += 1
```

**Best Practice:** Prefer `for ch in s` or `enumerate(s)` over manual index management — it's clearer and less error-prone.

---

## 11. String Copying and Immutability

### 11.1 Why Strings Are Immutable

Strings cannot be changed after creation. This design enables safe sharing of string objects, predictable behavior as dictionary keys, and simpler reasoning about code.

### 11.2 Effects of Modification Attempts

```python
s = "hello"
try:
    s[0] = "H"
except TypeError as e:
    print("Error:", e)
```
**Output:** `Error: 'str' object does not support item assignment`

Any "change" produces a new string:
```python
s = "hello"
new_s = "H" + s[1:]
print(s, new_s)   # hello Hello
```

### 11.3 Common Misconceptions

* "Strings are passed by reference and can be mutated through a function" — **false**; reassigning inside a function does not affect the caller's variable.
* "`+=` modifies the string in place" — **false**; it creates a new object and rebinds the variable.

```python
def modify(s):
    s += " world"
    return s

text = "hello"
modify(text)
print(text)   # hello (unchanged)
```

**Best Practice:** Treat strings as values, not as mutable containers — use lists (then `join()`) when you need to build up text incrementally.

---

## 12. String Searching Techniques

```python
s = "The quick brown fox"
print("quick" in s)            # True
print(s.find("quick"))          # 4
print(s.index("quick"))         # 4
print(s.startswith("The"))      # True
print(s.endswith("fox"))        # True
```

### Comparison Table

| Technique | Returns | On Not Found | Use When |
|---|---|---|---|
| `in` | `bool` | `False` | Simple membership check |
| `find()` | index or `-1` | `-1` | Need the position, tolerate absence |
| `index()` | index | raises `ValueError` | Need the position, absence is exceptional |
| `startswith()` | `bool` | `False` | Check prefix |
| `endswith()` | `bool` | `False` | Check suffix |

**Best Practice:** Use `in` for plain existence checks; reserve `find()`/`index()` for when the position itself matters.

---

## 13. String Manipulation Techniques

### 13.1 Reverse a String
```python
s = "Python"
print(s[::-1])   # nohtyP
```

### 13.2 Remove Spaces
```python
s = "H e l l o"
print(s.replace(" ", ""))   # Hello
```

### 13.3 Remove Special Characters
```python
import re
s = "Hello, World! #2026"
print(re.sub(r"[^A-Za-z0-9 ]", "", s))   # Hello World 2026
```

### 13.4 Count Words
```python
s = "The quick brown fox jumps"
print(len(s.split()))   # 5
```

### 13.5 Count Characters
```python
from collections import Counter
print(Counter("banana"))   # Counter({'a': 3, 'n': 2, 'b': 1})
```

### 13.6 Palindrome Check
```python
def is_palindrome(s):
    s = s.lower().replace(" ", "")
    return s == s[::-1]

print(is_palindrome("Madam"))         # True
print(is_palindrome("race a car"))    # False
```

### 13.7 Anagram Check
```python
def is_anagram(a, b):
    return sorted(a.lower()) == sorted(b.lower())

print(is_anagram("listen", "silent"))   # True
```

**Best Practice:** For heavy text-cleaning logic, combine `re` with string methods rather than chaining many `replace()` calls.

---

## 14. Strings and Regular Expressions

### 14.1 Introduction to the `re` Module

The `re` module enables pattern-based search, validation, and transformation of strings.

```python
import re
```

### 14.2 `match()` — checks for a match at the **start** of the string

```python
result = re.match(r"\d+", "123abc")
print(result.group())   # 123
```

### 14.3 `search()` — finds the first match **anywhere** in the string

```python
result = re.search(r"\d+", "abc123def")
print(result.group())   # 123
```

### 14.4 `findall()` — returns **all** non-overlapping matches as a list

```python
print(re.findall(r"\d+", "a1 b22 c333"))   # ['1', '22', '333']
```

### 14.5 `finditer()` — returns an iterator of match objects

```python
for m in re.finditer(r"\d+", "a1 b22 c333"):
    print(m.group(), m.span())
```
**Output:**
```
1 (1, 2)
22 (4, 6)
333 (8, 11)
```

### 14.6 `sub()` — substitutes matches with a replacement

```python
print(re.sub(r"\s+", " ", "a   b\tc\n d"))   # a b c d
```

### 14.7 `re.split()` — splits a string using a regex pattern

```python
print(re.split(r"[,;]\s*", "a, b; c,d"))   # ['a', 'b', 'c', 'd']
```

**Common Mistake:** Forgetting that `match()` only anchors at the start — use `search()` or `fullmatch()` for other positions.

**Best Practice:** Pre-compile patterns with `re.compile()` when reused many times, e.g. in a loop.

```python
pattern = re.compile(r"\d+")
print(pattern.findall("a1 b22"))
```

---

## 15. Real-World String Processing

### 15.1 Email Validation

```python
import re
def is_valid_email(email):
    pattern = r"^[\w.+-]+@[\w-]+\.[a-zA-Z]{2,}$"
    return bool(re.match(pattern, email))

print(is_valid_email("user@example.com"))   # True
print(is_valid_email("bad-email"))           # False
```

### 15.2 URL Processing

```python
from urllib.parse import urlparse
url = "https://www.example.com/path?query=python"
parsed = urlparse(url)
print(parsed.scheme, parsed.netloc, parsed.path, parsed.query)
```
**Output:** `https www.example.com /path query=python`

### 15.3 Log Parsing

```python
log = '127.0.0.1 - - [20/Jun/2026:10:00:00] "GET /index.html HTTP/1.1" 200'
import re
match = re.search(r'^(\S+).*\[(.*?)\].*"(\w+) (\S+).*" (\d+)', log)
ip, ts, method, path, status = match.groups()
print(ip, ts, method, path, status)
```

### 15.4 CSV Processing

```python
line = "name,age,city"
fields = line.split(",")
print(fields)   # ['name', 'age', 'city']
```
**Note:** For real CSV files, prefer the built-in `csv` module — manual splitting breaks on quoted commas.

### 15.5 Text Cleaning

```python
raw = "  Hello,   World!!\n"
clean = re.sub(r"\s+", " ", raw).strip()
print(clean)   # Hello, World!!
```

### 15.6 Data Preprocessing (e.g., for NLP)

```python
text = "Python is GREAT!! Visit https://python.org now."
text = text.lower()
text = re.sub(r"http\S+", "", text)
text = re.sub(r"[^a-z\s]", "", text)
tokens = text.split()
print(tokens)
```
**Output:** `['python', 'is', 'great', 'visit', 'now']`

**Best Practice:** Build text-cleaning pipelines as small, named, testable functions rather than one large chained expression.

---

## 16. Performance Considerations

### 16.1 Efficient Concatenation

Building a large string by repeated `+=` inside a loop creates many intermediate string objects.

```python
# Less efficient for many iterations:
result = ""
for word in ["a", "b", "c"] * 1000:
    result += word

# More efficient:
result = "".join(["a", "b", "c"] * 1000)
```

### 16.2 `join()` vs `+`

* `+` / `+=` is fine for a small, fixed number of concatenations.
* `join()` is the preferred approach when combining many pieces (e.g., from a loop or list comprehension), since it builds the final string in a single pass.

```python
words = ["Python", "is", "fun"]
sentence = " ".join(words)
print(sentence)
```

### 16.3 Best Practices

* Use f-strings for formatting — they are evaluated efficiently and read clearly.
* Use `"".join(list_of_strings)` instead of looping with `+=`.
* Avoid unnecessary repeated calls to `len()` inside tight loops — store it in a variable if used repeatedly.
* Use generator expressions with `join()` for memory efficiency on large data: `",".join(str(x) for x in data)`.

---

## 17. Common Mistakes

### 17.1 Index Errors
```python
s = "abc"
# print(s[3])   # IndexError: string index out of range
```
**Fix:** Remember valid indices range from `0` to `len(s)-1` (or `-1` to `-len(s)`).

### 17.2 Wrong Slicing
```python
s = "Python"
print(s[2:10])   # 'thon' — no error, just stops at the end
```
**Mistake:** Expecting an `IndexError` for slices — slicing is forgiving, unlike direct indexing.

### 17.3 Misusing `replace()`
```python
s = "  hello  "
print(s.replace(" ", ""))  # removes ALL spaces, not just edges — use strip() instead
```

### 17.4 Incorrect Comparisons
```python
print("10" > "9")   # False! Lexicographic comparison, not numeric.
print(int("10") > int("9"))   # True
```

### 17.5 Formatting Mistakes
```python
# print("{} {}".format("only one value"))  # IndexError: not enough args
print("{0} {0}".format("ok"))   # 'ok ok' — reusing the same index is fine
```

**Best Practice:** Always test formatting strings with realistic inputs, and validate the number of placeholders matches the number of supplied values.

---

## 18. Best Practices

* **Readability:** Prefer f-strings and named placeholders over cryptic `%`-formatting for new code.
* **Naming:** Use descriptive variable names (`user_name`, not `s1`) when strings carry meaning.
* **Formatting:** Keep format specifiers consistent across a codebase (decide on `.2f` vs `:,.2f` for currency, for example).
* **Validation:** Always validate and sanitize external input (user-submitted text, file content) before using it in further processing.
* **Security Considerations:**
  * Never use string concatenation to build SQL queries — use parameterized queries to avoid SQL injection.
  * Be cautious with `eval()`/`exec()` on string input from untrusted sources.
  * Escape or encode strings appropriately when inserting into HTML, shell commands, or URLs to avoid injection attacks.

```python
# Unsafe:
# query = "SELECT * FROM users WHERE name = '" + user_input + "'"
# Safe (parameterized, using sqlite3 as an example):
# cursor.execute("SELECT * FROM users WHERE name = ?", (user_input,))
```

---

## 19. Python String Interview Questions

### Beginner

**1. What is a string in Python?**
A sequence of Unicode characters used to represent text, created with quotes or `str()`.

**2. Are strings mutable or immutable in Python?**
Immutable — once created, a string's contents cannot be changed in place.

**3. How do you find the length of a string?**
Using the built-in `len()` function: `len("hello")` → `5`.

**4. How do you access the last character of a string?**
Using negative indexing: `s[-1]`.

**5. What is the difference between `'` and `"""` quotes?**
Single/double quotes create single-line strings; triple quotes allow multi-line strings and embedded quotes without escaping.

**6. How do you reverse a string?**
Using slicing: `s[::-1]`.

**7. How do you convert a string to uppercase/lowercase?**
`s.upper()` and `s.lower()`.

**8. What does `s.strip()` do?**
Removes leading and trailing whitespace (or specified characters).

**9. How do you check if a substring exists in a string?**
Using the `in` operator: `"sub" in s`.

**10. How do you concatenate two strings?**
Using `+`: `"a" + "b"`, or `f"{a}{b}"`.

**11. What is the output of `"3" + "4"`?**
`"34"` — string concatenation, not numeric addition.

**12. How do you split a string into a list of words?**
`s.split()` splits on whitespace by default.

**13. How do you join a list of strings into one string?**
`"sep".join(list_of_strings)`.

**14. What does `s.isdigit()` check?**
Whether all characters in the string are digits.

**15. How do you repeat a string multiple times?**
Using `*`: `"ab" * 3` → `"ababab"`.

### Intermediate

**16. What's the difference between `find()` and `index()`?**
`find()` returns `-1` if not found; `index()` raises `ValueError`.

**17. What is the difference between `split()` and `rsplit()`?**
`split()` splits from the left; `rsplit()` splits from the right when `maxsplit` is limited; results otherwise match for unlimited splits.

**18. How does string slicing handle out-of-range indices?**
It does not raise an error — it simply returns whatever is available within bounds.

**19. What's the difference between `str.format()` and f-strings?**
f-strings embed expressions directly in `{}` and are evaluated at runtime where written; `.format()` uses positional/keyword substitution with a separate template.

**20. Why is `"".join(list)` preferred over `+=` in a loop?**
`join()` builds the result in one pass; repeated `+=` can create many intermediate string objects, which is less efficient for large numbers of concatenations.

**21. What is the difference between `isdigit()`, `isdecimal()`, and `isnumeric()`?**
They progressively broaden what counts as numeric: `isdecimal()` is the strictest (decimal digits only), `isdigit()` adds some digit-like characters, and `isnumeric()` is the broadest (also includes fractions and numeral characters).

**22. How do you remove a specific prefix/suffix from a string safely (Python 3.9+)?**
`s.removeprefix("pre")` and `s.removesuffix("suf")` — they only remove the exact prefix/suffix if present, unlike `replace()`, which matches anywhere.

**23. What does `str.maketrans()` do?**
Builds a character-to-character (or character-to-None) mapping table for use with `translate()`.

**24. How do you check if a string is a valid identifier?**
`s.isidentifier()`.

**25. What is string interning, conceptually (without internals)?**
Python may reuse the same object for identical small/simple string literals for efficiency; this is an implementation detail and shouldn't be relied upon with `is` for equality checks — use `==`.

**26. How would you count the frequency of each character in a string?**
Using `collections.Counter(s)`.

**27. How do you check if two strings are anagrams?**
Compare their sorted character lists: `sorted(a) == sorted(b)`.

**28. What does `partition()` return if the separator isn't found?**
A 3-tuple: `(original_string, '', '')`.

**29. How can you format a number with thousands separators?**
`f"{number:,}"`.

**30. What's the difference between `lower()` and `casefold()`?**
`casefold()` is more aggressive and handles special Unicode case-folding rules (e.g., German `ß` → `ss`) that `lower()` doesn't.

**31. How do you safely format strings containing user-supplied values to avoid errors?**
Validate the number/type of values match the placeholders, and prefer f-strings or `.format()` (avoid `%` with untrusted format strings).

**32. How do you check if a string starts or ends with one of several options?**
Pass a tuple: `s.startswith(("Mr.", "Ms.", "Dr."))`.

**33. What happens when you multiply a string by a negative number or zero?**
The result is an empty string.

**34. How do you remove all digits from a string?**
`re.sub(r"\d", "", s)` or `"".join(c for c in s if not c.isdigit())`.

**35. How would you validate an email address using regex?**
Use a pattern like `r"^[\w.+-]+@[\w-]+\.[a-zA-Z]{2,}$"` with `re.match()`.

### Advanced

**36. Why can't you modify a string in place, and what's the practical implication?**
Strings are immutable by design; the practical implication is that any "modification" creates a new object, so building large strings incrementally should use `join()` or other mutable structures rather than repeated reassignment.

**37. How does Python compare strings of different lengths lexicographically?**
It compares character by character; if one string is a strict prefix of the other, the shorter one is considered smaller.

**38. What's the difference between `re.match()`, `re.search()`, and `re.fullmatch()`?**
`match()` anchors at the start, `search()` looks anywhere, and `fullmatch()` requires the entire string to match the pattern.

**39. How would you efficiently process a very large text file line by line?**
Iterate over the file object directly (`for line in file:`) rather than loading the whole file into memory, and process/clean each line incrementally.

**40. What's the difference between encoding and decoding a string?**
Encoding converts a `str` (Unicode text) into `bytes` using a scheme like UTF-8; decoding converts `bytes` back into a `str`.

**41. How do you handle a `UnicodeDecodeError` when reading a file?**
Specify the correct encoding explicitly, or pass `errors="replace"`/`errors="ignore"` to `decode()`/`open()` to control how invalid bytes are handled.

**42. What's the benefit of pre-compiling a regex pattern with `re.compile()`?**
It avoids recompiling the same pattern on every call when the pattern is reused many times, such as inside a loop.

**43. How would you detect overlapping vs non-overlapping matches with regex?**
`re.findall()`/`finditer()` find non-overlapping matches by default; overlapping matches require manual handling (e.g., advancing the search position by one and re-searching).

**44. How can you make string comparisons case- and locale-insensitive?**
Normalize both strings with `casefold()`, and consider Unicode normalization (`unicodedata.normalize()`) for accented characters.

**45. What's the difference between `str` and `bytes` in Python 3?**
`str` represents Unicode text (sequence of code points); `bytes` represents raw binary data (sequence of integers 0–255). Conversion between them requires explicit encoding/decoding.

**46. How would you safely build a SQL query using string data?**
Use parameterized queries / prepared statements rather than concatenating strings directly, to prevent SQL injection.

**47. How do you remove duplicate whitespace while preserving single spaces?**
`re.sub(r"\s+", " ", s).strip()`.

**48. How would you implement a case-insensitive "contains" check across many languages correctly?**
Use `casefold()` on both strings before checking membership, since it normalizes case more thoroughly than `lower()` for non-ASCII text.

**49. What's a potential pitfall of using `split()` without arguments versus `split(" ")`?**
`split()` with no arguments splits on any whitespace and collapses consecutive separators, while `split(" ")` splits strictly on single spaces and can produce empty strings between consecutive spaces.

**50. How would you design a text-cleaning pipeline for NLP preprocessing?**
Break it into small composable steps — lowercase, remove URLs/markup, remove punctuation/non-letters, normalize whitespace, tokenize — each implemented as a small, testable function applied in sequence.

**51. Why might two visually identical strings compare as unequal?**
They may use different Unicode representations of the same visual character (e.g., a precomposed accented character vs. a base character plus a combining accent); normalizing with `unicodedata.normalize()` resolves this.

**52. How do you efficiently check membership of a string against a large set of possible values?**
Convert the set of values to a `set` and use `in` — set membership checks are much faster than checking against a list or chaining many comparisons.

---

## 20. Practice Exercises

### Easy (1–20)

1. Print your name in uppercase. → `"PYTHON".upper()` → `PYTHON`
2. Find the length of `"Hello, World!"`. → `13`
3. Reverse the string `"Python"`. → `nohtyP`
4. Check if `"radar"` is a palindrome. → `True`
5. Concatenate `"Hello"` and `"World"` with a space. → `Hello World`
6. Repeat `"ab"` three times. → `ababab`
7. Convert `"hello world"` to title case. → `Hello World`
8. Count occurrences of `"a"` in `"banana"`. → `3`
9. Check if `"42"` is numeric. → `True`
10. Remove leading/trailing spaces from `"  hi  "`. → `hi`
11. Replace `"cat"` with `"dog"` in `"I have a cat"`. → `I have a dog`
12. Split `"a-b-c"` by `"-"`. → `['a', 'b', 'c']`
13. Join `['x','y','z']` with `"-"`. → `x-y-z`
14. Check if `"abc123"` is alphanumeric. → `True`
15. Get the first character of `"Python"`. → `P`
16. Get the last 3 characters of `"Programming"`. → `ing`
17. Check if `"Hello"` starts with `"He"`. → `True`
18. Check if `"Hello"` ends with `"lo"`. → `True`
19. Convert `42` to a string. → `"42"`
20. Find the index of `"o"` in `"Hello"`. → `4`

### Medium (1–20)

1. Count the number of vowels in `"Beautiful Day"`. → `6`
2. Check if two strings are anagrams: `"listen"`, `"silent"`. → `True`
3. Remove all punctuation from `"Hi! How are you?"`. → `Hi How are you`
4. Count words in `"The quick brown fox"`. → `4`
5. Capitalize the first letter of every word in `"hello world foo"`. → `Hello World Foo`
6. Check if a string contains only digits: `"007"`. → `True`
7. Find all numbers in `"abc123def456"` using regex. → `['123', '456']`
8. Replace multiple spaces with a single space in `"a   b    c"`. → `a b c`
9. Check if `"Hello"` and `"hello"` are equal ignoring case. → `True`
10. Extract the domain from `"user@example.com"`. → `example.com`
11. Convert `"snake_case_name"` to `"camelCaseName"`. → `camelCaseName`
12. Find the most frequent character in `"aabbbcc"`. → `b`
13. Check if a string is a valid Python identifier: `"my_var1"`. → `True`
14. Remove duplicate characters from `"aabbccdd"` while preserving order. → `abcd`
15. Pad `"7"` to width 3 with leading zeros. → `007`
16. Center `"hi"` in a field of width 10 with `*`. → `****hi****`
17. Split `"key:value:extra"` into at most 2 parts. → `['key', 'value:extra']`
18. Find the longest word in `"I love programming in Python"`. → `programming`
19. Check whether `"Madam, I'm Adam"` is a palindrome ignoring spaces/punctuation/case. → `True`
20. Convert a list of numbers `[1,2,3]` into a comma-separated string. → `1,2,3`

### Hard (1–20)

1. Validate an email address with regex: `"test@example.com"`. → `True`
2. Parse a log line and extract IP, timestamp, and status code.
3. Implement a basic word-frequency counter for a paragraph of text.
4. Detect all overlapping occurrences of `"aa"` in `"aaaa"`. → `3`
5. Implement a function that compresses repeated characters: `"aaabbc"` → `"a3b2c1"`.
6. Implement basic run-length decoding: `"a3b2c1"` → `"aaabbc"`.
7. Write a function to check balanced parentheses in a string.
8. Implement a simple tokenizer that splits text into words and punctuation separately.
9. Write a function to mask all but the last 4 digits of a credit-card-like string: `"1234567812345678"` → `"************5678"`.
10. Implement Caesar cipher encryption/decryption for lowercase letters.
11. Write a function to find the longest common prefix among a list of strings.
12. Implement a simple template engine that replaces `{{name}}` placeholders with provided values.
13. Write a function that converts a number to words (e.g., `123` → `"one hundred twenty three"`).
14. Implement a basic CSV-line parser that respects quoted commas.
15. Write a function to detect if a string is a valid IPv4 address.
16. Implement a sliding-window function to find the longest substring without repeating characters.
17. Write a function that normalizes Unicode accented text to plain ASCII.
18. Implement a simple Markdown-to-plain-text stripper (removes `*`, `_`, `#`, etc.).
19. Write a function to pretty-print a multi-line block of text with consistent indentation.
20. Implement a basic URL slug generator: `"Hello World! 2026"` → `"hello-world-2026"`.

---

## 21. Cheat Sheet

### Creation
```python
'single'  "double"  '''triple'''  """triple"""
str(123)            # '123'
```

### Indexing & Slicing
```python
s[0]        # first char
s[-1]       # last char
s[a:b]      # substring [a, b)
s[a:b:c]    # step c
s[::-1]     # reversed
```

### Operators
```python
a + b     # concatenation
a * n     # repetition
x in a    # membership
a == b    # equality
a < b     # lexicographic comparison
```

### Common Built-ins
```python
len(s) ord(c) chr(i) repr(s) sorted(s) min(s) max(s) enumerate(s)
```

### Common Methods (by category)
| Category | Methods |
|---|---|
| Case | `lower upper capitalize title swapcase casefold` |
| Align/Pad | `center ljust rjust zfill` |
| Search | `find rfind index rindex count startswith endswith` |
| Validate | `isalpha isdigit isnumeric isdecimal isalnum islower isupper istitle isspace isidentifier isprintable` |
| Modify | `replace strip lstrip rstrip removeprefix removesuffix` |
| Split | `split rsplit splitlines partition rpartition` |
| Join | `join` |
| Translate | `maketrans translate` |

### Formatting
```python
f"{value:.2f}"     # 2 decimal places
f"{value:>10}"     # right-align width 10
f"{value:,}"       # thousands separator
f"{value:.1%}"     # percentage
f"{value:#x}"      # hex with prefix
```

### Regex Quick Reference
```python
import re
re.match(pattern, s)      # match at start
re.search(pattern, s)     # match anywhere
re.findall(pattern, s)    # all matches as list
re.finditer(pattern, s)   # all matches as iterator
re.sub(pattern, repl, s)  # substitute
re.split(pattern, s)      # split by pattern
```

### Common Patterns
```python
"".join(parts)                       # efficient concatenation
s.strip().lower()                    # normalize input
re.sub(r"\s+", " ", s).strip()       # collapse whitespace
sorted(a) == sorted(b)               # anagram check
s == s[::-1]                         # palindrome check
```

---

## 22. Summary

This guide covered Python strings end-to-end:

* **Fundamentals:** what strings are, how they're created (single/double/triple quotes, `str()`), and why they're immutable.
* **Representation:** characters, Unicode, escape sequences, and raw strings.
* **Access:** indexing, negative indexing, and slicing with steps.
* **Operations:** concatenation, repetition, membership, and lexicographic comparison.
* **Built-in functions:** `len()`, `ord()`, `chr()`, `repr()`, `sorted()`, and more.
* **String methods:** case conversion, alignment, searching, validation, modification, splitting, joining, and translation.
* **Formatting:** `%`-formatting, `.format()`, and f-strings, including width, precision, and number/currency/date formatting.
* **Unicode and internationalization:** UTF-8 encoding, emojis, and non-English text.
* **Iteration, immutability, and searching techniques**, with practical comparisons.
* **Manipulation techniques:** reversing, cleaning, counting, palindrome/anagram checks.
* **Regular expressions:** `match`, `search`, `findall`, `finditer`, `sub`, and `split`.
* **Real-world applications:** email validation, URL parsing, log parsing, CSV handling, and text preprocessing.
* **Performance:** efficient concatenation with `join()`, and general best practices.
* **Common mistakes and best practices** to avoid bugs and write clean, secure string-handling code.
* **Interview preparation:** 52 questions spanning beginner to advanced topics.
* **Practice exercises:** 60 exercises (20 easy, 20 medium, 20 hard) to reinforce every concept.
* **Cheat sheet:** a fast reference for daily use.

Mastering strings is foundational to almost every Python program — from simple scripts to data pipelines, web applications, and natural language processing. Revisit this guide as a reference whenever you need a refresher on syntax, methods, or best practices.
