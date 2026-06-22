<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" style="height:64px;margin-right:32px"/>

# Python Iterators Fundamentals

## Table of Contents

1. [What is Iteration](#1-what-is-iteration)
2. [Iterable Objects](#2-iterable-objects)
3. [Iterator Objects](#3-iterator-objects)
4. [Iterator Protocol](#4-iterator-protocol)
5. [The `iter()` Function](#5-the-iter-function)
6. [The `next()` Function](##6-the-next-function)
7. [Manual Iteration](#7-manual-iteration)
8. [Relationship Between Iterables and Iterators](#8-relationship-between-iterables-and-iterators)
9. [Creating Custom Iterators](#9-creating-custom-iterators)
10. [Iterator Design Patterns](#10-iterator-design-patterns)
11. [Performance Considerations](#11-performance-considerations)
12. [Best Practices](#12-best-practices)
13. [Common Errors](#13-common-errors)
14. [Interview Questions](#14-interview-questions)
15. [Practice Exercises](#15-practice-exercises)

***

## 1. What is Iteration

### Definition

**Iteration** is the process of accessing elements of a collection sequentially, one at a time, without needing to know the internal structure of the collection. In Python, iteration powers `for` loops, list comprehensions, and functions like `sum()`, `max()`, and `list()`.

### Iterable vs Iterator

| Concept | Definition | Examples |
| :-- | :-- | :-- |
| **Iterable** | An object capable of returning an iterator when `iter()` is called | `list`, `tuple`, `dict`, `string`, `set`, `range`, `file` |
| **Iterator** | An object representing a stream of data with `__iter__()` and `__next__()` methods | Result of `iter(list)`, custom iterator classes |

```python
# Iterable
my_list = [1, 2, 3]
iterable = my_list  # list is iterable

# Iterator
my_iterator = iter(my_list)  # Creates an iterator
```


### Why Iteration Exists

1. **Abstraction**: Decouples access logic from data structure
2. **Uniformity**: Works with any iterable using the same syntax
3. **Memory Efficiency**: Enables lazy evaluation (loading data on-demand)
4. **Scalability**: Handles infinite sequences and massive datasets

### Memory Advantages

```python
# Without iteration (loads all data)
data = [x for x in range(1000000)]  # 1 million integers in memory

# With iteration (lazy evaluation)
for x in range(1000000):  # Only one integer at a time
    process(x)
```

**Key benefit**: Iterators maintain only the current state, not the entire collection.

***

## 2. Iterable Objects

Python provides several built-in iterable types:

### list

```python
my_list = [1, 2, 3, 4]
for item in my_list:
    print(item)  # 1, 2, 3, 4
```

- Ordered, mutable collection
- Supports indexing and slicing


### tuple

```python
my_tuple = (10, 20, 30)
for num in my_tuple:
    print(num)  # 10, 20, 30
```

- Ordered, immutable collection
- Faster than lists, memory-efficient


### set

```python
my_set = {1, 2, 3, 3, 4}  # {1, 2, 3, 4}
for item in my_set:
    print(item)  # Order not guaranteed
```

- Unordered, unique elements
- No indexing


### dict

```python
my_dict = {'a': 1, 'b': 2, 'c': 3}
for key in my_dict:
    print(key)  # 'a', 'b', 'c' (keys by default)

for value in my_dict.values():
    print(value)  # 1, 2, 3

for key, value in my_dict.items():
    print(key, value)  # ('a', 1), ('b', 2), ('c', 3)
```

- Key-value pairs
- Iterates over keys by default


### string

```python
my_string = "Python"
for char in my_string:
    print(char)  # 'P', 'y', 't', 'h', 'o', 'n'
```

- Sequence of characters
- Indexable and sliceable


### range

```python
for i in range(5):  # 0, 1, 2, 3, 4
    print(i)

for i in range(2, 10, 2):  # 2, 4, 6, 8
    print(i)
```

- Generates numbers lazily (no memory for all values)
- `range(start, stop, step)`


### file objects

```python
with open('data.txt', 'r') as file:
    for line in file:  # Iterates line by line
        print(line.strip())
```

- Iterates over lines
- Memory-efficient for large files

***

## 3. Iterator Objects

### What is an Iterator?

An **iterator** is an object that implements the iterator protocol:

- `__iter__()` - Returns the iterator itself
- `__next__()` - Returns the next element or raises `StopIteration`

```python
my_list = [1, 2, 3]
iterator = iter(my_list)

print(type(iterator))  # <class 'list_iterator'>
```


### How Iterators Work

1. Iterator maintains **internal state** (current position)
2. Each `__next__()` call returns the next element
3. State advances after each call
4. When exhausted, raises `StopIteration`
```python
iterator = iter([10, 20, 30])

print(next(iterator))  # 10 (state: position 1)
print(next(iterator))  # 20 (state: position 2)
print(next(iterator))  # 30 (state: position 3)
print(next(iterator))  # StopIteration!
```


### State Management

```python
class Counter:
    def __init__(self, start, end):
        self.current = start
        self.end = end
    
    def __iter__(self):
        return self
    
    def __next__(self):
        if self.current > self.end:
            raise StopIteration
        value = self.current
        self.current += 1  # State mutation
        return value

counter = Counter(1, 3)
for num in counter:
    print(num)  # 1, 2, 3
```


### Exhaustion

Once an iterator is exhausted, it cannot be reused:

```python
iterator = iter([1, 2, 3])

list(iterator)  # [1, 2, 3]
list(iterator)  # [] (empty! iterator is exhausted)
```

**Solution**: Create a new iterator from the original iterable.

***

## 4. Iterator Protocol

The iterator protocol is the foundation of Python's iteration system.

### `__iter__()`

- Called by `iter(obj)`
- Returns the iterator object
- For iterators, returns itself

```python
class MyIterator:
    def __iter__(self):
        return self  # Iterator returns itself
```


### `__next__()`

- Called by `next(iterator)`
- Returns the next item
- Raises `StopIteration` when exhausted

```python
class MyIterator:
    def __init__(self, data):
        self.data = data
        self.index = 0
    
    def __next__(self):
        if self.index >= len(self.data):
            raise StopIteration
        value = self.data[self.index]
        self.index += 1
        return value
```


### `StopIteration`

Built-in exception signaling end of iteration:

```python
class SimpleIterator:
    def __init__(self, limit):
        self.limit = limit
        self.count = 0
    
    def __iter__(self):
        return self
    
    def __next__(self):
        if self.count >= self.limit:
            raise StopIteration  # Signal exhaustion
        self.count += 1
        return self.count

for num in SimpleIterator(3):
    print(num)  # 1, 2, 3
```


### Execution Flow

```
for loop                     Manual iteration
──────────────────────────────────────────────────
for item in iterable:        iterator = iter(iterable)
    process(item)            while True:
                                 try:
                                     item = next(iterator)
                                     process(item)
                                 except StopIteration:
                                     break
```

**The `for` loop automatically:**

1. Calls `iter(iterable)` to get iterator
2. Calls `next(iterator)` repeatedly
3. Handles `StopIteration` silently

***

## 5. The `iter()` Function

### Purpose

`iter(obj)` converts an **iterable** into an **iterator**.

```python
my_list = [1, 2, 3]
iterator = iter(my_list)  # list → list_iterator
```


### Behavior

```python
# From iterable
iterable = [1, 2, 3]
iterator = iter(iterable)
print(next(iterator))  # 1

# From iterator (returns itself)
iterator2 = iter(iterator)
print(iterator2 is iterator)  # True
```


### Internal Mechanism

```python
# What iter() does internally:
def iter(obj):
    if hasattr(obj, '__iter__'):
        return obj.__iter__()
    elif hasattr(obj, '__getitem__'):
        # Legacy: objects with __getitem__ only
        return SequenceIterator(obj)
    else:
        raise TypeError(f"'{type(obj).__name__}' object is not iterable")
```


### Examples

```python
# String
iter_str = iter("hello")
print(next(iter_str))  # 'h'

# Dictionary (iterates keys)
iter_dict = iter({'a': 1, 'b': 2})
print(next(iter_dict))  # 'a'

# Set
iter_set = iter({1, 2, 3})
print(next(iter_set))  # 1 (order undefined)

# range
iter_range = iter(range(5))
print(next(iter_range))  # 0
```


***

## 6. The `next()` Function

### Syntax

```python
next(iterator)          # Raises StopIteration if exhausted
next(iterator, default) # Returns default if exhausted
```


### Default Values

```python
iterator = iter([1, 2])

print(next(iterator))       # 1
print(next(iterator))       # 2
print(next(iterator, -1))   # -1 (default, no exception)
```


### Error Handling

```python
iterator = iter([1])

try:
    result = next(iterator)
    print(result)  # 1
    
    result = next(iterator)
    print(result)
except StopIteration:
    print("Iterator exhausted!")  # This runs
```


### Examples

```python
# Safe iteration with default
words = ['hello', 'world']
word_iter = iter(words)

first = next(word_iter, '')  # 'hello'
second = next(word_iter, '')  # 'world'
third = next(word_iter, '')   # '' (default)

# Manual loop replacement
iterator = iter(range(5))
while True:
    try:
        item = next(iterator)
        print(item)
    except StopIteration:
        break
```


***

## 7. Manual Iteration

Manual iteration replicates what `for` loops do internally.

### Pattern

```python
while True:
    try:
        item = next(iterator)
        # Process item
        print(item)
    except StopIteration:
        # Iterator exhausted - exit loop
        break
```


### Step-by-Step Explanation

```python
# 1. Create iterator from iterable
my_list = [10, 20, 30]
iterator = iter(my_list)

# 2. Loop until StopIteration
while True:
    try:
        # 3. Get next item
        item = next(iterator)
        
        # 4. Process item
        print(f"Processing: {item}")
        
    except StopIteration:
        # 5. Caught exhaustion - exit
        print("Done iterating!")
        break
```

**Output:**

```
Processing: 10
Processing: 20
Processing: 30
Done iterating!
```


### Why Use Manual Iteration?

1. **Custom control flow** (skip items, conditionally break)
2. **Multiple iterators** in one loop
3. **Look-ahead/look-back** patterns
4. **Integration** with non-Python iteration systems
```python
# Example: Skip even numbers
iterator = iter(range(10))
while True:
    try:
        num = next(iterator)
        if num % 2 == 0:
            continue  # Skip even
        print(num)  # 1, 3, 5, 7, 9
    except StopIteration:
        break
```


***

## 8. Relationship Between Iterables and Iterators

### Core Concept

```
Iterable (list, tuple, dict)
    │
    ├─ iter(obj) ──→ Iterator (list_iterator)
    │                     │
    │                     ├─ __iter__() → self
    │                     │
    │                     └─ __next__() → item or StopIteration
    │
    └─ for loop automates this!
```


### Diagram

```
┌─────────────────────────────────────────────────┐
│           ITERABLE OBJECT (e.g., list)          │
│  [1, 2, 3, 4, 5]                                │
└───────────────────┬─────────────────────────────┘
                    │
                    │ iter(obj)
                    ▼
┌─────────────────────────────────────────────────┐
│           ITERATOR OBJECT                       │
│  list_iterator with internal state (index=0)    │
└───────────────────┬─────────────────────────────┘
                    │
                    │ next(iterator) × 5
                    ▼
┌─────────────────────────────────────────────────┐
│  Returns: 1 → 2 → 3 → 4 → 5 → StopIteration     │
└─────────────────────────────────────────────────┘
```


### Key Relationships

```python
# 1. iter(obj) creates iterator from iterable
my_list = [1, 2, 3]
iterator = iter(my_list)

# 2. next(iterator) gets next element
print(next(iterator))  # 1

# 3. Iterators are also iterables (return themselves)
iterator2 = iter(iterator)
print(iterator2 is iterator)  # True

# 4. for loop automates the entire process
for item in my_list:  # Automatically: iter() + next() + StopIteration handling
    print(item)
```


### Verification

```python
from collections.abc import Iterable, Iterator

my_list = [1, 2, 3]
iterator = iter(my_list)

print(isinstance(my_list, Iterable))    # True
print(isinstance(iterator, Iterator))   # True
print(isinstance(iterator, Iterable))   # True (iterators are also iterables)
```


***

## 9. Creating Custom Iterators

### Counter Iterator

Counts from `start` to `end` (inclusive).

```python
class Counter:
    def __init__(self, start, end):
        self.current = start
        self.end = end
    
    def __iter__(self):
        return self  # Iterator returns itself
    
    def __next__(self):
        if self.current > self.end:
            raise StopIteration
        value = self.current
        self.current += 1  # Mutate state
        return value

# Usage
for num in Counter(1, 5):
    print(num)  # 1, 2, 3, 4, 5
```

**Line-by-line:**

- `__init__`: Stores start/end, initializes `current`
- `__iter__`: Returns self (required by protocol)
- `__next__`: Checks limit, returns value, increments state


### Reverse Iterator

Iterates a sequence in reverse.

```python
class Reverse:
    def __init__(self, data):
        self.data = data
        self.index = len(data)  # Start at end
    
    def __iter__(self):
        return self
    
    def __next__(self):
        if self.index == 0:
            raise StopIteration
        self.index -= 1  # Move backward
        return self.data[self.index]

# Usage
for char in Reverse("Python"):
    print(char)  # n, o, h, t, y, P
```


### Range-like Iterator

Custom implementation similar to `range()`.

```python
class CustomRange:
    def __init__(self, start, stop, step=1):
        self.start = start
        self.stop = stop
        self.step = step
        self.current = start
    
    def __iter__(self):
        return self
    
    def __next__(self):
        # Handle both positive and negative steps
        if self.step > 0 and self.current >= self.stop:
            raise StopIteration
        if self.step < 0 and self.current <= self.stop:
            raise StopIteration
        
        value = self.current
        self.current += self.step
        return value

# Usage
for num in CustomRange(0, 10, 2):
    print(num)  # 0, 2, 4, 6, 8

for num in CustomRange(5, 0, -1):
    print(num)  # 5, 4, 3, 2, 1
```


### Iterator Returning New Object (Better Design)

```python
class CountUp:
    def __init__(self, start, end):
        self.start = start
        self.end = end
    
    def __iter__(self):
        # Return NEW iterator each time
        return CountIterator(self.start, self.end)

class CountIterator:
    def __init__(self, start, end):
        self.current = start
        self.end = end
    
    def __iter__(self):
        return self
    
    def __next__(self):
        if self.current > self.end:
            raise StopIteration
        value = self.current
        self.current += 1
        return value

# Usage - can iterate multiple times!
counter = CountUp(1, 3)
print(list(counter))  # [1, 2, 3]
print(list(counter))  # [1, 2, 3] (works again!)
```


***

## 10. Iterator Design Patterns

### Lazy Evaluation

Generate values on-demand, not upfront.

```python
def lazy_range(n):
    """Generate 0 to n-1 lazily"""
    for i in range(n):
        yield i  # Generator = automatic iterator

# Memory efficient
for num in lazy_range(1000000):
    if num > 5:
        break  # Only 7 values created
```

**Comparison:**

```python
# Lazy (iterator)
lazy = lazy_range(1000000)  # O(1) memory

# Not lazy (list)
not_lazy = [i for i in range(1000000)]  # O(n) memory
```


### Stream Processing

Process data as it flows through an iterator chain.

```python
def read_lines(filename):
    with open(filename) as f:
        for line in f:  # File is iterator
            yield line

def filter_long_lines(lines, min_length):
    for line in lines:
        if len(line) >= min_length:
            yield line

def extract_words(lines):
    for line in lines:
        for word in line.split():
            yield word

# Chain iterators (stream processing)
words = extract_words(filter_long_lines(read_lines('data.txt'), 10))
for word in words:
    print(word)  # Process one word at a time
```


### Sequential Traversal

Traverse multiple sequences sequentially.

```python
class SequentialIterator:
    def __init__(self, *iterables):
        self.iterables = iterables
        self.current_iter = 0
        self.current_iterator = None
    
    def __iter__(self):
        return self
    
    def __next__(self):
        while self.current_iter < len(self.iterables):
            if self.current_iterator is None:
                self.current_iterator = iter(self.iterables[self.current_iter])
            
            try:
                return next(self.current_iterator)
            except StopIteration:
                self.current_iter += 1
                self.current_iterator = None
        
        raise StopIteration

# Usage
seq = SequentialIterator([1, 2], [3, 4], [5, 6])
print(list(seq))  # [1, 2, 3, 4, 5, 6]
```


***

## 11. Performance Considerations

### Memory Usage

| Approach | Memory | Use Case |
| :-- | :-- | :-- |
| `list(range(n))` | O(n) | Small datasets, need indexing |
| `range(n)` | O(1) | Large datasets, sequential access |
| Generator | O(1) | Infinite streams, filtering |

```python
import sys

# List - stores all elements
my_list = list(range(10000))
print(sys.getsizeof(my_list))  # ~80000 bytes

# range - stores only start/stop/step
my_range = range(10000)
print(sys.getsizeof(my_range))  # ~48 bytes

# Iterator - stores only current state
my_iter = iter(my_range)
print(sys.getsizeof(my_iter))  # ~56 bytes
```


### Speed Comparison

```python
import time

# List comprehension
start = time.time()
result = [x * 2 for x in range(1000000)]
print(f"List: {time.time() - start:.4f}s")

# Generator
start = time.time()
result = sum(x * 2 for x in range(1000000))
print(f"Generator: {time.time() - start:.4f}s")
```

**Typical results:**

- List: Faster for single pass (no per-iteration overhead)
- Generator: Faster for partial processing, filtering, or when memory is constrained


### Comparison with Lists

| Feature | List | Iterator |
| :-- | :-- | :-- |
| Memory | High (all elements) | Low (current state) |
| Indexing | Yes (`list[5]`) | No |
| Reversible | Yes | No (exhausted) |
| Length | `len(list)` | Unknown |
| Mutability | Yes | No (state only) |
| Multiple passes | Yes | No |

```python
# Best for lists
data = [1, 2, 3, 4, 5]
print(data[2])  # 3
print(len(data))  # 5
print(data[::-1])  # [5, 4, 3, 2, 1]

# Best for iterators
data_iter = iter(range(1000000))
print(next(data_iter))  # 0
# Cannot do: data_iter[2], len(data_iter), data_iter[::-1]
```


***

## 12. Best Practices

### ✅ Do

1. **Use `for` loops** for simple iteration (cleaner, automatic)
```python
for item in iterable:
    process(item)
```

2. **Use generators** for lazy evaluation
```python
def read_large_file(filename):
    with open(filename) as f:
        for line in f:
            yield line
```

3. **Use `next()` with default** to avoid exceptions
```python
first = next(iterator, None)
```

4. **Reset iterators** by creating new ones
```python
iterator = iter(data)
# ... use iterator ...
iterator = iter(data)  # Reset
```

5. **Use `enumerate()`** for index + value
```python
for i, item in enumerate(iterable):
    print(i, item)
```


### ❌ Don't

1. **Don't reuse exhausted iterators**
```python
# Bad
iterator = iter([1, 2, 3])
list(iterator)  # [1, 2, 3]
list(iterator)  # [] (empty!)
```

2. **Don't store iterators** across function calls (they're stateful)
```python
# Bad
def get_iterator():
    return iter([1, 2, 3])

iter1 = get_iterator()
iter2 = get_iterator()
# iter1 and iter2 are independent, but...
```

3. **Don't assume iterator length**
```python
# Bad
iterator = iter(data)
for i in range(len(iterator)):  # TypeError!
    ...
```

4. **Don't mix iteration methods** on same iterator
```python
# Bad
iterator = iter([1, 2, 3, 4])
for item in iterator:
    if item == 2:
        break
rest = list(iterator)  # Works, but confusing
```


***

## 13. Common Errors

### TypeError: object is not iterable

```python
# Error
num = 42
for x in num:  # TypeError: 'int' object is not iterable
    print(x)

# Fix
num = 42
for x in [num]:  # Wrap in list
    print(x)
```


### TypeError: 'xxx' object is not an iterator

```python
# Error
my_list = [1, 2, 3]
print(next(my_list))  # TypeError: 'list' object is not an iterator

# Fix
print(next(iter(my_list)))  # Correct: convert to iterator first
```


### StopIteration Not Caught

```python
# Error
iterator = iter([1, 2])
result = next(iterator)  # 1
result = next(iterator)  # 2
result = next(iterator)  # StopIteration!

# Fix with default
result = next(iterator, None)  # None

# Fix with try/except
try:
    result = next(iterator)
except StopIteration:
    result = None
```


### Reusing Iterator

```python
# Error
iterator = iter([1, 2, 3])
sum1 = sum(iterator)  # 6
sum2 = sum(iterator)  # 0 (exhausted!)

# Fix
data = [1, 2, 3]
iterator = iter(data)
sum1 = sum(iterator)  # 6
iterator = iter(data)  # Reset
sum2 = sum(iterator)  # 6
```


### Infinite Iterator Without Break

```python
# Error
def infinite_counter():
    i = 0
    while True:
        yield i
        i += 1

for num in infinite_counter():  # Never ends!
    print(num)

# Fix - add break condition
for num in infinite_counter():
    if num > 5:
        break
    print(num)
```


***

## 14. Interview Questions

### Question 1: What is the difference between iterable and iterator?

**Answer:**

- **Iterable**: Object that can return an iterator via `iter()` (e.g., list, tuple, dict)
- **Iterator**: Object with `__iter__()` and `__next__()` methods that maintains state and yields elements sequentially

```python
# Iterable
my_list = [1, 2, 3]
hasattr(my_list, '__iter__')  # True

# Iterator
my_iterator = iter(my_list)
hasattr(my_iterator, '__next__')  # True
```


### Question 2: How does a `for` loop work internally?

**Answer:**

```python
# What for loop does:
for item in iterable:
    process(item)

# Internally:
iterator = iter(iterable)
while True:
    try:
        item = next(iterator)
        process(item)
    except StopIteration:
        break
```


### Question 3: Create a custom iterator that yields Fibonacci numbers

```python
class Fibonacci:
    def __init__(self, limit):
        self.limit = limit
        self.count = 0
        self.a, self.b = 0, 1
    
    def __iter__(self):
        return self
    
    def __next__(self):
        if self.count >= self.limit:
            raise StopIteration
        result = self.a
        self.a, self.b = self.b, self.a + self.b
        self.count += 1
        return result

# Usage
for num in Fibonacci(10):
    print(num)  # 0, 1, 1, 2, 3, 5, 8, 13, 21, 34
```


### Question 4: What happens when you call `iter()` on an iterator?

**Answer:** It returns the iterator itself (by convention, iterators implement `__iter__()` to return `self`).

```python
iterator = iter([1, 2, 3])
iterator2 = iter(iterator)
print(iterator is iterator2)  # True
```


### Question 5: Implement an iterator that flattens a nested list

```python
class FlattenIterator:
    def __init__(self, nested_list):
        self.nested_list = nested_list
        self.outer_idx = 0
        self.inner_iter = None
    
    def __iter__(self):
        return self
    
    def __next__(self):
        while self.outer_idx < len(self.nested_list):
            if self.inner_iter is None:
                self.inner_iter = iter(self.nested_list[self.outer_idx])
            
            try:
                return next(self.inner_iter)
            except StopIteration:
                self.outer_idx += 1
                self.inner_iter = None
        
        raise StopIteration

# Usage
nested = [[1, 2], [3, 4], [5]]
print(list(FlattenIterator(nested)))  # [1, 2, 3, 4, 5]
```


### Question 6: Why are iterators one-time use?

**Answer:** Iterators maintain internal state (current position). Once exhausted, the state is at the end and cannot reset automatically. This design:

- Saves memory (no need to store position history)
- Enables infinite sequences
- Encourages explicit reset patterns

***

## 15. Practice Exercises

### Exercise 1: Circle Iterator

Create an iterator that cycles through colors indefinitely.

```python
# Solution
class ColorCycle:
    def __init__(self, colors):
        self.colors = colors
        self.index = 0
    
    def __iter__(self):
        return self
    
    def __next__(self):
        color = self.colors[self.index]
        self.index = (self.index + 1) % len(self.colors)
        return color

# Test
colors = ColorCycle(['red', 'green', 'blue'])
for i in range(7):
    print(next(colors))  # red, green, blue, red, green, blue, red
```


### Exercise 2: Word Length Iterator

Iterator yielding lengths of words in a string.

```python
# Solution
class WordLengths:
    def __init__(self, text):
        self.words = text.split()
        self.index = 0
    
    def __iter__(self):
        return self
    
    def __next__(self):
        if self.index >= len(self.words):
            raise StopIteration
        length = len(self.words[self.index])
        self.index += 1
        return length

# Test
text = "Python is awesome"
for length in WordLengths(text):
    print(length)  # 6, 2, 7
```


### Exercise 3: Zip Iterator

Combine two iterators into pairs (like `zip()`).

```python
# Solution
class ZipIterator:
    def __init__(self, iter1, iter2):
        self.iter1 = iter(iter1)
        self.iter2 = iter(iter2)
    
    def __iter__(self):
        return self
    
    def __next__(self):
        try:
            item1 = next(self.iter1)
            item2 = next(self.iter2)
            return (item1, item2)
        except StopIteration:
            raise StopIteration

# Test
pairs = ZipIterator([1, 2, 3], ['a', 'b', 'c'])
print(list(pairs))  # [(1, 'a'), (2, 'b'), (3, 'c')]
```


### Exercise 4: Filter Iterator

Create an iterator that filters elements based on a condition.

```python
# Solution
class FilterIterator:
    def __init__(self, iterable, condition):
        self.iterator = iter(iterable)
        self.condition = condition
    
    def __iter__(self):
        return self
    
    def __next__(self):
        while True:
            item = next(self.iterator)
            if self.condition(item):
                return item

# Test
evens = FilterIterator(range(10), lambda x: x % 2 == 0)
print(list(evens))  # [0, 2, 4, 6, 8]

words = FilterIterator(['hi', 'hello', 'bye'], lambda w: len(w) > 2)
print(list(words))  # ['hello']
```


### Exercise 5: Manual Sum Using Iterator

Implement `sum()` using manual iteration.

```python
# Solution
def manual_sum(iterable):
    iterator = iter(iterable)
    total = 0
    while True:
        try:
            total += next(iterator)
        except StopIteration:
            break
    return total

# Test
print(manual_sum([1, 2, 3, 4, 5]))  # 15
print(manual_sum(range(100)))  # 4950
```


### Exercise 6: Take Iterator (First N Elements)

```python
# Solution
def take(n, iterable):
    iterator = iter(iterable)
    result = []
    for _ in range(n):
        try:
            result.append(next(iterator))
        except StopIteration:
            break
    return result

# Test
print(take(3, range(10)))  # [0, 1, 2]
print(take(5, [1, 2]))  # [1, 2]
```


### Exercise 7: Challenge - Infinite Prime Iterator

```python
# Solution
def is_prime(n):
    if n < 2:
        return False
    for i in range(2, int(n ** 0.5) + 1):
        if n % i == 0:
            return False
    return True

class PrimeIterator:
    def __init__(self):
        self.current = 2
    
    def __iter__(self):
        return self
    
    def __next__(self):
        while not is_prime(self.current):
            self.current += 1
        prime = self.current
        self.current += 1
        return prime

# Test
primes = PrimeIterator()
for i in range(10):
    print(next(primes))  # 2, 3, 5, 7, 11, 13, 17, 19, 23, 29
```


***

## Quick Reference Card

```python
# Create iterator
iterator = iter(iterable)

# Get next element
item = next(iterator)
item = next(iterator, default)  # With default

# Manual loop
while True:
    try:
        item = next(iterator)
        process(item)
    except StopIteration:
        break

# Check if iterable
from collections.abc import Iterable
isinstance(obj, Iterable)  # True

# Check if iterator
from collections.abc import Iterator
isinstance(obj, Iterator)  # True

# Custom iterator
class MyIterator:
    def __iter__(self):
        return self
    
    def __next__(self):
        # Return value or raise StopIteration
        pass
```


***

**Master iterators, master Python iteration!** 🚀

