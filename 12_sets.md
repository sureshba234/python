# 12. Sets — Complete Reference

A Python `set` is an **unordered collection of unique, hashable elements**, backed internally by a **hash table** — the same fundamental data structure that powers `dict`. Everything distinctive about sets — O(1) average-case membership testing, no duplicates, no guaranteed order — flows directly from that one implementation choice.

```python
>>> s = {1, 2, 3, 2, 1}
>>> s
{1, 2, 3}          # duplicates silently collapsed
>>> 2 in s          # O(1) average — doesn't scan the whole set
True
```

---

## Table of Contents

1. [Hash Tables](#1-hash-tables)
2. [Collision Resolution](#2-collision-resolution)
3. [Load Factor](#3-load-factor)
4. [Hash Randomization](#4-hash-randomization)

---

## 1. Hash Tables

### 1.1 The Core Idea

A hash table stores elements in **slots** of a fixed-size array. To find which slot an element belongs in, you run it through a **hash function** that converts it to an integer, then reduce that integer to a valid array index (typically `hash(x) & (size - 1)`, which works because the table size is always a power of two).

```
element ──► hash() ──► big integer ──► & mask ──► slot index
   42      hash(42)=42        42        & 7    =     2
```

If you know which slot an item *should* be in, you can jump straight there — no scanning the whole collection. That's the entire reason hash tables give average **O(1)** membership testing, insertion, and deletion, versus O(n) for scanning a list.

### 1.2 CPython's `set` Struct

```c
/* Objects/setobject.c, simplified */
typedef struct {
    setentry *table;          // pointer to the array of slots (key + cached hash)
    Py_ssize_t fill;          // active entries + dummy (deleted) entries
    Py_ssize_t used;          // active entries only — what len(s) returns
    Py_ssize_t mask;          // table size - 1 (table size is always a power of 2)
    setentry smalltable[8];   // small embedded table, used while the set stays tiny
} PySetObject;

typedef struct {
    PyObject *key;             // pointer to the actual element
    Py_hash_t hash;            // its hash value, CACHED here so it's never recomputed
} setentry;
```

A few things worth noticing immediately:

- The table always starts at **8 slots** (`PySet_MINSIZE`), embedded directly inside the set object itself (`smalltable`) — for small sets, there's no separate heap allocation for the table at all.
- The hash of every element is **cached** in its slot, alongside the pointer to the element — so re-checking "does this slot match?" never has to recompute a hash.
- **`fill` and `used` are different numbers.** `fill` counts active entries *plus* entries marked as deleted-but-not-yet-purged ("dummy" slots — see [Section 2.4](#24-deletions-and-dummy-entries)); `used` (what `len()` reports) counts only genuinely active entries.

### 1.3 Why Elements Must Be Hashable

Every operation on a set depends on being able to compute `hash(element)` and have that hash stay **constant for the object's lifetime** — if an element's hash could change after insertion, the table would be looking in the wrong slot for it forever. This is exactly why **mutable objects can't go into a set**:

```python
>>> {1, 2, 3}          # fine — ints are immutable
{1, 2, 3}
>>> {[1, 2], [3, 4]}    # lists are mutable → unhashable
TypeError: unhashable type: 'list'
>>> {(1, 2), (3, 4)}    # tuples ARE hashable (if their contents are too)
{(1, 2), (3, 4)}
```

### 1.4 Measuring the Table

```python
>>> import sys
>>> sys.getsizeof(set())        # empty set: just the 8-slot embedded table + header
216
```

Watching it grow as elements are added (each `add()` either fits in existing slots or triggers a resize — see [Section 3](#3-load-factor)):

```python
s = set()
prev = None
for i in range(20):
    s.add(i)
    sz = sys.getsizeof(s)
    if sz != prev:
        print(f"len={len(s):2d}  getsizeof={sz}")
        prev = sz
```

Actual measured output (CPython 3.12):

```
len= 1  getsizeof=216
len= 5  getsizeof=728
len=19  getsizeof=2264
```

The jumps happen at specific length thresholds tied directly to the load factor rule covered in Section 3 — the table doesn't grow by one slot at a time; it jumps to a whole new, larger power-of-two-sized table and re-inserts everything.

### 1.5 Why Sets Have No Defined Order

Because an element's position is determined by `hash(element) & mask` rather than insertion order, two elements inserted in any sequence can land anywhere in the table depending purely on their hash values — there's no positional/sequential relationship to preserve. This is precisely why `set` iteration order is unspecified and can even differ between two sets containing the same elements if they were built differently:

```python
>>> {3, 1, 2}
{1, 2, 3}           # looks "sorted" here purely by coincidence of small-int hashing
>>> {"banana", "apple", "cherry"}
{'banana', 'cherry', 'apple'}   # not sorted, not insertion order — hash-bucket order
```

---

## 2. Collision Resolution

### 2.1 What a Collision Is

Two different elements can hash to the **same slot index** — either because they share the exact same hash value, or (far more commonly) because two different hash values reduce to the same index after the `& mask` operation. A hash table *must* have a strategy for handling this, or the second element would simply overwrite the first.

CPython's sets use **open addressing**: when a slot is already occupied by a different element, the algorithm doesn't give up — it computes a *next* candidate slot to check, and keeps going until it finds either the element it's looking for or an empty slot.

### 2.2 CPython's Actual Probing Sequence

This is a **hybrid of linear probing and pseudo-random probing**, taken directly from `Objects/setobject.c` (the comments there even cite Knuth's *The Art of Computer Programming*, Vol. 3, Algorithm D):

```c
#define LINEAR_PROBES 9
#define PERTURB_SHIFT 5

size_t perturb = hash;
size_t mask    = so->mask;
size_t i       = (size_t)hash & mask;     /* initial slot guess */

while (1) {
    /* check slot i, then up to LINEAR_PROBES (9) consecutive slots after it */
    ...
    /* none of those matched and none were empty → jump elsewhere */
    perturb >>= PERTURB_SHIFT;             /* fold in more bits of the original hash */
    i = (i * 5 + 1 + perturb) & mask;      /* linear-congruential-style jump */
}
```

**Why this specific hybrid, instead of one or the other?**

- **Pure linear probing** (just check `i+1`, `i+2`, `i+3`, ...) is *cache-friendly* — those slots are right next to each other in memory, which is cheap to read — but it tends to form long "clumps" of collisions when many keys hash near each other.
- **Pure random/scattered probing** spreads collisions out well, avoiding clumping, but each jump is a scattered memory access, which is slower per-step on real hardware (cache misses).
- CPython's compromise: check up to **9 nearby slots linearly first** (cheap, cache-friendly — catches the common case fast), and only fall back to the **pseudo-random jump** (`i = (i*5 + 1 + perturb) & mask`) if all of those are occupied by non-matches. As probing continues, `perturb` is right-shifted by 5 bits each round, gradually mixing in higher bits of the original hash so the jump sequence doesn't repeat in a short cycle.

### 2.3 Watching Collisions Happen at the Python Level

You can force collisions deliberately by overriding `__hash__` so multiple distinct objects land in the same initial slot — the set still works correctly, it just has to probe further to tell them apart using `__eq__`:

```python
class BadHash:
    def __init__(self, value):
        self.value = value
    def __hash__(self):
        return 1                      # EVERY instance collides on the same initial slot
    def __eq__(self, other):
        return isinstance(other, BadHash) and self.value == other.value
    def __repr__(self):
        return f"BadHash({self.value})"

s = set()
for i in range(5):
    s.add(BadHash(i))

print(s)        # {BadHash(0), BadHash(3), BadHash(4), BadHash(1), BadHash(2)}
print(len(s))   # 5 — all five are stored correctly despite the hash collision
```

All five objects share `hash() == 1`, so they all start probing from the same slot — but because `__eq__` correctly distinguishes them, the table probes onward each time it finds an occupied-but-unequal slot, and all five end up safely stored in different slots. **Correctness is preserved**; only *speed* suffers (every lookup now has to probe several slots and run `__eq__` checks instead of landing directly in one).

### 2.4 Deletions and Dummy Entries

If you simply emptied a slot on `discard()`/`remove()`, you'd break probing for *other* elements that originally collided with the deleted one and got pushed further along the probe sequence — clearing the slot would make it look like the probe sequence ends there, causing later lookups to incorrectly report "not found." CPython's fix: deleted slots are marked with a special **dummy** placeholder object instead of being nulled out, so probing knows to keep going past them, while insertion is still allowed to reuse that slot for a new element:

```c
static PyObject _dummy_struct;
#define dummy (&_dummy_struct)
```

This is exactly why `fill` (active + dummy slots) and `used` (active-only) are tracked separately — `fill` reflects how "worn" the probe sequences have become, which matters for deciding when to resize and rebuild the table from scratch (see Section 3).

### 2.5 Why Collisions Make Hash Quality Matter

If many of your elements' `__hash__` values collide (or worse, are all identical, as in the demo above), set operations degrade from average-case O(1) toward worst-case O(n) — every lookup has to probe through the whole cluster of colliding elements. This is precisely *why* a well-distributed hash function matters, and it's the underlying motivation for the next two sections: CPython keeps the table no more than ~60% full (so most probe sequences stay short) and randomizes string hashing (so an attacker can't deliberately engineer mass collisions).

---

## 3. Load Factor

### 3.1 Definition

**Load factor** is the fraction of a hash table's slots that are currently occupied:

```
load factor = (active + dummy entries) / table size
            =  fill / size
```

The higher the load factor, the more often probing collides with an occupied slot, and the slower lookups/insertions get on average. Every hash table implementation has to pick a **threshold** above which it grows the table and redistributes everything — too low wastes memory; too high makes every operation slower.

### 3.2 CPython's Exact Threshold

Straight from `set_add_entry()` in `Objects/setobject.c`:

```c
if ((size_t)so->fill * 5 < mask * 3)
    return 0;                                            /* no resize needed yet */
return set_table_resize(so, so->used > 50000 ? so->used * 2 : so->used * 4);
```

Rearranged, `fill * 5 < mask * 3` means: *keep growing without resizing as long as* `fill / size < 3/5`. In other words: **CPython resizes a set's table once it's about 60% full** (`mask` is `size - 1`, so for any reasonably-sized table this ratio is effectively `fill/size`).

### 3.3 The Growth Multiplier

When a resize *does* trigger, the new minimum required size is:

- **`used * 4`** for sets with 50,000 or fewer elements — i.e., grow to roughly **4×** the current element count.
- **`used * 2`** once the set exceeds 50,000 elements — a gentler **2×** growth, since at that scale quadrupling would waste a lot of memory for not much practical benefit.

That "minimum required size" is then rounded **up to the next power of two** (table sizes in CPython's set/dict implementation are always powers of two, so the `& mask` trick can substitute for the more expensive `%` modulo operation):

```c
size_t newsize = PySet_MINSIZE;          /* 8 */
while (newsize <= (size_t)minused) {
    newsize <<= 1;                        /* keep doubling until big enough */
}
```

### 3.4 Worked Example

Starting from an empty set (`size = 8`, threshold to resize = `fill ≥ ⌈8 × 3/5⌉ = 5`):

| Adding element # | `fill` after add | Resize triggered? | New table size |
|---|---|---|---|
| 1–4 | 1–4 | No (`4×5=20 < 7×3=21`) | stays 8 |
| 5 | 5 | **Yes** (`5×5=25 ≥ 21`) | `used*4 = 20` → round up to **32** |
| ... | ... | grows again once ~60% of 32 (≈19) is filled | next power of two ≥ `19*4=76` → **128** |

This lines up with what you actually observe via `sys.getsizeof()` — note the jump happens at `len=5` (matching the threshold computed above) and again once the set's grown enough to cross 60% of the new, larger table:

```
len= 1  getsizeof=216
len= 5  getsizeof=728     ← resize: 8 slots → 32 slots
len=19  getsizeof=2264    ← resize: 32 slots → 128 slots
```

### 3.5 Why 60%, and Why Power-of-Two Growth?

- **60% balances memory against speed.** A lower threshold (resize earlier, e.g. at 40%) wastes memory holding mostly-empty tables. A higher one (e.g. 90%) saves memory but lets probe sequences get long and slow before a resize ever happens.
- **Always sizing to powers of two** lets every "mod table size" operation become a fast bitwise AND (`hash & mask`) instead of a true division-based modulo — a meaningful speed win, since this happens on every single set operation.
- **Growing by 4× (not just 2×)** when the set is still small means small sets resize *rarely* — most of the cost of building a small-to-medium set happens in just one or two resize events, not a long tail of them.

### 3.6 Shrinking on Deletion

Notably, **CPython does not automatically shrink a set's table when elements are removed** — only insertions trigger the resize check shown above. A set that grew to hold a million elements and then had 999,000 of them removed keeps the large table (mostly empty/dummy slots) until enough new elements are inserted to trigger a fresh resize-and-rebuild — at which point dummy entries get purged and the table can shrink to fit the current `used` count.

### 3.7 Practical Implication

If you know roughly how many elements a set will eventually hold, you generally don't need to do anything special — but if you're building a very large set incrementally inside a tight loop, be aware that you'll incur a handful of full table-rebuild events along the way (each O(n) at the moment it happens, though amortized O(1) per insertion overall, exactly like the list resizing strategy covered in Module 10).

---

## 4. Hash Randomization

### 4.1 The Problem It Solves: Hash-Flooding (Algorithmic Complexity) Attacks

If an attacker can predict exactly how your hash function maps strings to slots, they can craft a batch of strings that **all collide into the same slot on purpose** — turning every lookup involving those strings from O(1) into O(n), and a whole request handling many such strings into O(n²). Submitted as form data, JSON keys, or any other attacker-controlled input processed into a `dict` or `set`, this becomes a practical **denial-of-service vector** (this exact attack was demonstrated publicly against multiple languages' hash implementations around 2011–2012, prompting fixes across the ecosystem).

### 4.2 The Fix: Salt the Hash With a Random Seed

Since **Python 3.3** (PEP 456 / earlier PEP 0029-era work), CPython randomizes the hash of `str`, `bytes`, and `datetime` objects by mixing in a random value (a "salt") generated **fresh each time the interpreter process starts**. The hash algorithm itself is **SipHash** (a fast, cryptographically-motivated hash function designed specifically to resist this kind of attack), keyed by that per-process random salt.

```python
>>> hash('hello')      # run 1 (fresh process)
-804049323645942343
>>> hash('hello')      # run 2 (a different fresh process)
-56665481643861534
>>> hash('hello')      # run 3 (yet another fresh process)
-3906680965699850943
```

Each of those was a **separate process** — note the hash is different every single time, because each process picked a new random salt at startup. **Within one running process**, `hash('hello')` always returns the *same* value every time you call it — randomization happens once, at startup, not on every call:

```python
>>> hash('hello') == hash('hello')   # same process, same value, always
True
```

### 4.3 Which Types Are Affected

| Type | Randomized? |
|---|---|
| `str` | **Yes** |
| `bytes` | **Yes** |
| `datetime.date` / `datetime.datetime` | **Yes** (derived from their string-like internals) |
| `int` | **No** — `hash(n) == n` for most ints (with a special rule for very large ones), deterministic across processes |
| `float` | **No** — deterministic |
| `tuple` (of non-randomized elements) | Deterministic, *unless* it contains a randomized type like `str` |

```python
>>> hash(42)            # process 1
42
>>> hash(42)            # process 2
42                        # identical — ints are never randomized
>>> hash(42.5)
1152921504606847018       # also identical across processes
```

This selective scope makes sense: the attack this defends against specifically targets **attacker-supplied text data** (form fields, JSON keys, query parameters) — numeric types aren't the practical attack surface, and randomizing them would only break things (like needing numeric hashes to be predictable for some interop/serialization edge cases) without closing any real vulnerability.

### 4.4 Controlling It: `PYTHONHASHSEED`

You can disable or pin randomization with the `PYTHONHASHSEED` environment variable — almost always for **reproducibility in testing/debugging**, never in production, since that reintroduces the exact vulnerability randomization exists to prevent:

```bash
PYTHONHASHSEED=0 python3 -c "print(hash('hello'))"   # deterministic, fixed seed
PYTHONHASHSEED=0 python3 -c "print(hash('hello'))"   # SAME value every time now
```

Measured:

```
$ PYTHONHASHSEED=0 python3 -c "print(hash('hello'))"
-2096571579003691106
$ PYTHONHASHSEED=0 python3 -c "print(hash('hello'))"
-2096571579003691106
$ PYTHONHASHSEED=0 python3 -c "print(hash('hello'))"
-2096571579003691106
```

| `PYTHONHASHSEED` value | Effect |
|---|---|
| unset (default) | Random seed chosen fresh every process start — randomization **on** |
| `0` | Randomization **disabled entirely** — same behavior as Python versions before this feature existed |
| any other integer | Used as a **fixed** seed — reproducible across runs, but still "randomized" relative to seed `0` |

You can check whether randomization is active in the current process via `sys.flags.hash_randomization` (`1` = on, the default).

### 4.5 Why This Matters for `set` and `dict` Specifically

Since `set` membership and `dict` key lookups are *entirely* driven by `hash(element) & mask`, randomizing the hash function means an attacker who doesn't know your process's random seed **cannot predict which elements will collide** — so they can't engineer the worst-case O(n²) scenario described in 4.1. This is a direct, practical consequence of everything in Sections 1–3: the *data structure* (hash table) creates the *vulnerability* (predictable collisions), and *hash randomization* closes it at the hash-function level rather than by changing the data structure itself.

### 4.6 Practical Consequences You Might Run Into

- **Don't rely on `hash()` output being stable across separate Python process runs** for anything you persist to disk or send over a network (caching schemes, serialized hash-based indexes) — it will differ on the next run unless `PYTHONHASHSEED` is pinned.
- **Set/dict iteration order can differ between runs of the same program**, even with identical insertion sequences, purely because the hash values (and therefore the slot each string lands in) differ per process.
- **Test suites that need reproducible output** (e.g. asserting on the exact printed contents of a set) should pin `PYTHONHASHSEED` rather than relying on insertion order or sorting before comparing.

---

## Summary Cheat-Sheet

```
Hash table     → array of slots; index = hash(x) & mask (mask = size-1, size = power of 2);
                 set entries cache BOTH the element pointer and its hash
Collisions     → open addressing: check up to 9 nearby slots linearly (cache-friendly),
                 then jump via i = (i*5 + 1 + perturb) & mask, perturb >>= 5 each round
Deletions      → leave a "dummy" marker, not an empty slot — so later probes for OTHER
                 colliding elements don't stop early and report false negatives
Load factor    → fill/size; CPython resizes once fill*5 >= mask*3  (≈60% full);
                 grows to next power of two ≥ used*4 (or used*2 above 50,000 elements)
Hash random.   → str/bytes/datetime hashes are salted with a random per-process seed
                 (SipHash) to prevent attacker-crafted hash collisions (DoS);
                 int/float hashes are NOT randomized; pin with PYTHONHASHSEED for
                 reproducible debugging only, never in production
```
