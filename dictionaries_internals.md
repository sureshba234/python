# Python Dictionaries — Internals

## Table of Contents
1. [Hashing](#1-hashing)
2. [Collision Handling](#2-collision-handling)
3. [Split Table](#3-split-table)
4. [Compact Dictionary](#4-compact-dictionary)
5. [Ordered Dictionaries](#5-ordered-dictionaries)

---

## 1. Hashing

### What it is
Hashing converts a key (any hashable object) into a fixed-size integer called a **hash value**, using a **hash function**. Python calls `hash(key)` to get this number.

```python
print(hash(42))        # 42
print(hash("hello"))   # some large integer
```

### Why dictionaries need it
A dict needs a way to go straight to where a value is stored, instead of scanning every entry. Hashing turns "search for this key" into "do some arithmetic and jump there directly":

```
index = hash(key) & (table_size - 1)   # CPython uses bitmasking, not %
table[index] = (key, value)
```

This is why dict lookup is **O(1) on average** — one hash computation plus one array access, instead of checking every item like a list would (O(n)).

### The one rule that must always hold
**If two objects are equal, they must have the same hash.**

```python
print(1 == 1.0)             # True
print(hash(1) == hash(1.0)) # True — required
```

The reverse isn't required — two *different* objects can share a hash. That's called a **collision** (see next section).

### What can be hashed
Only **immutable-ish** objects: `int`, `float`, `str`, `tuple` (if its contents are hashable), `frozenset`. Mutable types — `list`, `dict`, `set` — are **not hashable**, because if a key could change after being inserted, its hash would change too, and the dict would no longer know where to find it.

```python
d = {[1, 2]: "x"}
# TypeError: unhashable type: 'list'
```

If you write a custom class and override `__eq__`, you must also override `__hash__` to keep them consistent — otherwise equal objects could end up with different hashes, silently breaking dict/set behavior.

### A practical note
Since Python 3.3, string hashing is randomized per process (a security measure called hash randomization, to prevent attackers from crafting keys that all collide). This means `hash("abc")` gives a different number each time you run the script — never rely on it staying the same across runs.

---

## 2. Collision Handling

### What a collision is
A collision happens when two different keys hash to the **same slot** in the table. Since the table has limited slots but there are infinitely many possible keys, collisions are guaranteed to happen eventually — no hash function can avoid them entirely.

```python
# hypothetical table_size = 8
hash("apple") % 8 == 3
hash("grape") % 8 == 3   # collision — both want slot 3
```

### The two general strategies
| Strategy | Idea |
|---|---|
| **Separate chaining** | Each slot holds a list of all entries that hashed there |
| **Open addressing** | If a slot is taken, probe for another empty slot, following a defined sequence |

Python's `dict` uses **open addressing** — everything lives in one array, no chains/linked lists.

### How Python resolves a collision
When the first slot is occupied, Python computes another candidate slot using a **probing sequence**, and keeps probing until it finds an empty slot (for insertion) or the matching key (for lookup):

```
Insert "apple" → slot 3 (empty) → place it there
Insert "grape" → slot 3 (taken!) → probe → slot 6 (empty) → place it there
```

CPython's actual probe sequence isn't simple linear probing (`+1, +2, +3...`), because that tends to cluster entries together. Instead it uses the full hash value to "perturb" the next index, scattering probes more evenly:

```c
perturb = hash
i = hash & mask
while slot i is occupied by a different key:
    perturb >>= 5
    i = (5*i + 1 + perturb) & mask
```

Lookup uses this same sequence: compute the first index, check if the key matches, and if not, keep following the same probe steps until it either finds the key or hits a truly empty slot (meaning "not present").

### Deletion is trickier than it looks
You can't just clear a slot when deleting, because later entries might have been placed somewhere else *because* this slot was full at the time. Clearing it would break the probe chain for those entries, making them unreachable. So deleted slots are marked with a special **tombstone** marker instead — lookups treat a tombstone as "keep probing past me," while new insertions treat it as "this slot is free to reuse."

### Load factor
This is how full the table is: `entries / table_size`. Python keeps this below roughly 2/3 by resizing (growing) the table once it gets too full — fewer empty slots means more collisions and longer probe sequences, so resizing keeps lookups close to O(1).

---

## 3. Split Table

### The problem it solves
Before this optimization (Python 3.3, PEP 412), every dictionary stored its keys and values together. For objects — where many instances of the same class have identical attribute names — this meant the **same key strings** (`"x"`, `"y"`, etc.) were duplicated in memory for every single instance.

```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

p1 = Point(1, 2)
p2 = Point(3, 4)
# Without split tables: keys "x" and "y" stored separately in p1.__dict__ AND p2.__dict__
```

### The fix
A split-table dict separates storage into two parts:
- A **shared keys table** — holding the key names, used by *all* instances of the class.
- A **private values array** per instance — holding only that instance's values, in the same order as the shared keys.

```
Shared keys table (ONE copy):     index 0: "x"   index 1: "y"

p1's values: [1, 2]
p2's values: [3, 4]
```

Since most instances of a class set the same attributes in the same order, they can all point to one shared keys table instead of each carrying their own copy.

### When it applies
This is specifically used for an object's `__dict__` (its instance attributes) — **not** for dictionaries you create yourself with `{}`. If you delete an attribute, or different instances set attributes in a different order, the dict "splits off" into a regular combined table for that instance and loses the sharing benefit.

### Why it matters
It significantly cuts memory use for programs that create many instances of the same class — a very common situation — without changing how the dict behaves from the outside.

---

## 4. Compact Dictionary

### The old layout (before Python 3.6)
A dict's hash table directly stored `(hash, key, value)` in a single sparse array. To keep collisions low, this array had to stay mostly empty (often a third to two-thirds unused), and each empty slot still reserved space as if it held a full entry. Iterating the dict visited slots in hash order — not the order you inserted things — so dicts had no defined iteration order.

```
index 0: EMPTY
index 1: EMPTY
index 2: (hash, "b", 2)
index 3: EMPTY
index 4: (hash, "a", 1)
...
```

### The new layout (Python 3.6+)
The compact dict splits storage into two separate arrays:

1. A **sparse indices array** — only stores small integers (indices), sized to keep collisions low.
2. A **dense entries array** — stores the actual `(hash, key, value)` triples, packed tightly with no gaps, in the exact order they were inserted.

```
Sparse indices:           Dense entries (insertion order, no gaps):
index 4 → 0                entries[0] = (hash, "a", 1)
index 2 → 1                entries[1] = (hash, "b", 2)
index 7 → 2                entries[2] = (hash, "c", 3)
```

Lookup now takes one extra step: hash the key, probe the sparse array to get an index, then jump to that position in the dense array.

### Why this saves memory
The sparse array — the one that needs lots of empty slots to avoid collisions — now only holds small integers instead of full key-value entries, so empty slots cost almost nothing. The dense array holds the real data but has zero wasted space, since it's filled with no gaps. Together this cut typical dict memory use by roughly 20-25%.

### The bonus: insertion order, for free
Because the dense array is filled strictly in the order keys are added, simply iterating it in order naturally gives you insertion order — no extra linked list or bookkeeping needed. This is the exact reason regular Python dicts started preserving insertion order starting with this change (see next section).

---

## 5. Ordered Dictionaries

### The guarantee
Since Python 3.7 (an implementation detail in 3.6, made an official language guarantee in 3.7), a regular `dict` preserves **insertion order** when you iterate over it:

```python
d = {}
d["z"] = 1
d["a"] = 2
d["m"] = 3
print(list(d.keys()))   # ['z', 'a', 'm'] — insertion order, not sorted
```

### What counts as "insertion order"
- The order keys were **first** added.
- Updating an existing key's value does **not** move it.
- Deleting a key and re-adding it moves it to the **end**, since it's a fresh insertion.

```python
d = {"a": 1, "b": 2, "c": 3}
d["a"] = 100        # update — position unchanged
print(d)              # {'a': 100, 'b': 2, 'c': 3}

del d["a"]
d["a"] = 999          # re-insert — goes to the end now
print(d)               # {'b': 2, 'c': 3, 'a': 999}
```

This ordering isn't a separate feature bolted on — it falls directly out of the compact dict's dense array always being filled in insertion order (Section 4).

### `collections.OrderedDict`
This class predates the 3.7 guarantee and still has a few things a regular `dict` doesn't:

| Feature | `dict` | `OrderedDict` |
|---|---|---|
| Preserves insertion order | Yes | Yes |
| `move_to_end(key, last=True/False)` | No | Yes |
| Equality is order-sensitive | No | Yes |

```python
from collections import OrderedDict

od = OrderedDict([("a", 1), ("b", 2), ("c", 3)])
od.move_to_end("a")
print(od)   # OrderedDict([('b', 2), ('c', 3), ('a', 1)])
```

`move_to_end()` is the building block for an LRU cache — moving the most recently used key to the end (or evicting from the front with `popitem(last=False)`).

Regular dict equality ignores order:
```python
print({"a": 1, "b": 2} == {"b": 2, "a": 1})   # True
```
But `OrderedDict` equality cares about order:
```python
od1 = OrderedDict([("a", 1), ("b", 2)])
od2 = OrderedDict([("b", 2), ("a", 1)])
print(od1 == od2)   # False
```

Today, plain `dict` is the default choice for ordered key-value storage; `OrderedDict` is reached for specifically when you need `move_to_end()` or order-sensitive equality.
