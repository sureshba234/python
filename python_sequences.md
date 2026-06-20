# Python Sequences — A Complete Guide

## Table of Contents

1. [What Is a Sequence](#1-what-is-a-sequence)
2. [The Sequence Abstraction](#2-the-sequence-abstraction)
3. [Built-in Sequence Types](#3-built-in-sequence-types)
4. [Mutable vs Immutable Sequences](#4-mutable-vs-immutable-sequences)
5. [Abstract Base Classes: `Sequence` and `MutableSequence`](#5-abstract-base-classes-sequence-and-mutablesequence)
6. [Indexing](#6-indexing)
7. [Slicing](#7-slicing)
8. [Common Sequence Operations](#8-common-sequence-operations)
9. [Iteration and the Iterable/Sequence Relationship](#9-iteration-and-the-iterablesequence-relationship)
10. [Unpacking Sequences](#10-unpacking-sequences)
11. [Concatenation and Repetition](#11-concatenation-and-repetition)
12. [Sequence Comparison and Equality](#12-sequence-comparison-and-equality)
13. [Copying Sequences](#13-copying-sequences)
14. [Building a Custom Sequence](#14-building-a-custom-sequence)
15. [Sequences vs Mappings vs Sets](#15-sequences-vs-mappings-vs-sets)
16. [Performance Notes](#16-performance-notes)
17. [Type Hints for Sequences](#17-type-hints-for-sequences)
18. [Best Practices](#18-best-practices)
19. [Common Mistakes](#19-common-mistakes)
20. [Interview Questions](#20-interview-questions)
21. [Quick Reference](#21-quick-reference)

---

## 1. What Is a Sequence

A **sequence** is an **ordered collection** of items accessed by integer index, starting at `0`. Sequences support indexing, slicing, length checks, and iteration in a defined, positional order.

```python
s = [10, 20, 30]
print(s[0])
print(s[-1])
print(len(s))
```

**Output:**
```
10
30
3
```

`list`, `tuple`, `str`, `bytes`, `bytearray`, and `range` are all sequences. Just as `dict` is the primary concrete **mapping**, `list` is the primary concrete **mutable sequence**, and `tuple`/`str` are primary concrete **immutable sequences**.

---

## 2. The Sequence Abstraction

Like mappings, "sequence" is a **protocol**, not a single type. Any object that supports `__getitem__` with integer indices (starting from 0) and `__len__` can behave like a sequence, even without being a built-in type.

```
            Sequence Protocol
   ┌───────────────────────────────┐
   │ __getitem__(index)             │
   │ __len__()                      │
   │ __contains__   (derivable)     │
   │ __iter__       (derivable)     │
   │ index(), count() (derivable)   │
   └───────────────────────────────┘
              │
   ┌──────────┼──────────┬─────────┐
   │          │          │         │
  list      tuple       str    custom classes
(mutable) (immutable) (immutable) (e.g. a Deck of cards)
```

This is why old-style classes that only define `__getitem__` (without even `__iter__`) can still be looped over with `for` — Python falls back to repeated indexing starting at 0 until an `IndexError` is raised.

---

## 3. Built-in Sequence Types

| Type | Mutable | Holds | Example |
|---|---|---|---|
| `list` | Yes | Any objects | `[1, "a", 3.0]` |
| `tuple` | No | Any objects | `(1, "a", 3.0)` |
| `str` | No | Characters | `"hello"` |
| `bytes` | No | Integers 0–255 | `b"hello"` |
| `bytearray` | Yes | Integers 0–255 | `bytearray(b"hello")` |
| `range` | No (and not stored in memory) | Integers | `range(0, 10)` |

```python
my_list = [1, 2, 3]
my_tuple = (1, 2, 3)
my_str = "abc"
my_bytes = b"abc"
my_bytearray = bytearray(b"abc")
my_range = range(5)

for seq in (my_list, my_tuple, my_str, my_bytes, my_bytearray, my_range):
    print(type(seq).__name__, "->", seq[0], len(seq))
```

**Output:**
```
list -> 1 3
tuple -> 1 3
str -> a 3
bytes -> 97 3
bytearray -> 97 3
range -> 0 5
```

> Note: indexing `bytes`/`bytearray` returns an **integer** (the byte's value), not a one-character string.

---

## 4. Mutable vs Immutable Sequences

| Feature | Mutable (`list`, `bytearray`) | Immutable (`tuple`, `str`, `bytes`, `range`) |
|---|---|---|
| Can change in place | Yes | No |
| Supports item assignment | Yes (`s[0] = x`) | No (`TypeError`) |
| Hashable | No | Yes (if elements are hashable) |
| Use as dict key / set element | No | Yes |
| Typical use | Data that changes over time | Fixed records, constants, dict keys |

```python
t = (1, 2, 3)
try:
    t[0] = 99
except TypeError as e:
    print("Error:", e)

l = [1, 2, 3]
l[0] = 99
print(l)
```

**Output:**
```
Error: 'tuple' object does not support item assignment
[99, 2, 3]
```

---

## 5. Abstract Base Classes: `Sequence` and `MutableSequence`

The `collections.abc` module defines the formal contract for sequences, mirroring the pattern used for mappings.

```python
from collections.abc import Sequence, MutableSequence

print(isinstance([1, 2], Sequence))         # True
print(isinstance([1, 2], MutableSequence))  # True
print(isinstance((1, 2), Sequence))         # True
print(isinstance((1, 2), MutableSequence))  # False (tuples are immutable)
```

**Output:**
```
True
True
True
False
```

### `Sequence` (Read-Only Interface)

Requires implementing only:
- `__getitem__`
- `__len__`

From these two, the ABC automatically provides `__contains__`, `__iter__`, `__reversed__`, `index()`, and `count()`.

### `MutableSequence` (Read-Write Interface)

Extends `Sequence` and additionally requires:
- `__setitem__`
- `__delitem__`
- `insert()`

From these, it automatically provides `append()`, `reverse()`, `extend()`, `pop()`, `remove()`, and `__iadd__` (`+=`).

```
              Sequence
   (read-only: __contains__, __iter__,
    __reversed__, index, count)
                  │
                  ▼
           MutableSequence
   (adds: append, reverse, extend,
    pop, remove, __iadd__)
                  │
                  ▼
                 list
   (concrete, optimized implementation)
```

---

## 6. Indexing

```python
s = ["a", "b", "c", "d", "e"]

print(s[0])      # first item
print(s[-1])     # last item
print(s[2])      # third item

try:
    print(s[10])
except IndexError as e:
    print("Error:", e)
```

**Output:**
```
a
e
c
Error: list index out of range
```

Negative indices count from the end: `-1` is the last item, `-2` the second-to-last, and so on.

```
Index:     0    1    2    3    4
Value:    'a'  'b'  'c'  'd'  'e'
Negative: -5   -4   -3   -2   -1
```

---

## 7. Slicing

Slicing extracts a sub-sequence using `sequence[start:stop:step]`.

```python
s = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

print(s[2:5])      # items 2,3,4
print(s[:3])       # first 3 items
print(s[7:])       # from index 7 to end
print(s[::2])      # every second item
print(s[::-1])     # reversed
print(s[-3:])      # last 3 items
```

**Output:**
```
[2, 3, 4]
[0, 1, 2]
[7, 8, 9]
[0, 2, 4, 6, 8]
[9, 8, 7, 6, 5, 4, 3, 2, 1, 0]
[7, 8, 9]
```

### Slicing Never Raises `IndexError`

```python
s = [1, 2, 3]
print(s[10:20])    # out-of-range slices return empty, not an error
```

**Output:**
```
[]
```

### Slice Assignment (Mutable Sequences Only)

```python
s = [1, 2, 3, 4, 5]
s[1:3] = [20, 30, 40]
print(s)
```

**Output:**
```
[1, 20, 30, 40, 4, 5]
```

> Slice assignment can change the length of a list, since you replace a *range* with a (potentially differently-sized) iterable.

---

## 8. Common Sequence Operations

These work across **all** sequence types unless noted otherwise.

| Operation | Description | Example |
|---|---|---|
| `len(s)` | Length | `len([1,2,3])` → `3` |
| `s[i]` | Indexing | `[1,2,3][0]` → `1` |
| `s[i:j]` | Slicing | `[1,2,3][0:2]` → `[1, 2]` |
| `x in s` | Membership | `2 in [1,2,3]` → `True` |
| `s + s2` | Concatenation | `[1] + [2]` → `[1, 2]` |
| `s * n` | Repetition | `[1] * 3` → `[1, 1, 1]` |
| `s.index(x)` | First index of `x` | `[1,2,3].index(2)` → `1` |
| `s.count(x)` | Count of `x` | `[1,1,2].count(1)` → `2` |
| `min(s)` / `max(s)` | Smallest / largest | `max([3,1,2])` → `3` |
| `sorted(s)` | New sorted list | `sorted([3,1,2])` → `[1, 2, 3]` |
| `reversed(s)` | Reversed iterator | `list(reversed([1,2,3]))` → `[3, 2, 1]` |

```python
nums = [4, 1, 7, 1, 9]
print(len(nums))
print(7 in nums)
print(nums.index(1))     # first occurrence
print(nums.count(1))
print(sorted(nums))
print(list(reversed(nums)))
```

**Output:**
```
5
True
1
2
[1, 1, 4, 7, 9]
[9, 1, 7, 1, 4]
```

---

## 9. Iteration and the Iterable/Sequence Relationship

Every sequence is **iterable**, but not every iterable is a sequence. An iterable just needs `__iter__` (or fallback `__getitem__`); a sequence additionally guarantees **ordered, indexable** access and a defined `__len__`.

```
        Iterable
   (anything with __iter__,
    e.g. generators, sets, files)
            │
            ▼
        Sequence
   (ordered + indexable + sized:
    list, tuple, str, range...)
```

```python
def my_generator():
    yield 1
    yield 2

gen = my_generator()
print(hasattr(gen, "__iter__"))   # True - it's iterable
print(hasattr(gen, "__getitem__"))  # False - not a sequence
```

**Output:**
```
True
False
```

Generators are iterable but **not** sequences — they can't be indexed (`gen[0]` fails) and don't support `len()`.

---

## 10. Unpacking Sequences

```python
a, b, c = [1, 2, 3]
print(a, b, c)

first, *rest = [1, 2, 3, 4, 5]
print(first, rest)

*init, last = [1, 2, 3, 4, 5]
print(init, last)

x, (y, z) = [1, (2, 3)]
print(x, y, z)
```

**Output:**
```
1 2 3
1 [2, 3, 4, 5]
[1, 2, 3, 4] 5
1 2 3
```

> The number of variables must match the sequence length unless a starred (`*`) expression is used to capture the remainder.

```python
try:
    a, b = [1, 2, 3]
except ValueError as e:
    print("Error:", e)
```

**Output:**
```
Error: too many values to unpack (expected 2)
```

---

## 11. Concatenation and Repetition

```python
print([1, 2] + [3, 4])
print((1, 2) + (3, 4))
print("ab" + "cd")
print([0] * 5)
print("ab" * 3)
```

**Output:**
```
[1, 2, 3, 4]
(1, 2, 3, 4)
abcd
[0, 0, 0, 0, 0]
ababab
```

> **Mixing sequence types in `+` is not allowed** — you cannot concatenate a `list` with a `tuple` directly.

```python
try:
    [1, 2] + (3, 4)
except TypeError as e:
    print("Error:", e)
```

**Output:**
```
Error: can only concatenate list (not "tuple") to list
```

---

## 12. Sequence Comparison and Equality

Sequences compare **element-by-element**, in order (lexicographic comparison), as long as both are the same type or compatible types.

```python
print([1, 2, 3] == [1, 2, 3])
print([1, 2] < [1, 2, 3])    # shorter, equal-prefix list is "less"
print([1, 3] > [1, 2, 9])    # first differing element decides
print((1, 2) == [1, 2])      # different types -> not equal
```

**Output:**
```
True
True
True
False
```

> A `tuple` and a `list` with identical elements are **never equal** (`==`) because Python considers their *type* as part of equality for sequences — only same-type sequences compare element-wise.

---

## 13. Copying Sequences

### Shallow Copy

```python
original = [1, 2, [3, 4]]
copy1 = original.copy()      # or original[:] or list(original)
copy1[2].append(5)
print(original)
```

**Output:**
```
[1, 2, [3, 4, 5]]
```

Shallow copy duplicates the top-level sequence; nested mutable objects are still shared.

### Deep Copy

```python
import copy
original = [1, 2, [3, 4]]
deep = copy.deepcopy(original)
deep[2].append(5)
print(original)
print(deep)
```

**Output:**
```
[1, 2, [3, 4]]
[1, 2, [3, 4, 5]]
```

### Common Shallow-Copy Idioms

```python
a = [1, 2, 3]
b1 = a.copy()
b2 = a[:]
b3 = list(a)
print(b1, b2, b3)
```

**Output:**
```
[1, 2, 3] [1, 2, 3] [1, 2, 3]
```

---

## 14. Building a Custom Sequence

You can create your own sequence type by subclassing `collections.abc.Sequence` (read-only) or `MutableSequence` (read-write), implementing only the required methods.

```python
from collections.abc import Sequence

class Deck(Sequence):
    SUITS = ["Hearts", "Diamonds", "Clubs", "Spades"]
    RANKS = list(range(1, 14))

    def __init__(self):
        self._cards = [(rank, suit) for suit in self.SUITS for rank in self.RANKS]

    def __getitem__(self, index):
        return self._cards[index]

    def __len__(self):
        return len(self._cards)


deck = Deck()
print(len(deck))
print(deck[0])
print(deck[-1])
print(deck[:3])
print((1, "Hearts") in deck)
```

**Output:**
```
52
(1, 'Hearts')
(13, 'Spades')
[(1, 'Hearts'), (2, 'Hearts'), (3, 'Hearts')]
True
```

Because `Deck` inherits from `Sequence`, it automatically supports `in`, iteration with `for`, `index()`, `count()`, and `reversed()` — all derived from just `__getitem__` and `__len__`.

```python
for card in deck[:2]:
    print(card)
print(deck.count((1, "Hearts")))
print(deck.index((5, "Clubs")))
```

**Output:**
```
(1, 'Hearts')
(2, 'Hearts')
1
30
```

### Adding Mutability

```python
from collections.abc import MutableSequence

class BoundedList(MutableSequence):
    def __init__(self, max_size):
        self._data = []
        self.max_size = max_size

    def __getitem__(self, index):
        return self._data[index]

    def __setitem__(self, index, value):
        self._data[index] = value

    def __delitem__(self, index):
        del self._data[index]

    def __len__(self):
        return len(self._data)

    def insert(self, index, value):
        if len(self._data) >= self.max_size:
            raise OverflowError("BoundedList is full")
        self._data.insert(index, value)


bl = BoundedList(max_size=3)
bl.append(1)
bl.append(2)
bl.append(3)
print(list(bl))

try:
    bl.append(4)
except OverflowError as e:
    print("Error:", e)
```

**Output:**
```
[1, 2, 3]
Error: BoundedList is full
```

`BoundedList` automatically inherits `append()`, `extend()`, `pop()`, `remove()`, `reverse()`, and `+=` from `MutableSequence`, just by implementing five core methods.

---

## 15. Sequences vs Mappings vs Sets

| Feature | Sequence (`list`) | Mapping (`dict`) | Set (`set`) |
|---|---|---|---|
| Access by | Integer index (0-based) | Key | N/A (membership only) |
| Order | Positional, defined | Insertion order | Unordered (logically) |
| Duplicates | Allowed | Keys unique | Elements unique |
| Slicing | Yes | No | No |
| Core protocol | `__getitem__(int)`, `__len__` | `__getitem__(key)`, `__iter__`, `__len__` | `__contains__`, set ops |
| Typical use | Ordered collections, records | Labeled lookups | Uniqueness/membership |

---

## 16. Performance Notes

| Operation | `list` | `tuple` | `str` |
|---|---|---|---|
| Indexing `s[i]` | O(1) | O(1) | O(1) |
| Slicing `s[a:b]` | O(k) (k = slice length) | O(k) | O(k) |
| `in` (membership) | O(n) | O(n) | O(n) |
| Append (`list.append`) | Amortized O(1) | N/A (immutable) | N/A (immutable) |
| Insert at front | O(n) | N/A | N/A |
| Concatenation `+` | O(n) (new object) | O(n) | O(n) |
| `len(s)` | O(1) | O(1) | O(1) |

> **Tip:** Repeatedly concatenating strings or tuples in a loop (`result += item`) is O(n²) overall because each `+` creates a new object. For strings, prefer building a list and joining once with `"".join(parts)`.

```python
# Inefficient
result = ""
for word in ["a", "b", "c", "d"]:
    result += word

# Efficient
parts = ["a", "b", "c", "d"]
result = "".join(parts)
print(result)
```

**Output:**
```
abcd
```

---

## 17. Type Hints for Sequences

```python
from typing import Sequence, MutableSequence, List

def total(numbers: Sequence[int]) -> int:
    return sum(numbers)

def append_zero(numbers: MutableSequence[int]) -> None:
    numbers.append(0)

def make_numbers() -> List[int]:
    return [1, 2, 3]
```

- Use `Sequence[T]` for parameters that only need **read** access (accepts `list`, `tuple`, custom sequences) — this is more permissive than `List[T]`.
- Use `MutableSequence[T]` when the function needs to **mutate in place** (e.g., `append`, item assignment).
- Use `List[T]` (or `list[T]` in 3.9+) only when a real `list` is specifically required.

```python
total([1, 2, 3])         # works
total((1, 2, 3))          # also works - tuple satisfies Sequence
```

---

## 18. Best Practices

- Prefer **`tuple`** for fixed, record-like data (coordinates, RGB values, function return groups) — immutability communicates intent and allows hashing.
- Prefer **`list`** for collections that grow, shrink, or get reordered.
- Use **slicing** instead of manual loops for extracting sub-ranges — it's more concise and implemented in C.
- Use **`Sequence`** (not `List`) in type hints for read-only parameters to accept the widest range of valid inputs.
- Use **unpacking** (`a, b, *rest = seq`) instead of manual indexing when extracting multiple elements.
- Use **`"".join(...)`** instead of repeated `+=` concatenation for strings, and `list.extend()` instead of repeated `+` for lists in loops.
- When designing a custom sequence, subclass `collections.abc.Sequence` / `MutableSequence` instead of reimplementing the full interface.

---

## 19. Common Mistakes

### Mistake 1: Mutable Default Argument as a Sequence

```python
# WRONG
def add_item(item, items=[]):
    items.append(item)
    return items

print(add_item("a"))
print(add_item("b"))   # unexpectedly contains 'a' too
```

**Output:**
```
['a']
['a', 'b']
```

**Fix:**
```python
def add_item(item, items=None):
    if items is None:
        items = []
    items.append(item)
    return items
```

### Mistake 2: Confusing Shallow Copy with Deep Copy

```python
matrix = [[0, 0, 0]] * 3      # WRONG: all 3 rows are the SAME list
matrix[0][0] = 1
print(matrix)
```

**Output:**
```
[[1, 0, 0], [1, 0, 0], [1, 0, 0]]
```

**Fix:**
```python
matrix = [[0, 0, 0] for _ in range(3)]
matrix[0][0] = 1
print(matrix)
```

**Output:**
```
[[1, 0, 0], [0, 0, 0], [0, 0, 0]]
```

### Mistake 3: Modifying a List While Iterating Over It

```python
nums = [1, 2, 3, 4, 5]
for n in nums:
    if n % 2 == 0:
        nums.remove(n)   # skips elements due to shifting indices
print(nums)
```

**Output:**
```
[1, 3, 4, 5]
```

**Fix:** Iterate over a copy, or build a new list.
```python
nums = [1, 2, 3, 4, 5]
nums = [n for n in nums if n % 2 != 0]
print(nums)
```

**Output:**
```
[1, 3, 5]
```

### Mistake 4: Assuming `tuple == list` with Equal Elements

```python
print((1, 2, 3) == [1, 2, 3])   # False - different types
```

**Output:**
```
False
```

### Mistake 5: Off-by-One Errors in Slicing

```python
s = [0, 1, 2, 3, 4]
print(s[1:3])    # NOT [1,2,3] - stop index is exclusive
```

**Output:**
```
[1, 2]
```

---

## 20. Interview Questions

1. **What is the difference between a sequence and a list?**
   A sequence is the general abstract concept/protocol (ordered, index-accessible collection); `list` is one concrete, mutable implementation. `tuple`, `str`, and `range` are also sequences.
2. **What two methods must an object implement to satisfy `collections.abc.Sequence`?**
   `__getitem__` and `__len__`.
3. **What extra methods does `MutableSequence` require beyond `Sequence`?**
   `__setitem__`, `__delitem__`, and `insert()`.
4. **What is the key difference between a `list` and a `tuple`?**
   Lists are mutable; tuples are immutable, and therefore hashable (if their contents are hashable) and usable as dict keys/set elements.
5. **Why does slicing never raise an `IndexError`, while indexing can?**
   Slicing computes a valid sub-range by clamping start/stop to the sequence bounds, while indexing requires an exact, in-bounds position.
6. **What's the difference between a sequence and an iterable?**
   All sequences are iterable, but not all iterables are sequences — an iterable only needs `__iter__`, while a sequence additionally guarantees ordered, integer-indexed access and `__len__` (e.g., a generator is iterable but not a sequence).
7. **Why is `[[0]*3]*3` a common bug when creating a 2D list?**
   It creates three references to the **same** inner list, so mutating one row affects all rows; use a list comprehension instead.
8. **How would you reverse a sequence without mutating the original?**
   `reversed(seq)` (returns an iterator) or slicing `seq[::-1]` (returns a new sequence).
9. **Why is `"".join(list_of_strings)` preferred over repeated `+=` concatenation in a loop?**
   Each `+=` on immutable strings creates a new string object, making repeated concatenation O(n²); `join()` builds the result once.
10. **Why does `(1, 2) == [1, 2]` evaluate to `False`?**
    Sequence equality in Python also requires matching types — a tuple and a list, even with identical elements, are never `==`.

---

## 21. Quick Reference

```python
# Built-in sequence types
list, tuple, str, bytes, bytearray, range

# Indexing
s[0]        # first
s[-1]       # last

# Slicing
s[a:b]      # items a..b-1
s[a:b:c]    # with step
s[::-1]     # reversed

# Common ops
len(s)
x in s
s + s2          # concatenation (same type)
s * n           # repetition
s.index(x)
s.count(x)
sorted(s)
reversed(s)

# Unpacking
a, b, c = seq
first, *rest = seq
*init, last = seq

# Copying
s.copy()
s[:]
list(s)
import copy; copy.deepcopy(s)

# ABCs
from collections.abc import Sequence, MutableSequence
isinstance([], Sequence)         # True
isinstance([], MutableSequence)  # True
isinstance((), MutableSequence)  # False

# Type hints
from typing import Sequence, MutableSequence, List
def f(s: Sequence[int]) -> int: ...
def g(s: MutableSequence[int]) -> None: ...
```

---

## Summary

A **sequence** is the abstract concept of an **ordered, integer-indexed collection** — `list` is its primary mutable implementation, while `tuple`, `str`, `bytes`, and `range` are immutable implementations. The `collections.abc` module formalizes the contract with `Sequence` (read-only: `__getitem__`, `__len__`) and `MutableSequence` (read-write: adds `__setitem__`, `__delitem__`, `insert`), letting you build fully-featured custom sequence types from just a couple of core methods. Mastering indexing, slicing, unpacking, and the mutable/immutable distinction is essential — sequences are the backbone of ordered data handling throughout Python, from simple lists to strings, byte buffers, and custom record types.
