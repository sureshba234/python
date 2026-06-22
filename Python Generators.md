<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" style="height:64px;margin-right:32px"/>

# Python Generators

## Table of Contents

1. [Why Generators Exist](#1-why-generators-exist)
2. [Generator Functions](#2-generator-functions)
3. [Yield Statement](#3-yield-statement)
4. [Generator Lifecycle](#4-generator-lifecycle)
5. [Generator Objects](#5-generator-objects)
6. [Generator Expressions](#6-generator-expressions)
7. [Memory Comparisons](#7-memory-comparisons)
8. [Generator Methods](#8-generator-methods)
9. [Yield From](#9-yield-from)
10. [Recursive Generators](#10-recursive-generators)
11. [Coroutine Basics](#11-coroutine-basics)
12. [Real-World Applications](#12-real-world-applications)
13. [Best Practices](#13-best-practices)
14. [Common Mistakes](#14-common-mistakes)
15. [Interview Questions](#15-interview-questions)
16. [Exercises](#16-exercises)

***

## 1. Why Generators Exist

Python generators solve three critical problems in data processing:

### Memory Problems

When processing large datasets, traditional approaches create entire lists in memory:

```python
# Problem: Creates entire list in memory
def get_numbers(n):
    return [i for i in range(n)]  # All n numbers stored simultaneously

numbers = get_numbers(10_000_000)  # ~80MB for integers
```

For massive datasets (millions/billions of items), this exhausts RAM and causes program failure.

### Lazy Evaluation

Generators implement **lazy evaluation** - values are computed only when requested, not upfront:

```python
# Solution: Values computed on-demand
def get_numbers(n):
    for i in range(n):
        yield i  # Computed when next() is called

numbers = get_numbers(10_000_000)  # Only ~48 bytes for generator object
```


### Streaming

Generators enable **streaming** processing of infinite or continuous data:

```python
# Infinite sequence - never exhausted
def infinite_counter():
    count = 0
    while True:
        yield count
        count += 1

counter = infinite_counter()
print(next(counter))  # 0
print(next(counter))  # 1
# Continues indefinitely
```

This is essential for:

- Real-time data streams
- Log file processing
- Network socket reading
- Sensor data streams

***

## 2. Generator Functions

A **generator function** is a function that uses the `yield` statement instead of `return`.

### The `yield` Statement

`yield` is the core mechanism that makes a function a generator:

```python
def simple_generator():
    yield 1
    yield 2
    yield 3
```

**Key differences from `return`:**


| Aspect | `return` | `yield` |
| :-- | :-- | :-- |
| Execution | Ends function | Suspends function |
| State | Lost | Preserved |
| Calls | One value | Multiple values |
| Result | Regular function | Generator object |

### How `yield` Works

When Python sees `yield` in a function:

1. Function becomes a **generator function**
2. Calling it returns a **generator object** (not executing the function)
3. Each `next()` call executes until `yield`, returns the value, and suspends
4. Next `next()` resumes from suspension point

### Examples

```python
# Basic generator function
def count_up_to(n):
    """Count from 0 to n-1"""
    for i in range(n):
        yield i

# Usage
counter = count_up_to(5)
print(next(counter))  # 0
print(next(counter))  # 1
print(next(counter))  # 2
```

```python
# Generator with multiple yields
def three_values():
    yield "first"
    yield "second"
    yield "third"

gen = three_values()
print(next(gen))  # "first"
print(next(gen))  # "second"
print(next(gen))  # "third"
```

```python
# Generator with computation
def squared_numbers(n):
    """Yield squared numbers from 0 to n-1"""
    for i in range(n):
        yield i ** 2

squares = squared_numbers(5)
for square in squares:
    print(square)  # 0, 1, 4, 9, 16
```


***

## 3. Yield Statement

The `yield` statement is more complex than it appears. Let's explore it in depth.

### Execution Suspension

When `yield` is executed:

```python
def example():
    print("Before yield")
    value = yield 42
    print("After yield", value)
```

**Execution flow:**

1. `next(gen)` called
2. `"Before yield"` printed
3. `yield 42` executed:
    - Value `42` returned to caller
    - Function **suspended** at this line
    - Local state preserved (`value` variable exists but not assigned)
4. Next `next(gen, None)` called:
    - Function **resumes** after `yield`
    - `value` gets `None` (default for `next()`)
    - `"After yield None"` printed
    - Function ends (raises `StopIteration`)

### Resumption

Function resumes **exactly** where it was suspended:

```python
def resume_example():
    for i in range(3):
        print(f"Before yield {i}")
        yield i
        print(f"After yield {i}")

gen = resume_example()
print("=== First next() ===")
print(next(gen))  # Before yield 0, yields 0

print("=== Second next() ===")
print(next(gen))  # After yield 0, Before yield 1, yields 1

print("=== Third next() ===")
print(next(gen))  # After yield 1, Before yield 2, yields 2
```

Output:

```
=== First next() ===
Before yield 0
0
=== Second next() ===
After yield 0
Before yield 1
1
=== Third next() ===
After yield 1
Before yield 2
2
```


### State Preservation

All local state is preserved between yields:

```python
def state_preservation():
    total = 0
    for i in range(5):
        total += i
        print(f"total = {total}, i = {i}")
        yield total

gen = state_preservation()
print(next(gen))  # total=0, i=0, yields 0
print(next(gen))  # total=1, i=1, yields 1
print(next(gen))  # total=3, i=2, yields 3
# total and i preserved!
```


### Detailed Execution Flow

```python
def execution_flow():
    a = 1
    print(f"1. a = {a}")
    yield a
    
    b = 2
    print(f"2. b = {b}, a = {a}")
    yield a + b
    
    a = 10
    print(f"3. a = {a}, b = {b}")
    yield a * b

gen = execution_flow()

print("=== Call 1 ===")
result1 = next(gen)
print(f"Got: {result1}")

print("=== Call 2 ===")
result2 = next(gen)
print(f"Got: {result2}")

print("=== Call 3 ===")
result3 = next(gen)
print(f"Got: {result3}")
```

Output:

```
=== Call 1 ===
1. a = 1
Got: 1
=== Call 2 ===
2. b = 2, a = 1
Got: 3
=== Call 3 ===
3. a = 10, b = 2
Got: 20
```


***

## 4. Generator Lifecycle

A generator object has five distinct lifecycle stages:

### 1. Creation

```python
def generator_func():
    yield 1

gen = generator_func()  # Created but not executed
print(type(gen))        # <class 'generator'>
print(gen)              # <generator object generator_func at 0x...>
```

At creation:

- Function code **not executed**
- Generator object instantiated
- Ready for first `next()` call


### 2. Execution

```python
def execution_demo():
    print("Starting execution")
    yield 1
    print("Continuing")
    yield 2

gen = execution_demo()
print("Before first next()")
result = next(gen)  # Execution starts here
print(f"Got: {result}")
```

Output:

```
Before first next()
Starting execution
Got: 1
```


### 3. Pause

```python
def pause_demo():
    yield 1  # Suspends here
    yield 2  # Suspends here
    yield 3  # Suspends here

gen = pause_demo()
next(gen)  # Paused at first yield
next(gen)  # Paused at second yield
# Generator still exists, can resume
```

At pause:

- Execution suspended at `yield`
- All local variables preserved
- Can resume with another `next()`


### 4. Resume

```python
def resume_demo():
    value = yield "first"
    print(f"Resumed with: {value}")
    yield "second"

gen = resume_demo()
print(next(gen))              # Pauses, yields "first"
print(next(gen))              # Resumes, prints "Resumed with: None", yields "second"
```

Output:

```
first
Resumed with: None
second
```


### 5. Termination

```python
def termination_demo():
    yield 1
    yield 2
    # No more yields - function ends

gen = termination_demo()
print(next(gen))  # 1
print(next(gen))  # 2
print(next(gen))  # Raises StopIteration
```

Termination occurs when:

- Function reaches end (no more `yield`)
- `return` statement executed
- `close()` method called
- Exception raised

```python
def with_return():
    yield 1
    return "done"  # Terminates generator

gen = with_return()
print(next(gen))  # 1
next(gen)         # StopIteration (return value ignored)
```


***

## 5. Generator Objects

Generator objects are first-classPython objects with type, attributes, and methods.

### Type

```python
def gen_func():
    yield 1

gen = gen_func()
print(type(gen))  # <class 'generator'>
print(isinstance(gen, object))  # True
```


### Attributes

```python
def demo():
    x = 10
    yield x

gen = demo()
next(gen)

# Access attributes
print(gen.gi_code)        # Code object
print(gen.gi_frame)       # Frame object (execution state)
print(gen.gi_running)     # False (not currently running)
print(gen.gi_yieldfrom)   # None (not using yield from)
```

Key attributes:

- `gi_code`: Code object of the generator
- `gi_frame`: Execution frame (contains local variables)
- `gi_running`: Boolean, True if generator is executing
- `gi_yieldfrom`: Object being yielded from (if using `yield from`)


### Methods

```python
def method_demo():
    yield 1
    yield 2

gen = method_demo()

# next() - built-in, calls gen.__next__()
print(next(gen))  # 1

# __next__() - explicit
print(gen.__next__)  # 2

# __iter__() - makes generator iterable
for item in gen:
    print(item)  # 2 (only remaining value)
```


### Complete Example

```python
def generator_methods():
    value = 0
    while value < 3:
        print(f"Yielding {value}")
        yield value
        value += 1

gen = generator_methods()

# Check type
print(f"Type: {type(gen)}")

# Iterate
print("Iterating:")
for item in gen:
    print(f"  Got: {item}")

# Check attributes after iteration
print(f"gi_running: {gen.gi_running}")  # False (finished)
```

Output:

```
Type: <class 'generator'>
Iterating:
Yielding 0
  Got: 0
Yielding 1
  Got: 1
Yielding 2
  Got: 2
gi_running: False
```


***

## 6. Generator Expressions

**Generator expressions** are inline generator definitions using concise syntax.

### Basic Syntax

```python
# Generator expression
gen = (x for x in range(5))
print(type(gen))  # <class 'generator'>

# Usage
for value in gen:
    print(value)  # 0, 1, 2, 3, 4
```


### Comparison with Comprehensions

```python
# Generator expression (lazy)
gen_expr = (x for x in range(5))
print(type(gen_expr))  # generator

# List comprehension (immediate, stores all)
list_comp = [x for x in range(5)]
print(type(list_comp))  # list
print(list_comp)        # [0, 1, 2, 3, 4]

# Set comprehension
set_comp = {x for x in range(5)}
print(type(set_comp))  # set
print(set_comp)        # {0, 1, 2, 3, 4}

# Dict comprehension
dict_comp = {x: x**2 for x in range(5)}
print(type(dict_comp))  # dict
print(dict_comp)        # {0: 0, 1: 1, 2: 4, 3: 9, 4: 16}
```


### Key Differences

| Feature | Generator Expression | List Comp | Set Comp | Dict Comp |
| :-- | :-- | :-- | :-- | :-- |
| Type | `generator` | `list` | `set` | `dict` |
| Evaluation | Lazy | Immediate | Immediate | Immediate |
| Memory | O(1) | O(n) | O(n) | O(n) |
| Reusable | No (single pass) | Yes | Yes | Yes |
| Ordered | Yes | Yes | No | Yes (3.7+) |
| Supports indexing | No | Yes | No | Yes (keys) |

### Generator Expression Examples

```python
# With condition
squares = (x**2 for x in range(10) if x % 2 == 0)
for square in squares:
    print(square)  # 0, 4, 16, 36, 64
```

```python
# Nested generator expressions
pairs = ((x, y) for x in range(3) for y in range(3))
for pair in pairs:
    print(pair)  # (0,0), (0,1), (0,2), (1,0), ...
```

```python
# Chained generator expressions
gen1 = (x for x in range(5))
gen2 = (x * 2 for x in gen1)
gen3 = (x + 1 for x in gen2)

for value in gen3:
    print(value)  # 1, 3, 5, 7, 9
```

```python
# Generator expression with function
def process(n):
    return n ** 3 + 1

result = (process(x) for x in range(4))
for value in result:
    print(value)  # 1, 2, 9, 28
```


***

## 7. Memory Comparisons

Generators provide massive memory savings compared to lists.

### Benchmark Setup

```python
import sys
import time

def list_version(n):
    return [i ** 2 for i in range(n)]

def generator_version(n):
    return (i ** 2 for i in range(n))

def test_memory(n):
    list_data = list_version(n)
    gen_data = generator_version(n)
    
    list_size = sys.getsizeof(list_data)
    gen_size = sys.getsizeof(gen_data)
    
    print(f"n = {n}")
    print(f"List size: {list_size} bytes ({list_size/1024/1024:.2f} MB)")
    print(f"Generator size: {gen_size} bytes ({gen_size/1024:.2f} KB)")
    print(f"Savings: {list_size - gen_size} bytes ({(list_size - gen_size)/list_size*100:.1f}%)")
    print()

# Run benchmarks
for n in [1000, 10000, 100000, 1000000]:
    test_memory(n)
```

Let me execute this benchmark:

