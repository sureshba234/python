<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" style="height:64px;margin-right:32px"/>

# Python Decorators Part 1: Foundations

## Table of Contents

1. [Introduction to Decorators](#section-1-introduction-to-decorators)
2. [Functions as First-Class Objects](#section-2-functions-as-first-class-objects)
3. [Higher-Order Functions](#section-3-higher-order-functions)
4. [Closures](#section-4-closures)
5. [Building Your First Decorator](#section-5-building-your-first-decorator)
6. [Decoration Process Step by Step](#section-6-decoration-process-step-by-step)
7. [Decorators with Arguments](#section-7-decorators-with-arguments)
8. [Preserving Metadata](#section-8-preserving-metadata)
9. [Decorators with Parameters](#section-9-decorators-with-parameters)
10. [Multiple Decorators](#section-10-multiple-decorators)
11. [Exercises](#section-11-exercises)
12. [Interview Questions](#section-12-interview-questions)
13. [Cheat Sheet \& Summary](#final-cheat-sheet-summary)

***

## Section 1 — Introduction to Decorators

### What is a Decorator?

A **decorator** is a function that takes another function (or class) as input, extends or modifies its behavior, and returns a new function without permanently modifying the original function's code.

```python
@decorator
def func():
    pass
```

This is syntactically equivalent to:

```python
func = decorator(func)
```


### Why Decorators Exist

Decorators solve the **code repetition problem**. Instead of writing the same setup/teardown code in multiple functions, you:

1. Write it once in a decorator
2. Apply it to multiple functions using `@`

### History of Decorators

- Introduced in **Python 2.4** (2004)
- Inspired by **Java annotations** and **C\# attributes**
- Became a cornerstone of **Python's functional programming** capabilities
- Enhanced in Python 3 with `functools.wraps`


### Problems Decorators Solve

| Problem | Without Decorator | With Decorator |
| :-- | :-- | :-- |
| Logging | Repeat `print()` in every function | Single `@log` decorator |
| Authentication | Check credentials everywhere | Single `@auth` decorator |
| Timing | Write timing code repeatedly | Single `@timer` decorator |
| Caching | Manual cache management | Single `@cache` decorator |

### Real-World Use Cases

1. **Logging**: Track function calls and arguments
2. **Authentication**: Check user permissions before executing
3. **Caching**: Store results to avoid recomputation
4. **Timing**: Measure execution time
5. **Retry Logic**: Automatically retry failed operations
6. **Rate Limiting**: Control how often functions execute
7. **Input Validation**: Check argument validity
8. **Transaction Management**: Wrap database operations

### Decorator Mental Model

```
┌─────────────────────────────────────────────┐
│           Original Function                 │
│           def original():                   │
│               print("Hello")                │
└─────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────┐
│           Decorator Function                │
│           def decorator(func):              │
│               def wrapper():                │
│                   print("Before")           │
│                   func()                    │
│                   print("After")            │
│               return wrapper                │
└─────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────┐
│           Decorated Function (Wrapper)      │
│           def wrapper():                    │
│               print("Before")               │
│               original()                    │
│               print("After")                │
└─────────────────────────────────────────────┘
```


### Execution Flow

```python
@my_decorator
def greet():
    print("Hello!")

# What Python does:
def greet():
    print("Hello!")

greet = my_decorator(greet)  # Function is replaced!

# Now calling greet() actually calls the wrapper
greet()  # Executes wrapper, not original
```

**Visual Execution Flow:**

```
Call greet()
    ↓
Wrapper executes
    ↓
Print "Before"
    ↓
Original greet() executes
    ↓
Print "Hello!"
    ↓
Print "After"
```


***

## Section 2 — Functions as First-Class Objects

### Function Objects

In Python, **functions are objects** just like integers, strings, or lists. They have:

- A type (`function`)
- An identity (`id()`)
- Attributes (`__name__`, `__doc__`, etc.)
- Can be stored in variables
- Can be passed as arguments
- Can be returned from other functions

```python
def greet():
    return "Hello"

print(type(greet))        # <class 'function'>
print(id(greet))          # Memory address
print(greet.__name__)     # 'greet'
```


### Assigning Functions

```python
def greet():
    return "Hello"

# Assign function to another variable
say_hello = greet

print(say_hello())  # "Hello"
print(say_hello.__name__)  # 'greet' (name doesn't change)
```


### Passing Functions

```python
def greet():
    return "Hello"

def execute(func):
    return func()

result = execute(greet)  # Pass function as argument
print(result)  # "Hello"
```


### Returning Functions

```python
def create_greeting():
    def greet():
        return "Hello from inner function!"
    return greet  # Return the function object

greeting_func = create_greeting()
print(greeting_func())  # "Hello from inner function!"
```


### Function References

```python
def add(x, y):
    return x + y

def subtract(x, y):
    return x - y

# Store functions in a list
operations = [add, subtract]

print(operations[0](5, 3))  # 8 (add)
print(operations[1](5, 3))  # 2 (subtract)
```


### Lexical Scoping

**Lexical scoping** means a function can access variables from the scope where it was **defined**, not where it's called.

```python
def outer():
    message = "Hello from outer"
    
    def inner():
        return message  # Accesses outer's variable
    
    return inner

func = outer()
print(func())  # "Hello from outer"
```


### Nested Functions

```python
def outer(x):
    def inner(y):
        return x + y
    return inner

add_five = outer(5)
print(add_five(3))  # 8 (5 + 3)
```


### Memory Model Explanation

```
┌─────────────────────────────────────────────┐
│         Outer Function Scope                │
│         message = "Hello"                   │
└─────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────┐
│         Inner Function Object               │
│         def inner():                        │
│             return message                  │
│         __closure__ → [message]             │
└─────────────────────────────────────────────┘
```

When `outer()` returns `inner`, the `inner` function object **keeps a reference** to `message` via its closure, even though `outer`'s scope has ended.

***

## Section 3 — Higher-Order Functions

### Definition

A **higher-order function** is a function that:

1. **Accepts functions as arguments**, OR
2. **Returns a function** as its result

### Functional Programming Foundations

Higher-order functions are central to **functional programming**:

- Treat functions as data
- Compose functions
- Avoid mutable state
- Use function composition


### Functions Accepting Functions

```python
def apply_operation(func, x, y):
    return func(x, y)

def add(a, b):
    return a + b

def multiply(a, b):
    return a * b

print(apply_operation(add, 5, 3))      # 8
print(apply_operation(multiply, 5, 3)) # 15
```


### Functions Returning Functions

```python
def create_multiplier(factor):
    def multiplier(x):
        return x * factor
    return multiplier

double = create_multiplier(2)
triple = create_multiplier(3)

print(double(5))  # 10
print(triple(5))  # 15
```


### Relationship to Decorators

**Decorators are higher-order functions!** They return a wrapper function.

```python
def decorator(func):  # Higher-order: accepts function
    def wrapper():    # Higher-order: returns function
        return func()
    return wrapper
```


### Examples: map, filter, sorted

#### `map()` - Apply function to all elements

```python
numbers = [1, 2, 3, 4, 5]

# Using regular function
def square(x):
    return x ** 2

squared = map(square, numbers)
print(list(squared))  # [1, 4, 9, 16, 25]

# Using lambda
squared = map(lambda x: x ** 2, numbers)
print(list(squared))  # [1, 4, 9, 16, 25]
```


#### `filter()` - Filter elements based on function

```python
numbers = [1, 2, 3, 4, 5, 6]

# Keep only even numbers
evens = filter(lambda x: x % 2 == 0, numbers)
print(list(evens))  # [2, 4, 6]
```


#### `sorted()` - Sort with custom function

```python
words = ["apple", "banana", "cherry", "date"]

# Sort by length
sorted_by_length = sorted(words, key=len)
print(sorted_by_length)  # ['date', 'apple', 'banana', 'cherry']

# Sort by last character
sorted_by_last = sorted(words, key=lambda x: x[-1])
print(sorted_by_last)  # ['date', 'apple', 'cherry', 'banana']
```


### Higher-Order Function Visualization

```
┌─────────────────────────────────────────────┐
│           Higher-Order Function             │
│                                             │
│  Input:  func(A, B) → C                     │
│          data = [A1, A2, A3]                │
│                                             │
│  Process: Apply func to each element        │
│                                             │
│  Output: [C1, C2, C3]                       │
└─────────────────────────────────────────────┘
```


***

## Section 4 — Closures

### Closure Fundamentals

A **closure** is a function that:

1. Is defined inside another function
2. References variables from the outer function's scope
3. Can access those variables even after the outer function has returned
```python
def outer():
    x = 10
    def inner():
        return x  # inner references x from outer
    return inner

closure_func = outer()
print(closure_func())  # 10
```


### Free Variables

**Free variables** are variables referenced in a nested function that are defined in the outer function.

```python
def outer():
    message = "Hello"
    count = 0
    
    def inner():
        # message and count are free variables
        print(message)
        count += 1
        return count
    
    return inner
```


### Cell Objects

Python stores free variables in **cell objects**:

```python
def outer():
    x = 10
    y = 20
    def inner():
        return x + y
    return inner

func = outer()

# Inspect the closure
print(func.__closure__)  # (<cell object>, <cell object>)
print(func.__closure__[0].cell_contents)  # 10
print(func.__closure__[1].cell_contents)  # 20
```


### Inspecting Closures

```python
def outer():
    x = 10
    def inner():
        return x
    return inner

inner_func = outer()

# Free variable names
print(inner_func.__code__.co_freevars)  # ('x',)

# Closure object
print(inner_func.__closure__)  # (<cell at 0x...: int object at 0x...>)

# Cell contents
print(inner_func.__closure__[0].cell_contents)  # 10
```


### Closure Lifetime

Closures **persist** even after the outer function returns:

```python
def create_counter():
    count = 0
    def increment():
        count += 1
        return count
    return increment

counter = create_counter()
print(counter())  # 1
print(counter())  # 2
print(counter())  # 3

# count variable persists!
```

**Important**: For mutable free variables, you need to handle reassignment carefully:

```python
def create_counter():
    count = [0]  # Use list to make it mutable
    def increment():
        count[0] += 1
        return count[0]
    return increment

counter = create_counter()
print(counter())  # 1
print(counter())  # 2
```


### Closure Memory Behavior

```python
import sys

def outer():
    large_data = [i for i in range(10000)]
    def inner():
        return len(large_data)
    return inner

func = outer()

# large_data is still in memory via closure
print(func())  # 10000

# Check closure size
print(sys.getsizeof(func.__closure__))
```

**Memory visualization:**

```
┌─────────────────────────────────────────────┐
│         Outer Function Scope (ended)        │
│         large_data = [0, 1, 2, ..., 9999]   │
└─────────────────────────────────────────────┘
                    ↓ (reference kept)
┌─────────────────────────────────────────────┐
│         Inner Function Object               │
│         __closure__ → [cell(large_data)]    │
└─────────────────────────────────────────────┘
```

The `large_data` list persists in memory because the closure holds a reference to it.

***

## Section 5 — Building Your First Decorator

### Wrapper Functions

A **wrapper function** is the inner function that decorators return. It wraps the original function and adds behavior.

```python
def my_decorator(func):
    def wrapper():
        print("Before function call")
        func()  # Call original function
        print("After function call")
    return wrapper
```


### Decoration Process

```python
@my_decorator
def greet():
    print("Hello!")

# Equivalent to:
def greet():
    print("Hello!")

greet = my_decorator(greet)  # greet is now wrapper!
```


### Returning Wrappers

```python
def decorator(func):
    def wrapper():
        print("Wrapper")
        func()
    return wrapper  # Return the wrapper function
```


### Call Chain

```
User calls: greet()
    ↓
Actually calls: wrapper()
    ↓
Wrapper prints: "Before"
    ↓
Wrapper calls: original_greet()
    ↓
Original prints: "Hello!"
    ↓
Wrapper prints: "After"
```


### Complete Example with Line-by-Line Explanation

```python
def simple_decorator(func):
    # Line 1: Define decorator that accepts a function
    def wrapper():
        # Line 2: Define wrapper that will replace original function
        print("Before calling the function")
        # Line 3: Add before behavior
        func()
        # Line 4: Call original function
        print("After calling the function")
        # Line 5: Add after behavior
    return wrapper
    # Line 6: Return wrapper to replace original function

@simple_decorator
def say_hello():
    print("Hello!")

# Line 7: Define original function
# Line 8: Decorate it (say_hello = simple_decorator(say_hello))

say_hello()
# Line 9: Call decorated function (actually calls wrapper)
```

**Output:**

```
Before calling the function
Hello!
After calling the function
```


### Visual Step-by-Step

```
Step 1: Define decorator
def simple_decorator(func):
    def wrapper():
        print("Before")
        func()
        print("After")
    return wrapper

Step 2: Define original function
def say_hello():
    print("Hello!")

Step 3: Apply decorator (automatic)
say_hello = simple_decorator(say_hello)
# say_hello is now wrapper!

Step 4: Call decorated function
say_hello()
# Executes wrapper, not original
```


***

## Section 6 — Decoration Process Step by Step

### 1. Function Creation

Python first creates the function object:

```python
def greet():
    print("Hello")
```

**Namespace after creation:**

```python
namespace = {
    'greet': <function greet at 0x...>
}
```


### 2. Decorator Invocation

When Python sees `@decorator`, it immediately calls the decorator:

```python
@my_decorator
def greet():
    print("Hello")
```

**What happens:**

```python
greet = my_decorator(greet)
```

**Namespace after invocation:**

```python
namespace = {
    'greet': <function wrapper at 0x...>  # Replaced!
}
```


### 3. Wrapper Creation

Inside the decorator, a wrapper function is created:

```python
def my_decorator(func):
    def wrapper():
        print("Before")
        func()
        print("After")
    return wrapper
```

**Wrapper object:**

```python
wrapper = <function wrapper at 0x...>
wrapper.__closure__ = [<cell for func>]
```


### 4. Function Replacement

The original function is replaced in the namespace:

```
Before: greet → original function
After:  greet → wrapper function
```

**Memory diagram:**

```
┌─────────────────┐
│   Namespace     │
│   greet ────────┼───→ [ORIGINAL FUNCTION] (orphaned)
└─────────────────┘
                    └───→ [WRAPPER FUNCTION] (now referenced)
```


### 5. Runtime Execution

When you call `greet()`:

```python
greet()  # Actually calls wrapper()
```

**Execution flow:**

```
┌─────────────────────────────────────┐
│  greet() called                     │
└─────────────────────────────────────┘
           ↓
┌─────────────────────────────────────┐
│  wrapper() executes                 │
│  print("Before")                    │
└─────────────────────────────────────┘
           ↓
┌─────────────────────────────────────┐
│  func() called (original greet)     │
│  print("Hello")                     │
└─────────────────────────────────────┘
           ↓
┌─────────────────────────────────────┐
│  print("After")                     │
└─────────────────────────────────────┘
```


### Complete Namespace Transformation

```python
# Before decoration
namespace = {}

def greet():
    print("Hello")

namespace = {'greet': <function greet>}

# Apply decorator
@my_decorator
def greet():
    print("Hello")

# After decoration
namespace = {'greet': <function wrapper>}

# Original function is lost (unless decorator preserves it)
```


### Visual Timeline

```
Time:    0          1          2          3          4
         │          │          │          │          │
         │ Function │ Decorator│ Wrapper  │Function  │Runtime
         │ Creation │ Invocation│Creation │Replac.   │Exec.
         │          │          │          │          │
         ▼          ▼          ▼          ▼          ▼
Func:   [greet]   [greet]   [wrapper]  [wrapper]  wrapper()
         │          │          │          │          │
         │          └──────────┘          │          │
         │           returns              │          │
         └────────────────────────────────┘          │
                        replaced                     └─►
```


***

## Section 7 — Decorators with Arguments

### *args and **kwargs

To create decorators that work with **any function**, use `*args` and `**kwargs`:

```python
def decorator(func):
    def wrapper(*args, **kwargs):
        print("Before")
        result = func(*args, **kwargs)  # Forward all arguments
        print("After")
        return result
    return wrapper
```


### Argument Forwarding

```python
@decorator
def greet(name, age):
    return f"Hello {name}, age {age}"

greet("Alice", 25)  # Works with any arguments!
```

**Flow:**

```
greet("Alice", 25)
    ↓
wrapper(*args, **kwargs)
    args = ("Alice", 25)
    kwargs = {}
    ↓
func(*args, **kwargs)
    greet("Alice", 25)
    ↓
Returns: "Hello Alice, age 25"
```


### Returning Values

```python
def decorator(func):
    def wrapper(*args, **kwargs):
        print("Before")
        result = func(*args, **kwargs)
        print(f"Result: {result}")
        return result  # Must return!
    return wrapper

@decorator
def add(a, b):
    return a + b

result = add(5, 3)
print(result)  # 8
```


### Complete Example: Timer Decorator

```python
import time

def timer(func):
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        end = time.time()
        print(f"{func.__name__} took {end - start:.4f} seconds")
        return result
    return wrapper

@timer
def slow_function(n):
    time.sleep(1)
    return n ** 2

result = slow_function(10)
print(result)  # 100
```

**Output:**

```
slow_function took 1.0012 seconds
100
```


### Diagram: Argument Flow

```
┌─────────────────────────────────────────────┐
│           User Calls: func(1, 2, x=3)       │
└─────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────┐
│           wrapper(*args, **kwargs)          │
│           args = (1, 2)                     │
│           kwargs = {'x': 3}                 │
└─────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────┐
│           func(*args, **kwargs)             │
│           func(1, 2, x=3)                   │
└─────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────┐
│           Original function executes        │
│           Returns result                    │
└─────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────┐
│           wrapper returns result            │
└─────────────────────────────────────────────┘
```


### Practical Example: Logging Decorator

```python
def log(func):
    def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__} with args={args}, kwargs={kwargs}")
        result = func(*args, **kwargs)
        print(f"{func.__name__} returned {result}")
        return result
    return wrapper

@log
def multiply(a, b, c=1):
    return a * b * c

multiply(2, 3)
multiply(2, 3, c=4)
```

**Output:**

```
Calling multiply with args=(2, 3), kwargs={}
multiply returned 6
Calling multiply with args=(2, 3), kwargs={'c': 4}
multiply returned 24
```


***

## Section 8 — Preserving Metadata

### Why Metadata Matters

When you decorate a function, the wrapper replaces it. This **loses metadata**:

```python
def greet():
    """Greet someone"""
    print("Hello")

print(greet.__name__)   # 'greet'
print(greet.__doc__)    # 'Greet someone'

@decorator
def greet():
    """Greet someone"""
    print("Hello")

print(greet.__name__)   # 'wrapper' (LOST!)
print(greet.__doc__)    # None (LOST!)
```


### Function Metadata Attributes

| Attribute | Purpose | Example |
| :-- | :-- | :-- |
| `__name__` | Function name | `'greet'` |
| `__doc__` | Docstring | `'Greet someone'` |
| `__module__` | Module name | `'__main__'` |
| `__annotations__` | Type annotations | `{'name': str}` |
| `__qualname__` | Qualified name | `'main.greet'` |

### functools.wraps

`functools.wraps` copies metadata from original function to wrapper:

```python
from functools import wraps

def decorator(func):
    @wraps(func)  # Copies metadata!
    def wrapper(*args, **kwargs):
        print("Before")
        return func(*args, **kwargs)
    return wrapper

@decorator
def greet(name):
    """Greet someone by name"""
    return f"Hello {name}"

print(greet.__name__)   # 'greet' (PRESERVED!)
print(greet.__doc__)    # 'Greet someone by name' (PRESERVED!)
```


### functools.update_wrapper

`update_wrapper` explicitly copies attributes:

```python
from functools import update_wrapper

def decorator(func):
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    
    update_wrapper(wrapper, func)  # Manual copy
    return wrapper
```


### Internal Behavior of wraps

```python
# wraps is actually a decorator factory
def wraps(wrapped):
    def decorator(wrapper):
        wrapper.__name__ = wrapped.__name__
        wrapper.__doc__ = wrapped.__doc__
        wrapper.__module__ = wrapped.__module__
        wrapper.__annotations__ = wrapped.__annotations__
        wrapper.__qualname__ = wrapped.__qualname__
        wrapper.__wrapped__ = wrapped  # Original function!
        return wrapper
    return decorator
```


### Complete Example with Metadata

```python
from functools import wraps

def log_decorator(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__}")
        return func(*args, **kwargs)
    return wrapper

@log_decorator
def greet(name: str, age: int) -> str:
    """Greet a person"""
    return f"Hello {name}, age {age}"

# All metadata preserved!
print(greet.__name__)        # 'greet'
print(greet.__doc__)         # 'Greet a person'
print(greet.__annotations__) # {'name': str, 'age': int, 'return': str}
print(greet.__qualname__)    # 'main.greet'

# Can access original function
print(greet.__wrapped__)     # <function greet at 0x...>
```


### Why `__wrapped__` Matters

`__wrapped__` allows tools to access the original function:

```python
# Used by:
# - Documentation generators (Sphinx)
# - Type checkers (mypy)
# - Debuggers
# - Testing frameworks

import inspect
print(inspect.getsource(greet.__wrapped__))
```


***

## Section 9 — Decorators with Parameters

### Basic Syntax

```python
@repeat(3)
def greet():
    print("Hello")
```

This means: `greet = repeat(3)(greet)`

### Three-Layer Structure

```
decorator factory → decorator → wrapper
    repeat(3)      → repeat  → wrapper
```


### Complete Example: repeat Decorator

```python
def repeat(times):
    # Layer 1: Factory (receives decorator arguments)
    def decorator(func):
        # Layer 2: Decorator (receives function)
        @wraps(func)
        def wrapper(*args, **kwargs):
            # Layer 3: Wrapper (replaces original function)
            for _ in range(times):
                func(*args, **kwargs)
        return wrapper
    return decorator

@repeat(3)
def greet(name):
    print(f"Hello {name}")

greet("Alice")
```

**Output:**

```
Hello Alice
Hello Alice
Hello Alice
```


### Execution Order

```
1. @repeat(3) encountered
   → repeat(3) called
   → Returns decorator function

2. def greet() defined
   → decorator(greet) called
   → Returns wrapper function

3. greet = wrapper
   → greet now references wrapper

4. greet("Alice") called
   → wrapper("Alice") executes
   → Loops 3 times, calling original greet
```


### Visual Execution Order

```
┌─────────────────────────────────────────────┐
│  Step 1: repeat(3) called                   │
│  Returns: decorator function                │
└─────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────┐
│  Step 2: decorator(greet) called            │
│  Returns: wrapper function                  │
└─────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────┐
│  Step 3: greet = wrapper                    │
│  greet now references wrapper               │
└─────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────┐
│  Step 4: greet("Alice") called              │
│  Executes wrapper, loops 3 times            │
└─────────────────────────────────────────────┘
```


### More Examples

#### Example 1: limit Calls

```python
def limit(max_calls):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            if wrapper.calls >= max_calls:
                raise Exception(f"Max calls ({max_calls}) exceeded!")
            wrapper.calls += 1
            return func(*args, **kwargs)
        wrapper.calls = 0
        return wrapper
    return decorator

@limit(2)
def api_call():
    print("API called")

api_call()  # OK
api_call()  # OK
api_call()  # Exception!
```


#### Example 2: Retry Decorator

```python
def retry(max_attempts=3):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(max_attempts):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    print(f"Attempt {attempt + 1} failed: {e}")
            raise Exception("All attempts failed")
        return wrapper
    return decorator

@retry(max_attempts=3)
def unstable_function():
    import random
    if random.random() < 0.7:
        raise Exception("Random failure")
    return "Success!"

print(unstable_function())
```


#### Example 3: Count Calls Decorator

```python
def count_calls(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        wrapper.calls += 1
        print(f"Call #{wrapper.calls}")
        return func(*args, **kwargs)
    wrapper.calls = 0
    return wrapper

@count_calls
def greet(name):
    print(f"Hello {name}")

greet("Alice")  # Call #1
greet("Bob")    # Call #2
greet("Carol")  # Call #3
```


### Diagram: Three-Layer Flow

```
┌─────────────────────────────────────────────┐
│  @repeat(3)                                 │
│      ↓                                      │
│  repeat(3) → returns decorator              │
└─────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────┐
│  def greet(): ...                           │
│      ↓                                      │
│  decorator(greet) → returns wrapper         │
└─────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────┐
│  greet = wrapper                            │
│  greet("Alice") → wrapper("Alice")          │
│      ↓                                      │
│  Loops 3 times, calls original greet        │
└─────────────────────────────────────────────┘
```


### Key Insight

```python
@repeat(3)
def greet():
    pass

# Is equivalent to:
def greet():
    pass

greet = repeat(3)(greet)
# Step 1: repeat(3) → decorator
# Step 2: decorator(greet) → wrapper
# Step 3: greet = wrapper
```


***

## Section 10 — Multiple Decorators

### Stack Syntax

```python
@A
@B
@C
def func():
    pass
```


### Equivalent Transformation

```python
@A
@B
@C
def func():
    pass

# Is equivalent to:
def func():
    pass

func = A(B(C(func)))
```


### Execution Order

Decorators are applied **bottom-to-top** (closest to function first):

```
1. C(func) applied first
2. B(C(func)) applied second
3. A(B(C(func))) applied last
```


### Visual Execution Order

```
┌─────────────────────────────────────────────┐
│           Function Definition               │
│           def func(): ...                   │
└─────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────┐
│    Step 1: C(func) → wrapped_by_C           │
└─────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────┐
│    Step 2: B(wrapped_by_C) → wrapped_by_B   │
└─────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────┐
│    Step 3: A(wrapped_by_B) → wrapped_by_A   │
│    func = wrapped_by_A                      │
└─────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────┐
│    Call: func() → wrapped_by_A()            │
│    → wrapped_by_B()                         │
│    → wrapped_by_C()                         │
│    → original func()                        │
└─────────────────────────────────────────────┘
```


### Complete Example

```python
def decorator_A(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        print("A: Before")
        result = func(*args, **kwargs)
        print("A: After")
        return result
    return wrapper

def decorator_B(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        print("B: Before")
        result = func(*args, **kwargs)
        print("B: After")
        return result
    return wrapper

def decorator_C(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        print("C: Before")
        result = func(*args, **kwargs)
        print("C: After")
        return result
    return wrapper

@decorator_A
@decorator_B
@decorator_C
def greet():
    print("  greet: Hello!")

greet()
```

**Output:**

```
A: Before
B: Before
C: Before
  greet: Hello!
C: After
B: After
A: After
```


### Call Chain Visualization

```
greet() called
    ↓
wrapper_A() executes
    ├─ Print "A: Before"
    ↓
wrapper_B() executes
    ├─ Print "B: Before"
    ↓
wrapper_C() executes
    ├─ Print "C: Before"
    ↓
original_greet() executes
    ├─ Print "greet: Hello!"
    ↓
wrapper_C() continues
    ├─ Print "C: After"
    ↓
wrapper_B() continues
    ├─ Print "B: After"
    ↓
wrapper_A() continues
    ├─ Print "A: After"
```


### Stack Diagram

```
┌─────────────────────────────────────────────┐
│           Call Stack (runtime)              │
├─────────────────────────────────────────────┤
│  wrapper_A() ← Top (outermost)              │
│  wrapper_B() ← Middle                       │
│  wrapper_C() ← Bottom (innermost)           │
│  original_greet() ← Core                    │
└─────────────────────────────────────────────┘
```


### Key Points

1. **Application order**: Bottom-to-top (C → B → A)
2. **Execution order**: Top-to-bottom (A → B → C → func)
3. **Return order**: Bottom-to-top (func → C → B → A)

### Practical Example:Auth + Log + RateLimit

```python
def authenticate(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        print("✓ Auth: Checking credentials")
        return func(*args, **kwargs)
    return wrapper

def log(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        print("✓ Log: Recording call")
        return func(*args, **kwargs)
    return wrapper

def rate_limit(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        print("✓ RateLimit: Checking limit")
        return func(*args, **kwargs)
    return wrapper

@authenticate
@log
@rate_limit
def api_endpoint():
    print("  API: Processing request")

api_endpoint()
```

**Output:**

```
✓ Auth: Checking credentials
✓ Log: Recording call
✓ RateLimit: Checking limit
  API: Processing request
```


***

## Section 11 — Exercises

### Easy Exercises

#### Exercise 1: Simple Printer Decorator

**Problem:** Create a decorator that prints "Function called" before calling the function.

```python
# Your code here
def print_call(func):
    def wrapper():
        print("Function called")
        func()
    return wrapper

@print_call
def greet():
    print("Hello!")

greet()
```

**Solution:**

```python
def print_call(func):
    def wrapper():
        print("Function called")
        func()
    return wrapper

@print_call
def greet():
    print("Hello!")

greet()  # Output: "Function called" then "Hello!"
```


#### Exercise 2: Result Printer

**Problem:** Create a decorator that prints the result of a function.

```python
def print_result(func):
    def wrapper(*args, **kwargs):
        result = func(*args, **kwargs)
        print(f"Result: {result}")
        return result
    return wrapper

@print_result
def add(a, b):
    return a + b

add(5, 3)
```

**Solution:**

```python
def print_result(func):
    def wrapper(*args, **kwargs):
        result = func(*args, **kwargs)
        print(f"Result: {result}")
        return result
    return wrapper

@print_result
def add(a, b):
    return a + b

add(5, 3)  # Output: "Result: 8"
```


### Medium Exercises

#### Exercise 3: Timer Decorator

**Problem:** Create a decorator that measures and prints execution time.

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
def slow_function():
    time.sleep(0.5)
    return "Done"

slow_function()
```

**Solution:**

```python
import time
from functools import wraps

def timer(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        end = time.time()
        print(f"{func.__name__} took {end - start:.4f}s")
        return result
    return wrapper

@timer
def slow_function():
    time.sleep(0.5)
    return "Done"

slow_function()
```


#### Exercise 4: Call Counter

**Problem:** Create a decorator that counts how many times a function is called.

```python
def counter(func):
    func.calls = 0
    def wrapper(*args, **kwargs):
        func.calls += 1
        return func(*args, **kwargs)
    return wrapper

@counter
def greet(name):
    return f"Hello {name}"

greet("Alice")
greet("Bob")
print(greet.calls)  # 2
```

**Solution:**

```python
from functools import wraps

def counter(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        wrapper.calls += 1
        return func(*args, **kwargs)
    wrapper.calls = 0
    return wrapper

@counter
def greet(name):
    return f"Hello {name}"

greet("Alice")
greet("Bob")
print(greet.calls)  # 2
```


### Advanced Exercises

#### Exercise 5: Decorator with Parameters (Retry)

**Problem:** Create a `retry` decorator that retries a function up to N times on failure.

```python
def retry(max_attempts=3):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(max_attempts):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    print(f"Attempt {attempt + 1} failed: {e}")
            raise Exception("All attempts failed")
        return wrapper
    return decorator

@retry(max_attempts=3)
def unstable():
    import random
    if random.random() < 0.7:
        raise Exception("Random failure")
    return "Success"

print(unstable())
```

**Solution:**

```python
from functools import wraps

def retry(max_attempts=3):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(max_attempts):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    print(f"Attempt {attempt + 1} failed: {e}")
            raise Exception("All attempts failed")
        return wrapper
    return decorator

@retry(max_attempts=3)
def unstable():
    import random
    if random.random() < 0.7:
        raise Exception("Random failure")
    return "Success"

print(unstable())
```


#### Exercise 6: Multiple Decorators with State

**Problem:** Create two decorators that share state using a class.

```python
class DecoratorState:
    def __init__(self):
        self.call_count = 0

state = DecoratorState()

def count_calls(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        state.call_count += 1
        print(f"Call #{state.call_count}")
        return func(*args, **kwargs)
    return wrapper

def log_calls(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        print(f"Logging: {func.__name__}")
        return func(*args, **kwargs)
    return wrapper

@count_calls
@log_calls
def greet(name):
    return f"Hello {name}"

greet("Alice")
greet("Bob")
print(f"Total calls: {state.call_count}")
```

**Solution:**

```python
from functools import wraps

class DecoratorState:
    def __init__(self):
        self.call_count = 0

state = DecoratorState()

def count_calls(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        state.call_count += 1
        print(f"Call #{state.call_count}")
        return func(*args, **kwargs)
    return wrapper

def log_calls(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        print(f"Logging: {func.__name__}")
        return func(*args, **kwargs)
    return wrapper

@count_calls
@log_calls
def greet(name):
    return f"Hello {name}"

greet("Alice")
greet("Bob")
print(f"Total calls: {state.call_count}")
```


#### Exercise 7: Decorator Factory Returning Class

**Problem:** Create a decorator that returns a class instead of a function.

```python
def make_logger(level):
    def decorator(func):
        class LoggedFunc:
            def __init__(self, func):
                self.func = func
                self.level = level
            
            def __call__(self, *args, **kwargs):
                print(f"[{self.level}] Calling {self.func.__name__}")
                return self.func(*args, **kwargs)
        
        return LoggedFunc(func)
    return decorator

@make_logger("INFO")
def greet(name):
    return f"Hello {name}"

print(greet("Alice"))
```

**Solution:**

```python
from functools import wraps

def make_logger(level):
    def decorator(func):
        class LoggedFunc:
            def __init__(self, func):
                self.func = func
                self.level = level
            
            def __call__(self, *args, **kwargs):
                print(f"[{self.level}] Calling {self.func.__name__}")
                return self.func(*args, **kwargs)
            
            def __getattribute__(self, name):
                if name in ['func', 'level']:
                    return object.__getattribute__(self, name)
                return getattr(self.func, name)
        
        return LoggedFunc(func)
    return decorator

@make_logger("INFO")
def greet(name):
    return f"Hello {name}"

print(greet("Alice"))
print(greet.__name__)  # Should work due to __getattribute__
```


***

## Section 12 — Interview Questions

### Question 1: What is a decorator in Python?

**Answer:** A decorator is a function that takes another function as input, extends its behavior, and returns a new function without modifying the original function's code. Decorators use the `@` syntax and are equivalent to `func = decorator(func)`.

### Question 2: Explain the difference between `@decorator` and `func = decorator(func)`.

**Answer:** They are **exactly equivalent**. `@decorator` is just syntactic sugar that Python applies automatically when defining a function. Both achieve the same result of replacing `func` with the decorated version.

### Question 3: Why do we need `functools.wraps`?

**Answer:** `functools.wraps` preserves the original function's metadata (`__name__`, `__doc__`, `__annotations__`, etc.) in the wrapper function. Without it, the wrapper loses all metadata, making debugging, documentation, and type checking difficult.

### Question 4: What is a closure?

**Answer:** A closure is a nested function that references variables from its outer scope. Even after the outer function returns, the closure can access those "free variables" through its `__closure__` attribute.

### Question 5: What are `*args` and `**kwargs` used for in decorators?

**Answer:** They allow decorators to work with **any function** regardless of its argument signature. `*args` captures positional arguments as a tuple, and `**kwargs` captures keyword arguments as a dictionary.

### Question 6: Explain the execution order of multiple decorators.

**Answer:** Decorators are applied **bottom-to-top** but executed **top-to-bottom**:

```python
@A
@B
@C
def func(): pass

# Applied: C → B → A
# Executed: A → B → C → func
```


### Question 7: What is a higher-order function?

**Answer:** A higher-order function is a function that either accepts functions as arguments or returns a function. Decorators are higher-order functions because they return a wrapper function.

### Question 8: What is the difference between a decorator and a decorator factory?

**Answer:**

- **Decorator**: `def decorator(func): ...` (takes function, returns wrapper)
- **Decorator factory**: `def factory(arg): def decorator(func): ...` (takes arguments, returns decorator)


### Question 9: How do you preserve `__wrapped__` in a decorator?

**Answer:** Use `functools.wraps`, which automatically sets `__wrapped__` to the original function. This allows tools to access the undecorated function.

### Question 10: What happens if a decorator doesn't return the result of calling the original function?

**Answer:** The decorated function will return `None` (or whatever the wrapper returns), losing the original function's return value. Always return `func(*args, **kwargs)`.

### Question 11: Can you decorate a class with a decorator?

**Answer:** Yes! Class decorators work the same way:

```python
@decorator
class MyClass:
    pass
# Equivalent to: MyClass = decorator(MyClass)
```


### Question 12: What is the purpose of `__closure__` and `__code__.co_freevars`?

**Answer:**

- `__closure__`: Tuple of cell objects containing free variables
- `__code__.co_freevars`: Tuple of names of free variables

They allow inspecting what variables a closure captures.

### Question 13: Write a decorator that times a function.

**Answer:**

```python
import time
from functools import wraps

def timer(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        end = time.time()
        print(f"{func.__name__} took {end-start:.4f}s")
        return result
    return wrapper
```


### Question 14: What is lexical scoping?

**Answer:** Lexical scoping means a function can access variables from the scope where it was **defined**, not where it's called. This is essential for closures to work.

### Question 15: How do you create a decorator with parameters?

**Answer:** Use a decorator factory:

```python
def repeat(times):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for _ in range(times):
                func(*args, **kwargs)
        return wrapper
    return decorator

@repeat(3)
def greet(): pass
```


### Question 16: What is the problem with this decorator?

```python
def bad_decorator(func):
    def wrapper():
        return func()
    return wrapper
```

**Answer:** It doesn't accept `*args, **kwargs`, so it only works with functions that take no arguments. It will fail for any function with parameters.

### Question 17: What is a cell object?

**Answer:** A cell object is an internal Python object that stores free variables in a closure. It allows the closure to maintain references to variables even after the outer function returns.

### Question 18: Can decorators be chained?

**Answer:** Yes! Multiple decorators on a function are essentially chaining:

```python
@A
@B
@C
def func(): pass
# Equivalent to: func = A(B(C(func)))
```


### Question 19: What is the difference between `__name__` and `__qualname__`?

**Answer:**

- `__name__`: Simple function name (e.g., `'greet'`)
- `__qualname__`: Qualified name including scope (e.g., `'module.greet'` or `'ClassName.method'`)


### Question 20: Implement a decorator that caches function results.

**Answer:**

```python
from functools import wraps

def cache(func):
    @wraps(func)
    def wrapper(*args):
        if args not in wrapper.cache:
            wrapper.cache[args] = func(*args)
        return wrapper.cache[args]
    wrapper.cache = {}
    return wrapper

@cache
def fibonacci(n):
    if n < 2:
        return n
    return fibonacci(n-1) + fibonacci(n-2)
```


### Question 21: What happens when you access a function's `__wrapped__` attribute?

**Answer:** You get the original undecorated function. This is useful for debugging, documentation generation, and accessing the original implementation.

### Question 22: Why should decorators return the wrapper function?

**Answer:** The wrapper function replaces the original function. If you don't return it, the decorator returns `None`, and the function becomes `None`, causing errors when called.

***

## Final — Cheat Sheet \& Summary

### Decorator Cheat Sheet

```python
# Basic decorator
def decorator(func):
    def wrapper(*args, **kwargs):
        # Before
        result = func(*args, **kwargs)
        # After
        return result
    return wrapper

# With functools.wraps
from functools import wraps

def decorator(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper

# With parameters (factory)
def decorator(param):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            return func(*args, **kwargs)
        return wrapper
    return decorator

# Multiple decorators
@A
@B
@C
def func(): pass
# = func = A(B(C(func)))
```


### Summary Tables

#### Decorator Types

| Type | Syntax | Example |
| :-- | :-- | :-- |
| Basic | `@decorator` | `@log` |
| With params | `@decorator(arg)` | `@repeat(3)` |
| Multiple | Stack `@` | `@A @B @C` |
| Class-based | `@ClassName` | `@Singleton` |

#### Key Attributes

| Attribute | Purpose | Preserved by |
| :-- | :-- | :-- |
| `__name__` | Function name | `wraps` |
| `__doc__` | Docstring | `wraps` |
| `__wrapped__` | Original function | `wraps` |
| `__annotations__` | Type hints | `wraps` |
| `__qualname__` | Qualified name | `wraps` |

#### Common Decorator Patterns

```python
# 1. Logging
def log(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__}")
        return func(*args, **kwargs)
    return wrapper

# 2. Timing
def timer(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        print(f"{func.__name__}: {time.time()-start:.4f}s")
        return result
    return wrapper

# 3. Retry
def retry(n=3):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for _ in range(n):
                try:
                    return func(*args, **kwargs)
                except:
                    pass
        return wrapper
    return decorator

# 4. Cache
def cache(func):
    @wraps(func)
    def wrapper(*args):
        wrapper.cache.setdefault(args, func(*args))
        return wrapper.cache[args]
    wrapper.cache = {}
    return wrapper
```


### Key Takeaways

1. **Decorators are functions** that take functions and return new functions
2. **`@decorator` = `func = decorator(func)`** (syntactic sugar)
3. **Always use `*args, **kwargs`** to accept any function signature
4. **Always use `functools.wraps`** to preserve metadata
5. **Closures** enable decorators to access original function
6. **Multiple decorators** apply bottom-to-top, execute top-to-bottom
7. **Decorator factories** enable decorators with parameters
8. **Wrapper functions** replace original functions in the namespace

### Mental Model Summary

```
Original Function
       ↓
   Decorator
       ↓
   Wrapper (replaces original)
       ↓
   User calls wrapper
       ↓
   Wrapper calls original
       ↓
   Original returns result
       ↓
   Wrapper returns result
```


### Best Practices

✅ **DO:**

- Use `functools.wraps`
- Accept `*args, **kwargs`
- Return the result of `func()`
- Keep decorators simple
- Document decorator behavior

❌ **DON'T:**

- Forget to return wrapper
- Forget to return `func()` result
- Hardcode argument signatures
- Make decorators too complex
- Lose original function metadata


### Quick Reference

```python
# Import necessary tools
from functools import wraps, update_wrapper

# Basic structure
def my_decorator(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        # Before logic
        result = func(*args, **kwargs)
        # After logic
        return result
    return wrapper

# With parameters
def my_decorator(param):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            return func(*args, **kwargs)
        return wrapper
    return decorator
```


***

**You've now mastered the foundations of Python decorators!** 🎉

Next steps: Explore advanced topics like class-based decorators, decorator chains, async decorators, and real-world frameworks that use decorators extensively (Flask, Django, FastAPI).

