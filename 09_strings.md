# 9. Strings — Complete Reference

Python `str` objects are **immutable sequences of Unicode code points**. Every character you see in a Python string — a letter, an emoji, a Chinese character — is stored internally as a *code point* (an integer), not as raw bytes. Bytes only enter the picture when you **encode** a string (turn it into bytes for storage/network) or **decode** bytes (turn them back into a string).

```
"hello" → sequence of 5 code points → [104, 101, 108, 108, 111]
```

---

## Table of Contents

1. [Unicode](#1-unicode)
2. [UTF-8](#2-utf-8)
3. [Other Common Encodings](#3-other-common-encodings)
4. [Encoding (str → bytes)](#4-encoding-str--bytes)
5. [Decoding (bytes → str)](#5-decoding-bytes--str)
6. [String Internals (CPython, PEP 393)](#6-string-internals-cpython-pep-393)
7. [String Interning](#7-string-interning)
8. [String Literals & Escape Sequences](#8-string-literals--escape-sequences)
9. [Every String Method](#9-every-string-method)
10. [Regular Expressions](#10-regular-expressions)

---

## 1. Unicode

### 1.1 The Problem Unicode Solves

Before Unicode, every region had its own character encoding (ASCII for English, Latin-1 for Western Europe, Shift-JIS for Japanese, etc.). The same byte value meant a different character depending on which encoding you assumed. There was no single scheme that could represent text from every language at once.

**Unicode** is a single, universal character set that assigns every character in (almost) every human writing system — plus symbols, emoji, and control characters — a unique number called a **code point**.

### 1.2 Code Points

A code point is written as `U+XXXX` in hexadecimal, and ranges from `U+0000` to `U+10FFFF` (about 1.1 million possible values, of which roughly 149,000 are currently assigned, as of Unicode 15/16).

| Character | Code Point | Name |
|---|---|---|
| `A` | U+0041 | LATIN CAPITAL LETTER A |
| `a` | U+0061 | LATIN SMALL LETTER A |
| `é` | U+00E9 | LATIN SMALL LETTER E WITH ACUTE |
| `好` | U+597D | CJK UNIFIED IDEOGRAPH-597D |
| `😀` | U+1F600 | GRINNING FACE |

In Python, `ord()` converts a character to its code point integer, and `chr()` does the reverse:

```python
>>> ord('A')
65
>>> hex(ord('A'))
'0x41'
>>> chr(0x1F600)
'😀'
>>> chr(20).encode  # control characters are code points too
```

### 1.3 The Unicode Plane Structure

Code points are grouped into 17 **planes** of 65,536 (0x10000) code points each:

```
Plane 0   U+0000  – U+FFFF    Basic Multilingual Plane (BMP)
                                → almost all living scripts, CJK, symbols
Plane 1   U+10000 – U+1FFFF   Supplementary Multilingual Plane
                                → historic scripts, emoji, musical notation
Plane 2   U+20000 – U+2FFFF   Supplementary Ideographic Plane (rare CJK)
Planes 3–13                    mostly unassigned
Plane 14  U+E0000 – U+EFFFF   Supplementary Special-purpose Plane
Planes 15–16                   Private Use Areas
```

Most text you'll ever process (Latin, Cyrillic, Greek, Arabic, Hebrew, CJK, Devanagari) lives in the **BMP**. Emoji and a few historic scripts live above it — this matters later when we talk about internal storage width.

### 1.4 Unicode ≠ an Encoding

This is the single most important distinction in this whole topic:

| Concept | What it is |
|---|---|
| **Unicode** | A *mapping* from characters → code points (numbers). It says nothing about bytes. |
| **Encoding** (UTF-8, UTF-16, UTF-32, ASCII, Latin-1...) | A *rule for turning code points into bytes* (and back). |

A Python `str` is a sequence of Unicode code points. It only becomes bytes — and therefore only needs an encoding — when you write it to a file, send it over a socket, or otherwise serialize it.

```python
>>> s = "café"
>>> type(s)
<class 'str'>
>>> len(s)          # 4 code points, regardless of how many bytes it would take
4
```

### 1.5 Normalization (brief)

Some characters can be represented in more than one way — e.g. `é` can be a single code point `U+00E9` **or** `e` (`U+0065`) followed by a combining acute accent `U+0301`. These look identical but compare as different strings. The `unicodedata` module's `normalize()` function reconciles this:

```python
import unicodedata
a = "café"                              # é as one code point
b = "cafe\u0301"                        # e + combining accent
a == b                                  # False!
unicodedata.normalize("NFC", b) == a    # True — NFC composes to single code points
```

---

## 2. UTF-8

UTF-8 is the encoding that turns Unicode code points into bytes, and it is the **default encoding in Python 3** (for source files, `str.encode()`, and most I/O).

### 2.1 Why UTF-8 Won

- **Backward compatible with ASCII** — any valid ASCII file is already valid UTF-8.
- **Variable width** — common characters (English letters) take 1 byte; rarer ones take more. This keeps Western text compact.
- **Self-synchronizing** — you can tell where a character starts just by looking at one byte, and there's no byte-order (endianness) ambiguity like UTF-16/32 have.
- **No embedded NUL bytes** for non-NUL characters — plays nicely with C-style strings.

### 2.2 The Byte Pattern

UTF-8 uses 1 to 4 bytes per code point:

| Code point range | Bytes used | Byte pattern |
|---|---|---|
| U+0000 – U+007F | 1 byte | `0xxxxxxx` |
| U+0080 – U+07FF | 2 bytes | `110xxxxx 10xxxxxx` |
| U+0800 – U+FFFF | 3 bytes | `1110xxxx 10xxxxxx 10xxxxxx` |
| U+10000 – U+10FFFF | 4 bytes | `11110xxx 10xxxxxx 10xxxxxx 10xxxxxx` |

- A leading bit of `0` → this byte *is* an entire ASCII character.
- A leading `11...` → this byte *starts* a multi-byte sequence; the number of leading 1s tells you how many bytes follow.
- A leading `10` → this is a **continuation byte**, part of a sequence started earlier.

### 2.3 Worked Example

```python
>>> "A".encode("utf-8")
b'A'                  # 1 byte: 0x41
>>> "é".encode("utf-8")
b'\xc3\xa9'            # 2 bytes: 11000011 10101001
>>> "好".encode("utf-8")
b'\xe5\xa5\xbd'        # 3 bytes
>>> "😀".encode("utf-8")
b'\xf0\x9f\x98\x80'    # 4 bytes
```

```python
>>> s = "café"
>>> len(s)                 # 4 code points
4
>>> len(s.encode("utf-8")) # but 5 bytes! (é takes 2 bytes)
5
```

This is the key trap for beginners: **`len()` on a `str` counts characters (code points); `len()` on `bytes` counts bytes.** They are only equal when the string is pure ASCII.

### 2.4 UTF-8 vs UTF-16 vs UTF-32

| Encoding | Width | Notes |
|---|---|---|
| **UTF-8** | 1–4 bytes (variable) | ASCII-compatible, no byte-order issues, dominant on the web/disk |
| **UTF-16** | 2 or 4 bytes (variable) | Used internally by Windows, Java, JavaScript strings; needs a Byte Order Mark (BOM) or explicit endianness (`utf-16-le`/`utf-16-be`); characters above the BMP need **surrogate pairs** (two 16-bit units) |
| **UTF-32** | always 4 bytes | Simple (fixed width = direct code point), but wastes space — every character costs 4 bytes even `'A'` |

```python
>>> "A😀".encode("utf-16")
b'\xff\xfeA\x00=\xd8\x00\xde'   # BOM + surrogate pair for the emoji
>>> "A😀".encode("utf-32")
b'\xff\xfe\x00\x00A\x00\x00\x00\x00\xf6\x01\x00'  # fixed 4 bytes each
```

---

## 3. Other Common Encodings

| Encoding | Range covered | Notes |
|---|---|---|
| `ascii` | U+0000–U+007F only | 1 byte each; raises `UnicodeEncodeError` for anything outside this range |
| `latin-1` / `iso-8859-1` | U+0000–U+00FF only | 1 byte each; every byte 0–255 maps directly — **never raises an error**, which makes it a popular (if lossy) fallback |
| `cp1252` (Windows-1252) | superset of Latin-1 with some symbols | Common in old Windows files |
| `utf-8-sig` | same as UTF-8 | Strips/writes a UTF-8 BOM, useful for files Excel expects to have one |

```python
>>> "ñ".encode("ascii")
Traceback (most recent call last):
UnicodeEncodeError: 'ascii' codec can't encode character '\xf1' in position 0
>>> "ñ".encode("latin-1")
b'\xf1'
```

---

## 4. Encoding (str → bytes)

`str.encode(encoding="utf-8", errors="strict")` converts a string of code points into a `bytes` object.

### 4.1 Signature

```python
str.encode(encoding='utf-8', errors='strict') -> bytes
```

### 4.2 The `errors` Parameter — What Happens on a Bad Character

| Mode | Behavior |
|---|---|
| `"strict"` (default) | Raise `UnicodeEncodeError` |
| `"ignore"` | Drop the offending character silently |
| `"replace"` | Replace with `?` |
| `"xmlcharrefreplace"` | Replace with an XML numeric reference, e.g. `&#8364;` |
| `"backslashreplace"` | Replace with a Python escape, e.g. `\u20ac` |
| `"namereplace"` | Replace with `\N{NAME}` form |

```python
>>> "€100".encode("ascii", errors="strict")
UnicodeEncodeError: 'ascii' codec can't encode character '\u20ac' ...

>>> "€100".encode("ascii", errors="ignore")
b'100'
>>> "€100".encode("ascii", errors="replace")
b'?100'
>>> "€100".encode("ascii", errors="xmlcharrefreplace")
b'&#8364;100'
>>> "€100".encode("ascii", errors="backslashreplace")
b'\\u20ac100'
>>> "€100".encode("ascii", errors="namereplace")
b'\\N{EURO SIGN}100'
```

---

## 5. Decoding (bytes → str)

`bytes.decode(encoding="utf-8", errors="strict")` does the reverse — turns raw bytes back into a `str`. **You must know (or correctly guess) the original encoding**, because bytes carry no inherent information about what encoding produced them.

```python
str_obj = bytes_obj.decode(encoding='utf-8', errors='strict')
```

```python
>>> data = b'\xc3\xa9'
>>> data.decode("utf-8")
'é'
>>> data.decode("latin-1")     # same bytes, WRONG encoding assumed
'Ã©'                            # mojibake! classic UTF-8-decoded-as-Latin-1 bug
```

`errors` accepts the same modes as encoding, plus one decode-only mode:

| Mode | Behavior on decode |
|---|---|
| `"strict"` | Raise `UnicodeDecodeError` |
| `"ignore"` | Drop the bad bytes |
| `"replace"` | Insert the replacement character `�` (U+FFFD) |
| `"surrogateescape"` | Map unrepresentable bytes to private-use surrogate code points so they can round-trip back to identical bytes later — used heavily by `os` module functions for filenames |

```python
>>> b'\xff\xfeA'.decode("utf-8", errors="strict")
UnicodeDecodeError: 'utf-8' codec can't decode byte 0xff in position 0: invalid start byte
>>> b'\xff\xfeA'.decode("utf-8", errors="replace")
'��A'
```

### 5.1 Mental Model: the Encode/Decode Round-Trip

```
str  ──encode(enc)──▶  bytes  ──decode(enc)──▶  str
"café" ──"utf-8"──▶ b'caf\xc3\xa9' ──"utf-8"──▶ "café"
```

If you encode with one codec and decode with a different one, you get **mojibake** (garbled text) — this is the #1 source of "weird characters" bugs in real-world code (CSV files from Excel, web scraping, database imports).

---

## 6. String Internals (CPython, PEP 393)

This section explains *how CPython actually stores a `str` object in memory* — useful for understanding performance, `sys.getsizeof()` numbers, and why certain operations are fast or slow.

### 6.1 The Old Way (pre-3.3) vs PEP 393

Before Python 3.3, every `str` object used a *fixed* internal width — either "narrow" builds (2 bytes/char, breaking on wide characters) or "wide" builds (4 bytes/char, always). Both wasted memory for ASCII-heavy text.

**PEP 393 ("Flexible String Representation")**, implemented since Python 3.3, fixed this: CPython picks the **narrowest storage width that can hold every character actually in that specific string**, decided once at creation time.

| Internal Kind | Bytes/char | Used when... |
|---|---|---|
| `PyUnicode_1BYTE_KIND` | 1 byte | every code point ≤ U+00FF (Latin-1 range) |
| `PyUnicode_2BYTE_KIND` | 2 bytes | every code point ≤ U+FFFF (fits in BMP), and at least one is > U+00FF |
| `PyUnicode_4BYTE_KIND` | 4 bytes | at least one code point > U+FFFF (e.g. emoji) |

```python
>>> import sys
>>> sys.getsizeof('a')        # pure ASCII → 1-byte kind
50
>>> sys.getsizeof('é')        # Latin-1 range → still 1-byte kind
61
>>> sys.getsizeof('好')       # BMP, beyond Latin-1 → 2-byte kind
60
>>> sys.getsizeof('😀')       # beyond BMP → 4-byte kind
76
```

Note this means **a string is not always cheaper just because it has fewer characters** — `'好'` (one CJK character) actually uses *more* bytes per character internally than `'hello'` does, because the whole string is upgraded to whichever width its *widest* character requires.

### 6.2 The C Struct Layout (Conceptual)

CPython represents every `str` with a layered struct, roughly:

```c
struct PyASCIIObject {
    PyObject_HEAD            // refcount, type pointer
    Py_ssize_t length;       // number of code points
    Py_hash_t hash;          // cached hash (-1 until first hashed), so dict
                             // lookups of the same string don't rehash
    struct {
        unsigned int interned:2;   // 0, 1, or 2 — see Interning section
        unsigned int kind:3;       // which of the 3 storage widths
        unsigned int compact:1;
        unsigned int ascii:1;      // pure ASCII fast-path flag
        unsigned int ready:1;
    } state;
    wchar_t *wstr;            // legacy cache, usually NULL now
};
// if ascii==1, the actual character data is stored immediately
// after this struct ("compact ASCII" — single allocation, very cheap)

struct PyCompactUnicodeObject {
    PyASCIIObject _base;
    Py_ssize_t utf8_length;   // cached UTF-8 byte length
    char *utf8;               // cached UTF-8 encoding (computed lazily, reused)
    Py_ssize_t wstr_length;
};

struct PyUnicodeObject {       // only for "legacy"/non-compact strings (rare)
    PyCompactUnicodeObject _base;
    union {
        void *any;
        Py_UCS1 *latin1;
        Py_UCS2 *ucs2;
        Py_UCS4 *ucs4;
    } data;
};
```

Key practical takeaways from this layout:

- **The hash is cached.** Computing `hash("some long string")` once means every subsequent dict/set lookup with that exact string object reuses the cached value — this is part of why strings make such fast dict keys.
- **The UTF-8 encoding is cached** the first time it's computed, so repeated `.encode('utf-8')` calls on the same object after the first are cheap.
- **ASCII strings get a single, compact memory allocation** with the header and the character data packed together — the most memory-efficient and CPU-cache-friendly representation.

### 6.3 Immutability and Why It Matters

`str` objects are **immutable** — once created, the character data never changes. Every method that looks like it "modifies" a string (`.upper()`, `.replace()`, `+=`, slicing) actually **creates and returns a brand-new string object**.

```python
>>> s = "hello"
>>> id(s)
4385672112
>>> s += " world"
>>> id(s)
4385822960          # different object — the original "hello" wasn't touched
```

**Why immutability is a deliberate design choice:**

1. **Hashability** — only immutable objects can be safely used as dict keys / set members, because their hash must never change while they're stored in a hash table.
2. **Safety for sharing** — since a string can never change underneath you, it's safe for two variables to point at the *same* string object (see Interning below) without one accidentally affecting the other.
3. **Thread-safety** — read-only data needs no locking.

**Performance consequence — quadratic string building:**

```python
# BAD: O(n²) — each += allocates an entirely new, longer string
result = ""
for word in big_list_of_words:
    result += word + " "

# GOOD: O(n) — list collects references, join allocates the final string once
result = " ".join(big_list_of_words)
```

### 6.4 String Slicing and Memory

```python
>>> s = "hello world"
>>> sub = s[0:5]
>>> sub
'hello'
```
Unlike some other languages, CPython slicing on `str` creates a brand-new, independent string object (it does **not** keep a reference into the parent's buffer) — important to know if you were hoping slicing a giant string would be "free."

---

## 7. String Interning

### 7.1 What Interning Means

**Interning** is the technique of keeping only *one* copy of each distinct immutable value in memory and having every reference to "that value" point to the *same* object, instead of creating duplicates. CPython maintains a global interned-string table for exactly this purpose.

The practical effect: two interned strings with the same content are not just `==` (equal value) but also `is` (the *same object in memory*).

```python
>>> a = "hello"
>>> b = "hello"
>>> a is b
True          # both point to the one interned 'hello' object
>>> a == b
True
```

### 7.2 When CPython Interns Automatically

CPython's compiler interns strings under these circumstances (implementation detail — can vary across versions, never something to *rely* on):

1. **String literals that look like identifiers** (letters, digits, underscores only) are interned at compile time, because they're likely to be used as variable/attribute/dict-key names repeatedly.
2. **Compile-time constant folding**: if the *same* literal text appears, possibly built from concatenated literals, within one code object, the compiler often produces a single shared constant.
3. Strings used as `dict` keys for things like `**kwargs`, attribute names, module/function names — these get interned by `dict` and `import` machinery for fast comparison.

```python
>>> a = "hello"          # identifier-like literal → interned
>>> b = "hello"
>>> a is b
True

>>> c = "hello world"    # contains a space — NOT identifier-like
>>> d = "hello world"
>>> c is d
True   # ⚠ still True here, because the compiler folded both LITERALS
       # in the same module into one shared constant at compile time —
       # NOT the same mechanism as identifier interning, but same visible effect
```

### 7.3 When Interning Does *NOT* Happen

Strings **built at runtime** (via concatenation of variables, `.join()`, `str()` conversions, slicing, `.format()`, f-strings with variables, I/O, etc.) are **not** automatically interned, even if their value matches an existing string:

```python
>>> x = "".join(["hel", "lo", " ", "wor", "ld"])
>>> y = "".join(["hel", "lo", " ", "wor", "ld"])
>>> x == y
True
>>> x is y
False         # two distinct objects — built at runtime, never interned

>>> n = 100
>>> s1 = str(n)
>>> s2 = str(n)
>>> s1 is s2
False         # str(int) always builds a fresh object
```

### 7.4 Forcing Interning Manually: `sys.intern()`

You can opt any string into the interned table yourself with `sys.intern()`. It returns the canonical interned object — either the string you passed in (now added to the table) or a previously-interned object with the same value:

```python
import sys

x = sys.intern("".join(["hel", "lo"]))
y = sys.intern("".join(["hel", "lo"]))
x is y           # True — both calls return the SAME interned object
```

**When this is actually useful:** if you're processing millions of repeated string tokens (e.g. parsing a huge log file where the same status strings — `"ERROR"`, `"INFO"`, `"WARNING"` — repeat constantly), manually interning them with `sys.intern()` saves memory (one copy instead of millions of duplicates) and speeds up equality checks (pointer comparison instead of character-by-character comparison).

### 7.5 The Golden Rule

> **Never use `is` to compare string *values*. Always use `==`.**

```python
# WRONG — relies on an implementation detail that can silently change
if my_string is "hello":
    ...

# RIGHT
if my_string == "hello":
    ...
```

`is` checks *object identity*; interning is a CPython memory-optimization detail, not a language guarantee. Other Python implementations (PyPy, Jython, IronPython) may intern differently or not at all, and even CPython's own behavior has changed between versions.

---

## 8. String Literals & Escape Sequences

### 8.1 Quoting Styles

```python
single = 'hello'
double = "hello"
triple_single = '''multi
line'''
triple_double = """multi
line, often used for docstrings"""
```
Single and double quotes are functionally identical — pick one consistently; use the other when your string contains a quote character to avoid escaping:

```python
s = "it's fine"      # avoids escaping the apostrophe
s = 'she said "hi"'  # avoids escaping the double quotes
```

### 8.2 Common Escape Sequences

| Escape | Meaning |
|---|---|
| `\n` | newline |
| `\t` | tab |
| `\\` | literal backslash |
| `\'` `\"` | literal quote |
| `\r` | carriage return |
| `\0` | null character |
| `\xhh` | character with hex value `hh` (1 byte, ≤ 0xFF) |
| `\uXXXX` | Unicode character, 4 hex digits (BMP, ≤ U+FFFF) |
| `\UXXXXXXXX` | Unicode character, 8 hex digits (any code point) |
| `\N{NAME}` | Unicode character by official name |

```python
>>> "\u00e9"
'é'
>>> "\U0001F600"
'😀'
>>> "\N{GRINNING FACE}"
'😀'
```

### 8.3 Raw Strings

`r"..."` disables escape-sequence processing — every backslash is literal. Essential for regular expressions and Windows file paths:

```python
>>> path = r"C:\Users\new_folder"   # without 'r', \n would become a newline!
>>> pattern = r"\d+\.\d+"            # regex meaning "digits, dot, digits"
```

### 8.4 f-strings (Formatted String Literals)

```python
name, age = "Suresh", 23
f"{name} is {age} years old"          # 'Suresh is 23 years old'
f"{age * 2 = }"                       # 'age * 2 = 46'  (debug specifier, 3.8+)
f"{3.14159:.2f}"                      # '3.14'
f"{'right':>10}"                      # '     right'
```

### 8.5 Byte Strings

`b"..."` creates a `bytes` object directly (not a `str`) — only ASCII characters and `\x` escapes are allowed inside:

```python
>>> b = b"hello"
>>> type(b)
<class 'bytes'>
```

---

## 9. Every String Method

CPython's `str` type defines **47 public methods**. Each one is listed below with its signature, a one-line explanation, and a runnable example. They're grouped by purpose.

### 9.1 Case Conversion

#### `str.upper()`
Converts all characters to uppercase.
```python
>>> "hello World".upper()
'HELLO WORLD'
```

#### `str.lower()`
Converts all characters to lowercase.
```python
>>> "Hello World".lower()
'hello world'
```

#### `str.capitalize()`
Makes the first character uppercase, the rest lowercase.
```python
>>> "hello WORLD".capitalize()
'Hello world'
```

#### `str.title()`
Makes the first letter of every word uppercase, rest lowercase.
```python
>>> "hello world".title()
'Hello World'
>>> "they're bill's friends".title()    # apostrophes count as word boundaries
"They'Re Bill'S Friends"                # ⚠ known quirk
```

#### `str.swapcase()`
Flips uppercase ↔ lowercase for every character.
```python
>>> "Hello World".swapcase()
'hELLO wORLD'
```

#### `str.casefold()`
Aggressive lowercasing for *caseless matching* — handles edge cases `.lower()` misses (e.g. the German ß).
```python
>>> "straße".casefold()
'strasse'
>>> "straße".lower()
'straße'        # .lower() does NOT expand ß
```

### 9.2 Search & Count

#### `str.find(sub, start=0, end=len)`
Returns the lowest index of `sub`, or **`-1`** if not found (never raises).
```python
>>> "hello world".find("world")
6
>>> "hello world".find("xyz")
-1
```

#### `str.rfind(sub, start=0, end=len)`
Like `find()`, but searches from the right; returns the *highest* index.
```python
>>> "abcabc".rfind("abc")
3
```

#### `str.index(sub, start=0, end=len)`
Like `find()`, but **raises `ValueError`** if not found.
```python
>>> "hello world".index("world")
6
>>> "hello world".index("xyz")
ValueError: substring not found
```

#### `str.rindex(sub, start=0, end=len)`
Like `rfind()`, but raises `ValueError` if not found.
```python
>>> "abcabc".rindex("abc")
3
```

#### `str.count(sub, start=0, end=len)`
Counts non-overlapping occurrences of `sub`.
```python
>>> "abababab".count("ab")
4
>>> "aaaa".count("aa")
2          # non-overlapping! not 3
```

#### `str.startswith(prefix, start=0, end=len)`
Returns `True`/`False`. `prefix` can be a string **or a tuple of strings**.
```python
>>> "hello.py".startswith("hello")
True
>>> "report.csv".startswith((".py", ".csv", ".txt"))
True
```

#### `str.endswith(suffix, start=0, end=len)`
Same as above, checking the end.
```python
>>> "report.csv".endswith((".py", ".csv"))
True
```

### 9.3 Validation (`is*` Predicates)

All of these return `False` on an **empty string**, and `True` only if *every* character in the string satisfies the condition (and the string is non-empty).

#### `str.isalpha()`
True if every character is alphabetic (Unicode letters from any language).
```python
>>> "hello".isalpha()
True
>>> "héllo".isalpha()
True
>>> "hello123".isalpha()
False
```

#### `str.isdigit()`
True if every character is a digit, including some Unicode digit forms (e.g. superscripts) that don't have numeric *value* semantics.
```python
>>> "12345".isdigit()
True
>>> "²".isdigit()
True
```

#### `str.isdecimal()`
True only for characters representing **base-10 decimal digits** — the strictest of the three digit checks; required for `int()` conversion in some locales.
```python
>>> "12345".isdecimal()
True
>>> "²".isdecimal()
False
```

#### `str.isnumeric()`
True for digits plus characters with numeric meaning, like fractions or Roman/CJK numerals — the broadest of the three.
```python
>>> "½".isnumeric()
True
>>> "Ⅷ".isnumeric()     # Roman numeral 8
True
```

> **Digit hierarchy:** `isdecimal()` ⊆ `isdigit()` ⊆ `isnumeric()` (decimal digits are a subset of digits, which are a subset of all numeric characters).

#### `str.isalnum()`
True if every character is alphabetic **or** numeric (any of the three digit kinds above).
```python
>>> "abc123".isalnum()
True
>>> "abc 123".isalnum()
False     # space is neither
```

#### `str.isascii()`
True if every character is in the ASCII range (U+0000–U+007F). Empty string returns `True` here (the one exception to the "empty = False" rule).
```python
>>> "hello".isascii()
True
>>> "héllo".isascii()
False
```

#### `str.isspace()`
True if every character is whitespace (space, tab, newline, etc.).
```python
>>> "   \t\n".isspace()
True
```

#### `str.islower()`
True if all cased characters are lowercase and at least one cased character exists.
```python
>>> "hello world".islower()
True
>>> "hello123".islower()
True
>>> "123".islower()
False     # no cased characters at all
```

#### `str.isupper()`
True if all cased characters are uppercase and at least one exists.
```python
>>> "HELLO".isupper()
True
```

#### `str.istitle()`
True if the string is "titlecased" — each word starts with an uppercase letter followed only by lowercase letters.
```python
>>> "Hello World".istitle()
True
>>> "Hello world".istitle()
False
```

#### `str.isidentifier()`
True if the string would be a valid Python identifier (variable name) syntactically — does **not** check against reserved keywords.
```python
>>> "my_var".isidentifier()
True
>>> "2fast".isidentifier()
False
>>> "class".isidentifier()
True       # syntactically valid even though it's a reserved keyword
>>> import keyword
>>> keyword.iskeyword("class")   # use this to catch the keyword case
True
```

#### `str.isprintable()`
True if every character is printable (or the string is empty) — `False` if it contains control characters like `\n` or `\t`.
```python
>>> "hello".isprintable()
True
>>> "hello\n".isprintable()
False
```

### 9.4 Whitespace & Padding

#### `str.strip(chars=None)`
Removes leading/trailing whitespace by default, or any character in `chars` if given.
```python
>>> "  hello  ".strip()
'hello'
>>> "xxhelloxx".strip("x")
'hello'
```

#### `str.lstrip(chars=None)`
Same as `strip()`, only from the left.
```python
>>> "  hello  ".lstrip()
'hello  '
```

#### `str.rstrip(chars=None)`
Same as `strip()`, only from the right.
```python
>>> "  hello  ".rstrip()
'  hello'
```

#### `str.center(width, fillchar=' ')`
Centers the string in a field of the given width.
```python
>>> "hi".center(10, "-")
'----hi----'
```

#### `str.ljust(width, fillchar=' ')`
Left-justifies (pads on the right).
```python
>>> "hi".ljust(10, ".")
'hi........'
```

#### `str.rjust(width, fillchar=' ')`
Right-justifies (pads on the left).
```python
>>> "hi".rjust(10, ".")
'........hi'
```

#### `str.zfill(width)`
Pads a numeric string with leading zeros (handles a leading `+`/`-` sign correctly).
```python
>>> "42".zfill(5)
'00042'
>>> "-42".zfill(5)
'-0042'
```

#### `str.expandtabs(tabsize=8)`
Replaces every `\t` with enough spaces to reach the next tab stop.
```python
>>> "a\tb".expandtabs(4)
'a   b'
```

### 9.5 Splitting & Joining

#### `str.split(sep=None, maxsplit=-1)`
Splits on `sep` (or any whitespace run, collapsing multiples, if `sep` is omitted).
```python
>>> "a, b,  c".split(",")
['a', ' b', '  c']
>>> "  a   b  c  ".split()       # sep=None: smart whitespace splitting
['a', 'b', 'c']
>>> "a,b,c,d".split(",", maxsplit=2)
['a', 'b', 'c,d']
```

#### `str.rsplit(sep=None, maxsplit=-1)`
Like `split()`, but applies `maxsplit` starting from the right.
```python
>>> "a,b,c,d".rsplit(",", maxsplit=2)
['a,b', 'c', 'd']
```

#### `str.splitlines(keepends=False)`
Splits on line boundaries (`\n`, `\r\n`, `\r`, and several Unicode line separators), without leaving empty trailing entries the way `split("\n")` would.
```python
>>> "line1\nline2\r\nline3".splitlines()
['line1', 'line2', 'line3']
>>> "line1\nline2".splitlines(keepends=True)
['line1\n', 'line2']
```

#### `str.partition(sep)`
Splits on the **first** occurrence of `sep` into exactly 3 parts: (before, sep, after). If not found: (original, '', '').
```python
>>> "key=value=extra".partition("=")
('key', '=', 'value=extra')
```

#### `str.rpartition(sep)`
Same, but splits on the **last** occurrence.
```python
>>> "key=value=extra".rpartition("=")
('key=value', '=', 'extra')
```

#### `str.join(iterable)`
Joins an iterable of strings using `self` as the separator — the efficient, idiomatic way to build a string from parts.
```python
>>> ",".join(["a", "b", "c"])
'a,b,c'
>>> "".join(["h", "e", "l", "l", "o"])
'hello'
```

### 9.6 Modification & Replacement

#### `str.replace(old, new, count=-1)`
Returns a copy with occurrences of `old` replaced by `new`; `count` limits how many (from the left).
```python
>>> "ababab".replace("a", "X")
'XbXbXb'
>>> "ababab".replace("a", "X", 1)
'Xbabab'
```

#### `str.translate(table)`
Replaces characters using a mapping table (built with `str.maketrans`) — much faster than chained `.replace()` calls for character-level substitution.
```python
>>> table = str.maketrans("abc", "123")
>>> "cabbage".translate(table)
'31221ge'
```

#### `str.maketrans(x, y=None, z=None)` *(static method)*
Builds the translation table used by `translate()`.
- One arg: a dict mapping ordinals/characters → replacement.
- Two args: equal-length strings, character-for-character mapping.
- Three args: two equal-length strings + a string of characters to delete (map to `None`).
```python
>>> table = str.maketrans("abc", "123", "xyz")
>>> "axbycz".translate(table)
'123'          # a→1, b→2, c→3, and x,y,z are deleted
```

#### `str.removeprefix(prefix)` *(Python 3.9+)*
Removes `prefix` if present, otherwise returns the string unchanged — safer than slicing, which can silently cut the wrong characters if the prefix isn't actually there.
```python
>>> "test_file.py".removeprefix("test_")
'file.py'
>>> "file.py".removeprefix("test_")
'file.py'      # unchanged, no error
```

#### `str.removesuffix(suffix)` *(Python 3.9+)*
Same idea, from the end.
```python
>>> "archive.tar.gz".removesuffix(".gz")
'archive.tar'
```

### 9.7 Formatting

#### `str.format(*args, **kwargs)`
The general-purpose template-formatting method, predecessor to f-strings (still very common for templates stored as data, e.g. config files).
```python
>>> "{} is {} years old".format("Suresh", 23)
'Suresh is 23 years old'
>>> "{name} scored {score:.1f}%".format(name="Suresh", score=91.567)
'Suresh scored 91.6%'
>>> "{0} + {0} = {1}".format(2, 4)     # positional index reuse
'2 + 2 = 4'
```

#### `str.format_map(mapping)`
Like `.format(**mapping)`, but uses the mapping object directly without copying it into a dict first — useful with custom mapping types (e.g. one that supplies defaults for missing keys).
```python
>>> data = {"name": "Suresh", "role": "Developer"}
>>> "{name}: {role}".format_map(data)
'Suresh: Developer'
```

### 9.8 Encoding

#### `str.encode(encoding='utf-8', errors='strict')`
Covered in depth in [Section 4](#4-encoding-str--bytes).
```python
>>> "café".encode("utf-8")
b'caf\xc3\xa9'
```

### 9.9 Quick Reference — All 47 Methods at a Glance

```
Case:        upper, lower, capitalize, title, swapcase, casefold
Search:      find, rfind, index, rindex, count, startswith, endswith
Validate:    isalnum, isalpha, isascii, isdecimal, isdigit, isidentifier,
             islower, isnumeric, isprintable, isspace, istitle, isupper
Whitespace:  strip, lstrip, rstrip, center, ljust, rjust, zfill, expandtabs
Split/Join:  split, rsplit, splitlines, partition, rpartition, join
Modify:      replace, translate, maketrans, removeprefix, removesuffix
Format:      format, format_map
Encode:      encode
```

---

## 10. Regular Expressions

Regular expressions (regex) describe **patterns** for matching text — far more powerful than `find()`/`replace()` for anything beyond a fixed literal substring. Python's regex engine lives in the built-in `re` module.

```python
import re
```

### 10.1 Core Functions

| Function | What it does |
|---|---|
| `re.search(pattern, string)` | Finds the **first** match anywhere in the string; returns a `Match` object or `None` |
| `re.match(pattern, string)` | Like `search`, but only matches at the **start** of the string |
| `re.fullmatch(pattern, string)` | Matches only if the **entire** string matches the pattern |
| `re.findall(pattern, string)` | Returns a **list of all matches** (as strings, or tuples if there are groups) |
| `re.finditer(pattern, string)` | Like `findall`, but returns an **iterator of Match objects** (memory-efficient for huge text) |
| `re.sub(pattern, repl, string, count=0)` | Replaces matches with `repl`; returns the new string |
| `re.subn(pattern, repl, string)` | Like `sub`, but returns `(new_string, number_of_replacements)` |
| `re.split(pattern, string)` | Splits the string wherever the pattern matches |
| `re.compile(pattern, flags=0)` | Pre-compiles a pattern into a reusable `Pattern` object (faster for repeated use) |

```python
>>> re.search(r"\d+", "Order #4521 shipped")
<re.Match object; span=(7, 11), match='4521'>
>>> re.match(r"\d+", "Order #4521 shipped")
None                  # doesn't match at position 0
>>> re.findall(r"\d+", "a12 b34 c56")
['12', '34', '56']
>>> re.sub(r"\d+", "#", "a12 b34 c56")
'a# b# c#'
```

### 10.2 Pattern Syntax — Metacharacters

| Symbol | Meaning |
|---|---|
| `.` | Any character except newline (unless `re.DOTALL`) |
| `^` | Start of string (or line, with `re.MULTILINE`) |
| `$` | End of string (or line, with `re.MULTILINE`) |
| `*` | 0 or more of the preceding token |
| `+` | 1 or more of the preceding token |
| `?` | 0 or 1 of the preceding token (also marks "lazy" quantifiers, see below) |
| `{m,n}` | Between `m` and `n` repetitions |
| `[]` | Character class — matches any **one** character inside |
| `\|` | Alternation ("or") |
| `()` | Grouping (and capturing) |
| `\` | Escapes a metacharacter, or starts a special sequence like `\d` |

### 10.3 Character Classes & Special Sequences

| Sequence | Matches |
|---|---|
| `\d` | Any digit (`[0-9]` in ASCII mode) |
| `\D` | Any non-digit |
| `\w` | "Word" character: letters, digits, underscore |
| `\W` | Any non-word character |
| `\s` | Any whitespace (space, tab, newline...) |
| `\S` | Any non-whitespace |
| `\b` | Word boundary (zero-width — matches a *position*, not a character) |
| `\B` | Not a word boundary |
| `[abc]` | `a`, `b`, or `c` |
| `[^abc]` | Any character **except** `a`, `b`, `c` |
| `[a-z]` | Any character in the range `a` to `z` |
| `[a-zA-Z0-9_]` | Equivalent to `\w` (in ASCII mode) |

```python
>>> re.findall(r"\w+", "hello, world! 123")
['hello', 'world', '123']
>>> re.findall(r"[aeiou]", "hello world")
['e', 'o', 'o']
>>> re.search(r"\bcat\b", "concatenate")
None                # \b prevents matching inside "concatenate"
>>> re.search(r"\bcat\b", "a cat sat")
<re.Match object; span=(2, 5), match='cat'>
```

### 10.4 Quantifiers — Greedy vs Lazy

By default, quantifiers are **greedy** — they match as much as possible. Adding `?` after a quantifier makes it **lazy** — it matches as little as possible.

```python
>>> re.findall(r"<.+>", "<a><b>")        # greedy: spans across both tags
['<a><b>']
>>> re.findall(r"<.+?>", "<a><b>")       # lazy: stops at the first '>'
['<a>', '<b>']
```

| Quantifier | Greedy | Lazy |
|---|---|---|
| 0 or more | `*` | `*?` |
| 1 or more | `+` | `+?` |
| 0 or 1 | `?` | `??` |
| exactly m–n | `{m,n}` | `{m,n}?` |

### 10.5 Groups

#### Capturing groups `(...)`
```python
>>> m = re.search(r"(\d{3})-(\d{4})", "Call 555-1234")
>>> m.group(0)     # the whole match
'555-1234'
>>> m.group(1)     # first group
'555'
>>> m.group(2)     # second group
'1234'
>>> m.groups()     # all groups as a tuple
('555', '1234')
```

#### Named groups `(?P<name>...)`
```python
>>> m = re.search(r"(?P<area>\d{3})-(?P<number>\d{4})", "Call 555-1234")
>>> m.group("area")
'555'
>>> m.groupdict()
{'area': '555', 'number': '1234'}
```

#### Non-capturing groups `(?:...)`
Groups for the purpose of applying a quantifier or alternation, without creating a numbered capture:
```python
>>> re.findall(r"(?:ab)+", "ababab xyz ababab")
['ababab', 'ababab']
```

#### Backreferences `\1`, `\2`, ...
Refer back to a previously captured group **within the same pattern** — useful for matching repeated words:
```python
>>> re.findall(r"\b(\w+) \1\b", "hello hello world world end")
['hello', 'world']
```

### 10.6 Lookahead & Lookbehind (Zero-Width Assertions)

These check that something is (or isn't) adjacent, **without consuming characters** as part of the match.

| Syntax | Meaning |
|---|---|
| `(?=...)` | Positive lookahead — followed by `...` |
| `(?!...)` | Negative lookahead — NOT followed by `...` |
| `(?<=...)` | Positive lookbehind — preceded by `...` |
| `(?<!...)` | Negative lookbehind — NOT preceded by `...` |

```python
>>> re.findall(r"\d+(?=px)", "10px 20em 30px")   # numbers immediately before "px"
['10', '30']
>>> re.findall(r"(?<=\$)\d+", "Price: $50, Qty: 3")  # numbers right after "$"
['50']
```

### 10.7 Match Object Methods

```python
m = re.search(r"(\d{3})-(\d{4})", "Call 555-1234 now")
m.group()      # '555-1234'  — entire match
m.start()      # 5           — start index in the original string
m.end()        # 13          — end index
m.span()       # (5, 13)     — (start, end)
m.string       # 'Call 555-1234 now' — the original input string
```

### 10.8 Flags

| Flag | Effect |
|---|---|
| `re.IGNORECASE` / `re.I` | Case-insensitive matching |
| `re.MULTILINE` / `re.M` | `^`/`$` match at the start/end of **every line**, not just the whole string |
| `re.DOTALL` / `re.S` | `.` also matches newline characters |
| `re.VERBOSE` / `re.X` | Allows whitespace and `#` comments inside the pattern, for readability |

```python
>>> re.findall(r"^\w+", "line one\nline two", re.MULTILINE)
['line', 'line']

>>> pattern = re.compile(r"""
...     (?P<area>\d{3})   # area code
...     -
...     (?P<number>\d{4}) # local number
... """, re.VERBOSE)
>>> pattern.search("555-1234").groupdict()
{'area': '555', 'number': '1234'}
```

### 10.9 Compiling Patterns for Reuse

If you'll run the same pattern many times (e.g. inside a loop processing thousands of lines), `re.compile()` parses the pattern once and reuses the compiled form — meaningfully faster than calling `re.search(pattern, ...)` fresh each time, even though the module-level functions internally cache the last few compiled patterns:

```python
pattern = re.compile(r"\b\w+@\w+\.\w+\b")   # simple email-like pattern
for line in huge_log_file:
    for match in pattern.finditer(line):
        print(match.group())
```

### 10.10 Common Practical Patterns

```python
# Simple email matcher (not fully RFC-compliant, but good for most real-world text)
email_pattern = r"[\w.+-]+@[\w-]+\.[\w.-]+"

# Phone number (US-style, flexible separators)
phone_pattern = r"\(?\d{3}\)?[-.\s]?\d{3}[-.\s]?\d{4}"

# Extract all hashtags
hashtag_pattern = r"#\w+"

# Validate a simple identifier-style username
username_pattern = r"^[a-zA-Z_][a-zA-Z0-9_]{2,15}$"

# Strip HTML tags (simple cases only — use a real HTML parser for anything complex)
clean_text = re.sub(r"<[^>]+>", "", "<p>Hello <b>world</b></p>")
# 'Hello world'
```

```python
>>> re.findall(r"[\w.+-]+@[\w-]+\.[\w.-]+", "Contact: a@b.com or x.y@test.co.in")
['a@b.com', 'x.y@test.co.in']
```

### 10.11 `re.split` with Capturing Groups

If the split pattern contains capturing groups, the captured text is **included** in the result list — handy for splitting while keeping the delimiters:

```python
>>> re.split(r"(\s*,\s*)", "a, b,  c")
['a', ', ', 'b', ',  ', 'c']
>>> re.split(r"\s*,\s*", "a, b,  c")       # no group → delimiters dropped
['a', 'b', 'c']
```

---

## Summary Cheat-Sheet

```
Unicode        → universal character-to-number mapping (code points, U+XXXX)
UTF-8          → variable-width (1-4 byte) encoding of code points; Python's default
encode()       → str → bytes      (needs an encoding)
decode()       → bytes → str      (needs to know which encoding was used)
PEP 393        → CPython stores each str at the narrowest width (1/2/4 bytes/char)
                 that fits ALL its characters
Interning      → identical immutable strings can share one object; check with `is`,
                 but always COMPARE values with `==`
str is immutable → every "modifying" method returns a NEW string
47 str methods   → case, search, validate (is*), whitespace, split/join,
                    modify/replace, format, encode
re module        → search/match/findall/finditer/sub/subn/split/compile,
                    groups, named groups, lookaround, flags
```
