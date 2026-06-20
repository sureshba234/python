# Python Mappings — A Complete Guide

## Table of Contents

1. [What Is a Mapping](#1-what-is-a-mapping)
2. [The Mapping Abstraction](#2-the-mapping-abstraction)
3. [`dict` — The Core Mapping Type](#3-dict--the-core-mapping-type)
4. [Abstract Base Classes: `Mapping` and `MutableMapping`](#4-abstract-base-classes-mapping-and-mutablemapping)
5. [Built-in and Standard Library Mapping Types](#5-built-in-and-standard-library-mapping-types)
6. [Read-Only Mappings: `MappingProxyType`](#6-read-only-mappings-mappingproxytype)
7. [Building a Custom Mapping](#7-building-a-custom-mapping)
8. [Mapping Protocol Requirements](#8-mapping-protocol-requirements)
9. [Mappings vs Sequences vs Sets](#9-mappings-vs-sequences-vs-sets)
10. [Common Operations Across Mapping Types](#10-common-operations-across-mapping-types)
11. [Type Hints for Mappings](#11-type-hints-for-mappings)
12. [Best Practices](#12-best-practices)
13. [Common Mistakes](#13-common-mistakes)
14. [Interview Questions](#14-interview-questions)
15. [Quick Reference](#15-quick-reference)

---

## 1. What Is a Mapping

A **mapping** is a general category of data structure that associates **keys** with **values**, allowing lookup of a value by its key rather than by position. `dict` is the most common mapping type in Python, but it is just one implementation of the broader **mapping protocol**.

```python
# dict is a mapping
m = {"a": 1, "b": 2}
print(m["a"])
```

**Output:**
```
1
```

Conceptually, "mapping" is to "dict" what "sequence" is to "list" — a dict is a concrete mapping, just as a list is a concrete sequence. Other objects (like `os.environ`, `collections.Counter`, or custom classes) can also behave as mappings without literally being `dict` instances.

---

## 2. The Mapping Abstraction

Python defines mapping behavior as a **protocol** — a set of methods an object must support to be treated as a mapping, regardless of its internal implementation. This is part of Python's general philosophy of **duck typing**: "if it behaves like a mapping, it is one."

```
          Mapping Protocol
   ┌─────────────────────────────┐
   │ __getitem__(key)             │
   │ __len__()                    │
   │ __iter__()                   │
   │ __contains__(key)  (optional)│
   └─────────────────────────────┘
              │
   ┌──────────┴───────────┐
   │                       │
 dict                 custom classes
 (mutable)            (e.g. database row,
                        config wrapper)
```

Any object implementing `__getitem__`, `__iter__`, and `__len__` can be recognized as a mapping by tools like `collections.abc.Mapping`.

---

## 3. `dict` — The Core Mapping Type

`dict` is Python's built-in, mutable mapping implementation. It is the mapping type used by default for object attributes (`__dict__`), keyword arguments, and JSON parsing.

```python
person = {"name": "Alice", "age": 30}
person["city"] = "NYC"
print(person)
```

**Output:**
```
{'name': 'Alice', 'age': 30, 'city': 'NYC'}
```

Key properties:

| Property | Description |
|---|---|
| Mutable | Keys/values can be added, changed, removed |
| Ordered | Preserves insertion order (3.7+) |
| Keys | Must be hashable and unique |
| Values | Any type |
| Lookup | O(1) average time |

> For a deep, dedicated walkthrough of `dict` specifically — every method, comprehension, sorting, JSON handling, exercises, and projects — see the companion **Python Dictionaries** guide. This document instead focuses on the broader **mapping** concept: the protocol, abstract base classes, and the family of mapping types built on top of (or alongside) `dict`.

---

## 4. Abstract Base Classes: `Mapping` and `MutableMapping`

Python's `collections.abc` module defines formal **abstract base classes (ABCs)** describing what it means to be a mapping.

```python
from collections.abc import Mapping, MutableMapping

print(isinstance({}, Mapping))          # True
print(isinstance({}, MutableMapping))   # True
```

**Output:**
```
True
True
```

### `Mapping` (Read-Only Interface)

Requires implementing only:
- `__getitem__`
- `__iter__`
- `__len__`

From these three, the ABC automatically provides `__contains__`, `keys()`, `values()`, `items()`, `get()`, `__eq__`, and `__ne__` for free.

### `MutableMapping` (Read-Write Interface)

Extends `Mapping` and additionally requires:
- `__setitem__`
- `__delitem__`

From these, it automatically provides `pop()`, `popitem()`, `clear()`, `update()`, and `setdefault()`.

```
            Mapping
   (read-only: get, keys, values,
    items, __contains__, __eq__)
                │
                ▼
         MutableMapping
   (adds: set, delete, pop, popitem,
    clear, update, setdefault)
                │
                ▼
              dict
   (concrete, optimized implementation)
```

This is why `dict` "for free" supports so many methods — it satisfies `MutableMapping`, and conceptually inherits its rich method set (in practice `dict` implements them directly for performance, but the *contract* comes from the ABC).

---

## 5. Built-in and Standard Library Mapping Types

| Type | Mutable | Ordered | Purpose |
|---|---|---|---|
| `dict` | Yes | Yes | General-purpose mapping |
| `collections.OrderedDict` | Yes | Yes (explicit) | Order-sensitive equality, `move_to_end()` |
| `collections.defaultdict` | Yes | Yes | Auto-default missing keys |
| `collections.Counter` | Yes | Yes | Counting hashable items |
| `collections.ChainMap` | Yes (writes go to first dict) | Yes | Layered/combined view of multiple dicts |
| `types.MappingProxyType` | No | Matches wrapped dict | Read-only view over a dict |
| `os.environ` | Yes (special) | OS-dependent | Mapping interface over environment variables |

### `defaultdict`

```python
from collections import defaultdict
d = defaultdict(int)
d["x"] += 1
print(d)
```

**Output:**
```
defaultdict(<class 'int'>, {'x': 1})
```

### `Counter`

```python
from collections import Counter
c = Counter("banana")
print(c)
```

**Output:**
```
Counter({'a': 3, 'n': 2, 'b': 1})
```

### `OrderedDict`

```python
from collections import OrderedDict
d = OrderedDict(a=1, b=2)
d.move_to_end("a")
print(d)
```

**Output:**
```
OrderedDict([('b', 2), ('a', 1)])
```

### `ChainMap`

```python
from collections import ChainMap
defaults = {"theme": "light"}
overrides = {"theme": "dark"}
view = ChainMap(overrides, defaults)
print(view["theme"])
```

**Output:**
```
dark
```

> `ChainMap` does not copy or merge the underlying dictionaries — it searches them in order at lookup time, and writes always go to the **first** mapping in the chain.

---

## 6. Read-Only Mappings: `MappingProxyType`

`types.MappingProxyType` wraps an existing dictionary in a **read-only view**. It's useful for exposing internal state without allowing external mutation.

```python
from types import MappingProxyType

settings = {"debug": False}
read_only = MappingProxyType(settings)

print(read_only["debug"])

try:
    read_only["debug"] = True
except TypeError as e:
    print("Error:", e)
```

**Output:**
```
False
Error: 'mappingproxy' object does not support item assignment
```

> Note that the proxy is a **live view**: if the original dictionary changes, the proxy reflects it immediately.

```python
settings["debug"] = True
print(read_only["debug"])
```

**Output:**
```
True
```

---

## 7. Building a Custom Mapping

You can create your own mapping type by subclassing `collections.abc.Mapping` (read-only) or `MutableMapping` (read-write) and implementing only the required methods.

```python
from collections.abc import Mapping

class CaseInsensitiveDict(Mapping):
    def __init__(self, data=None):
        self._data = {}
        if data:
            for k, v in data.items():
                self._data[k.lower()] = v

    def __getitem__(self, key):
        return self._data[key.lower()]

    def __iter__(self):
        return iter(self._data)

    def __len__(self):
        return len(self._data)


headers = CaseInsensitiveDict({"Content-Type": "application/json"})
print(headers["content-type"])
print(headers["CONTENT-TYPE"])
print(list(headers.items()))
```

**Output:**
```
application/json
application/json
[('content-type', 'application/json')]
```

Because `CaseInsensitiveDict` inherits from `Mapping`, it automatically gets `.get()`, `.keys()`, `.values()`, `.items()`, `in`, and equality comparisons — all derived from the three methods we implemented.

```python
print("content-type" in headers)
print(headers.get("missing", "N/A"))
```

**Output:**
```
True
N/A
```

### Adding Mutability

```python
from collections.abc import MutableMapping

class TrackedDict(MutableMapping):
    def __init__(self):
        self._data = {}
        self.write_count = 0

    def __getitem__(self, key):
        return self._data[key]

    def __setitem__(self, key, value):
        self._data[key] = value
        self.write_count += 1

    def __delitem__(self, key):
        del self._data[key]

    def __iter__(self):
        return iter(self._data)

    def __len__(self):
        return len(self._data)


t = TrackedDict()
t["a"] = 1
t["b"] = 2
print(dict(t), "writes:", t.write_count)
```

**Output:**
```
{'a': 1, 'b': 2} writes: 2
```

`TrackedDict` automatically inherits `pop()`, `update()`, `setdefault()`, `clear()`, and `popitem()` from `MutableMapping`, without writing them manually.

---

## 8. Mapping Protocol Requirements

| ABC | Must Implement | Gets For Free |
|---|---|---|
| `Mapping` | `__getitem__`, `__iter__`, `__len__` | `get`, `keys`, `values`, `items`, `__contains__`, `__eq__`, `__ne__` |
| `MutableMapping` | All of `Mapping` + `__setitem__`, `__delitem__` | `pop`, `popitem`, `clear`, `update`, `setdefault` |

This minimal-implementation, maximal-functionality pattern is the same template used throughout `collections.abc` (compare `Sequence`/`MutableSequence` for lists, `Set`/`MutableSet` for sets).

---

## 9. Mappings vs Sequences vs Sets

| Feature | Mapping (`dict`) | Sequence (`list`) | Set (`set`) |
|---|---|---|---|
| Access by | Key | Integer index | N/A (membership only) |
| Order | Insertion order | Position order | Unordered (logically) |
| Duplicates | Keys unique | Values can repeat | Elements unique |
| Core methods | `__getitem__`, `keys`, `items` | `__getitem__`, `__len__` | `__contains__`, set ops |
| Typical use | Labeled lookups | Ordered collections | Uniqueness/membership |

---

## 10. Common Operations Across Mapping Types

Because all mapping types share the same protocol, the same patterns work across `dict`, `defaultdict`, `OrderedDict`, custom mappings, and (with limits) `MappingProxyType`:

```python
def describe(mapping):
    print("Length:", len(mapping))
    print("Has 'a':", "a" in mapping)
    print("Keys:", list(mapping.keys()))
    print("Items:", list(mapping.items()))

describe({"a": 1, "b": 2})
```

**Output:**
```
Length: 2
Has 'a': True
Keys: ['a', 'b']
Items: [('a', 1), ('b', 2)]
```

This is the practical power of the mapping abstraction: **code written against the `Mapping` interface works for any conforming type**, whether it's a plain `dict`, a database-row wrapper, or a config object.

---

## 11. Type Hints for Mappings

```python
from typing import Mapping, MutableMapping, Dict

def read_config(config: Mapping[str, str]) -> str:
    return config.get("env", "production")

def update_config(config: MutableMapping[str, str]) -> None:
    config["env"] = "staging"

def make_config() -> Dict[str, str]:
    return {"env": "development"}
```

- Use `Mapping[K, V]` in function signatures when you only need **read** access — it accepts `dict`, `MappingProxyType`, custom mappings, etc.
- Use `MutableMapping[K, V]` when the function needs to **write**.
- Use `Dict[K, V]` (or `dict[K, V]` in 3.9+) only when you specifically require a real `dict` (e.g., for performance or a specific method like `.copy()`'s exact return type).

> Accepting `Mapping` instead of `dict` in function signatures is a best practice — it widens what callers can pass in without losing safety.

---

## 12. Best Practices

- Prefer **`Mapping`** (not `dict`) in type hints for function parameters that only read data — it's more permissive and decouples your code from a specific implementation.
- Use **`MappingProxyType`** to expose internal dictionaries to callers without risking external mutation.
- Reach for **`defaultdict`** or **`Counter`** instead of manually checking and initializing keys.
- Use **`ChainMap`** for layered configuration (e.g., CLI args > environment variables > defaults) instead of repeatedly merging dictionaries.
- When designing a custom mapping, subclass `collections.abc.Mapping` / `MutableMapping` rather than reimplementing the whole interface from scratch.
- Keep custom mapping keys hashable and consistent with `__eq__`/`__hash__` if they are custom objects.

---

## 13. Common Mistakes

### Mistake 1: Assuming All Mappings Are Ordered the Same Way

`ChainMap` order reflects the order of the mappings passed to it, not necessarily insertion order of the combined data — don't assume it behaves identically to a merged `dict`.

```python
from collections import ChainMap
cm = ChainMap({"a": 1}, {"b": 2})
print(list(cm.items()))
```

**Output:**
```
[('b', 2), ('a', 1)]
```

### Mistake 2: Mutating Through a Read-Only Proxy's Source Object

```python
from types import MappingProxyType
data = {"x": 1}
proxy = MappingProxyType(data)
data["x"] = 2          # allowed — proxy is read-only, not the source dict
print(proxy["x"])      # reflects the change anyway
```

**Output:**
```
2
```

`MappingProxyType` prevents writes *through the proxy*, not changes to the underlying dictionary.

### Mistake 3: Forgetting `MutableMapping` Subclasses Need `__hash__` Considerations

Mutable mappings are inherently unhashable (like `dict`) — you cannot use them as dictionary keys or set elements.

```python
d = {"a": 1}
try:
    {d: "value"}
except TypeError as e:
    print("Error:", e)
```

**Output:**
```
Error: unhashable type: 'dict'
```

### Mistake 4: Treating `Mapping` Type Hints as Runtime Guarantees

Type hints like `Mapping[str, int]` are not enforced at runtime; they only guide static type checkers (e.g., `mypy`). Validate input explicitly if runtime safety matters.

---

## 14. Interview Questions

1. **What is the difference between a mapping and a dictionary?**
   A mapping is the general abstract concept/protocol (key → value lookup); `dict` is Python's concrete, built-in mapping implementation.
2. **What three methods must an object implement to satisfy `collections.abc.Mapping`?**
   `__getitem__`, `__iter__`, and `__len__`.
3. **What extra methods does `MutableMapping` require beyond `Mapping`?**
   `__setitem__` and `__delitem__`.
4. **Why would you type-hint a function parameter as `Mapping[str, int]` instead of `Dict[str, int]`?**
   To allow any read-only mapping-compatible object (not just `dict`) to be passed in, increasing flexibility.
5. **What does `MappingProxyType` do, and is it a copy or a view?**
   It wraps a dictionary in a read-only interface; it's a **live view**, not a copy, so changes to the original dict are reflected.
6. **How does `ChainMap` differ from merging dictionaries with `update()` or `|`?**
   `ChainMap` doesn't copy or merge data — it searches the underlying mappings in order at lookup time, and writes go only to the first mapping.
7. **Why are dictionaries themselves not hashable, and therefore not usable as dictionary keys?**
   Because they're mutable, and hashable objects must have a stable hash value for their lifetime.
8. **Name three standard-library mapping types other than `dict`.**
   `defaultdict`, `OrderedDict`, `Counter`, and `ChainMap` are common answers (any three).
9. **What automatically-provided methods do you get for free by subclassing `Mapping`?**
   `get()`, `keys()`, `values()`, `items()`, `__contains__()`, `__eq__()`, `__ne__()`.
10. **Give an example of a real-world object (besides `dict`) that behaves as a mapping.**
    `os.environ` (environment variables) behaves as a mapping interface over OS-level data.

---

## 15. Quick Reference

```python
# Mapping ABCs
from collections.abc import Mapping, MutableMapping
isinstance({}, Mapping)          # True
isinstance({}, MutableMapping)   # True

# Standard library mapping types
from collections import defaultdict, OrderedDict, Counter, ChainMap
from types import MappingProxyType

defaultdict(int)                 # auto-default missing keys
OrderedDict(a=1, b=2)            # explicit order tracking, move_to_end()
Counter("aab")                   # counting hashable items
ChainMap(d1, d2)                 # layered, non-copying view
MappingProxyType(d)              # read-only live view

# Custom mapping (read-only)
class MyMap(Mapping):
    def __getitem__(self, key): ...
    def __iter__(self): ...
    def __len__(self): ...

# Custom mapping (read-write)
class MyMutableMap(MutableMapping):
    def __getitem__(self, key): ...
    def __setitem__(self, key, value): ...
    def __delitem__(self, key): ...
    def __iter__(self): ...
    def __len__(self): ...

# Type hints
from typing import Mapping, MutableMapping, Dict
def read(m: Mapping[str, int]) -> int: ...
def write(m: MutableMapping[str, int]) -> None: ...
```

---

## Summary

A **mapping** is the abstract concept of key-based lookup; `dict` is its primary, optimized, built-in implementation in Python. The `collections.abc` module formalizes this with `Mapping` (read-only protocol) and `MutableMapping` (read-write protocol), letting you build custom mapping-like objects with minimal code while inheriting a rich, consistent API. Beyond `dict`, the standard library offers specialized mappings — `defaultdict`, `OrderedDict`, `Counter`, `ChainMap`, and the read-only `MappingProxyType` — each suited to different real-world needs: auto-defaults, explicit ordering, counting, layered configuration, and safe read-only exposure, respectively. Understanding mappings as a *protocol* rather than just "the dict type" is what lets you write flexible, interface-driven Python code.
