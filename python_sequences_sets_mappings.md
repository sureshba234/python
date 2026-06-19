# Python Data Types — Sequences, Sets & Mappings (Complete Reference)

Covers: `str`, `list`, `tuple`, `range` (Sequences) · `set`, `frozenset` (Sets) · `dict` (Mappings)

Each topic follows: Difficulty → Definition → Purpose → Syntax → Examples → DSA Concepts → Internal Working → Memory Behavior → Time Complexity → Space Complexity → Optimization → Common Mistakes → Best Practices → Interview Questions → Real-World Usage → Related Topics

---

# 1. `str` (String)

### Difficulty
[Beginner] → [Intermediate] for internals (interning, encoding), [Advanced] for algorithmic string problems.

### Definition
A `str` is an **immutable, ordered sequence of Unicode characters**. Once created, its contents can never be changed — every "modification" produces a brand-new string object.

### Purpose
To represent and manipulate text: names, messages, file contents, parsing, formatting, pattern matching, etc.

### Syntax
```python
s1 = 'hello'
s2 = "world"
s3 = '''multi
line'''
s4 = str(123)        # "123"
s5 = f"value: {s1}"  # f-string
```

### Examples
```python
name = "Claude"
print(name.upper())        # CLAUDE
print(name[0])              # C
print(name[::-1])           # edualC (reversed via slicing)
print(name + " AI")         # Claude AI
print(len(name))            # 6
print("a" in name)          # False
print(name.replace("C", "K"))  # Klaude
```

### DSA Questions and Concepts
- Reverse a string / check palindrome
- Check if two strings are anagrams
- Longest substring without repeating characters
- String compression (run-length encoding)
- Implement `strstr()` (substring search) — naive vs KMP vs Z-algorithm
- Longest common prefix / longest common subsequence
- Valid parentheses (using string as stack source)

### Internal Working
- CPython stores strings as **arrays of Unicode code points** (PEP 393 "flexible string representation") — internally uses 1, 2, or 4 bytes per character depending on the widest character present (Latin-1, UCS-2, or UCS-4), to save memory.
- Strings are **immutable**: any operation like concatenation or replacement creates a *new* string object in memory; the old one is left untouched (and garbage collected if unreferenced).
- **String interning**: CPython automatically caches and reuses small string literals (identifiers-like strings, e.g., `"hello"`) so that `a = "hello"; b = "hello"` may point to the *same* object in memory (`a is b` → `True`), as a memory optimization. This isn't guaranteed for all strings (e.g., very long ones or ones built at runtime).

### Memory Behavior
- Strings have memory overhead beyond raw character data (Python object header: type pointer, reference count, length, hash cache, etc.).
- Because of immutability, repeated concatenation (`s = s + "x"` in a loop) is **memory- and time-inefficient** — each `+` creates a new string and copies all old data.
- Use `''.join(list_of_strings)` instead — builds the final string in one pass.

### Time Complexity
| Operation | Complexity |
|---|---|
| Indexing `s[i]` | O(1) |
| Slicing `s[a:b]` | O(k) where k = slice length |
| Concatenation `s1 + s2` | O(n + m) |
| `in` (substring search) | O(n*m) worst case (naive) |
| `len(s)` | O(1) (length is cached) |
| `.replace()`, `.split()`, `.join()` | O(n) |
| Iteration | O(n) |

### Space Complexity
O(n) where n = number of characters. Each modification operation that returns a new string also costs O(n) extra space.

### Optimization Techniques
- Use `str.join()` instead of `+=` in loops.
- Use f-strings (`f"{x}"`) instead of `%` formatting or `.format()` — faster and more readable.
- Use `str.translate()` with a translation table for fast character-by-character replacement (faster than chained `.replace()` calls).
- Pre-compile regex patterns with `re.compile()` if matching repeatedly.

### Common Mistakes
- Trying to mutate a string directly: `s[0] = 'H'` → raises `TypeError`.
- Using `+=` in a loop for large strings (quadratic time).
- Confusing `==` (value equality) with `is` (identity) — usually want `==` for strings.
- Forgetting that string comparison is lexicographic, not numeric: `"10" < "9"` is `True`.

### Best Practices
- Use f-strings for formatting.
- Use `.strip()`, `.lower()`/`.upper()` carefully when comparing user input.
- Prefer `str.join()` for building large strings.
- Use raw strings (`r"..."`) for regex patterns to avoid double-escaping.

### Interview Questions
1. How are strings stored internally in Python (PEP 393)?
2. Why is string concatenation in a loop inefficient? How to fix it?
3. Implement string reversal without using slicing.
4. What's the difference between `is` and `==` for strings?
5. Explain string interning.

### Real-World Usage
- Parsing logs, JSON/CSV data
- Web scraping / text processing
- Natural language processing (NLP) pipelines
- Building CLI tools, templating engines

### Related Topics
`list`, `bytes`/`bytearray`, regular expressions (`re` module), `str.format`, Unicode encoding (UTF-8)

---

# 2. `list`

### Difficulty
[Beginner] → [Intermediate]/[Advanced] for internals and amortized complexity analysis.

### Definition
A `list` is a **mutable, ordered, dynamic array** that can hold elements of any (and mixed) data types.

### Purpose
General-purpose container for ordered, changeable collections of items — the Python "workhorse" data structure.

### Syntax
```python
l1 = [1, 2, 3]
l2 = list((4, 5, 6))     # from a tuple
l3 = [x for x in range(5)]  # list comprehension
l4 = []                   # empty list
```

### Examples
```python
nums = [3, 1, 4, 1, 5]
nums.append(9)         # [3, 1, 4, 1, 5, 9]
nums.sort()             # [1, 1, 3, 4, 5, 9]
nums.pop()               # removes & returns last item (9)
nums.insert(0, 100)     # insert at index 0
print(nums[1:3])         # slicing
squares = [x**2 for x in range(5)]   # [0, 1, 4, 9, 16]
```

### DSA Questions and Concepts
- Two Sum, Three Sum (array-based)
- Reverse a list in place
- Find duplicates / missing number
- Kadane's algorithm (max subarray sum)
- Sliding window problems
- Merge intervals
- Implement a stack/queue using a list
- Binary search on a sorted list
- Sorting algorithms (bubble/merge/quick sort)

### Internal Working
- CPython implements `list` as a **dynamic array** (`PyListObject`) — a contiguous block of pointers to objects, *not* the objects themselves.
- It **over-allocates** memory: when a list grows beyond capacity, CPython grows the underlying array by roughly 1.125x plus a small constant, so repeated `append()` is amortized O(1) instead of O(n) every time.
- Since list elements are *pointers* to objects, lists can hold mixed types, at the cost of an extra indirection compared to a C array of raw values.

### Memory Behavior
- A list typically uses **more memory than needed** for its current length, due to over-allocation (trade-off for fast `append`).
- `del`, `pop()`, and slicing can shrink a list, but CPython doesn't always shrink the underlying array immediately.
- Nested lists store *references*, so `=` assignment doesn't duplicate inner lists (shallow copy issue — see Common Mistakes).

### Time Complexity
| Operation | Complexity |
|---|---|
| Indexing `l[i]` | O(1) |
| `append()` | O(1) amortized |
| `pop()` (from end) | O(1) |
| `pop(0)` / `insert(0, x)` (from front) | O(n) |
| `in` (search) | O(n) |
| `sort()` | O(n log n) (Timsort) |
| Slicing `l[a:b]` | O(k) |
| `len()` | O(1) |

### Space Complexity
O(n), with extra overhead from over-allocation (typically less than 2x the minimum needed).

### Optimization Techniques
- Use `collections.deque` for frequent front insert/remove (O(1) vs O(n)).
- Use list comprehensions instead of `for` + `.append()` — faster internally.
- Pre-allocate with `[None] * n` if final size is known.
- Use NumPy arrays for large numeric data (less overhead, vectorized math).

### Common Mistakes
- **Shallow copy trap**: `new = old` is the same object, not a copy. Use `.copy()`, `list(old)`, or `copy.deepcopy()` for nested lists.
- Mutable default argument bug:
```python
def add(item, lst=[]):   # BAD — same list reused across calls!
    lst.append(item)
    return lst
```
- Using `pop(0)` in a loop → O(n²) overall instead of `deque.popleft()`.
- Modifying a list while iterating over it.

### Best Practices
- Use list comprehensions for clarity and speed.
- Use `enumerate()` instead of manual index tracking.
- Use `deque` for queue-like behavior.
- Use `.copy()` or `[:]` when an independent copy is needed.

### Interview Questions
1. How does Python's list grow internally? Why is `append()` amortized O(1)?
2. Shallow copy vs deep copy?
3. Why is `pop(0)` slow? Faster alternative?
4. Implement a list-based stack and queue.
5. Explain the mutable default argument bug.

### Real-World Usage
- Storing records, search results, task queues
- Data preprocessing pipelines
- Stacks/queues/buffers, adjacency lists for graphs

### Related Topics
`tuple`, `array`, `collections.deque`, `numpy.ndarray`, list comprehensions

---

# 3. `tuple`

### Difficulty
[Beginner] → [Intermediate] for internals/use-cases like hashability.

### Definition
A `tuple` is an **immutable, ordered** collection of elements, similar to a list but cannot be modified after creation.

### Purpose
Used when data should not change after creation — fixed records, function returns with multiple values, dictionary keys (since tuples are hashable if their contents are), and to signal "this collection is read-only."

### Syntax
```python
t1 = (1, 2, 3)
t2 = 1, 2, 3          # parentheses optional
t3 = (5,)             # single-element tuple — comma is mandatory!
t4 = tuple([4, 5, 6]) # from a list
t5 = ()               # empty tuple
```

### Examples
```python
point = (3, 4)
x, y = point          # unpacking
print(point[0])        # 3
def min_max(nums):
    return min(nums), max(nums)   # returns a tuple
lo, hi = min_max([4, 1, 9, 2])
```

### DSA Questions and Concepts
- Using tuples as dictionary keys (e.g., coordinates in grid-based BFS/DFS)
- Storing (distance, node) pairs in heaps/priority queues (`heapq`)
- Multiple return values from functions (common in algorithm design)
- Representing edges in graphs as `(u, v)` or `(u, v, weight)`

### Internal Working
- Like lists, tuples store pointers to objects in a contiguous block, but since tuples are **fixed-size and immutable**, CPython can optimize their creation and storage — no over-allocation needed, since the size never changes.
- Tuples of small fixed sizes (especially empty tuples and short literal tuples) may be cached/reused by CPython for efficiency in some cases.
- Because they're immutable, tuples are **hashable** (if all their elements are hashable) — meaning they can be used as dictionary keys or set elements, unlike lists.

### Memory Behavior
- Tuples generally use **less memory than lists** holding the same elements, since they don't need extra capacity for growth.
- No copy-on-write needed since tuples can't be mutated — sharing referenced objects is completely safe.

### Time Complexity
| Operation | Complexity |
|---|---|
| Indexing `t[i]` | O(1) |
| Slicing | O(k) |
| Concatenation | O(n + m) |
| `in` (search) | O(n) |
| `len()` | O(1) |

### Space Complexity
O(n) — but typically less overhead than an equivalent list.

### Optimization Techniques
- Use tuples instead of lists when data is fixed — saves memory and signals intent.
- Use tuples as compound dictionary/set keys instead of converting to strings.
- Use `namedtuple` (from `collections`) for readable, self-documenting tuples with named fields.

### Common Mistakes
- Forgetting the trailing comma for single-element tuples: `(5)` is just an `int`, not a tuple — you need `(5,)`.
- Trying to mutate: `t[0] = 100` → `TypeError`.
- Confusing tuple immutability with "deep immutability" — a tuple containing a list still allows that inner list to be mutated:
```python
t = ([1, 2], 3)
t[0].append(99)   # allowed! Tuple itself wasn't changed, but its mutable contents were
```

### Best Practices
- Use tuples for fixed collections / records (e.g., RGB colors, coordinates).
- Use `namedtuple` or `dataclass` for more readable structured data.
- Prefer tuples for function returns of multiple values.

### Interview Questions
1. Why are tuples hashable but lists are not?
2. What happens if a tuple contains a mutable object — is it still "immutable"?
3. When would you choose a tuple over a list?
4. What is tuple unpacking and how is it used in swapping variables?

### Real-World Usage
- Returning multiple values from functions
- Database records (rows) are often represented as tuples
- Dictionary keys for grid coordinates in pathfinding algorithms
- Immutable configuration data

### Related Topics
`list`, `namedtuple`, `dataclasses`, hashability, dictionary keys

---

# 4. `range`

### Difficulty
[Beginner], with [Intermediate] depth around its lazy-evaluation internals.

### Definition
`range` represents an **immutable sequence of numbers**, typically used for looping a specific number of times. It does **not** store all numbers in memory — it's a lightweight object that computes values on demand.

### Purpose
Efficiently generate sequences of integers for loops, indexing, and slicing operations without the memory cost of materializing a full list.

### Syntax
```python
r1 = range(5)         # 0, 1, 2, 3, 4
r2 = range(2, 10)     # 2 to 9
r3 = range(0, 20, 2)  # 0, 2, 4, ..., 18  (start, stop, step)
r4 = range(10, 0, -1) # countdown 10 to 1
```

### Examples
```python
for i in range(5):
    print(i)             # 0 1 2 3 4

print(list(range(3, 8)))      # [3, 4, 5, 6, 7]
print(len(range(0, 100, 5)))  # 20
print(5 in range(10))          # True (O(1) check, not iteration!)
```

### DSA Questions and Concepts
- Used heavily in iterative algorithms: nested loops for matrix traversal, brute-force solutions
- Generating index ranges for binary search, two-pointer techniques
- Simulating counters in dynamic programming (filling DP tables)

### Internal Working
- `range` is implemented as a special object that stores only **three values**: `start`, `stop`, and `step`. It computes each element on the fly using simple arithmetic when indexed or iterated — it never stores the full sequence in memory.
- This is different from Python 2's `range()`, which *did* return a full list (Python 2 had `xrange()` for the lazy version — Python 3 unified this into a smart `range`).
- Because `range` knows its start/stop/step, membership tests (`x in range(...)`) and indexing are computed mathematically in O(1), not by scanning.

### Memory Behavior
- Extremely memory-efficient — `range(10**18)` takes the same tiny amount of memory as `range(5)`, since no actual numbers are stored.
- Contrast with `list(range(10**18))` which would attempt to allocate an enormous amount of memory and likely crash/hang.

### Time Complexity
| Operation | Complexity |
|---|---|
| Indexing `r[i]` | O(1) |
| `in` (membership) | O(1) |
| Iteration (full loop) | O(n) |
| `len()` | O(1) |

### Space Complexity
O(1) — constant space regardless of range size (this is its key advantage over lists).

### Optimization Techniques
- Always prefer `range()` over manually building a list of numbers for iteration.
- Use `range` with a `step` parameter to skip elements directly instead of filtering afterward.
- Combine with `enumerate()` or `zip()` for parallel iteration patterns.

### Common Mistakes
- Believing `range` produces a list — calling `range(n)[index]` works, but treating it like a list (e.g., trying `.append()`) fails since it's immutable and fixed-structure.
- Off-by-one errors: `range(5)` goes from `0` to `4`, NOT including `5`.
- Inefficiently converting to a list unnecessarily: `for i in list(range(n))` — the `list()` call is wasteful; just iterate over `range(n)` directly.

### Best Practices
- Use `range(len(my_list))` only when you need indices; otherwise iterate directly over the list or use `enumerate()`.
- Use negative steps for countdowns: `range(n, 0, -1)`.
- Avoid materializing huge ranges into lists unless truly necessary.

### Interview Questions
1. How is `range` different from a list in terms of memory?
2. Why is `x in range(a, b, c)` O(1) instead of O(n)?
3. What's the difference between Python 2's `range` and Python 3's `range`?
4. Write a function to reverse iterate over a range without using `reversed()`.

### Real-World Usage
- Loop counters in any iterative algorithm
- Pagination logic (generating page number sequences)
- Simulating array indices for matrix/grid traversal
- Batch processing data in fixed-size chunks

### Related Topics
`list`, generators, `enumerate()`, `itertools`, lazy evaluation

---

# 5. `set`

### Difficulty
[Beginner] → [Intermediate]/[Advanced] for hash table internals and complexity.

### Definition
A `set` is a **mutable, unordered collection of unique elements**. Duplicate values are automatically eliminated, and elements must be hashable.

### Purpose
Efficiently test membership, eliminate duplicates, and perform mathematical set operations (union, intersection, difference) — much faster than doing the equivalent with lists.

### Syntax
```python
s1 = {1, 2, 3}
s2 = set([1, 2, 2, 3])    # {1, 2, 3} — duplicates removed
s3 = set()                 # empty set — NOTE: {} creates a dict, not a set!
s4 = {x for x in range(5) if x % 2 == 0}   # set comprehension
```

### Examples
```python
a = {1, 2, 3}
b = {2, 3, 4}
print(a | b)    # union: {1, 2, 3, 4}
print(a & b)    # intersection: {2, 3}
print(a - b)    # difference: {1}
print(a ^ b)    # symmetric difference: {1, 4}
a.add(10)
a.discard(1)     # removes if present, no error if missing
print(3 in a)    # O(1) membership check
```

### DSA Questions and Concepts
- Detect duplicates in an array (`len(set(arr)) != len(arr)`)
- Find intersection/union of two arrays
- Check for subsets
- Longest consecutive sequence (using a set for O(1) lookups)
- Two Sum using a set for O(n) solution
- Cycle detection in graphs (visited set)
- Deduplication tasks

### Internal Working
- A `set` is implemented internally using a **hash table** — similar to `dict`, but storing only keys (no values).
- Each element's `__hash__()` value determines which "bucket" it's placed in. On insertion/lookup, Python computes the hash, jumps directly to the relevant bucket, and (using open addressing with probing) resolves any collisions.
- Because of hashing, elements **must be hashable** — meaning they must implement `__hash__` and `__eq__` consistently. This is why you *can't* put a `list` inside a `set` (lists are unhashable), but you *can* put a `tuple` (if its contents are also hashable).
- Sets, like dicts, are unordered in concept, although since Python 3.7+ iteration order tends to follow insertion order *for dicts* — this guarantee does **not** apply to sets; set order is implementation-defined and can appear arbitrary.

### Memory Behavior
- Sets use more memory per-element than lists due to the hash table structure (open slots reserved to keep load factor low and minimize collisions).
- Resizing happens automatically as the set grows, similar to dynamic array resizing in lists, but governed by hash table load-factor thresholds.

### Time Complexity
| Operation | Complexity (average) | Worst Case |
|---|---|---|
| `add()` | O(1) | O(n) |
| `remove()`/`discard()` | O(1) | O(n) |
| `in` (membership) | O(1) | O(n) |
| Union/Intersection | O(len(s1) + len(s2)) | — |
| Iteration | O(n) | — |

(Worst case O(n) only occurs with many hash collisions, which is rare with good hash functions.)

### Space Complexity
O(n), with additional overhead from the hash table's reserved empty slots (typically sets use noticeably more memory per element than an equivalent list).

### Optimization Techniques
- Use a `set` instead of a `list` whenever you only need fast membership testing, not order.
- Use set operations (`&`, `|`, `-`) instead of manual loops for combining/comparing collections — they're implemented in optimized C code.
- Use `frozenset` if the set itself needs to be hashable (e.g., as a dict key) or never modified.

### Common Mistakes
- Writing `{}` expecting an empty set — this actually creates an empty **dict**. Use `set()` instead.
- Assuming sets maintain insertion order (they don't, unlike dicts since 3.7+).
- Trying to add unhashable types (like lists) to a set → `TypeError`.
- Using a set when order matters for the output.

### Best Practices
- Use sets for deduplication and fast lookup tasks.
- Convert lists to sets only when you need set-specific operations, not just because "it might be faster" — conversion itself costs O(n).
- Use set comprehensions for cleaner code: `{x for x in items if condition}`.

### Interview Questions
1. How is a Python `set` implemented internally?
2. Why must set elements be hashable?
3. Difference between `set` and `frozenset`?
4. How would you find duplicates in an array using a set in O(n) time?
5. Why doesn't `{}` create an empty set?

### Real-World Usage
- Removing duplicate entries from datasets
- Fast membership testing (e.g., blacklists, visited nodes in graph traversal)
- Mathematical set operations in data analysis (finding common elements between datasets)
- Tracking unique visitors, tags, or categories

### Related Topics
`frozenset`, `dict` (shares hash table implementation), hashable objects, `collections.Counter`

---

# 6. `frozenset`

### Difficulty
[Beginner]/[Intermediate]

### Definition
A `frozenset` is an **immutable version of a set** — same hash-table-based uniqueness and fast membership testing, but cannot be modified after creation.

### Purpose
Used when you need set semantics (uniqueness, fast lookup, set operations) but also need the result to be hashable — e.g., to use as a dictionary key, or as an element inside another set, or simply to guarantee the collection won't be accidentally mutated.

### Syntax
```python
fs1 = frozenset([1, 2, 3])
fs2 = frozenset({4, 5, 6})
fs3 = frozenset()        # empty frozenset
```

### Examples
```python
fs = frozenset([1, 2, 3])
# fs.add(4)              # AttributeError — no mutation methods exist

s2 = frozenset([2, 3, 4])
print(fs & s2)            # frozenset({2, 3}) — intersection still works
print(fs | s2)             # union

# Using as a dict key (impossible with a regular set!)
graph_cache = {frozenset([1, 2]): "edge between 1 and 2"}
```

### DSA Questions and Concepts
- Memoization where the "state" being cached is a set of items (e.g., subset-sum/DP problems where visited combinations are stored as frozensets in a cache dictionary)
- Representing undirected graph edges as frozensets (so `{1,2}` and `{2,1}` are treated identically)
- Storing sets-of-sets (e.g., for power-set generation, since sets can't nest sets but frozensets can be nested inside sets)

### Internal Working
- Implemented using essentially the same hash table machinery as `set`, but **without** any of the mutating methods (`add`, `remove`, `discard`, `update`, etc.) exposed.
- Because it's immutable, CPython can compute and cache its hash value once, making `frozenset` itself hashable — this is the entire reason it exists.

### Memory Behavior
- Similar memory footprint to an equivalent `set` (same underlying hash table structure).
- Since it can't grow or shrink, no need for the same kind of resizing overhead.

### Time Complexity
Identical to `set` for all read/query operations:
| Operation | Complexity |
|---|---|
| `in` (membership) | O(1) average |
| Union/Intersection/Difference | O(len(s1) + len(s2)) |
| Iteration | O(n) |

(No add/remove operations exist, since it's immutable.)

### Space Complexity
O(n) — same order as `set`.

### Optimization Techniques
- Use `frozenset` instead of `tuple(sorted(some_set))` when you need a hashable, order-independent representation of a collection (a tuple requires sorting to ignore order; a frozenset doesn't).
- Use frozensets as keys in memoization caches when the "state" is naturally a set of items.

### Common Mistakes
- Trying to call mutating methods (`.add()`, `.remove()`) — these don't exist on `frozenset` and will raise `AttributeError`.
- Assuming `frozenset` preserves any particular order — it doesn't (same as regular sets).
- Using a regular `set` as a dictionary key by mistake — this raises `TypeError: unhashable type: 'set'`; you need `frozenset` instead.

### Best Practices
- Use `frozenset` whenever a "set-like" collection needs to be hashable (dict key, set element).
- Use it to represent constants/fixed collections that should never change once defined (e.g., a fixed set of valid options).

### Interview Questions
1. Why can't a regular `set` be used as a dictionary key, but a `frozenset` can?
2. When would you choose `frozenset` over `tuple` for representing an unordered, immutable group of items?
3. How would you use `frozenset` to represent undirected graph edges uniquely?

### Real-World Usage
- Caching/memoization keys for combinations of items
- Representing fixed configuration option sets
- Undirected graph edge representations
- Storing collections-of-collections (sets containing other sets, which requires the inner ones to be frozensets)

### Related Topics
`set`, hashability, `dict` keys, memoization

---

# 7. `dict` (Dictionary)

### Difficulty
[Beginner] → [Advanced] for internal hash table mechanics, collision resolution, and insertion-order guarantees.

### Definition
A `dict` is a **mutable collection of key–value pairs**, where each key must be unique and hashable, providing fast lookup, insertion, and deletion by key.

### Purpose
The go-to structure for mapping relationships — associating identifiers (keys) with data (values): configs, records, caches, counters, graphs (adjacency lists), JSON-like data, and more.

### Syntax
```python
d1 = {"name": "Claude", "age": 5}
d2 = dict(name="Claude", age=5)
d3 = dict([("a", 1), ("b", 2)])
d4 = {}                              # empty dict
d5 = {k: v for k, v in zip(["a","b"], [1,2])}   # dict comprehension
```

### Examples
```python
person = {"name": "Alice", "age": 30}
print(person["name"])          # Alice
person["city"] = "NYC"          # add new key
person.update({"age": 31})      # update existing key
print(person.get("zip", "N/A")) # safe access with default
for k, v in person.items():
    print(k, v)
del person["age"]
```

### DSA Questions and Concepts
- Two Sum (using dict for O(n) instead of O(n²))
- Frequency counting (anagram checks, character counts) — often via `collections.Counter` (a dict subclass)
- Group Anagrams
- LRU Cache implementation (dict + doubly linked list)
- Graph representation as adjacency list: `dict[node] = [neighbors]`
- Memoization in dynamic programming (`dict` as a cache keyed by subproblem state)
- Implementing a HashMap from scratch (classic interview question)

### Internal Working
- CPython implements `dict` as a **hash table**. Each key's hash determines its slot. On collision, **open addressing with perturbation** (a pseudo-random probing sequence) is used to find the next available slot, rather than chaining (linked lists), which most textbooks describe — CPython's specific implementation differs from the "classic" chained hash table.
- Since **Python 3.6** (officially guaranteed from **3.7+**), dictionaries **preserve insertion order** — this was a side effect of an internal optimization (a more compact dict layout separating hash/key/value storage from the order-tracking array) that became a language guarantee.
- Internally, CPython dicts use a *combined table* design: one compact array stores entries in insertion order, and a separate sparse hash table maps hash values to indices into that compact array — this reduces memory usage compared to the older implementation while preserving fast O(1) average lookups.
- Resizing happens once the dict's load factor crosses a threshold (~2/3 full), similar to sets and lists, doubling (or more) the underlying table size and "rehashing" all existing entries into new slots.

### Memory Behavior
- Dicts use more memory per entry than an equivalent list of tuples, due to the hash table overhead (extra reserved slots to avoid frequent rehashing, hash caching, etc.).
- Each resize operation requires O(n) time to rehash everything — but this happens rarely and is amortized across many insertions.
- `dict.fromkeys()` is a memory-efficient way to initialize many keys with the same default value.

### Time Complexity
| Operation | Average | Worst Case |
|---|---|---|
| `d[key]` (get) | O(1) | O(n) |
| `d[key] = value` (set) | O(1) | O(n) |
| `del d[key]` | O(1) | O(n) |
| `key in d` | O(1) | O(n) |
| Iteration (`.items()`, `.keys()`, `.values()`) | O(n) | — |

(Worst case occurs with pathological hash collisions — extremely rare with good hash functions and Python's built-in protections against hash-flooding attacks.)

### Space Complexity
O(n), with extra constant-factor overhead from the hash table structure (typically dicts use noticeably more memory per key-value pair than a list of tuples would).

### Optimization Techniques
- Use `dict.get(key, default)` instead of checking `if key in d` then accessing — avoids double lookup.
- Use `collections.defaultdict` to avoid manual "if key not present, initialize" boilerplate.
- Use `collections.Counter` for frequency-counting tasks — it's a specialized, optimized dict subclass.
- Use dict comprehensions for building dicts from iterables — faster and more readable than manual loops.
- For very large, static lookup tables, consider whether a simpler array/list with index-based lookup might be even faster if keys are sequential integers.

### Common Mistakes
- Using a mutable object (like a `list`) as a dictionary key → `TypeError: unhashable type`.
- Modifying a dict while iterating over it directly (`for k in d: del d[k]`) → `RuntimeError: dictionary changed size during iteration`. Iterate over `list(d.keys())` instead if you need to modify during iteration.
- Assuming pre-3.7 dicts preserve order — code relying on this in older Python versions would be a portability bug.
- Confusing `d.keys()` with a list — it returns a *view object*, which dynamically reflects changes to the dict.

### Best Practices
- Use `.get()` or `.setdefault()` for safe key access with defaults.
- Use `defaultdict` or `Counter` instead of manual initialization patterns.
- Use dict comprehensions for transforming data structures.
- Use meaningful, hashable keys (strings, tuples, frozensets — not lists).

### Interview Questions
1. How does Python's dict achieve O(1) average lookup time?
2. Explain why dicts maintain insertion order since Python 3.7.
3. What's the difference between `dict.get()` and `dict[key]`?
4. How would you implement a simple hash map from scratch (handling collisions)?
5. What happens internally when a dict resizes?
6. Implement an LRU cache using a dict.

### Real-World Usage
- JSON data manipulation (Python dicts map directly to/from JSON objects)
- Caching/memoization in dynamic programming and web backends
- Configuration management (settings stored as key-value pairs)
- Counting/frequency analysis (word counts, histogram data)
- Graph adjacency list representations
- Database-like in-memory lookups

### Related Topics
`set` (shares hash table mechanics), `collections.defaultdict`, `collections.Counter`, `collections.OrderedDict`, JSON serialization, hashable objects

---

# Summary Comparison Table

| Type | Mutable? | Ordered? | Duplicates Allowed? | Hashable? | Typical Use Case |
|---|---|---|---|---|---|
| `str` | No | Yes | Yes (chars can repeat) | Yes | Text data |
| `list` | Yes | Yes | Yes | No | General-purpose ordered collection |
| `tuple` | No | Yes | Yes | Yes (if elements are) | Fixed records, dict keys |
| `range` | No | Yes | N/A (numeric sequence) | Yes | Efficient loop counters |
| `set` | Yes | No (insertion order not guaranteed) | No | No | Uniqueness, fast membership |
| `frozenset` | No | No | No | Yes | Hashable set, dict keys, set-of-sets |
| `dict` | Yes (values; keys fixed set) | Yes (since 3.7+) | Keys: No, Values: Yes | Keys must be hashable | Key-value mapping |

