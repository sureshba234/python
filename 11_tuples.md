# 11. Tuples — Complete Reference

A `tuple` is an **immutable, ordered** sequence — like a `list` that can never change shape or contents after creation. That single restriction (immutability) is what unlocks everything distinctive about tuples: they're cheaper to store, safe to use as dict keys, and they're the mechanism behind one of Python's most-used pieces of syntax — multiple assignment.

```python
point = (3, 4)          # a tuple
point[0] = 99            # TypeError: 'tuple' object does not support item assignment
```

---

## Table of Contents

1. [Packing](#1-packing)
2. [Unpacking](#2-unpacking)
3. [Memory Optimization](#3-memory-optimization)
4. [Named Tuples](#4-named-tuples)

---

## 1. Packing

### 1.1 What "Packing" Means

**Packing** is the act of taking several separate values and bundling them into a single tuple. You've actually been doing this every time you write a comma-separated list of values — the **parentheses are optional**; it's the **comma** that creates the tuple.

```python
>>> point = 3, 4
>>> point
(3, 4)
>>> type(point)
<class 'tuple'>
```

```python
>>> point = (3, 4)    # identical result — parens just add clarity
>>> point
(3, 4)
```

### 1.2 The Comma Is What Matters, Not the Parentheses

This trips up almost everyone at least once:

```python
>>> not_a_tuple = (5)      # just an int in parens — NO comma, NOT a tuple
>>> type(not_a_tuple)
<class 'int'>
>>> single_tuple = (5,)    # the trailing comma makes it a 1-element tuple
>>> type(single_tuple)
<class 'tuple'>
>>> single_tuple = 5,      # parens aren't even needed
>>> type(single_tuple)
<class 'tuple'>
```

### 1.3 Functions "Pack" Multiple Return Values Automatically

When a function appears to "return multiple values," what's actually happening is that the `return` statement packs them into one tuple:

```python
def minmax(values):
    return min(values), max(values)    # packs two values into a tuple

result = minmax([3, 1, 4, 1, 5, 9])
print(result)          # (1, 9)
print(type(result))    # <class 'tuple'>
```

If you write `lo, hi = minmax(...)`, you're packing on the `return` line and then immediately unpacking on the assignment line (see [Section 2](#2-unpacking)) — the two operations are mirror images of each other.

### 1.4 `*args` Packs Positional Arguments Into a Tuple

Inside a function, a parameter prefixed with `*` collects any number of extra positional arguments into a tuple:

```python
def f(*args):
    print(args, type(args))

f(1, 2, 3)
# (1, 2, 3) <class 'tuple'>
```

This is the *same* packing mechanism as Section 1.1 — `*args` is just packing applied automatically by the function-call machinery, which is why `args` always comes out as a `tuple`, never a `list`.

### 1.5 Packing an Arbitrary Number of Values With `*`

You can also pack "everything else" into a tuple during assignment — covered fully under unpacking, since it's the same `*` syntax used in reverse (see [Section 2.3](#23-starred-unpacking-collecting-the-rest)).

---

## 2. Unpacking

### 2.1 Basic Unpacking

**Unpacking** is the reverse of packing: taking a tuple (or any iterable) and assigning its elements to separate variable names in one statement.

```python
>>> point = (3, 4)
>>> x, y = point
>>> x
3
>>> y
4
```

You can skip the intermediate variable entirely:

```python
>>> x, y = 3, 4     # pack on the right, unpack on the left, in one line
```

**The number of variables must match the number of elements**, or Python raises a `ValueError`:

```python
>>> a, b = (1, 2, 3)
ValueError: too many values to unpack (expected 2)
>>> a, b, c = (1, 2)
ValueError: not enough values to unpack (expected 3, got 2)
```

### 2.2 The Classic Use Case: Swapping Variables

```python
>>> a, b = 5, 10
>>> a, b = b, a       # the right side is fully packed into a temp tuple FIRST,
>>> a, b              # then unpacked into a and b — no manual temp variable needed
(10, 5)
```

This works because Python evaluates the entire right-hand side (packing `b, a` into a temporary tuple) **before** performing any assignment — so there's no risk of `a`'s new value contaminating what gets assigned to `b`.

### 2.3 Starred Unpacking: Collecting "The Rest"

A single name prefixed with `*` on the left-hand side soaks up **any number of leftover elements** into a list (note: a `list`, not a tuple, when used this way):

```python
>>> first, *middle, last = [1, 2, 3, 4, 5]
>>> first
1
>>> middle
[2, 3, 4]
>>> last
5
```

The starred name can be in any position — at the start, middle, or end:

```python
>>> *rest, last = [1, 2, 3, 4]
>>> rest, last
([1, 2, 3], 4)

>>> first, *rest = [1, 2, 3, 4]
>>> first, rest
(1, [2, 3, 4])
```

If there's nothing left over, the starred variable becomes an **empty list**, not an error:

```python
>>> a, *b = [1]
>>> a, b
(1, [])
```

> Only **one** starred expression is allowed per unpacking assignment — Python can't infer how to split "the rest" between two unknowns.

### 2.4 Nested Unpacking

If an element of the iterable is itself a tuple (or other iterable), you can unpack it in place by mirroring its structure on the left:

```python
>>> x, (y, z) = 1, (2, 3)
>>> x, y, z
(1, 2, 3)

>>> data = [("Suresh", (23, "Hyderabad")), ("Asha", (29, "Pune"))]
>>> for name, (age, city) in data:
...     print(name, age, city)
Suresh 23 Hyderabad
Asha 29 Pune
```

### 2.5 Unpacking in `for` Loops

This is one of the most common places unpacking shows up in real code — every iteration of a `for` loop over pairs/tuples is an unpacking assignment:

```python
pairs = [(1, "a"), (2, "b"), (3, "c")]
for number, letter in pairs:
    print(number, letter)

# Extremely common with dict.items():
scores = {"Suresh": 91, "Asha": 88}
for name, score in scores.items():
    print(f"{name}: {score}")
```

### 2.6 Unpacking Into Function Calls

The `*` operator also unpacks an iterable directly into a function call's positional arguments:

```python
>>> def add(a, b, c):
...     return a + b + c
>>> nums = (1, 2, 3)
>>> add(*nums)        # equivalent to add(1, 2, 3)
6
>>> print(*[1, 2, 3])  # equivalent to print(1, 2, 3)
1 2 3
```

And `**` unpacks a dict into keyword arguments:

```python
>>> def greet(name, greeting):
...     return f"{greeting}, {name}!"
>>> kwargs = {"name": "Suresh", "greeting": "Hello"}
>>> greet(**kwargs)
'Hello, Suresh!'
```

### 2.7 Unpacking Works on Any Iterable — Not Just Tuples

Despite the name of this chapter, packing/unpacking syntax isn't tuple-exclusive — it works with any iterable (lists, strings, ranges, generators...):

```python
>>> a, b, c = "xyz"          # string is iterable, one character each
>>> a, b, c
('x', 'y', 'z')
>>> a, b, c = range(3)
>>> a, b, c
(0, 1, 2)
```

---

## 3. Memory Optimization

Tuples aren't just "lists that can't change" — their immutability lets CPython store and manage them more cheaply in several concrete ways.

### 3.1 Smaller Per-Object Overhead

A tuple's C struct doesn't need a separate `allocated` field for spare capacity, because a tuple's size is fixed forever at creation — there's no over-allocation strategy to track (contrast with [Module 10, Section 1.2](#) on lists). Measured directly:

```python
>>> import sys
>>> sys.getsizeof(())                          # empty tuple
40
>>> sys.getsizeof([])                          # empty list
56
>>> sys.getsizeof((1, 2, 3))
64
>>> sys.getsizeof([1, 2, 3])
88
>>> sys.getsizeof((1,2,3,4,5,6,7,8,9,10))
120
>>> sys.getsizeof([1,2,3,4,5,6,7,8,9,10])
136
```

A tuple of the same elements is consistently smaller than the equivalent list — the gap comes entirely from the spare capacity a list keeps for future growth, plus the extra `allocated` bookkeeping field, neither of which a tuple needs.

### 3.2 The Empty Tuple Is a Cached Singleton

CPython keeps exactly **one** empty-tuple object for the entire process and hands out the same object every time you write `()`:

```python
>>> a = ()
>>> b = ()
>>> a is b
True          # always the same object — no allocation happens at all
```

### 3.3 Literal Tuples of Constants Are Folded at Compile Time

Just like the string-literal interning discussed in Module 9, when a tuple literal's contents are themselves all compile-time constants, CPython's compiler can fold the whole tuple into a single shared constant — even across separate calls to the same function:

```python
def make():
    return (1, 2, 3)     # this tuple is a literal-of-constants

t1 = make()
t2 = make()
t1 is t2                  # True — both calls return the SAME pre-built constant object,
                           # created once when the function was compiled, not on every call
```

**This stops being true the moment any element comes from a variable or a runtime computation** — at that point the tuple genuinely has to be built fresh each time:

```python
def make(x, y, z):
    return (x, y, z)      # contents depend on runtime arguments

t1 = make(1, 2, 3)
t2 = make(1, 2, 3)
t1 is t2                   # False — two distinct tuple objects, built at call time

t3 = tuple([1, 2, 3])      # tuple() applied to a list is also a runtime construction
t4 = tuple([1, 2, 3])
t3 is t4                    # False
```

> As with string interning, **never rely on `is` to compare tuple values** — these identity outcomes are CPython implementation/optimizer details, not language guarantees. Always use `==` for value comparison.

### 3.4 No Resizing Overhead, Ever

Because a tuple can never grow or shrink, CPython never has to over-allocate spare slots "just in case" the way it does for lists (see Module 10, Section 2). Every byte a tuple occupies is doing useful work — this is the single biggest reason tuples are cheaper:

```python
>>> import sys
>>> sys.getsizeof((1,) * 100) / sys.getsizeof([1] * 100)
0.857...   # roughly 14% smaller for the same 100 elements, no spare capacity at all
```

### 3.5 Hashability — Tuples Can Be Dict Keys / Set Members

Because tuples are immutable, CPython can safely compute and cache a hash for them — which means a tuple (unlike a list) can be used as a dictionary key or stored in a `set`:

```python
>>> coordinates = {}
>>> coordinates[(3, 4)] = "treasure"
>>> coordinates[(3, 4)]
'treasure'

>>> locations = {(0, 0), (1, 1), (2, 2)}
>>> (1, 1) in locations
True
```

```python
>>> hash((1, 2, 3))      # works — every element is itself hashable
2528502973977326415
>>> hash([1, 2, 3])
TypeError: unhashable type: 'list'
>>> hash((1, [2, 3]))    # fails — a tuple containing a MUTABLE element is NOT hashable
TypeError: unhashable type: 'list'
```

This last example matters: a tuple's *own* immutability doesn't make it hashable if it contains something mutable — hashability requires every nested element to be immutable too, all the way down.

### 3.6 Faster Iteration and Construction

Because the interpreter knows a tuple's size can never change, several operations skip bookkeeping that a list requires — tuple creation from a known-size literal and iteration over a tuple are both marginally faster than the list equivalent, since there's no `allocated`-vs-`used` distinction to consult.

### 3.7 Practical Guidance: When to Choose `tuple` Over `list`

| Use a `tuple` when... | Use a `list` when... |
|---|---|
| The collection's size/contents are fixed once created (a coordinate, an RGB color, a database row) | You'll be appending, removing, or reordering elements |
| You need it as a dict key or set member | You don't need hashability |
| You're returning multiple values from a function | — |
| You want to signal "this is a fixed record" to readers of your code | You want to signal "this is a growable collection" |
| Memory footprint matters at scale (millions of small fixed-size records) | — |

---

## 4. Named Tuples

A plain tuple's biggest readability weakness is that you access fields by **position** — `point[0]`, `point[1]` — which says nothing about what those positions *mean*. **Named tuples** solve this: they're tuples with field names attached, so you get tuple-level memory efficiency *and* attribute-style access.

### 4.1 `collections.namedtuple` — the Classic Approach

```python
from collections import namedtuple

Point = namedtuple("Point", ["x", "y"])   # creates a new TYPE called Point

p = Point(1, 2)
print(p)            # Point(x=1, y=2)  — readable repr, unlike a plain tuple
print(p.x, p.y)      # 1 2              — access by name
print(p[0], p[1])     # 1 2              — STILL works by position too — it's a real tuple
print(isinstance(p, tuple))   # True
```

The field names can also be given as a single space- or comma-separated string:

```python
Point = namedtuple("Point", "x y")          # equivalent to ["x", "y"]
Point = namedtuple("Point", "x, y")          # also equivalent
```

### 4.2 Named-Tuple-Specific Extras

```python
p = Point(1, 2)

p._fields            # ('x', 'y')                — introspect field names
p._asdict()           # {'x': 1, 'y': 2}          — convert to an OrderedDict-like dict
p._replace(x=99)        # Point(x=99, y=2)        — returns a NEW instance (still immutable!)
```

`_replace()` matters because named tuples are **still immutable** — you can't do `p.x = 99`. `_replace()` gives you the common "change one field, keep the rest" pattern without manually rebuilding the whole tuple:

```python
>>> p.x = 99
AttributeError: can't set attribute
>>> p2 = p._replace(x=99)
>>> p, p2
(Point(x=1, y=2), Point(x=99, y=2))    # original untouched; p2 is a separate object
```

### 4.3 Default Values

```python
Point3D = namedtuple("Point3D", ["x", "y", "z"], defaults=[0])
# defaults fill from the RIGHT — only the trailing fields get defaults

Point3D(1, 2)          # Point3D(x=1, y=2, z=0)
Point3D(1, 2, 5)        # Point3D(x=1, y=2, z=5)
```

### 4.4 `typing.NamedTuple` — the Type-Annotated, Class-Based Approach

Since Python 3.6+, you can define a named tuple using regular class syntax with type annotations — preferred in modern code because it reads like a normal class and plays nicely with type checkers (mypy, Pyright):

```python
from typing import NamedTuple

class Point(NamedTuple):
    x: int
    y: int = 0          # default value, just like a dataclass field

p = Point(5)
print(p)            # Point(x=5, y=0)
print(p.x, p.y)      # 5 0
print(isinstance(p, tuple))   # True — still a genuine tuple under the hood
```

You can also add methods to a `typing.NamedTuple` subclass, since it's a real class:

```python
class Point(NamedTuple):
    x: int
    y: int

    def distance_from_origin(self):
        return (self.x ** 2 + self.y ** 2) ** 0.5

p = Point(3, 4)
print(p.distance_from_origin())   # 5.0
```

### 4.5 Memory Comparison: Named Tuple vs. Plain Class vs. Dict

This is where named tuples earn their keep at scale — they cost the **same memory as a plain tuple**, because a named tuple genuinely *is* a tuple subclass; the field names live once on the **class**, not duplicated in every instance:

```python
import sys
from collections import namedtuple

Point = namedtuple("Point", ["x", "y"])

class PlainPoint:
    def __init__(self, x, y):
        self.x = x
        self.y = y

class SlottedPoint:
    __slots__ = ("x", "y")
    def __init__(self, x, y):
        self.x = x
        self.y = y

d  = {"x": 1, "y": 2}
t  = (1, 2)
nt = Point(1, 2)
pp = PlainPoint(1, 2)
sp = SlottedPoint(1, 2)

print("dict:               ", sys.getsizeof(d))
print("plain tuple:        ", sys.getsizeof(t))
print("namedtuple:         ", sys.getsizeof(nt))
print("regular class inst.:", sys.getsizeof(pp), "+ its __dict__:", sys.getsizeof(pp.__dict__))
print("__slots__ class:    ", sys.getsizeof(sp))
```

Actual measured output:

```
dict:                184
plain tuple:          56
namedtuple:           56
regular class inst.:  48 + its __dict__: 296
__slots__ class:      48
```

**Key takeaway:** a `namedtuple` instance is exactly as small as a plain tuple (56 bytes here), while a regular class instance carries a whole separate `__dict__` (296 bytes here, on top of the 48-byte object header) just to store two attributes. A `dict` with the same two key-value pairs is over 3× the size of the namedtuple. If you're creating millions of small fixed-shape records (rows from a database query, parsed log entries, coordinate pairs), this difference compounds fast.

### 4.6 Named Tuple vs. `dataclass` — When to Use Which

| | `namedtuple` / `typing.NamedTuple` | `dataclass` |
|---|---|---|
| Mutable? | No — immutable, like any tuple | Yes by default (can opt into `frozen=True`) |
| Memory | Same as a plain tuple (very small) | Uses `__dict__` like a regular class, unless `__slots__` is added |
| Behaves like a tuple? | Yes — indexable, iterable, unpackable | No — not a tuple, no positional unpacking by default |
| Best for | Fixed, small, immutable records — especially in large quantities | General-purpose objects, mutable state, more complex behavior |

```python
# A named tuple unpacks just like any tuple — a dataclass does NOT, by default:
Point = namedtuple("Point", ["x", "y"])
x, y = Point(1, 2)        # works fine: (1, 2)

from dataclasses import dataclass
@dataclass
class DPoint:
    x: int
    y: int

x, y = DPoint(1, 2)        # TypeError: cannot unpack non-iterable DPoint object
```

### 4.7 Quick Decision Guide

- Need a lightweight, **immutable**, tuple-compatible record with named fields (e.g. a row from `csv.reader`, a coordinate, an RGB triple)? → **named tuple**.
- Need something **mutable**, or with rich behavior/validation/inheritance? → **`dataclass`** or a regular class.
- Just need a quick, throwaway bundle of values with no names attached at all? → **plain tuple**.

---

## Summary Cheat-Sheet

```
Packing       → comma creates a tuple: x = 1, 2, 3   (parens optional)
                also how *args and `return a, b` work
Unpacking     → a, b = (1, 2)
                *middle catches "the rest" as a list: a, *m, b = [1,2,3,4]
                works on any iterable, not just tuples; nested patterns allowed
Swap idiom    → a, b = b, a   (right side fully packed before any assignment happens)
Memory        → tuples skip the spare-capacity bookkeeping lists need → smaller,
                fixed-size, hashable (if all elements are themselves hashable)
() singleton  → every empty tuple () is literally the SAME cached object
Named tuples  → namedtuple() / typing.NamedTuple — attribute access + tuple memory
                footprint; immutable; ._replace() to "modify"; same size as a plain tuple,
                much smaller than an equivalent dict or regular class instance
```
