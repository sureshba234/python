# The Complete Guide to Python `range()`

A beginner-to-advanced reference for one of Python's most fundamental built-in types.

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Syntax](#2-syntax)
3. [Creating Range Objects](#3-creating-range-objects)
4. [Accessing Elements](#4-accessing-elements)
5. [Slicing Ranges](#5-slicing-ranges)
6. [Iterating Through Ranges](#6-iterating-through-ranges)
7. [Step Parameter](#7-step-parameter)
8. [Reverse Ranges](#8-reverse-ranges)
9. [Membership Testing](#9-membership-testing)
10. [Attributes and Methods](#10-attributes-and-methods)
11. [Built-in Functions with Range](#11-built-in-functions-with-range)
12. [Converting Range](#12-converting-range)
13. [Range vs List](#13-range-vs-list)
14. [Time Complexity](#14-time-complexity)
15. [Practical Applications](#15-practical-applications)
16. [Common Mistakes](#16-common-mistakes)
17. [Interview Questions](#17-interview-questions)
18. [Coding Exercises](#18-coding-exercises)
19. [Best Practices](#19-best-practices)
20. [Cheat Sheet](#20-cheat-sheet)
21. [Summary](#21-summary)

---

## 1. Introduction

### Definition

`range()` is a **built-in immutable sequence type** in Python that represents an arithmetic progression of integers. It does not store all its values in memory — instead, it generates each value on demand using its `start`, `stop`, and `step` values.

### Syntax

```python
range(stop)
range(start, stop)
range(start, stop, step)
```

### Explanation

* `range()` produces a sequence of numbers, commonly used to control the number of times a loop runs.
* It behaves like a sequence (supports indexing, slicing, `len()`, `in`), but it is **not a list** — it's a distinct, memory-efficient type.
* Values are computed lazily — only when requested — rather than being precomputed and stored.

### Why `range()` Is Used

* To repeat an action a specific number of times.
* To generate a sequence of numbers without manually writing them out.
* To iterate over indexes of another sequence (e.g., a list).
* To produce numeric sequences for calculations, simulations, and pattern generation.

### Advantages of `range()`

| Advantage | Description |
|---|---|
| Memory efficient | Stores only `start`, `stop`, `step` — not every number |
| Fast creation | Creating a `range` is O(1) regardless of size |
| Immutable | Cannot be accidentally modified |
| Sequence behavior | Supports indexing, slicing, `len()`, membership testing |
| Reusable | Can be iterated over multiple times |

### Common Use Cases

* `for` loop counters
* Generating index sequences
* Building lists/sets/tuples of numbers
* Pagination calculations
* Mathematical and simulation tasks

### Examples

```python
# Loop 5 times
for i in range(5):
    print(i)
# Output: 0 1 2 3 4

# Generate numbers 1 to 10
numbers = list(range(1, 11))
print(numbers)
# Output: [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

# Even numbers from 0 to 20
evens = list(range(0, 21, 2))
print(evens)
```

### Common Pitfalls

* Assuming `range()` is a list (it's a distinct type — use `list(range(...))` to convert).
* Forgetting that `stop` is **exclusive**.

### Best Practices

* Prefer `range()` over manually building lists of numbers for loops — it's clearer and more efficient.
* Only convert to `list()`/`tuple()`/`set()` when you actually need a concrete collection.

---

## 2. Syntax

### Definition

`range()` accepts one, two, or three integer arguments that define the sequence's boundaries and step size.

### Syntax

```python
range(stop)
range(start, stop)
range(start, stop, step)
```

### Explanation of Parameters

| Parameter | Required? | Default | Description |
|---|---|---|---|
| `start` | Optional | `0` | The first value in the sequence (inclusive) |
| `stop` | Required | — | The value where the sequence ends (**exclusive**) |
| `step` | Optional | `1` | The difference between consecutive values |

**Rules:**

* All arguments must be integers (or objects implementing `__index__`).
* `step` cannot be `0` — this raises a `ValueError`.
* If `step` is negative, the sequence counts down, and `start` must be greater than `stop` to produce any values.
* If `start >= stop` with a positive step (or `start <= stop` with a negative step), the range is **empty**, not an error.

### Examples

```python
range(5)            # 0, 1, 2, 3, 4
range(2, 8)          # 2, 3, 4, 5, 6, 7
range(0, 10, 3)      # 0, 3, 6, 9
range(10, 0, -2)     # 10, 8, 6, 4, 2
range(5, 5)          # empty range
range(5, 1)          # empty range (positive step, start > stop)
```

```python
range(5, 1, -1)  # 5, 4, 3, 2  (valid: negative step with start > stop)
```

### Common Pitfalls

* Forgetting `stop` is exclusive: `range(1, 5)` gives `1, 2, 3, 4`, not `1, 2, 3, 4, 5`.
* Using `step=0`, which raises:

```python
range(0, 10, 0)
# ValueError: range() arg 3 must not be zero
```

### Best Practices

* Use the single-argument form `range(n)` only for simple "loop n times" cases.
* Use the three-argument form when the step is anything other than `1`.
* Name variables clearly when boundaries aren't obvious (e.g., `start_year`, `end_year`).

---

## 3. Creating Range Objects

### Definition

Creating a range object means calling `range()` with the desired arguments. The result is a `range` object — not a list of numbers.

### Syntax

```python
range(stop)
range(start, stop)
range(start, stop, step)
```

### Explanation

When you call `range()`, Python immediately returns a lightweight `range` object. No numbers are generated until you iterate, index, or convert it.

### Examples

```python
r1 = range(5)
print(r1)             # range(0, 5)
print(type(r1))       # <class 'range'>

r2 = range(1, 10)
print(r2)             # range(1, 10)

r3 = range(1, 20, 2)
print(r3)             # range(1, 20, 2)
print(list(r3))       # [1, 3, 5, 7, 9, 11, 13, 15, 17, 19]
```

```python
# Empty ranges are valid objects, not errors
empty = range(5, 5)
print(list(empty))    # []
print(len(empty))     # 0
```

### Common Pitfalls

* Expecting `print(range(5))` to show `[0, 1, 2, 3, 4]` — it shows `range(0, 5)` instead.
* Treating an empty range as an error condition instead of handling it gracefully.

### Best Practices

* Don't convert to `list()` just to "see" the values during debugging in production code — use it only when needed, or inspect with `list(r)` temporarily while debugging.
* Store a `range` in a variable if you plan to reuse the same sequence multiple times — it's cheap to keep around.

---

## 4. Accessing Elements

### Definition

Although `range` objects don't store their values, they support **indexing** just like lists or tuples, computing the requested value mathematically.

### Syntax

```python
r[index]
r[-index]
```

### Explanation

* **Positive indexing** starts at `0` for the first element.
* **Negative indexing** starts at `-1` for the last element.
* Indexing out of bounds raises an `IndexError`.

### Examples

```python
r = range(10, 20)

print(r[0])    # 10  (first element)
print(r[5])    # 15
print(r[-1])   # 19  (last element)
print(r[-2])   # 18  (second-to-last)
```

```python
r = range(0, 20, 5)   # 0, 5, 10, 15
print(r[2])     # 10
print(r[-1])    # 15
```

```python
r = range(5)
print(r[10])
# IndexError: range object index out of range
```

### Common Pitfalls

* Assuming indexing requires generating the whole sequence first — it doesn't; it's computed directly from `start`, `stop`, `step`.
* Forgetting that index errors still apply, just like with lists.

### Best Practices

* Use negative indexing to cleanly access the last element(s) without computing `len(r) - 1`.
* Validate index bounds when accepting user-supplied indices.

---

## 5. Slicing Ranges

### Definition

Slicing a `range` returns a **new `range` object** representing a sub-sequence — it does not return a list.

### Syntax

```python
r[start:stop]
r[start:stop:step]
```

### Explanation

Slicing follows the same rules as list slicing (exclusive stop, optional step, supports negative indices), but the result is always another `range`, computed without iterating.

### Examples

```python
r = range(0, 20, 2)   # 0,2,4,6,8,10,12,14,16,18

print(r[2:5])          # range(4, 10, 2)
print(list(r[2:5]))    # [4, 6, 8]

print(r[:4])           # range(0, 8, 2)
print(list(r[:4]))     # [0, 2, 4, 6]

print(r[::-1])         # range(18, -2, -2)
print(list(r[::-1]))   # [18, 16, 14, ..., 0]
```

```python
r = range(1, 11)
print(list(r[2:8:2]))  # [3, 5, 7]
```

### Common Pitfalls

* Expecting a slice of a `range` to return a `list` — it returns another `range`.
* Forgetting that reversing via `r[::-1]` produces a range with a negative step and adjusted bounds.

### Best Practices

* Take advantage of range slicing to create derived ranges without converting to a list first — this keeps the memory benefits.
* Use `reversed(r)` instead of `r[::-1]` when you just want to *iterate* in reverse (slightly clearer intent).

---

## 6. Iterating Through Ranges

### Definition

Iteration walks through every value a `range` represents, generating each value one at a time.

### Syntax

```python
for i in range(...):
    ...

while condition:
    ...

for i, value in enumerate(range(...)):
    ...

for a, b in zip(range(...), range(...)):
    ...
```

### Explanation

* **`for` loop**: the idiomatic, preferred way to iterate over a range.
* **`while` loop**: a manual alternative requiring explicit counter management — more error-prone.
* **`enumerate()`**: pairs each value with its position; useful when range values represent something else (less common, but works).
* **`zip()`**: pairs values from a range with values from another iterable — useful for parallel iteration.

### Examples

**`for` loop:**

```python
for i in range(5):
    print(f"Iteration {i}")
```

**`while` loop (manual equivalent):**

```python
i = 0
while i < 5:
    print(f"Iteration {i}")
    i += 1
```

**`enumerate()` with range:**

```python
for index, value in enumerate(range(10, 15)):
    print(index, value)
# 0 10
# 1 11
# 2 12
# 3 13
# 4 14
```

**`zip()` with range:**

```python
names = ["Alice", "Bob", "Charlie"]
for num, name in zip(range(1, 4), names):
    print(num, name)
# 1 Alice
# 2 Bob
# 3 Charlie
```

```python
# Using range to index two lists in parallel
prices = [10, 20, 30]
quantities = [2, 3, 1]
total = 0
for i in range(len(prices)):
    total += prices[i] * quantities[i]
print(total)  # 110
```

### Common Pitfalls

* Using `while` loops for simple counting tasks — more verbose and bug-prone (off-by-one errors, forgetting to increment).
* Using `range(len(my_list))` plus indexing when a direct `for item in my_list` would be simpler (when the index itself isn't needed).
* Modifying the loop variable inside a `for...in range()` loop, expecting it to affect iteration — it doesn't, since `range` values are generated independently each step.

### Best Practices

* Prefer `for i in range(n)` over manual `while` loops for fixed-count iteration.
* Use `enumerate(sequence)` instead of `range(len(sequence))` when you need both index and value of an existing sequence.
* Use `zip()` for parallel iteration instead of indexing multiple sequences manually.

---

## 7. Step Parameter

### Definition

The `step` parameter controls the interval between consecutive values in the sequence.

### Syntax

```python
range(start, stop, step)
```

### Explanation

* **Positive step**: counts upward (`start` must be less than `stop` to get values).
* **Negative step**: counts downward (`start` must be greater than `stop` to get values).
* **`step` omitted**: defaults to `1`.
* **`step = 0`**: invalid, raises `ValueError`.

### Examples

**Positive step:**

```python
print(list(range(0, 10, 1)))   # [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
print(list(range(0, 10, 3)))   # [0, 3, 6, 9]
```

**Negative step (reverse counting):**

```python
print(list(range(10, 0, -1)))   # [10, 9, 8, ..., 1]
print(list(range(20, 0, -5)))   # [20, 15, 10, 5]
```

**Larger steps for skipping values:**

```python
print(list(range(0, 100, 25)))  # [0, 25, 50, 75]
```

### Common Pitfalls

* Using a positive step with `start > stop` — produces an **empty** range (not an error, but easy to overlook):

```python
print(list(range(10, 0, 1)))   # [] — likely a bug!
```

* Using a negative step with `start < stop` — also produces an empty range:

```python
print(list(range(0, 10, -1)))  # [] — likely a bug!
```

### Best Practices

* When counting down, double-check that `step` is negative AND `start > stop`.
* Add a comment when the step is non-obvious (e.g., `range(0, 100, 7)  # every 7th value`).

---

## 8. Reverse Ranges

### Definition

A reverse range counts downward from a higher number to a lower one, achieved with a negative `step`.

### Syntax

```python
range(start, stop, -step)
```

### Explanation

To count down, `start` must be greater than `stop`, and `step` must be negative. Alternatively, `reversed()` can reverse an existing ascending range.

### Examples

```python
# Countdown from 10 to 1
for i in range(10, 0, -1):
    print(i)
# 10, 9, 8, ..., 1
```

```python
# Reverse an existing range using reversed()
r = range(1, 11)
for i in reversed(r):
    print(i)
# 10, 9, ..., 1
```

```python
# Reverse with a custom step
print(list(range(20, 0, -4)))   # [20, 16, 12, 8, 4]
```

### Applications

* Countdown timers
* Displaying data in reverse chronological order (using indices)
* Reverse iteration over a list's indices:

```python
data = ["a", "b", "c", "d"]
for i in range(len(data) - 1, -1, -1):
    print(data[i])
# d, c, b, a
```

* Generating descending number sequences for reports or simulations.

### Common Pitfalls

* Forgetting that `stop` is exclusive even when counting down — `range(10, 0, -1)` stops at `1`, not `0`. To include `0`, use `range(10, -1, -1)`.
* Mixing up `reversed(range(...))` with manually building a negative-step range — both work, but readability differs.

### Best Practices

* Prefer `reversed(range(n))` when reversing a "natural" ascending range improves readability.
* Use explicit negative-step ranges (`range(n, -1, -1)`) when the starting point isn't `0`-based.

---

## 9. Membership Testing

### Definition

Membership testing checks whether a value exists within a range's sequence, using the `in` and `not in` operators.

### Syntax

```python
value in range(...)
value not in range(...)
```

### Explanation

Python checks membership in a `range` **mathematically** (using `start`, `stop`, `step`) rather than scanning every element, making it very fast regardless of the range's size.

### Examples

```python
print(5 in range(10))         # True
print(15 not in range(10))    # True
print(15 in range(10))        # False
```

```python
r = range(0, 100, 5)
print(50 in r)    # True  (50 is a multiple of 5 within bounds)
print(51 in r)    # False (not a multiple of step)
```

```python
print(-1 in range(0, 10))     # False
print(0 in range(0, 10))      # True
print(10 in range(0, 10))     # False (stop is exclusive)
```

### Common Pitfalls

* Assuming all numbers "within bounds" are members — step matters:

```python
print(7 in range(0, 10, 2))  # False, 7 isn't reachable via step 2
```

* Forgetting `stop` is excluded from membership.

### Best Practices

* Use `in range(...)` for fast bounds/membership checks instead of manual comparisons when step matters.
* For simple "is this between two numbers" checks without a step, a direct comparison (`low <= x < high`) can be more readable than constructing a range.
## 10. Attributes and Methods

### Definition

`range` objects expose three read-only attributes (`start`, `stop`, `step`) and two methods (`count()`, `index()`) inherited from the general sequence protocol.

### Syntax

```python
r.start
r.stop
r.step

r.count(value)
r.index(value)
```

### Explanation

| Member | Description |
|---|---|
| `start` | The first value used to define the range (default `0`) |
| `stop` | The exclusive upper/lower bound |
| `step` | The increment between values (default `1`) |
| `count(value)` | Returns how many times `value` appears (`0` or `1`, since ranges have no duplicates) |
| `index(value)` | Returns the position of `value`; raises `ValueError` if not found |

### Examples

```python
r = range(2, 20, 3)

print(r.start)   # 2
print(r.stop)    # 20
print(r.step)    # 3
```

```python
r = range(0, 10, 2)
print(r.count(4))    # 1
print(r.count(5))    # 0 (5 isn't in the sequence)
```

```python
r = range(10, 20)
print(r.index(15))   # 5  (15 is the 6th value, index 5)

print(r.index(100))
# ValueError: 100 is not in range
```

### Common Pitfalls

* Trying to assign to `start`/`stop`/`step` — they are read-only:

```python
r = range(5)
r.start = 1
# AttributeError: readonly attribute
```

* Calling `index()` on a value not present, without handling the `ValueError`.

### Best Practices

* Use `r.start`, `r.stop`, `r.step` to introspect a range passed into a function instead of re-deriving bounds manually.
* Guard `index()` calls with `if value in r:` first, or wrap in `try/except` if the value might be missing.

---

## 11. Built-in Functions with Range

### Definition

Because `range` implements the sequence protocol, many built-in functions work with it directly — often without ever converting it to a list.

### Syntax

```python
len(r)
min(r)
max(r)
sum(r)
list(r)
tuple(r)
set(r)
sorted(r)
```

### Explanation

| Function | Behavior on a `range` |
|---|---|
| `len(r)` | Number of elements, computed mathematically |
| `min(r)` | Smallest value (depends on step direction) |
| `max(r)` | Largest value |
| `sum(r)` | Sum of all elements |
| `list(r)` | Converts to a `list` |
| `tuple(r)` | Converts to a `tuple` |
| `set(r)` | Converts to a `set` |
| `sorted(r)` | Returns a sorted **list** (ranges are already ordered, but this always returns ascending order) |

### Examples

```python
r = range(1, 11)

print(len(r))      # 10
print(min(r))       # 1
print(max(r))       # 10
print(sum(r))       # 55
print(list(r))      # [1, 2, ..., 10]
print(tuple(r))     # (1, 2, ..., 10)
print(set(r))       # {1, 2, ..., 10}
```

```python
r = range(10, 0, -2)   # 10, 8, 6, 4, 2
print(min(r))    # 2
print(max(r))    # 10
print(sorted(r)) # [2, 4, 6, 8, 10]  (always ascending, regardless of step)
```

```python
empty = range(0)
print(len(empty))   # 0
print(sum(empty))   # 0
print(min(empty))
# ValueError: min() arg is an empty sequence
```

### Common Pitfalls

* Calling `min()`/`max()` on an empty range without a default — raises `ValueError`. Use `min(r, default=None)`.
* Assuming `sorted(r)` changes anything for an ascending range — it doesn't; it only matters for descending (negative-step) ranges.

### Best Practices

* Use `sum(range(...))` instead of manual loops for arithmetic series — it's both faster and clearer.
* Provide a `default` to `min()`/`max()` when the range might be empty:

```python
print(min(range(0), default=0))   # 0
```

---

## 12. Converting Range

### Definition

Converting transforms a `range` object into a concrete collection type — useful when you need mutability, duplication-checks via hashing (set), or to pass data to APIs expecting a list/tuple.

### Syntax

```python
list(range(...))
tuple(range(...))
set(range(...))
```

### Explanation

| Conversion | Result Properties |
|---|---|
| `list(range(...))` | Ordered, mutable, allows duplicates (though a range itself has none) |
| `tuple(range(...))` | Ordered, immutable |
| `set(range(...))` | Unordered, no duplicates, fast membership testing |

Converting **materializes every value in memory** — losing the memory-efficiency benefit of `range`. Only convert when necessary.

### Examples

```python
print(list(range(5)))     # [0, 1, 2, 3, 4]
print(tuple(range(5)))    # (0, 1, 2, 3, 4)
print(set(range(5)))      # {0, 1, 2, 3, 4}
```

```python
# Practical use: build a lookup set for fast membership checks
valid_ids = set(range(1000, 2000))
print(1500 in valid_ids)   # True — O(1) set lookup
```

```python
# Mutable list from a range, then modify it
nums = list(range(5))
nums.append(100)
print(nums)   # [0, 1, 2, 3, 4, 100]
```

### Common Pitfalls

* Converting a huge range to a list unnecessarily, e.g. `list(range(10_000_000))` just to iterate over it — wastes memory; iterate the `range` directly instead.
* Forgetting that converting to a `set` loses order.

### Best Practices

* Iterate directly over `range` objects in loops — avoid converting unless you need list/tuple/set-specific behavior (mutation, hashing, etc.).
* Convert to `set` only when you need fast repeated membership testing on the same values.

---

## 13. Range vs List

### Definition

A `range` is a compact, immutable representation of an arithmetic sequence; a `list` is a general-purpose, mutable, heterogeneous container that stores every element explicitly.

### Comparison Table

| Aspect | `range` | `list` |
|---|---|---|
| **Memory usage** | Minimal — stores only `start`, `stop`, `step` | Stores every element individually |
| **Performance (creation)** | O(1), instant regardless of size | O(n) — must build every element |
| **Mutability** | Immutable | Mutable (append, remove, sort, etc.) |
| **Element types** | Integers only | Any type, mixed types allowed |
| **Indexing/Slicing** | Supported, computed mathematically | Supported, stored directly |
| **Membership testing** | O(1) (mathematical check) | O(n) (linear scan) |
| **Use case** | Loop counters, numeric sequences, index generation | General data storage, heterogeneous collections, frequent mutation |
| **Reusability** | Reusable; same object can be iterated repeatedly | Reusable; iteration doesn't consume it either |

### Explanation

* Use `range` when you need a sequence of evenly spaced integers and don't need to store unrelated data or mutate the sequence.
* Use `list` when you need to store arbitrary data, modify elements, or build something other than a strict arithmetic progression.

### Examples

```python
import sys

r = range(1_000_000)
l = list(range(1_000_000))

print(sys.getsizeof(r))   # tiny, constant size (e.g., 48 bytes)
print(sys.getsizeof(l))   # large, grows with element count
```

```python
# range cannot be mutated
r = range(5)
r.append(6)
# AttributeError: 'range' object has no attribute 'append'

# list can
l = list(range(5))
l.append(6)
print(l)   # [0, 1, 2, 3, 4, 6]
```

### Common Pitfalls

* Defaulting to `list(range(n))` out of habit when a plain `range(n)` would suffice for iteration.
* Assuming a `range` can hold floats or strings — it only holds integers.

### Best Practices

* Default to `range` for loop control and numeric generation; convert to `list` only when mutability or non-integer mixing is required.
* When working with very large sequences, always prefer `range` for memory efficiency.

---

## 14. Time Complexity

### Definition

Time complexity describes how an operation's cost scales with the size of the range, expressed in Big-O notation.

### Complexity Table

| Operation | Complexity | Notes |
|---|---|---|
| Creation (`range(...)`) | O(1) | Only stores `start`, `stop`, `step` |
| Indexing (`r[i]`) | O(1) | Computed directly via arithmetic |
| Slicing (`r[a:b:c]`) | O(1) | Returns a new range, computed directly |
| Membership testing (`x in r`) | O(1) | Computed via bounds/step check, not a scan |
| Iteration (full loop) | O(n) | Must produce each of the `n` values once |
| Length calculation (`len(r)`) | O(1) | Computed mathematically from `start`, `stop`, `step` |
| `min()` / `max()` | O(1) | Determined from bounds and step direction |
| `count(value)` | O(1) | Checked via membership logic |
| `index(value)` | O(1) | Computed directly |
| `list(r)` / `tuple(r)` / `set(r)` | O(n) | Must materialize every element |

### Explanation

This is the key distinguishing feature of `range` compared to a `list`: nearly every operation that would be O(n) on a list (indexing aside) is **O(1)** on a range, because the value is computed from a formula rather than retrieved from stored data.

### Examples

```python
import time

big_range = range(10**12)   # a trillion elements

start = time.perf_counter()
print(999_999_999_999 in big_range)
print(time.perf_counter() - start)
# Near-instant — O(1), no scanning of a trillion elements
```

### Common Pitfalls

* Assuming operations on `range` scale linearly like they would on an equivalent `list` — most don't.
* Forgetting that iteration (and any conversion) is still O(n), even for a `range`.

### Best Practices

* Leverage `range`'s O(1) membership/indexing/length operations for performance-critical bounds checks instead of converting to a list first.
* Remember that full iteration is always O(n) — there's no way around visiting every value if you truly need them all.

---

## 15. Practical Applications

### Definition

Beyond simple loops, `range()` underlies many everyday programming patterns.

### 15.1 Counting Loops

```python
total = 0
for i in range(1, 101):
    total += i
print(total)   # 5050
```

### 15.2 Number Generation

```python
# Generate first 10 square numbers
squares = [i ** 2 for i in range(1, 11)]
print(squares)
# [1, 4, 9, 16, 25, 36, 49, 64, 81, 100]
```

### 15.3 Pagination

```python
def get_page_bounds(page_number, page_size):
    start = (page_number - 1) * page_size
    stop = start + page_size
    return range(start, stop)

page_range = get_page_bounds(page_number=3, page_size=10)
print(page_range)            # range(20, 30)
print(list(page_range))      # [20, 21, ..., 29]
```

### 15.4 Data Processing (chunking)

```python
def chunk_indices(total_items, chunk_size):
    return [range(i, min(i + chunk_size, total_items))
            for i in range(0, total_items, chunk_size)]

for chunk in chunk_indices(23, 10):
    print(list(chunk))
# [0..9], [10..19], [20, 21, 22]
```

### 15.5 Pattern Programs

```python
# Right-angled triangle pattern
for i in range(1, 6):
    print("*" * i)
# *
# **
# ***
# ****
# *****
```

```python
# Number pyramid using nested ranges
n = 5
for i in range(1, n + 1):
    print(" " * (n - i) + " ".join(str(j) for j in range(1, i + 1)))
```

### 15.6 Simulations

```python
import random

# Simulate 10 dice rolls
rolls = [random.randint(1, 6) for _ in range(10)]
print(rolls)
```

```python
# Simple discrete-time simulation: population growth over 5 steps
population = 100
for year in range(1, 6):
    population = int(population * 1.05)
    print(f"Year {year}: {population}")
```

### Common Pitfalls

* Re-implementing pagination/chunking logic with manual index math instead of `range`'s built-in start/stop/step semantics.
* Using nested loops with ranges when a list comprehension would be clearer for simple generation tasks.

### Best Practices

* Encapsulate `range`-based logic (like pagination) into small, well-named helper functions for reuse and clarity.
* Use list/generator comprehensions with `range` for concise number generation instead of manual `append` loops.
## 16. Common Mistakes

### Mistake 1: Assuming `range()` produces a list

**Problem:**
```python
r = range(5)
print(r)   # range(0, 5), not [0, 1, 2, 3, 4]
```
**Explanation:** `range` is its own type with its own `repr`; it never auto-converts.
**Solution:** Use `list(range(5))` if you actually need a list.

---

### Mistake 2: Forgetting `stop` is exclusive

**Problem:**
```python
for i in range(1, 5):
    print(i)   # 1, 2, 3, 4 — not 5!
```
**Explanation:** The `stop` value is never included in the sequence.
**Solution:** Use `range(1, 6)` to include `5`, or always add 1 to `stop` when an inclusive upper bound is intended.

---

### Mistake 3: Using `step=0`

**Problem:**
```python
range(0, 10, 0)
# ValueError: range() arg 3 must not be zero
```
**Explanation:** A step of `0` would create an infinite, non-progressing sequence, so Python disallows it.
**Solution:** Always use a non-zero integer for `step`.

---

### Mistake 4: Wrong step direction (empty range bug)

**Problem:**
```python
print(list(range(10, 0, 1)))   # [] — likely unintended!
```
**Explanation:** With a positive step, `start` must be less than `stop`; otherwise the range is silently empty (no error raised).
**Solution:** Use a negative step for descending sequences: `range(10, 0, -1)`.

---

### Mistake 5: Trying to mutate a range

**Problem:**
```python
r = range(5)
r[0] = 10
# TypeError: 'range' object does not support item assignment
```
**Explanation:** `range` is immutable.
**Solution:** Convert to a list first: `l = list(range(5)); l[0] = 10`.

---

### Mistake 6: Using floats in `range()`

**Problem:**
```python
range(0, 1, 0.1)
# TypeError: 'float' object cannot be interpreted as an integer
```
**Explanation:** `range()` only supports integers.
**Solution:** Use `numpy.arange()`, a manual loop with float increments, or scale integers and divide:
```python
values = [i / 10 for i in range(0, 10)]   # 0.0, 0.1, ..., 0.9
```

---

### Mistake 7: Confusing `range(n)` with `range(1, n)`

**Problem:**
```python
for i in range(5):   # 0,1,2,3,4 — developer expected 1,2,3,4,5
    print(i)
```
**Explanation:** The single-argument form starts at `0` by default.
**Solution:** Use `range(1, n + 1)` when a 1-based, inclusive-of-`n` sequence is intended.

---

### Mistake 8: Unnecessary list conversion for iteration

**Problem:**
```python
for i in list(range(1_000_000)):
    pass
```
**Explanation:** Converting to a list before iterating wastes memory; iteration works directly on `range`.
**Solution:** Iterate the range directly: `for i in range(1_000_000):`.

---

### Mistake 9: Using `range(len(seq))` when iterating values directly would do

**Problem:**
```python
items = ["a", "b", "c"]
for i in range(len(items)):
    print(items[i])
```
**Explanation:** This adds unnecessary indexing when only the values are needed.
**Solution:**
```python
for item in items:
    print(item)
```
Use `range(len(...))` only when the index itself is needed (and `enumerate()` is usually better even then).

---

### Mistake 10: Forgetting `enumerate()` exists

**Problem:**
```python
items = ["a", "b", "c"]
for i in range(len(items)):
    print(i, items[i])
```
**Explanation:** Manually pairing index and value duplicates what `enumerate()` does more cleanly.
**Solution:**
```python
for i, item in enumerate(items):
    print(i, item)
```

---

### Mistake 11: Calling `min()`/`max()` on an empty range

**Problem:**
```python
min(range(0))
# ValueError: min() arg is an empty sequence
```
**Explanation:** An empty range has no elements to compare.
**Solution:** Supply a default: `min(range(0), default=None)`.

---

### Mistake 12: Assuming negative indices behave unexpectedly

**Problem:**
```python
r = range(5)
print(r[-1])  # Some assume this errors; it works fine: 4
```
**Explanation:** Negative indexing on `range` works just like lists — this is a misconception, not a bug, but it trips up beginners who aren't sure it's supported.
**Solution:** Trust standard negative indexing rules; `range` fully supports them.

---

### Mistake 13: Comparing two ranges by identity instead of equality

**Problem:**
```python
r1 = range(0, 5)
r2 = range(0, 5)
print(r1 is r2)   # False — different objects
```
**Explanation:** `is` checks identity, not equivalence of sequence values.
**Solution:** Use `==` to compare ranges by value:
```python
print(r1 == r2)   # True
```

---

### Mistake 14: Expecting `range` to support non-integer membership cleanly

**Problem:**
```python
print(2.5 in range(0, 5))   # False, but not an error — can be confusing
```
**Explanation:** `range` membership checks work with non-integers but will only return `True` for whole numbers that fit the sequence.
**Solution:** Be explicit about expecting integers; cast or validate input first if needed.

---

### Mistake 15: Misunderstanding `index()` errors

**Problem:**
```python
r = range(0, 10, 2)
r.index(3)
# ValueError: 3 is not in range
```
**Explanation:** `3` isn't a multiple of the step starting from `0`, so it isn't part of the sequence.
**Solution:** Check membership first: `if 3 in r: ...` or handle the exception.

---

### Mistake 16: Using `range()` for non-integer business logic ranges (e.g., currency)

**Problem:**
```python
for cents in range(0, 100, 1):   # trying to represent dollars as cents manually
    dollars = cents / 100
```
**Explanation:** While this technically works, mixing units (cents as the loop variable, dollars derived) can become error-prone and unclear.
**Solution:** Keep the loop variable in the unit you ultimately need, or clearly document the unit conversion.

---

### Mistake 17: Off-by-one errors when looping over array indices

**Problem:**
```python
arr = [10, 20, 30]
for i in range(1, len(arr)):  # skips index 0 unintentionally
    print(arr[i])
```
**Explanation:** Developer intended to loop over all elements but started at `1`.
**Solution:** Use `range(len(arr))` for all indices, or iterate values directly with `for item in arr`.

---

### Mistake 18: Recreating `range` objects unnecessarily inside a loop

**Problem:**
```python
for i in range(1000000):
    for j in range(1000000):  # recreated every outer iteration (cheap, but often misunderstood as expensive)
        pass
```
**Explanation:** Some developers wrongly assume range creation is the expensive part and try to "cache" it — this is usually unnecessary since `range()` creation is O(1).
**Solution:** Don't over-optimize range creation; focus on reducing actual iteration work instead.

---

### Mistake 19: Using `range` where a `while` loop with a dynamic condition is actually needed

**Problem:**
```python
# Trying to loop "until a condition changes", forced into a range
for i in range(1000):
    if some_dynamic_condition():
        break
```
**Explanation:** If the number of iterations isn't truly known in advance, forcing a `range` with an arbitrary large bound is a workaround, not a clean solution.
**Solution:** Use a `while` loop when the stopping condition is dynamic, not a fixed count.

---

### Mistake 20: Assuming `sum(range(...))` and manual loop summation always perform identically

**Problem:**
```python
total = 0
for i in range(1, 1_000_001):
    total += i
```
**Explanation:** This works, but is slower and more verbose than necessary for a simple arithmetic sum.
**Solution:** Use the built-in: `total = sum(range(1, 1_000_001))`, or the closed-form formula for arithmetic series when performance is critical.

---

### Mistake 21 (Bonus): Confusing `range(start, stop)` argument order

**Problem:**
```python
range(10, 1)   # empty! developer wanted 1 to 10
```
**Explanation:** With a default positive step, `start` must come before `stop` numerically.
**Solution:** Use `range(1, 11)` for ascending, or add a negative step if descending was intended: `range(10, 0, -1)`.

---

### Mistake 22 (Bonus): Believing ranges support negative `stop` implicitly meaning "from the end"

**Problem:**
```python
print(list(range(-5)))   # [] — not "last 5 elements" of anything
```
**Explanation:** `range` has no concept of "the end of a sequence" — negative numbers are just negative integers, not relative positions (unlike list slicing).
**Solution:** Don't conflate `range` arguments with list/slice negative-index semantics; they mean different things.

---

## Summary Table of Mistakes

| # | Mistake | Fix |
|---|---|---|
| 1 | Treating range as a list | Convert with `list()` only when needed |
| 2 | Forgetting exclusive `stop` | Add 1 for inclusive behavior |
| 3 | `step=0` | Use non-zero step |
| 4 | Wrong step direction | Match step sign to start/stop direction |
| 5 | Mutating a range | Convert to list first |
| 6 | Floats in range | Use list comprehension or `numpy.arange` |
| 7 | Confusing `range(n)` with 1-based ranges | Use `range(1, n+1)` |
| 8 | Unneeded list conversion | Iterate range directly |
| 9 | Indexing when value suffices | Iterate values directly |
| 10 | Ignoring `enumerate()` | Use `enumerate()` for index+value |
| 11 | `min()/max()` on empty range | Provide a `default` |
| 12 | Doubting negative indexing | Trust standard indexing rules |
| 13 | Using `is` instead of `==` | Use `==` for value comparison |
| 14 | Non-integer membership confusion | Validate/cast inputs |
| 15 | Unhandled `index()` errors | Check membership first |
| 16 | Unit confusion in loop variables | Keep units consistent/clear |
| 17 | Off-by-one start index | Double check loop bounds |
| 18 | Over-optimizing range creation | Focus on real bottlenecks |
| 19 | Forcing dynamic loops into range | Use `while` for dynamic conditions |
| 20 | Manual summation instead of `sum()` | Use built-in `sum()` |
| 21 | Wrong start/stop order | Match order to step direction |
| 22 | Misreading negative `stop` as "from the end" | Remember range ≠ slicing semantics |
## 17. Interview Questions

### Beginner (20)

**1. What is `range()` in Python?**
A built-in function/type that generates an immutable sequence of integers defined by `start`, `stop`, and `step`.

**2. What does `range(5)` produce?**
The sequence `0, 1, 2, 3, 4` — five values starting at 0, since `stop` is exclusive.

**3. Is the `stop` value included in the sequence?**
No. `range(start, stop)` always excludes `stop`.

**4. What is the default value of `start`?**
`0`.

**5. What is the default value of `step`?**
`1`.

**6. Can `step` be `0`?**
No — it raises a `ValueError`.

**7. What type does `range()` return?**
A `range` object (`<class 'range'>`), not a list.

**8. How do you convert a range to a list?**
`list(range(...))`.

**9. How do you get the number of elements in a range?**
`len(r)`.

**10. Does `range()` support negative numbers?**
Yes, both as `start`/`stop` values and as a negative `step`.

**11. How do you create a range from 1 to 10 inclusive?**
`range(1, 11)` — since `stop` is exclusive, use `11` to include `10`.

**12. How do you reverse a range?**
Use `reversed(range(...))`, or build it with a negative step: `range(stop, start, -1)`.

**13. Can you index into a range like a list?**
Yes — e.g., `range(5)[2]` returns `2`.

**14. What happens if you index out of bounds?**
An `IndexError` is raised.

**15. Does `range()` support negative indexing?**
Yes — `range(5)[-1]` returns `4`.

**16. How do you check if a value exists in a range?**
Using the `in` operator: `5 in range(10)`.

**17. What is `range(0)`?**
An empty range with zero elements.

**18. How do you loop 10 times using range?**
```python
for i in range(10):
    ...
```

**19. Is `range` mutable?**
No, it's immutable.

**20. Can a range contain floating-point numbers?**
No — `range()` only accepts integers.

---

### Intermediate (20)

**1. Why is `range()` more memory-efficient than a list of the same numbers?**
It stores only `start`, `stop`, and `step`, and computes each value on demand instead of storing every value individually.

**2. What is the time complexity of checking membership in a range?**
O(1) — Python computes it mathematically instead of scanning.

**3. What is the time complexity of `len(range(...))`?**
O(1) — calculated directly from `start`, `stop`, `step`.

**4. How does slicing a range work, and what does it return?**
Slicing computes a new `start`/`stop`/`step` and returns another `range` object, not a list.

**5. What's the difference between `r[::-1]` and `reversed(r)` for a range `r`?**
Both reverse iteration order; `r[::-1]` returns a new range object with adjusted bounds, while `reversed(r)` returns a reverse iterator. Both work correctly on ranges.

**6. How would you generate all even numbers between 1 and 50?**
`range(2, 51, 2)`.

**7. What happens when `start > stop` with a positive step?**
The range is empty — no error is raised.

**8. How can you safely get `min()` of a range that might be empty?**
Use a default: `min(r, default=None)`.

**9. How do `count()` and `index()` behave on a `range`?**
`count(value)` returns `1` if present, `0` otherwise (no duplicates possible). `index(value)` returns its position or raises `ValueError`.

**10. Are two `range` objects with the same values but different `step` representations equal?**
Equality is based on the resulting sequence of values, not literal attribute equality — Python compares ranges by their effective sequence. For example, `range(0, 3) == range(0, 3, 1)` is `True`.

**11. How do you efficiently chunk a sequence of `n` items into groups of `k`?**
```python
[range(i, min(i + k, n)) for i in range(0, n, k)]
```

**12. Can `range` be used with `zip()`? Give an example.**
Yes — e.g., `zip(range(1, 4), ["a", "b", "c"])` pairs `1-a, 2-b, 3-c`.

**13. What's the output of `list(range(5, 5))`?**
`[]` — an empty list, since `start == stop`.

**14. How would you iterate over a list's indices in reverse?**
```python
for i in range(len(data) - 1, -1, -1):
    ...
```

**15. Why might converting a huge range to a list be problematic?**
It materializes every value in memory, losing range's main memory advantage and potentially causing high memory usage.

**16. How do you generate a sequence of numbers from 100 down to 0 in steps of 10?**
`range(100, -10, -10)` (note the `stop` is `-10` to include `0`).

**17. Is it valid to pass keyword arguments to `range()`, like `range(start=1, stop=10)`?**
No — `range()` only accepts positional arguments.

**18. What exception is raised by `range(1, 10, 0)`?**
`ValueError: range() arg 3 must not be zero`.

**19. How can `range` be used for pagination logic?**
Compute `start = (page - 1) * page_size` and `stop = start + page_size`, then use `range(start, stop)` to represent the item indices on that page.

**20. Does iterating over the same `range` object twice work correctly?**
Yes — unlike some iterators, a `range` object can be iterated over multiple times because each iteration creates a fresh internal iterator.

---

### Advanced (10)

**1. Why are membership tests and length calculations O(1) for `range` but O(n) for `list`?**
Because a `range`'s values follow a known arithmetic formula (`start + i*step`), Python can algebraically determine length or membership without inspecting individual elements; a `list` has no such formula and must be scanned or have its length tracked/stored per element basis.

**2. How would you implement a function that merges two ranges into one if they are contiguous, otherwise return `None`?**
Compare `step`s for equality, then check if one range's `stop` equals the other's `start` (accounting for step), constructing a new `range` spanning both if contiguous; otherwise return `None`. This requires explicit logic since `range` has no built-in merge operation.

**3. How can you determine, without iterating, whether two ranges represent the exact same set of integers (possibly with different `step` representations)?**
Normalize both ranges by converting to their canonical form (actual `len()`, first element, and effective step when `len > 1`), then compare those properties — this avoids materializing either range while correctly handling equivalent-but-differently-expressed ranges.

**4. Why can't `range()` be used to represent a sequence of floats, and what alternatives exist?**
`range()`'s internal arithmetic and equality/hash guarantees rely on exact integer math; floats introduce rounding errors that would break guarantees like `len()` and membership testing. Alternatives include list comprehensions with division, `numpy.arange()`, or `numpy.linspace()` for float-based numeric sequences.

**5. How does `range` support efficient negative slicing (e.g., `r[-3:]`)?**
The same arithmetic used for positive indexing is applied to negative indices by translating them relative to the range's length, all computed in constant time without iterating.

**6. What's a performance pitfall of using `range(len(x))` plus indexing versus directly iterating `x`?**
Indexing into `x` inside the loop incurs a separate lookup operation per iteration (especially costly for non-list sequences with O(n) indexing); directly iterating yields elements without that overhead and is generally more readable.

**7. How would you generate a Fibonacci-like sequence using range-based logic versus a generator function — which is better and why?**
`range()` cannot directly express Fibonacci (it's not arithmetic), so a generator function (using `yield`) is the appropriate tool; `range` is limited to arithmetic progressions, while generators support arbitrary recurrence relations.

**8. Explain why `range(2**63)` doesn't cause memory issues, while `list(range(2**63))` would be infeasible.**
`range` stores only its three defining integers regardless of magnitude, so creating it is O(1) in memory; converting to a `list` requires allocating storage for every individual value, which for `2**63` elements vastly exceeds available memory.

**9. How would you safely compute the union of values from multiple large `range` objects without excessive memory use?**
Where possible, work with `range` boundaries to detect overlap/merge contiguous ranges algebraically; only fall back to materializing as `set` for the (hopefully smaller) decomposed pieces, avoiding full materialization of very large ranges.

**10. Why does `for i in range(n): list.append(i)` scale linearly, even though range creation itself is O(1)?**
Because `range` creation being O(1) only covers the act of defining the bounds; the loop body still executes `n` times, and each iteration both generates the next value (O(1) per step) and performs an append, so total iteration cost remains O(n) — range's efficiency advantage is in *storage*, not in eliminating per-iteration work.
## 18. Coding Exercises

### Beginner (15)

**1. Print numbers from 1 to 10.**
```python
for i in range(1, 11):
    print(i)
```

**2. Print all even numbers between 1 and 20.**
```python
for i in range(2, 21, 2):
    print(i)
```

**3. Print numbers from 10 down to 1.**
```python
for i in range(10, 0, -1):
    print(i)
```

**4. Calculate the sum of numbers from 1 to 100.**
```python
print(sum(range(1, 101)))   # 5050
```

**5. Create a list of squares of numbers from 1 to 10.**
```python
squares = [i ** 2 for i in range(1, 11)]
print(squares)
```

**6. Print all multiples of 5 between 0 and 50.**
```python
for i in range(0, 51, 5):
    print(i)
```

**7. Find the length of `range(5, 50, 3)` without converting to a list.**
```python
print(len(range(5, 50, 3)))
```

**8. Check whether 25 is in `range(0, 100, 5)`.**
```python
print(25 in range(0, 100, 5))   # True
```

**9. Print the first and last value of `range(10, 100, 4)` using indexing.**
```python
r = range(10, 100, 4)
print(r[0], r[-1])
```

**10. Convert `range(1, 6)` into a tuple.**
```python
print(tuple(range(1, 6)))
```

**11. Print "Hello" exactly 5 times using range.**
```python
for _ in range(5):
    print("Hello")
```

**12. Generate a list of odd numbers from 1 to 19.**
```python
odds = list(range(1, 20, 2))
print(odds)
```

**13. Print numbers from 1 to 5 alongside their index using `enumerate()`.**
```python
for index, value in enumerate(range(1, 6)):
    print(index, value)
```

**14. Find the sum of the first 10 even numbers using `range()`.**
```python
print(sum(range(2, 21, 2)))   # 110
```

**15. Use a `range` slice to get every second number from `range(0, 20)` starting at index 1.**
```python
r = range(0, 20)
print(list(r[1::2]))
```

---

### Intermediate (15)

**1. Write a function that returns `True` if a number is within `range(start, stop)` without converting to a list.**
```python
def in_bounds(n, start, stop):
    return n in range(start, stop)

print(in_bounds(5, 0, 10))   # True
```

**2. Generate a multiplication table for a number `n` from 1 to 10.**
```python
def multiplication_table(n):
    for i in range(1, 11):
        print(f"{n} x {i} = {n * i}")

multiplication_table(7)
```

**3. Write a function that paginates a list given a page number and page size.**
```python
def paginate(data, page, page_size):
    start = (page - 1) * page_size
    stop = start + page_size
    return data[start:stop]

data = list(range(1, 101))
print(paginate(data, page=3, page_size=10))
```

**4. Reverse a list using only `range()` (no slicing, no `reversed()`).**
```python
def reverse_list(data):
    result = []
    for i in range(len(data) - 1, -1, -1):
        result.append(data[i])
    return result

print(reverse_list([1, 2, 3, 4, 5]))
```

**5. Find all numbers divisible by both 3 and 5 between 1 and 100.**
```python
result = [i for i in range(1, 101) if i % 3 == 0 and i % 5 == 0]
print(result)
```

**6. Implement a chunking function that splits a range into sub-ranges of size `k`.**
```python
def chunk(n, k):
    return [range(i, min(i + k, n)) for i in range(0, n, k)]

for c in chunk(23, 5):
    print(list(c))
```

**7. Print a number pyramid pattern from 1 to `n` rows.**
```python
def pyramid(n):
    for i in range(1, n + 1):
        print(" ".join(str(j) for j in range(1, i + 1)))

pyramid(5)
```

**8. Write a function to compute factorial using `range()`.**
```python
def factorial(n):
    result = 1
    for i in range(2, n + 1):
        result *= i
    return result

print(factorial(5))   # 120
```

**9. Find the index of the first multiple of 7 in `range(1, 100)`.**
```python
r = range(1, 100)
for i, value in enumerate(r):
    if value % 7 == 0:
        print(i, value)
        break
```

**10. Write a function to check if a range is empty without iterating.**
```python
def is_empty(r):
    return len(r) == 0

print(is_empty(range(5, 5)))   # True
print(is_empty(range(5, 10)))  # False
```

**11. Generate a list of (index, value) tuples for every 3rd value in `range(0, 30)`.**
```python
r = range(0, 30, 3)
pairs = list(enumerate(r))
print(pairs)
```

**12. Simulate rolling a die 20 times and count how many times each face appears.**
```python
import random
from collections import Counter

rolls = [random.randint(1, 6) for _ in range(20)]
print(Counter(rolls))
```

**13. Write a function that returns the overlapping range of two ranges (same step), or `None`.**
```python
def overlap(r1, r2):
    start = max(r1.start, r2.start)
    stop = min(r1.stop, r2.stop)
    if start >= stop:
        return None
    return range(start, stop, r1.step)

print(overlap(range(0, 10), range(5, 15)))   # range(5, 10)
```

**14. Build a calendar-like grid printing days 1-30 in rows of 7 using `range()`.**
```python
days = range(1, 31)
for i in range(0, len(days), 7):
    week = list(days[i:i + 7])
    print(week)
```

**15. Write a function that returns the nth value of an arithmetic sequence defined by a range, without iterating.**
```python
def nth_value(r, n):
    return r[n]

print(nth_value(range(5, 100, 5), 10))   # 55
```

---

### Advanced (10)

**1. Write a function to merge two contiguous ranges (same step) into one, returning `None` if they aren't contiguous.**
```python
def merge_ranges(r1, r2):
    if r1.step != r2.step:
        return None
    if r1.stop == r2.start:
        return range(r1.start, r2.stop, r1.step)
    if r2.stop == r1.start:
        return range(r2.start, r1.stop, r1.step)
    return None

print(merge_ranges(range(0, 5), range(5, 10)))   # range(0, 10)
```

**2. Implement a function that determines if two ranges (potentially different step representations) contain identical sets of values, without materializing either.**
```python
def same_values(r1, r2):
    if len(r1) != len(r2):
        return False
    if len(r1) == 0:
        return True
    if r1[0] != r2[0]:
        return False
    if len(r1) == 1:
        return True
    return r1.step == r2.step

print(same_values(range(0, 5), range(0, 5, 1)))   # True
print(same_values(range(0, 1), range(0, 1, 5)))   # True (single element)
```

**3. Write a generator-based "sliding window" over a range without converting it to a list.**
```python
def sliding_window(r, size):
    values = []
    it = iter(r)
    for _ in range(size):
        values.append(next(it))
    yield tuple(values)
    for value in it:
        values.pop(0)
        values.append(value)
        yield tuple(values)

print(list(sliding_window(range(1, 8), 3)))
# [(1,2,3), (2,3,4), (3,4,5), (4,5,6), (5,6,7)]
```

**4. Implement binary search over a `range` (treating it as a sorted virtual array) to find a target value's index.**
```python
def range_binary_search(r, target):
    low, high = 0, len(r) - 1
    while low <= high:
        mid = (low + high) // 2
        if r[mid] == target:
            return mid
        elif r[mid] < target:
            low = mid + 1
        else:
            high = mid - 1
    return -1

print(range_binary_search(range(0, 1000, 3), 300))
```

**5. Write a function that splits a large range into `n` roughly equal sub-ranges (for parallel processing).**
```python
def split_range(r, n):
    total = len(r)
    chunk_size = -(-total // n)  # ceiling division
    return [r[i:i + chunk_size] for i in range(0, total, chunk_size)]

for part in split_range(range(0, 23), 4):
    print(list(part))
```

**6. Implement a function that counts how many integers two ranges have in common, without full materialization for huge ranges.**
```python
def count_common(r1, r2):
    if r1.step != r2.step:
        # fallback for differing steps (small ranges only)
        return len(set(r1) & set(r2))
    start = max(r1.start, r2.start)
    stop = min(r1.stop, r2.stop)
    if start >= stop:
        return 0
    return len(range(start, stop, r1.step))

print(count_common(range(0, 100, 5), range(50, 200, 5)))
```

**7. Build a function that, given a range and a value, returns the closest value in the range without iterating.**
```python
def closest_value(r, value):
    if value <= r.start and r.step > 0:
        return r.start
    if value >= r[-1] and r.step > 0:
        return r[-1]
    steps = round((value - r.start) / r.step)
    steps = max(0, min(steps, len(r) - 1))
    return r[steps]

print(closest_value(range(0, 100, 7), 50))
```

**8. Implement a custom iterator class that mimics `range()` for educational purposes (supporting `len`, indexing, and iteration).**
```python
class MyRange:
    def __init__(self, start, stop=None, step=1):
        if stop is None:
            start, stop = 0, start
        self.start, self.stop, self.step = start, stop, step

    def __len__(self):
        return max(0, -(-(self.stop - self.start) // self.step)) if self.step > 0 \
            else max(0, -(-(self.start - self.stop) // -self.step))

    def __getitem__(self, i):
        if i < 0:
            i += len(self)
        if not 0 <= i < len(self):
            raise IndexError("MyRange index out of range")
        return self.start + i * self.step

    def __iter__(self):
        for i in range(len(self)):
            yield self[i]

print(list(MyRange(1, 10, 2)))   # [1, 3, 5, 7, 9]
```

**9. Write a function that, given a list of ranges, returns the total count of unique integers they cover (handling overlaps), efficiently.**
```python
def total_unique_coverage(ranges):
    intervals = sorted((r.start, r.stop) for r in ranges if len(r) > 0 and r.step == 1)
    if not intervals:
        return 0
    merged = [intervals[0]]
    for start, stop in intervals[1:]:
        last_start, last_stop = merged[-1]
        if start <= last_stop:
            merged[-1] = (last_start, max(last_stop, stop))
        else:
            merged.append((start, stop))
    return sum(stop - start for start, stop in merged)

print(total_unique_coverage([range(0, 10), range(5, 15), range(20, 25)]))
# 20
```

**10. Implement pagination metadata (total pages, has_next, has_prev) using only `range()` and arithmetic — no slicing of the actual dataset.**
```python
def pagination_meta(total_items, page, page_size):
    total_pages = max(1, -(-total_items // page_size))
    item_range = range((page - 1) * page_size, min(page * page_size, total_items))
    return {
        "total_pages": total_pages,
        "current_page": page,
        "has_next": page < total_pages,
        "has_prev": page > 1,
        "item_indices": item_range,
    }

print(pagination_meta(total_items=95, page=3, page_size=10))
```
## 19. Best Practices

### Clean Code Tips

* Use `for i in range(n)` for simple counting; reserve `while` loops for dynamic, condition-based termination.
* Prefer `enumerate(sequence)` over `range(len(sequence))` whenever you need both index and value of an existing collection.
* Name range-producing variables descriptively (`page_indices`, `valid_years`) rather than generic `r` in non-trivial code.
* Avoid "magic numbers" in range bounds — use named constants (`range(1, MAX_RETRIES + 1)`).
* When a step isn't `1`, add a short comment explaining its purpose (`range(0, 24, 6)  # every 6 hours`).

### Performance Tips

* Don't convert a `range` to a `list`/`tuple`/`set` unless you specifically need that type's behavior — iteration works directly on `range`.
* Use `range`'s O(1) membership testing (`x in range(...)`) instead of building a list just to check bounds.
* Use `sum(range(...))` instead of manual accumulation loops for arithmetic series.
* For very large sequences, always prefer `range` over a literal list — the memory savings are substantial.
* Provide `default=` to `min()`/`max()` when a range might be empty, to avoid unnecessary `try/except` overhead.

### Readability Guidelines

* Be explicit about inclusive vs. exclusive bounds in comments or variable names when it isn't obvious (`stop_exclusive`, `last_valid_index + 1`).
* Avoid deeply nested ranges/loops when a comprehension or helper function would be clearer.
* When reversing, choose between `reversed(range(...))` and a negative-step range based on which reads more naturally for the situation — both are valid.
* Keep `start`, `stop`, `step` consistent in style (e.g., always include all three explicitly in performance-sensitive or non-obvious code, even when defaults would technically work).

---

## 20. Cheat Sheet

### Syntax

```python
range(stop)
range(start, stop)
range(start, stop, step)
```

### Parameters

| Param | Default | Inclusive? |
|---|---|---|
| `start` | `0` | Yes |
| `stop` | required | No (exclusive) |
| `step` | `1` | n/a |

### Creating

```python
range(5)            # 0..4
range(2, 8)          # 2..7
range(0, 10, 2)      # 0,2,4,6,8
range(10, 0, -1)     # 10..1
```

### Accessing & Slicing

```python
r[0]        # first element
r[-1]       # last element
r[2:5]      # sub-range (returns a range)
r[::-1]     # reversed range
```

### Iterating

```python
for i in range(n): ...
for i, v in enumerate(range(n)): ...
for a, b in zip(range(n), other_iterable): ...
for i in reversed(range(n)): ...
```

### Membership & Attributes

```python
x in r
x not in r
r.start
r.stop
r.step
r.count(x)
r.index(x)
```

### Built-ins

```python
len(r)
min(r)
max(r)
sum(r)
sorted(r)
list(r)
tuple(r)
set(r)
```

### Common Patterns

| Goal | Code |
|---|---|
| Loop n times | `for _ in range(n):` |
| 1-based inclusive loop | `range(1, n + 1)` |
| Reverse loop | `range(n - 1, -1, -1)` or `reversed(range(n))` |
| Every k-th value | `range(start, stop, k)` |
| Index of last element | `r[-1]` |
| Empty check | `len(r) == 0` |
| Pagination start/stop | `range((page-1)*size, page*size)` |
| Chunking | `range(0, n, chunk_size)` |

---

## 21. Summary

* `range()` is an **immutable, memory-efficient sequence type** representing an arithmetic progression of integers, defined by `start`, `stop`, and `step`.
* It supports the full sequence protocol: indexing, negative indexing, slicing, `len()`, membership testing, and iteration — all without storing every value in memory.
* Most operations on `range` (`len`, indexing, membership, `min`/`max`) run in **O(1)** time because values are computed mathematically rather than stored or scanned — the major advantage over an equivalent `list`.
* `range` objects are **immutable** — they cannot be modified in place; convert to `list`/`tuple`/`set` only when mutability or specific collection behavior is genuinely needed.
* The `stop` value is always **exclusive**; getting bounds right (especially with custom `step` values and negative steps) is the most common source of bugs.
* `range()` powers a huge range of practical patterns: loop counters, number generation, pagination, data chunking, pattern printing, and simulations.
* Best practice is to use `range` directly wherever possible — iterate it, test membership in it, slice it — and only materialize it into another collection type when that specific type's capabilities (mutability, hashing, arbitrary types) are required.
* Mastering `range()` — including its step semantics, slicing behavior, and O(1) guarantees — is foundational to writing clean, efficient, idiomatic Python loops and numeric sequences.
