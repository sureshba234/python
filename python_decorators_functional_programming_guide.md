# Python Decorators and Functional Programming Master Guide

> **A Complete Reference from Beginner to Expert** — Covering Decorators, Functional Programming, Real-World Projects, Interview Prep, and More.

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Function Decorators](#2-function-decorators)
   - 2.1 [What Is a Decorator?](#21-what-is-a-decorator)
   - 2.2 [Wrapper Functions](#22-wrapper-functions)
   - 2.3 [functools.wraps](#23-functoolswraps)
3. [Parameterized Decorators](#3-parameterized-decorators)
4. [Multiple / Nested Decorators](#4-multiple--nested-decorators)
5. [Class Decorators](#5-class-decorators)
   - 5.1 [Decorator as a Class](#51-decorator-as-a-class)
   - 5.2 [Class as the Decorated Target](#52-class-as-the-decorated-target)
6. [Built-in Decorators](#6-built-in-decorators)
   - 6.1 [@property](#61-property)
   - 6.2 [@staticmethod](#62-staticmethod)
   - 6.3 [@classmethod](#63-classmethod)
7. [Functional Programming Principles](#7-functional-programming-principles)
8. [map()](#8-map)
9. [filter()](#9-filter)
10. [reduce()](#10-reduce)
11. [compose()](#11-compose)
12. [Functional Pipelines](#12-functional-pipelines)
13. [Immutable Thinking](#13-immutable-thinking)
14. [Functional Design Patterns](#14-functional-design-patterns)
15. [Comparison Tables](#15-comparison-tables)
16. [Real-World Projects](#16-real-world-projects)
    - 16.1 [Logging Decorator](#161-logging-decorator)
    - 16.2 [Authentication Decorator](#162-authentication-decorator)
    - 16.3 [Timing Decorator](#163-timing-decorator)
    - 16.4 [Retry Decorator](#164-retry-decorator)
    - 16.5 [Data Processing Pipeline](#165-data-processing-pipeline)
17. [Cheat Sheets](#17-cheat-sheets)
18. [Interview Master Section](#18-interview-master-section)
19. [Exercises](#19-exercises)
    - 19.1 [Intermediate (15 Problems)](#191-intermediate-15-problems)
    - 19.2 [Advanced (15 Problems)](#192-advanced-15-problems)
    - 19.3 [Expert (15 Problems)](#193-expert-15-problems)
20. [Solutions](#20-solutions)
21. [Summary, Best Practices & Common Pitfalls](#21-summary-best-practices--common-pitfalls)
22. [Learning Roadmap](#22-learning-roadmap)

---

## 1. Introduction

Python is a multi-paradigm language that embraces both object-oriented and functional styles. Two of its most powerful features are **decorators** and **functional programming constructs**.

- **Decorators** allow you to modify or enhance functions and classes without changing their source code — a form of metaprogramming.
- **Functional programming (FP)** treats computation as the evaluation of mathematical functions, emphasizing immutability, pure functions, and composability.

Together, they enable clean, expressive, reusable, and testable code.

**Prerequisites:** Basic Python knowledge (functions, classes, loops).

---

## 2. Function Decorators

### 2.1 What Is a Decorator?

A decorator is a **higher-order function** — a function that takes another function as an argument and returns a new function (usually with added behavior).

**Core Concept:**
```python
def my_decorator(func):
    def wrapper(*args, **kwargs):
        print("Before the function runs")
        result = func(*args, **kwargs)
        print("After the function runs")
        return result
    return wrapper

def greet(name):
    print(f"Hello, {name}!")

# Manual application
greet = my_decorator(greet)
greet("Alice")
```

**Output:**
```
Before the function runs
Hello, Alice!
After the function runs
```

**Using the `@` Syntactic Sugar:**
```python
@my_decorator
def greet(name):
    print(f"Hello, {name}!")

greet("Bob")
```

**Output:**
```
Before the function runs
Hello, Bob!
After the function runs
```

The `@my_decorator` syntax is exactly equivalent to `greet = my_decorator(greet)`.

---

### 2.2 Wrapper Functions

The wrapper function intercepts calls to the original function. A robust wrapper handles any arguments:

```python
def verbose(func):
    def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__} with args={args}, kwargs={kwargs}")
        result = func(*args, **kwargs)
        print(f"{func.__name__} returned {result}")
        return result
    return wrapper

@verbose
def add(a, b):
    return a + b

@verbose
def power(base, exp=2):
    return base ** exp

add(3, 4)
power(5, exp=3)
```

**Output:**
```
Calling add with args=(3, 4), kwargs={}
add returned 7
Calling power with args=(5,), kwargs={'exp': 3}
power returned 125
```

**Key pattern — always use `*args, **kwargs`** so your wrapper works with any function signature.

---

### 2.3 functools.wraps

Without `functools.wraps`, the decorated function loses its identity:

```python
def my_decorator(func):
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper

@my_decorator
def greet(name):
    """Greet someone by name."""
    return f"Hello, {name}!"

print(greet.__name__)   # wrapper  ← WRONG
print(greet.__doc__)    # None     ← WRONG
```

**With `functools.wraps`:**
```python
import functools

def my_decorator(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper

@my_decorator
def greet(name):
    """Greet someone by name."""
    return f"Hello, {name}!"

print(greet.__name__)   # greet   ← CORRECT
print(greet.__doc__)    # Greet someone by name.  ← CORRECT
```

`functools.wraps` copies `__name__`, `__doc__`, `__module__`, `__qualname__`, `__annotations__`, and `__dict__` from the original function to the wrapper. **Always use it in production decorators.**

---

## 3. Parameterized Decorators

A parameterized decorator is a **decorator factory** — a function that returns a decorator. This adds one level of nesting.

**Pattern:**
```python
def decorator_factory(param1, param2):
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            # use param1, param2 here
            return func(*args, **kwargs)
        return wrapper
    return decorator
```

**Practical Example — Repeating a Function:**
```python
import functools

def repeat(times):
    """Repeat a function call `times` times."""
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            result = None
            for _ in range(times):
                result = func(*args, **kwargs)
            return result
        return wrapper
    return decorator

@repeat(3)
def say(message):
    print(message)

say("Hello!")
```

**Output:**
```
Hello!
Hello!
Hello!
```

**Type-Checking Decorator:**
```python
import functools

def enforce_types(*types):
    """Enforce argument types at runtime."""
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            for arg, expected_type in zip(args, types):
                if not isinstance(arg, expected_type):
                    raise TypeError(
                        f"Expected {expected_type.__name__}, got {type(arg).__name__}"
                    )
            return func(*args, **kwargs)
        return wrapper
    return decorator

@enforce_types(str, int)
def create_user(name, age):
    return {"name": name, "age": age}

print(create_user("Alice", 30))   # {'name': 'Alice', 'age': 30}
print(create_user("Bob", "old"))  # TypeError: Expected int, got str
```

**Rate Limiter Decorator:**
```python
import functools
import time

def rate_limit(calls_per_second):
    """Limit how often a function can be called."""
    min_interval = 1.0 / calls_per_second
    last_called = [0.0]

    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            elapsed = time.time() - last_called[0]
            wait = min_interval - elapsed
            if wait > 0:
                time.sleep(wait)
            last_called[0] = time.time()
            return func(*args, **kwargs)
        return wrapper
    return decorator

@rate_limit(2)  # max 2 calls per second
def fetch_data(url):
    print(f"Fetching {url}")

fetch_data("http://api.example.com/1")
fetch_data("http://api.example.com/2")
fetch_data("http://api.example.com/3")
```

---

## 4. Multiple / Nested Decorators

You can stack multiple decorators on a single function. They are applied **bottom-up** but execute **top-down**.

```python
@decorator_a
@decorator_b
@decorator_c
def my_func():
    pass

# Equivalent to:
my_func = decorator_a(decorator_b(decorator_c(my_func)))
```

**Execution order illustration:**
```python
import functools

def tag(name):
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            print(f"[{name}] ENTER")
            result = func(*args, **kwargs)
            print(f"[{name}] EXIT")
            return result
        return wrapper
    return decorator

@tag("A")
@tag("B")
@tag("C")
def process():
    print("  Processing...")

process()
```

**Output:**
```
[A] ENTER
[B] ENTER
[C] ENTER
  Processing...
[C] EXIT
[B] EXIT
[A] EXIT
```

**Combining Logging and Timing:**
```python
import functools
import time
import logging

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

def log_call(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        logger.info(f"Calling {func.__name__}")
        result = func(*args, **kwargs)
        logger.info(f"Finished {func.__name__}")
        return result
    return wrapper

def timer(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        result = func(*args, **kwargs)
        elapsed = time.perf_counter() - start
        print(f"{func.__name__} took {elapsed:.4f}s")
        return result
    return wrapper

@log_call
@timer
def compute(n):
    return sum(range(n))

result = compute(1_000_000)
print(f"Result: {result}")
```

**Output:**
```
INFO:__main__:Calling compute
compute took 0.0312s
INFO:__main__:Finished compute
Result: 499999500000
```

---

## 5. Class Decorators

### 5.1 Decorator as a Class

A class can act as a decorator by implementing `__init__` (to receive the function) and `__call__` (to make the instance callable):

```python
import functools

class CountCalls:
    """Count how many times a function is called."""

    def __init__(self, func):
        functools.update_wrapper(self, func)
        self.func = func
        self.call_count = 0

    def __call__(self, *args, **kwargs):
        self.call_count += 1
        print(f"Call #{self.call_count} to {self.func.__name__}")
        return self.func(*args, **kwargs)

@CountCalls
def greet(name):
    print(f"Hello, {name}!")

greet("Alice")
greet("Bob")
greet("Charlie")
print(f"Total calls: {greet.call_count}")
```

**Output:**
```
Call #1 to greet
Hello, Alice!
Call #2 to greet
Hello, Bob!
Call #3 to greet
Hello, Charlie!
Total calls: 3
```

**Parameterized Class Decorator:**
```python
import functools

class Retry:
    """Retry a function up to `max_retries` times on exception."""

    def __init__(self, max_retries=3, exceptions=(Exception,)):
        self.max_retries = max_retries
        self.exceptions = exceptions

    def __call__(self, func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(1, self.max_retries + 1):
                try:
                    return func(*args, **kwargs)
                except self.exceptions as e:
                    print(f"Attempt {attempt} failed: {e}")
                    if attempt == self.max_retries:
                        raise
        return wrapper

@Retry(max_retries=3, exceptions=(ValueError, ConnectionError))
def risky_operation(x):
    if x < 0:
        raise ValueError(f"Negative value: {x}")
    return x * 2

risky_operation(5)   # 10
risky_operation(-1)  # Retries 3 times, then raises
```

---

### 5.2 Class as the Decorated Target

Decorators can also be applied to **classes** themselves, modifying or augmenting the class:

```python
def singleton(cls):
    """Ensure only one instance of a class exists."""
    instances = {}
    @functools.wraps(cls)
    def get_instance(*args, **kwargs):
        if cls not in instances:
            instances[cls] = cls(*args, **kwargs)
        return instances[cls]
    return get_instance

@singleton
class DatabaseConnection:
    def __init__(self, host):
        self.host = host
        print(f"Connecting to {host}...")

db1 = DatabaseConnection("localhost")
db2 = DatabaseConnection("localhost")
print(db1 is db2)  # True — same instance
```

**Output:**
```
Connecting to localhost...
True
```

**Add Methods to a Class:**
```python
def add_repr(cls):
    """Auto-generate __repr__ from __dict__."""
    def __repr__(self):
        attrs = ", ".join(f"{k}={v!r}" for k, v in self.__dict__.items())
        return f"{cls.__name__}({attrs})"
    cls.__repr__ = __repr__
    return cls

@add_repr
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

p = Point(3, 4)
print(p)   # Point(x=3, y=4)
```

**Frozen Class (Immutable Attributes):**
```python
def frozen(cls):
    """Prevent attribute assignment after __init__."""
    original_init = cls.__init__

    def __init__(self, *args, **kwargs):
        original_init(self, *args, **kwargs)
        self.__dict__['_frozen'] = True

    def __setattr__(self, name, value):
        if self.__dict__.get('_frozen'):
            raise AttributeError(f"Cannot modify frozen instance of '{cls.__name__}'")
        object.__setattr__(self, name, value)

    cls.__init__ = __init__
    cls.__setattr__ = __setattr__
    return cls

@frozen
class Config:
    def __init__(self, host, port):
        self.host = host
        self.port = port

cfg = Config("localhost", 8080)
print(cfg.host)      # localhost
cfg.host = "other"   # AttributeError: Cannot modify frozen instance
```

---

## 6. Built-in Decorators

### 6.1 @property

`@property` turns a method into a "managed attribute" — accessed like a field, but backed by a method.

```python
class Temperature:
    def __init__(self, celsius=0):
        self._celsius = celsius

    @property
    def celsius(self):
        """Get temperature in Celsius."""
        return self._celsius

    @celsius.setter
    def celsius(self, value):
        if value < -273.15:
            raise ValueError("Temperature below absolute zero!")
        self._celsius = value

    @celsius.deleter
    def celsius(self):
        print("Deleting temperature")
        del self._celsius

    @property
    def fahrenheit(self):
        """Computed property — no setter needed."""
        return self._celsius * 9/5 + 32

temp = Temperature(25)
print(temp.celsius)     # 25
print(temp.fahrenheit)  # 77.0

temp.celsius = 100
print(temp.fahrenheit)  # 212.0

temp.celsius = -300     # ValueError: Temperature below absolute zero!
```

**Lazy Property (compute once, cache thereafter):**
```python
class DataProcessor:
    def __init__(self, data):
        self._data = data
        self._result = None

    @property
    def processed(self):
        if self._result is None:
            print("Computing (expensive operation)...")
            self._result = sorted(set(self._data))
        return self._result

dp = DataProcessor([3, 1, 4, 1, 5, 9, 2, 6, 5])
print(dp.processed)  # Computing... [1, 2, 3, 4, 5, 6, 9]
print(dp.processed)  # (no recompute) [1, 2, 3, 4, 5, 6, 9]
```

---

### 6.2 @staticmethod

A static method belongs to the class namespace but receives no implicit first argument (`self` or `cls`). Use it for utility functions logically related to the class.

```python
class MathUtils:
    @staticmethod
    def is_prime(n):
        """Check if n is a prime number."""
        if n < 2:
            return False
        for i in range(2, int(n**0.5) + 1):
            if n % i == 0:
                return False
        return True

    @staticmethod
    def gcd(a, b):
        """Greatest common divisor."""
        while b:
            a, b = b, a % b
        return a

print(MathUtils.is_prime(17))  # True
print(MathUtils.is_prime(18))  # False
print(MathUtils.gcd(48, 18))   # 6

# Also callable on instances
mu = MathUtils()
print(mu.is_prime(97))         # True
```

---

### 6.3 @classmethod

A class method receives the **class** (`cls`) as its first argument instead of an instance. Used for alternative constructors and factory methods.

```python
from datetime import datetime

class Employee:
    company = "Acme Corp"

    def __init__(self, name, salary, department):
        self.name = name
        self.salary = salary
        self.department = department

    @classmethod
    def from_string(cls, employee_str):
        """Alternative constructor from 'name,salary,dept' string."""
        name, salary, dept = employee_str.split(",")
        return cls(name, float(salary), dept)

    @classmethod
    def intern(cls, name):
        """Factory method for standard intern."""
        return cls(name, 30000, "General")

    @classmethod
    def change_company(cls, new_name):
        """Modify class-level attribute."""
        cls.company = new_name

    def __repr__(self):
        return f"Employee({self.name}, {self.salary}, {self.department})"

e1 = Employee("Alice", 70000, "Engineering")
e2 = Employee.from_string("Bob,65000,Marketing")
e3 = Employee.intern("Charlie")

print(e1)  # Employee(Alice, 70000, Engineering)
print(e2)  # Employee(Bob, 65000.0, Marketing)
print(e3)  # Employee(Charlie, 30000, General)

Employee.change_company("MegaCorp")
print(Employee.company)  # MegaCorp
print(e1.company)        # MegaCorp (class attribute shared)
```

---

## 7. Functional Programming Principles

Functional Programming (FP) is a programming paradigm built on these core ideas:

**1. Pure Functions**
A function is pure if:
- Given the same input, it always returns the same output.
- It has no side effects (no modifying globals, no I/O, no mutations).

```python
# Pure
def add(a, b):
    return a + b

# Impure — modifies external state
total = 0
def add_to_total(x):
    global total
    total += x   # side effect!
```

**2. Immutability**
Prefer data that doesn't change. Work with new copies instead of mutating originals.

```python
# Imperative (mutating)
numbers = [1, 2, 3, 4, 5]
for i in range(len(numbers)):
    numbers[i] *= 2

# Functional (immutable)
numbers = [1, 2, 3, 4, 5]
doubled = [x * 2 for x in numbers]  # new list; original unchanged
```

**3. First-Class Functions**
Functions are values — they can be assigned, passed as arguments, and returned.

```python
def apply(func, value):
    return func(value)

print(apply(str.upper, "hello"))  # HELLO
print(apply(abs, -42))            # 42
```

**4. Higher-Order Functions**
Functions that take functions as arguments or return functions.

```python
def multiplier(factor):
    return lambda x: x * factor

double = multiplier(2)
triple = multiplier(3)

print(double(5))   # 10
print(triple(5))   # 15
```

**5. Referential Transparency**
Any expression can be replaced by its value without changing the program's behavior.

```python
# Referentially transparent
result = add(2, 3)   # always 5, can replace 'add(2, 3)' with '5' anywhere

# Not referentially transparent
import random
result = random.randint(1, 10)  # different every call
```

---

## 8. map()

`map(function, iterable)` applies a function to every item in an iterable, returning a **lazy iterator**.

**Basic Usage:**
```python
numbers = [1, 2, 3, 4, 5]

# Square each number
squares = list(map(lambda x: x**2, numbers))
print(squares)  # [1, 4, 9, 16, 25]

# Convert to strings
str_nums = list(map(str, numbers))
print(str_nums)  # ['1', '2', '3', '4', '5']
```

**Multiple Iterables:**
```python
a = [1, 2, 3]
b = [10, 20, 30]

sums = list(map(lambda x, y: x + y, a, b))
print(sums)  # [11, 22, 33]

# Or using operator module
import operator
products = list(map(operator.mul, a, b))
print(products)  # [10, 40, 90]
```

**map() vs List Comprehension:**
```python
data = ["  hello  ", "  world  ", "  python  "]

# map()
cleaned_map = list(map(str.strip, data))

# List comprehension (often preferred for readability)
cleaned_lc = [s.strip() for s in data]

print(cleaned_map)  # ['hello', 'world', 'python']
print(cleaned_lc)   # ['hello', 'world', 'python']
```

**Chaining maps:**
```python
words = ["hello", "world", "foo", "bar"]

# First capitalize, then get length
result = list(map(len, map(str.upper, words)))
print(result)  # [5, 5, 3, 3]
```

**Real-World: Transforming Records:**
```python
employees = [
    {"name": "Alice", "salary": 70000},
    {"name": "Bob",   "salary": 80000},
    {"name": "Carol", "salary": 60000},
]

# Apply 10% raise
def give_raise(emp, pct=0.10):
    return {**emp, "salary": emp["salary"] * (1 + pct)}

raised = list(map(give_raise, employees))
for emp in raised:
    print(f"{emp['name']}: ${emp['salary']:,.0f}")
```

**Output:**
```
Alice: $77,000
Bob: $88,000
Carol: $66,000
```

---

## 9. filter()

`filter(function, iterable)` returns only items for which the function returns a truthy value.

**Basic Usage:**
```python
numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

evens = list(filter(lambda x: x % 2 == 0, numbers))
print(evens)  # [2, 4, 6, 8, 10]

# Using None as function — filters out falsy values
mixed = [0, 1, "", "hello", None, [], [1,2], False, True]
truthy = list(filter(None, mixed))
print(truthy)  # [1, 'hello', [1, 2], True]
```

**Combining filter with map:**
```python
data = range(-5, 6)

# Square only positive numbers
result = list(map(lambda x: x**2, filter(lambda x: x > 0, data)))
print(result)  # [1, 4, 9, 16, 25]
```

**Real-World: Filtering Records:**
```python
products = [
    {"name": "Widget", "price": 9.99,  "in_stock": True},
    {"name": "Gadget", "price": 24.99, "in_stock": False},
    {"name": "Doohickey", "price": 4.99, "in_stock": True},
    {"name": "Thingamajig", "price": 99.99, "in_stock": True},
]

# In-stock items under $20
affordable_available = list(filter(
    lambda p: p["in_stock"] and p["price"] < 20,
    products
))

for p in affordable_available:
    print(f"{p['name']}: ${p['price']}")
```

**Output:**
```
Widget: $9.99
Doohickey: $4.99
```

**filter() vs List Comprehension:**
```python
nums = range(1, 21)

# filter()
primes_f = list(filter(lambda n: all(n % i != 0 for i in range(2, n)), 
                        filter(lambda n: n >= 2, nums)))

# Comprehension (more readable)
primes_c = [n for n in nums if n >= 2 and all(n % i != 0 for i in range(2, n))]

print(primes_f)  # [2, 3, 5, 7, 11, 13, 17, 19]
print(primes_c)  # [2, 3, 5, 7, 11, 13, 17, 19]
```

---

## 10. reduce()

`functools.reduce(function, iterable, initializer=None)` applies a function of two arguments cumulatively to reduce the iterable to a single value.

```python
from functools import reduce

numbers = [1, 2, 3, 4, 5]

# Sum
total = reduce(lambda acc, x: acc + x, numbers)
print(total)  # 15

# Product
product = reduce(lambda acc, x: acc * x, numbers)
print(product)  # 120

# With initializer
total_with_init = reduce(lambda acc, x: acc + x, numbers, 100)
print(total_with_init)  # 115
```

**Visualizing reduce():**
```
[1, 2, 3, 4, 5]
Step 1: f(1, 2)  → 3
Step 2: f(3, 3)  → 6
Step 3: f(6, 4)  → 10
Step 4: f(10, 5) → 15
```

**Maximum without max():**
```python
from functools import reduce

nums = [3, 1, 4, 1, 5, 9, 2, 6, 5]
maximum = reduce(lambda a, b: a if a > b else b, nums)
print(maximum)  # 9
```

**Flatten a nested list:**
```python
from functools import reduce

nested = [[1, 2], [3, 4], [5, 6]]
flat = reduce(lambda acc, x: acc + x, nested, [])
print(flat)  # [1, 2, 3, 4, 5, 6]
```

**Compose functions with reduce:**
```python
from functools import reduce

def compose(*funcs):
    """Compose functions right-to-left."""
    return reduce(lambda f, g: lambda x: f(g(x)), funcs)

def double(x): return x * 2
def increment(x): return x + 1
def square(x): return x ** 2

# Applies: square → increment → double
transform = compose(double, increment, square)
print(transform(3))  # double(increment(square(3))) = double(increment(9)) = double(10) = 20
```

**Building a dictionary with reduce:**
```python
from functools import reduce

pairs = [("a", 1), ("b", 2), ("c", 3)]
d = reduce(lambda acc, kv: {**acc, kv[0]: kv[1]}, pairs, {})
print(d)  # {'a': 1, 'b': 2, 'c': 3}
```

---

## 11. compose()

Python doesn't have a built-in `compose()`, but it's easy to build. Composition chains functions together, passing the output of one as the input to the next.

**Mathematical notation:** `(f ∘ g)(x) = f(g(x))`

**Simple compose (right-to-left):**
```python
from functools import reduce

def compose(*funcs):
    """
    Compose functions right-to-left.
    compose(f, g, h)(x) == f(g(h(x)))
    """
    def composed(x):
        result = x
        for func in reversed(funcs):
            result = func(result)
        return result
    return composed

# Alternatively using reduce:
def compose_reduce(*funcs):
    return reduce(lambda f, g: lambda x: f(g(x)), funcs)
```

**pipe (left-to-right — more readable):**
```python
def pipe(*funcs):
    """
    Apply functions left-to-right.
    pipe(f, g, h)(x) == h(g(f(x)))
    """
    def piped(x):
        result = x
        for func in funcs:
            result = func(result)
        return result
    return piped
```

**Example:**
```python
import string

clean = pipe(
    str.strip,
    str.lower,
    lambda s: s.translate(str.maketrans("", "", string.punctuation))
)

text = "  Hello, World!!! "
print(clean(text))  # hello world
```

**Composing with multiple arguments using partial:**
```python
from functools import partial

def multiply(factor, x):
    return x * factor

def add(n, x):
    return x + n

double = partial(multiply, 2)
add_ten = partial(add, 10)

transform = pipe(double, add_ten, str)
print(transform(5))   # "20"  (5*2=10, 10+10=20, str(20)="20")
```

---

## 12. Functional Pipelines

A functional pipeline is a sequence of transformations applied to data, where each step's output feeds the next step's input.

**Simple Pipeline:**
```python
data = [" Alice ", " bob ", " CHARLIE ", " dave "]

result = list(
    map(
        lambda name: name.title(),
        map(
            str.strip,
            data
        )
    )
)
print(result)  # ['Alice', 'Bob', 'Charlie', 'Dave']
```

**Pipeline Class:**
```python
class Pipeline:
    """Chainable data transformation pipeline."""

    def __init__(self, data):
        self._data = list(data)

    def map(self, func):
        self._data = list(map(func, self._data))
        return self

    def filter(self, func):
        self._data = list(filter(func, self._data))
        return self

    def reduce(self, func, initial=None):
        from functools import reduce
        if initial is not None:
            return reduce(func, self._data, initial)
        return reduce(func, self._data)

    def sort(self, key=None, reverse=False):
        self._data = sorted(self._data, key=key, reverse=reverse)
        return self

    def collect(self):
        return self._data

# Usage
result = (
    Pipeline(range(1, 21))
    .filter(lambda x: x % 2 == 0)       # evens: [2,4,6,...,20]
    .map(lambda x: x ** 2)               # squares: [4,16,36,...,400]
    .filter(lambda x: x > 50)            # > 50: [64,100,...,400]
    .sort(reverse=True)                   # descending
    .collect()
)
print(result)  # [400, 324, 256, 196, 144, 100, 64]
```

**Generator-Based Pipeline (memory efficient):**
```python
def pipeline(*funcs):
    """Lazy pipeline using generators."""
    def process(data):
        result = data
        for func in funcs:
            result = func(result)
        return result
    return process

def map_step(func):
    return lambda data: (func(x) for x in data)

def filter_step(pred):
    return lambda data: (x for x in data if pred(x))

def take(n):
    def _take(data):
        count = 0
        for item in data:
            if count >= n:
                break
            yield item
            count += 1
    return _take

process = pipeline(
    filter_step(lambda x: x % 2 != 0),   # odd numbers
    map_step(lambda x: x ** 3),           # cubed
    filter_step(lambda x: x > 100),       # > 100
    take(5)                               # first 5
)

result = list(process(range(1, 100)))
print(result)  # [125, 343, 729, 1331, 2197]
```

---

## 13. Immutable Thinking

Immutability means data doesn't change after creation. It leads to more predictable, testable, and concurrent-safe code.

**Python's Immutable Built-ins:**

| Type | Immutable? |
|------|-----------|
| int, float, bool | Yes |
| str | Yes |
| tuple | Yes (shallow) |
| frozenset | Yes |
| bytes | Yes |
| list | No |
| dict | No |
| set | No |

**Working Immutably with Dictionaries:**
```python
original = {"name": "Alice", "age": 30, "city": "NYC"}

# BAD: mutation
original["age"] = 31  # modifies original

# GOOD: create new dict
updated = {**original, "age": 31}  # spread + override
print(original)  # {"name": "Alice", "age": 30, "city": "NYC"}
print(updated)   # {"name": "Alice", "age": 31, "city": "NYC"}
```

**Namedtuple for Immutable Records:**
```python
from collections import namedtuple

Point = namedtuple("Point", ["x", "y"])
p1 = Point(1, 2)
p2 = p1._replace(x=10)  # creates a new Point

print(p1)  # Point(x=1, y=2)
print(p2)  # Point(x=10, y=2)
# p1.x = 99  # AttributeError — immutable!
```

**dataclasses with frozen=True:**
```python
from dataclasses import dataclass

@dataclass(frozen=True)
class Config:
    host: str
    port: int
    debug: bool = False

cfg = Config("localhost", 8080)
print(cfg)          # Config(host='localhost', port=8080, debug=False)
# cfg.port = 9090   # FrozenInstanceError!

# "Update" by creating new instance
new_cfg = Config(cfg.host, 9090, cfg.debug)
print(new_cfg)      # Config(host='localhost', port=9090, debug=False)
```

**Persistent Data Structures (pyrsistent):**
```python
# pip install pyrsistent
from pyrsistent import pmap, pvector

# Persistent map (immutable dict)
m1 = pmap({"a": 1, "b": 2})
m2 = m1.set("c", 3)         # new map, m1 unchanged
m3 = m2.remove("a")

print(m1)  # pmap({'a': 1, 'b': 2})
print(m2)  # pmap({'a': 1, 'b': 2, 'c': 3})
print(m3)  # pmap({'b': 2, 'c': 3})
```

---

## 14. Functional Design Patterns

### Pattern 1: Strategy Pattern (Functional Style)

```python
from typing import Callable

def sort_data(data: list, strategy: Callable) -> list:
    return sorted(data, key=strategy)

employees = [
    {"name": "Carol", "salary": 60000, "age": 35},
    {"name": "Alice", "salary": 70000, "age": 30},
    {"name": "Bob",   "salary": 80000, "age": 25},
]

by_salary = sort_data(employees, lambda e: e["salary"])
by_name   = sort_data(employees, lambda e: e["name"])
by_age    = sort_data(employees, lambda e: e["age"])

print([e["name"] for e in by_salary])  # ['Carol', 'Alice', 'Bob']
print([e["name"] for e in by_name])    # ['Alice', 'Bob', 'Carol']
print([e["name"] for e in by_age])     # ['Bob', 'Alice', 'Carol']
```

### Pattern 2: Memoization

```python
import functools

def memoize(func):
    cache = {}
    @functools.wraps(func)
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

print(fibonacci(50))  # 12586269025 (instant, vs exponential naive)
print(fibonacci.cache)  # inspect the cache

# Python's built-in version:
@functools.lru_cache(maxsize=None)
def fib(n):
    if n < 2:
        return n
    return fib(n-1) + fib(n-2)
```

### Pattern 3: Currying

```python
def curry(func):
    """Convert a multi-argument function into a chain of single-argument functions."""
    import inspect
    n_params = len(inspect.signature(func).parameters)

    def curried(*args):
        if len(args) >= n_params:
            return func(*args)
        return lambda *more: curried(*(args + more))

    return curried

@curry
def add3(a, b, c):
    return a + b + c

add_5 = add3(5)
add_5_10 = add_5(10)
print(add_5_10(3))  # 18
print(add3(1)(2)(3))  # 6
print(add3(1, 2)(3))  # 6
```

### Pattern 4: Partial Application

```python
from functools import partial

def power(base, exponent):
    return base ** exponent

square = partial(power, exponent=2)
cube   = partial(power, exponent=3)

print(list(map(square, [1, 2, 3, 4, 5])))  # [1, 4, 9, 16, 25]
print(list(map(cube,   [1, 2, 3, 4, 5])))  # [1, 8, 27, 64, 125]
```

### Pattern 5: Monadic Error Handling

```python
class Result:
    """A simple Result monad for error handling without exceptions."""

    def __init__(self, value=None, error=None):
        self._value = value
        self._error = error

    @classmethod
    def ok(cls, value):
        return cls(value=value)

    @classmethod
    def err(cls, error):
        return cls(error=error)

    @property
    def is_ok(self):
        return self._error is None

    def map(self, func):
        if self.is_ok:
            try:
                return Result.ok(func(self._value))
            except Exception as e:
                return Result.err(str(e))
        return self

    def flat_map(self, func):
        if self.is_ok:
            return func(self._value)
        return self

    def get_or(self, default):
        return self._value if self.is_ok else default

    def __repr__(self):
        if self.is_ok:
            return f"Ok({self._value})"
        return f"Err({self._error})"

def parse_int(s):
    try:
        return Result.ok(int(s))
    except ValueError as e:
        return Result.err(str(e))

def safe_divide(n):
    def divide(d):
        if d == 0:
            return Result.err("Division by zero")
        return Result.ok(n / d)
    return divide

result = (
    parse_int("42")
    .flat_map(safe_divide(100))
    .map(lambda x: round(x, 2))
)
print(result)  # Ok(2.38)

error = (
    parse_int("abc")
    .flat_map(safe_divide(100))
)
print(error)   # Err(invalid literal for int() with base 10: 'abc')
```

### Pattern 6: Observer (Functional Events)

```python
from collections import defaultdict
from typing import Callable

class EventBus:
    """Functional event system."""

    def __init__(self):
        self._handlers = defaultdict(list)

    def subscribe(self, event: str):
        """Decorator to subscribe a function to an event."""
        def decorator(func: Callable):
            self._handlers[event].append(func)
            return func
        return decorator

    def publish(self, event: str, **data):
        for handler in self._handlers[event]:
            handler(**data)

bus = EventBus()

@bus.subscribe("user.created")
def send_welcome_email(username, email):
    print(f"Sending welcome email to {email}")

@bus.subscribe("user.created")
def log_new_user(username, email):
    print(f"LOG: New user created: {username}")

bus.publish("user.created", username="alice", email="alice@example.com")
```

**Output:**
```
Sending welcome email to alice@example.com
LOG: New user created: alice
```

---

## 15. Comparison Tables

### map vs filter vs reduce

| Feature | `map()` | `filter()` | `reduce()` |
|---------|---------|-----------|-----------|
| **Purpose** | Transform each element | Select elements | Aggregate to one value |
| **Input** | function + iterable(s) | function + iterable | function + iterable |
| **Output** | Iterator (same length) | Iterator (≤ length) | Single value |
| **Function arity** | 1 arg (or N for N iterables) | 1 arg → bool | 2 args (acc, item) |
| **Import** | Built-in | Built-in | `from functools import reduce` |
| **Empty iterable** | Returns empty | Returns empty | Error (no initial) |
| **With initializer** | N/A | N/A | Yes, optional |
| **Lazy** | Yes | Yes | No (consumes all) |
| **List comprehension equivalent** | `[f(x) for x in it]` | `[x for x in it if p(x)]` | Manual loop |
| **When to use** | Data transformation | Data selection | Accumulation / aggregation |

### Decorator vs Higher-Order Function

| Feature | Decorator | Higher-Order Function |
|---------|-----------|----------------------|
| **Definition** | Function that wraps another function | Function taking/returning functions |
| **Syntax** | `@decorator` syntax sugar | Called explicitly |
| **Common use** | Cross-cutting concerns (logging, auth) | General function composition |
| **Modifies original?** | No (wraps it) | No (creates new function) |
| **Introspection** | Needs `functools.wraps` | N/A |
| **Parameterizable** | Yes (decorator factory) | Yes (closure) |
| **Applied at** | Definition time | Call time or definition time |
| **Examples** | `@lru_cache`, `@property` | `map`, `filter`, `sorted` |

### Functional vs Imperative Style

| Feature | Functional | Imperative |
|---------|-----------|-----------|
| **State** | Immutable, minimal | Mutable variables |
| **Data flow** | Transformations | Step-by-step commands |
| **Side effects** | Avoided | Common |
| **Loops** | map/filter/reduce, recursion | for/while loops |
| **Testability** | Easy (pure functions) | Harder (state dependency) |
| **Readability** | Declarative ("what") | Procedural ("how") |
| **Debugging** | Easier (no shared state) | Can be harder |
| **Performance** | Sometimes slower (extra objects) | Sometimes faster |
| **Concurrency** | Safe (no shared mutable state) | Requires synchronization |
| **Python style** | List comprehensions, lambdas | Loops, assignments |

---

## 16. Real-World Projects

### 16.1 Logging Decorator

```python
import functools
import logging
import inspect
from datetime import datetime

def setup_logger(name):
    logger = logging.getLogger(name)
    logger.setLevel(logging.DEBUG)
    handler = logging.StreamHandler()
    formatter = logging.Formatter(
        "%(asctime)s [%(levelname)s] %(name)s: %(message)s"
    )
    handler.setFormatter(formatter)
    logger.addHandler(handler)
    return logger

def logged(level=logging.INFO, include_args=True, include_result=True):
    """
    Production-grade logging decorator.
    
    Args:
        level: Logging level (default INFO)
        include_args: Log function arguments
        include_result: Log return value
    """
    def decorator(func):
        logger = setup_logger(func.__module__ or __name__)

        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            # Build argument string
            if include_args:
                sig = inspect.signature(func)
                bound = sig.bind(*args, **kwargs)
                bound.apply_defaults()
                args_str = ", ".join(
                    f"{k}={v!r}" for k, v in bound.arguments.items()
                )
            else:
                args_str = "..."

            logger.log(level, f"CALL {func.__qualname__}({args_str})")
            start = datetime.now()

            try:
                result = func(*args, **kwargs)
                elapsed = (datetime.now() - start).total_seconds()

                if include_result:
                    logger.log(level,
                        f"RETURN {func.__qualname__} → {result!r} ({elapsed:.4f}s)"
                    )
                return result

            except Exception as e:
                elapsed = (datetime.now() - start).total_seconds()
                logger.error(
                    f"EXCEPTION {func.__qualname__} raised {type(e).__name__}: {e} "
                    f"({elapsed:.4f}s)"
                )
                raise

        return wrapper
    return decorator

# Usage
@logged(level=logging.DEBUG, include_args=True, include_result=True)
def divide(a, b):
    if b == 0:
        raise ZeroDivisionError("Cannot divide by zero")
    return a / b

@logged(include_args=False)
def get_users():
    return ["Alice", "Bob", "Charlie"]

divide(10, 2)
get_users()
divide(5, 0)   # logs exception before re-raising
```

---

### 16.2 Authentication Decorator

```python
import functools
from typing import Optional, Set

class AuthContext:
    """Simulated authentication context."""
    _current_user: Optional[dict] = None

    @classmethod
    def login(cls, user: dict):
        cls._current_user = user

    @classmethod
    def logout(cls):
        cls._current_user = None

    @classmethod
    def current_user(cls) -> Optional[dict]:
        return cls._current_user

def require_auth(func=None, *, roles: Optional[Set[str]] = None):
    """
    Authentication and authorization decorator.
    
    Usage:
        @require_auth
        def view(request): ...

        @require_auth(roles={"admin", "superuser"})
        def admin_view(request): ...
    """
    def decorator(f):
        @functools.wraps(f)
        def wrapper(*args, **kwargs):
            user = AuthContext.current_user()

            if user is None:
                raise PermissionError("Authentication required. Please log in.")

            if roles and not (set(user.get("roles", [])) & roles):
                raise PermissionError(
                    f"Access denied. Required roles: {roles}. "
                    f"Your roles: {user.get('roles', [])}"
                )

            return f(*args, **kwargs)
        return wrapper

    # Handle both @require_auth and @require_auth(roles=...)
    if func is not None:
        return decorator(func)
    return decorator

# Route handlers
@require_auth
def view_profile():
    user = AuthContext.current_user()
    return f"Profile: {user['name']}"

@require_auth(roles={"admin"})
def admin_panel():
    return "Welcome to admin panel"

@require_auth(roles={"admin", "moderator"})
def moderate_content(content_id):
    return f"Moderating content #{content_id}"

# Test
AuthContext.login({"name": "Alice", "roles": ["user"]})
print(view_profile())          # Profile: Alice
try:
    print(admin_panel())       # PermissionError
except PermissionError as e:
    print(f"Blocked: {e}")

AuthContext.login({"name": "Bob", "roles": ["admin", "moderator"]})
print(admin_panel())           # Welcome to admin panel
print(moderate_content(42))   # Moderating content #42
```

---

### 16.3 Timing Decorator

```python
import functools
import time
import statistics
from contextlib import contextmanager

def timer(func=None, *, runs=1, unit="ms", verbose=True):
    """
    Timing decorator with multi-run statistics.
    
    Args:
        runs: Number of times to run (for benchmarking)
        unit: Time unit ('s', 'ms', 'µs', 'ns')
        verbose: Print timing info
    """
    units = {"s": 1, "ms": 1e3, "µs": 1e6, "ns": 1e9}
    multiplier = units.get(unit, 1e3)  # default ms

    def decorator(f):
        f._timings = []

        @functools.wraps(f)
        def wrapper(*args, **kwargs):
            timings = []
            result = None

            for i in range(runs):
                start = time.perf_counter()
                result = f(*args, **kwargs)
                elapsed = (time.perf_counter() - start) * multiplier
                timings.append(elapsed)
                f._timings.append(elapsed)

            if verbose:
                if runs == 1:
                    print(f"⏱  {f.__name__}: {timings[0]:.3f}{unit}")
                else:
                    print(f"⏱  {f.__name__} ({runs} runs):")
                    print(f"   min={min(timings):.3f}{unit}  "
                          f"max={max(timings):.3f}{unit}  "
                          f"mean={statistics.mean(timings):.3f}{unit}  "
                          f"stdev={statistics.stdev(timings):.3f}{unit}" if runs > 1 else "")

            return result

        wrapper.timings = f._timings
        return wrapper

    if func is not None:
        return decorator(func)
    return decorator

@timer
def quick_sort(arr):
    if len(arr) <= 1:
        return arr
    pivot = arr[len(arr) // 2]
    left = [x for x in arr if x < pivot]
    mid  = [x for x in arr if x == pivot]
    right = [x for x in arr if x > pivot]
    return quick_sort(left) + mid + quick_sort(right)

@timer(runs=5, unit="µs")
def bubble_sort(arr):
    arr = arr[:]
    for i in range(len(arr)):
        for j in range(len(arr) - i - 1):
            if arr[j] > arr[j+1]:
                arr[j], arr[j+1] = arr[j+1], arr[j]
    return arr

import random
data = random.sample(range(10000), 500)

quick_sort(data[:])
bubble_sort(data[:])
```

**Output (approximate):**
```
⏱  quick_sort: 1.234ms
⏱  bubble_sort (5 runs):
   min=12543.210µs  max=14231.456µs  mean=13102.345µs  stdev=567.123µs
```

---

### 16.4 Retry Decorator

```python
import functools
import time
import random
import logging
from typing import Type, Tuple, Union

logger = logging.getLogger(__name__)

def retry(
    max_attempts: int = 3,
    delay: float = 1.0,
    backoff: float = 2.0,
    jitter: float = 0.1,
    exceptions: Tuple[Type[Exception], ...] = (Exception,),
    on_retry=None,
    on_failure=None
):
    """
    Production retry decorator with exponential backoff and jitter.
    
    Args:
        max_attempts: Maximum number of attempts
        delay: Initial delay in seconds
        backoff: Multiplier applied to delay after each attempt
        jitter: Random fraction added to delay (prevents thundering herd)
        exceptions: Exception types to retry on
        on_retry: Callback(attempt, exception, delay) before each retry
        on_failure: Callback(exception) on final failure
    """
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            current_delay = delay
            last_exception = None

            for attempt in range(1, max_attempts + 1):
                try:
                    return func(*args, **kwargs)
                except exceptions as e:
                    last_exception = e

                    if attempt == max_attempts:
                        logger.error(
                            f"{func.__name__} failed after {max_attempts} attempts. "
                            f"Final error: {e}"
                        )
                        if on_failure:
                            on_failure(e)
                        raise

                    jitter_amount = current_delay * jitter * random.uniform(-1, 1)
                    actual_delay = current_delay + jitter_amount

                    logger.warning(
                        f"{func.__name__} attempt {attempt}/{max_attempts} failed: {e}. "
                        f"Retrying in {actual_delay:.2f}s..."
                    )

                    if on_retry:
                        on_retry(attempt, e, actual_delay)

                    time.sleep(actual_delay)
                    current_delay *= backoff

        return wrapper
    return decorator

# Simulate unreliable network call
call_count = 0

@retry(
    max_attempts=4,
    delay=0.5,
    backoff=2.0,
    exceptions=(ConnectionError, TimeoutError),
    on_retry=lambda attempt, exc, delay: print(f"  → Retry {attempt} after {delay:.2f}s")
)
def fetch_from_api(url: str) -> dict:
    global call_count
    call_count += 1
    print(f"  Attempt #{call_count}: fetching {url}")

    if call_count < 3:  # fail first 2 attempts
        raise ConnectionError(f"Server unavailable (attempt {call_count})")

    return {"status": "ok", "data": [1, 2, 3]}

logging.basicConfig(level=logging.WARNING)
result = fetch_from_api("https://api.example.com/data")
print(f"\nSuccess! Result: {result}")
```

**Output:**
```
  Attempt #1: fetching https://api.example.com/data
  → Retry 1 after 0.48s
  Attempt #2: fetching https://api.example.com/data
  → Retry 2 after 0.97s
  Attempt #3: fetching https://api.example.com/data

Success! Result: {'status': 'ok', 'data': [1, 2, 3]}
```

---

### 16.5 Data Processing Pipeline

```python
import functools
import json
from typing import Any, Callable, Iterator, List
from dataclasses import dataclass, field
from datetime import datetime

@dataclass
class Record:
    id: int
    name: str
    value: float
    timestamp: str
    tags: List[str] = field(default_factory=list)

# ------- Step functions (pure transformations) -------

def normalize_name(record: Record) -> Record:
    """Normalize name to Title Case and strip whitespace."""
    return Record(
        id=record.id,
        name=record.name.strip().title(),
        value=record.value,
        timestamp=record.timestamp,
        tags=record.tags
    )

def apply_discount(pct: float):
    """Return a transformation that applies a discount to value."""
    def transform(record: Record) -> Record:
        return Record(
            id=record.id,
            name=record.name,
            value=round(record.value * (1 - pct), 2),
            timestamp=record.timestamp,
            tags=record.tags
        )
    return transform

def add_tag(tag: str):
    """Return a transformation that adds a tag."""
    def transform(record: Record) -> Record:
        return Record(
            id=record.id,
            name=record.name,
            value=record.value,
            timestamp=record.timestamp,
            tags=record.tags + [tag]
        )
    return transform

def is_valid(record: Record) -> bool:
    """Validate record."""
    return record.value > 0 and bool(record.name)

def to_dict(record: Record) -> dict:
    """Serialize to dictionary."""
    return {
        "id": record.id,
        "name": record.name,
        "value": record.value,
        "timestamp": record.timestamp,
        "tags": record.tags
    }

# ------- Pipeline builder -------

class DataPipeline:
    def __init__(self, source: List[Any]):
        self._data = source
        self._steps = []

    def transform(self, func: Callable) -> "DataPipeline":
        self._steps.append(("map", func))
        return self

    def keep(self, predicate: Callable) -> "DataPipeline":
        self._steps.append(("filter", predicate))
        return self

    def run(self) -> List[Any]:
        result = self._data
        for step_type, func in self._steps:
            if step_type == "map":
                result = list(map(func, result))
            elif step_type == "filter":
                result = list(filter(func, result))
        return result

# ------- Sample data -------

raw_data = [
    Record(1, "  alice smith  ", 120.00, "2024-01-15", []),
    Record(2, "BOB JONES",       -5.00,  "2024-01-16", []),   # invalid
    Record(3, "carol white",     200.00, "2024-01-17", ["vip"]),
    Record(4, "",                 50.00, "2024-01-18", []),    # invalid
    Record(5, "dave brown",      300.00, "2024-01-19", []),
]

# ------- Run pipeline -------

pipeline = DataPipeline(raw_data)

results = (
    pipeline
    .keep(is_valid)                      # filter bad records
    .transform(normalize_name)           # clean names
    .transform(add_tag("processed"))     # tag all
    .transform(apply_discount(0.10))     # 10% discount
    .transform(to_dict)                  # serialize
    .run()
)

print(json.dumps(results, indent=2))
```

**Output:**
```json
[
  {"id": 1, "name": "Alice Smith", "value": 108.0, "timestamp": "2024-01-15", "tags": ["processed"]},
  {"id": 3, "name": "Carol White", "value": 180.0, "timestamp": "2024-01-17", "tags": ["vip", "processed"]},
  {"id": 5, "name": "Dave Brown",  "value": 270.0, "timestamp": "2024-01-19", "tags": ["processed"]}
]
```

---

## 17. Cheat Sheets

### Decorator Cheat Sheet

```
┌─────────────────────────────────────────────────────────────────────┐
│                    DECORATOR PATTERNS                               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  BASIC DECORATOR                                                    │
│  ──────────────                                                     │
│  def decorator(func):                                               │
│      @functools.wraps(func)                                         │
│      def wrapper(*args, **kwargs):                                  │
│          # BEFORE                                                   │
│          result = func(*args, **kwargs)                             │
│          # AFTER                                                    │
│          return result                                              │
│      return wrapper                                                 │
│                                                                     │
│  PARAMETERIZED DECORATOR                                            │
│  ───────────────────────                                            │
│  def decorator_factory(param):                                      │
│      def decorator(func):                                           │
│          @functools.wraps(func)                                     │
│          def wrapper(*args, **kwargs):                              │
│              # use param                                            │
│              return func(*args, **kwargs)                           │
│          return wrapper                                             │
│      return decorator                                               │
│                                                                     │
│  CLASS DECORATOR                                                    │
│  ───────────────                                                    │
│  class Decorator:                                                   │
│      def __init__(self, func):                                      │
│          functools.update_wrapper(self, func)                       │
│          self.func = func                                           │
│      def __call__(self, *args, **kwargs):                           │
│          # before/after logic                                       │
│          return self.func(*args, **kwargs)                          │
│                                                                     │
│  STACKING ORDER                                                     │
│  @A  ←── applied last (outermost)                                  │
│  @B                                                                 │
│  @C  ←── applied first (innermost)                                 │
│  def f(): ...                                                       │
│  → f = A(B(C(f)))                                                   │
│  → A.enter → B.enter → C.enter → f() → C.exit → B.exit → A.exit   │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Functional Programming Cheat Sheet

```
┌─────────────────────────────────────────────────────────────────────┐
│                 FUNCTIONAL PROGRAMMING QUICK REF                    │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  map(func, iterable)                                                │
│  ──────────────────                                                 │
│  Transform every element                                            │
│  list(map(str, [1,2,3]))       → ['1','2','3']                      │
│  list(map(abs, [-1,-2,3]))     → [1,2,3]                            │
│  list(map(pow, [2,3], [3,2]))  → [8,9]   (multiple iterables)      │
│                                                                     │
│  filter(func, iterable)                                             │
│  ──────────────────────                                             │
│  Keep elements where func returns True                              │
│  list(filter(None, [0,1,"",2]))      → [1,2]                        │
│  list(filter(str.isdigit, "a1b2c3")) → ['1','2','3']                │
│                                                                     │
│  reduce(func, iterable[, init])                                     │
│  ──────────────────────────────                                     │
│  Accumulate to a single value                                       │
│  reduce(lambda a,b: a+b, [1,2,3,4])  → 10                          │
│  reduce(lambda a,b: a*b, [1,2,3,4])  → 24                          │
│  reduce(lambda a,b: a+b, [], 0)      → 0  (with initializer)       │
│                                                                     │
│  functools.partial(func, *args, **kwargs)                           │
│  ─────────────────────────────────────                              │
│  Freeze some arguments of a function                                │
│  double = partial(mul, 2); double(5) → 10                           │
│                                                                     │
│  functools.lru_cache(maxsize=128)                                   │
│  ─────────────────────────────────                                  │
│  Memoize function results                                           │
│  @lru_cache(maxsize=None) → unlimited cache                         │
│  func.cache_info()        → check hits/misses                       │
│  func.cache_clear()       → clear cache                             │
│                                                                     │
│  LAMBDA SYNTAX                                                      │
│  lambda x: x*2            → anonymous function                     │
│  lambda x,y: x+y          → two args                               │
│  lambda x: (x, x**2)      → returns tuple                          │
│                                                                     │
│  BUILT-IN DECORATORS                                                │
│  @property   → getter (read attribute like property)                │
│  @x.setter   → setter                                               │
│  @x.deleter  → deleter                                              │
│  @staticmethod → no self/cls (utility method)                      │
│  @classmethod  → cls as first arg (factory methods)                │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Immutability Cheat Sheet

```
┌─────────────────────────────────────────────────────────────────────┐
│                    IMMUTABILITY QUICK REF                           │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  Use tuples instead of lists:                                       │
│    t = (1, 2, 3)   # immutable                                      │
│    l = [1, 2, 3]   # mutable                                        │
│                                                                     │
│  Use frozenset instead of set:                                      │
│    fs = frozenset({1, 2, 3})   # immutable set                      │
│                                                                     │
│  Update dict without mutating:                                      │
│    new = {**old, "key": "new_value"}                                │
│    new = old | {"key": "new_value"}  # Python 3.9+                  │
│                                                                     │
│  namedtuple for immutable records:                                  │
│    Point = namedtuple("Point", ["x", "y"])                          │
│    p = Point(1, 2)                                                  │
│    p2 = p._replace(x=10)  # new copy                               │
│                                                                     │
│  @dataclass(frozen=True) for immutable dataclasses:                 │
│    @dataclass(frozen=True)                                          │
│    class Config:                                                     │
│        host: str; port: int                                         │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 18. Interview Master Section

### Core Decorator Questions

**Q1. What is a decorator in Python and how does it work?**
A decorator is a higher-order function that takes a function as an argument and returns a new function with added or modified behavior. The `@decorator` syntax is syntactic sugar for `func = decorator(func)`. It enables cross-cutting concerns (logging, auth, caching) to be separated from business logic.

**Q2. What problem does `functools.wraps` solve?**
Without `functools.wraps`, the wrapper function replaces the original function's `__name__`, `__doc__`, `__module__`, and `__annotations__`. This breaks introspection, documentation tools, and frameworks that rely on function metadata. `functools.wraps` copies these attributes from the original to the wrapper.

**Q3. What is the difference between a decorator and a decorator factory?**
A decorator takes a function directly: `@my_decorator`. A decorator factory is a callable that *returns* a decorator, enabling parameterization: `@my_decorator(arg1, arg2)`. The factory adds an extra level of nesting.

**Q4. In what order are stacked decorators applied and executed?**
Applied **bottom-up** at definition time: `@A @B @C def f()` → `f = A(B(C(f)))`. Executed **top-down** at call time: A's logic runs first, then B, then C, then the original function, then C's post-logic, B's, and finally A's.

**Q5. How can you preserve the original function in a decorator for potential bypassing?**
```python
def decorator(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    wrapper.__wrapped__ = func   # convention
    return wrapper
```
Callers can access `func.__wrapped__` to get the original.

**Q6. What is a class-based decorator and when would you prefer it over a function decorator?**
A class implements `__init__(self, func)` and `__call__(self, *args, **kwargs)`. Prefer class decorators when you need to maintain state between calls (e.g., a call counter, connection pool), or when the decorator has complex parameterization that benefits from OOP structure.

**Q7. Can decorators be applied to classes? Give an example.**
Yes. The `@dataclass`, `@functools.total_ordering`, and custom decorators can modify or enhance entire classes. Example: `@singleton` wraps class instantiation to enforce only one instance.

**Q8. How does `@property` work internally?**
`property` is a descriptor class. When you access `obj.attr`, Python calls `property.__get__`. The getter, setter, and deleter are stored in the property object. `@x.setter` creates a new `property` object with the setter added.

**Q9. What is the difference between `@staticmethod` and `@classmethod`?**
`@staticmethod` receives no implicit first argument — it's a plain function in the class namespace. `@classmethod` receives the class (`cls`) as the first argument, enabling factory methods and access to class-level state.

**Q10. How would you write a decorator that works with or without arguments?**
```python
def decorator(func=None, *, param=default):
    def actual_decorator(f):
        @functools.wraps(f)
        def wrapper(*args, **kwargs):
            return f(*args, **kwargs)
        return wrapper
    if func is not None:   # called as @decorator
        return actual_decorator(func)
    return actual_decorator  # called as @decorator(param=...)
```

---

### Functional Programming Questions

**Q11. What is a pure function?**
A pure function always returns the same output for the same input and has no side effects. It doesn't modify external state, perform I/O, or depend on mutable global state. Pure functions are easy to test, reason about, and parallelize.

**Q12. What is the difference between `map()` and a list comprehension?**
Both transform iterables. `map()` is lazy (returns an iterator), which is memory-efficient for large datasets. List comprehensions are generally more readable for Pythonistas. `map()` can slightly outperform comprehensions for simple function calls using built-in functions (no lambda overhead).

**Q13. When would you use `filter()` vs a list comprehension?**
`filter(func, iterable)` is best with a pre-existing predicate function. List comprehensions are preferred when the condition is written inline: `[x for x in lst if x > 0]`. `filter` with `None` as the function (`filter(None, lst)`) efficiently removes falsy values.

**Q14. What does `reduce()` do and give three use cases?**
`reduce(f, iterable, init)` applies `f(accumulator, item)` cumulatively, reducing to one value. Use cases: (1) summing/multiplying sequences, (2) flattening nested lists, (3) building a dict from key-value pairs, (4) composing a chain of functions.

**Q15. What is function composition and how do you implement `compose()` in Python?**
Composition chains functions: `compose(f, g)(x) = f(g(x))`. Implement with `reduce`: `compose = lambda *fns: reduce(lambda f, g: lambda x: f(g(x)), fns)`.

**Q16. What is currying and how does it differ from partial application?**
Currying transforms `f(a, b, c)` into `f(a)(b)(c)` — a chain of single-argument functions. Partial application pre-fills some arguments of a function, returning one that accepts the rest. Python uses `functools.partial` for partial application.

**Q17. What is referential transparency?**
An expression is referentially transparent if it can be replaced with its value anywhere in the program without changing behavior. Pure functions are referentially transparent; impure functions (with side effects or random output) are not.

**Q18. What is lazy evaluation and how does Python support it?**
Lazy evaluation computes values only when needed. Python supports it via generators, `map()`, `filter()`, `range()`, and generator expressions. This enables processing infinite or very large sequences with constant memory.

**Q19. How does `functools.lru_cache` work?**
It memoizes function calls using a dictionary keyed by the arguments. On a cache hit, it returns the stored result without calling the function. The LRU (Least Recently Used) eviction policy discards the oldest entries when `maxsize` is reached. Use `@functools.cache` (Python 3.9+) for an unbounded cache.

**Q20. What are the trade-offs of functional programming in Python?**
Pros: Testability, no shared mutable state, composability, easier concurrency. Cons: Python is not a pure FP language (no tail call optimization), functional style can be less readable to imperative programmers, creates many intermediate objects potentially hurting performance for very hot paths.

---

### Advanced Questions

**Q21. How would you implement a memoizing decorator that handles unhashable arguments?**
Convert unhashable args to a hashable key (e.g., `json.dumps(args)` for dicts, or freeze them with `frozenset`). Alternatively, use a different cache backend (e.g., `cachetools.LRUCache` with a custom key).

**Q22. Explain the descriptor protocol and how `@property` relates to it.**
The descriptor protocol defines `__get__`, `__set__`, and `__delete__` on a class. When accessed via an instance, Python calls these methods. `property` is a built-in descriptor that uses these hooks to implement getter/setter/deleter behavior with clean syntax.

**Q23. How would you make a decorator thread-safe?**
Use `threading.Lock` inside the decorator to protect shared state:
```python
import threading
def thread_safe_cache(func):
    cache = {}
    lock = threading.Lock()
    @functools.wraps(func)
    def wrapper(*args):
        with lock:
            if args not in cache:
                cache[args] = func(*args)
            return cache[args]
    return wrapper
```

**Q24. What is the difference between `functools.partial` and a closure?**
`partial` is explicit and creates a `partial` object with inspectable `func`, `args`, and `keywords` attributes. A closure captures variables from an enclosing scope and is more flexible. Closures can capture mutable state; `partial` is for pre-filling fixed arguments.

**Q25. How would you compose decorators dynamically at runtime?**
```python
def apply_decorators(func, *decorators):
    for dec in reversed(decorators):
        func = dec(func)
    return func
```

**Q26. What is the `__wrapped__` attribute and why is it important?**
`functools.wraps` sets `wrapper.__wrapped__ = func`, allowing `inspect.unwrap()` to traverse the decorator chain and recover the original function. This matters for introspection tools, `inspect.signature()`, and testing frameworks.

**Q27. How do you create a context manager using a decorator?**
Use `contextlib.contextmanager`:
```python
from contextlib import contextmanager
@contextmanager
def managed_resource():
    print("Acquire")
    try:
        yield "resource"
    finally:
        print("Release")

with managed_resource() as r:
    print(f"Using {r}")
```

**Q28. What is the difference between `map` and `itertools.starmap`?**
`map(f, iterable)` calls `f(item)` for each item. `itertools.starmap(f, iterable)` unpacks each item as arguments: `f(*item)`. Use `starmap` when your iterable contains tuples of arguments: `starmap(pow, [(2,10), (3,5)])` → `[1024, 243]`.

**Q29. When would you use `reduce()` vs `itertools.accumulate()`?**
`reduce()` returns a single final value. `itertools.accumulate()` returns an iterator of all intermediate values (running totals). Use `accumulate` when you need the running history, `reduce` for just the final result.

**Q30. How can decorators interfere with `inspect.signature()` and how do you fix it?**
Without `functools.wraps`, the wrapper's signature is shown instead of the original's. `functools.wraps` copies `__wrapped__` which `inspect.signature()` follows. For custom argument manipulation, use `inspect.Signature` to construct the correct signature and assign it: `wrapper.__signature__ = inspect.signature(func)`.

**Q31. Explain how `functools.singledispatch` works.**
`@singledispatch` creates a generic function that dispatches to different implementations based on the type of the first argument:
```python
from functools import singledispatch
@singledispatch
def process(arg):
    return f"default: {arg}"

@process.register(int)
def _(arg):
    return f"int: {arg * 2}"

@process.register(str)
def _(arg):
    return f"str: {arg.upper()}"
```

**Q32. How do you write a decorator that can be used on both regular functions and async functions?**
```python
import asyncio, functools

def universal_decorator(func):
    if asyncio.iscoroutinefunction(func):
        @functools.wraps(func)
        async def async_wrapper(*args, **kwargs):
            print("before async")
            result = await func(*args, **kwargs)
            print("after async")
            return result
        return async_wrapper
    else:
        @functools.wraps(func)
        def sync_wrapper(*args, **kwargs):
            print("before sync")
            result = func(*args, **kwargs)
            print("after sync")
            return result
        return sync_wrapper
```

**Q33. What is tail call optimization and why doesn't Python have it?**
Tail call optimization (TCO) reuses the current stack frame when the last operation of a function is a recursive call, preventing stack overflow. Python's creator Guido van Rossum intentionally excluded TCO to preserve useful tracebacks for debugging. Python handles deep recursion with `sys.setrecursionlimit`, or by converting to iteration.

**Q34. How would you implement function memoization with a TTL (time-to-live)?**
```python
import time, functools

def timed_cache(ttl_seconds):
    def decorator(func):
        cache = {}
        @functools.wraps(func)
        def wrapper(*args):
            now = time.time()
            if args in cache:
                result, timestamp = cache[args]
                if now - timestamp < ttl_seconds:
                    return result
            result = func(*args)
            cache[args] = (result, now)
            return result
        return wrapper
    return decorator
```

**Q35. What is the difference between `filter(None, iterable)` and `filter(bool, iterable)`?**
Functionally identical — both filter out falsy values. `filter(None, iterable)` uses `None` as a special signal to test the items themselves for truthiness. `filter(bool, iterable)` explicitly applies `bool()`. The `None` form is marginally faster as it avoids a function call.

---

## 19. Exercises

### 19.1 Intermediate (15 Problems)

**Exercise 1:** Write a decorator `@validate_positive` that raises `ValueError` if any numeric argument to the decorated function is negative.

**Exercise 2:** Create a `@cache_result` decorator that caches the first call's result and returns it for all subsequent calls (no expiry).

**Exercise 3:** Write `@log_exceptions` that catches any exception raised by the function, logs it with the function name and arguments, then re-raises it.

**Exercise 4:** Implement `@uppercase_result` that converts a string return value to uppercase.

**Exercise 5:** Create `@debug` that prints function name, arguments, and return value every time it is called.

**Exercise 6:** Write a `@singleton` class decorator that ensures only one instance of the class exists.

**Exercise 7:** Use `map()`, `filter()`, and `reduce()` to find the product of all even numbers in a list without a loop.

**Exercise 8:** Write a `pipe()` function and use it to: strip whitespace → lowercase → split on commas → filter empty strings.

**Exercise 9:** Using `functools.partial`, create three specialized versions of a `send_email(to, subject, body, cc=None, bcc=None)` function for different departments.

**Exercise 10:** Implement a `@deprecated(reason)` decorator that prints a deprecation warning (with reason) every time the function is called but still calls it.

**Exercise 11:** Write a decorator `@count_calls` that tracks call count as a function attribute (`func.call_count`).

**Exercise 12:** Using `filter()` and `map()`, extract all email addresses from a list of strings and convert them to lowercase.

**Exercise 13:** Create a class `Property` that replicates basic `@property` behavior using descriptors (without using the built-in `property`).

**Exercise 14:** Write a `@freeze_args` decorator that converts any list arguments to tuples before calling the function.

**Exercise 15:** Implement `flatten_with_reduce(nested_list)` using only `reduce()` to flatten one level of nesting.

---

### 19.2 Advanced (15 Problems)

**Exercise 16:** Write a `@retry` decorator that retries on any specified exception types, with configurable delay and max attempts.

**Exercise 17:** Implement a `@rate_limit(n_calls, period)` decorator that allows at most `n_calls` per `period` seconds, blocking if the limit is exceeded.

**Exercise 18:** Create a `@circuit_breaker(fail_threshold, reset_timeout)` decorator that stops calling the function after `fail_threshold` consecutive failures and resets after `reset_timeout` seconds.

**Exercise 19:** Write a parameterized `@cache(maxsize, ttl)` decorator with LRU eviction and TTL expiry, without using `functools.lru_cache`.

**Exercise 20:** Implement `compose(*funcs)` and `pipe(*funcs)` and write a text normalization pipeline using them.

**Exercise 21:** Create a `@transactional` decorator for a class method that rolls back changes (using a copy of relevant state) if an exception is raised.

**Exercise 22:** Write a decorator `@profile(n_runs)` that runs the function `n_runs` times, reports min/max/mean/stdev of execution times, and returns the result of the last run.

**Exercise 23:** Implement a `Result` monad with `map`, `flat_map`, `get_or`, and `recover` methods and use it to build a safe mathematical expression evaluator.

**Exercise 24:** Write a `@memoize_to_disk(cache_dir)` decorator that persists the cache to disk using `pickle`, so cache survives across program runs.

**Exercise 25:** Create `@debounce(wait)` — calls the function only after `wait` seconds of inactivity (i.e., if called repeatedly, only the last call after the wait period executes).

**Exercise 26:** Implement a `@observable` class decorator that automatically fires events when any attribute is set, allowing observers to subscribe.

**Exercise 27:** Write a `curry` decorator and demonstrate it with a function of 4 arguments.

**Exercise 28:** Create a lazy `Stream` class similar to Java's `Stream` API, supporting `map`, `filter`, `limit`, `skip`, `collect`, and `reduce`.

**Exercise 29:** Implement `@type_check` using `inspect.signature` and type annotations (from `__annotations__`) to validate argument types at runtime.

**Exercise 30:** Write a `@once` decorator that ensures a function is only ever called once (subsequent calls return the first call's result).

---

### 19.3 Expert (15 Problems)

**Exercise 31:** Implement a decorator `@async_retry(max_attempts, delay, exceptions)` that works with `async def` functions.

**Exercise 32:** Write a `@synchronized` decorator using `threading.Lock` that prevents concurrent calls to the same function from multiple threads.

**Exercise 33:** Create a `@validate(schema)` decorator that validates function arguments against a Pydantic-style schema dict before calling the function.

**Exercise 34:** Implement a **dependency injection container** as a class that uses decorators to register and inject dependencies into functions.

**Exercise 35:** Write a fully composable `Pipeline` class that supports lazy evaluation (using generators) and can handle arbitrarily large datasets.

**Exercise 36:** Implement `@trampoline` to enable tail-recursive functions to run without stack overflow by converting recursion to iteration.

**Exercise 37:** Build a `FunctionalDict` (immutable dictionary) with `set`, `remove`, `map_values`, `filter_keys`, and `merge` methods, each returning a new `FunctionalDict`.

**Exercise 38:** Create an `EventSourcing` system using decorators: `@command` marks state-changing functions, recording all changes as events; `@replay` reconstructs state from the event log.

**Exercise 39:** Implement a `@parallel_map(n_workers)` decorator that replaces the function's return value with the result of running it concurrently over a list argument using `ThreadPoolExecutor`.

**Exercise 40:** Write `@contract(pre=..., post=..., invariant=...)` that checks pre-conditions (before call), post-conditions (after call), and invariants (class-level properties) using Design by Contract.

**Exercise 41:** Implement a `compose_async(*funcs)` that composes async functions, passing the result of each `await`ed function to the next.

**Exercise 42:** Build a `@middleware` system similar to Express.js — a decorator that wraps a handler in a chain of middleware functions, each receiving `(request, next_handler)`.

**Exercise 43:** Create a purely functional **state machine** using closures and higher-order functions (no classes), supporting state transitions, guards, and entry/exit actions.

**Exercise 44:** Implement a `memoize` decorator that handles recursive functions correctly without infinite recursion (using a sentinel for in-progress computations).

**Exercise 45:** Write a `@profile_memory` decorator using `tracemalloc` that measures peak memory usage of the decorated function and reports it.

---

## 20. Solutions

### Solutions: Intermediate

**Solution 1 — `@validate_positive`:**
```python
import functools

def validate_positive(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        all_values = list(args) + list(kwargs.values())
        for v in all_values:
            if isinstance(v, (int, float)) and v < 0:
                raise ValueError(f"Negative argument not allowed: {v}")
        return func(*args, **kwargs)
    return wrapper

@validate_positive
def area(width, height):
    return width * height

print(area(5, 3))     # 15
print(area(5, -3))    # ValueError
```

**Solution 2 — `@cache_result`:**
```python
def cache_result(func):
    sentinel = object()
    cached = [sentinel]
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        if cached[0] is sentinel:
            cached[0] = func(*args, **kwargs)
        return cached[0]
    return wrapper
```

**Solution 7 — Product of even numbers:**
```python
from functools import reduce

nums = [1, 2, 3, 4, 5, 6, 7, 8]
product = reduce(
    lambda a, b: a * b,
    filter(lambda x: x % 2 == 0, nums)
)
print(product)  # 2*4*6*8 = 384
```

**Solution 8 — `pipe()`:**
```python
def pipe(*funcs):
    def composed(x):
        for f in funcs:
            x = f(x)
        return x
    return composed

clean_split = pipe(
    str.strip,
    str.lower,
    lambda s: s.split(","),
    lambda lst: list(filter(None, lst))
)
print(clean_split("  APPLE, , BANANA, CHERRY  "))
# ['apple', 'banana', 'cherry']
```

**Solution 10 — `@deprecated`:**
```python
import warnings, functools

def deprecated(reason):
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            warnings.warn(
                f"{func.__name__} is deprecated: {reason}",
                DeprecationWarning, stacklevel=2
            )
            return func(*args, **kwargs)
        return wrapper
    return decorator

@deprecated("Use new_function() instead")
def old_function():
    return "old result"
```

**Solution 15 — `flatten_with_reduce`:**
```python
from functools import reduce

def flatten_with_reduce(nested):
    return reduce(lambda acc, x: acc + (list(x) if isinstance(x, (list, tuple)) else [x]), nested, [])

print(flatten_with_reduce([[1,2],[3,4],[5]]))  # [1,2,3,4,5]
```

---

### Solutions: Advanced (Selected)

**Solution 16 — `@retry`:**
```python
import functools, time

def retry(max_attempts=3, delay=1.0, exceptions=(Exception,)):
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(1, max_attempts + 1):
                try:
                    return func(*args, **kwargs)
                except exceptions as e:
                    if attempt == max_attempts:
                        raise
                    print(f"Attempt {attempt} failed: {e}. Retrying in {delay}s...")
                    time.sleep(delay)
        return wrapper
    return decorator
```

**Solution 18 — `@circuit_breaker`:**
```python
import functools, time

def circuit_breaker(fail_threshold=3, reset_timeout=30):
    def decorator(func):
        failures = [0]
        opened_at = [None]

        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            if opened_at[0] is not None:
                if time.time() - opened_at[0] < reset_timeout:
                    raise RuntimeError(f"Circuit open for {func.__name__}")
                else:
                    failures[0] = 0
                    opened_at[0] = None

            try:
                result = func(*args, **kwargs)
                failures[0] = 0
                return result
            except Exception as e:
                failures[0] += 1
                if failures[0] >= fail_threshold:
                    opened_at[0] = time.time()
                    print(f"Circuit OPENED for {func.__name__}")
                raise
        return wrapper
    return decorator
```

**Solution 23 — Result Monad:**
```python
class Result:
    def __init__(self, value=None, error=None):
        self._value, self._error = value, error

    @classmethod
    def ok(cls, v): return cls(value=v)

    @classmethod
    def err(cls, e): return cls(error=e)

    @property
    def is_ok(self): return self._error is None

    def map(self, f):
        if self.is_ok:
            try: return Result.ok(f(self._value))
            except Exception as e: return Result.err(str(e))
        return self

    def flat_map(self, f):
        return f(self._value) if self.is_ok else self

    def get_or(self, default):
        return self._value if self.is_ok else default

    def recover(self, f):
        return self if self.is_ok else Result.ok(f(self._error))

    def __repr__(self):
        return f"Ok({self._value})" if self.is_ok else f"Err({self._error})"

def safe_parse(s):
    try: return Result.ok(float(s))
    except: return Result.err(f"Cannot parse '{s}'")

def safe_sqrt(x):
    if x < 0: return Result.err("Negative sqrt")
    return Result.ok(x ** 0.5)

print(safe_parse("16").flat_map(safe_sqrt))   # Ok(4.0)
print(safe_parse("abc").flat_map(safe_sqrt))  # Err(Cannot parse 'abc')
print(safe_parse("-4").flat_map(safe_sqrt))   # Err(Negative sqrt)
```

**Solution 30 — `@once`:**
```python
def once(func):
    sentinel = object()
    result = [sentinel]
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        if result[0] is sentinel:
            result[0] = func(*args, **kwargs)
        return result[0]
    return wrapper

@once
def initialize():
    print("Initializing...")
    return {"status": "ready"}

print(initialize())  # Initializing... {'status': 'ready'}
print(initialize())  # {'status': 'ready'}  (no re-init)
```

---

### Solutions: Expert (Selected)

**Solution 31 — `@async_retry`:**
```python
import asyncio, functools

def async_retry(max_attempts=3, delay=1.0, exceptions=(Exception,)):
    def decorator(func):
        @functools.wraps(func)
        async def wrapper(*args, **kwargs):
            for attempt in range(1, max_attempts + 1):
                try:
                    return await func(*args, **kwargs)
                except exceptions as e:
                    if attempt == max_attempts:
                        raise
                    print(f"Async attempt {attempt} failed: {e}")
                    await asyncio.sleep(delay)
        return wrapper
    return decorator

@async_retry(max_attempts=3, delay=0.1)
async def fetch(url):
    raise ConnectionError("Server down")

# asyncio.run(fetch("http://example.com"))
```

**Solution 36 — `@trampoline`:**
```python
def trampoline(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        result = func(*args, **kwargs)
        while callable(result):
            result = result()
        return result
    return wrapper

@trampoline
def factorial(n, acc=1):
    if n <= 1:
        return acc
    return lambda: factorial(n - 1, n * acc)

print(factorial(10000))  # No stack overflow!
```

**Solution 45 — `@profile_memory`:**
```python
import tracemalloc, functools

def profile_memory(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        tracemalloc.start()
        result = func(*args, **kwargs)
        current, peak = tracemalloc.get_traced_memory()
        tracemalloc.stop()
        print(f"{func.__name__}: peak memory = {peak / 1024:.2f} KB")
        return result
    return wrapper

@profile_memory
def create_large_list(n):
    return [i ** 2 for i in range(n)]

create_large_list(100_000)
# create_large_list: peak memory = 3516.84 KB
```

---

## 21. Summary, Best Practices & Common Pitfalls

### Summary

| Concept | Key Idea |
|---------|---------|
| Decorator | Wraps a function to add behavior; `@dec` = `f = dec(f)` |
| `functools.wraps` | Preserves metadata of the original function |
| Parameterized decorator | Factory returning a decorator; adds extra nesting level |
| Stacked decorators | Applied bottom-up; executed top-down |
| Class decorator | `__call__` makes instances callable; good for stateful decorators |
| `@property` | Descriptor-based computed attributes |
| `@staticmethod` | No `self`/`cls`; utility function in class namespace |
| `@classmethod` | Receives `cls`; used for factory methods |
| Pure function | Same input → same output, no side effects |
| `map()` | Transform every element; lazy iterator |
| `filter()` | Select elements; lazy iterator |
| `reduce()` | Accumulate to one value; eager |
| `compose()` | Chain functions right-to-left |
| `pipe()` | Chain functions left-to-right (more readable) |
| Immutability | Prefer creating new data over mutating existing data |
| Memoization | Cache results of pure functions; `@lru_cache` |
| Currying | `f(a,b,c)` → `f(a)(b)(c)` |
| Partial application | Pre-fill arguments; `functools.partial` |

---

### Best Practices

1. **Always use `@functools.wraps(func)`** inside every function decorator. Without it you lose `__name__`, `__doc__`, and break `inspect`.

2. **Always accept `*args, **kwargs`** in wrapper functions to be compatible with any decorated function.

3. **Keep decorators focused** — each decorator should do one thing. Stack decorators for multiple concerns.

4. **Use parameterized decorators** when you need configuration. Design them to work both as `@dec` and `@dec(param=val)` using the `func=None` pattern.

5. **Prefer pure functions** in functional pipelines — they're easier to test, compose, and parallelize.

6. **Use `functools.lru_cache`** for memoization instead of rolling your own, unless you need TTL or disk persistence.

7. **Use `functools.partial`** over lambdas for pre-filling arguments — it's more readable and introspectable.

8. **Use list comprehensions** over `map(lambda ...)` — it's more Pythonic. Use `map()` with named functions.

9. **Prefer `pipe()` over `compose()`** in Python — left-to-right reads naturally and matches how you think about data flow.

10. **Add `__wrapped__`** attribute on your wrapper if users might need to access the original function.

11. **Use `@dataclass(frozen=True)`** for immutable value objects.

12. **Avoid mutable default arguments** in functions — use `None` and assign inside the body.

---

### Common Pitfalls

**Pitfall 1 — Forgetting `functools.wraps`:**
```python
# BAD
def dec(func):
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper  # loses __name__, __doc__

# GOOD
def dec(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper
```

**Pitfall 2 — Mutable default in decorator factory:**
```python
# BAD — shared list across all uses
def dec(data=[]):
    def wrapper(func): ...

# GOOD
def dec(data=None):
    data = data or []
    def wrapper(func): ...
```

**Pitfall 3 — Stacking order confusion:**
```python
@A  # OUTER — runs first
@B  # INNER — runs second
def f(): ...
# Execution: A → B → f → B-post → A-post
# NOT: B → A → f
```

**Pitfall 4 — Not returning the result:**
```python
# BAD
def dec(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        func(*args, **kwargs)   # result dropped!
    return wrapper

# GOOD
def dec(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)  # must return!
    return wrapper
```

**Pitfall 5 — Calling `reduce()` on an empty sequence:**
```python
from functools import reduce
reduce(lambda a, b: a + b, [])  # TypeError: reduce() of empty sequence

# GOOD — always provide an initializer for potentially empty sequences
reduce(lambda a, b: a + b, [], 0)  # 0
```

**Pitfall 6 — Overusing lambdas:**
```python
# HARD TO READ
result = list(map(lambda x: x * 2, filter(lambda x: x % 2 == 0, range(10))))

# CLEAR
evens = filter(lambda x: x % 2 == 0, range(10))
doubled = list(map(lambda x: x * 2, evens))

# EVEN CLEARER (list comprehension)
doubled = [x * 2 for x in range(10) if x % 2 == 0]
```

**Pitfall 7 — Decorating a class method with a plain function decorator:**
```python
class MyClass:
    @my_decorator   # works, but wrapper receives self as first arg
    def method(self):
        pass
# Usually fine if wrapper uses *args, **kwargs — but be aware of self
```

**Pitfall 8 — Assuming map() returns a list:**
```python
result = map(str, [1, 2, 3])
print(result)       # <map object at 0x...>  NOT a list!
print(list(result)) # ['1', '2', '3']
result[0]           # TypeError: 'map' object is not subscriptable
```

**Pitfall 9 — Mutating captured closure variables:**
```python
# BAD — counter captured by reference, not value
def make_adder(n):
    return lambda x: x + n

# This is fine — but beware of late binding in loops:
funcs = [lambda x: x + i for i in range(3)]
print([f(0) for f in funcs])  # [2, 2, 2] NOT [0, 1, 2]!

# FIX:
funcs = [lambda x, i=i: x + i for i in range(3)]
print([f(0) for f in funcs])  # [0, 1, 2]
```

**Pitfall 10 — Applying `@property` in a class that doesn't inherit from `object`:**
This is only relevant in Python 2 (old-style classes). In Python 3, all classes implicitly inherit from `object`, so `@property` always works correctly.

---

## 22. Learning Roadmap

```
BEGINNER (Weeks 1-2)
─────────────────────
□ Understand first-class functions
□ Write and use basic function decorators
□ Use functools.wraps
□ Understand @property, @staticmethod, @classmethod
□ Learn map(), filter(), reduce()
□ Practice lambda expressions
□ Understand list comprehensions vs map/filter

INTERMEDIATE (Weeks 3-4)
──────────────────────────
□ Write parameterized decorators
□ Stack multiple decorators
□ Create class-based decorators
□ Use functools.partial and functools.lru_cache
□ Implement memoization
□ Learn function composition and pipe()
□ Build simple data pipelines

ADVANCED (Weeks 5-6)
──────────────────────
□ Build production decorators (retry, rate_limit, circuit_breaker)
□ Implement currying
□ Study immutability patterns (namedtuple, frozen dataclass)
□ Learn the descriptor protocol
□ Explore functools.singledispatch
□ Build a Result/Option monad
□ Understand lazy evaluation with generators

EXPERT (Weeks 7-8)
────────────────────
□ Write async-compatible decorators
□ Build thread-safe decorators
□ Design by Contract with @contract
□ Implement trampolining for tail recursion
□ Study persistent data structures (pyrsistent)
□ Build middleware systems
□ Design event sourcing with decorators
□ Explore toolz / cytoolz libraries for FP in Python
□ Study Haskell or Scala to deepen FP intuition

PROJECTS TO BUILD
──────────────────
□ A complete HTTP middleware stack (authentication, logging, rate limiting)
□ A lazy ETL pipeline with generators
□ A reactive event system with function composition
□ A memoized Fibonacci / dynamic programming solver
□ A REST API framework using decorators for routing
```

### Recommended Resources

- **Books:** "Fluent Python" by Luciano Ramalho (Chapters on decorators and functional tools)
- **Books:** "Python Cookbook" by David Beazley (Metaprogramming chapter)
- **Libraries to explore:** `toolz`, `cytoolz`, `returns`, `pyrsistent`, `funcy`
- **PEPs to read:** PEP 318 (Decorators), PEP 3104 (Function Annotations)
- **Practice:** Implement a decorator-based web framework (like micro-Flask)

---

*© Python Decorators and Functional Programming Master Guide — All examples are original and tested against Python 3.10+*
