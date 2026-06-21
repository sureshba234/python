# Advanced Python Function Parameters Master Guide

> A complete reference from beginner to expert — covering every dimension of Python functions, parameters, and functional programming patterns.

---

## Table of Contents

1. [Positional-Only Parameters](#1-positional-only-parameters)
2. [Keyword-Only Parameters](#2-keyword-only-parameters)
3. [Variadic Parameters — *args and **kwargs](#3-variadic-parameters----args-and-kwargs)
4. [Argument Unpacking](#4-argument-unpacking)
5. [Complete Parameter Order Reference](#5-complete-parameter-order-reference)
6. [First-Class Functions](#6-first-class-functions)
7. [Higher-Order Functions](#7-higher-order-functions)
8. [Lambda Functions](#8-lambda-functions)
9. [Partial Functions](#9-partial-functions)
10. [Comparison Tables](#10-comparison-tables)
11. [Real-World Projects](#11-real-world-projects)
12. [Cheat Sheets](#12-cheat-sheets)
13. [Interview Master Section](#13-interview-master-section)
14. [Exercises with Solutions](#14-exercises-with-solutions)
15. [Summary, Best Practices, and Common Pitfalls](#15-summary-best-practices-and-common-pitfalls)
16. [Learning Roadmap](#16-learning-roadmap)

---

## 1. Positional-Only Parameters

### What Are They?

Positional-only parameters are function parameters that **must** be passed by position — callers cannot use them as keyword arguments. They are defined using the `/` separator in the function signature.

Introduced in **Python 3.8 (PEP 570)**.

### Syntax

```python
def function_name(param1, param2, /, param3, param4):
    ...
```

Everything **before** the `/` is positional-only.

### Basic Example

```python
def greet(first_name, last_name, /):
    return f"Hello, {first_name} {last_name}!"

# ✅ Correct — positional
print(greet("Suresh", "Kumar"))
# Output: Hello, Suresh Kumar!

# ❌ Error — cannot use keyword for positional-only params
print(greet(first_name="Suresh", last_name="Kumar"))
# TypeError: greet() got some positional-only arguments passed as keyword arguments
```

### Mixed: Positional-Only + Regular

```python
def power(base, exponent, /, *, round_result=False):
    result = base ** exponent
    return round(result, 2) if round_result else result

# base and exponent → positional-only
# round_result → keyword-only (after *)
print(power(2, 10))                        # 1024
print(power(2, 0.5, round_result=True))    # 1.41
```

### Why Use Positional-Only Parameters?

**Reason 1: API stability** — You can rename the parameter internally without breaking callers.

```python
# Library v1.0
def calculate(x, y, /):
    return x + y

# Library v2.0 — renaming internally is safe
def calculate(a, b, /):   # callers never used keyword names
    return a + b
```

**Reason 2: Mimic built-in behavior**

```python
# Built-ins like len(), print() enforce positional usage
len([1, 2, 3])        # ✅
len(obj=[1, 2, 3])    # ❌ TypeError
```

**Reason 3: Prevent misuse of parameter names**

```python
def connect(host, port, /):
    print(f"Connecting to {host}:{port}")
# Callers cannot accidentally rely on the name "host" or "port"
```

### Practical Example — Math Library

```python
def slope(x1, y1, x2, y2, /):
    """Calculate slope between two points. Arguments are positional-only."""
    return (y2 - y1) / (x2 - x1)

print(slope(0, 0, 4, 8))   # 2.0
```

---

## 2. Keyword-Only Parameters

### What Are They?

Keyword-only parameters **must** be passed using their name — you cannot provide them positionally. They are defined by placing a bare `*` (or `*args`) before them.

### Syntax

```python
def function_name(param1, *, keyword_only1, keyword_only2=default):
    ...
```

Everything **after** the bare `*` is keyword-only.

### Basic Example

```python
def send_email(to, *, subject, body, cc=None):
    print(f"To: {to}")
    print(f"Subject: {subject}")
    print(f"Body: {body}")
    if cc:
        print(f"CC: {cc}")

# ✅ Correct
send_email("user@example.com", subject="Hello", body="How are you?")

# ❌ Error — subject must be keyword
send_email("user@example.com", "Hello", "How are you?")
# TypeError: send_email() takes 1 positional argument but 3 were given
```

### Keyword-Only with *args

```python
def log(*messages, level="INFO", timestamp=True):
    prefix = f"[{level}]" + (" [TS]" if timestamp else "")
    for msg in messages:
        print(f"{prefix} {msg}")

log("Server started", "Port 8080 open", level="DEBUG")
# [DEBUG] [TS] Server started
# [DEBUG] [TS] Port 8080 open
```

### Enforcing Explicit Intent

```python
def delete_records(table, *, confirm=False, dry_run=True):
    if not confirm:
        raise ValueError("You must explicitly pass confirm=True")
    if dry_run:
        print(f"[DRY RUN] Would delete all records from {table}")
    else:
        print(f"Deleting all records from {table}")

# Safe by design
delete_records("users", confirm=True, dry_run=False)
# Deleting all records from users
```

### Real-World: Django-style ORM Filter

```python
def filter_users(queryset, *, active=True, role=None, limit=100):
    results = [u for u in queryset if u.get("active") == active]
    if role:
        results = [u for u in results if u.get("role") == role]
    return results[:limit]

users = [
    {"name": "Alice", "active": True, "role": "admin"},
    {"name": "Bob",   "active": False, "role": "user"},
    {"name": "Carol", "active": True,  "role": "user"},
]

print(filter_users(users, active=True, role="admin"))
# [{'name': 'Alice', 'active': True, 'role': 'admin'}]
```

---

## 3. Variadic Parameters — *args and **kwargs

### Understanding *args

`*args` allows a function to accept **any number of positional arguments**. Inside the function, `args` is a **tuple**.

```python
def add(*args):
    return sum(args)

print(add(1, 2))           # 3
print(add(1, 2, 3, 4, 5)) # 15
print(add())               # 0
```

### Understanding **kwargs

`**kwargs` allows a function to accept **any number of keyword arguments**. Inside the function, `kwargs` is a **dict**.

```python
def display_info(**kwargs):
    for key, value in kwargs.items():
        print(f"{key}: {value}")

display_info(name="Suresh", city="Hyderabad", role="Developer")
# name: Suresh
# city: Hyderabad
# role: Developer
```

### Combining *args and **kwargs

```python
def flexible(*args, **kwargs):
    print("Positional:", args)
    print("Keyword:", kwargs)

flexible(1, 2, 3, name="Alice", age=25)
# Positional: (1, 2, 3)
# Keyword: {'name': 'Alice', 'age': 25}
```

### Forwarding Arguments

A very common pattern — forwarding to another function:

```python
def log_call(func, *args, **kwargs):
    print(f"Calling {func.__name__} with args={args}, kwargs={kwargs}")
    result = func(*args, **kwargs)
    print(f"Result: {result}")
    return result

def multiply(a, b):
    return a * b

log_call(multiply, 3, 4)
# Calling multiply with args=(3, 4), kwargs={}
# Result: 12
```

### *args: Tuple Behavior

```python
def show_types(*args):
    print(type(args))          # <class 'tuple'>
    print(args[0])             # First item
    for i, arg in enumerate(args):
        print(f"  args[{i}] = {arg!r}")

show_types("a", 10, True, [1, 2])
# <class 'tuple'>
# a
#   args[0] = 'a'
#   args[1] = 10
#   args[2] = True
#   args[3] = [1, 2]
```

### **kwargs: Dict Behavior

```python
def configure(**kwargs):
    print(type(kwargs))           # <class 'dict'>
    print(kwargs.get("debug"))    # Access by key
    kwargs.setdefault("timeout", 30)
    return kwargs

result = configure(host="localhost", debug=True)
print(result)
# {'host': 'localhost', 'debug': True, 'timeout': 30}
```

### Decorator Pattern Using *args and **kwargs

```python
import time

def timer(func):
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        end = time.time()
        print(f"{func.__name__} took {end - start:.4f}s")
        return result
    return wrapper

@timer
def slow_sum(n):
    return sum(range(n))

print(slow_sum(1_000_000))
# slow_sum took 0.0412s
# 499999500000
```

### Accepting Mixed Parameters

```python
def full_signature(pos_only, /, regular, *args, kw_only, **kwargs):
    print(f"pos_only:  {pos_only}")
    print(f"regular:   {regular}")
    print(f"args:      {args}")
    print(f"kw_only:   {kw_only}")
    print(f"kwargs:    {kwargs}")

full_signature(1, 2, 3, 4, 5, kw_only="must", extra="yes", flag=True)
# pos_only:  1
# regular:   2
# args:      (3, 4, 5)
# kw_only:   must
# kwargs:    {'extra': 'yes', 'flag': True}
```

---

## 4. Argument Unpacking

### Unpacking with * (Iterable → Positional Args)

The `*` operator unpacks any iterable (list, tuple, set, generator) into positional arguments when calling a function.

```python
def add(a, b, c):
    return a + b + c

numbers = [10, 20, 30]
print(add(*numbers))   # 60

# Equivalent to: add(10, 20, 30)
```

### Unpacking with ** (Dict → Keyword Args)

The `**` operator unpacks a dictionary into keyword arguments.

```python
def greet(name, greeting="Hello", punctuation="!"):
    return f"{greeting}, {name}{punctuation}"

config = {"name": "Suresh", "greeting": "Namaste", "punctuation": "."}
print(greet(**config))
# Namaste, Suresh.
```

### Combining * and ** in a Call

```python
def display(a, b, c, *, label, color="blue"):
    print(f"[{color}] {label}: {a}, {b}, {c}")

positional = (1, 2, 3)
keyword = {"label": "Values", "color": "green"}

display(*positional, **keyword)
# [green] Values: 1, 2, 3
```

### Partial Unpacking (Python 3.5+)

```python
first, *rest = [1, 2, 3, 4, 5]
print(first)  # 1
print(rest)   # [2, 3, 4, 5]

*start, last = [10, 20, 30, 40]
print(start)  # [10, 20, 30]
print(last)   # 40

head, *middle, tail = range(6)
print(head, middle, tail)  # 0 [1, 2, 3, 4] 5
```

### Merging Iterables and Dicts

```python
# Merging lists
a = [1, 2, 3]
b = [4, 5, 6]
combined = [*a, *b]
print(combined)  # [1, 2, 3, 4, 5, 6]

# Merging dicts (last key wins on collision)
defaults = {"timeout": 30, "retry": 3, "debug": False}
overrides = {"debug": True, "timeout": 60}
config = {**defaults, **overrides}
print(config)
# {'timeout': 60, 'retry': 3, 'debug': True}
```

### Unpacking Generator Expressions

```python
def sum_squares(a, b, c):
    return a + b + c

gen = (x**2 for x in [1, 2, 3])
print(sum_squares(*gen))   # 14 (1 + 4 + 9)
```

### Practical: Passing Dynamic Arguments

```python
import os

def run_command(executable, *flags, env=None, **options):
    cmd = [executable, *flags]
    for key, value in options.items():
        cmd.append(f"--{key}={value}")
    print("Command:", " ".join(cmd))

extra_flags = ["--verbose", "--no-color"]
extra_opts = {"output": "result.txt", "jobs": "4"}

run_command("pytest", *extra_flags, env="test", **extra_opts)
# Command: pytest --verbose --no-color --output=result.txt --jobs=4
```

---

## 5. Complete Parameter Order Reference

Python enforces a strict order for parameter types in a function signature:

```
def f(pos_only, /, regular, *args, kw_only, **kwargs):
```

| Position | Type | Separator |
|----------|------|-----------|
| 1 | Positional-only | Before `/` |
| 2 | Regular (pos or kw) | Between `/` and `*` |
| 3 | Variadic positional | `*args` |
| 4 | Keyword-only | After `*` or `*args` |
| 5 | Variadic keyword | `**kwargs` |

### Full Example

```python
def complete(
    pos_only_a,          # positional-only
    pos_only_b,          # positional-only
    /,
    regular_c,           # positional or keyword
    regular_d="default", # positional or keyword with default
    *args,               # extra positional → tuple
    kw_only_e,           # keyword-only, required
    kw_only_f=True,      # keyword-only with default
    **kwargs             # extra keyword → dict
):
    print(pos_only_a, pos_only_b, regular_c, regular_d,
          args, kw_only_e, kw_only_f, kwargs)

complete(1, 2, 3, 4, 5, 6, kw_only_e="must", extra="val")
# 1 2 3 4 (5, 6) must True {'extra': 'val'}
```

---

## 6. First-Class Functions

### What Does "First-Class" Mean?

In Python, functions are **first-class objects**. This means:

- A function can be **assigned** to a variable
- A function can be **passed** as an argument to another function
- A function can be **returned** from another function
- A function can be stored in **data structures** (list, dict, etc.)

### 6.1 Assigning Functions to Variables

```python
def square(x):
    return x ** 2

# Assign function to variable
compute = square

print(compute(5))       # 25
print(compute is square) # True — same object

# Check attributes
print(square.__name__)  # square
print(compute.__name__) # square
```

### 6.2 Storing Functions in Data Structures

```python
def add(a, b): return a + b
def subtract(a, b): return a - b
def multiply(a, b): return a * b
def divide(a, b): return a / b if b != 0 else None

# Dict of functions
operations = {
    "+": add,
    "-": subtract,
    "*": multiply,
    "/": divide,
}

op = "+"
result = operations[op](10, 5)
print(f"10 {op} 5 = {result}")   # 10 + 5 = 15

# List of functions
pipeline = [str.strip, str.lower, str.title]
text = "  hello world  "
for fn in pipeline:
    text = fn(text)
print(text)  # Hello World
```

### 6.3 Passing Functions as Arguments

```python
def apply(func, value):
    return func(value)

def double(x): return x * 2
def negate(x): return -x

print(apply(double, 7))   # 14
print(apply(negate, 7))   # -7
print(apply(abs, -99))    # 99 — built-in also works!
```

### 6.4 Returning Functions from Functions

```python
def make_multiplier(factor):
    def multiplier(x):
        return x * factor
    return multiplier

triple = make_multiplier(3)
print(triple(10))    # 30
print(triple(7))     # 21

# The returned function "remembers" factor via closure
print(triple.__closure__[0].cell_contents)   # 3
```

### 6.5 Functions Have Attributes

```python
def documented_function(x, y):
    """Adds two numbers together."""
    return x + y

print(documented_function.__name__)   # documented_function
print(documented_function.__doc__)    # Adds two numbers together.
print(documented_function.__module__) # __main__

# You can even add custom attributes
documented_function.version = "1.0"
print(documented_function.version)   # 1.0
```

---

## 7. Higher-Order Functions

A **higher-order function** is a function that either:
- Accepts one or more functions as arguments, OR
- Returns a function as its result (or both)

### Built-in Higher-Order Functions

#### map()

Applies a function to every element of an iterable.

```python
numbers = [1, 2, 3, 4, 5]

# map(function, iterable) → iterator
squared = list(map(lambda x: x**2, numbers))
print(squared)   # [1, 4, 9, 16, 25]

# With a named function
def to_celsius(f):
    return (f - 32) * 5 / 9

fahrenheit = [32, 68, 86, 104]
celsius = list(map(to_celsius, fahrenheit))
print(celsius)   # [0.0, 20.0, 30.0, 40.0]
```

#### filter()

Keeps only elements where the function returns True.

```python
numbers = range(-5, 6)

positives = list(filter(lambda x: x > 0, numbers))
print(positives)   # [1, 2, 3, 4, 5]

# Filter with a named predicate
def is_even(n): return n % 2 == 0

evens = list(filter(is_even, range(10)))
print(evens)   # [0, 2, 4, 6, 8]
```

#### sorted() with key

```python
people = [
    {"name": "Charlie", "age": 30},
    {"name": "Alice",   "age": 25},
    {"name": "Bob",     "age": 35},
]

# Sort by age (function as key=)
by_age = sorted(people, key=lambda p: p["age"])
print([p["name"] for p in by_age])   # ['Alice', 'Charlie', 'Bob']

# Sort by name length, then alphabetically
words = ["banana", "fig", "apple", "kiwi"]
sorted_words = sorted(words, key=lambda w: (len(w), w))
print(sorted_words)   # ['fig', 'kiwi', 'apple', 'banana']
```

#### reduce() from functools

```python
from functools import reduce

numbers = [1, 2, 3, 4, 5]

product = reduce(lambda acc, x: acc * x, numbers)
print(product)   # 120

# With initial value
total = reduce(lambda acc, x: acc + x, numbers, 100)
print(total)   # 115
```

### Custom Higher-Order Functions

```python
def apply_twice(func, value):
    """Applies func twice to value."""
    return func(func(value))

def increment(x): return x + 1

print(apply_twice(increment, 5))   # 7
print(apply_twice(str.upper, "hello"))  # HELLO (already upper, stays HELLO)
```

### Function Composition

```python
def compose(*functions):
    """Compose multiple functions right-to-left."""
    def composed(value):
        for func in reversed(functions):
            value = func(value)
        return value
    return composed

# compose(f, g, h)(x) == f(g(h(x)))
normalize = compose(str.title, str.strip, str.lower)
print(normalize("  HELLO WORLD  "))   # Hello World
```

### Memoization — A Higher-Order Utility

```python
def memoize(func):
    cache = {}
    def wrapper(*args):
        if args not in cache:
            cache[args] = func(*args)
        return cache[args]
    wrapper.cache = cache
    return wrapper

@memoize
def fibonacci(n):
    if n < 2:
        return n
    return fibonacci(n - 1) + fibonacci(n - 2)

print(fibonacci(40))          # 102334155
print(fibonacci.cache[40])    # 102334155 (from cache)
```

### retry Higher-Order Function

```python
import time

def retry(func, times=3, delay=1.0, exceptions=(Exception,)):
    def wrapper(*args, **kwargs):
        for attempt in range(1, times + 1):
            try:
                return func(*args, **kwargs)
            except exceptions as e:
                print(f"Attempt {attempt} failed: {e}")
                if attempt < times:
                    time.sleep(delay)
        raise RuntimeError(f"All {times} attempts failed")
    return wrapper

import random

@retry(times=3, delay=0.1)
def unstable_api_call():
    if random.random() < 0.7:   # 70% chance of failure
        raise ConnectionError("Network error")
    return "Success!"

# May retry up to 3 times
# print(unstable_api_call())
```

---

## 8. Lambda Functions

### What Is a Lambda?

A **lambda** is an anonymous, single-expression function. It can take any number of arguments but can only contain **one expression** (no statements, no assignments, no multi-line body).

### Syntax

```python
lambda parameters: expression
```

### Basic Lambda Examples

```python
# Simple lambda
add = lambda a, b: a + b
print(add(3, 4))   # 7

# Lambda with default value
greet = lambda name, greeting="Hello": f"{greeting}, {name}!"
print(greet("Suresh"))          # Hello, Suresh!
print(greet("Suresh", "Hi"))    # Hi, Suresh!

# Lambda with conditional expression
classify = lambda n: "positive" if n > 0 else "negative" if n < 0 else "zero"
print(classify(5))    # positive
print(classify(-3))   # negative
print(classify(0))    # zero
```

### Lambdas Are Function Objects

```python
fn = lambda x: x ** 2
print(type(fn))          # <class 'function'>
print(fn.__name__)       # <lambda>
print(callable(fn))      # True
```

### Lambdas as Inline Keys

```python
# Sort by second element of tuple
pairs = [(1, 'b'), (3, 'a'), (2, 'c')]
pairs.sort(key=lambda pair: pair[1])
print(pairs)   # [(3, 'a'), (1, 'b'), (2, 'c')]

# Multi-key sort
employees = [
    ("Alice", "Engineering", 90000),
    ("Bob",   "Marketing",   70000),
    ("Carol", "Engineering", 85000),
]
# Sort by department, then by salary descending
employees.sort(key=lambda e: (e[1], -e[2]))
print(employees)
# [('Alice', 'Engineering', 90000), ('Carol', 'Engineering', 85000), ('Bob', 'Marketing', 70000)]
```

### Lambdas with map, filter, reduce

```python
data = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

# map: square each
squared = list(map(lambda x: x**2, data))

# filter: keep even
evens = list(filter(lambda x: x % 2 == 0, data))

# Both together
even_squares = list(map(lambda x: x**2, filter(lambda x: x % 2 == 0, data)))
print(even_squares)   # [4, 16, 36, 64, 100]
```

### Immediately Invoked Lambda (IIFE)

```python
result = (lambda x, y: x * y)(6, 7)
print(result)   # 42
```

### Lambdas in GUI Callbacks (tkinter style)

```python
# Conceptual example — how lambdas solve the late-binding closure problem
buttons = []
for i in range(5):
    # Capturing i with default argument (lambda's own parameter)
    handler = lambda event, n=i: print(f"Button {n} clicked")
    buttons.append(handler)

buttons[2](None)   # Button 2 clicked
buttons[4](None)   # Button 4 clicked
```

### Lambda Limitations

```python
# ❌ Cannot use statements
# fn = lambda x: if x > 0: return x   # SyntaxError

# ❌ Cannot have multiple lines
# fn = lambda x:
#     y = x + 1
#     return y

# ❌ Cannot have assignments
# fn = lambda x: y = x + 1   # SyntaxError

# ✅ Use regular function for complex logic
def process(x):
    y = x + 1
    if y > 10:
        return y * 2
    return y
```

---

## 9. Partial Functions

### What Is functools.partial?

`functools.partial` creates a new callable by **fixing** some arguments (positional or keyword) of an existing function. The resulting partial object can be called later with the remaining arguments.

Think of it as "pre-filling" a function.

### Basic Usage

```python
from functools import partial

def multiply(a, b):
    return a * b

# Fix a=2 → creates a "double" function
double = partial(multiply, 2)

print(double(5))    # 10  (multiply(2, 5))
print(double(10))   # 20  (multiply(2, 10))
```

### Partial with Keyword Arguments

```python
from functools import partial

def power(base, exponent):
    return base ** exponent

square = partial(power, exponent=2)
cube   = partial(power, exponent=3)

print(square(5))   # 25
print(cube(3))     # 27
```

### Partial vs Lambda

```python
from functools import partial

def greet(greeting, punctuation, name):
    return f"{greeting}, {name}{punctuation}"

# Using partial
hello = partial(greet, "Hello", "!")
# Using lambda
hi    = lambda name: greet("Hi", ".", name)

print(hello("Alice"))   # Hello, Alice!
print(hi("Bob"))        # Hi, Bob.
```

### Partial Objects Have Attributes

```python
from functools import partial

def add(a, b, c=0):
    return a + b + c

add_five = partial(add, b=5)

print(add_five.func)    # <function add at ...>
print(add_five.args)    # ()
print(add_five.keywords)  # {'b': 5}

print(add_five(10))     # 15
print(add_five(10, c=2))  # 17
```

### Real-World: Logging with Partial

```python
from functools import partial
import logging

logging.basicConfig(level=logging.DEBUG)
logger = logging.getLogger("myapp")

# Create specialized loggers
debug = partial(logger.debug, extra={"component": "DB"})
error = partial(logger.error, extra={"component": "DB"})

# Now use them cleanly
# debug("Query executed: %s", sql)
# error("Connection failed: %s", err)
```

### Real-World: Data Transformation Pipeline

```python
from functools import partial

def scale(factor, value):
    return value * factor

def clamp(minimum, maximum, value):
    return max(minimum, min(maximum, value))

def round_to(digits, value):
    return round(value, digits)

# Create specialized transforms
normalize    = partial(scale, 1/255)       # Normalize 0-255 to 0-1
clip_unit    = partial(clamp, 0.0, 1.0)
round_4dp    = partial(round_to, 4)

# Build pipeline
def pipeline(*transforms):
    def process(value):
        for t in transforms:
            value = t(value)
        return value
    return process

pixel_to_float = pipeline(normalize, clip_unit, round_4dp)

print(pixel_to_float(128))   # 0.502
print(pixel_to_float(255))   # 1.0
print(pixel_to_float(300))   # 1.0 (clamped)
```

### Real-World: API Request Builder

```python
from functools import partial
import json

def api_request(method, endpoint, *, base_url, headers=None, data=None):
    url = f"{base_url}{endpoint}"
    print(f"{method} {url}")
    if headers:
        print(f"Headers: {headers}")
    if data:
        print(f"Body: {json.dumps(data)}")
    return {"status": 200, "url": url}

# Pre-configure for a specific service
github_api = partial(
    api_request,
    base_url="https://api.github.com",
    headers={"Authorization": "Bearer TOKEN", "Accept": "application/vnd.github.v3+json"}
)

# Now create verb-specific helpers
github_get  = partial(github_api, "GET")
github_post = partial(github_api, "POST")

github_get("/repos/python/cpython")
# GET https://api.github.com/repos/python/cpython
# Headers: {'Authorization': 'Bearer TOKEN', ...}

github_post("/user/repos", data={"name": "my-project", "private": False})
# POST https://api.github.com/user/repos
# Headers: {...}
# Body: {"name": "my-project", "private": false}
```

---

## 10. Comparison Tables

### Table 1: Positional-Only vs Keyword-Only

| Feature | Positional-Only (`/`) | Keyword-Only (`*`) |
|---|---|---|
| Separator syntax | `/` | `*` or `*args` |
| How to pass | By position only | By name only |
| Can rename internally | ✅ Safe | ❌ Breaking change |
| Prevents misuse | Prevents name reliance | Prevents positional guessing |
| Python version | 3.8+ | 3.0+ |
| Default values allowed | ✅ Yes | ✅ Yes |
| Example | `def f(x, y, /):` | `def f(*, x, y):` |

---

### Table 2: *args vs **kwargs

| Feature | `*args` | `**kwargs` |
|---|---|---|
| Collects | Extra positional args | Extra keyword args |
| Type inside function | `tuple` | `dict` |
| Order in signature | Before `**kwargs` | Last |
| Iteration | `for arg in args` | `for key, val in kwargs.items()` |
| Unpacking to call | `func(*args)` | `func(**kwargs)` |
| Can be renamed | ✅ Any name with `*` | ✅ Any name with `**` |
| Common use | Variable-length collections | Config/options |

---

### Table 3: Function vs Lambda

| Feature | Regular Function (`def`) | Lambda |
|---|---|---|
| Name | Has `__name__` | Always `<lambda>` |
| Body | Multiple statements | Single expression only |
| Docstring | ✅ Supported | ❌ Not supported |
| Assignments | ✅ Yes | ❌ No |
| Can be recursive | ✅ Yes (by name) | ⚠️ Only via variable |
| Debugging | Easier (named) | Harder (anonymous) |
| Readability | Higher for complex logic | Higher for simple inline use |
| Use case | Any function | Sorting keys, callbacks, HOF args |

---

### Table 4: Regular Function vs Partial Function

| Feature | Regular Function | Partial Function |
|---|---|---|
| Arguments | All supplied at call time | Some fixed at creation |
| Created with | `def` | `functools.partial(func, ...)` |
| Callable | ✅ Yes | ✅ Yes |
| Has `__name__` | ✅ Original name | ❌ (use `functools.update_wrapper`) |
| Introspection | `.args`, `.defaults` | `.func`, `.args`, `.keywords` |
| Best for | General use | Specializing existing functions |
| Alternative | Lambda | Lambda (less clear) |

---

## 11. Real-World Projects

### Project 1: Plugin System

A plugin system uses first-class functions to register and dispatch handlers dynamically.

```python
class PluginSystem:
    """A flexible plugin registry using first-class functions."""
    
    def __init__(self):
        self._plugins = {}
    
    def register(self, name):
        """Decorator to register a plugin by name."""
        def decorator(func):
            self._plugins[name] = func
            print(f"Plugin '{name}' registered")
            return func
        return decorator
    
    def execute(self, name, *args, **kwargs):
        """Execute a registered plugin."""
        if name not in self._plugins:
            raise KeyError(f"No plugin named '{name}'")
        return self._plugins[name](*args, **kwargs)
    
    def list_plugins(self):
        return list(self._plugins.keys())


# Usage
plugins = PluginSystem()

@plugins.register("csv_export")
def export_csv(data, *, delimiter=",", header=True):
    lines = []
    if header and data:
        lines.append(delimiter.join(data[0].keys()))
    for row in data:
        lines.append(delimiter.join(str(v) for v in row.values()))
    return "\n".join(lines)

@plugins.register("json_export")
def export_json(data, *, indent=2):
    import json
    return json.dumps(data, indent=indent)

@plugins.register("count")
def count_records(data, **kwargs):
    return len(data)

# Register a lambda plugin
plugins._plugins["upper_names"] = lambda data, **kw: [
    {**r, "name": r["name"].upper()} for r in data if "name" in r
]

# Run
sample_data = [
    {"name": "Alice", "score": 95},
    {"name": "Bob",   "score": 87},
]

print(plugins.execute("csv_export", sample_data))
# name,score
# Alice,95
# Bob,87

print(plugins.execute("count", sample_data))
# 2

print(plugins.execute("upper_names", sample_data))
# [{'name': 'ALICE', 'score': 95}, {'name': 'BOB', 'score': 87}]

print(plugins.list_plugins())
# ['csv_export', 'json_export', 'count', 'upper_names']
```

---

### Project 2: Event Dispatcher

An event dispatcher maps event names to lists of handler functions.

```python
from functools import partial
from typing import Callable, Any

class EventDispatcher:
    """Dispatch named events to registered handlers."""
    
    def __init__(self):
        self._handlers: dict[str, list[Callable]] = {}
    
    def on(self, event: str):
        """Decorator: register a function as handler for event."""
        def decorator(func):
            self._handlers.setdefault(event, []).append(func)
            return func
        return decorator
    
    def emit(self, event: str, *args, **kwargs):
        """Fire all handlers for the given event."""
        handlers = self._handlers.get(event, [])
        if not handlers:
            print(f"[WARN] No handlers for event: '{event}'")
            return []
        results = []
        for handler in handlers:
            result = handler(*args, **kwargs)
            results.append(result)
        return results
    
    def once(self, event: str, func: Callable):
        """Register a handler that fires only once."""
        def one_time(*args, **kwargs):
            self._handlers[event].remove(one_time)
            return func(*args, **kwargs)
        self._handlers.setdefault(event, []).append(one_time)
    
    def off(self, event: str, func: Callable = None):
        """Remove a specific handler or all handlers for event."""
        if func is None:
            self._handlers.pop(event, None)
        elif event in self._handlers:
            self._handlers[event] = [h for h in self._handlers[event] if h != func]


# Usage
dispatcher = EventDispatcher()

@dispatcher.on("user.login")
def log_login(user_id, *, ip=None):
    print(f"LOG: User {user_id} logged in from {ip}")

@dispatcher.on("user.login")
def send_welcome(user_id, **kwargs):
    print(f"EMAIL: Welcome back, user {user_id}!")

@dispatcher.on("user.logout")
def cleanup_session(user_id, **kwargs):
    print(f"SESSION: Cleared session for {user_id}")

# One-time analytics handler
dispatcher.once("user.login", lambda uid, **kw: print(f"ANALYTICS: First login event: {uid}"))

dispatcher.emit("user.login", "U001", ip="192.168.1.1")
# LOG: User U001 logged in from 192.168.1.1
# EMAIL: Welcome back, user U001!
# ANALYTICS: First login event: U001

dispatcher.emit("user.login", "U002", ip="10.0.0.5")
# LOG: User U002 logged in from 10.0.0.5
# EMAIL: Welcome back, user U002!
# (no ANALYTICS — already fired once)

dispatcher.emit("user.logout", "U001")
# SESSION: Cleared session for U001
```

---

### Project 3: Callback Framework

A flexible callback framework with before/after hooks.

```python
from functools import wraps
from typing import Callable

class CallbackFramework:
    """Attach before/after callbacks to any function."""
    
    def __init__(self):
        self._before: list[Callable] = []
        self._after:  list[Callable] = []
        self._error:  list[Callable] = []
    
    def before(self, func):
        self._before.append(func)
        return func
    
    def after(self, func):
        self._after.append(func)
        return func
    
    def on_error(self, func):
        self._error.append(func)
        return func
    
    def wrap(self, func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            context = {"args": args, "kwargs": kwargs, "func": func.__name__}
            
            for cb in self._before:
                cb(context)
            
            try:
                result = func(*args, **kwargs)
                context["result"] = result
                for cb in self._after:
                    cb(context)
                return result
            except Exception as e:
                context["error"] = e
                for cb in self._error:
                    cb(context)
                raise
        return wrapper


# Usage
framework = CallbackFramework()

@framework.before
def validate_input(ctx):
    print(f"  [BEFORE] Calling {ctx['func']} with args={ctx['args']}")

@framework.after
def log_result(ctx):
    print(f"  [AFTER]  Result: {ctx['result']}")

@framework.on_error
def handle_error(ctx):
    print(f"  [ERROR]  {ctx['func']} raised: {ctx['error']}")

@framework.wrap
def divide(a, b):
    return a / b

print("Test 1:")
divide(10, 2)
# [BEFORE] Calling divide with args=(10, 2)
# [AFTER]  Result: 5.0

print("\nTest 2 (error):")
try:
    divide(10, 0)
except ZeroDivisionError:
    pass
# [BEFORE] Calling divide with args=(10, 0)
# [ERROR]  divide raised: division by zero
```

---

### Project 4: Data Processing Pipeline

A composable pipeline using partial functions and higher-order functions.

```python
from functools import partial, reduce
from typing import Callable, Any, Iterable

# --- Core pipeline utilities ---

def compose(*funcs):
    """Compose functions right-to-left: compose(f, g)(x) == f(g(x))"""
    return reduce(lambda f, g: lambda x: f(g(x)), funcs)

def pipe(*funcs):
    """Pipe functions left-to-right: pipe(f, g)(x) == g(f(x))"""
    return reduce(lambda f, g: lambda x: g(f(x)), funcs)

# --- Reusable transform functions ---

def field(key, record):
    return record.get(key)

def keep_if(predicate, records):
    return [r for r in records if predicate(r)]

def transform_field(key, func, record):
    return {**record, key: func(record[key])}

def add_field(key, value_func, record):
    return {**record, key: value_func(record)}

def select_fields(keys, record):
    return {k: record[k] for k in keys if k in record}

# --- Partial specializations ---

get_name   = partial(field, "name")
get_salary = partial(field, "salary")

is_senior = lambda r: r.get("years_exp", 0) >= 5
is_active  = lambda r: r.get("active", True)

keep_seniors = partial(keep_if, is_senior)
keep_active  = partial(keep_if, is_active)

apply_bonus  = partial(transform_field, "salary", lambda s: round(s * 1.10, 2))
apply_tax    = partial(transform_field, "salary", lambda s: round(s * 0.85, 2))
add_category = partial(add_field, "category",
                       lambda r: "senior" if r.get("years_exp", 0) >= 5 else "junior")

summarize = partial(select_fields, ["name", "salary", "category"])

# --- Build the pipeline ---

process_employees = pipe(
    keep_active,
    keep_seniors,
    lambda records: [apply_bonus(r) for r in records],
    lambda records: [apply_tax(r) for r in records],
    lambda records: [add_category(r) for r in records],
    lambda records: [summarize(r) for r in records],
)

# --- Run it ---
employees = [
    {"name": "Alice",   "salary": 80000, "years_exp": 7, "active": True},
    {"name": "Bob",     "salary": 50000, "years_exp": 2, "active": True},
    {"name": "Carol",   "salary": 95000, "years_exp": 10, "active": False},
    {"name": "Dave",    "salary": 70000, "years_exp": 6, "active": True},
]

import json
results = process_employees(employees)
print(json.dumps(results, indent=2))
# [
#   {"name": "Alice", "salary": 74800.0, "category": "senior"},
#   {"name": "Dave",  "salary": 65450.0, "category": "senior"}
# ]
```

---

## 12. Cheat Sheets

### Cheat Sheet 1: Parameter Syntax Quick Reference

```
┌─────────────────────────────────────────────────────────────┐
│            PYTHON FUNCTION PARAMETER QUICK REFERENCE         │
├──────────────┬──────────────────┬──────────────────────────┤
│ Type         │ Syntax           │ Notes                     │
├──────────────┼──────────────────┼──────────────────────────┤
│ Positional   │ def f(x, y):     │ Can pass positionally     │
│              │                  │ or as keyword             │
├──────────────┼──────────────────┼──────────────────────────┤
│ Pos-only     │ def f(x, y, /):  │ MUST be positional        │
│              │                  │ (3.8+)                    │
├──────────────┼──────────────────┼──────────────────────────┤
│ Kw-only      │ def f(*, x, y):  │ MUST use keyword name     │
├──────────────┼──────────────────┼──────────────────────────┤
│ *args        │ def f(*args):    │ Collects extra positional │
│              │                  │ → tuple                   │
├──────────────┼──────────────────┼──────────────────────────┤
│ **kwargs     │ def f(**kw):     │ Collects extra keyword    │
│              │                  │ → dict                    │
├──────────────┼──────────────────┼──────────────────────────┤
│ Default      │ def f(x=10):     │ Used when not provided    │
├──────────────┼──────────────────┼──────────────────────────┤
│ Full order   │ pos_only, /,     │ Must appear in this order │
│              │ regular, *args,  │                           │
│              │ kw_only, **kw    │                           │
└──────────────┴──────────────────┴──────────────────────────┘
```

### Cheat Sheet 2: Argument Unpacking

```python
# * unpacks iterables to positional args
func(*[1, 2, 3])           → func(1, 2, 3)
func(*range(3))            → func(0, 1, 2)
func(*(x for x in "abc")) → func('a', 'b', 'c')

# ** unpacks dicts to keyword args
func(**{"a": 1, "b": 2})  → func(a=1, b=2)

# Merge
[*a, *b]           # Merge lists
{**d1, **d2}       # Merge dicts (d2 wins on collision)
(*t, extra_item)   # Extend tuple (creates new tuple)

# Destructuring
first, *rest = [1, 2, 3, 4]       # first=1, rest=[2,3,4]
*start, last = [1, 2, 3, 4]       # start=[1,2,3], last=4
a, *mid, b = range(5)             # a=0, mid=[1,2,3], b=4
```

### Cheat Sheet 3: Lambda Patterns

```python
# Identity
identity  = lambda x: x

# Constant
const_42  = lambda *_: 42

# Conditional
clamp     = lambda x, lo, hi: max(lo, min(hi, x))

# Sort key
by_second = lambda t: t[1]
by_len    = lambda s: len(s)
multi_key = lambda d: (d["dept"], -d["salary"])

# Chained calls
process   = lambda x: x.strip().lower().title()

# With default
greet     = lambda name, g="Hi": f"{g}, {name}"

# Immediately invoked
result    = (lambda x: x**2)(5)   # 25

# Capture loop variable with default arg
handlers  = [lambda e, i=i: print(i) for i in range(5)]
```

### Cheat Sheet 4: functools.partial Patterns

```python
from functools import partial

# Fix positional args
double = partial(multiply, 2)

# Fix keyword args
to_hex = partial(int, base=16)
print(to_hex("FF"))   # 255

# Fix both
log_error = partial(log, level="ERROR", timestamp=True)

# Inspect
p = partial(func, 1, 2, key="val")
p.func        # original function
p.args        # (1, 2)
p.keywords    # {"key": "val"}

# update_wrapper for better introspection
from functools import update_wrapper
update_wrapper(p, func)
```

### Cheat Sheet 5: Higher-Order Functions

```python
# map: transform each element
list(map(fn, iterable))
list(map(fn, iter1, iter2))     # Two iterables, fn(a, b)

# filter: keep matching elements
list(filter(predicate, iterable))
list(filter(None, iterable))    # Remove falsy values

# sorted with key
sorted(items, key=fn)
sorted(items, key=fn, reverse=True)

# reduce
from functools import reduce
reduce(fn, iterable)
reduce(fn, iterable, initial)

# any / all (HOFs that take iterables of booleans)
any(predicate(x) for x in items)
all(predicate(x) for x in items)

# zip
list(zip(iter1, iter2))

# Compose two functions
compose = lambda f, g: lambda x: f(g(x))
```

---

## 13. Interview Master Section

### Section A: Conceptual Questions

**Q1. What is the difference between positional-only and keyword-only parameters?**

Positional-only parameters (before `/`) can only be passed by position — you cannot use their name. Keyword-only parameters (after `*` or `*args`) can only be passed using their name. Positional-only were added in Python 3.8; keyword-only have been available since Python 3.0.

---

**Q2. What is the type of `args` inside a `*args` function? What about `kwargs`?**

`args` is a `tuple`. `kwargs` is a `dict`. This means `args` is immutable and ordered; `kwargs` is mutable and ordered (Python 3.7+).

---

**Q3. What happens if you put a mutable default argument like `def f(x=[])`?**

```python
def append_to(elem, lst=[]):
    lst.append(elem)
    return lst

print(append_to(1))   # [1]
print(append_to(2))   # [1, 2]  ← BUG! Shared state
print(append_to(3))   # [1, 2, 3]
```

The default `[]` is evaluated once at function definition time, not each call. The fix is to use `None` as the sentinel:

```python
def append_to(elem, lst=None):
    if lst is None:
        lst = []
    lst.append(elem)
    return lst
```

---

**Q4. What is a first-class function?**

A function that can be assigned to variables, passed as arguments, returned from other functions, and stored in data structures. In Python, all functions are first-class objects because they are instances of the `function` type.

---

**Q5. What is a higher-order function?**

A function that takes one or more functions as arguments and/or returns a function. Examples: `map()`, `filter()`, `sorted(key=...)`, `functools.reduce()`, decorators.

---

**Q6. What is a closure?**

A closure is a nested function that "closes over" (remembers) variables from its enclosing scope even after that scope has finished executing.

```python
def counter(start=0):
    count = [start]
    def increment(n=1):
        count[0] += n
        return count[0]
    return increment

c = counter(10)
print(c())     # 11
print(c(5))    # 16
```

---

**Q7. Why does this lambda in a loop produce unexpected output?**

```python
funcs = [lambda: i for i in range(3)]
print([f() for f in funcs])   # [2, 2, 2] — not [0, 1, 2]!
```

All lambdas capture `i` by reference (late binding). By the time they are called, the loop is done and `i=2`. Fix:

```python
funcs = [lambda i=i: i for i in range(3)]
print([f() for f in funcs])   # [0, 1, 2] ✅
```

---

**Q8. What is `functools.partial` and when would you use it?**

`partial` creates a new callable with some arguments pre-filled. Use it to specialize a general function, adapt function signatures for callbacks/APIs, or avoid repetitive argument passing.

---

**Q9. What is the difference between `*args` and `**kwargs` when unpacking at a call site?**

- `func(*iterable)` unpacks an iterable into positional arguments
- `func(**mapping)` unpacks a dict into keyword arguments

These are the inverse of `*args`/`**kwargs` in the definition — collection vs. spreading.

---

**Q10. Can you pass `**kwargs` to a function that doesn't accept `**kwargs`?**

Yes, as long as the dict keys match the function's parameter names exactly:

```python
def greet(name, greeting):
    return f"{greeting}, {name}"

data = {"name": "Alice", "greeting": "Hello"}
print(greet(**data))   # Hello, Alice
```

---

### Section B: Code Analysis Questions

**Q11. What is the output?**

```python
def f(a, b=2, *args, c, **kwargs):
    print(a, b, args, c, kwargs)

f(1, 3, 4, 5, c=6, d=7)
```

Answer: `1 3 (4, 5) 6 {'d': 7}`

---

**Q12. Is this valid? What does it print?**

```python
def add(a, b, /):
    return a + b

print(add(*(3, 4)))
```

Answer: Yes, valid. Output: `7`. The `*` unpacks the tuple into positional arguments.

---

**Q13. What is the output?**

```python
from functools import partial

def f(a, b, c):
    return a - b - c

g = partial(f, c=10)
print(g(100, 50))
```

Answer: `40` (100 - 50 - 10)

---

**Q14. Is this lambda valid?**

```python
fn = lambda x, *args, y=0, **kw: (x, args, y, kw)
print(fn(1, 2, 3, y=4, z=5))
```

Answer: Yes, valid. Output: `(1, (2, 3), 4, {'z': 5})`. Lambdas support full parameter syntax.

---

**Q15. What does this return?**

```python
def make_adder(n):
    return lambda x: x + n

add3 = make_adder(3)
add7 = make_adder(7)

print(add3(add7(10)))
```

Answer: `20`. `add7(10)` = 17, then `add3(17)` = 20.

---

**Q16. What is the output of this code?**

```python
def apply(f, *args):
    return f(*args)

print(apply(sorted, [3,1,2], key=lambda x: -x))
```

Answer: `[3, 2, 1]`

---

**Q17. Does this code raise an error? Why?**

```python
def f(x, y, /):
    return x + y

d = {"x": 1, "y": 2}
f(**d)
```

Answer: Yes — `TypeError: f() got some positional-only arguments passed as keyword arguments`. Even though `x` and `y` exist as keys, they cannot be passed as keywords to positional-only params.

---

**Q18. What is the output?**

```python
ops = {
    "double": lambda x: x * 2,
    "negate": lambda x: -x,
    "square": lambda x: x ** 2,
}

result = 3
for name in ["double", "square", "negate"]:
    result = ops[name](result)

print(result)
```

Answer: `3 → 6 → 36 → -36`. Output: `-36`

---

**Q19. Is `*` valid in a function call? What does it do?**

```python
a = [1, 2]
b = [3, 4]
print([*a, *b])
print(max(*a, *b))
```

Answer: Yes. `[*a, *b]` creates `[1, 2, 3, 4]`. `max(*a, *b)` unpacks both into `max(1, 2, 3, 4)` = `4`.

---

**Q20. What is a decorator in Python and how is it related to higher-order functions?**

A decorator is syntactic sugar for passing a function to a higher-order function and binding the result back to the original name:

```python
@decorator
def func(): ...
# Equivalent to:
def func(): ...
func = decorator(func)
```

A decorator is a higher-order function: it accepts a function as input and returns a function (usually a wrapper).

---

### Section C: Design & Application Questions

**Q21. When should you choose partial over lambda?**

Use `partial` when pre-filling real function arguments for reuse — it preserves the original function's metadata and is more readable. Use `lambda` for quick one-off inline transforms that don't need a name or reuse.

---

**Q22. How would you write a type-safe, reusable `map` that works with partial?**

```python
from functools import partial

def safe_map(func, iterable, default=None):
    results = []
    for item in iterable:
        try:
            results.append(func(item))
        except Exception:
            results.append(default)
    return results

safe_int = partial(safe_map, int, default=0)
print(safe_int(["1", "2", "abc", "4"]))   # [1, 2, 0, 4]
```

---

**Q23. What are the dangers of `*args` with mutable default arguments?**

The main danger with `*args` is that if you collect args into a list and mutate them, you affect the original objects if they are mutable. Separately, mutable defaults (like `def f(x=[])`) share state across calls.

---

**Q24. How can `**kwargs` help with forward-compatible APIs?**

```python
def v1_api(name, **kwargs):
    # kwargs absorbs future parameters
    # without breaking existing callers
    return {"name": name}

# v2 adds age, old code still works
def v2_api(name, *, age=None, **kwargs):
    return {"name": name, "age": age}
```

---

**Q25. Explain function composition vs chaining.**

```python
# Composition: apply functions in a pipeline
def compose(*fns):
    from functools import reduce
    return reduce(lambda f, g: lambda x: f(g(x)), fns)

# Chaining: method chaining on objects
result = "  hello  ".strip().title().replace("o", "0")
# H0llo

# Composition on plain functions
normalize = compose(str.title, str.strip)
print(normalize("  hello  "))   # Hello
```

---

**Q26. How would you implement a function that counts how many times it has been called?**

```python
def call_counter(func):
    def wrapper(*args, **kwargs):
        wrapper.calls += 1
        return func(*args, **kwargs)
    wrapper.calls = 0
    return wrapper

@call_counter
def greet(name):
    return f"Hello, {name}"

greet("Alice")
greet("Bob")
print(greet.calls)   # 2
```

---

**Q27. How do *args interact with positional-only and keyword-only parameters?**

```python
def f(pos_only, /, regular, *args, kw_only, **kwargs):
    pass

# pos_only → must be positional
# regular  → can be positional or keyword
# *args    → captures any remaining positional args
# kw_only  → after *args, must be keyword
# **kwargs → captures remaining keyword args
```

---

**Q28. What is the difference between `map` and a list comprehension?**

Both apply a transformation to each element. `map` is lazy (returns an iterator), slightly faster for large datasets with named functions, but less readable for complex expressions. List comprehensions are eager (returns a list), support conditions (`if`), are generally more Pythonic, and can flatten/combine iterables.

---

**Q29. When would you use `functools.reduce` vs a loop?**

`reduce` is expressive for fold operations (combining all elements into one) — like computing product or merging dicts. A loop is clearer for side-effectful operations or when the logic is complex. `reduce` is often replaced by built-ins (`sum`, `max`, `any`, `all`).

---

**Q30. How do you document a function that uses *args and **kwargs?**

```python
def func(*args, **kwargs):
    """
    Process arbitrary inputs.
    
    Args:
        *args: Variable positional arguments.
            Each should be a numeric value to process.
        **kwargs: Arbitrary keyword arguments.
            Supported keys:
                - verbose (bool): Enable verbose output.
                - precision (int): Decimal places. Default: 2.
    
    Returns:
        dict: Processed results.
    """
    pass
```

---

### Section D: Tricky Questions

**Q31. What is the output?**

```python
f = lambda: lambda x: x + 1
print(f()(5))
```

Answer: `6`. `f()` returns `lambda x: x + 1`, then that is called with `5`.

---

**Q32. Can you use `*` to unpack inside a function signature default?**

```python
# ❌ Not valid
def f(x=*[1, 2]):
    pass
# SyntaxError

# ✅ Unpack at call site
f(*[1, 2])
```

---

**Q33. What is the difference between these two?**

```python
from functools import partial

def f(a, b, c):
    return (a, b, c)

g1 = partial(f, 1, 2)   # Fix a=1, b=2
g2 = lambda c: f(1, 2, c)  # Same effect

print(g1(3))    # (1, 2, 3)
print(g2(3))    # (1, 2, 3)
```

Answer: Same output, but `partial` preserves the original function as `g1.func` and is more introspectable. The lambda creates a new anonymous function.

---

**Q34. Is `**` valid in a function definition? What about just `*`?**

- `**name` in a definition collects keyword arguments into a dict → valid
- `*name` in a definition collects positional arguments into a tuple → valid
- Bare `*` in a definition (no name) marks the start of keyword-only params → valid
- Bare `**` is NOT valid in a definition

---

**Q35. What happens if you call `partial` on a partial object?**

```python
from functools import partial

def f(a, b, c):
    return a + b + c

g = partial(f, 1)
h = partial(g, 2)   # partial of a partial

print(h(3))   # 6 — equivalent to f(1, 2, 3)
```

It works! `partial` of a `partial` simply chains the pre-filled arguments.

---

## 14. Exercises with Solutions

### Intermediate Exercises (15)

---

**Exercise I-1.** Write a function `only_positional(x, y, z, /)` that returns the product of three numbers. Verify that passing by keyword raises `TypeError`.

```python
def only_positional(x, y, z, /):
    return x * y * z

print(only_positional(2, 3, 4))    # 24
try:
    only_positional(x=2, y=3, z=4)
except TypeError as e:
    print(f"Error: {e}")
# Error: only_positional() got some positional-only arguments passed as keyword arguments
```

---

**Exercise I-2.** Write a function `configure_server` where `host` and `port` are positional-only and `debug`, `timeout` are keyword-only with defaults.

```python
def configure_server(host, port, /, *, debug=False, timeout=30):
    return {"host": host, "port": port, "debug": debug, "timeout": timeout}

print(configure_server("localhost", 8080))
# {'host': 'localhost', 'port': 8080, 'debug': False, 'timeout': 30}

print(configure_server("0.0.0.0", 443, debug=True, timeout=60))
# {'host': '0.0.0.0', 'port': 443, 'debug': True, 'timeout': 60}
```

---

**Exercise I-3.** Write a function `stats(*numbers)` that returns a dict with min, max, sum, and average.

```python
def stats(*numbers):
    if not numbers:
        return {}
    return {
        "min":     min(numbers),
        "max":     max(numbers),
        "sum":     sum(numbers),
        "average": sum(numbers) / len(numbers),
        "count":   len(numbers)
    }

print(stats(4, 7, 2, 9, 1))
# {'min': 1, 'max': 9, 'sum': 23, 'average': 4.6, 'count': 5}
```

---

**Exercise I-4.** Write `make_greeting(**defaults)` that returns a function accepting a name and merging caller kwargs with defaults.

```python
def make_greeting(**defaults):
    def greet(name, **overrides):
        opts = {**defaults, **overrides}
        return f"{opts.get('greeting', 'Hello')}, {name}{opts.get('punctuation', '!')}"
    return greet

formal = make_greeting(greeting="Good day", punctuation=".")
casual = make_greeting(greeting="Hey")

print(formal("Alice"))           # Good day, Alice.
print(casual("Bob"))             # Hey, Bob!
print(casual("Carol", punctuation="?"))  # Hey, Carol?
```

---

**Exercise I-5.** Use `map` and `filter` (no list comprehension) to get the squares of all odd numbers in `range(1, 20)`.

```python
odd_squares = list(map(lambda x: x**2, filter(lambda x: x % 2 != 0, range(1, 20))))
print(odd_squares)
# [1, 9, 25, 49, 81, 121, 169, 225, 289, 361]
```

---

**Exercise I-6.** Write a lambda that takes a string and returns a version with each word capitalized and numbers removed.

```python
import re
clean_title = lambda s: " ".join(re.sub(r"\d", "", w).strip().capitalize()
                                  for w in s.split() if re.sub(r"\d", "", w).strip())

print(clean_title("hello 3 world 99 python"))
# Hello World Python
```

---

**Exercise I-7.** Write `apply_all(value, *funcs)` that applies each function to the value in sequence.

```python
def apply_all(value, *funcs):
    for fn in funcs:
        value = fn(value)
    return value

result = apply_all("  HELLO WORLD  ", str.strip, str.lower, str.title)
print(result)   # Hello World
```

---

**Exercise I-8.** Use `functools.partial` to create a `fahrenheit_to_celsius` and `celsius_to_fahrenheit` from a single base function.

```python
from functools import partial

def convert_temp(value, *, from_unit, to_unit):
    if from_unit == "F" and to_unit == "C":
        return (value - 32) * 5/9
    if from_unit == "C" and to_unit == "F":
        return value * 9/5 + 32
    return value

f_to_c = partial(convert_temp, from_unit="F", to_unit="C")
c_to_f = partial(convert_temp, from_unit="C", to_unit="F")

print(round(f_to_c(98.6), 2))   # 37.0
print(c_to_f(100))              # 212.0
```

---

**Exercise I-9.** Write a `flatten(*iterables)` that accepts any number of iterables and returns a flat list.

```python
def flatten(*iterables):
    result = []
    for it in iterables:
        result.extend(it)
    return result

print(flatten([1, 2], (3, 4), range(5, 8)))
# [1, 2, 3, 4, 5, 6, 7]
```

---

**Exercise I-10.** Store the functions `int`, `float`, `str`, `bool` in a dict and use it to convert a value based on a type string.

```python
converters = {"int": int, "float": float, "str": str, "bool": bool}

def convert(value, type_name):
    if type_name not in converters:
        raise ValueError(f"Unknown type: {type_name}")
    return converters[type_name](value)

print(convert("42", "int"))    # 42
print(convert("3.14", "float"))  # 3.14
print(convert(0, "bool"))      # False
```

---

**Exercise I-11.** Write a higher-order function `validate(func, validator)` that calls `func` only if `validator(args)` returns True.

```python
def validate(func, validator, error="Validation failed"):
    def wrapper(*args, **kwargs):
        if not validator(*args, **kwargs):
            raise ValueError(error)
        return func(*args, **kwargs)
    return wrapper

def divide(a, b):
    return a / b

safe_divide = validate(divide, lambda a, b: b != 0, "Cannot divide by zero")

print(safe_divide(10, 2))   # 5.0
try:
    safe_divide(10, 0)
except ValueError as e:
    print(e)   # Cannot divide by zero
```

---

**Exercise I-12.** Unpack two lists into a function call using `*` and merge two dicts using `**`.

```python
def describe(name, role, city, *, active, department):
    return f"{name} ({role}) at {city} — dept: {department}, active: {active}"

positional_args = ["Suresh", "Developer", "Hyderabad"]
keyword_args    = {"active": True, "department": "Engineering"}

print(describe(*positional_args, **keyword_args))
# Suresh (Developer) at Hyderabad — dept: Engineering, active: True
```

---

**Exercise I-13.** Write a function `make_pipeline(*transforms)` that returns a function applying each transform in order.

```python
def make_pipeline(*transforms):
    def pipeline(value):
        for t in transforms:
            value = t(value)
        return value
    return pipeline

clean = make_pipeline(str.strip, str.lower, str.title)
print(clean("  hello from hyderabad  "))
# Hello From Hyderabad
```

---

**Exercise I-14.** Use a lambda as a key to sort a list of tuples by the absolute difference between their two elements.

```python
pairs = [(1, 5), (3, 4), (10, 2), (7, 7), (0, 9)]
sorted_pairs = sorted(pairs, key=lambda t: abs(t[0] - t[1]))
print(sorted_pairs)
# [(7, 7), (3, 4), (1, 5), (10, 2), (0, 9)]
```

---

**Exercise I-15.** Use `partial` to create a left-pad function with a fixed width of 10 and fill character `0`.

```python
from functools import partial

def pad_string(s, width, fillchar=" "):
    return str(s).rjust(width, fillchar)

zero_pad_10 = partial(pad_string, width=10, fillchar="0")

print(zero_pad_10(42))     # 0000000042
print(zero_pad_10(12345))  # 0000012345
print(zero_pad_10("AB"))   # 00000000AB
```

---

### Advanced Exercises (15)

---

**Exercise A-1.** Implement a `once(func)` decorator using closures that ensures a function is called at most once.

```python
def once(func):
    called = [False]
    result = [None]
    def wrapper(*args, **kwargs):
        if not called[0]:
            result[0] = func(*args, **kwargs)
            called[0] = True
        return result[0]
    return wrapper

@once
def initialize():
    print("Initializing...")
    return 42

print(initialize())   # Initializing... / 42
print(initialize())   # 42 (no print — not called again)
```

---

**Exercise A-2.** Implement `compose(*funcs)` that creates a right-to-left function composition.

```python
from functools import reduce

def compose(*funcs):
    def composed(x):
        return reduce(lambda v, f: f(v), reversed(funcs), x)
    return composed

to_title_words = compose(
    str.title,
    str.strip,
    " ".join,
    lambda s: s.split()
)

print(to_title_words("  hello world python  "))
# Hello World Python
```

---

**Exercise A-3.** Write a `dispatch` function that selects a handler based on the type of the first argument.

```python
def dispatch(**handlers):
    def dispatcher(value, *args, **kwargs):
        handler = handlers.get(type(value).__name__)
        if handler is None:
            raise TypeError(f"No handler for type: {type(value).__name__}")
        return handler(value, *args, **kwargs)
    return dispatcher

stringify = dispatch(
    int=lambda x: f"Integer: {x}",
    str=lambda x: f"String: '{x}'",
    list=lambda x: f"List of {len(x)} items",
    dict=lambda x: f"Dict with keys: {list(x.keys())}",
)

print(stringify(42))            # Integer: 42
print(stringify("hello"))       # String: 'hello'
print(stringify([1, 2, 3]))    # List of 3 items
print(stringify({"a": 1}))     # Dict with keys: ['a']
```

---

**Exercise A-4.** Write a `memoize_with_ttl(ttl_seconds)` decorator that caches function results for a given time window.

```python
import time

def memoize_with_ttl(ttl_seconds):
    def decorator(func):
        cache = {}  # key → (result, timestamp)
        def wrapper(*args):
            now = time.time()
            if args in cache:
                result, ts = cache[args]
                if now - ts < ttl_seconds:
                    print(f"  [CACHE HIT] {args}")
                    return result
            result = func(*args)
            cache[args] = (result, now)
            return result
        wrapper.cache = cache
        return wrapper
    return decorator

@memoize_with_ttl(ttl_seconds=2)
def expensive_computation(n):
    print(f"  [COMPUTING] n={n}")
    return n ** 2

print(expensive_computation(5))   # COMPUTING, result: 25
print(expensive_computation(5))   # CACHE HIT, result: 25
time.sleep(2.1)
print(expensive_computation(5))   # COMPUTING again (TTL expired)
```

---

**Exercise A-5.** Create a `Registry` class that uses `**kwargs` to register multiple handlers at once and partial to pre-configure them.

```python
from functools import partial

class Registry:
    def __init__(self):
        self._handlers = {}
    
    def register(self, **name_to_func):
        self._handlers.update(name_to_func)
    
    def configure(self, name, **fixed_kwargs):
        if name not in self._handlers:
            raise KeyError(f"'{name}' not registered")
        self._handlers[name] = partial(self._handlers[name], **fixed_kwargs)
    
    def call(self, name, *args, **kwargs):
        return self._handlers[name](*args, **kwargs)

def format_output(data, *, indent=0, prefix=""):
    pad = " " * indent
    return f"{pad}{prefix}{data}"

reg = Registry()
reg.register(default=format_output, debug=format_output, error=format_output)
reg.configure("debug", indent=4, prefix="[DEBUG] ")
reg.configure("error", indent=0, prefix="[ERROR] ")

print(reg.call("default", "Hello"))
print(reg.call("debug", "Variable x=5"))
print(reg.call("error", "File not found"))
# Hello
#     [DEBUG] Variable x=5
# [ERROR] File not found
```

---

**Exercise A-6.** Implement a `pipe` that supports branching (fan-out): one value goes to multiple transforms, results collected as a tuple.

```python
def fanout(*transforms):
    """Apply multiple transforms to the same input, return tuple of results."""
    def process(value):
        return tuple(t(value) for t in transforms)
    return process

analyze = fanout(
    lambda s: len(s),
    lambda s: s.upper(),
    lambda s: s[::-1],
    lambda s: s.count(" "),
)

print(analyze("hello world"))
# (11, 'HELLO WORLD', 'dlrow olleh', 1)
```

---

**Exercise A-7.** Build a `FluentBuilder` that uses first-class functions to add operations lazily.

```python
class FluentBuilder:
    def __init__(self, value):
        self._value = value
        self._ops = []
    
    def apply(self, func, *args, **kwargs):
        from functools import partial
        self._ops.append(partial(func, **kwargs) if kwargs else func)
        return self
    
    def build(self):
        result = self._value
        for op in self._ops:
            result = op(result)
        return result

result = (FluentBuilder("  Hello World  ")
          .apply(str.strip)
          .apply(str.lower)
          .apply(str.title)
          .build())
print(result)   # Hello World
```

---

**Exercise A-8.** Use `**kwargs` to build a flexible SQL-like WHERE clause generator.

```python
def where(**conditions):
    """Generate a filter function from keyword conditions."""
    def matches(record):
        return all(record.get(k) == v for k, v in conditions.items())
    return matches

users = [
    {"name": "Alice", "role": "admin",  "active": True},
    {"name": "Bob",   "role": "user",   "active": True},
    {"name": "Carol", "role": "admin",  "active": False},
]

active_admins = list(filter(where(role="admin", active=True), users))
print(active_admins)
# [{'name': 'Alice', 'role': 'admin', 'active': True}]
```

---

**Exercise A-9.** Implement `curry(func)` — convert a multi-argument function into a chain of single-argument functions.

```python
import inspect

def curry(func):
    n = len(inspect.signature(func).parameters)
    def curried(*args):
        if len(args) >= n:
            return func(*args[:n])
        return lambda *more: curried(*(args + more))
    return curried

@curry
def add3(a, b, c):
    return a + b + c

print(add3(1)(2)(3))      # 6
print(add3(1, 2)(3))      # 6
print(add3(1)(2, 3))      # 6
print(add3(1, 2, 3))      # 6
```

---

**Exercise A-10.** Write a `throttle(func, max_calls_per_second)` higher-order function.

```python
import time

def throttle(func, max_calls_per_second):
    interval = 1.0 / max_calls_per_second
    last_called = [0.0]
    
    def wrapper(*args, **kwargs):
        now = time.time()
        elapsed = now - last_called[0]
        if elapsed < interval:
            wait = interval - elapsed
            print(f"  [THROTTLE] Waiting {wait:.3f}s")
            time.sleep(wait)
        last_called[0] = time.time()
        return func(*args, **kwargs)
    return wrapper

@throttle(max_calls_per_second=2)
def fetch(url):
    return f"Data from {url}"

for i in range(4):
    print(fetch(f"https://api.example.com/item/{i}"))
```

---

**Exercise A-11.** Build a function `overload(**type_dispatch)` that dispatches based on argument type.

```python
def overload(**type_funcs):
    """Dispatch to different functions based on argument type."""
    def dispatcher(*args, **kwargs):
        for arg in args:
            type_name = type(arg).__name__
            if type_name in type_funcs:
                return type_funcs[type_name](*args, **kwargs)
        raise TypeError(f"No overload for types: {[type(a).__name__ for a in args]}")
    return dispatcher

double = overload(
    int=lambda x: x * 2,
    str=lambda x: x + x,
    list=lambda x: x + x,
)

print(double(5))         # 10
print(double("Hi"))      # HiHi
print(double([1, 2]))    # [1, 2, 1, 2]
```

---

**Exercise A-12.** Implement a lazy `Stream` class using higher-order functions.

```python
class Stream:
    def __init__(self, iterable):
        self._data = iter(iterable)
        self._ops = []
    
    def map(self, func):
        self._ops.append(("map", func))
        return self
    
    def filter(self, predicate):
        self._ops.append(("filter", predicate))
        return self
    
    def collect(self):
        result = self._data
        for op_type, func in self._ops:
            if op_type == "map":
                result = map(func, result)
            elif op_type == "filter":
                result = filter(func, result)
        return list(result)

numbers = Stream(range(1, 21))
result = (numbers
          .filter(lambda x: x % 2 == 0)
          .map(lambda x: x ** 2)
          .filter(lambda x: x > 50)
          .collect())
print(result)   # [64, 100, 144, 196, 256, 324, 400]
```

---

**Exercise A-13.** Write `partial_right(func, *args)` — pre-fill arguments from the right.

```python
def partial_right(func, *right_args):
    def wrapper(*left_args, **kwargs):
        return func(*left_args, *right_args, **kwargs)
    return wrapper

def divide(a, b):
    return a / b

half = partial_right(divide, 2)    # divide(x, 2)
print(half(10))   # 5.0
print(half(7))    # 3.5

def power(base, exponent):
    return base ** exponent

square_root = partial_right(power, 0.5)
print(square_root(9))    # 3.0
print(square_root(16))   # 4.0
```

---

**Exercise A-14.** Build a function pipeline that supports error handling per-step.

```python
def safe_pipeline(*steps, on_error=None):
    def run(value):
        for step in steps:
            try:
                value = step(value)
            except Exception as e:
                if on_error:
                    return on_error(e, step, value)
                raise
        return value
    return run

process = safe_pipeline(
    str.strip,
    int,                         # may raise ValueError
    lambda x: 100 / x,          # may raise ZeroDivisionError
    lambda x: round(x, 2),
    on_error=lambda e, step, val: f"Error in step: {e}"
)

print(process("   25   "))   # 4.0
print(process("  abc  "))    # Error in step: invalid literal...
print(process("  0   "))     # Error in step: division by zero
```

---

**Exercise A-15.** Implement a function signature inspector using `inspect` and `*args`/`**kwargs`.

```python
import inspect
from functools import wraps

def signature_logger(func):
    sig = inspect.signature(func)
    
    @wraps(func)
    def wrapper(*args, **kwargs):
        bound = sig.bind(*args, **kwargs)
        bound.apply_defaults()
        print(f"\nCall to: {func.__name__}")
        for name, value in bound.arguments.items():
            param = sig.parameters[name]
            kind = param.kind.name
            print(f"  {name} ({kind}) = {value!r}")
        return func(*args, **kwargs)
    
    return wrapper

@signature_logger
def create_user(user_id, /, name, *roles, active=True, **metadata):
    return {"id": user_id, "name": name, "roles": roles,
            "active": active, **metadata}

create_user(1, "Alice", "admin", "editor", active=True, department="IT")
# Call to: create_user
#   user_id (POSITIONAL_ONLY) = 1
#   name (POSITIONAL_OR_KEYWORD) = 'Alice'
#   roles (VAR_POSITIONAL) = ('admin', 'editor')
#   active (KEYWORD_ONLY) = True
#   metadata (VAR_KEYWORD) = {'department': 'IT'}
```

---

### Expert Exercises (15)

---

**Exercise E-1.** Implement a full `functools.lru_cache` from scratch using a `dict` and doubly-linked list logic (simplified).

```python
from collections import OrderedDict

def lru_cache(maxsize=128):
    def decorator(func):
        cache = OrderedDict()
        hits = [0]
        misses = [0]
        
        def wrapper(*args):
            if args in cache:
                hits[0] += 1
                cache.move_to_end(args)
                return cache[args]
            misses[0] += 1
            result = func(*args)
            cache[args] = result
            if len(cache) > maxsize:
                cache.popitem(last=False)
            return result
        
        wrapper.cache_info = lambda: {
            "hits": hits[0], "misses": misses[0],
            "maxsize": maxsize, "currsize": len(cache)
        }
        wrapper.cache_clear = lambda: cache.clear()
        return wrapper
    return decorator

@lru_cache(maxsize=5)
def fib(n):
    if n < 2: return n
    return fib(n-1) + fib(n-2)

print(fib(20))
print(fib.cache_info())
```

---

**Exercise E-2.** Build a `FunctionChain` that allows `|` (pipe) operator to chain functions.

```python
class FunctionChain:
    def __init__(self, func):
        self._func = func
    
    def __call__(self, *args, **kwargs):
        return self._func(*args, **kwargs)
    
    def __or__(self, other):
        def chained(*args, **kwargs):
            return other(self._func(*args, **kwargs))
        return FunctionChain(chained)
    
    def __repr__(self):
        return f"FunctionChain({self._func.__name__})"

fn = FunctionChain(lambda s: s.strip()) | str.lower | str.title

print(fn("  hello world  "))   # Hello World
```

---

**Exercise E-3.** Implement a type-checked argument validator using `*args`, `**kwargs`, and Python type hints.

```python
import inspect
from functools import wraps

def typed(func):
    hints = func.__annotations__
    sig = inspect.signature(func)
    
    @wraps(func)
    def wrapper(*args, **kwargs):
        bound = sig.bind(*args, **kwargs)
        bound.apply_defaults()
        for name, value in bound.arguments.items():
            if name in hints and name != "return":
                expected = hints[name]
                if not isinstance(value, expected):
                    raise TypeError(
                        f"Argument '{name}' must be {expected.__name__}, "
                        f"got {type(value).__name__}"
                    )
        result = func(*args, **kwargs)
        if "return" in hints and not isinstance(result, hints["return"]):
            raise TypeError(f"Return type must be {hints['return'].__name__}")
        return result
    return wrapper

@typed
def greet(name: str, times: int = 1) -> str:
    return (f"Hello, {name}! " * times).strip()

print(greet("Suresh", 2))
# Hello, Suresh! Hello, Suresh!

try:
    greet(123, 1)   # name should be str
except TypeError as e:
    print(e)
# Argument 'name' must be str, got int
```

---

**Exercise E-4.** Write a `lazy_partial` that defers argument binding until call time.

```python
class lazy_partial:
    """Like functools.partial but evaluates callables at call time."""
    
    def __init__(self, func, *args, **kwargs):
        self._func = func
        self._args = args
        self._kwargs = kwargs
    
    def __call__(self, *call_args, **call_kwargs):
        # Evaluate callable args at call time
        resolved_args = tuple(
            a() if callable(a) else a for a in self._args
        ) + call_args
        resolved_kwargs = {
            k: v() if callable(v) else v
            for k, v in self._kwargs.items()
        }
        resolved_kwargs.update(call_kwargs)
        return self._func(*resolved_args, **resolved_kwargs)

import time

def log_event(event, timestamp, *, level="INFO"):
    print(f"[{level}] {timestamp:.0f}: {event}")

# timestamp is evaluated fresh each call
lazy_log = lazy_partial(log_event, lambda: time.time(), level="DEBUG")

lazy_log("Server started")
time.sleep(0.01)
lazy_log("Request received")
```

---

**Exercise E-5.** Build a decorator factory `retry_with_backoff(times, backoff_factor)`.

```python
import time
from functools import wraps

def retry_with_backoff(times=3, backoff_factor=2.0, exceptions=(Exception,)):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            delay = 1.0
            for attempt in range(1, times + 1):
                try:
                    return func(*args, **kwargs)
                except exceptions as e:
                    if attempt == times:
                        raise
                    print(f"[Attempt {attempt}/{times}] Failed: {e}. Retrying in {delay:.1f}s")
                    time.sleep(delay)
                    delay *= backoff_factor
        return wrapper
    return decorator

import random

@retry_with_backoff(times=3, backoff_factor=2.0, exceptions=(ValueError,))
def flaky_service(n):
    if random.random() < 0.7:
        raise ValueError("Service unavailable")
    return f"Success: {n}"
```

---

**Exercise E-6.** Implement a mini `functools.reduce` that also supports short-circuit (like `any`/`all`).

```python
def reduce_while(func, iterable, initial=None, *, stop_if=None):
    """Reduce with optional early-exit condition."""
    it = iter(iterable)
    if initial is None:
        try:
            acc = next(it)
        except StopIteration:
            raise TypeError("reduce_while() of empty iterable with no initial value")
    else:
        acc = initial
    
    for item in it:
        acc = func(acc, item)
        if stop_if and stop_if(acc):
            return acc
    return acc

# Normal reduce
product = reduce_while(lambda a, x: a * x, [1, 2, 3, 4, 5])
print(product)   # 120

# Early exit when accumulator > 100
result = reduce_while(
    lambda a, x: a * x,
    [2, 3, 4, 5, 6, 7, 8],
    initial=1,
    stop_if=lambda acc: acc > 100
)
print(result)   # 120 (stops when 2*3*4*5 = 120 > 100)
```

---

**Exercise E-7.** Use `partial` and `kwargs` to build a composable HTTP middleware stack.

```python
from functools import partial

def middleware_stack(*middlewares):
    """Build a WSGI-style middleware stack."""
    def run(handler, request):
        def call_next(req, idx=0):
            if idx >= len(middlewares):
                return handler(req)
            mw = middlewares[idx]
            return mw(req, partial(call_next, idx=idx+1))
        return call_next(request)
    return run

def auth_middleware(request, next_handler):
    if not request.get("token"):
        return {"status": 401, "body": "Unauthorized"}
    return next_handler(request)

def logging_middleware(request, next_handler):
    print(f"  [LOG] {request.get('method', 'GET')} {request.get('path', '/')}")
    response = next_handler(request)
    print(f"  [LOG] Response: {response.get('status')}")
    return response

def rate_limit_middleware(request, next_handler):
    # Simplified: always allow
    return next_handler(request)

stack = middleware_stack(logging_middleware, auth_middleware, rate_limit_middleware)

def handler(request):
    return {"status": 200, "body": f"Hello, {request['user']}"}

print(stack(handler, {"method": "GET", "path": "/api", "token": "abc", "user": "Suresh"}))
print(stack(handler, {"method": "GET", "path": "/api"}))  # No token
```

---

**Exercise E-8.** Build a mutable `partial` that lets you update pre-filled arguments.

```python
class MutablePartial:
    def __init__(self, func, *args, **kwargs):
        self.func = func
        self.args = list(args)
        self.kwargs = dict(kwargs)
    
    def update(self, *args, **kwargs):
        if args:
            self.args = list(args)
        self.kwargs.update(kwargs)
        return self
    
    def __call__(self, *call_args, **call_kwargs):
        merged_kwargs = {**self.kwargs, **call_kwargs}
        return self.func(*self.args, *call_args, **merged_kwargs)
    
    def __repr__(self):
        return f"MutablePartial({self.func.__name__}, args={self.args}, kwargs={self.kwargs})"

def connect(host, port, *, timeout=30, ssl=False):
    return f"{'https' if ssl else 'http'}://{host}:{port} (timeout={timeout})"

conn = MutablePartial(connect, "localhost", 8080, timeout=10)
print(conn())
# http://localhost:8080 (timeout=10)

conn.update(timeout=60, ssl=True)
print(conn())
# https://localhost:8080 (timeout=60)
```

---

**Exercise E-9.** Implement `negate(predicate)` and `all_of(*predicates)` and `any_of(*predicates)` combinators.

```python
def negate(pred):
    return lambda *args, **kw: not pred(*args, **kw)

def all_of(*preds):
    return lambda *args, **kw: all(p(*args, **kw) for p in preds)

def any_of(*preds):
    return lambda *args, **kw: any(p(*args, **kw) for p in preds)

is_even    = lambda x: x % 2 == 0
is_positive = lambda x: x > 0
gt_ten     = lambda x: x > 10

is_odd              = negate(is_even)
positive_even_large = all_of(is_positive, is_even, gt_ten)
odd_or_large        = any_of(is_odd, gt_ten)

numbers = range(-5, 20)
print([x for x in numbers if positive_even_large(x)])  # [12, 14, 16, 18]
print([x for x in range(1, 10) if is_odd(x)])          # [1, 3, 5, 7, 9]
print([x for x in range(1, 20) if odd_or_large(x)])
# [1, 3, 5, 7, 9, 11, 12, 13, 14, 15, 16, 17, 18, 19]
```

---

**Exercise E-10.** Create a `trampolined` decorator that prevents stack overflow in recursive lambdas.

```python
class Thunk:
    def __init__(self, func, *args, **kwargs):
        self.func = func
        self.args = args
        self.kwargs = kwargs

def trampoline(func):
    def wrapper(*args, **kwargs):
        result = func(*args, **kwargs)
        while isinstance(result, Thunk):
            result = result.func(*result.args, **result.kwargs)
        return result
    return wrapper

@trampoline
def factorial(n, acc=1):
    if n <= 1:
        return acc
    return Thunk(factorial, n - 1, acc * n)

print(factorial(10))     # 3628800
print(factorial(1000))   # Works! (no RecursionError)
```

---

**Exercise E-11.** Build a `pipe_async` that chains sync and async functions.

```python
import asyncio
from functools import partial

def pipe_async(*steps):
    """Chain sync and async functions."""
    async def pipeline(value):
        for step in steps:
            if asyncio.iscoroutinefunction(step):
                value = await step(value)
            else:
                value = step(value)
        return value
    return pipeline

async def fetch_user(user_id):
    await asyncio.sleep(0.01)
    return {"id": user_id, "name": "Suresh", "score": 88}

def add_grade(user):
    score = user["score"]
    user["grade"] = "A" if score >= 90 else "B" if score >= 80 else "C"
    return user

async def save_user(user):
    await asyncio.sleep(0.01)
    print(f"  Saved: {user}")
    return user

process_user = pipe_async(fetch_user, add_grade, save_user)

asyncio.run(process_user(42))
# Saved: {'id': 42, 'name': 'Suresh', 'score': 88, 'grade': 'B'}
```

---

**Exercise E-12.** Implement `freeze(**defaults)` — creates a read-only snapshot of kwargs as a callable.

```python
class frozen_kwargs:
    """Immutable snapshot of kwargs that can be applied to any function."""
    
    __slots__ = ("_data",)
    
    def __init__(self, **kwargs):
        object.__setattr__(self, "_data", dict(kwargs))
    
    def __setattr__(self, key, value):
        raise AttributeError("frozen_kwargs is immutable")
    
    def apply(self, func, *args, **overrides):
        return func(*args, **{**self._data, **overrides})
    
    def merge(self, **extra):
        return frozen_kwargs(**{**self._data, **extra})
    
    def __repr__(self):
        return f"frozen_kwargs({self._data})"

db_config = frozen_kwargs(host="db.internal", port=5432, user="app")
read_config  = db_config.merge(database="readonly_db")
write_config = db_config.merge(database="write_db")

def connect(host, port, user, database):
    return f"postgres://{user}@{host}:{port}/{database}"

print(read_config.apply(connect))
# postgres://app@db.internal:5432/readonly_db
print(write_config.apply(connect))
# postgres://app@db.internal:5432/write_db
```

---

**Exercise E-13.** Write a `polymorphic` decorator that chooses implementation based on argument count.

```python
def polymorphic(func):
    implementations = {}
    
    def register(*arities):
        def decorator(impl):
            for n in arities:
                implementations[n] = impl
            return impl
        return decorator
    
    def wrapper(*args, **kwargs):
        n = len(args) + len(kwargs)
        impl = implementations.get(n) or implementations.get(-1)
        if not impl:
            raise TypeError(f"No implementation for {n} argument(s)")
        return impl(*args, **kwargs)
    
    wrapper.register = register
    return wrapper

@polymorphic
def area():
    pass

@area.register(1)
def _(r):
    import math
    return math.pi * r ** 2   # Circle

@area.register(2)
def _(w, h):
    return w * h   # Rectangle

@area.register(3)
def _(a, b, c):
    s = (a + b + c) / 2
    return (s*(s-a)*(s-b)*(s-c)) ** 0.5   # Triangle (Heron's)

print(round(area(5), 2))        # 78.54 (circle)
print(area(4, 6))               # 24 (rectangle)
print(round(area(3, 4, 5), 2))  # 6.0 (triangle)
```

---

**Exercise E-14.** Build a `SignalSlot` system (Qt-style) using first-class functions and `**kwargs`.

```python
from functools import partial

class Signal:
    def __init__(self, *arg_types):
        self._arg_types = arg_types
        self._slots = []
    
    def connect(self, slot, **fixed_kwargs):
        """Connect a slot, optionally pre-binding some kwargs."""
        if fixed_kwargs:
            slot = partial(slot, **fixed_kwargs)
        self._slots.append(slot)
        return lambda: self._slots.remove(slot)   # Returns disconnector
    
    def emit(self, *args, **kwargs):
        for slot in self._slots:
            slot(*args, **kwargs)

class Button:
    clicked = Signal(str)
    
    def __init__(self, label):
        self.label = label
    
    def press(self):
        self.clicked.emit(self.label)

def on_click(label, *, prefix="CLICK"):
    print(f"[{prefix}] Button '{label}' was pressed")

btn = Button("Submit")

disconnect = btn.clicked.connect(on_click)
btn.clicked.connect(on_click, prefix="LOG")

btn.press()
# [CLICK] Button 'Submit' was pressed
# [LOG] Button 'Submit' was pressed

disconnect()   # Remove first slot
btn.press()
# [LOG] Button 'Submit' was pressed
```

---

**Exercise E-15.** Implement a functional `Option` (Maybe monad) using first-class functions.

```python
class Option:
    """Functional Option/Maybe type using first-class functions."""
    
    def __init__(self, value):
        self._value = value
        self._has_value = value is not None
    
    @classmethod
    def some(cls, value):
        return cls(value)
    
    @classmethod
    def none(cls):
        return cls(None)
    
    def map(self, func):
        if not self._has_value:
            return Option.none()
        try:
            return Option.some(func(self._value))
        except Exception:
            return Option.none()
    
    def flat_map(self, func):
        if not self._has_value:
            return Option.none()
        return func(self._value)
    
    def get_or(self, default):
        return self._value if self._has_value else default
    
    def filter(self, predicate):
        if self._has_value and predicate(self._value):
            return self
        return Option.none()
    
    def __repr__(self):
        return f"Some({self._value!r})" if self._has_value else "None"

# Usage
from functools import partial

safe_div  = lambda a, b: a / b if b != 0 else None
safe_sqrt = lambda x: x ** 0.5 if x >= 0 else None

result = (Option.some("  42  ")
          .map(str.strip)
          .map(int)
          .filter(lambda x: x > 0)
          .map(partial(safe_div, 1000))
          .map(round)
          .get_or(-1))

print(result)   # 24

bad = (Option.some("abc")
       .map(int)   # raises ValueError → becomes None
       .map(lambda x: x * 2)
       .get_or(0))

print(bad)   # 0
```

---

## 15. Summary, Best Practices, and Common Pitfalls

### Summary

Python's function parameter system is rich and layered. Here is a complete mental model:

```
def func(
    pos_only_1, pos_only_2,    ← only by position (/)
    /,
    regular_1, regular_2=10,   ← positional OR keyword
    *args,                     ← any extra positional → tuple
    kw_only_1, kw_only_2=True, ← only by keyword (after *)
    **kwargs                   ← any extra keyword → dict
): ...
```

First-class functions, higher-order functions, lambdas, and partials form Python's **functional programming toolkit** — enabling composable, reusable, and expressive code.

---

### Best Practices

**Parameters:**
- Use positional-only (`/`) for parameters that are implementation details or math-like (e.g., `slope(x1, y1, x2, y2, /)`)
- Use keyword-only (`*`) for boolean flags, configuration options, and anything that needs explicit naming for safety
- Keep `*args` for genuinely variable-length inputs; use explicit parameters when the count is known
- Use `**kwargs` for pass-through options, config dictionaries, and forward-compatible APIs
- Never use mutable objects (`[]`, `{}`, `set()`) as default values — use `None` instead

**Lambdas:**
- Keep lambdas for single, simple expressions — if you need a name for it, use `def`
- Capture loop variables with default arguments: `lambda x, i=i: ...`
- Prefer list comprehensions over `map(lambda ...)` for readability

**Partial Functions:**
- Use `partial` when you repeatedly call a function with the same subset of arguments
- Use `functools.update_wrapper` or `functools.wraps` when metadata matters
- Build specialized APIs from general functions with `partial` rather than writing wrapper functions

**Higher-Order Functions:**
- Write HOFs that are pure (no side effects) when possible — easier to compose and test
- Document the expected signature of function arguments in docstrings
- Use `functools.wraps` in all decorators to preserve the original function's metadata

---

### Common Pitfalls

**Pitfall 1: Mutable Default Argument**

```python
# ❌ Bug
def append(item, lst=[]):
    lst.append(item)
    return lst

# ✅ Fix
def append(item, lst=None):
    if lst is None:
        lst = []
    lst.append(item)
    return lst
```

**Pitfall 2: Lambda Late Binding in Loops**

```python
# ❌ All print 9 (last value of i)
fns = [lambda: i**2 for i in range(10)]

# ✅ Capture early with default argument
fns = [lambda i=i: i**2 for i in range(10)]
```

**Pitfall 3: Forgetting *args is a Tuple (Immutable)**

```python
def f(*args):
    args.append(99)   # ❌ AttributeError: 'tuple' object has no attribute 'append'
    args = list(args)
    args.append(99)   # ✅
```

**Pitfall 4: Keyword-Only After **kwargs (SyntaxError)**

```python
# ❌ SyntaxError
def f(**kwargs, *, extra): ...

# ✅ kw-only BEFORE **kwargs
def f(*, extra, **kwargs): ...
```

**Pitfall 5: Overusing Lambda for Complex Logic**

```python
# ❌ Unreadable
process = lambda x: sorted([v for v in x.values() if v > 0], reverse=True)[0]

# ✅ Use a named function
def get_top_positive(data):
    positives = [v for v in data.values() if v > 0]
    return sorted(positives, reverse=True)[0] if positives else None
```

**Pitfall 6: Calling partial with Conflicting Fixed Args**

```python
from functools import partial

def f(a, b):
    return a - b

g = partial(f, b=10)
# g(a=5)     # ✅  → f(a=5, b=10) = -5
# g(5, b=20) # ✅  → f(5, b=20) = -15 (overrides)
# g(5, 20)   # ❌  TypeError — b already set as keyword, can't pass positionally
```

**Pitfall 7: Unpacking More Items Than Parameters**

```python
def add(a, b): return a + b

nums = [1, 2, 3]
add(*nums)   # ❌ TypeError: add() takes 2 positional arguments but 3 were given
add(*nums[:2])   # ✅
```

---

## 16. Learning Roadmap

```
┌─────────────────────────────────────────────────────────────────┐
│              PYTHON FUNCTIONS LEARNING ROADMAP                  │
├─────────────────────────────────────────────────────────────────┤
│ STAGE 1 — FOUNDATIONS (Week 1)                                  │
│  □ Positional parameters and default values                     │
│  □ *args and **kwargs basics                                    │
│  □ Argument unpacking (* and **)                                │
│  □ Practice: Calculator, string formatters, config builders     │
├─────────────────────────────────────────────────────────────────┤
│ STAGE 2 — SPECIAL SYNTAX (Week 2)                               │
│  □ Positional-only parameters (/)                               │
│  □ Keyword-only parameters (*)                                  │
│  □ Full parameter ordering rules                                │
│  □ Practice: Safe APIs, math functions, config validators       │
├─────────────────────────────────────────────────────────────────┤
│ STAGE 3 — FUNCTIONAL PROGRAMMING (Week 3)                       │
│  □ First-class functions (assign, pass, return, store)          │
│  □ Higher-order functions (map, filter, sorted, reduce)         │
│  □ Lambda functions (syntax, use cases, limitations)            │
│  □ Practice: Data pipelines, sorting systems, transform chains  │
├─────────────────────────────────────────────────────────────────┤
│ STAGE 4 — CLOSURES & DECORATORS (Week 4)                        │
│  □ Closures and captured variables                              │
│  □ Writing your own decorators                                  │
│  □ functools.wraps                                              │
│  □ Decorator factories                                          │
│  □ Practice: Timer, logger, retry, cache decorators             │
├─────────────────────────────────────────────────────────────────┤
│ STAGE 5 — PARTIAL & COMPOSITION (Week 5)                        │
│  □ functools.partial — basics and use cases                     │
│  □ Function composition and pipe patterns                       │
│  □ Currying and partial application                             │
│  □ Practice: API builders, transform pipelines, plugin systems  │
├─────────────────────────────────────────────────────────────────┤
│ STAGE 6 — ADVANCED PATTERNS (Week 6+)                           │
│  □ Trampoline recursion                                         │
│  □ Monadic patterns (Option/Result types)                       │
│  □ Signal/slot event systems                                    │
│  □ Async higher-order functions                                 │
│  □ Type-safe function wrappers                                  │
│  □ Read: PEP 3102, PEP 570, functools docs                      │
└─────────────────────────────────────────────────────────────────┘
```

### Recommended Resources

- **PEP 570** — Python Positional-Only Parameters (3.8+)
- **PEP 3102** — Keyword-Only Arguments
- **Python Docs** — `functools` module
- **Fluent Python** (Luciano Ramalho) — Chapters on Functions as Objects
- **Python Cookbook** (David Beazley) — Chapters on Metaprogramming

---

*End of Advanced Python Function Parameters Master Guide*
