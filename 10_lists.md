# 10. Lists — Complete Reference

A Python `list` is CPython's implementation of a **dynamic array** — a resizable, ordered, mutable sequence that can hold objects of any type, in any mix. To really understand *why* lists behave (and perform) the way they do, you have to look past the Python-level API and into how CPython actually stores them in memory.

```
[10, "hello", 3.14, [1, 2]]   ← one list, four elements, four different types
```

---

## Table of Contents

1. [Dynamic Array Internals](#1-dynamic-array-internals)
2. [Resizing Strategy](#2-resizing-strategy)
3. [Memory Layout](#3-memory-layout)
4. [Comprehensions](#4-comprehensions)
5. [Nested Lists](#5-nested-lists)

---

## 1. Dynamic Array Internals

### 1.1 What "Dynamic Array" Means

A plain array (like a C array) has a **fixed size** decided at creation — you can't add a 6th element to a 5-slot array without making a whole new array. A **dynamic array** solves this by:

1. Allocating more space than is currently needed (spare capacity).
2. Tracking how much of that space is actually in use.
3. Transparently allocating a *bigger* block and copying everything over, only when the spare capacity runs out.

This is exactly what Python's `list` does. It is **not** a linked list, and it is **not** infinitely resizable for free — every `list` is, under the hood, a contiguous block of pointers with some unused room at the end.

### 1.2 The C Struct

CPython defines `list` with this struct (simplified from `Include/cpython/listobject.h`):

```c
typedef struct {
    PyObject_VAR_HEAD       /* expands to: ob_refcnt, ob_type, ob_size  */
    PyObject **ob_item;     /* pointer to a contiguous array of object pointers */
    Py_ssize_t allocated;   /* number of slots currently allocated (capacity) */
} PyListObject;
```

| Field | Meaning |
|---|---|
| `ob_size` (inside `PyObject_VAR_HEAD`) | The **logical length** — what `len(my_list)` returns |
| `ob_item` | A pointer to a separate, contiguous block of `PyObject*` pointers (the actual storage) |
| `allocated` | How many slots that block *can* hold right now — always `>= ob_size` |

The crucial insight: **`ob_size` and `allocated` are two different numbers.** `len()` only ever shows you `ob_size`. The list object is almost always carrying spare, unused capacity that you can't see from the Python level — you can only see it indirectly via `sys.getsizeof()`.

### 1.3 Lists Store *Pointers*, Not Values

`ob_item` is an array of **pointers to `PyObject`s**, not the objects themselves. Every element of a Python list — even a list of plain integers — is a reference to an object that lives elsewhere on the heap.

```python
>>> a = [1, 2, 3]
>>> b = [1, 2, 999999]
>>> a[0] is 1            # small ints (-5 to 256) are cached/interned by CPython
True
```

This is *why* a Python list can hold mixed types in the first place: every slot is just a generic pointer, so it's equally happy pointing at an `int`, a `str`, another `list`, or anything else. The cost of this flexibility is an extra layer of **indirection** — to read `a[0]`, CPython must follow the pointer in slot 0 to wherever that integer object actually lives, rather than reading a raw value directly out of the array (which is what `array.array` or a NumPy array does instead — see [Section 3.5](#35-lists-vs-arrayarray-vs-numpy-array)).

### 1.4 Amortized O(1) Append — the Payoff

Because of the spare capacity, `list.append()` is usually just:

```c
ob_item[ob_size] = new_object;
ob_size += 1;
```

— an O(1) operation: write a pointer, bump a counter. **Occasionally**, when `ob_size` would exceed `allocated`, a full resize-and-copy happens (O(n) for that one call). Averaged ("amortized") over many appends, this works out to **O(1) per append** — this is the whole point of over-allocating, covered next.

---

## 2. Resizing Strategy

### 2.1 The Naive (Bad) Strategy

If a list grew by exactly 1 slot every time you appended 1 element, every single `append()` would force a full reallocation and copy of the entire array — turning what looks like a cheap operation into an O(n) one, and building a list of n elements would cost O(n²) overall. Nobody implements it this way.

### 2.2 CPython's Actual Strategy: Over-allocation

When CPython's `list_resize()` (in `Objects/listobject.c`) needs more room, it doesn't allocate exactly what's needed — it **over-allocates**, growing by roughly **1/8th (12.5%) plus a small constant**:

```c
/* CPython source, Objects/listobject.c — list_resize() */
new_allocated = ((size_t)newsize + (newsize >> 3) + (newsize < 9 ? 3 : 6)) & ~(size_t)3;
```

In plain terms: *take the size you need, add one-eighth of it, add a small constant, then round to a multiple of 4.* The exact constants have shifted slightly across Python versions (it's an implementation detail, not a language guarantee), but the **growth factor has always been close to 1.125×** — deliberately gentler than the 2× (doubling) strategy used by some other languages' dynamic arrays, as a memory/speed tradeoff tuned for typical Python usage patterns.

### 2.3 Watching It Happen

You can observe the real capacity (not just the length) by comparing `sys.getsizeof()` before and after appends, since each pointer slot is 8 bytes on a 64-bit build:

```python
import sys

lst = []
base = sys.getsizeof(lst)          # bytes used by an empty list's header
for i in range(20):
    lst.append(i)
    capacity = (sys.getsizeof(lst) - base) // 8
    print(f"len={len(lst):2d}  capacity={capacity}")
```

Actual measured output (CPython 3.12, 64-bit):

```
len= 1  capacity=4
len= 2  capacity=4
len= 3  capacity=4
len= 4  capacity=4
len= 5  capacity=8     ← resized: old capacity (4) used up
len= 6  capacity=8
len= 7  capacity=8
len= 8  capacity=8
len= 9  capacity=16    ← resized again
...
len=17  capacity=24
len=25  capacity=32
len=33  capacity=40
len=41  capacity=52
len=53  capacity=64
len=65  capacity=76
len=77  capacity=92
len=93  capacity=108
len=109 capacity=128
```

Notice: capacity jumps happen **less and less often relative to how many elements you've added** — by the time the list has 100 elements, it might go hundreds more appends before resizing again. That's amortized O(1) in action: most appends just write into already-allocated spare room; resizes get rarer (in append-count terms) as the list grows, even though each individual resize copies more data.

> **Caveat:** these exact numbers are a CPython 3.12 implementation detail, not part of the Python language specification. Other versions, and other implementations (PyPy, etc.), are free to use a different growth strategy. Never write code that depends on the *exact* capacity of a list.

### 2.4 Shrinking

Lists also shrink (e.g. after `del`, `pop()`, or `clear()`), but CPython is conservative about it — it only releases the over-allocated memory back to the system once the list's used size drops to roughly **half** of its allocated capacity, to avoid repeatedly shrinking and re-growing (a "thrashing" pattern) if you're popping and appending in a loop.

### 2.5 Why This Matters in Practice

```python
# If you already know the final size, pre-sizing avoids ALL intermediate resizes:
n = 1_000_000
fast = [None] * n              # one allocation, then fill by index
for i in range(n):
    fast[i] = i * 2

# vs. growing one element at a time — still fast (amortized O(1)), but
# does cause a handful of reallocation-and-copy events along the way
slow = []
for i in range(n):
    slow.append(i * 2)
```

`list(range(n))` and list comprehensions are even better than both of the above in practice — CPython can often size the result list correctly **up front** when it knows (or can estimate) how many items are coming, skipping incremental resizing entirely.

---

## 3. Memory Layout

### 3.1 Two-Level Structure

A Python list is really **two separate memory allocations** working together:

```
┌─────────────────────────────┐
│  PyListObject (fixed header) │   ← one small, fixed-size block
│  ob_refcnt                   │
│  ob_type                     │
│  ob_size      = 3            │   (logical length: len() returns this)
│  allocated    = 4            │   (capacity: how many slots exist)
│  ob_item    ──┼───────────┐  │
└───────────────┼───────────┼──┘
                 │           │
                 ▼           │
   ┌─────────┬─────────┬─────────┬─────────┐
   │ slot[0] │ slot[1] │ slot[2] │ slot[3]  │   ← separate block: array of
   │  ptr ●  │  ptr ●  │  ptr ●  │ (unused) │     PyObject* pointers (8 bytes each, 64-bit)
   └────┼────┴────┼────┴────┼────┴──────────┘
        │         │         │
        ▼         ▼         ▼
     ┌─────┐   ┌─────┐   ┌─────┐
     │ int │   │ str │   │list │     ← the ACTUAL objects, scattered
     │  10 │   │"hi" │   │[1,2]│       wherever on the heap, each with
     └─────┘   └─────┘   └─────┘       its own refcount, type, data
```

The header block stays small and fixed-size no matter how big the list gets — it's the `ob_item` block (the pointer array) that grows according to the resizing strategy above.

### 3.2 Measuring It

```python
>>> import sys
>>> sys.getsizeof([])
56                       # empty list: just the header, allocated=0
>>> sys.getsizeof([1, 2, 3])
88                       # header (56) + 4 pointer slots × 8 bytes = 88
>>> sys.getsizeof([1,2,3,4,5,6,7,8,9,10])
136                      # header (56) + 10 pointer slots × 8 bytes = 136
```

Note `sys.getsizeof()` on a list **only counts the header + the pointer array** — it does *not* count the size of the objects the pointers point to. The integers, strings, or nested lists *inside* a list live in their own separately-allocated memory and are not part of the list's own footprint:

```python
>>> sys.getsizeof([1, 2, 3])           # the list itself
88
>>> sys.getsizeof(1) * 3               # NOT how much memory the elements use —
                                        # ints 1, 2, 3 are cached/shared anyway
```

### 3.3 Pointer Sharing

Because slots store pointers, two different lists can point at the *exact same* underlying object:

```python
>>> shared = [100, 200]
>>> list_a = [shared, "x"]
>>> list_b = [shared, "y"]
>>> list_a[0] is list_b[0]
True            # both pointers point to the SAME inner list object
>>> list_a[0].append(300)
>>> list_b[0]
[100, 200, 300]   # visible through list_b too — same object!
```

This is the root cause of most "nested list" bugs — see [Section 5](#5-nested-lists).

### 3.4 Why Slicing and Concatenation Create New Pointer Arrays

```python
>>> a = [1, 2, 3]
>>> b = a[:]              # slicing → brand-new list, new ob_item array,
>>> a is b                #            COPIES the pointers (not the objects)
False
>>> a[0] is b[0]
True                      # same underlying int object — copy is "shallow"
```

`a[:]`, `list(a)`, `a.copy()`, and `a + b` all allocate a fresh `ob_item` pointer array and copy the *pointers* across — never the objects those pointers refer to. That's exactly what "shallow copy" means at the implementation level.

### 3.5 Lists vs. `array.array` vs. NumPy Array

| Structure | Stores | Memory per element | Type |
|---|---|---|---|
| `list` | Pointers to objects | 8 bytes (pointer) **+** the object's own full size, elsewhere | Heterogeneous — any type |
| `array.array('i', ...)` | Raw values, packed contiguously | exactly `sizeof(C type)`, e.g. 4 bytes for `'i'` | Homogeneous — one C type only |
| `numpy.array` | Raw values, packed contiguously, with vectorized C operations | Depends on `dtype`, e.g. 8 bytes for `float64` | Homogeneous — one dtype |

```python
>>> import sys, array
>>> sys.getsizeof([1, 2, 3, 4, 5])           # list of 5 small ints
96
>>> sys.getsizeof(array.array('i', [1,2,3,4,5]))   # packed C ints
84                                            # much closer to raw 5×4=20 bytes + header
```

This is *why* NumPy is dramatically faster and more memory-efficient for large numeric data: it skips the pointer-indirection layer entirely and stores raw numbers back-to-back, which also lets the CPU use cache-friendly, vectorized operations.

---

## 4. Comprehensions

A **list comprehension** builds a new list in a single, declarative expression instead of a manual loop with `.append()`.

### 4.1 Basic Syntax

```python
[expression for item in iterable]
```

```python
>>> squares = [x**2 for x in range(6)]
>>> squares
[0, 1, 4, 9, 16, 25]
```

is the comprehension equivalent of:

```python
squares = []
for x in range(6):
    squares.append(x**2)
```

### 4.2 With a Filter Condition

```python
[expression for item in iterable if condition]
```

```python
>>> evens = [x for x in range(20) if x % 2 == 0]
>>> evens
[0, 2, 4, 6, 8, 10, 12, 14, 16, 18]
```

### 4.3 Conditional Expression *Inside* the Output

This is different from a filter — it doesn't drop items, it **transforms** them based on a condition (note the `if/else` comes *before* the `for`, with no filtering `if` clause at all):

```python
>>> labels = ["even" if x % 2 == 0 else "odd" for x in range(5)]
>>> labels
['even', 'odd', 'even', 'odd', 'even']
```

You can combine both — a transforming `if/else` *and* a filtering `if`:

```python
>>> [("even" if x % 2 == 0 else "odd") for x in range(10) if x > 5]
['odd', 'even', 'odd', 'even']
```

### 4.4 Multiple `for` Clauses (Nested Loops)

Comprehensions can contain more than one `for`, processed **left to right**, exactly like nested `for` loops written out longhand:

```python
>>> pairs = [(x, y) for x in range(3) for y in range(2)]
>>> pairs
[(0, 0), (0, 1), (1, 0), (1, 1), (2, 0), (2, 1)]
```

is equivalent to:

```python
pairs = []
for x in range(3):       # outer loop comes FIRST in the comprehension
    for y in range(2):   # inner loop comes SECOND
        pairs.append((x, y))
```

This is also the standard idiom for **flattening a nested list**:

```python
>>> nested = [[1, 2], [3, 4], [5, 6]]
>>> flat = [item for sublist in nested for item in sublist]
>>> flat
[1, 2, 3, 4, 5, 6]
```

### 4.5 Nested Comprehensions (a Comprehension *as* the Expression)

Don't confuse "multiple `for` clauses" (above) with a comprehension *nested inside* another comprehension's expression — used to build a list of lists (e.g. a matrix):

```python
>>> matrix = [[row * 3 + col for col in range(3)] for row in range(3)]
>>> matrix
[[0, 1, 2], [3, 4, 5], [6, 7, 8]]
```

Here the **inner** `[row * 3 + col for col in range(3)]` is itself a full comprehension, evaluated fresh for every value of `row` in the **outer** comprehension — producing a genuinely independent list each time (no shared-reference pitfall — see [Section 5.2](#52-the-classic-pitfall-multiplying-a-list-of-lists)).

### 4.6 Comprehensions Have Their Own Scope

Since Python 3, the loop variable inside a comprehension does **not** leak into the surrounding scope (unlike a plain `for` loop, where it does):

```python
>>> x = "outer"
>>> result = [x for x in range(3)]
>>> x
'outer'        # untouched — the comprehension's `x` was local to the comprehension
```

### 4.7 The Walrus Operator (`:=`) Inside Comprehensions

Lets you compute a value once, use it in the condition, *and* reuse it in the output expression — avoiding redundant computation:

```python
>>> data = [1, 2, 3, 4, 5]
>>> [y for x in data if (y := x * 2) > 4]
[6, 8, 10]
```

Without the walrus, you'd either compute `x * 2` twice, or fall back to a manual loop.

### 4.8 List Comprehension vs. `map()`/`filter()`

```python
>>> list(map(lambda x: x**2, range(6)))      # functional style
[0, 1, 4, 9, 16, 25]
>>> [x**2 for x in range(6)]                  # comprehension — more readable for most cases
[0, 1, 4, 9, 16, 25]
```

Comprehensions are generally preferred in idiomatic Python for anything beyond a single, simple function call — they read left-to-right like English and don't need a `lambda`.

### 4.9 Why Comprehensions Are Faster Than a Manual Append Loop

This isn't just style — it's measurably faster, because of how each compiles to bytecode. A manual loop has to repeatedly look up the `.append` **method** (an attribute lookup) on every iteration, then call it like any other function call. A comprehension compiles to a **dedicated `LIST_APPEND` bytecode instruction** that pushes directly onto the list's internal array, skipping the attribute lookup and function-call overhead entirely:

```python
import dis
dis.dis(compile("[i*2 for i in range(5)]", "<string>", "eval"))
```
```
...
 18 GET_ITER
 20 FOR_ITER          ...
 24 LOAD_FAST       (i)
 26 LOAD_CONST      (2)
 28 BINARY_OP       (*)
 30 LIST_APPEND     2        ← direct, specialized opcode — no method lookup
 32 JUMP_BACKWARD   ...
```

This is also part of why `list(generator_expression)` and `[x for x in ...]` tend to outperform building the same list via a `for` loop with `result.append(x)` — fewer Python-level operations execute per item.

### 4.10 Comprehension vs. Generator Expression

Square brackets `[...]` build the **entire list in memory immediately**. Parentheses `(...)` create a **generator expression** — a lazy iterator that produces values one at a time, on demand, without ever materializing the whole sequence:

```python
>>> list_comp = [x**2 for x in range(1_000_000)]   # builds 1M items NOW, uses real memory
>>> gen_expr  = (x**2 for x in range(1_000_000))    # builds nothing yet — just an iterator
>>> sys.getsizeof(list_comp)
8448728
>>> sys.getsizeof(gen_expr)
200                                                  # tiny — regardless of "size"
```

Use a generator expression when you're going to consume the values once, in order (e.g. feeding `sum()`, `for x in ...`, or another function) and don't need random access or the full list kept around.

---

## 5. Nested Lists

A "nested list" is simply a list whose elements are themselves lists (or other lists, arbitrarily deep) — Python's equivalent of a matrix, grid, or tree, since there's no dedicated multi-dimensional array type built into the language itself.

### 5.1 Basic Access

```python
>>> grid = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
>>> grid[0]            # first row
[1, 2, 3]
>>> grid[0][2]          # row 0, column 2
3
>>> grid[1][1] = 99     # mutate a single cell
>>> grid
[[1, 2, 3], [4, 99, 6], [7, 8, 9]]
```

### 5.2 The Classic Pitfall: Multiplying a List of Lists

This is, by a wide margin, the single most common nested-list bug in Python:

```python
>>> matrix = [[0] * 3] * 3      # looks reasonable... but is NOT 3 independent rows
>>> matrix
[[0, 0, 0], [0, 0, 0], [0, 0, 0]]
>>> matrix[0][0] = 99
>>> matrix
[[99, 0, 0], [99, 0, 0], [99, 0, 0]]    # ⚠ ALL THREE rows changed!
>>> [id(row) for row in matrix]
[139934307865152, 139934307865152, 139934307865152]   # same object, 3 times
```

**Why this happens:** `[0] * 3` creates one inner list `[0, 0, 0]`. The outer `* 3` then makes a new *outer* list containing **three pointers to that same single inner list object** — it never calls anything that would create three separate inner lists. Multiplication on a list just copies *references*, the same way assignment does.

**The fix** — use a comprehension, so the inner list expression is **re-evaluated fresh** for each iteration, genuinely creating a new object every time:

```python
>>> matrix = [[0] * 3 for _ in range(3)]   # the inner [0]*3 runs 3 SEPARATE times
>>> matrix[0][0] = 99
>>> matrix
[[99, 0, 0], [0, 0, 0], [0, 0, 0]]          # only row 0 changed — correct!
>>> [id(row) for row in matrix]
[139934303890432, 139934303892416, 139934303892480]   # three distinct objects
```

> This pitfall does **not** affect flat lists of immutable values — `[0, 0, 0] * 3` is perfectly safe, because there's no mutable inner object being shared; you're just duplicating pointers to the same immutable `0`, and immutable objects can never be "corrupted" by sharing.

### 5.3 Shallow Copy vs. Deep Copy

`list.copy()`, `list(x)`, and `x[:]` all perform a **shallow copy** — they create a new *outer* list, but the elements inside (if mutable, like nested lists) are still shared pointers to the *same* inner objects:

```python
>>> import copy
>>> original = [[1, 2], [3, 4]]
>>> shallow = original.copy()
>>> shallow[0][0] = 999
>>> original
[[999, 2], [3, 4]]      # ⚠ the original changed too! inner lists are shared
```

`copy.deepcopy()` recursively copies *everything*, all the way down, so nothing is shared at any depth:

```python
>>> original = [[1, 2], [3, 4]]
>>> deep = copy.deepcopy(original)
>>> deep[0][0] = 999
>>> original
[[1, 2], [3, 4]]        # untouched — deep copy made fully independent inner lists
```

| Copy method | Outer list | Inner (nested) lists |
|---|---|---|
| `b = a` | Same object (not a copy at all) | Same objects |
| `b = a.copy()` / `a[:]` / `list(a)` | New object | **Shared** with `a` |
| `b = copy.deepcopy(a)` | New object | **Independent**, recursively |

### 5.4 Iterating Nested Lists

```python
grid = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]

# row by row
for row in grid:
    print(row)

# every individual element
for row in grid:
    for value in row:
        print(value, end=" ")

# with indices, when you need to write back into the grid
for r in range(len(grid)):
    for c in range(len(grid[r])):
        grid[r][c] *= 10
```

### 5.5 Flattening a Nested List

```python
nested = [[1, 2], [3, 4], [5, 6]]

# comprehension with two `for` clauses (see Section 4.4)
flat = [item for sublist in nested for item in sublist]

# itertools.chain — efficient, doesn't build intermediate lists
import itertools
flat = list(itertools.chain.from_iterable(nested))

# sum() trick — works, but is O(n²) for large inputs (avoid for big data!)
flat = sum(nested, [])

print(flat)   # [1, 2, 3, 4, 5, 6]
```

For **arbitrarily deep** nesting (unknown how many levels down), a recursive function is needed:

```python
def flatten(lst):
    result = []
    for item in lst:
        if isinstance(item, list):
            result.extend(flatten(item))   # recurse into the nested list
        else:
            result.append(item)
    return result

deep_nested = [1, [2, 3, [4, 5, [6, 7]], 8], 9]
print(flatten(deep_nested))
# [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

### 5.6 Jagged (Irregular) Nested Lists

Unlike a true 2D array, nothing forces every row to be the same length — Python nested lists are perfectly happy being "jagged":

```python
>>> jagged = [[1], [2, 3], [4, 5, 6]]
>>> [len(row) for row in jagged]
[1, 2, 3]
```

This flexibility is convenient, but it also means you can't always assume `len(grid[0]) == len(grid[1])` — defensive code that processes nested lists from untrusted sources should check row lengths if uniformity matters.

### 5.7 When to Reach for Something Other Than Nested Lists

Nested lists work fine for small grids and simple data, but for serious numeric or large-scale grid work, prefer:

- **NumPy arrays** (`numpy.ndarray`) — true multi-dimensional arrays, contiguous memory, vectorized math, far less overhead (see [Section 3.5](#35-lists-vs-arrayarray-vs-numpy-array)).
- **`list` of `tuple`** rows if the rows should be immutable.
- A **`dict`-of-coordinates** (e.g. `{(row, col): value}`) for sparse grids where most cells are empty/default.

---

## Summary Cheat-Sheet

```
Dynamic array     → list = header (ob_size, allocated) + separate pointer array (ob_item)
Resizing          → over-allocates ~1.125× + small constant; NOT exact doubling;
                     gives amortized O(1) append
Memory layout     → list stores POINTERS, not values; sys.getsizeof() only counts the
                     header + pointer array, never the pointed-to objects
Comprehensions    → [expr for x in it if cond]; own scope (Py3); compiles to a dedicated
                     LIST_APPEND opcode, faster than a manual .append() loop
[x]*n pitfall     → [[0]*3]*3 shares ONE inner list 3×; use [[0]*3 for _ in range(3)]
Copy depth        → .copy()/[:] = shallow (inner lists shared); copy.deepcopy() = fully
                     independent at every level
Flatten           → [v for row in nested for v in row]  /  itertools.chain.from_iterable
```
