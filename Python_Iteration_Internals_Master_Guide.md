# Python Iteration Internals Master Guide

> A comprehensive, interview-ready reference to iterables, iterators, the iteration protocols, custom iterator design, and advanced iteration techniques in Python.

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Section 1: Iterables](#2-section-1-iterables)
3. [Section 2: Iterators](#3-section-2-iterators)
4. [Section 3: Iterator Protocol](#4-section-3-iterator-protocol)
5. [Section 4: Iterable Protocol](#5-section-4-iterable-protocol)
6. [Section 5: StopIteration](#6-section-5-stopiteration)
7. [Section 6: Custom Iterators](#7-section-6-custom-iterators)
8. [Section 7: Advanced Iteration Techniques](#8-section-7-advanced-iteration-techniques)
9. [Visualization Section](#9-visualization-section)
10. [Comparison Tables](#10-comparison-tables)
11. [Real-World Projects](#11-real-world-projects)
12. [Advanced Patterns](#12-advanced-patterns)
13. [Cheat Sheets](#13-cheat-sheets)
14. [Interview Master Section](#14-interview-master-section)
15. [Exercises](#15-exercises)
16. [Final Section](#16-final-section)

---

## 1. Introduction

Iteration is the backbone of Python. Every `for` loop, comprehension, `with` statement over multiple files, and most of the standard library's data-processing tools rest on two small, elegant protocols: the **Iterable Protocol** and the **Iterator Protocol**.

Understanding these protocols deeply lets you:

- Write memory-efficient, lazy data pipelines.
- Design custom container classes that behave like native Python types.
- Debug confusing bugs like "why can I only loop over this once?"
- Answer almost any Python interview question about `for` loops, generators, or `StopIteration`.

This guide builds from first principles (what is an iterable?) up to production-grade patterns (streaming CSV processors, pagination systems, producer-consumer pipelines), entirely from a **language-level** perspective — no CPython source code or internals are discussed, only behavior you can observe and rely on.


---

## 2. Section 1: Iterables

### 2.1 Introduction

An **iterable** is any Python object capable of returning its members one at a time, allowing it to be looped over with a `for` statement. It is a *capability*, not a specific data structure — many unrelated types (lists, strings, files, dictionaries) all qualify as iterables because they each agree to a single contract.

### 2.2 Conceptual Understanding

An object is iterable if it implements `__iter__()` (the **Iterable Protocol**), which must return an **iterator**. Some older-style objects are iterable via `__getitem__()` with integer indices starting at 0, but modern code should always rely on `__iter__()`.

Key idea: **An iterable knows how to produce an iterator. It is not itself necessarily an iterator.**

```
Iterable ──iter()──> Iterator ──next()──> Values...
```

### 2.3 Syntax

```python
for item in iterable:
    ...

# Equivalent low-level form
it = iter(iterable)
while True:
    try:
        item = next(it)
    except StopIteration:
        break
```

### 2.4 Examples

| Type | Example | Iterable? | Notes |
|---|---|---|---|
| `list` | `[1, 2, 3]` | Yes | Ordered, re-iterable |
| `tuple` | `(1, 2, 3)` | Yes | Immutable, re-iterable |
| `str` | `"abc"` | Yes | Iterates over characters |
| `set` | `{1, 2, 3}` | Yes | Unordered, re-iterable |
| `dict` | `{"a": 1}` | Yes | Iterates over keys by default |
| `range` | `range(5)` | Yes | Lazy, re-iterable, no storage |
| file object | `open("f.txt")` | Yes | Iterates over lines, **single-pass** |
| generator | `(x for x in range(5))` | Yes | Lazy, **single-pass** |

```python
numbers = [1, 2, 3]
text = "abc"
data = {"x": 1, "y": 2}
nums_range = range(3)

for n in numbers:
    print(n, end=" ")
print()

for ch in text:
    print(ch, end=" ")
print()

for key in data:
    print(key, end=" ")
print()

for n in nums_range:
    print(n, end=" ")
print()
```

**Output:**
```
1 2 3 
a b c 
x y 
0 1 2 
```

### 2.5 `iter(obj)`

Calling `iter()` on an iterable retrieves a fresh iterator object from it.

```python
nums = [10, 20, 30]
it1 = iter(nums)
it2 = iter(nums)
print(it1 is it2)        # False — independent iterators
print(next(it1))         # 10
print(next(it1))         # 20
print(next(it2))         # 10 — it2 is unaffected by it1
```

**Output:**
```
False
10
20
10
```

### 2.6 Traversability and Multiple Passes

| Iterable Type | Re-iterable (multi-pass)? | Why |
|---|---|---|
| `list`, `tuple`, `str`, `set`, `dict` | Yes | `__iter__()` returns a *new* iterator each call |
| `range` | Yes | Computes values lazily on demand, stateless object |
| file object | No (single-pass) | The file's read cursor is shared state |
| generator | No (single-pass) | Internal execution state is exhausted once |
| `itertools` iterators (e.g. `map`, `zip`, `filter`) | No (single-pass) | They *are* iterators, not just iterables |

```python
gen = (x * x for x in range(3))
print(list(gen))   # [0, 1, 4]
print(list(gen))   # [] — already exhausted, cannot restart
```

**Output:**
```
[0, 1, 4]
[]
```

### 2.7 Common Mistakes

- Assuming every iterable can be looped over multiple times (generators and file objects cannot).
- Confusing an iterable with an iterator — calling `next()` directly on a `list` raises `TypeError: 'list' object is not an iterator`.
- Exhausting a generator accidentally by passing it to `list()` and then trying to reuse it.

### 2.8 Best Practices

- If you need to iterate multiple times, store data in a `list`/`tuple`, or wrap re-creation logic in a function/class that returns a new iterator each time.
- Use generators and `itertools` for streaming, single-pass pipelines where memory matters.
- Never mutate a list while iterating over it directly — iterate over a copy instead (`for x in list(original):`).

### 2.9 Performance Considerations

| Approach | Memory | Use Case |
|---|---|---|
| `list` of all items | O(n) | Need random access, multiple passes |
| `range` | O(1) | Pure numeric sequences |
| Generator expression | O(1) | Streaming, one pass, large/infinite data |
| File iteration | O(1) per line | Large files line-by-line |

### 2.10 Interview Questions

**Q1: Is every iterator also an iterable?**
A: Yes. Iterators implement `__iter__()` (which returns `self`), satisfying the iterable contract.

**Q2: Is every iterable also an iterator?**
A: No. Most iterables (lists, strings) are not iterators themselves; they only know how to *produce* one.

**Q3: How do you check if an object is iterable without iterating it?**
A: Use `from collections.abc import Iterable; isinstance(obj, Iterable)`, or check `hasattr(obj, "__iter__")`. Note this checks structural capability, not guaranteed correct behavior.

### 2.11 Exercises with Solutions

**Exercise 1:** Write a function `is_reiterable(obj)` that returns `True` if calling `iter()` twice produces two independent iterators.

```python
def is_reiterable(obj):
    try:
        it1 = iter(obj)
        it2 = iter(obj)
        return it1 is not it2
    except TypeError:
        return False

print(is_reiterable([1, 2, 3]))                 # True
print(is_reiterable((x for x in range(3))))     # False (generator returns itself)
```

**Output:**
```
True
False
```

**Exercise 2:** Given `data = {"a": 1, "b": 2, "c": 3}`, iterate and print key-value pairs using `.items()`.

```python
data = {"a": 1, "b": 2, "c": 3}
for k, v in data.items():
    print(f"{k} -> {v}")
```

**Output:**
```
a -> 1
b -> 2
c -> 3
```

---

## 3. Section 2: Iterators

### 3.1 Introduction

An **iterator** is an object that produces a stream of values, one at a time, via repeated calls to `next()`, and remembers exactly where it left off. It is a stateful, single-use traversal engine over (or generator of) data.

### 3.2 Conceptual Understanding

An iterator is *not* the data — it is a **cursor** that tracks position/state and knows how to compute or fetch the next value. Once a value is yielded, it cannot be "un-yielded"; the iterator moves strictly forward.

### 3.3 Syntax

```python
it = iter([1, 2, 3])
next(it)   # 1
next(it)   # 2
next(it)   # 3
next(it)   # raises StopIteration
```

### 3.4 Examples

```python
it = iter(["a", "b", "c"])
print(next(it))
print(next(it))
print(next(it))
try:
    print(next(it))
except StopIteration:
    print("Exhausted!")
```

**Output:**
```
a
b
c
Exhausted!
```

### 3.5 State Tracking, Exhaustion, Consumption

| Stage | State of Iterator | Result of `next()` |
|---|---|---|
| Fresh | Position = start | First value |
| Mid-traversal | Position = k | (k+1)-th value |
| Exhausted | Position = end | Raises `StopIteration` forever |

Visual model:

```
[ a, b, c ]
  ^
  cursor (position 0)

next() -> 'a', cursor moves to position 1
next() -> 'b', cursor moves to position 2
next() -> 'c', cursor moves to position 3 (end)
next() -> StopIteration (cursor stays at end)
```

Once exhausted, an iterator **stays exhausted** — calling `next()` again keeps raising `StopIteration`. It cannot be reset; you must call `iter()` on the original iterable again (if that iterable supports re-iteration).

### 3.6 Common Mistakes

- Calling `next()` on an exhausted iterator expecting it to "wrap around" or restart.
- Forgetting that consuming an iterator via `list(it)`, `sum(it)`, or a `for` loop **permanently drains it**.
- Sharing one iterator across multiple consumers expecting each to see all items (they split the stream instead).

```python
it = iter([1, 2, 3, 4])
first_two = [next(it), next(it)]
remaining = list(it)   # only [3, 4] — first two are gone forever
print(first_two, remaining)
```

**Output:**
```
[1, 2] [3, 4]
```

### 3.7 Best Practices

- Treat iterators as **disposable, single-use** objects.
- Use `itertools.tee()` if multiple independent consumers need the same stream.
- Provide a default for `next(it, default)` when absence of more items is a normal, non-exceptional case.

```python
it = iter([])
value = next(it, "no-more-data")
print(value)
```

**Output:**
```
no-more-data
```

### 3.8 Performance Considerations

| Operation | Cost |
|---|---|
| `next(it)` | O(1) amortized for most built-in iterators |
| Re-creating iterator via `iter()` | O(1) for sequences, O(n) "cost" hidden if it copies data |
| Holding a long-lived iterator | O(1) memory regardless of source size — this is the core efficiency win |

### 3.9 Interview Questions

**Q1: What's the difference between consuming and inspecting an iterator?**
A: There is no "peek" built into the protocol — calling `next()` always *consumes* a value. To peek, you must store the value yourself (e.g., `itertools` lookahead patterns).

**Q2: Can you reset an iterator?**
A: No native reset exists. You must build a new iterator, typically by calling `iter()` on the original iterable again.

**Q3: What happens if you call `next()` after `StopIteration` has already been raised once?**
A: It raises `StopIteration` again — the exhausted state is permanent for that iterator instance.

### 3.10 Exercises with Solutions

**Exercise 1:** Write a function that safely retrieves the first `n` items from any iterator without crashing if fewer are available.

```python
def take(iterator, n):
    result = []
    for _ in range(n):
        try:
            result.append(next(iterator))
        except StopIteration:
            break
    return result

it = iter([1, 2])
print(take(it, 5))   # [1, 2]
```

**Output:**
```
[1, 2]
```

**Exercise 2:** Demonstrate that two separate `for` loops over the *same* iterator object split the data rather than each seeing everything.

```python
it = iter(range(6))
print("First loop:")
for x in it:
    print(x)
    if x == 2:
        break

print("Second loop (continues from where first left off):")
for x in it:
    print(x)
```

**Output:**
```
First loop:
0
1
2
Second loop (continues from where first left off):
3
4
5
```

---

## 4. Section 3: Iterator Protocol

### 4.1 Introduction

The **Iterator Protocol** is the contract that makes an object usable wherever Python expects an iterator: it must implement `__iter__()` (returning itself) and `__next__()` (returning the next value or raising `StopIteration`).

### 4.2 Conceptual Understanding

| Method | Required Behavior |
|---|---|
| `__iter__(self)` | Returns `self` |
| `__next__(self)` | Returns the next value, or raises `StopIteration` when exhausted |

This protocol allows a `for` loop to work on *any* object satisfying it, regardless of internal implementation — lists, files, sockets, network streams, or hand-written classes all look identical from the loop's point of view.

### 4.3 Syntax

```python
class MyIterator:
    def __iter__(self):
        return self

    def __next__(self):
        # compute / fetch next value
        # raise StopIteration when done
        ...
```

### 4.4 How `for item in iterable:` Works — Conceptual Equivalent

```python
for item in iterable:
    BODY
```

is conceptually equivalent to:

```python
_it = iter(iterable)        # calls iterable.__iter__()
while True:
    try:
        item = next(_it)    # calls _it.__next__()
    except StopIteration:
        break
    BODY
```

### 4.5 Diagram

```
for item in iterable:
        │
        ▼
   iter(iterable)  ──calls──► iterable.__iter__() ──► returns Iterator
        │
        ▼
   loop: next(iterator) ──calls──► iterator.__next__()
        │                                  │
        │                         returns value ──► BODY runs
        │                                  │
        │                         raises StopIteration
        ▼                                  │
      loop ends  ◄────────────────────────┘
```

### 4.6 Examples

```python
class CountUpTo:
    """Iterator that counts from 1 to limit."""
    def __init__(self, limit):
        self.limit = limit
        self.current = 0

    def __iter__(self):
        return self

    def __next__(self):
        if self.current >= self.limit:
            raise StopIteration
        self.current += 1
        return self.current

counter = CountUpTo(4)
for n in counter:
    print(n, end=" ")
print()

# Manual protocol usage
manual = CountUpTo(3)
it = iter(manual)
print(next(it), next(it), next(it))
```

**Output:**
```
1 2 3 4 
1 2 3
```

### 4.7 Common Mistakes

- Forgetting `__iter__()` returns `self` — without it, `for` loops fail with `TypeError: iter() returned non-iterator`.
- Returning a sentinel value (like `None` or `-1`) instead of raising `StopIteration` to signal "done" — `for` will loop forever or misbehave.
- Mutating shared state inside `__next__()` inconsistently, causing skipped or duplicated values.

### 4.8 Best Practices

- Keep `__next__()` side-effect-minimal and deterministic.
- Always raise `StopIteration` (not return a special value) to end iteration.
- Prefer generator functions over manual classes when state is simple — the `yield` keyword writes the protocol for you.

### 4.9 Performance Considerations

| Implementation Style | Pros | Cons |
|---|---|---|
| Hand-written class with `__next__` | Full control, can hold rich state | More boilerplate |
| Generator function (`yield`) | Concise, protocol auto-implemented | Slightly less control over object identity |

### 4.10 Interview Questions

**Q1: What must `__iter__()` return on an iterator class?**
A: `self`. This is what lets iterators also be used directly in `for` loops.

**Q2: What two dunder methods define the Iterator Protocol?**
A: `__iter__()` and `__next__()`.

**Q3: Why does `__iter__()` exist on an iterator at all if it just returns `self`?**
A: So iterators satisfy the Iterable Protocol too, letting them be passed anywhere an iterable is expected (e.g., directly into a `for` loop, or into `iter()` again harmlessly).

### 4.11 Exercises with Solutions

**Exercise 1:** Implement an iterator class `Squares` that yields the squares of 1..n.

```python
class Squares:
    def __init__(self, n):
        self.n = n
        self.current = 0

    def __iter__(self):
        return self

    def __next__(self):
        self.current += 1
        if self.current > self.n:
            raise StopIteration
        return self.current ** 2

print(list(Squares(5)))
```

**Output:**
```
[1, 4, 9, 16, 25]
```

---

## 5. Section 4: Iterable Protocol

### 5.1 Introduction

The **Iterable Protocol** requires only a single method: `__iter__()`, which must return an iterator (an object obeying the Iterator Protocol).

### 5.2 Conceptual Understanding

| Requirement | Detail |
|---|---|
| Must implement | `__iter__(self)` |
| Must return | An object implementing `__iter__()` + `__next__()` |
| May be | The same object (if it's also an iterator) or a fresh helper iterator object |

A well-designed iterable returns a **brand-new iterator** on every call to `__iter__()`, enabling multiple independent passes — this is what separates a reusable "container" from a single-use "stream."

### 5.3 Syntax

```python
class MyIterable:
    def __iter__(self):
        return SomeIteratorObject(...)
```

### 5.4 Building a Custom Iterable Class (separate from its Iterator)

```python
class NumberRange:
    """An iterable that can be traversed multiple times."""
    def __init__(self, start, end):
        self.start = start
        self.end = end

    def __iter__(self):
        return NumberRangeIterator(self.start, self.end)


class NumberRangeIterator:
    def __init__(self, current, end):
        self.current = current
        self.end = end

    def __iter__(self):
        return self

    def __next__(self):
        if self.current >= self.end:
            raise StopIteration
        value = self.current
        self.current += 1
        return value


nr = NumberRange(1, 4)
print(list(nr))   # First pass
print(list(nr))   # Second pass — works because __iter__ creates a fresh iterator
```

**Output:**
```
[1, 2, 3]
[1, 2, 3]
```

### 5.5 Common Mistakes

- Collapsing iterable and iterator into one class with shared mutable state, accidentally making the "iterable" single-pass (a very common bug).
- Forgetting `__iter__()` entirely, relying on `__getitem__()` fallback iteration, which is legacy and has surprising edge cases.

```python
# BUGGY: this object is both iterable and iterator with shared state
class BuggyRange:
    def __init__(self, n):
        self.n = n
        self.i = 0

    def __iter__(self):
        return self          # returns itself, not a fresh iterator!

    def __next__(self):
        if self.i >= self.n:
            raise StopIteration
        self.i += 1
        return self.i

br = BuggyRange(3)
print(list(br))   # [1, 2, 3]
print(list(br))   # [] -- BUG: second pass is empty, state was already consumed
```

**Output:**
```
[1, 2, 3]
[]
```

### 5.6 Best Practices

- Separate **Iterable** (the container, holds configuration/data) from **Iterator** (the cursor, holds traversal state) when multi-pass behavior is desired.
- It's perfectly fine for a class to be both iterable and its own iterator **only when** single-pass, stream-like semantics are intended (this is exactly what generators do).

### 5.7 Performance Considerations

| Design | Memory | Re-iterable | Typical Use |
|---|---|---|---|
| Separate Iterable + Iterator classes | Low (no caching needed) | Yes | Containers, ranges, custom collections |
| Combined Iterable/Iterator (self-returning) | Low | No | Streams, generators, one-shot pipelines |

### 5.8 Interview Questions

**Q1: What is the minimum requirement for an object to be iterable?**
A: It must implement `__iter__()` returning an iterator (or support the legacy `__getitem__()` fallback).

**Q2: Why might a class deliberately implement both Iterable and Iterator protocols in one object?**
A: To represent a single-use stream cheaply — no separate iterator object needs to be allocated, at the cost of losing multi-pass support.

### 5.9 Exercises with Solutions

**Exercise 1:** Fix the `BuggyRange` class above so it supports multiple independent passes.

```python
class FixedRange:
    def __init__(self, n):
        self.n = n

    def __iter__(self):
        return FixedRangeIterator(self.n)


class FixedRangeIterator:
    def __init__(self, n):
        self.n = n
        self.i = 0

    def __iter__(self):
        return self

    def __next__(self):
        if self.i >= self.n:
            raise StopIteration
        self.i += 1
        return self.i


fr = FixedRange(3)
print(list(fr))
print(list(fr))
```

**Output:**
```
[1, 2, 3]
[1, 2, 3]
```

---

## 6. Section 5: StopIteration

### 6.1 Introduction

`StopIteration` is the built-in exception that signals "there are no more items." It is the universal termination signal used by every `for` loop, comprehension, and iteration-consuming function in Python.

### 6.2 Conceptual Understanding

| Aspect | Detail |
|---|---|
| Purpose | Signal end-of-sequence to loop constructs |
| Raised by | `__next__()` implementations, or automatically when a generator function returns |
| Caught by | `for` loops, comprehensions, `next()`'s 2-argument form (suppressed), `itertools` consumers |
| Carries data? | Yes — `StopIteration(value)`; for generators, `value` becomes the generator's "return value", retrievable via `.value` on the exception, or via `yield from` |

### 6.3 Behavior and Loop Termination

```python
def gen():
    yield 1
    yield 2
    return "done"     # becomes StopIteration's value

g = gen()
print(next(g))
print(next(g))
try:
    next(g)
except StopIteration as e:
    print("Stopped with value:", e.value)
```

**Output:**
```
1
2
Stopped with value: done
```

### 6.4 Flow Diagram

```
            ┌────────────┐
   next() ─►│ __next__() │
            └─────┬──────┘
                  │
        ┌─────────┴─────────┐
        ▼                   ▼
   value available     no value left
        │                   │
        ▼                   ▼
   return value        raise StopIteration
        │                   │
        ▼                   ▼
   loop body runs      loop catches it, exits cleanly
```

### 6.5 Examples

```python
it = iter([1, 2])
print(next(it))
print(next(it))
try:
    next(it)
except StopIteration:
    print("No more items — loop would exit here")
```

**Output:**
```
1
2
No more items — loop would exit here
```

A crucial historical nuance (PEP 479): if `StopIteration` is raised *accidentally* inside a generator's body (not as the natural end-of-generator signal), Python converts it into a `RuntimeError` to avoid silently terminating an enclosing loop.

```python
def broken_gen():
    yield 1
    raise StopIteration   # accidental — NOT just a normal generator return
    yield 2

g = broken_gen()
print(next(g))
try:
    next(g)
except RuntimeError as e:
    print("Converted to RuntimeError:", e)
```

**Output:**
```
1
Converted to RuntimeError: generator raised StopIteration
```

### 6.6 Common Mistakes

- Manually raising `StopIteration` inside a generator function body instead of simply using `return` — triggers `RuntimeError` (PEP 479) on modern Python.
- Letting `StopIteration` propagate *out* of a `for` loop body unexpectedly when calling `next()` directly inside the loop, silently terminating the loop early.
- Catching `Exception` broadly and accidentally swallowing `StopIteration`, masking real bugs.

### 6.7 Best Practices

- In generator functions, use `return` to end iteration; let Python raise `StopIteration` for you.
- In manual iterator classes, `raise StopIteration` explicitly inside `__next__()`.
- Use `next(it, default)` to avoid try/except boilerplate when a default makes sense.

### 6.8 Performance Considerations

`StopIteration` is a normal Python exception; raising/catching it has a small but real overhead per loop termination — negligible compared to the cost of iterating the elements themselves, but worth knowing when chaining millions of tiny iterators in tight loops.

### 6.9 Interview Questions

**Q1: What's the difference between a generator's `return` and `raise StopIteration`?**
A: `return value` inside a generator is the idiomatic way to end iteration; Python automatically raises `StopIteration(value)` for you. Explicitly raising `StopIteration` yourself inside the generator body is now treated as a bug and converted to `RuntimeError` (PEP 479).

**Q2: Does `StopIteration` carry information?**
A: Yes, an optional `value` attribute, accessible via the exception instance or as the generator's "return value" when using `yield from`.

**Q3: What happens if you call `next()` on an exhausted iterator inside a `for` loop body (not the loop's own iteration)?**
A: The `StopIteration` propagates as a real exception out of the loop body — it does **not** silently end the surrounding `for` loop, since that loop is driven by its *own* iterator/`next()` call, not yours.

### 6.10 Exercises with Solutions

**Exercise 1:** Write a generator that yields three values then returns a status message; retrieve that message safely.

```python
def task_runner():
    yield "step1"
    yield "step2"
    yield "step3"
    return "ALL_STEPS_COMPLETE"

g = task_runner()
results = []
status = None
while True:
    try:
        results.append(next(g))
    except StopIteration as e:
        status = e.value
        break

print(results)
print(status)
```

**Output:**
```
['step1', 'step2', 'step3']
ALL_STEPS_COMPLETE
```

---

## 7. Section 6: Custom Iterators

### 7.1 Introduction

This section builds six custom iterators of increasing complexity, each following the same design template: **Design → Code → Output → Analysis**. These patterns map directly onto real production needs: counters, countdowns, filtered sequences, mathematical sequences, chunked file reads, and paginated APIs.

### 7.2 Common Mistakes (apply to all below)

- Forgetting to raise `StopIteration` for a terminating condition (infinite loop).
- Mutating the wrong attribute (cursor vs. limit), causing off-by-one errors.
- Re-using one iterator instance across unrelated loops, causing unexpected exhaustion.

### 7.3 Best Practices (apply to all below)

- Validate constructor arguments early (fail fast).
- Keep `__next__()` minimal — one clear computation per call.
- Prefer generator functions for simple cases; use classes when you need extra methods, introspectable state, or to combine with other class-based APIs.

---

### 7.4 Counter Iterator

**Design:** Counts upward indefinitely from a starting value (an *infinite* iterator — terminates only when the consumer stops asking).

```python
class Counter:
    def __init__(self, start=0, step=1):
        self.current = start
        self.step = step

    def __iter__(self):
        return self

    def __next__(self):
        value = self.current
        self.current += self.step
        return value


c = Counter(start=5, step=2)
it = iter(c)
print([next(it) for _ in range(5)])
```

**Output:**
```
[5, 7, 9, 11, 13]
```

**Analysis:** No `StopIteration` is ever raised — this models infinite streams (e.g., ID generators, clock ticks). Consumers must bound consumption themselves (`itertools.islice`, a manual counter, or `break`).

---

### 7.5 Countdown Iterator

**Design:** Counts downward from `start` to `0`, terminating with `StopIteration` once below zero.

```python
class Countdown:
    def __init__(self, start):
        self.current = start

    def __iter__(self):
        return self

    def __next__(self):
        if self.current < 0:
            raise StopIteration
        value = self.current
        self.current -= 1
        return value


for n in Countdown(3):
    print(n, end=" ")
print()
```

**Output:**
```
3 2 1 0 
```

**Analysis:** A finite iterator with a clear termination condition — the canonical "textbook" iterator shape.

---

### 7.6 Even Number Iterator

**Design:** Filters a numeric range, yielding only even numbers up to a limit.

```python
class EvenNumbers:
    def __init__(self, limit):
        self.limit = limit
        self.current = 0

    def __iter__(self):
        return self

    def __next__(self):
        if self.current > self.limit:
            raise StopIteration
        value = self.current
        self.current += 2
        return value


print(list(EvenNumbers(10)))
```

**Output:**
```
[0, 2, 4, 6, 8, 10]
```

**Analysis:** Demonstrates that an iterator's "step" can encode filtering logic directly, avoiding the need for a separate `filter()` call — useful when filtering is the iterator's entire reason for existing.

---

### 7.7 Fibonacci Iterator

**Design:** Generates Fibonacci numbers lazily, one at a time, without storing the whole sequence.

```python
class Fibonacci:
    def __init__(self, count):
        self.count = count
        self.a, self.b = 0, 1
        self.produced = 0

    def __iter__(self):
        return self

    def __next__(self):
        if self.produced >= self.count:
            raise StopIteration
        value = self.a
        self.a, self.b = self.b, self.a + self.b
        self.produced += 1
        return value


print(list(Fibonacci(10)))
```

**Output:**
```
[0, 1, 1, 2, 3, 5, 8, 13, 21, 34]
```

**Analysis:** Demonstrates O(1) memory state (just two running values) regardless of how many terms are requested — the hallmark advantage of iterator-based design over precomputing a list.

---

### 7.8 File Chunk Iterator

**Design:** Reads a file in fixed-size byte chunks instead of loading it all into memory — essential for large files.

```python
class FileChunkIterator:
    def __init__(self, filepath, chunk_size=1024):
        self.filepath = filepath
        self.chunk_size = chunk_size
        self.file = open(filepath, "r")

    def __iter__(self):
        return self

    def __next__(self):
        chunk = self.file.read(self.chunk_size)
        if not chunk:
            self.file.close()
            raise StopIteration
        return chunk

# Example usage (assuming 'sample.txt' exists):
# for chunk in FileChunkIterator("sample.txt", chunk_size=4):
#     print(repr(chunk))
```

**Demonstration with an in-memory substitute (`io.StringIO`) so the example is runnable without a real file:**

```python
import io

class ChunkIteratorFromStream:
    def __init__(self, stream, chunk_size=4):
        self.stream = stream
        self.chunk_size = chunk_size

    def __iter__(self):
        return self

    def __next__(self):
        chunk = self.stream.read(self.chunk_size)
        if not chunk:
            raise StopIteration
        return chunk

stream = io.StringIO("HelloWorldPython")
for chunk in ChunkIteratorFromStream(stream, chunk_size=5):
    print(repr(chunk))
```

**Output:**
```
'Hello'
'World'
'Pytho'
'n'
```

**Analysis:** Demonstrates the **streaming pattern**: regardless of total file size, memory usage stays bounded by `chunk_size`. Always close resources (here, on exhaustion or via context managers in production code).

---

### 7.9 Pagination Iterator

**Design:** Simulates fetching "pages" of records (e.g., from a database or API) one page at a time.

```python
class PaginatedIterator:
    def __init__(self, data, page_size):
        self.data = data
        self.page_size = page_size
        self.index = 0

    def __iter__(self):
        return self

    def __next__(self):
        if self.index >= len(self.data):
            raise StopIteration
        page = self.data[self.index : self.index + self.page_size]
        self.index += self.page_size
        return page


records = list(range(1, 11))
for page in PaginatedIterator(records, page_size=3):
    print(page)
```

**Output:**
```
[1, 2, 3]
[4, 5, 6]
[7, 8, 9]
[10]
```

**Analysis:** Each `next()` call returns a *batch* rather than a single item — a common real-world pattern for API pagination, batch ETL jobs, and UI "load more" features. Note the last page may be shorter than `page_size`.

### 7.10 Performance Considerations (Custom Iterators)

| Iterator | Memory | Time per `next()` | Notes |
|---|---|---|---|
| Counter | O(1) | O(1) | Infinite — bound externally |
| Countdown | O(1) | O(1) | Finite, deterministic |
| EvenNumbers | O(1) | O(1) | Filtering folded into stepping |
| Fibonacci | O(1) | O(1) | No sequence storage needed |
| FileChunkIterator | O(chunk_size) | O(chunk_size) | Disk I/O bound |
| PaginatedIterator | O(page_size) | O(page_size) (slicing) | Good for batch APIs |

### 7.11 Interview Questions

**Q1: Why is an iterator-based Fibonacci generator more memory-efficient than building a list of Fibonacci numbers first?**
A: It only ever holds the last two values in memory (O(1)), regardless of how many terms are eventually requested, whereas a list holds all generated terms simultaneously (O(n)).

**Q2: What's the danger of a Counter-style infinite iterator?**
A: If consumed naively by `list()` or an unbounded `for` loop, it will run forever (or until memory/resources are exhausted) — infinite iterators must always be paired with an external bound like `itertools.islice` or an explicit `break`.

### 7.12 Exercises with Solutions

**Exercise 1:** Modify the `PaginatedIterator` to also expose the current page number via a `page_number` attribute.

```python
class PaginatedIteratorWithPageNumber:
    def __init__(self, data, page_size):
        self.data = data
        self.page_size = page_size
        self.index = 0
        self.page_number = 0

    def __iter__(self):
        return self

    def __next__(self):
        if self.index >= len(self.data):
            raise StopIteration
        page = self.data[self.index : self.index + self.page_size]
        self.index += self.page_size
        self.page_number += 1
        return self.page_number, page


for num, page in PaginatedIteratorWithPageNumber(list(range(1, 8)), 3):
    print(f"Page {num}: {page}")
```

**Output:**
```
Page 1: [1, 2, 3]
Page 2: [4, 5, 6]
Page 3: [7]
```

---

## 8. Section 7: Advanced Iteration Techniques

### 8.1 Introduction

Python's standard library provides powerful building blocks that wrap the iteration protocols into expressive, lazy, composable tools. Mastering them lets you replace explicit loops with concise, efficient pipelines.

### 8.2 Conceptual Understanding

All the tools below either **consume** iterables or **produce lazy iterators**. None of them require their input to be a list — any iterable works, including infinite generators (for the ones that support laziness).

### 8.3 Reference Table

| Tool | Lazy? | Returns | Typical Use |
|---|---|---|---|
| `enumerate(it, start=0)` | Yes | Iterator of `(index, value)` | Need index + value together |
| `zip(*iterables)` | Yes | Iterator of tuples | Pair up parallel sequences |
| `reversed(seq)` | Yes (for sequences) | Iterator, reverse order | Reverse traversal without copying |
| `sorted(it, key=, reverse=)` | No (returns a list) | `list` | Need fully ordered output |
| `iter(obj[, sentinel])` | N/A | Iterator | Manual protocol entry point |
| `next(it[, default])` | N/A | Next value or default | Manual stepping |
| `any(it)` | Yes (short-circuits) | `bool` | "Does any element satisfy X?" |
| `all(it)` | Yes (short-circuits) | `bool` | "Do all elements satisfy X?" |
| `map(func, *iterables)` | Yes | Iterator | Transform each element |
| `filter(func, iterable)` | Yes | Iterator | Keep elements matching predicate |
| Generator expression `(x for x in it)` | Yes | Generator | Inline lazy transformation |

### 8.4 Examples

```python
fruits = ["apple", "banana", "cherry"]

for i, f in enumerate(fruits, start=1):
    print(i, f)
```

**Output:**
```
1 apple
2 banana
3 cherry
```

```python
names = ["Alice", "Bob"]
ages = [30, 25]
for name, age in zip(names, ages):
    print(f"{name} is {age}")
```

**Output:**
```
Alice is 30
Bob is 25
```

```python
print(list(reversed([1, 2, 3, 4])))
print(sorted([3, 1, 2], reverse=True))
```

**Output:**
```
[4, 3, 2, 1]
[3, 2, 1]
```

```python
nums = [1, 2, 3, 4, 5]
print(any(n > 4 for n in nums))
print(all(n > 0 for n in nums))
print(list(map(lambda x: x * 2, nums)))
print(list(filter(lambda x: x % 2 == 0, nums)))
```

**Output:**
```
True
True
[2, 4, 6, 8, 10]
[2, 4]
```

```python
squares = (x * x for x in range(5))
print(next(squares))
print(list(squares))
```

**Output:**
```
0
[1, 4, 9, 16]
```

### 8.5 Lazy Evaluation, Streaming Data, Large Datasets

Lazy evaluation means values are computed **only when requested**, not upfront. This is what allows Python to process datasets larger than available memory, or even infinite streams.

```python
import itertools

def infinite_counter():
    n = 0
    while True:
        yield n
        n += 1

# Lazily take only the first 5 values from an infinite stream
first_five = list(itertools.islice(infinite_counter(), 5))
print(first_five)
```

**Output:**
```
[0, 1, 2, 3, 4]
```

| Evaluation Style | When Computed | Memory Profile | Example |
|---|---|---|---|
| Eager | Immediately, all at once | O(n) | `[x*x for x in range(10**7)]` |
| Lazy | On demand, one at a time | O(1) per step | `(x*x for x in range(10**7))` |

### 8.6 Common Mistakes

- Calling `sorted()` on an infinite generator — it will hang forever since `sorted()` must materialize everything first.
- Chaining multiple `map()`/`filter()` calls and forgetting the result is an iterator (single-pass) — trying to iterate twice yields nothing the second time.
- Using `zip()` on unequal-length iterables and being surprised it silently stops at the shortest one (use `itertools.zip_longest` if padding is needed).

```python
zipped = zip([1, 2, 3], ["a", "b"])
print(list(zipped))   # stops at shortest iterable
```

**Output:**
```
[(1, 'a'), (2, 'b')]
```

### 8.7 Best Practices

- Prefer generator expressions/`map`/`filter` over building intermediate lists when chaining transformations on large data.
- Use `any()`/`all()` for early-exit boolean checks instead of manually looping — they short-circuit automatically.
- Reach for `itertools` (`islice`, `chain`, `tee`, `groupby`, `zip_longest`, `accumulate`) before hand-rolling iteration logic.

### 8.8 Performance Considerations

| Pattern | Relative Speed | Memory |
|---|---|---|
| List comprehension (eager) | Fast for small/medium data, full materialization | O(n) |
| Generator expression (lazy) | Slightly more per-item overhead, but streaming | O(1) |
| `map()`/`filter()` (lazy, C-implemented) | Comparable to generator expressions, sometimes faster | O(1) |
| `sorted()` | Requires full materialization — O(n log n) time, O(n) space | O(n) |

### 8.9 Interview Questions

**Q1: Why does `zip()` stop at the shortest iterable instead of raising an error?**
A: By design, to safely support pairing iterables of unknown or differing lengths without extra bookkeeping; use `itertools.zip_longest` when padding to the longest is required.

**Q2: Is `map(func, iterable)` lazy in Python 3?**
A: Yes — it returns an iterator and computes `func(item)` only when `next()` is called on it, unlike Python 2 where `map()` returned a list.

**Q3: How would you process a 50 GB log file without exhausting memory?**
A: Iterate line-by-line using the file object's own iterator protocol (`for line in f:`), combined with generator-based filtering/transformation — never call `.readlines()` or load the whole file into a list.

### 8.10 Exercises with Solutions

**Exercise 1:** Using `map` and `filter`, extract the squares of only the odd numbers from 1 to 10.

```python
nums = range(1, 11)
odd_squares = list(map(lambda x: x * x, filter(lambda x: x % 2 == 1, nums)))
print(odd_squares)
```

**Output:**
```
[1, 9, 25, 49, 81]
```

**Exercise 2:** Use `itertools.islice` to get items 3 through 6 (inclusive, 0-indexed) of an infinite generator.

```python
import itertools

def naturals():
    n = 0
    while True:
        yield n
        n += 1

result = list(itertools.islice(naturals(), 3, 7))
print(result)
```

**Output:**
```
[3, 4, 5, 6]
```

---

## 9. Visualization Section

### 9.1 The Core Pipeline

```
 ┌───────────┐   iter(obj)   ┌───────────┐   next()   ┌─────────┐
 │  Iterable │ ────────────► │  Iterator │ ─────────► │  Value  │
 └───────────┘               └───────────┘            └─────────┘
                                    │
                                    │ (repeat next() calls)
                                    ▼
                            ┌────────────────┐
                            │ StopIteration  │
                            │ (no more data) │
                            └────────────────┘
```

### 9.2 StopIteration Flow in a `for` Loop

```
   for item in iterable:
          │
          ▼
   it = iter(iterable)
          │
          ▼
  ┌──────────────────┐
  │   loop forever:   │
  │                   │
  │   try:            │
  │     item=next(it) │───► success ───► run loop BODY ───► back to top
  │   except          │
  │     StopIteration:│
  │     break         │───► exit loop cleanly, no exception surfaces to caller
  └──────────────────┘
```

### 9.3 Multi-Pass vs Single-Pass Visual

```
Multi-pass iterable (e.g., list):

   list ──iter()──► Iterator A (independent)
   list ──iter()──► Iterator B (independent)
        (both can run to completion separately)

Single-pass iterable (e.g., generator):

   generator ──iter()──► returns itself
   generator ──iter()──► returns itself (SAME object, already advanced)
        (second "pass" continues where the first left off, or is empty)
```

---

## 10. Comparison Tables

### 10.1 Iterable vs Iterator

| Feature | Iterable | Iterator |
|---|---|---|
| Required method(s) | `__iter__()` | `__iter__()` and `__next__()` |
| Holds traversal state? | No (typically) | Yes |
| Can produce multiple iterators? | Yes (if well-designed) | No (it *is* one) |
| Example | `list`, `tuple`, `str`, `dict` | `list_iterator`, generator, file object |
| Used directly in `for`? | Yes (via implicit `iter()`) | Yes (already satisfies protocol) |
| Re-traversable? | Often yes | No — single-use |

### 10.2 Iterator Protocol vs Iterable Protocol

| Feature | Iterator Protocol | Iterable Protocol |
|---|---|---|
| Methods required | `__iter__()` + `__next__()` | `__iter__()` only |
| Purpose | Defines *how to advance* through data | Defines *how to obtain* an iterator |
| Return of `__iter__()` | `self` | A new (or shared) iterator object |
| Statefulness | Inherently stateful | Often stateless (delegates state to iterator) |
| Analogy | The cursor/bookmark | The book itself |

### 10.3 `enumerate` vs `zip` vs `map` vs `filter`

| Tool | Input Count | Output Shape | Common Use |
|---|---|---|---|
| `enumerate` | 1 iterable | `(index, value)` pairs | Index-aware loops |
| `zip` | 2+ iterables | Tuples of aligned elements | Parallel iteration |
| `map` | 1 function + 1+ iterables | Transformed values | Apply a function elementwise |
| `filter` | 1 predicate + 1 iterable | Subset of original values | Keep matching elements |

---

## 11. Real-World Projects

### 11.1 Log Analyzer (streaming, line-by-line)

```python
import io

log_data = """2024-01-01 INFO Service started
2024-01-01 ERROR Connection failed
2024-01-01 WARNING Disk space low
2024-01-01 ERROR Timeout occurred"""

class LogAnalyzer:
    """Iterates over a log stream, yielding only lines matching a level."""
    def __init__(self, stream, level):
        self.stream = stream
        self.level = level

    def __iter__(self):
        for line in self.stream:
            if self.level in line:
                yield line.strip()

stream = io.StringIO(log_data)
analyzer = LogAnalyzer(stream, "ERROR")
for line in analyzer:
    print(line)
```

**Output:**
```
2024-01-01 ERROR Connection failed
2024-01-01 ERROR Timeout occurred
```

### 11.2 CSV Stream Processor

```python
import csv
import io

csv_data = """name,score
Alice,88
Bob,42
Carol,95"""

def passing_students(stream, threshold=50):
    reader = csv.DictReader(stream)
    for row in reader:
        if int(row["score"]) >= threshold:
            yield row["name"]

stream = io.StringIO(csv_data)
print(list(passing_students(stream)))
```

**Output:**
```
['Alice', 'Carol']
```

### 11.3 Pagination System (API-style)

```python
class APIResultsPaginator:
    """Simulates iterating through paginated API results."""
    def __init__(self, fetch_page_func, page_size=2):
        self.fetch_page_func = fetch_page_func
        self.page_size = page_size
        self.page_num = 0

    def __iter__(self):
        return self

    def __next__(self):
        page = self.fetch_page_func(self.page_num, self.page_size)
        if not page:
            raise StopIteration
        self.page_num += 1
        return page


def mock_fetch(page_num, page_size):
    all_data = list(range(1, 8))
    start = page_num * page_size
    chunk = all_data[start: start + page_size]
    return chunk

for page in APIResultsPaginator(mock_fetch, page_size=3):
    print(page)
```

**Output:**
```
[1, 2, 3]
[4, 5, 6]
[7]
```

### 11.4 Data Validation Pipeline

```python
records = [
    {"id": 1, "age": 25},
    {"id": 2, "age": -5},
    {"id": 3, "age": 200},
    {"id": 4, "age": 40},
]

def valid_records(records):
    for r in records:
        if 0 <= r["age"] <= 120:
            yield r

def invalid_records(records):
    for r in records:
        if not (0 <= r["age"] <= 120):
            yield r

print("Valid:", list(valid_records(records)))
print("Invalid:", list(invalid_records(records)))
```

**Output:**
```
Valid: [{'id': 1, 'age': 25}, {'id': 4, 'age': 40}]
Invalid: [{'id': 2, 'age': -5}, {'id': 3, 'age': 200}]
```

### 11.5 Batch Processing Engine

```python
def batcher(iterable, batch_size):
    """Groups any iterable into lists of batch_size (last batch may be smaller)."""
    batch = []
    for item in iterable:
        batch.append(item)
        if len(batch) == batch_size:
            yield batch
            batch = []
    if batch:
        yield batch


def process_batch(batch):
    return sum(batch)

data = range(1, 11)
for batch in batcher(data, 4):
    print(f"Batch: {batch} -> Sum: {process_batch(batch)}")
```

**Output:**
```
Batch: [1, 2, 3, 4] -> Sum: 10
Batch: [5, 6, 7, 8] -> Sum: 26
Batch: [9, 10] -> Sum: 19
```

---

## 12. Advanced Patterns

### 12.1 Lazy Evaluation

Lazy evaluation defers computation until the value is actually needed, enabling infinite or huge sequences to be processed safely.

```python
def lazy_primes():
    """Infinite lazy prime generator (simple trial division)."""
    n = 2
    while True:
        if all(n % p for p in range(2, int(n ** 0.5) + 1)):
            yield n
        n += 1

import itertools
print(list(itertools.islice(lazy_primes(), 5)))
```

**Output:**
```
[2, 3, 5, 7, 11]
```

### 12.2 Iterator Pipelines

Chain small, single-purpose generators into a readable pipeline — each stage stays lazy.

```python
def read_numbers(it):
    for x in it:
        yield int(x)

def filter_even(it):
    for x in it:
        if x % 2 == 0:
            yield x

def square(it):
    for x in it:
        yield x * x

raw = ["1", "2", "3", "4", "5", "6"]
pipeline = square(filter_even(read_numbers(raw)))
print(list(pipeline))
```

**Output:**
```
[4, 16, 36]
```

### 12.3 Streaming Data Processing

```python
def moving_average(stream, window=3):
    buffer = []
    for value in stream:
        buffer.append(value)
        if len(buffer) > window:
            buffer.pop(0)
        if len(buffer) == window:
            yield sum(buffer) / window

data = [1, 2, 3, 4, 5, 6]
print(list(moving_average(data, window=3)))
```

**Output:**
```
[2.0, 3.0, 4.0, 5.0]
```

### 12.4 Producer-Consumer Pattern (single-threaded, generator-based)

```python
def producer(n):
    for i in range(n):
        yield f"item-{i}"

def consumer(stream):
    for item in stream:
        print(f"Consuming {item}")

consumer(producer(3))
```

**Output:**
```
Consuming item-0
Consuming item-1
Consuming item-2
```

### 12.5 State-Based Iterators (Finite State Machine Example)

```python
class TrafficLight:
    """Cycles through states indefinitely, demonstrating stateful iteration."""
    STATES = ["GREEN", "YELLOW", "RED"]

    def __init__(self):
        self.index = 0

    def __iter__(self):
        return self

    def __next__(self):
        state = self.STATES[self.index % len(self.STATES)]
        self.index += 1
        return state

import itertools
light = TrafficLight()
print(list(itertools.islice(light, 7)))
```

**Output:**
```
['GREEN', 'YELLOW', 'RED', 'GREEN', 'YELLOW', 'RED', 'GREEN']
```

---

## 13. Cheat Sheets

### 13.1 `iter()` Cheat Sheet

| Form | Behavior |
|---|---|
| `iter(iterable)` | Calls `iterable.__iter__()`, returns an iterator |
| `iter(callable, sentinel)` | Calls `callable()` repeatedly until result equals `sentinel`, then raises `StopIteration` |

```python
import functools
counter = itertools_count = iter(lambda: 5, 0)  # never hits 0 here, illustrative only
```

```python
# Practical sentinel example: read lines until a blank line
import io
stream = io.StringIO("a\nb\n\nc\n")
for line in iter(stream.readline, "\n"):
    print(repr(line))
```
**Output:**
```
'a\n'
'b\n'
```

### 13.2 `next()` Cheat Sheet

| Form | Behavior |
|---|---|
| `next(iterator)` | Returns next value or raises `StopIteration` |
| `next(iterator, default)` | Returns next value, or `default` instead of raising |

### 13.3 Iterable Protocol Cheat Sheet

```python
class MyIterable:
    def __iter__(self):
        return MyIteratorInstance   # must return an Iterator
```

| Checklist | ✔ |
|---|---|
| Implements `__iter__()` | Required |
| Returns object with `__next__()` | Required |
| Returns a *new* iterator each call (for multi-pass) | Recommended |

### 13.4 Iterator Protocol Cheat Sheet

```python
class MyIterator:
    def __iter__(self):
        return self
    def __next__(self):
        ...
        raise StopIteration
```

| Checklist | ✔ |
|---|---|
| Implements `__iter__()` returning `self` | Required |
| Implements `__next__()` | Required |
| Raises `StopIteration` on exhaustion | Required |
| Stays exhausted after first `StopIteration` | Convention |

### 13.5 `StopIteration` Cheat Sheet

| Situation | Correct Action |
|---|---|
| End of custom iterator class | `raise StopIteration` inside `__next__()` |
| End of generator function | Use `return` (Python raises it for you) |
| Need a default instead of exception | `next(it, default)` |
| Want generator's return value | Catch `StopIteration as e`, read `e.value` |
| Accidentally raising it mid-generator-body | Avoid — becomes `RuntimeError` (PEP 479) |

---

## 14. Interview Master Section

### 14.1 Conceptual Questions

**Q1. What is the difference between an iterable and an iterator?**
A: An iterable is anything `iter()` can be called on to get an iterator; an iterator is the object that actually produces values one at a time via `next()` and tracks position. A list is iterable but not itself an iterator.

**Q2. Can an object be both an iterable and an iterator at the same time?**
A: Yes — if its `__iter__()` returns `self`. Generators are the classic example.

**Q3. What two methods make up the Iterator Protocol?**
A: `__iter__()` and `__next__()`.

**Q4. What single method makes up the Iterable Protocol?**
A: `__iter__()`.

**Q5. What exception signals the end of iteration, and who is responsible for raising it?**
A: `StopIteration`. The iterator's `__next__()` (or the generator runtime, automatically) raises it.

**Q6. What's the conceptual equivalent of a `for` loop in terms of `iter()`/`next()`?**
A:
```python
it = iter(x)
while True:
    try:
        v = next(it)
    except StopIteration:
        break
    # loop body
```

**Q7. Why can you only loop over a generator once?**
A: A generator's `__iter__()` returns itself, and its internal execution state (the suspended frame) advances permanently with each `next()` call; there is no way to rewind it.

**Q8. What happens when you call `next()` on a list directly?**
A: `TypeError: 'list' object is not an iterator` — lists are iterable but are not iterators themselves.

**Q9. Are dictionaries ordered when iterated?**
A: Yes, since Python 3.7+, dictionaries preserve insertion order during iteration (a language guarantee, not just an implementation detail).

**Q10. What does iterating a dictionary directly give you?**
A: Its keys, by default. Use `.values()`, `.items()`, or `.keys()` explicitly for clarity.

### 14.2 Tricky / Gotcha Questions

**Q11. Why does this print an empty list the second time?**
```python
gen = (x for x in range(3))
print(list(gen))
print(list(gen))
```
A: Generators are single-pass iterators; once exhausted, they cannot restart. The second `list()` call sees no more items.

**Q12. What's wrong with this custom iterable?**
```python
class Bad:
    def __init__(self):
        self.data = [1, 2, 3]
        self.i = 0
    def __iter__(self):
        return self
    def __next__(self):
        if self.i >= len(self.data):
            raise StopIteration
        v = self.data[self.i]
        self.i += 1
        return v
```
A: Nothing is technically "wrong" for single-pass use, but it cannot support multiple independent traversals — calling `iter()` on it twice returns the same exhausted object the second time. If multi-pass behavior is needed, separate the Iterable from the Iterator.

**Q13. Why does `zip([1,2,3], [1,2])` only produce 2 pairs?**
A: `zip` stops as soon as the shortest input iterable is exhausted; it doesn't pad or raise an error by default.

**Q14. What happens if `__next__()` returns `None` to signal "done" instead of raising `StopIteration`?**
A: The loop won't terminate correctly — `None` is treated as a perfectly valid value, so the `for` loop will yield `None` forever (since `__next__()` keeps getting called), or until some other failure occurs. Always raise `StopIteration`, never return a sentinel.

**Q15. Is `range(10)` an iterator?**
A: No, it's an iterable — it supports `len()`, indexing, and repeated iteration, none of which iterators support. `iter(range(10))` returns a separate `range_iterator` object.

**Q16. Why does manually raising `StopIteration` inside a generator function body raise a `RuntimeError` instead?**
A: PEP 479 changed this specifically to prevent a bug class where an *accidental* `StopIteration` deep inside a generator (e.g., from a nested `next()` call) silently terminated an outer loop instead of surfacing as an error.

**Q17. What's the output and why?**
```python
it = iter([1, 2, 3])
for x in it:
    if x == 2:
        break
print(list(it))
```
A: `[3]` — breaking out of the loop doesn't reset the iterator; the for-loop already consumed `1` and `2`, so only `3` remains.

**Q18. Can you call `len()` on an iterator?**
A: Generally no — most iterators don't define `__len__()` because their total count may be unknown or infinite (e.g., a generator). Some specific iterator types may, but it's not part of the protocol.

**Q19. Does `for` loop catch every `StopIteration`, even ones raised by code unrelated to the loop's own iterator?**
A: No — only the `StopIteration` raised by the loop's own `next(it)` call (the one the `for` machinery itself issues) is caught. A `StopIteration` raised elsewhere inside the loop body propagates as a normal exception.

**Q20. What does `next(it, "default")` do if `it` is not actually exhausted?**
A: It simply returns the next available value as normal — the default is only used if `StopIteration` would otherwise have been raised.

### 14.3 Applied / Coding Questions

**Q21. Write code that proves `enumerate` is lazy.**
```python
e = enumerate([10, 20, 30])
print(next(e))
print(next(e))
```
**Output:**
```
(0, 10)
(1, 20)
```

**Q22. How would you implement an infinite iterator safely consumed in a `for` loop?**
A: Pair it with `break`, or use `itertools.islice(infinite_iter, n)` to bound consumption.

**Q23. How do you create two independent consumers from a single generator?**
A: Use `itertools.tee()`.
```python
import itertools
g = (x for x in range(3))
g1, g2 = itertools.tee(g)
print(list(g1), list(g2))
```
**Output:**
```
[0, 1, 2] [0, 1, 2]
```

**Q24. How do you reverse-iterate a custom sequence without copying it?**
A: Implement `__reversed__()` on the class, or implement `__len__()` + `__getitem__()` so the built-in `reversed()` fallback works.

**Q25. Write a one-liner that checks if all items in a (possibly huge/lazy) iterable are positive, without converting it to a list.**
```python
data = (x for x in [1, 2, 3, -4])
print(all(x > 0 for x in data))
```
**Output:**
```
False
```

**Q26. What does `next(iter({}))` do?**
A: Calling `iter({})` gives an empty dict's iterator; calling `next()` on it immediately raises `StopIteration` since there are no keys.

**Q27. How would you chain several iterables into one without copying their contents?**
```python
import itertools
combined = itertools.chain([1, 2], (3, 4), "56")
print(list(combined))
```
**Output:**
```
[1, 2, 3, 4, '5', '6']
```

**Q28. How can you group consecutive duplicate elements in an iterable lazily?**
```python
import itertools
data = [1, 1, 2, 2, 2, 3, 1]
for key, group in itertools.groupby(data):
    print(key, list(group))
```
**Output:**
```
1 [1, 1]
2 [2, 2, 2]
3 [3]
1 [1]
```

**Q29. How would you implement your own `enumerate()` using only the iterator protocol?**
```python
def my_enumerate(iterable, start=0):
    it = iter(iterable)
    index = start
    while True:
        try:
            value = next(it)
        except StopIteration:
            return
        yield index, value
        index += 1

print(list(my_enumerate(["a", "b"], start=1)))
```
**Output:**
```
[(1, 'a'), (2, 'b')]
```

**Q30. Why is `sum(x*x for x in range(10**8))` generally preferable memory-wise to `sum([x*x for x in range(10**8)])`?**
A: The generator expression evaluates lazily, holding only one value in memory at a time and feeding it directly to `sum()`, while the list comprehension materializes all 10^8 squared values into memory before summing.

**Q31. How do you detect, without exhausting it, whether a given object is an iterator (not just an iterable)?**
```python
from collections.abc import Iterator
print(isinstance(iter([1,2,3]), Iterator))   # True
print(isinstance([1,2,3], Iterator))         # False
```
**Output:**
```
True
False
```

**Q32. What's the output and why?**
```python
def gen():
    print("start")
    yield 1
    print("middle")
    yield 2

g = gen()
print("created")
print(next(g))
print(next(g))
```
A: Generator bodies don't execute *any* code until the first `next()` call — this proves laziness.
**Output:**
```
created
start
1
middle
2
```

---

## 15. Exercises

### 15.1 Intermediate (15)

**1. Write a function that returns the sum of a generator without converting it to a list.**
```python
def sum_gen(gen):
    total = 0
    for x in gen:
        total += x
    return total

print(sum_gen(x for x in range(5)))
```
**Output:** `10`

**2. Implement `__iter__`/`__next__` for a class that yields characters of a string in reverse.**
```python
class ReverseString:
    def __init__(self, text):
        self.text = text
        self.index = len(text)
    def __iter__(self):
        return self
    def __next__(self):
        if self.index == 0:
            raise StopIteration
        self.index -= 1
        return self.text[self.index]

print("".join(ReverseString("hello")))
```
**Output:** `olleh`

**3. Use `next()` with a default to safely get the first element of a possibly-empty list.**
```python
def first_or_default(lst, default=None):
    return next(iter(lst), default)

print(first_or_default([]))
print(first_or_default([7, 8]))
```
**Output:**
```
None
7
```

**4. Write a generator that yields squares of even numbers from 1 to n.**
```python
def even_squares(n):
    for x in range(1, n + 1):
        if x % 2 == 0:
            yield x * x

print(list(even_squares(10)))
```
**Output:** `[4, 16, 36, 64, 100]`

**5. Demonstrate manual iteration of a tuple using `iter()`/`next()`.**
```python
t = (10, 20, 30)
it = iter(t)
print(next(it), next(it), next(it))
```
**Output:** `10 20 30`

**6. Build a class-based iterable for a deck of card suits that supports multiple passes.**
```python
class Suits:
    def __iter__(self):
        return iter(["Hearts", "Diamonds", "Clubs", "Spades"])

s = Suits()
print(list(s))
print(list(s))
```
**Output:**
```
['Hearts', 'Diamonds', 'Clubs', 'Spades']
['Hearts', 'Diamonds', 'Clubs', 'Spades']
```

**7. Use `enumerate` to print 1-based line numbers for a list of strings.**
```python
lines = ["first", "second", "third"]
for i, line in enumerate(lines, start=1):
    print(f"{i}: {line}")
```
**Output:**
```
1: first
2: second
3: third
```

**8. Write a function using `filter()` to remove falsy values from a list.**
```python
data = [0, 1, "", "hi", None, 5, False]
print(list(filter(None, data)))
```
**Output:** `[1, 'hi', 5]`

**9. Implement a countdown timer iterator from N to 1 (no zero).**
```python
class CountdownTimer:
    def __init__(self, n):
        self.n = n
    def __iter__(self):
        return self
    def __next__(self):
        if self.n <= 0:
            raise StopIteration
        val = self.n
        self.n -= 1
        return val

print(list(CountdownTimer(5)))
```
**Output:** `[5, 4, 3, 2, 1]`

**10. Show that `sorted()` works on any iterable, not just lists.**
```python
print(sorted((x for x in [3, 1, 2])))
print(sorted("dcba"))
```
**Output:**
```
[1, 2, 3]
['a', 'b', 'c', 'd']
```

**11. Use `zip` to merge three parallel lists into a list of dicts.**
```python
keys = ["name", "age", "city"]
vals = [["Tom", 25, "NYC"], ["Ann", 30, "LA"]]
result = [dict(zip(keys, v)) for v in vals]
print(result)
```
**Output:** `[{'name': 'Tom', 'age': 25, 'city': 'NYC'}, {'name': 'Ann', 'age': 30, 'city': 'LA'}]`

**12. Write a generator function for the first n triangular numbers (1, 3, 6, 10...).**
```python
def triangular(n):
    total = 0
    for i in range(1, n + 1):
        total += i
        yield total

print(list(triangular(5)))
```
**Output:** `[1, 3, 6, 10, 15]`

**13. Detect manually (without `isinstance`) whether an object supports iteration, using `try/except`.**
```python
def supports_iteration(obj):
    try:
        iter(obj)
        return True
    except TypeError:
        return False

print(supports_iteration(123))
print(supports_iteration([1, 2]))
```
**Output:**
```
False
True
```

**14. Write code to print every other element of a list using `itertools.islice`.**
```python
import itertools
data = list(range(10))
print(list(itertools.islice(data, 0, None, 2)))
```
**Output:** `[0, 2, 4, 6, 8]`

**15. Convert a finite generator into a list, then prove it's now empty when reused.**
```python
def gen():
    yield 1
    yield 2

g = gen()
print(list(g))
print(list(g))
```
**Output:**
```
[1, 2]
[]
```

### 15.2 Advanced (15)

**1. Build a custom Range-like iterable supporting negative steps.**
```python
class MyRange:
    def __init__(self, start, stop, step=1):
        self.start, self.stop, self.step = start, stop, step
    def __iter__(self):
        current = self.start
        if self.step > 0:
            while current < self.stop:
                yield current
                current += self.step
        else:
            while current > self.stop:
                yield current
                current += self.step

print(list(MyRange(10, 0, -2)))
```
**Output:** `[10, 8, 6, 4, 2]`

**2. Implement a sliding window iterator over any iterable.**
```python
from collections import deque

def sliding_window(iterable, size):
    it = iter(iterable)
    window = deque(maxlen=size)
    for _ in range(size):
        window.append(next(it))
    yield tuple(window)
    for item in it:
        window.append(item)
        yield tuple(window)

print(list(sliding_window([1, 2, 3, 4, 5], 3)))
```
**Output:** `[(1, 2, 3), (2, 3, 4), (3, 4, 5)]`

**3. Build a peekable iterator wrapper supporting `.peek()` without consuming.**
```python
class PeekableIterator:
    _SENTINEL = object()
    def __init__(self, iterable):
        self._it = iter(iterable)
        self._cache = self._SENTINEL
    def __iter__(self):
        return self
    def __next__(self):
        if self._cache is not self._SENTINEL:
            value, self._cache = self._cache, self._SENTINEL
            return value
        return next(self._it)
    def peek(self, default=_SENTINEL):
        if self._cache is self._SENTINEL:
            try:
                self._cache = next(self._it)
            except StopIteration:
                if default is self._SENTINEL:
                    raise
                return default
        return self._cache

p = PeekableIterator([1, 2, 3])
print(p.peek())
print(next(p))
print(next(p))
```
**Output:**
```
1
1
2
```

**4. Implement `itertools.chain` manually using a generator.**
```python
def my_chain(*iterables):
    for it in iterables:
        for item in it:
            yield item

print(list(my_chain([1, 2], (3, 4), "ab")))
```
**Output:** `[1, 2, 3, 4, 'a', 'b']`

**5. Write a generator-based flatten function for arbitrarily nested lists.**
```python
def flatten(nested):
    for item in nested:
        if isinstance(item, list):
            yield from flatten(item)
        else:
            yield item

print(list(flatten([1, [2, [3, 4], 5], 6])))
```
**Output:** `[1, 2, 3, 4, 5, 6]`

**6. Build an iterator that yields running totals (cumulative sum) lazily.**
```python
def running_total(iterable):
    total = 0
    for x in iterable:
        total += x
        yield total

print(list(running_total([1, 2, 3, 4])))
```
**Output:** `[1, 3, 6, 10]`

**7. Implement a retry-iterator that yields attempt numbers until a condition succeeds (simulate with a counter).**
```python
def retry_attempts(success_on_attempt, max_attempts=5):
    for attempt in range(1, max_attempts + 1):
        yield attempt
        if attempt == success_on_attempt:
            return

for attempt in retry_attempts(3):
    print(f"Attempt {attempt}")
```
**Output:**
```
Attempt 1
Attempt 2
Attempt 3
```

**8. Build a custom iterator that yields prime numbers up to a limit using a class.**
```python
class Primes:
    def __init__(self, limit):
        self.limit = limit
        self.current = 1
    def __iter__(self):
        return self
    def __next__(self):
        self.current += 1
        while self.current <= self.limit:
            if all(self.current % d for d in range(2, int(self.current ** 0.5) + 1)):
                return self.current
            self.current += 1
        raise StopIteration

print(list(Primes(20)))
```
**Output:** `[2, 3, 5, 7, 11, 13, 17, 19]`

**9. Implement `itertools.tee`-like behavior manually using caching.**
```python
def manual_tee(iterable, n=2):
    it = iter(iterable)
    caches = [[] for _ in range(n)]
    def gen(cache):
        while True:
            if not cache:
                try:
                    value = next(it)
                except StopIteration:
                    return
                for c in caches:
                    c.append(value)
            yield cache.pop(0)
    return tuple(gen(c) for c in caches)

a, b = manual_tee([1, 2, 3])
print(next(a), next(a), list(b))
```
**Output:** `1 2 [1, 2, 3]`

**10. Write a generator that paginates a dictionary's items.**
```python
def paginate_dict(d, page_size):
    items = list(d.items())
    for i in range(0, len(items), page_size):
        yield dict(items[i:i + page_size])

d = {f"k{i}": i for i in range(5)}
print(list(paginate_dict(d, 2)))
```
**Output:** `[{'k0': 0, 'k1': 1}, {'k2': 2, 'k3': 3}, {'k4': 4}]`

**11. Implement a debounced iterator that skips consecutive duplicate values.**
```python
def dedupe_consecutive(iterable):
    sentinel = object()
    last = sentinel
    for item in iterable:
        if item != last:
            yield item
            last = item

print(list(dedupe_consecutive([1, 1, 2, 2, 1, 3, 3, 3])))
```
**Output:** `[1, 2, 1, 3]`

**12. Write an iterator wrapper that counts how many items have been consumed.**
```python
class CountingIterator:
    def __init__(self, iterable):
        self._it = iter(iterable)
        self.count = 0
    def __iter__(self):
        return self
    def __next__(self):
        value = next(self._it)
        self.count += 1
        return value

ci = CountingIterator([10, 20, 30])
print(next(ci), next(ci), ci.count)
```
**Output:** `10 20 2`

**13. Implement `zip_longest` manually using `itertools` building blocks (no library shortcut).**
```python
def my_zip_longest(*iterables, fillvalue=None):
    iterators = [iter(it) for it in iterables]
    while True:
        result = []
        active = False
        for it in iterators:
            try:
                result.append(next(it))
                active = True
            except StopIteration:
                result.append(fillvalue)
        if not active:
            return
        yield tuple(result)

print(list(my_zip_longest([1, 2, 3], ["a", "b"], fillvalue="-")))
```
**Output:** `[(1, 'a'), (2, 'b'), (3, '-')]`

**14. Build a lazy batch-and-process pipeline that computes averages per batch.**
```python
def batch_average(iterable, batch_size):
    batch = []
    for x in iterable:
        batch.append(x)
        if len(batch) == batch_size:
            yield sum(batch) / len(batch)
            batch = []
    if batch:
        yield sum(batch) / len(batch)

print(list(batch_average([10, 20, 30, 40, 50], 2)))
```
**Output:** `[15.0, 35.0, 50.0]`

**15. Implement an iterator that raises a custom exception (subclass of StopIteration is forbidden in modern Python) cleanly when exhausted, with a final summary.**
```python
class SummarizedIterator:
    def __init__(self, data):
        self.data = iter(data)
        self.total_seen = 0
    def __iter__(self):
        return self
    def __next__(self):
        try:
            value = next(self.data)
        except StopIteration:
            print(f"Done. Total items seen: {self.total_seen}")
            raise
        self.total_seen += 1
        return value

for x in SummarizedIterator([1, 2, 3]):
    pass
```
**Output:** `Done. Total items seen: 3`

### 15.3 Expert (15)

**1. Implement a coroutine-style producer-consumer using two generators communicating via `.send()`.**
```python
def consumer():
    total = 0
    while True:
        value = yield total
        if value is None:
            break
        total += value

c = consumer()
next(c)
print(c.send(5))
print(c.send(10))
```
**Output:**
```
5
15
```

**2. Build a generator-based finite state machine that processes a token stream.**
```python
def fsm(tokens):
    state = "START"
    for token in tokens:
        if state == "START" and token == "BEGIN":
            state = "RUNNING"
            yield state
        elif state == "RUNNING" and token == "END":
            state = "STOPPED"
            yield state
        elif state == "RUNNING":
            yield f"processing:{token}"

print(list(fsm(["BEGIN", "x", "y", "END"])))
```
**Output:** `['RUNNING', 'processing:x', 'processing:y', 'STOPPED']`

**3. Implement a lazy memoizing iterator (caches results so re-iteration via `iter()` is cheap after first full pass).**
```python
class CachingIterable:
    def __init__(self, iterable):
        self._source = iter(iterable)
        self._cache = []
        self._exhausted = False
    def __iter__(self):
        return self._gen()
    def _gen(self):
        for item in self._cache:
            yield item
        if not self._exhausted:
            for item in self._source:
                self._cache.append(item)
                yield item
            self._exhausted = True

ci = CachingIterable(x * x for x in range(4))
print(list(ci))
print(list(ci))   # second pass reuses cache, source already exhausted
```
**Output:**
```
[0, 1, 4, 9]
[0, 1, 4, 9]
```

**4. Build a generator-based depth-first tree traversal iterator for nested dict trees.**
```python
def dfs(tree, path=""):
    for key, value in tree.items():
        full_path = f"{path}/{key}" if path else key
        yield full_path
        if isinstance(value, dict):
            yield from dfs(value, full_path)

tree = {"root": {"a": {"a1": {}}, "b": {}}}
print(list(dfs(tree)))
```
**Output:** `['root', 'root/a', 'root/a/a1', 'root/b']`

**5. Implement an async-flavored simulation: an iterator that yields control back via `yield` while tracking "ticks" for a cooperative scheduler.**
```python
def task(name, ticks):
    for t in range(ticks):
        yield f"{name} tick {t}"

def scheduler(tasks):
    iterators = [iter(t) for t in tasks]
    while iterators:
        for it in iterators[:]:
            try:
                print(next(it))
            except StopIteration:
                iterators.remove(it)

scheduler([task("A", 2), task("B", 1)])
```
**Output:**
```
A tick 0
B tick 0
A tick 1
```

**6. Implement a generator-based rate limiter that only yields up to N items per "tick" (simulated, no real timing).**
```python
def rate_limited(iterable, per_tick):
    it = iter(iterable)
    while True:
        batch = []
        for _ in range(per_tick):
            try:
                batch.append(next(it))
            except StopIteration:
                if batch:
                    yield batch
                return
        yield batch

print(list(rate_limited(range(7), 3)))
```
**Output:** `[[0, 1, 2], [3, 4, 5], [6]]`

**7. Build a custom iterator-based LRU-ish cache iterator that yields (key, value, was_evicted) tuples while iterating insertions.**
```python
from collections import OrderedDict

def lru_trace(insertions, capacity):
    cache = OrderedDict()
    for key, value in insertions:
        evicted = None
        if key in cache:
            cache.move_to_end(key)
        cache[key] = value
        if len(cache) > capacity:
            evicted = cache.popitem(last=False)
        yield key, value, evicted

trace = list(lru_trace([("a",1), ("b",2), ("c",3), ("a",4)], capacity=2))
for entry in trace:
    print(entry)
```
**Output:**
```
('a', 1, None)
('b', 2, None)
('c', 3, ('a', 1))
('a', 4, ('b', 2))
```

**8. Implement `itertools.groupby`-equivalent manually without importing itertools.**
```python
def my_groupby(iterable, key=lambda x: x):
    it = iter(iterable)
    try:
        current = next(it)
    except StopIteration:
        return
    current_key = key(current)
    group = [current]
    for item in it:
        k = key(item)
        if k == current_key:
            group.append(item)
        else:
            yield current_key, group
            current_key, group = k, [item]
    yield current_key, group

data = [1, 1, 2, 2, 2, 3]
print([(k, g) for k, g in my_groupby(data)])
```
**Output:** `[(1, [1, 1]), (2, [2, 2, 2]), (3, [3])]`

**9. Implement a generator-based exponential backoff sequence generator with a max cap.**
```python
def backoff(base=1, factor=2, cap=30, attempts=6):
    delay = base
    for _ in range(attempts):
        yield min(delay, cap)
        delay *= factor

print(list(backoff()))
```
**Output:** `[1, 2, 4, 8, 16, 30]`

**10. Build a two-way generator pipeline where a downstream consumer can request a "skip n" via `.send()`.**
```python
def skippable_source(data):
    it = iter(data)
    skip = 0
    for item in it:
        if skip > 0:
            skip -= 1
            continue
        skip = yield item
        skip = skip or 0

g = skippable_source([1, 2, 3, 4, 5, 6])
print(next(g))
print(g.send(2))   # skip next 2 items
print(next(g))
```
**Output:**
```
1
4
5
```

**11. Implement a context-manager-based iterator that guarantees resource cleanup even on early `break`.**
```python
class ManagedResource:
    def __init__(self, data):
        self.data = data
    def __enter__(self):
        print("Resource opened")
        return iter(self.data)
    def __exit__(self, *exc):
        print("Resource closed")
        return False

with ManagedResource([1, 2, 3, 4]) as stream:
    for x in stream:
        print(x)
        if x == 2:
            break
```
**Output:**
```
Resource opened
1
2
Resource closed
```

**12. Implement a fan-out/fan-in pattern: split one iterator's items by predicate into two independent lazy streams using buffering.**
```python
def fan_out(iterable, predicate):
    true_buf, false_buf = [], []
    it = iter(iterable)
    def make_gen(own_buf, other_buf, want_true):
        while True:
            if own_buf:
                yield own_buf.pop(0)
                continue
            try:
                item = next(it)
            except StopIteration:
                return
            if predicate(item) == want_true:
                yield item
            else:
                other_buf.append(item)
    return make_gen(true_buf, false_buf, True), make_gen(false_buf, true_buf, False)

evens, odds = fan_out(range(10), lambda x: x % 2 == 0)
print(list(evens))
print(list(odds))
```
**Output:**
```
[0, 2, 4, 6, 8]
[1, 3, 5, 7, 9]
```

**13. Implement an iterator-based topological sort generator over a small DAG.**
```python
def topo_sort(graph):
    in_degree = {n: 0 for n in graph}
    for deps in graph.values():
        for d in deps:
            in_degree[d] += 1
    ready = [n for n, deg in in_degree.items() if deg == 0]
    while ready:
        node = ready.pop(0)
        yield node
        for dep in graph[node]:
            in_degree[dep] -= 1
            if in_degree[dep] == 0:
                ready.append(dep)

graph = {"a": ["b"], "b": ["c"], "c": []}
print(list(topo_sort(graph)))
```
**Output:** `['a', 'b', 'c']`

**14. Build a generator that simulates a producer-consumer queue with backpressure (bounded buffer, simulated synchronously).**
```python
def bounded_producer_consumer(items, buffer_size):
    buffer = []
    it = iter(items)
    exhausted = False
    while True:
        while len(buffer) < buffer_size and not exhausted:
            try:
                buffer.append(next(it))
            except StopIteration:
                exhausted = True
        if not buffer:
            return
        yield buffer.pop(0)

print(list(bounded_producer_consumer(range(6), buffer_size=2)))
```
**Output:** `[0, 1, 2, 3, 4, 5]`

**15. Implement a custom iterator that raises `StopIteration` with a meaningful payload, retrievable cleanly by the caller (mirrors generator `return` semantics for class-based iterators).**
```python
class PayloadIterator:
    def __init__(self, data):
        self.data = iter(data)
        self.processed = 0
    def __iter__(self):
        return self
    def __next__(self):
        try:
            value = next(self.data)
        except StopIteration:
            raise StopIteration(f"processed={self.processed}")
        self.processed += 1
        return value

it = PayloadIterator([1, 2, 3])
results = []
try:
    while True:
        results.append(next(it))
except StopIteration as e:
    print(results)
    print(e.value)
```
**Output:**
```
[1, 2, 3]
processed=3
```

---

## 16. Final Section

### 16.1 Summary

| Concept | One-Line Takeaway |
|---|---|
| Iterable | Anything `iter()` can be called on to get an iterator |
| Iterator | A stateful cursor producing values via `next()` |
| Iterable Protocol | Implement `__iter__()` returning an iterator |
| Iterator Protocol | Implement `__iter__()` (returns `self`) + `__next__()` |
| `iter()` | Entry point that converts an iterable into an iterator |
| `next()` | Pulls the next value, or raises/returns default on exhaustion |
| `StopIteration` | The universal "no more data" signal, caught automatically by `for` |
| Generators | The easiest way to get the Iterator Protocol "for free" via `yield` |
| `itertools` | The toolbox for composing lazy, memory-efficient pipelines |

### 16.2 Best Practices Checklist

- [ ] Use generators or generator expressions for large/streaming/infinite data.
- [ ] Separate Iterable (container) from Iterator (cursor) when multiple independent passes are required.
- [ ] Always raise `StopIteration` (never return a sentinel) to end custom `__next__()` implementations.
- [ ] Prefer `return` over `raise StopIteration` inside generator function bodies.
- [ ] Use `next(it, default)` to avoid try/except boilerplate when a default is acceptable.
- [ ] Reach for `itertools` (`islice`, `chain`, `tee`, `groupby`, `zip_longest`, `accumulate`) before hand-rolling iteration logic.
- [ ] Close external resources (files, sockets) deterministically, ideally via context managers, even in custom iterators.
- [ ] Bound any infinite iterator's consumption explicitly (`islice`, `break`, or a counter).

### 16.3 Common Mistakes Checklist

- [ ] Treating a generator or file object as re-iterable when it is single-pass.
- [ ] Calling `next()` directly on a plain iterable (list, tuple, dict) instead of on an iterator obtained via `iter()`.
- [ ] Returning `None`/sentinel values instead of raising `StopIteration` in custom iterators.
- [ ] Manually raising `StopIteration` inside a generator body instead of using `return` (triggers `RuntimeError` per PEP 479).
- [ ] Assuming `zip()` pads to the longest iterable (it truncates to the shortest by default).
- [ ] Forgetting that breaking out of a `for` loop leaves the iterator partially consumed, not reset.
- [ ] Letting an infinite iterator run unbounded in production code.

### 16.4 Performance Tips

| Goal | Recommended Approach |
|---|---|
| Process data larger than memory | Generators / generator expressions / file line iteration |
| Need fast multiple random-access reads | Materialize into a `list` once, accept O(n) memory |
| Need to short-circuit a boolean check | `any()` / `all()` (auto short-circuit) instead of manual loops |
| Need to combine several lazy stages | Chain generator functions / `itertools` rather than intermediate lists |
| Need a single value, not the whole result | `next(it, default)` instead of `list(it)[0]` |
| Need multiple independent consumers of one stream | `itertools.tee()` rather than re-running the source logic |

### 16.5 Learning Roadmap

```
1. Master built-in iterables (list, tuple, str, dict, set, range)
        │
        ▼
2. Understand iter() / next() and manual loop equivalence
        │
        ▼
3. Learn the Iterable & Iterator Protocols (dunder methods)
        │
        ▼
4. Write your first class-based custom iterator
        │
        ▼
5. Switch to generator functions (yield) for simpler iterators
        │
        ▼
6. Master StopIteration semantics, including PEP 479 nuances
        │
        ▼
7. Learn itertools: islice, chain, tee, groupby, zip_longest, accumulate
        │
        ▼
8. Build real lazy pipelines: streaming, batching, pagination
        │
        ▼
9. Tackle advanced patterns: coroutines (.send()), FSMs, producer-consumer
        │
        ▼
10. Apply everything to real-world systems: log analysis, ETL, APIs
```

---

*End of Python Iteration Internals Master Guide.*
