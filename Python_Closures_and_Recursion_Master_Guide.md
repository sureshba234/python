# Python Closures and Recursion Master Guide

> A comprehensive reference from beginner to expert — covering nested functions, closures, `nonlocal`, recursion, recursive algorithms, tail recursion, and memoization.

---

## Table of Contents

1. [Nested Functions](#1-nested-functions)
   - 1.1 [What Are Nested Functions?](#11-what-are-nested-functions)
   - 1.2 [Scope and Visibility](#12-scope-and-visibility)
   - 1.3 [Why Use Nested Functions?](#13-why-use-nested-functions)
   - 1.4 [Practical Examples](#14-practical-examples)

2. [Closures](#2-closures)
   - 2.1 [What Is a Closure?](#21-what-is-a-closure)
   - 2.2 [Capturing Variables](#22-capturing-variables)
   - 2.3 [State Preservation](#23-state-preservation)
   - 2.4 [Closure Inspection](#24-closure-inspection)
   - 2.5 [Common Closure Patterns](#25-common-closure-patterns)

3. [The `nonlocal` Keyword](#3-the-nonlocal-keyword)
   - 3.1 [Why `nonlocal` Exists](#31-why-nonlocal-exists)
   - 3.2 [Syntax and Rules](#32-syntax-and-rules)
   - 3.3 [Deep Nesting with `nonlocal`](#33-deep-nesting-with-nonlocal)
   - 3.4 [`nonlocal` vs `global`](#34-nonlocal-vs-global)

4. [Recursion Fundamentals](#4-recursion-fundamentals)
   - 4.1 [What Is Recursion?](#41-what-is-recursion)
   - 4.2 [Base Case](#42-base-case)
   - 4.3 [Recursive Case](#43-recursive-case)
   - 4.4 [The Call Stack — High Level](#44-the-call-stack--high-level)
   - 4.5 [Recursion Depth and Limits](#45-recursion-depth-and-limits)

5. [Recursive Algorithms](#5-recursive-algorithms)
   - 5.1 [Factorial](#51-factorial)
   - 5.2 [Fibonacci](#52-fibonacci)
   - 5.3 [Binary Search](#53-binary-search)
   - 5.4 [Tree Traversal](#54-tree-traversal)
   - 5.5 [Merge Sort](#55-merge-sort)
   - 5.6 [Power Function](#56-power-function)
   - 5.7 [Flatten Nested Lists](#57-flatten-nested-lists)

6. [Tail Recursion](#6-tail-recursion)
   - 6.1 [What Is Tail Recursion?](#61-what-is-tail-recursion)
   - 6.2 [Python's Limitations](#62-pythons-limitations)
   - 6.3 [Simulating Tail Recursion](#63-simulating-tail-recursion)
   - 6.4 [Trampolining](#64-trampolining)

7. [Memoization](#7-memoization)
   - 7.1 [The Problem Without Memoization](#71-the-problem-without-memoization)
   - 7.2 [Manual Memoization with a Dictionary](#72-manual-memoization-with-a-dictionary)
   - 7.3 [`functools.lru_cache`](#73-functoolslru_cache)
   - 7.4 [`functools.cache`](#74-functoolscache)
   - 7.5 [Performance Analysis](#75-performance-analysis)
   - 7.6 [Cache Invalidation Strategies](#76-cache-invalidation-strategies)

8. [Comparison Tables](#8-comparison-tables)
   - 8.1 [Loop vs Recursion](#81-loop-vs-recursion)
   - 8.2 [Closure vs Class](#82-closure-vs-class)
   - 8.3 [Normal Recursion vs Memoized Recursion](#83-normal-recursion-vs-memoized-recursion)

9. [Real-World Projects](#9-real-world-projects)
   - 9.1 [Recursive Directory Scanner](#91-recursive-directory-scanner)
   - 9.2 [Expression Evaluator](#92-expression-evaluator)
   - 9.3 [Configuration Closures](#93-configuration-closures)
   - 9.4 [Caching System](#94-caching-system)

10. [Cheat Sheets](#10-cheat-sheets)
    - 10.1 [Closures Cheat Sheet](#101-closures-cheat-sheet)
    - 10.2 [Recursion Cheat Sheet](#102-recursion-cheat-sheet)
    - 10.3 [Memoization Cheat Sheet](#103-memoization-cheat-sheet)

11. [Interview Master Section](#11-interview-master-section)
    - 11.1 [Closure Questions](#111-closure-questions)
    - 11.2 [Recursion Questions](#112-recursion-questions)
    - 11.3 [Memoization Questions](#113-memoization-questions)
    - 11.4 [Advanced Questions](#114-advanced-questions)

12. [Exercises](#12-exercises)
    - 12.1 [Intermediate (15 Problems)](#121-intermediate-15-problems)
    - 12.2 [Advanced (15 Problems)](#122-advanced-15-problems)
    - 12.3 [Expert (15 Problems)](#123-expert-15-problems)
    - 12.4 [Solutions](#124-solutions)

13. [Final Section](#13-final-section)
    - 13.1 [Summary](#131-summary)
    - 13.2 [Best Practices](#132-best-practices)
    - 13.3 [Common Pitfalls](#133-common-pitfalls)
    - 13.4 [Learning Roadmap](#134-learning-roadmap)

---

## 1. Nested Functions

### 1.1 What Are Nested Functions?

A **nested function** (also called an *inner function*) is a function defined inside another function. Python fully supports this: inner functions have access to the variables of their enclosing scope.

```python
def outer():
    message = "Hello from outer!"

    def inner():
        print(message)   # inner can see outer's variable

    inner()

outer()
```

**Output:**
```
Hello from outer!
```

The inner function `inner` is defined and called within the body of `outer`. It can read `message` directly.

---

### 1.2 Scope and Visibility

Python uses the **LEGB** rule to resolve variable names:

| Scope | Description |
|-------|-------------|
| **L** ocal | Variables defined inside the current function |
| **E** nclosing | Variables in enclosing (outer) functions |
| **G** lobal | Variables at the module level |
| **B** uilt-in | Built-in names like `len`, `print`, `range` |

```python
x = "global"

def outer():
    x = "enclosing"

    def inner():
        x = "local"
        print(x)   # local wins

    inner()
    print(x)       # enclosing

outer()
print(x)           # global
```

**Output:**
```
local
enclosing
global
```

---

### 1.3 Why Use Nested Functions?

- **Encapsulation** — hide helper logic that is irrelevant outside the outer function.
- **Code organisation** — keep related logic together.
- **Closures** — the main use case (see Section 2).
- **Decorator factories** — nested functions are the backbone of Python decorators.

---

### 1.4 Practical Examples

**Example: Input validation helper**

```python
def parse_age(value):
    def validate(v):
        if not isinstance(v, int):
            raise TypeError("Age must be an integer")
        if v < 0 or v > 150:
            raise ValueError("Age must be between 0 and 150")
        return v

    return validate(value)

print(parse_age(25))    # 25
print(parse_age(-5))    # ValueError
```

**Example: Reusing inner logic without polluting module namespace**

```python
def compute_statistics(numbers):
    def mean(data):
        return sum(data) / len(data)

    def variance(data):
        m = mean(data)
        return sum((x - m) ** 2 for x in data) / len(data)

    return {
        "mean": mean(numbers),
        "variance": variance(numbers),
    }

print(compute_statistics([2, 4, 4, 4, 5, 5, 7, 9]))
```

**Output:**
```
{'mean': 5.0, 'variance': 4.0}
```

---

## 2. Closures

### 2.1 What Is a Closure?

A **closure** is a nested function that *remembers* the variables from its enclosing scope even after the outer function has finished executing. Three conditions must be met:

1. There must be a nested function.
2. The nested function must refer to a variable from the enclosing scope.
3. The enclosing function must return the nested function.

```python
def make_greeting(name):
    # 'name' is a free variable captured by the closure
    def greet():
        print(f"Hello, {name}!")
    return greet   # return the function object, not its result

say_hello = make_greeting("Alice")
say_goodbye = make_greeting("Bob")

say_hello()    # Hello, Alice!
say_goodbye()  # Hello, Bob!
```

**Output:**
```
Hello, Alice!
Hello, Bob!
```

`make_greeting` has already returned, yet `say_hello` still has access to `name = "Alice"`. That persistent access is the closure.

---

### 2.2 Capturing Variables

Inner functions capture **references** to variables, not copies. This is critical to understand:

```python
def make_multipliers():
    multipliers = []
    for i in range(1, 4):
        def multiply(x, n=i):   # capture current value with default arg
            return x * n
        multipliers.append(multiply)
    return multipliers

double, triple, quadruple = make_multipliers()
print(double(10))    # 20
print(triple(10))    # 30
print(quadruple(10)) # 40
```

**The classic loop pitfall — without default argument:**

```python
def make_multipliers_buggy():
    multipliers = []
    for i in range(1, 4):
        def multiply(x):
            return x * i   # captures reference to i, not its value
        multipliers.append(multiply)
    return multipliers

fns = make_multipliers_buggy()
print(fns[0](10))  # 30  ← expected 10, got 30
print(fns[1](10))  # 30  ← expected 20
print(fns[2](10))  # 30  ← expected 30 (only this is correct)
```

All three functions see `i = 3` (the final value of the loop variable).

**Fixes:**

```python
# Fix 1: Default argument (most common)
def multiply(x, n=i):
    return x * n

# Fix 2: functools.partial
from functools import partial

def multiply(n, x):
    return n * x

multipliers = [partial(multiply, i) for i in range(1, 4)]

# Fix 3: Another closure
def make_multiplier(n):
    def multiply(x):
        return x * n
    return multiply

multipliers = [make_multiplier(i) for i in range(1, 4)]
```

---

### 2.3 State Preservation

Closures can maintain **mutable state** across calls — a lightweight alternative to classes:

```python
def make_counter(start=0, step=1):
    count = [start]   # list so we can mutate without nonlocal (pre-3.x style)

    def counter():
        current = count[0]
        count[0] += step
        return current

    return counter

c1 = make_counter()
c2 = make_counter(start=100, step=5)

print(c1())   # 0
print(c1())   # 1
print(c1())   # 2
print(c2())   # 100
print(c2())   # 105
```

**Output:**
```
0
1
2
100
105
```

Each closure instance has its own independent copy of `count`.

---

### 2.4 Closure Inspection

Python exposes closure internals through attributes:

```python
def make_adder(n):
    def add(x):
        return x + n
    return add

add5 = make_adder(5)

# Inspect the closure
print(add5.__closure__)           # tuple of cell objects
print(add5.__closure__[0].cell_contents)  # 5
print(add5.__code__.co_freevars)  # ('n',)
```

**Output:**
```
(<cell at 0x...: int object at 0x...>,)
5
('n',)
```

---

### 2.5 Common Closure Patterns

**Pattern 1: Decorator**

```python
def logger(func):
    def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__} with {args}, {kwargs}")
        result = func(*args, **kwargs)
        print(f"{func.__name__} returned {result}")
        return result
    return wrapper

@logger
def add(a, b):
    return a + b

add(3, 4)
```

**Output:**
```
Calling add with (3, 4), {}
add returned 7
```

**Pattern 2: Partial application**

```python
def power_factory(exponent):
    def power(base):
        return base ** exponent
    return power

square = power_factory(2)
cube   = power_factory(3)

print(square(4))  # 16
print(cube(3))    # 27
```

**Pattern 3: Event handler / callback**

```python
def make_button_handler(button_name):
    click_count = [0]

    def on_click():
        click_count[0] += 1
        print(f"Button '{button_name}' clicked {click_count[0]} time(s)")

    return on_click

login_click  = make_button_handler("Login")
submit_click = make_button_handler("Submit")

login_click()    # Button 'Login' clicked 1 time(s)
login_click()    # Button 'Login' clicked 2 time(s)
submit_click()   # Button 'Submit' clicked 1 time(s)
```

---

## 3. The `nonlocal` Keyword

### 3.1 Why `nonlocal` Exists

In Python 3, when you assign to a variable inside an inner function, Python treats it as a **new local variable** by default. If you want to *modify* a variable from the enclosing scope (not just read it), you need `nonlocal`.

```python
def counter():
    count = 0

    def increment():
        count += 1   # UnboundLocalError: assignment before reference
        return count

    return increment
```

Without `nonlocal`, Python sees `count += 1` as `count = count + 1`, where the left-hand `count` creates a new local, and the right-hand reads an unbound local — error.

---

### 3.2 Syntax and Rules

```python
def counter():
    count = 0

    def increment():
        nonlocal count    # declare we mean the enclosing scope's 'count'
        count += 1
        return count

    return increment

c = counter()
print(c())  # 1
print(c())  # 2
print(c())  # 3
```

**Rules for `nonlocal`:**

- `nonlocal` must name an existing variable in an **enclosing (non-global) scope**.
- It cannot refer to module-level (global) variables — use `global` for that.
- Multiple variables: `nonlocal x, y, z`
- The declaration must appear **before** any use of the variable in the function.

---

### 3.3 Deep Nesting with `nonlocal`

```python
def level_one():
    value = "L1"

    def level_two():
        nonlocal value
        value = "L2"

        def level_three():
            nonlocal value
            value = "L3"

        level_three()
        print(f"After L3: {value}")  # L3

    level_two()
    print(f"After L2: {value}")      # L3

level_one()
```

**Output:**
```
After L3: L3
After L2: L3
```

Each `nonlocal` declaration in a given function binds to the **nearest enclosing scope** where that variable is defined.

---

### 3.4 `nonlocal` vs `global`

```python
x = "module level"

def outer():
    x = "outer level"

    def inner_nonlocal():
        nonlocal x
        x = "modified by nonlocal"
        print(x)

    def inner_global():
        global x
        x = "modified by global"
        print(x)

    inner_nonlocal()
    print(f"outer x: {x}")    # modified by nonlocal

outer()
print(f"module x: {x}")       # module level (unchanged)
```

| Feature | `nonlocal` | `global` |
|---------|------------|---------|
| Target scope | Nearest enclosing function | Module (global) scope |
| Variable must exist | Yes, in enclosing function | No, can create a new global |
| Side effects on outer code | Contained within function chain | Affects the whole module |

---

## 4. Recursion Fundamentals

### 4.1 What Is Recursion?

**Recursion** is when a function calls itself, directly or indirectly, to solve a smaller version of the same problem. Every recursive solution decomposes a problem into:

1. A **base case** — the smallest, trivially solvable instance.
2. A **recursive case** — the general case that calls itself with a simpler input.

```
solve(n) = combine(n, solve(smaller_n))
```

---

### 4.2 Base Case

The base case **stops the recursion**. Without it, the function calls itself forever (until a stack overflow).

```python
def countdown(n):
    if n <= 0:         # base case
        print("Blastoff!")
        return
    print(n)
    countdown(n - 1)   # recursive case

countdown(5)
```

**Output:**
```
5
4
3
2
1
Blastoff!
```

---

### 4.3 Recursive Case

The recursive case must always move **toward** the base case. The standard template:

```python
def recursive_function(input):
    # 1. Check base case
    if is_base_case(input):
        return base_value

    # 2. Reduce the problem
    smaller = reduce(input)

    # 3. Recurse
    sub_result = recursive_function(smaller)

    # 4. Combine and return
    return combine(input, sub_result)
```

---

### 4.4 The Call Stack — High Level

Each function call is placed on the **call stack** as a frame containing local variables and the return address. Recursive calls stack up until the base case is hit, then unwind in reverse order.

```
factorial(4)
  └─ 4 * factorial(3)
       └─ 3 * factorial(2)
            └─ 2 * factorial(1)
                 └─ 1 (base case)
            └─ 2 * 1 = 2
       └─ 3 * 2 = 6
  └─ 4 * 6 = 24
```

Visualising with print statements:

```python
def factorial(n, depth=0):
    indent = "  " * depth
    print(f"{indent}factorial({n}) called")

    if n <= 1:
        print(f"{indent}base case → return 1")
        return 1

    result = n * factorial(n - 1, depth + 1)
    print(f"{indent}factorial({n}) → {result}")
    return result

factorial(4)
```

**Output:**
```
factorial(4) called
  factorial(3) called
    factorial(2) called
      factorial(1) called
      base case → return 1
    factorial(2) → 2
  factorial(3) → 6
factorial(4) → 24
```

---

### 4.5 Recursion Depth and Limits

Python limits the call stack to prevent crashes:

```python
import sys
print(sys.getrecursionlimit())  # typically 1000
```

You can raise it (use carefully):

```python
sys.setrecursionlimit(5000)
```

When exceeded:

```python
def infinite(n):
    return infinite(n + 1)

infinite(0)  # RecursionError: maximum recursion depth exceeded
```

---

## 5. Recursive Algorithms

### 5.1 Factorial

**Definition:** `n! = n × (n-1) × ... × 1`, with `0! = 1`

```python
def factorial(n):
    """Compute n! recursively."""
    if n < 0:
        raise ValueError("Factorial undefined for negative numbers")
    if n == 0:           # base case
        return 1
    return n * factorial(n - 1)   # recursive case

for i in range(8):
    print(f"{i}! = {factorial(i)}")
```

**Output:**
```
0! = 1
1! = 1
2! = 2
3! = 6
4! = 24
5! = 120
6! = 720
7! = 5040
```

**Iterative equivalent for comparison:**

```python
def factorial_iter(n):
    result = 1
    for i in range(2, n + 1):
        result *= i
    return result
```

---

### 5.2 Fibonacci

**Definition:** `fib(0)=0, fib(1)=1, fib(n)=fib(n-1)+fib(n-2)`

**Naive recursive (exponential time — O(2ⁿ)):**

```python
def fib_naive(n):
    if n <= 1:
        return n
    return fib_naive(n - 1) + fib_naive(n - 2)

print([fib_naive(i) for i in range(10)])
```

**Output:**
```
[0, 1, 1, 2, 3, 5, 8, 13, 21, 34]
```

**Problem:** `fib_naive(40)` takes seconds because the same subproblems are recomputed millions of times. See Section 7 for the memoised solution.

---

### 5.3 Binary Search

Binary search recursively halves a sorted list to find a target.

```python
def binary_search(arr, target, low=0, high=None):
    if high is None:
        high = len(arr) - 1

    if low > high:
        return -1          # base case: not found

    mid = (low + high) // 2

    if arr[mid] == target:
        return mid         # base case: found

    if arr[mid] < target:
        return binary_search(arr, target, mid + 1, high)  # search right half
    else:
        return binary_search(arr, target, low, mid - 1)   # search left half

nums = [2, 5, 8, 12, 16, 23, 38, 56, 72, 91]
print(binary_search(nums, 23))   # 5
print(binary_search(nums, 100))  # -1
```

**Output:**
```
5
-1
```

Time complexity: **O(log n)** — each call halves the search space.

---

### 5.4 Tree Traversal

Define a simple binary tree node:

```python
class Node:
    def __init__(self, value, left=None, right=None):
        self.value = value
        self.left  = left
        self.right = right

# Build a sample tree:
#        10
#       /  \
#      5    15
#     / \     \
#    3   7    20

root = Node(10,
    Node(5, Node(3), Node(7)),
    Node(15, right=Node(20))
)
```

**In-order traversal (Left → Root → Right) — produces sorted order for BST:**

```python
def inorder(node):
    if node is None:
        return
    inorder(node.left)
    print(node.value, end=" ")
    inorder(node.right)

inorder(root)   # 3 5 7 10 15 20
```

**Pre-order (Root → Left → Right) — useful for copying/serialising:**

```python
def preorder(node):
    if node is None:
        return
    print(node.value, end=" ")
    preorder(node.left)
    preorder(node.right)

preorder(root)  # 10 5 3 7 15 20
```

**Post-order (Left → Right → Root) — useful for deletion:**

```python
def postorder(node):
    if node is None:
        return
    postorder(node.left)
    postorder(node.right)
    print(node.value, end=" ")

postorder(root)  # 3 7 5 20 15 10
```

**Collecting results instead of printing:**

```python
def inorder_list(node):
    if node is None:
        return []
    return inorder_list(node.left) + [node.value] + inorder_list(node.right)

print(inorder_list(root))  # [3, 5, 7, 10, 15, 20]
```

---

### 5.5 Merge Sort

A classic divide-and-conquer algorithm — O(n log n) guaranteed.

```python
def merge_sort(arr):
    if len(arr) <= 1:      # base case
        return arr

    mid   = len(arr) // 2
    left  = merge_sort(arr[:mid])    # sort left half
    right = merge_sort(arr[mid:])    # sort right half

    return merge(left, right)

def merge(left, right):
    result, i, j = [], 0, 0
    while i < len(left) and j < len(right):
        if left[i] <= right[j]:
            result.append(left[i]); i += 1
        else:
            result.append(right[j]); j += 1
    result.extend(left[i:])
    result.extend(right[j:])
    return result

data = [38, 27, 43, 3, 9, 82, 10]
print(merge_sort(data))
```

**Output:**
```
[3, 9, 10, 27, 38, 43, 82]
```

---

### 5.6 Power Function

```python
def power(base, exp):
    """Compute base^exp using fast exponentiation — O(log n)."""
    if exp == 0:
        return 1
    if exp % 2 == 0:
        half = power(base, exp // 2)
        return half * half             # square — avoids double recursion
    return base * power(base, exp - 1)

print(power(2, 10))   # 1024
print(power(3, 5))    # 243
```

---

### 5.7 Flatten Nested Lists

```python
def flatten(nested):
    """Recursively flatten arbitrarily nested lists."""
    result = []
    for item in nested:
        if isinstance(item, list):
            result.extend(flatten(item))   # recurse into sub-list
        else:
            result.append(item)
    return result

data = [1, [2, 3], [4, [5, [6, 7]]], 8]
print(flatten(data))
```

**Output:**
```
[1, 2, 3, 4, 5, 6, 7, 8]
```

---

## 6. Tail Recursion

### 6.1 What Is Tail Recursion?

A function is **tail-recursive** when the recursive call is the **very last operation** — the result of the recursive call is returned directly without any further computation.

**Non-tail-recursive factorial:**

```python
def factorial(n):
    if n == 0:
        return 1
    return n * factorial(n - 1)   # multiplication happens AFTER recursive call
```

**Tail-recursive factorial:**

```python
def factorial_tail(n, accumulator=1):
    if n == 0:
        return accumulator          # no work after return
    return factorial_tail(n - 1, accumulator * n)  # tail call
```

In tail-recursive form, the accumulator carries the result forward so the caller has **nothing left to do** after the recursive call returns.

---

### 6.2 Python's Limitations

Most languages with tail-call optimisation (TCO) — such as Haskell, Scheme, and Kotlin — replace the tail-recursive call with a jump (goto), preventing stack growth. **Python does NOT implement TCO**, by design (Guido van Rossum has explicitly stated this keeps tracebacks readable).

This means even a perfectly tail-recursive Python function will grow the stack and hit `RecursionError` for large inputs:

```python
factorial_tail(10000)  # RecursionError despite being tail-recursive
```

---

### 6.3 Simulating Tail Recursion

Convert to an iterative loop — the safest approach in Python:

```python
def factorial_iter(n):
    """Iterative equivalent of tail-recursive factorial."""
    acc = 1
    while n > 0:
        acc *= n
        n   -= 1
    return acc

print(factorial_iter(10000))  # Works fine — no stack growth
```

---

### 6.4 Trampolining

A **trampoline** is a higher-order function that repeatedly calls a returned thunk (a zero-argument lambda) instead of letting the stack grow:

```python
def trampoline(f, *args, **kwargs):
    result = f(*args, **kwargs)
    while callable(result):
        result = result()   # call the thunk
    return result

def factorial_trampoline(n, acc=1):
    if n == 0:
        return acc
    return lambda: factorial_trampoline(n - 1, acc * n)  # return a thunk

print(trampoline(factorial_trampoline, 10))    # 3628800
print(trampoline(factorial_trampoline, 1000))  # Works for large n
```

The trampoline loop never lets the recursive stack grow more than one frame deep.

---

## 7. Memoization

### 7.1 The Problem Without Memoization

```python
import time

def fib_naive(n):
    if n <= 1:
        return n
    return fib_naive(n - 1) + fib_naive(n - 2)

start = time.perf_counter()
print(fib_naive(35))
print(f"Time: {time.perf_counter() - start:.3f}s")
```

**Output (approximate):**
```
9227465
Time: 2.841s
```

`fib(35)` makes over 29 million function calls. `fib(5)` alone is called thousands of times. This is exponential — O(2ⁿ).

---

### 7.2 Manual Memoization with a Dictionary

```python
def fib_memo(n, cache={}):
    if n in cache:
        return cache[n]
    if n <= 1:
        return n
    cache[n] = fib_memo(n - 1, cache) + fib_memo(n - 2, cache)
    return cache[n]

start = time.perf_counter()
print(fib_memo(35))
print(f"Time: {time.perf_counter() - start:.6f}s")
```

**Output:**
```
9227465
Time: 0.000047s
```

> **Warning:** Using a mutable default argument (`cache={}`) persists between calls, which is the intended behaviour here but can cause subtle bugs in other contexts. Use `cache=None` and initialise inside the function for safety:

```python
def fib_safe(n, cache=None):
    if cache is None:
        cache = {}
    if n in cache:
        return cache[n]
    if n <= 1:
        return n
    cache[n] = fib_safe(n - 1, cache) + fib_safe(n - 2, cache)
    return cache[n]
```

---

### 7.3 `functools.lru_cache`

`lru_cache` (Least Recently Used cache) automatically caches function results. It accepts a `maxsize` parameter.

```python
from functools import lru_cache
import time

@lru_cache(maxsize=None)   # None = unlimited cache
def fib(n):
    if n <= 1:
        return n
    return fib(n - 1) + fib(n - 2)

start = time.perf_counter()
print(fib(50))
print(f"Time: {time.perf_counter() - start:.6f}s")
print(fib.cache_info())
```

**Output:**
```
12586269025
Time: 0.000031s
CacheInfo(hits=48, misses=51, maxsize=None, currsize=51)
```

**With bounded cache (LRU eviction):**

```python
@lru_cache(maxsize=128)
def expensive(n):
    return n ** 2

expensive(5)
expensive(5)       # cache hit
expensive.cache_clear()   # clear all cached results
```

**Requirements:** Arguments must be **hashable** (no lists or dicts as arguments).

---

### 7.4 `functools.cache`

Introduced in Python 3.9, `cache` is equivalent to `lru_cache(maxsize=None)` but slightly faster because it skips the LRU bookkeeping.

```python
from functools import cache

@cache
def fib(n):
    if n <= 1:
        return n
    return fib(n - 1) + fib(n - 2)

print(fib(100))   # 354224848179261915075
```

**Use `cache` when:**
- You want an unlimited cache.
- You do not need LRU eviction.
- You are on Python 3.9+.

**Use `lru_cache(maxsize=N)` when:**
- You want to bound memory usage.
- You are on Python 3.2+.

---

### 7.5 Performance Analysis

```python
from functools import cache
import time

def benchmark(func, n, label):
    start = time.perf_counter()
    result = func(n)
    elapsed = time.perf_counter() - start
    print(f"{label:30s}: fib({n}) = {result}, time = {elapsed:.6f}s")

def fib_naive(n):
    if n <= 1: return n
    return fib_naive(n - 1) + fib_naive(n - 2)

@cache
def fib_cached(n):
    if n <= 1: return n
    return fib_cached(n - 1) + fib_cached(n - 2)

benchmark(fib_naive,  30, "Naive recursion")
benchmark(fib_cached, 30, "Memoized (cache)")
benchmark(fib_cached, 200, "Memoized (cache) n=200")
```

**Output (approximate):**
```
Naive recursion               : fib(30) = 832040, time = 0.182340s
Memoized (cache)              : fib(30) = 832040, time = 0.000041s
Memoized (cache) n=200        : fib(200) = 28..., time = 0.000019s
```

| n | Naive calls | Memoized calls |
|---|------------|----------------|
| 10 | 177 | 19 |
| 20 | 21891 | 39 |
| 30 | 2692537 | 59 |
| 40 | 331160281 | 79 |

Memoization reduces time complexity from **O(2ⁿ)** to **O(n)**.

---

### 7.6 Cache Invalidation Strategies

**Time-based expiry:**

```python
import time
from functools import wraps

def timed_cache(seconds):
    def decorator(func):
        cache = {}

        @wraps(func)
        def wrapper(*args):
            now = time.time()
            if args in cache:
                result, timestamp = cache[args]
                if now - timestamp < seconds:
                    return result        # cache hit
            result = func(*args)
            cache[args] = (result, now)  # store with timestamp
            return result

        wrapper.cache_clear = lambda: cache.clear()
        return wrapper
    return decorator

@timed_cache(seconds=5)
def fetch_data(url):
    print(f"  [fetching {url}]")
    return f"data from {url}"

print(fetch_data("api/users"))   # [fetching api/users]
print(fetch_data("api/users"))   # cache hit (within 5s)
```

---

## 8. Comparison Tables

### 8.1 Loop vs Recursion

| Feature | Loop (Iterative) | Recursion |
|---------|-----------------|-----------|
| **Readability** | Clear for simple sequences | Clear for tree/graph problems |
| **Performance** | Generally faster (no call overhead) | Slower due to function call overhead |
| **Stack usage** | O(1) stack frames | O(depth) stack frames |
| **Risk of stack overflow** | None | Yes, for deep recursion |
| **State management** | Explicit (loop variables) | Implicit (call stack carries state) |
| **Debugging** | Easier to trace | Stack traces can be long |
| **Best for** | Flat iterations, known counts | Trees, graphs, divide-and-conquer |
| **Python limit** | None | ~1000 default depth |
| **Code length** | Often more lines | Often fewer lines |
| **Mathematical elegance** | Low | High (mirrors math definitions) |

---

### 8.2 Closure vs Class

| Feature | Closure | Class |
|---------|---------|-------|
| **Syntax** | `def outer(): ... return inner` | `class Foo: def __init__(self): ...` |
| **State** | Captured free variables | Instance attributes (`self.*`) |
| **Multiple methods** | Difficult (return a dict of functions) | Natural |
| **Inheritance** | Not supported | Supported |
| **Memory** | Slightly lighter | Slightly heavier |
| **Introspection** | Limited (`__closure__`) | Rich (`dir()`, `vars()`, `inspect`) |
| **Readability** | Good for single-purpose callables | Better for complex stateful objects |
| **Pickling** | Cannot pickle closures | Classes are picklable |
| **Use case** | Decorators, callbacks, factories | Data + behaviour, APIs, libraries |

**Example — counter as closure vs class:**

```python
# Closure
def make_counter():
    n = 0
    def increment():
        nonlocal n
        n += 1
        return n
    return increment

# Class
class Counter:
    def __init__(self):
        self.n = 0
    def increment(self):
        self.n += 1
        return self.n
```

---

### 8.3 Normal Recursion vs Memoized Recursion

| Feature | Normal Recursion | Memoized Recursion |
|---------|-----------------|-------------------|
| **Time complexity (Fibonacci)** | O(2ⁿ) | O(n) |
| **Space complexity** | O(n) call stack | O(n) cache + O(n) stack |
| **Repeated subproblems** | Recomputed every time | Computed once, cached |
| **Code complexity** | Simple | Slight overhead (decorator) |
| **Suitable for overlapping subproblems** | No | Yes |
| **Suitable for non-overlapping problems** | Yes | Overkill (no benefit) |
| **Memory use** | Call stack only | Call stack + cache dictionary |
| **Python decorator** | None | `@cache` or `@lru_cache` |
| **Debugging** | Easier | Cache hits hide individual calls |
| **Large inputs** | Fails / very slow | Handles large inputs efficiently |

---

## 9. Real-World Projects

### 9.1 Recursive Directory Scanner

Recursively walk a directory tree, collecting files by extension.

```python
import os
from pathlib import Path

def scan_directory(root, extensions=None, depth=0, max_depth=10):
    """
    Recursively scan a directory and return a dict of
    {extension: [file_paths]} groupings.
    
    Parameters
    ----------
    root       : str or Path — directory to scan
    extensions : set of str or None — e.g. {'.py', '.txt'}; None = all files
    depth      : int — current recursion depth (internal)
    max_depth  : int — maximum recursion depth
    """
    root = Path(root)
    results = {}

    if depth > max_depth:
        return results   # base case: depth limit

    try:
        entries = list(root.iterdir())
    except PermissionError:
        return results   # base case: cannot access directory

    for entry in entries:
        if entry.is_dir():
            # Recurse into subdirectory
            sub_results = scan_directory(entry, extensions, depth + 1, max_depth)
            for ext, paths in sub_results.items():
                results.setdefault(ext, []).extend(paths)

        elif entry.is_file():
            ext = entry.suffix.lower()
            if extensions is None or ext in extensions:
                results.setdefault(ext, []).append(str(entry))

    return results


def print_scan_report(root, extensions=None):
    print(f"Scanning: {root}")
    results = scan_directory(root, extensions)
    total = sum(len(v) for v in results.values())
    print(f"Total files: {total}")
    for ext, paths in sorted(results.items()):
        print(f"  {ext or '(no ext)':12s}: {len(paths)} file(s)")
        for p in paths[:3]:
            print(f"      {p}")
        if len(paths) > 3:
            print(f"      ... and {len(paths) - 3} more")


# Usage:
# print_scan_report("/path/to/project", extensions={'.py', '.md'})
```

---

### 9.2 Expression Evaluator

A recursive descent evaluator for simple arithmetic expressions.

```python
class ExpressionEvaluator:
    """
    Evaluates arithmetic expressions like "3 + 5 * (2 - 8)".
    
    Grammar:
        expr   ::= term (('+' | '-') term)*
        term   ::= factor (('*' | '/') factor)*
        factor ::= NUMBER | '(' expr ')'
    """

    def __init__(self, text):
        self.text = text.replace(" ", "")
        self.pos  = 0

    def peek(self):
        return self.text[self.pos] if self.pos < len(self.text) else None

    def consume(self, expected):
        if self.peek() != expected:
            raise SyntaxError(f"Expected '{expected}', got '{self.peek()}'")
        self.pos += 1

    def number(self):
        start = self.pos
        if self.peek() == '-':
            self.pos += 1
        while self.peek() and self.peek().isdigit():
            self.pos += 1
        return float(self.text[start:self.pos])

    def factor(self):
        if self.peek() == '(':
            self.consume('(')
            result = self.expr()   # recursive call
            self.consume(')')
            return result
        return self.number()

    def term(self):
        result = self.factor()
        while self.peek() in ('*', '/'):
            op = self.peek(); self.pos += 1
            if op == '*': result *= self.factor()
            else:         result /= self.factor()
        return result

    def expr(self):
        result = self.term()
        while self.peek() in ('+', '-'):
            op = self.peek(); self.pos += 1
            if op == '+': result += self.term()
            else:         result -= self.term()
        return result

    def evaluate(self):
        return self.expr()


def evaluate(expression):
    return ExpressionEvaluator(expression).evaluate()

print(evaluate("3 + 5"))           # 8.0
print(evaluate("3 + 5 * 2"))       # 13.0
print(evaluate("(3 + 5) * 2"))     # 16.0
print(evaluate("10 / (2 + 3)"))    # 2.0
print(evaluate("2 * (3 + (4 - 1))")) # 12.0
```

**Output:**
```
8.0
13.0
16.0
2.0
12.0
```

---

### 9.3 Configuration Closures

Use closures to create pre-configured functions from a shared configuration:

```python
def make_api_client(base_url, api_key, timeout=30):
    """
    Returns a set of closures that are pre-configured
    with connection parameters.
    """
    headers = {
        "Authorization": f"Bearer {api_key}",
        "Content-Type": "application/json",
    }

    def get(endpoint, params=None):
        url = f"{base_url}/{endpoint.lstrip('/')}"
        print(f"GET {url} | params={params} | headers={headers} | timeout={timeout}")
        # In production: return requests.get(url, params=params, headers=headers, timeout=timeout)

    def post(endpoint, data=None):
        url = f"{base_url}/{endpoint.lstrip('/')}"
        print(f"POST {url} | data={data} | headers={headers} | timeout={timeout}")
        # In production: return requests.post(url, json=data, headers=headers, timeout=timeout)

    def delete(endpoint):
        url = f"{base_url}/{endpoint.lstrip('/')}"
        print(f"DELETE {url} | headers={headers}")

    return get, post, delete


# Create two pre-configured API clients
prod_get, prod_post, prod_delete = make_api_client(
    "https://api.example.com", "prod-key-xyz", timeout=60
)
dev_get, dev_post, _ = make_api_client(
    "https://dev.example.com", "dev-key-abc", timeout=5
)

prod_get("users", params={"page": 1})
dev_post("users", data={"name": "Alice"})
```

**Output:**
```
GET https://api.example.com/users | params={'page': 1} | headers={...} | timeout=60
POST https://dev.example.com/users | data={'name': 'Alice'} | headers={...} | timeout=5
```

---

### 9.4 Caching System

A feature-rich caching system built with closures:

```python
import time
import hashlib
import json
from functools import wraps

def make_cache(max_size=256, ttl_seconds=None):
    """
    Returns a decorator that caches function results.
    Supports max_size (LRU eviction) and optional TTL.
    """
    store = {}         # {key: (value, timestamp, access_time)}
    access_order = []  # for LRU tracking

    def _evict_lru():
        if len(store) >= max_size:
            lru_key = access_order.pop(0)
            del store[lru_key]

    def _make_key(args, kwargs):
        raw = json.dumps({"args": args, "kwargs": kwargs}, sort_keys=True, default=str)
        return hashlib.md5(raw.encode()).hexdigest()

    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            key = _make_key(args, kwargs)
            now = time.time()

            if key in store:
                value, created_at, _ = store[key]
                if ttl_seconds is None or (now - created_at) < ttl_seconds:
                    store[key] = (value, created_at, now)
                    if key in access_order:
                        access_order.remove(key)
                    access_order.append(key)
                    return value
                else:
                    del store[key]            # expired
                    access_order.remove(key)

            _evict_lru()
            result = func(*args, **kwargs)
            store[key] = (result, now, now)
            access_order.append(key)
            return result

        wrapper.cache_info = lambda: {
            "size": len(store), "max_size": max_size, "ttl": ttl_seconds
        }
        wrapper.cache_clear = lambda: (store.clear(), access_order.clear())
        return wrapper

    return decorator


@make_cache(max_size=100, ttl_seconds=10)
def slow_computation(n):
    time.sleep(0.1)   # simulate expensive work
    return n * n

print(slow_computation(5))         # computes: 0.1s
print(slow_computation(5))         # cached: instant
print(slow_computation.cache_info()) # {'size': 1, 'max_size': 100, 'ttl': 10}
```

---

## 10. Cheat Sheets

### 10.1 Closures Cheat Sheet

```
┌─────────────────────────────────────────────────────────────────┐
│                    CLOSURES CHEAT SHEET                         │
├─────────────────────────────────────────────────────────────────┤
│ ANATOMY OF A CLOSURE                                            │
│                                                                 │
│   def outer(x):          # enclosing function                  │
│       def inner(y):      # nested function                     │
│           return x + y   # x is a free variable (captured)    │
│       return inner       # return function, not result         │
│                                                                 │
│   add5 = outer(5)        # closure: inner + {x: 5}            │
│   add5(3)  → 8                                                  │
├─────────────────────────────────────────────────────────────────┤
│ MUTATING CAPTURED VARIABLES                                     │
│                                                                 │
│   def counter():                                                │
│       n = 0                                                     │
│       def inc():                                                │
│           nonlocal n     # ← required for assignment           │
│           n += 1                                                │
│           return n                                              │
│       return inc                                                │
├─────────────────────────────────────────────────────────────────┤
│ LOOP CAPTURE GOTCHA                                             │
│                                                                 │
│   # BUGGY: all functions see final value of i                  │
│   fns = [lambda x: x * i for i in range(3)]                   │
│                                                                 │
│   # FIXED: capture current value with default arg              │
│   fns = [lambda x, i=i: x * i for i in range(3)]              │
├─────────────────────────────────────────────────────────────────┤
│ INSPECT A CLOSURE                                               │
│                                                                 │
│   f.__closure__                   # tuple of cell objects      │
│   f.__closure__[0].cell_contents  # value of captured var      │
│   f.__code__.co_freevars          # names of free variables    │
└─────────────────────────────────────────────────────────────────┘
```

---

### 10.2 Recursion Cheat Sheet

```
┌─────────────────────────────────────────────────────────────────┐
│                    RECURSION CHEAT SHEET                        │
├─────────────────────────────────────────────────────────────────┤
│ TEMPLATE                                                        │
│                                                                 │
│   def solve(input):                                             │
│       if is_base_case(input):   # 1. base case                 │
│           return base_value                                     │
│       smaller = reduce(input)   # 2. reduce                    │
│       sub = solve(smaller)      # 3. recurse                   │
│       return combine(input,sub) # 4. combine                   │
├─────────────────────────────────────────────────────────────────┤
│ COMMON ALGORITHMS                                               │
│                                                                 │
│   Factorial  : f(n) = n * f(n-1)     base: f(0)=1             │
│   Fibonacci  : f(n) = f(n-1)+f(n-2)  base: f(0)=0,f(1)=1     │
│   Binary Srch: halve search space    base: low>high or found   │
│   Tree walk  : visit node, recurse   base: node is None        │
│   Merge sort : split, sort, merge    base: len<=1              │
├─────────────────────────────────────────────────────────────────┤
│ PYTHON LIMITS                                                   │
│                                                                 │
│   import sys                                                    │
│   sys.getrecursionlimit()   # default ~1000                    │
│   sys.setrecursionlimit(N)  # increase (use carefully)         │
├─────────────────────────────────────────────────────────────────┤
│ TRAMPOLINE (for deep recursion)                                 │
│                                                                 │
│   def trampoline(f, *args):                                     │
│       r = f(*args)                                              │
│       while callable(r): r = r()                               │
│       return r                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

### 10.3 Memoization Cheat Sheet

```
┌─────────────────────────────────────────────────────────────────┐
│                  MEMOIZATION CHEAT SHEET                        │
├─────────────────────────────────────────────────────────────────┤
│ QUICK COMPARISON                                                │
│                                                                 │
│   @cache              # Python 3.9+ — unlimited, fast          │
│   @lru_cache(None)    # unlimited (any Python 3.x)             │
│   @lru_cache(128)     # bounded — evicts LRU entries           │
├─────────────────────────────────────────────────────────────────┤
│ USAGE                                                           │
│                                                                 │
│   from functools import cache, lru_cache                       │
│                                                                 │
│   @cache                                                        │
│   def fib(n):                                                   │
│       if n <= 1: return n                                       │
│       return fib(n-1) + fib(n-2)                               │
│                                                                 │
│   fib(50)                  # instant                           │
│   fib.cache_info()         # hits, misses, maxsize, currsize   │
│   fib.cache_clear()        # wipe the cache                    │
├─────────────────────────────────────────────────────────────────┤
│ REQUIREMENTS                                                    │
│                                                                 │
│   • Arguments must be HASHABLE (no lists/dicts)                │
│   • Use tuple(lst) to convert lists before passing             │
│   • Does NOT work with methods that take 'self' directly       │
│     unless 'self' is hashable                                   │
├─────────────────────────────────────────────────────────────────┤
│ COMPLEXITY                                                      │
│                                                                 │
│   Fibonacci without memoization: O(2^n)                        │
│   Fibonacci with    memoization: O(n) time, O(n) space         │
└─────────────────────────────────────────────────────────────────┘
```

---

## 11. Interview Master Section

### 11.1 Closure Questions

**Q1. What is a closure? State three conditions for a closure to exist.**

A closure is a function that captures and remembers variables from its enclosing scope even after the outer function has returned. Three conditions: (1) there is a nested function, (2) the nested function references a variable from the enclosing scope, (3) the enclosing function returns the nested function.

**Q2. What is the output of the following code?**

```python
def make_adders():
    adders = []
    for n in range(3):
        adders.append(lambda x: x + n)
    return adders

f0, f1, f2 = make_adders()
print(f0(10), f1(10), f2(10))
```

**Answer:** `12 12 12` — all three lambdas capture the *reference* to `n`, which is `2` after the loop ends. Fix: `lambda x, n=n: x + n`.

**Q3. How would you fix the loop capture bug without using a default argument?**

Wrap in another function: `make_adder = lambda n: lambda x: x + n`, then `adders.append(make_adder(n))`.

**Q4. What does `__closure__` contain and how do you inspect it?**

It contains a tuple of `cell` objects. Each cell holds one captured variable. Access with `f.__closure__[i].cell_contents`. The names are in `f.__code__.co_freevars`.

**Q5. When would you prefer a closure over a class?**

When you need a single callable with private state, and you do not need inheritance, multiple methods, pickling, or introspection. Closures are lighter-weight and more concise for simple use cases like decorators and callback factories.

**Q6. Can a closure modify the enclosing variable? How?**

Yes, using `nonlocal`. Without it, assignment in the inner function creates a new local variable, masking the outer one and raising `UnboundLocalError` if you try to read before assigning.

**Q7. Explain why closures are important for decorators.**

A decorator wraps a function. The wrapper must remember the original function (a free variable). The original function is captured in the wrapper's closure, allowing the wrapper to call it while adding extra behaviour.

**Q8. What is a free variable in the context of closures?**

A free variable is a variable referenced inside a function that is neither defined in that function (local) nor is a global — it lives in an enclosing scope and is "captured" by the closure.

**Q9. What will this print?**

```python
def outer():
    x = 10
    def inner():
        print(x)
    x = 20
    return inner

f = outer()
f()
```

**Answer:** `20`. The closure captures a **reference** to `x`, not a snapshot. By the time `f()` is called, `x` has been updated to `20`.

**Q10. How can you create a closure with multiple independent states?**

Call the factory function multiple times. Each call creates a new closure with its own captured variables: `c1 = make_counter(); c2 = make_counter()` — `c1` and `c2` have independent state.

---

### 11.2 Recursion Questions

**Q11. What are the two essential components of a recursive function?**

(1) A **base case** that stops the recursion. (2) A **recursive case** that reduces the problem and calls the function on the smaller input, moving toward the base case.

**Q12. What happens without a base case?**

The function calls itself indefinitely until Python raises `RecursionError: maximum recursion depth exceeded`.

**Q13. What is the default recursion limit in Python and how do you change it?**

Default is typically 1000. Change with `sys.setrecursionlimit(N)`. This should be done with care — deep recursion can exhaust system memory.

**Q14. What is the time complexity of naive recursive Fibonacci?**

O(2ⁿ) — exponential. Each call spawns two more calls, and many subproblems are recomputed.

**Q15. What is the space complexity of recursive Fibonacci (naive)?**

O(n) — the maximum depth of the call stack is n.

**Q16. Implement recursive binary search.**

```python
def binary_search(arr, target, lo=0, hi=None):
    if hi is None: hi = len(arr) - 1
    if lo > hi: return -1
    mid = (lo + hi) // 2
    if arr[mid] == target: return mid
    if arr[mid] < target: return binary_search(arr, target, mid+1, hi)
    return binary_search(arr, target, lo, mid-1)
```

**Q17. When is recursion a better choice than iteration?**

When the problem has a naturally recursive structure — trees, graphs, divide-and-conquer, parsing, backtracking, and problems defined by mathematical induction. The recursive version is typically more readable and closer to the problem definition.

**Q18. Write a recursive function to compute the sum of digits of a number.**

```python
def digit_sum(n):
    n = abs(n)
    if n < 10: return n
    return n % 10 + digit_sum(n // 10)
```

**Q19. Explain the call stack with reference to recursion.**

Each function call pushes a frame onto the stack containing local variables and the return address. Recursive calls stack up until the base case is hit, then frames are popped (unwound) in reverse order as return values propagate up.

**Q20. How would you reverse a string recursively?**

```python
def reverse(s):
    if len(s) <= 1: return s
    return reverse(s[1:]) + s[0]
```

---

### 11.3 Memoization Questions

**Q21. What is memoization and how does it improve performance?**

Memoization caches the result of a function call indexed by its arguments. On subsequent calls with the same arguments, the cached result is returned immediately without recomputation. It trades space for time, reducing exponential algorithms to polynomial.

**Q22. What is the difference between `@cache` and `@lru_cache(maxsize=128)`?**

`@cache` (Python 3.9+) is an unbounded cache with no eviction. `@lru_cache(maxsize=128)` evicts the least-recently-used entry when the cache exceeds 128 entries, bounding memory usage.

**Q23. What types of arguments can be passed to a memoized function?**

Only **hashable** types: integers, floats, strings, tuples, frozensets, `None`. Lists, dicts, and sets are not hashable and will raise `TypeError`. Convert with `tuple(lst)` before passing.

**Q24. What does `cache_info()` return?**

A named tuple: `CacheInfo(hits, misses, maxsize, currsize)`. Hits = how many times a cached value was returned. Misses = how many times the function was actually called.

**Q25. Can you memoize a function that takes a list argument?**

Not directly (lists are not hashable). Work around it by converting to a tuple inside a wrapper:

```python
@cache
def process(items_tuple):
    ...

def process_list(lst):
    return process(tuple(lst))
```

---

### 11.4 Advanced Questions

**Q26. What is tail recursion? Does Python optimise it?**

Tail recursion is when the recursive call is the last operation in the function — nothing is done with the result after the call. Python does **not** perform tail-call optimisation (TCO). Every call, including tail calls, creates a new stack frame.

**Q27. What is trampolining and when would you use it in Python?**

Trampolining is a technique to simulate TCO: instead of calling itself, the function returns a thunk (a zero-argument callable). A trampoline driver loop calls the thunk repeatedly until a non-callable is returned. Use it when you need deep recursion without risking stack overflow.

**Q28. Implement a memoized recursive version of `nCr` (combinations).**

```python
from functools import cache

@cache
def nCr(n, r):
    if r == 0 or r == n: return 1
    return nCr(n-1, r-1) + nCr(n-1, r)
```

**Q29. What is dynamic programming and how does it relate to memoization?**

Dynamic programming (DP) solves optimisation problems by breaking them into overlapping subproblems. Memoization (top-down DP) is one approach; tabulation (bottom-up DP) is the other. Memoization is recursive + cached; tabulation fills a table iteratively.

**Q30. Explain mutual recursion. Provide an example.**

Two or more functions that call each other:

```python
def is_even(n):
    if n == 0: return True
    return is_odd(n - 1)

def is_odd(n):
    if n == 0: return False
    return is_even(n - 1)
```

**Q31. How would you detect if a closure is leaking references to large objects?**

Inspect `f.__closure__` — each cell object holds a reference that keeps the object alive. If a closure captures a large object it no longer needs, that object will not be garbage collected. Solutions: capture only what you need, or capture a smaller derived value.

**Q32. What is the difference between memoization and caching in general?**

Memoization is specifically about caching the return value of a **pure function** based on its arguments. Caching is a broader concept that includes time-based expiry, cache invalidation, distributed caches, HTTP caching, etc.

**Q33. Write a recursive function to generate all permutations of a list.**

```python
def permutations(lst):
    if len(lst) <= 1: return [lst]
    result = []
    for i, elem in enumerate(lst):
        rest = lst[:i] + lst[i+1:]
        for perm in permutations(rest):
            result.append([elem] + perm)
    return result
```

**Q34. What is the Tower of Hanoi problem? Solve it recursively.**

Move n disks from source to destination using an auxiliary peg, without placing a larger disk on a smaller one:

```python
def hanoi(n, source, target, auxiliary):
    if n == 1:
        print(f"Move disk 1 from {source} to {target}")
        return
    hanoi(n-1, source, auxiliary, target)
    print(f"Move disk {n} from {source} to {target}")
    hanoi(n-1, auxiliary, target, source)
```

**Q35. How would you convert a recursive function to use an explicit stack?**

Replace the call stack with a list used as a stack. Push tuples of (arguments, state) and pop them in a loop. This is especially useful for depth-first traversal:

```python
def dfs_iterative(root):
    if root is None: return
    stack = [root]
    while stack:
        node = stack.pop()
        print(node.value)
        if node.right: stack.append(node.right)
        if node.left:  stack.append(node.left)
```

---

## 12. Exercises

### 12.1 Intermediate (15 Problems)

**Problem 1.** Write a closure that returns a function to check if a number is divisible by a given divisor.

**Problem 2.** Create a closure-based `make_between(lo, hi)` that returns a function checking whether a value is in range `[lo, hi]`.

**Problem 3.** Write a recursive function to compute the GCD (greatest common divisor) of two numbers using Euclid's algorithm.

**Problem 4.** Implement a recursive function that counts the number of occurrences of a value in a nested list.

**Problem 5.** Create a closure `make_logger(prefix)` that returns a logging function that prepends the prefix to every message.

**Problem 6.** Write a recursive function to check if a string is a palindrome.

**Problem 7.** Implement a recursive function to compute the sum of all numbers in a nested list.

**Problem 8.** Use `nonlocal` to implement a running average closure that updates with each new number.

**Problem 9.** Write a recursive function that converts a decimal integer to binary (return as string).

**Problem 10.** Create a `make_validator(*validators)` closure that returns a function applying all validators to a value.

**Problem 11.** Write a recursive function that returns the depth (height) of a binary tree.

**Problem 12.** Implement `@lru_cache` on a function computing the number of ways to climb n stairs (taking 1 or 2 steps).

**Problem 13.** Write a closure-based retry mechanism: `make_retry(max_attempts)` returns a decorator that retries on exception.

**Problem 14.** Implement recursive bubble sort.

**Problem 15.** Write a `make_pipe(*funcs)` closure that composes functions left to right (output of one feeds the next).

---

### 12.2 Advanced (15 Problems)

**Problem 16.** Implement a memoized recursive coin change function: minimum number of coins to make amount `n` from a list of denominations.

**Problem 17.** Create a closure-based event system: `make_event()` returns `(subscribe, publish)` closures sharing an internal list of listeners.

**Problem 18.** Write a recursive function to solve the N-Queens problem and return all valid board configurations.

**Problem 19.** Implement a closure that acts as a simple state machine: takes a list of transitions `{state: {event: next_state}}` and returns a `trigger(event)` closure.

**Problem 20.** Write a memoized recursive solution for the "0/1 knapsack" problem.

**Problem 21.** Implement a recursive JSON pretty-printer (no `json.dumps`).

**Problem 22.** Create `make_once(func)`: a closure-based decorator ensuring `func` is called at most once; subsequent calls return the first result.

**Problem 23.** Implement a recursive function to generate all valid parentheses combinations for `n` pairs.

**Problem 24.** Write a trampoline-based tail-recursive Fibonacci.

**Problem 25.** Implement a `make_debounce(delay)` closure factory: the returned function delays execution and resets the timer on each call.

**Problem 26.** Write a recursive quick-sort implementation.

**Problem 27.** Implement a closure that tracks how many times a function has been called, how long it took in total, and exposes a `stats()` method.

**Problem 28.** Write a recursive function to evaluate a nested arithmetic expression represented as a Python tuple, e.g. `('+', 3, ('*', 4, 5))`.

**Problem 29.** Implement a recursive Sudoku solver.

**Problem 30.** Create a `memoize_method` decorator that correctly handles instance methods (does not pollute the class-level cache across instances).

---

### 12.3 Expert (15 Problems)

**Problem 31.** Implement a Y combinator in Python and use it to define factorial without naming the function inside itself.

**Problem 32.** Build a recursive descent parser for a simple boolean expression language: variables, `AND`, `OR`, `NOT`, and parentheses.

**Problem 33.** Implement a persistent (immutable) recursive data structure: a functional linked list with `head`, `tail`, `cons`, and `to_list` operations.

**Problem 34.** Write a closure-based dependency injection container that lazily resolves dependencies.

**Problem 35.** Implement the Ackermann function and use memoization to handle its explosive growth.

**Problem 36.** Create a generic `memoize` decorator that handles unhashable arguments by serialising them to a canonical JSON key.

**Problem 37.** Implement a recursive regex engine supporting `.`, `*`, and `+` operators.

**Problem 38.** Build a recursive interpreter for a tiny Lisp: `(define x 5)`, `(lambda (x) body)`, `(if cond t f)`, `(+ a b)`.

**Problem 39.** Write a recursive topological sort (DFS-based) for a directed acyclic graph.

**Problem 40.** Implement a `lazy_cache` closure decorator where results are computed only when first requested, stored, and never recomputed — thread-safe using a lock.

**Problem 41.** Create a recursive diff function comparing two nested dicts and returning a structured change log (added, removed, modified).

**Problem 42.** Implement `itertools.groupby` using closures and no imports.

**Problem 43.** Write a recursive program to solve the "word break" problem with memoization.

**Problem 44.** Implement a closure-based pipeline with error handling: each stage is a function; if a stage raises, the error closure is called with the stage name and exception.

**Problem 45.** Build a recursive minimax algorithm for tic-tac-toe with alpha-beta pruning.

---

### 12.4 Solutions

#### Solution to Problem 1 — Divisibility Closure

```python
def make_divisible_by(divisor):
    def check(n):
        return n % divisor == 0
    return check

is_even     = make_divisible_by(2)
is_multiple_of_5 = make_divisible_by(5)

print(is_even(10))           # True
print(is_even(7))            # False
print(is_multiple_of_5(25))  # True
```

#### Solution to Problem 3 — GCD

```python
def gcd(a, b):
    """Euclid's algorithm: gcd(a, 0) = a; gcd(a, b) = gcd(b, a % b)"""
    if b == 0:
        return a
    return gcd(b, a % b)

print(gcd(48, 18))   # 6
print(gcd(100, 75))  # 25
```

#### Solution to Problem 6 — Palindrome

```python
def is_palindrome(s):
    s = s.lower().replace(" ", "")
    if len(s) <= 1:
        return True
    if s[0] != s[-1]:
        return False
    return is_palindrome(s[1:-1])

print(is_palindrome("racecar"))    # True
print(is_palindrome("A man a plan a canal Panama"))  # True
print(is_palindrome("hello"))      # False
```

#### Solution to Problem 9 — Decimal to Binary

```python
def to_binary(n):
    if n == 0:
        return "0"
    if n == 1:
        return "1"
    return to_binary(n // 2) + str(n % 2)

print(to_binary(10))   # 1010
print(to_binary(255))  # 11111111
```

#### Solution to Problem 12 — Stair Climbing with `lru_cache`

```python
from functools import lru_cache

@lru_cache(maxsize=None)
def climb_stairs(n):
    """Number of ways to climb n stairs taking 1 or 2 steps."""
    if n <= 1: return 1
    if n == 2: return 2
    return climb_stairs(n - 1) + climb_stairs(n - 2)

for i in range(1, 11):
    print(f"climb_stairs({i}) = {climb_stairs(i)}")
```

**Output:**
```
climb_stairs(1) = 1
climb_stairs(2) = 2
climb_stairs(3) = 3
climb_stairs(4) = 5
climb_stairs(5) = 8
...
```

#### Solution to Problem 16 — Coin Change (Memoized)

```python
from functools import cache

def min_coins(denominations, amount):
    @cache
    def dp(remaining):
        if remaining == 0: return 0
        if remaining < 0: return float('inf')
        return 1 + min(dp(remaining - coin) for coin in denominations)

    result = dp(amount)
    return result if result != float('inf') else -1

print(min_coins((1, 5, 10, 25), 41))   # 4 (25+10+5+1)
print(min_coins((2,), 3))              # -1 (impossible)
```

#### Solution to Problem 22 — `make_once`

```python
def make_once(func):
    result = [None]
    called = [False]

    def wrapper(*args, **kwargs):
        if not called[0]:
            result[0] = func(*args, **kwargs)
            called[0] = True
        return result[0]

    return wrapper

@make_once
def expensive_setup():
    print("Setting up...")
    return 42

print(expensive_setup())  # Setting up... 42
print(expensive_setup())  # 42 (no "Setting up")
```

#### Solution to Problem 26 — Recursive Quick Sort

```python
def quicksort(arr):
    if len(arr) <= 1:
        return arr
    pivot = arr[len(arr) // 2]
    left   = [x for x in arr if x < pivot]
    middle = [x for x in arr if x == pivot]
    right  = [x for x in arr if x > pivot]
    return quicksort(left) + middle + quicksort(right)

print(quicksort([3, 6, 8, 10, 1, 2, 1]))
```

**Output:**
```
[1, 1, 2, 3, 6, 8, 10]
```

#### Solution to Problem 31 — Y Combinator

```python
# The Y combinator in Python (applicative-order / call-by-value form)
Y = lambda f: (lambda x: f(lambda v: x(x)(v)))(lambda x: f(lambda v: x(x)(v)))

factorial = Y(lambda self: lambda n: 1 if n == 0 else n * self(n - 1))

print(factorial(5))   # 120
print(factorial(10))  # 3628800
```

#### Solution to Problem 39 — Topological Sort

```python
def topological_sort(graph):
    """
    graph: dict {node: [neighbours]}
    Returns a list of nodes in topological order.
    """
    visited = set()
    order   = []

    def dfs(node):
        if node in visited:
            return
        visited.add(node)
        for neighbour in graph.get(node, []):
            dfs(neighbour)
        order.append(node)   # add after all descendants

    for node in graph:
        dfs(node)

    return order[::-1]   # reverse post-order = topological order

graph = {
    "A": ["C"],
    "B": ["C", "D"],
    "C": ["E"],
    "D": ["F"],
    "E": [],
    "F": [],
}
print(topological_sort(graph))   # e.g. ['B', 'A', 'D', 'C', 'F', 'E']
```

---

## 13. Final Section

### 13.1 Summary

This guide covered the full spectrum of Python's closure and recursion capabilities:

**Nested Functions** are the building block — functions defined within functions, with access to the enclosing scope via the LEGB rule.

**Closures** extend nested functions by remembering captured variables after the outer function returns. They enable stateful callables, decorator factories, and partial application — all without the overhead of a class.

**`nonlocal`** bridges the gap between read-only access to enclosing variables and full mutation, making closures truly stateful in a clean, explicit way.

**Recursion** is the art of solving problems by reducing them to simpler versions of themselves. Every recursive solution needs a base case (termination) and a recursive case (progress).

**Recursive Algorithms** — factorial, Fibonacci, binary search, tree traversal, merge sort — are classic computer science problems where recursion is the natural expression of the solution.

**Tail Recursion** is an optimisation concept relevant mainly in functional languages. Python does not support it natively, but trampolining offers a workaround for deep recursive needs.

**Memoization** transforms exponential recursive algorithms into polynomial ones by caching results. Python's `functools.cache` and `lru_cache` make this a one-decorator affair.

---

### 13.2 Best Practices

**Closures:**

- Always use `nonlocal` explicitly when assigning to enclosing variables.
- Prefer `n=n` default argument or `functools.partial` to fix loop-capture bugs.
- Keep closures small and focused on a single responsibility.
- Document what variables a closure captures, especially in long-lived objects.
- Do not capture large objects unnecessarily — this prevents garbage collection.

**Recursion:**

- Always define the base case first.
- Ensure every recursive call moves strictly closer to the base case.
- Add a `depth` parameter and a `max_depth` check for production code dealing with untrusted input.
- For problems with depth > 500, prefer iteration or increase the recursion limit judiciously.
- Add meaningful docstrings explaining the base case and what the function computes.

**Memoization:**

- Use `@cache` for pure functions with hashable arguments on Python 3.9+.
- Use `@lru_cache(maxsize=N)` when memory is a concern.
- Call `cache_clear()` when cached data may be stale.
- Convert unhashable inputs (lists → tuples) before passing to memoized functions.
- Do not memoize functions with side effects.

---

### 13.3 Common Pitfalls

| Pitfall | Description | Fix |
|---------|-------------|-----|
| **Loop capture bug** | Lambda/closure in a loop captures reference to loop variable, not its value | Use `n=n` default or `partial` |
| **Missing base case** | Recursion never terminates | Always define and test the base case first |
| **Wrong base case** | Base case is too narrow or returns incorrect value | Trace through small examples manually |
| **Stack overflow** | Deep recursion exceeds Python's limit | Convert to iteration or use memoization to reduce depth |
| **Memoizing impure functions** | Functions with side effects behave incorrectly when cached | Only memoize pure functions |
| **Unhashable arguments** | Passing lists/dicts to `@cache` raises TypeError | Convert to tuple/frozenset first |
| **nonlocal on global** | Using `nonlocal` for a module-level variable | Use `global` instead |
| **Shared mutable default** | `cache={}` as default arg shared across calls | Use `cache=None` and initialise inside |
| **Missing return in recursion** | Forgetting to return the recursive call result | Check every code path returns a value |
| **Over-caching** | Caching functions that take mutable or session-specific args | Only cache where pure, deterministic, expensive |

---

### 13.4 Learning Roadmap

```
BEGINNER
│
├── Understand Python scoping (LEGB rule)
├── Write simple nested functions
├── Understand basic recursion with factorial and Fibonacci
├── Implement binary search recursively
│
▼
INTERMEDIATE
│
├── Build closures with nonlocal state
├── Use closures for decorators and callbacks
├── Implement recursive sorting algorithms (merge sort, quick sort)
├── Apply @lru_cache to recursive problems
├── Traverse binary trees recursively
│
▼
ADVANCED
│
├── Build closure-based design patterns (factory, strategy, observer)
├── Implement dynamic programming solutions with memoization
├── Write recursive parsers (expression evaluators, JSON parsers)
├── Understand and implement trampolining
├── Solve classic DP problems (knapsack, coin change, edit distance)
│
▼
EXPERT
│
├── Implement Y combinator and combinators
├── Build recursive descent parsers for real languages
├── Implement persistent recursive data structures
├── Write recursive backtracking solvers (Sudoku, N-Queens)
├── Contribute to open-source projects using advanced closures
├── Study functional programming patterns (functor, monad) in Python
│
▼
MASTERY
│
├── Design APIs with closure-based DSLs
├── Write memoized graph algorithms for production
├── Profile and optimise recursive code with caching strategies
└── Teach and mentor others on closures and recursion
```

---

## Quick-Reference Index

| Topic | Section |
|-------|---------|
| LEGB scope rule | 1.2 |
| Closure definition | 2.1 |
| Loop capture bug | 2.2 |
| State preservation | 2.3 |
| Inspecting closures | 2.4 |
| `nonlocal` keyword | 3 |
| `nonlocal` vs `global` | 3.4 |
| Recursion template | 4.3 |
| Call stack visualisation | 4.4 |
| Recursion limit | 4.5 |
| Factorial | 5.1 |
| Fibonacci | 5.2 |
| Binary search | 5.3 |
| Tree traversal | 5.4 |
| Merge sort | 5.5 |
| Tail recursion | 6.1 |
| Python TCO limitation | 6.2 |
| Trampolining | 6.4 |
| `@cache` | 7.4 |
| `@lru_cache` | 7.3 |
| Performance comparison | 7.5 |
| Loop vs Recursion table | 8.1 |
| Closure vs Class table | 8.2 |
| Normal vs Memoized table | 8.3 |
| Directory scanner project | 9.1 |
| Expression evaluator project | 9.2 |
| Configuration closures project | 9.3 |
| Caching system project | 9.4 |
| 35 interview questions | 11 |
| 45 exercises with solutions | 12 |

---

*End of Python Closures and Recursion Master Guide*
