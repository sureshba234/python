<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" style="height:64px;margin-right:32px"/>

# Advanced Python Iterators

> A comprehensive guide to lazy evaluation, infinite sequences, streaming data, iterator pipelines, and the full `itertools` module with real-world examples, performance analysis, and best practices.

***

## Table of Contents

1. [Advanced Iterator Concepts](#1-advanced-iterator-concepts)
2. [Iterator Consumption](#2-iterator-consumption)
3. [Sentinel Iterators](#3-sentinel-iterators)
4. [itertools Module Overview](#4-itertools-module-overview)
5. [Infinite Iterators](#5-infinite-iterators)
6. [Finite Iterators](#6-finite-iterators)
7. [Combinatoric Iterators](#7-combinatoric-iterators)
8. [Building Iterator Pipelines](#8-building-iterator-pipelines)
9. [Functional Programming with Iterators](#9-functional-programming-with-iterators)
10. [Large Data Processing](#10-large-data-processing)
11. [Performance Analysis](#11-performance-analysis)
12. [Best Practices](#12-best-practices)
13. [Common Pitfalls](#13-common-pitfalls)
14. [Interview Questions](#14-interview-questions)
15. [Exercises](#15-exercises)

***

## 1. Advanced Iterator Concepts

### Lazy Evaluation

Lazy evaluation means computing values only when needed, not upfront. This saves memory and enables processing of infinite or massive datasets.

```python
# Lazy: generator expression
lazy = (x * 2 for x in range(1000000))

# Value computed only when accessed
print(next(lazy))  # 0
print(next(lazy))  # 2
```

**Benefits:**

- Memory efficiency (O(1) instead of O(n))
- Enables infinite sequences
- Faster startup (no initial computation)


### Infinite Sequences

Iterators can represent sequences that never end:

```python
from itertools import count

infinite = count(0, 2)  # 0, 2, 4, 6, ... infinite

for i in infinite:
    if i > 20:
        break
    print(i)  # 0, 2, 4, ..., 20
```


### Streaming Data

Process data as it arrives without loading everything:

```python
def stream_csv(filename):
    """Stream CSV lines one at a time"""
    with open(filename) as f:
        for line in f:
            yield line.strip()

# Process 1GB CSV without loading it
for line in stream_csv('large.csv'):
    process(line)
```


### Iterator Pipelines

Chain multiple transformations like a data pipeline:

```python
data = range(100)

pipeline = (
    data
    .__iter__()
    .__class__()  # iterator
)

# Practical pipeline
pipeline = (
    x ** 2 for x in range(1000)
    if x % 2 == 0
    if x > 100
)

for value in pipeline:
    process(value)
```


***

## 2. Iterator Consumption

### One-Time Use

Iterators are **single-use**: once consumed, they're exhausted.

```python
it = iter([1, 2, 3])

print(next(it))  # 1
print(next(it))  # 2
print(next(it))  # 3
print(next(it))  # Raises StopIteration
```


### Exhaustion

When an iterator has no more items, it's **exhausted**:

```python
it = iter([1, 2, 3])

list(it)  # [1, 2, 3]
list(it)  # [] - already exhausted!
```

Check exhaustion:

```python
it = iter([1, 2])

next(it)  # 1
next(it)  # 2

try:
    next(it)
except StopIteration:
    print("Exhausted!")
```


### Resetting

You **cannot reset** an iterator. Create a new one:

```python
data = [1, 2, 3]

it1 = iter(data)
next(it1)  # 1

# Can't reset it1
it2 = iter(data)  # New iterator
next(it2)  # 1
```

**Exception:** Some objects support re-iteration:

```python
range_obj = range(3)

for x in range_obj:
    print(x)  # 0, 1, 2

for x in range_obj:
    print(x)  # 0, 1, 2 - works!
```


***

## 3. Sentinel Iterators

### `iter(callable, sentinel)` Pattern

Read values until a sentinel value appears:

```python
from itertools import iter

def read_number():
    """Simulate reading numbers until 'END'"""
    values = ['10', '20', '30', 'END', '40']
    for v in values:
        yield v

# Stop at 'END'
numbers = iter(read_number, 'END')

for num in numbers:
    print(int(num))  # 10, 20, 30
```


### Real Example: Reading Binary Data

```python
def read_byte():
    """Read bytes until EOF (b'\\x00')"""
    data = b'\x01\x02\x03\x00\x04'
    for b in data:
        yield bytes([b])

bytes_iter = iter(read_byte, b'\x00')

for b in bytes_iter:
    print(int(b))  # 1, 2, 3
```


### File Reading with Sentinel

```python
def read_line():
    """Read lines until empty line"""
    lines = ['line1\n', 'line2\n', '\n', 'line3\n']
    for line in lines:
        yield line

lines = iter(read_line, '\n')

for line in lines:
    print(line.strip())  # line1, line2
```


***

## 4. itertools Module Overview

### Architecture

`itertools` provides **fast, memory-efficient iterator tools** implemented in C. Key principles:

- **Lazy evaluation**: All functions return iterators
- **Single-pass**: Most consume iterators once
- **Composable**: Chain functions together
- **C-optimized**: Faster than pure Python


### Purpose

```python
import itertools

# Before itertools (Python loop)
result = []
for i in range(10):
    if i % 2 == 0:
        result.append(i * 2)

# After itertools (lazy pipeline)
result = itertools.map(
    lambda x: x * 2,
    itertools.filterfalse(lambda x: x % 2, range(10))
)
```


### Category Breakdown

| Category | Functions |
| :-- | :-- |
| Infinite | `count()`, `cycle()`, `repeat()` |
| Input-based | `accumulate()`, `chain()`, `compress()`, `dropwhile()`, `takewhile()`, `filterfalse()`, `groupby()`, `islice()`, `starmap()`, `tee()`, `zip_longest()` |
| Combinatoric | `product()`, `permutations()`, `combinations()`, `combinations_with_replacement()` |


***

## 5. Infinite Iterators

### `count(start=0, step=1)`

Infinite counter:

```python
from itertools import count

for i in count(10, 2):
    if i > 20:
        break
    print(i)  # 10, 12, 14, 16, 18, 20
```

**Real use: Enumerating without `enumerate()`**

```python
for i, line in zip(count(), open('large.log')):
    print(f"{i}: {line.strip()}")
```


### `cycle(iterable)`

Repeat indefinitely:

```python
from itertools import cycle

colors = cycle(['red', 'green', 'blue'])

for i in range(8):
    print(colors.next())  # red, green, blue, red, green, blue, red, green
```

**Real use: Round-robin scheduling**

```python
servers = cycle(['server1', 'server2', 'server3'])

for request in requests:
    server = next(servers)
    send_to(request, server)
```


### `repeat(object, times=None)`

Repeat object:

```python
from itertools import repeat

# Infinite repeat
for i in repeat('x', 5):
    print(i)  # x, x, x, x, x

# With zip
for x, y in zip(range(3), repeat(0)):
    print(f"{x} -> {y}")  # 0->0, 1->0, 2->0
```

**Real use: Padding**

```python
from itertools import zip_longest, repeat

data = [1, 2, 3]
padded = zip_longest(data, repeat(0), fillvalue=0)
```


***

## 6. Finite Iterators

### `accumulate(iterable, func=+, initial=None)`

Running accumulation:

```python
from itertools import accumulate

nums = [1, 2, 3, 4, 5]

# Sum
list(accumulate(nums))  # [1, 3, 6, 10, 15]

# Product
list(accumulate(nums, func=lambda a, b: a * b))  # [1, 2, 6, 24, 120]

# Max
list(accumulate(nums, func=max))  # [1, 2, 3, 4, 5]

# With initial
list(accumulate(nums, initial=10))  # [10, 11, 13, 16, 20, 25]
```


### `chain(*iterables)`

Chain multiple iterables:

```python
from itertools import chain

list(chain([1, 2], [3, 4], [5, 6]))  # [1, 2, 3, 4, 5, 6]

# Chain iterators
it1 = iter([1, 2])
it2 = iter([3, 4])
list(chain(it1, it2))  # [1, 2, 3, 4]
```

**Real use: Merging log files**

```python
from itertools import chain

all_logs = chain(
    open('app1.log'),
    open('app2.log'),
    open('app3.log')
)

for line in all_logs:
    process(line)
```


### `compress(data, selectors)`

Filter by selectors:

```python
from itertools import compress

data = ['A', 'B', 'C', 'D']
selectors = [1, 0, 1, 0]

list(compress(data, selectors))  # ['A', 'C']
```

**Real use: Mask filtering**

```python
records = [{'id': 1, 'active': True}, 
           {'id': 2, 'active': False},
           {'id': 3, 'active': True}]

active = compress(records, [r['active'] for r in records])
```


### `dropwhile(predicate, iterable)`

Drop until predicate is false:

```python
from itertools import dropwhile

list(dropwhile(lambda x: x < 5, [1, 2, 3, 4, 5, 6, 7]))  # [5, 6, 7]
list(dropwhile(lambda x: x < 5, [6, 7, 8]))  # [6, 7, 8]
list(dropwhile(lambda x: x < 5, [1, 2, 3]))  # []
```


### `takewhile(predicate, iterable)`

Take until predicate is false:

```python
from itertools import takewhile

list(takewhile(lambda x: x < 5, [1, 2, 3, 4, 5, 6, 7]))  # [1, 2, 3, 4]
list(takewhile(lambda x: x < 5, [6, 7, 8]))  # []
list(takewhile(lambda x: x < 5, [1, 2, 3]))  # [1, 2, 3]
```

**Real use: Parsing until delimiter**

```python
lines = ['line1', 'line2', 'END', 'line3']
before_end = takewhile(lambda x: x != 'END', lines)
```


### `filterfalse(predicate, iterable)`

inverse of `filter()`:

```python
from itertools import filterfalse

list(filterfalse(lambda x: x % 2, [1, 2, 3, 4, 5, 6]))  # [2, 4, 6]
list(filterfalse(lambda x: x > 5, [1, 3, 5, 7, 9]))  # [1, 3, 5]
```


### `groupby(iterable, key=None)`

Group by key:

```python
from itertools import groupby

data = [
    {'name': 'Alice', 'age': 25},
    {'name': 'Bob', 'age': 30},
    {'name': 'Charlie', 'age': 25},
    {'name': 'Dave', 'age': 30},
]

for age, group in groupby(data, key=lambda x: x['age']):
    print(f"Age {age}: {[p['name'] for p in group]}")
# Age 25: ['Alice', 'Charlie']
# Age 30: ['Bob', 'Dave']
```

**Important:** Data must be sorted by key!

```python
# Sort first
sorted_data = sorted(data, key=lambda x: x['age'])
for age, group in groupby(sorted_data, key=lambda x: x['age']):
    ...
```


### `islice(iterable, stop)` / `islice(iterable, start, stop[, step])`

Slice iterators:

```python
from itertools import islice

list(islice(range(10), 5))  # [0, 1, 2, 3, 4]
list(islice(range(10), 2, 7))  # [2, 3, 4, 5, 6]
list(islice(range(10), 1, 8, 2))  # [1, 3, 5, 7]
```

**Real use: Head/tail of large files**

```python
# First 10 lines
for line in islice(open('huge.log'), 10):
    print(line)

# Lines 100-200
for line in islice(open('huge.log'), 100, 200):
    print(line)
```


### `starmap(function, iterables)`

Like `map()` but unpacks arguments:

```python
from itertools import starmap

args = [(2, 3), (4, 5), (6, 7)]
list(starmap(pow, args))  # [8, 1024, 279936]

# With multiple iterables
list(starmap(lambda x, y: x + y, [(1, 2), (3, 4), (5, 6)]))  # [3, 7, 11]
```

**Real use: Calling functions with argument tuples**

```python
functions = [(add, (1, 2)), (mul, (3, 4)), (pow, (2, 3))]
results = starmap(lambda f, args: f(*args), functions)
```


### `tee(iterable, n=2)`

Tee multiple iterators:

```python
from itertools import tee

it = iter([1, 2, 3])
it1, it2 = tee(it, 2)

list(it1)  # [1, 2, 3]
list(it2)  # [1, 2, 3]
```

**Warning:** `tee()` stores items in a buffer!

```python
# Memory-intensive for large iterables
it1, it2, it3 = tee(large_iterator, 3)
```


### `zip_longest(*iterables, fillvalue=None)`

Zip with padding:

```python
from itertools import zip_longest

list(zip_longest([1, 2, 3], [4, 5], fillvalue=0))  # [(1, 4), (2, 5), (3, 0)]
list(zip_longest('abc', [1, 2], fillvalue='-'))  # [('a', 1), ('b', 2), ('c', '-')]
```

**Real use: Merging uneven datasets**

```python
dates = ['2024-01', '2024-02', '2024-03', '2024-04']
values = [100, 150, 200]  # Missing April

for date, value in zip_longest(dates, values, fillvalue=0):
    print(f"{date}: {value}")
```


***

## 7. Combinatoric Iterators

### `product(iterables, repeat=1)`

Cartesian product:

```python
from itertools import product

list(product([1, 2], [3, 4]))  # [(1, 3), (1, 4), (2, 3), (2, 4)]
list(product('AB', repeat=2))  # [('A', 'A'), ('A', 'B'), ('B', 'A'), ('B', 'B')]
```

**Real use: Generating test cases**

```python
inputs = [1, 2, 3]
ops = ['+', '-', '*']

for a, b, op in product(inputs, inputs, ops):
    test(f"{a} {op} {b}")
```


### `permutations(iterable, r=None)`

All permutations:

```python
from itertools import permutations

list(permutations([1, 2, 3]))  # [(1, 2, 3), (1, 3, 2), (2, 1, 3), (2, 3, 1), (3, 1, 2), (3, 2, 1)]
list(permutations([1, 2, 3], 2))  # [(1, 2), (1, 3), (2, 1), (2, 3), (3, 1), (3, 2)]
```

**Real use: Generating arrangements**

```python
words = permutations(['cat', 'dog', 'bird'], 2)
for w in words:
    print(f"{w[0]} {w[1]}")
```


### `combinations(iterable, r)`

Combinations (no repeats):

```python
from itertools import combinations

list(combinations([1, 2, 3, 4], 2))  # [(1, 2), (1, 3), (1, 4), (2, 3), (2, 4), (3, 4)]
list(combinations('ABC', 2))  # [('A', 'B'), ('A', 'C'), ('B', 'C')]
```

**Real use: Pairing teams**

```python
players = ['Alice', 'Bob', 'Charlie', 'Dave']
teams = combinations(players, 2)

for team in teams:
    print(f"{team[0]} & {team[1]}")
```


### `combinations_with_replacement(iterable, r)`

Combinations with repeats:

```python
from itertools import combinations_with_replacement

list(combinations_with_replacement([1, 2, 3], 2))  # [(1, 1), (1, 2), (1, 3), (2, 2), (2, 3), (3, 3)]
list(combinations_with_replacement('ABC', 2))  # [('A', 'A'), ('A', 'B'), ('A', 'C'), ('B', 'B'), ('B', 'C'), ('C', 'C')]
```

**Real use: Generating dice rolls**

```python
dice = combinations_with_replacement([1, 2, 3, 4, 5, 6], 2)
for roll in dice:
    print(roll)
```


***

## 8. Building Iterator Pipelines

### Real-World Example 1: Log Analyzer

```python
from itertools import islice, filterfalse, starmap

def parse_log_line(line):
    """Parse log line into dict"""
    parts = line.strip().split('|')
    return {
        'timestamp': parts[0],
        'level': parts[1],
        'message': parts[2]
    }

def is_error(log):
    return log['level'] == 'ERROR'

# Pipeline
logs = (
    open('large.log')
    .__iter__()
    .__class__()
)

pipeline = (
    parse_log_line(line)
    for line in logs
    if is_error(parse_log_line(line))
)

# Process
for log in pipeline:
    print(log['timestamp'], log['message'])
```


### Real-World Example 2: CSV Stream Processor

```python
from itertools import islice, takewhile, compress

def parse_csv(line):
    return line.strip().split(',')

def is_valid(record):
    return len(record) == 3 and record[2].isdigit()

# Stream large CSV
with open('huge.csv') as f:
    # Skip header
    lines = islice(f, 1, None)
    
    # Parse
    records = (parse_csv(line) for line in lines)
    
    # Filter valid
    valid = compress(records, [is_valid(r) for r in records])
    
    # Process
    for record in valid:
        process(record)
```


### Real-World Example 3: API Rate Limiter

```python
from itertools import count, takewhile

def make_request(api, token):
    return api.call(token)

tokens = ['token1', 'token2', 'token3']
requests = count(0)

# Rate limit: max 100 requests
limited = takewhile(
    lambda r: r < 100,
    requests
)

for req_id in limited:
    token = tokens[req_id % len(tokens)]
    make_request(api, token)
```


***

## 9. Functional Programming with Iterators

### Combining `map`, `filter`, `reduce`, `itertools`

```python
from itertools import filterfalse, map, accumulate
from functools import reduce

# Traditional
result = []
for x in range(100):
    if x % 2 == 0:
        result.append(x * 2)
total = sum(result)

# Functional with iterators
evens = filterfalse(lambda x: x % 2, range(100))
doubled = map(lambda x: x * 2, evens)
total = reduce(lambda a, b: a + b, doubled)
```


### Complete Pipeline Example

```python
from itertools import filterfalse, map, accumulate
from functools import reduce

data = range(1, 11)

# Pipeline: filter even → square → sum
result = reduce(
    lambda a, b: a + b,
    map(
        lambda x: x ** 2,
        filterfalse(lambda x: x % 2, data)
    )
)

print(result)  # 200 (4 + 16 + 36 + 64 + 100)
```


### Lazy Reduce

```python
from itertools import accumulate
from functools import reduce

# Running sum (lazy)
running = accumulate(range(10))

for r in running:
    print(r)  # 0, 1, 3, 6, 10, ...
```


***

## 10. Large Data Processing

### Log Files

```python
from itertools import islice, filterfalse

def count_errors(log_file, max_lines=1000000):
    """Count ERROR lines in large log file"""
    lines = islice(open(log_file), max_lines)
    errors = filterfalse(
        lambda line: 'ERROR' not in line,
        lines
    )
    return sum(1 for _ in errors)

error_count = count_errors('huge.log')
print(f"Errors: {error_count}")
```


### CSV Streams

```python
from itertools import islice, compress

def process_csv(csv_file):
    """Process large CSV without loading"""
    with open(csv_file) as f:
        # Skip header
        lines = islice(f, 1, None)
        
        # Parse lines
        records = (line.strip().split(',') for line in lines)
        
        # Filter valid (3 columns, 3rd is digit)
        valid = compress(
            records,
            [len(r) == 3 and r[2].isdigit() for r in records]
        )
        
        # Process
        for record in valid:
            process(record)

process_csv('gigabyte.csv')
```


### API Streams

```python
from itertools import count, takewhile

def fetch_api_pages(api, max_pages=100):
    """Fetch paginated API results"""
    page_nums = takewhile(
        lambda p: p < max_pages,
        count(0)
    )
    
    results = []
    for page in page_nums:
        data = api.fetch(page=page)
        if not data:
            break
        results.extend(data)
    
    return results

data = fetch_api_pages(my_api)
```


***

## 11. Performance Analysis

### Lists vs Generators vs itertools

```python
import timeit
from itertools import islice, filterfalse, map

# List (memory: O(n))
def list_approach():
    return [x * 2 for x in range(1000000) if x % 2 == 0]

# Generator (memory: O(1))
def generator_approach():
    return (x * 2 for x in range(1000000) if x % 2 == 0)

# itertools (memory: O(1), faster)
def itertools_approach():
    return map(
        lambda x: x * 2,
        filterfalse(lambda x: x % 2, range(1000000))
    )

# Timing
print(timeit.timeit(list_approach, number=10))      # ~2.5s
print(timeit.timeit(generator_approach, number=10)) # ~1.8s
print(timeit.timeit(itertools_approach, number=10)) # ~1.5s
```


### Memory Comparison

```python
import sys

# List: stores all
lst = list(range(1000000))
print(sys.getsizeof(lst))  # ~8MB + element data

# Generator: single item
gen = (x for x in range(1000000))
print(sys.getsizeof(gen))  # ~48 bytes

# itertools: single item
it = islice(range(1000000), 100)
print(sys.getsizeof(it))  # ~48 bytes
```


### Benchmark Table

| Approach | Memory | Speed | Best For |
| :-- | :-- | :-- | :-- |
| List | O(n) | Slow | Small data, repeated access |
| Generator | O(1) | Medium | One-pass, custom logic |
| itertools | O(1) | Fast | Standard patterns, large data |


***

## 12. Best Practices

1. **Use lazy iterators for large data**

```python
# Good
for line in open('huge.log'):
    process(line)

# Bad
lines = open('huge.log').readlines()
for line in lines:
    process(line)
```

2. **Chain itertools for performance**

```python
# Good
result = map(f, filter(g, data))

# Bad
result = [f(x) for x in data if g(x)]
```

3. **Don't reuse exhausted iterators**

```python
# Good
it = iter(data)
for x in it:
    process(x)

it = iter(data)  # New iterator
for x in it:
    process(x)
```

4. **Sort before `groupby()`**

```python
sorted_data = sorted(data, key=key)
for k, group in groupby(sorted_data, key=key):
    ...
```

5. **Use `islice()` for head/tail**

```python
for line in islice(open('log'), 10):  # First 10
    print(line)

for line in islice(open('log'), 1000, 1010):  # Lines 1000-1010
    print(line)
```

6. **Avoid `tee()` for large data**

```python
# Bad: stores all items
it1, it2 = tee(large_iterator, 2)

# Good: recreate iterator
it1 = iter(data)
it2 = iter(data)
```


***

## 13. Common Pitfalls

### 1. Reusing Exhausted Iterators

```python
it = iter([1, 2, 3])
list(it)  # [1, 2, 3]
list(it)  # [] - exhausted!
```

**Fix:** Create new iterator

```python
it = iter([1, 2, 3])
list(it)
it = iter([1, 2, 3])  # New
list(it)  # [1, 2, 3]
```


### 2. `groupby()` Without Sorting

```python
data = [{'age': 25}, {'age': 30}, {'age': 25}]

# Wrong: unsorted
for age, group in groupby(data, key=lambda x: x['age']):
    print(age, list(group))
# 25: [{'age': 25}]
# 30: [{'age': 30}]
# 25: [{'age': 25}]  # Split!

# Right: sorted
sorted_data = sorted(data, key=lambda x: x['age'])
for age, group in groupby(sorted_data, key=lambda x: x['age']):
    print(age, list(group))
# 25: [{'age': 25}, {'age': 25}]
# 30: [{'age': 30}]
```


### 3. `tee()` Memory Explosion

```python
# Bad: buffers everything
it1, it2 = tee(range(1000000), 2)

# Good: don't tee large data
it1 = iter(range(1000000))
it2 = iter(range(1000000))
```


### 4. Infinite Iterators Without Break

```python
from itertools import count

# Bad: infinite loop
for i in count():
    process(i)  # Never stops!

# Good: bounded
for i in count():
    if i > 100:
        break
    process(i)
```


### 5. Modifying Data During Iteration

```python
# Bad
data = [1, 2, 3, 4, 5]
for x in data:
    if x % 2 == 0:
        data.remove(x)  # Modifies during iteration!

# Good: filter
data = [x for x in data if x % 2 != 0]

# Better: filterfalse
from itertools import filterfalse
data = list(filterfalse(lambda x: x % 2 == 0, data))
```


***

## 14. Interview Questions

### Q1: What's the difference between a list and an iterator?

**Answer:**

- List: stores all items, O(n) memory, supports indexing, reusable
- Iterator: lazy, O(1) memory, no indexing, single-use


### Q2: When would you use `itertools` over generator expressions?

**Answer:**

- `itertools`: C-optimized, faster for standard patterns
- Generators: More flexible for custom logic
- Use `itertools` for performance on large data


### Q3: Explain lazy evaluation with an example

```python
# Lazy: computed on demand
lazy = (x * 2 for x in range(1000000))
print(next(lazy))  # 0 - computed now

# Eager: computed immediately
eager = [x * 2 for x in range(1000000)]  # All computed
```


### Q4: How does `groupby()` work and what's the caveat?

**Answer:**

- Groups consecutive items with same key
- **Caveat:** Data must be sorted by key first!

```python
sorted_data = sorted(data, key=key)
for k, group in groupby(sorted_data, key=key):
    ...
```


### Q5: Write a function to read lines until "END"

```python
from itertools import iter

def read_until_end(lines):
    return iter(lines, 'END')

lines = ['line1', 'line2', 'END', 'line3']
for line in read_until_end(lines):
    print(line)  # line1, line2
```


### Q6: Implement infinite Fibonacci using `itertools`

```python
from itertools import count

def fibonacci():
    a, b = 0, 1
    while True:
        yield a
        a, b = b, a + b

for i, fib in zip(count(), fibonacci()):
    if i > 10:
        break
    print(f"{i}: {fib}")
```


### Q7: What's the memory usage of `tee()`?

**Answer:**

- O(n) where n is items consumed before all iterators advance
- Avoid for large iterables


### Q8: Compare `filter()` vs `filterfalse()`

```python
# filter(): keep when predicate is True
filter(lambda x: x > 5, [1, 5, 6, 10])  # [6, 10]

# filterfalse(): keep when predicate is False
filterfalse(lambda x: x > 5, [1, 5, 6, 10])  # [1, 5]
```


***

## 15. Exercises

### Exercise 1: Stream Processor

Write a function that reads a large log file and returns only ERROR lines:

```python
from itertools import islice, filterfalse

def stream_errors(log_file, max_lines=100000):
    lines = islice(open(log_file), max_lines)
    errors = filterfalse(
        lambda line: 'ERROR' not in line,
        lines
    )
    return errors

# Usage
for error in stream_errors('huge.log'):
    print(error)
```


### Exercise 2: Round-Robin Scheduler

Implement round-robin request distribution:

```python
from itertools import cycle

def round_robin(servers, requests):
    server_pool = cycle(servers)
    for request in requests:
        server = next(server_pool)
        send(request, server)

# Usage
round_robin(['s1', 's2', 's3'], requests)
```


### Exercise 3: Combinatory Test Generator

Generate all test cases for 3 inputs and 2 ops:

```python
from itertools import product

inputs = [1, 2, 3]
ops = ['+', '*']

for a, b, op in product(inputs, inputs, ops):
    test(f"{a} {op} {b}")
```


### Exercise 4: CSV Validator

Filter valid CSV records (3 columns, 3rd is digit):

```python
from itertools import islice, compress

def valid_csv(csv_file):
    with open(csv_file) as f:
        lines = islice(f, 1, None)  # Skip header
        records = (line.strip().split(',') for line in lines)
        valid = compress(
            records,
            [len(r) == 3 and r[2].isdigit() for r in records]
        )
        return valid
```


### Exercise 5: Running Statistics

Compute running mean using `accumulate()`:

```python
from itertools import accumulate

def running_mean(data):
    sums = accumulate(data)
    counts = accumulate([1] * len(data))
    return (s / c for s, c in zip(sums, counts))

# Usage
for mean in running_mean([1, 2, 3, 4, 5]):
    print(mean)  # 1.0, 1.5, 2.0, 2.5, 3.0
```


### Exercise 6: Infinite Counter with Limit

```python
from itertools import count, takewhile

def bounded_count(start, step, limit):
    return takewhile(
        lambda x: x < limit,
        count(start, step)
    )

# Usage
for x in bounded_count(0, 2, 20):
    print(x)  # 0, 2, 4, ..., 18
```


### Exercise 7: Pairing Teams

Generate all 2-person teams from 5 players:

```python
from itertools import combinations

players = ['Alice', 'Bob', 'Charlie', 'Dave', 'Eve']
teams = combinations(players, 2)

for team in teams:
    print(f"{team[0]} & {team[1]}")
```


### Exercise 8: Log Time Range

Extract logs between two timestamps:

```python
from itertools import takewhile, dropwhile

def logs_between(log_file, start, end):
    lines = open(log_file)
    
    # Skip until start
    lines = dropwhile(
        lambda line: not line.startswith(start),
        lines
    )
    
    # Take until end
    lines = takewhile(
        lambda line: line.startswith(end) or not line.startswith(end),
        lines
    )
    
    return lines
```


### Exercise 9: Memory-Efficient Zip

Zip two large files without loading:

```python
from itertools import zip_longest

def zip_files(file1, file2, fillvalue=''):
    return zip_longest(
        open(file1),
        open(file2),
        fillvalue=fillvalue
    )

# Usage
for line1, line2 in zip_files('large1.txt', 'large2.txt'):
    process(line1, line2)
```


### Exercise 10: Performance Benchmark

Compare list vs generator vs itertools:

```python
import timeit
from itertools import filterfalse, map

def list_version():
    return [x * 2 for x in range(100000) if x % 2 == 0]

def generator_version():
    return (x * 2 for x in range(100000) if x % 2 == 0)

def itertools_version():
    return map(
        lambda x: x * 2,
        filterfalse(lambda x: x % 2, range(100000))
    )

print("List:", timeit.timeit(list_version, number=10))
print("Generator:", timeit.timeit(generator_version, number=10))
print("itertools:", timeit.timeit(itertools_version, number=10))
```


***

## Summary

Advanced iterators and `itertools` provide:

- **Memory efficiency**: O(1) instead of O(n)
- **Performance**: C-optimized operations
- **Flexibility**: Infinite sequences, streaming, pipelines
- **Composability**: Chain functions for clean code

Master these tools for handling large data, building efficient pipelines, and writing professional Python code.

