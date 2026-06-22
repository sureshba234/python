<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" style="height:64px;margin-right:32px"/>

# \# ROLE

You are a Python Core Developer, Senior Python Instructor, Software Architect, Technical Author, and Interview Coach.

# TASK

Create a COMPLETE Markdown (.md) guide on:

# Python Exception Handling (Part 1)

Cover beginner to advanced practical exception handling concepts.

# OUTPUT RULES

- Generate ONLY Markdown.
- Start with:


# Python Exception Handling – Part 1

- Include TOC.
- Explain concepts from beginner to advanced.
- Include examples for every topic.
- Include outputs.
- Include diagrams where useful.
- Include best practices.
- Include interview questions.
- Include exercises with solutions.


# TOPICS

## 1. Introduction to Exceptions

- What is an exception?
- Errors vs Exceptions
- Why exceptions exist
- Exception lifecycle
- Real-world examples


## 2. Exception Architecture

- Runtime errors
- Exception propagation
- Handler search process
- Program termination


## 3. Built-in Exception Hierarchy

Cover complete hierarchy.

Explain:

- BaseException
- Exception
- ArithmeticError
- LookupError
- OSError
- RuntimeError
- TypeError
- ValueError
- NameError
- AttributeError
- ImportError
- MemoryError
- AssertionError
- Warning hierarchy

Include hierarchy diagrams.

## 4. BaseException vs Exception

- Differences
- KeyboardInterrupt
- SystemExit
- Best practices


## 5. Common Built-in Exceptions

Explain:

- TypeError
- ValueError
- KeyError
- IndexError
- NameError
- AttributeError
- ImportError
- ModuleNotFoundError
- FileNotFoundError
- PermissionError
- RuntimeError
- AssertionError
- MemoryError
- ZeroDivisionError

For each:

- Meaning
- Cause
- Example
- Prevention


## 6. try Statement

- try
- except
- else
- finally

Execution flow diagrams.

## 7. except Clauses

- Single exception
- Multiple exceptions
- Tuple exceptions
- Generic exceptions

Examples.

## 8. else Clause

- Purpose
- Execution rules
- Examples


## 9. finally Clause

- Cleanup guarantees
- Resource management
- Return behavior
- Edge cases


## 10. Raising Exceptions

- raise statement
- Re-raising
- Raising built-ins
- Raising custom exceptions


## 11. Exception Chaining

- raise from
- __cause__
- __context__
- Chained tracebacks


## 12. Custom Exceptions

- Creating custom exceptions
- Inheritance
- Domain-specific exceptions
- Real-world examples


## 13. Exception Objects

- args
- attributes
- custom state
- serialization


## 14. Assertions

- assert statement
- PYTHONOPTIMIZE
- Production considerations


## 15. Tracebacks

- Reading tracebacks
- Debugging
- traceback module


## 16. Logging Exceptions

- logging.exception()
- exc_info
- Structured logging


## 17. Context Managers and Exceptions

- with statement
- __enter__
- __exit__
- Exception suppression


## 18. Exception Handling in Functions and Classes

- Error propagation
- API design
- OOP exception patterns


## 19. Best Practices

## 20. Anti-Patterns

## 21. Interview Questions (Minimum 25)

## 22. Exercises with Solutions (Minimum 15)

## 23. Cheat Sheet

## Conclusion

```python
content = """# Python Exception Handling – Part 1

## Table of Contents

1. [Introduction to Exceptions](#1-introduction-to-exceptions)
2. [Exception Architecture](#2-exception-architecture)
3. [Built-in Exception Hierarchy](#3-built-in-exception-hierarchy)
4. [BaseException vs Exception](#4-baseexception-vs-exception)
5. [Common Built-in Exceptions](#5-common-built-in-exceptions)
6. [try Statement](#6-try-statement)
7. [except Clauses](#7-except-clauses)
8. [else Clause](#8-else-clause)
9. [finally Clause](#9-finally-clause)
10. [Raising Exceptions](#10-raising-exceptions)
11. [Exception Chaining](#11-exception-chaining)
12. [Custom Exceptions](#12-custom-exceptions)
13. [Exception Objects](#13-exception-objects)
14. [Assertions](#14-assertions)
15. [Tracebacks](#15-tracebacks)
16. [Logging Exceptions](#16-logging-exceptions)
17. [Context Managers and Exceptions](#17-context-managers-and-exceptions)
18. [Exception Handling in Functions and Classes](#18-exception-handling-in-functions-and-classes)
19. [Best Practices](#19-best-practices)
20. [Anti-Patterns](#20-anti-patterns)
21. [Interview Questions](#21-interview-questions-minimum-25)
22. [Exercises with Solutions](#22-exercises-with-solutions-minimum-15)
23. [Cheat Sheet](#23-cheat-sheet)
24. [Conclusion](#conclusion)

---

## 1. Introduction to Exceptions

### What is an Exception?

An **exception** is an event that occurs during program execution that disrupts the normal flow of instructions. When Python encounters an error condition, it raises an exception object containing information about the error.

```python
# Example: Exception occurs when dividing by zero
result = 10 / 0  # Raises ZeroDivisionError
```

**Output:**

```
Traceback (most recent call last):
```

File "<stdin>", line 1, in <module>

```
ZeroDivisionError: division by zero
```


### Errors vs Exceptions

| Aspect | Errors | Exceptions |
| :-- | :-- | :-- |
| **Nature** | Syntax problems, bugs in code | Runtime conditions that can be handled |
| **Detection** | During parsing/compilation | During program execution |
| **Handling** | Cannot be handled at runtime | Can be caught and handled |
| **Example** | `def func(` (missing closing paren) | `10 / 0` (division by zero) |

```python
# Error (Syntax Error) - cannot run this code
def func(  # SyntaxError: expected ')'
    pass

# Exception (Runtime Error) - code runs but raises exception
value = int("not a number")  # Raises ValueError
```

**Output:**

```
ValueError: invalid literal for int() with base 10: 'not a number'
```


### Why Exceptions Exist

1. **Separation of Error Handling**: Separate error handling code from normal business logic
2. **Program Continuity**: Allow programs to continue running after errors
3. **Informative Errors**: Provide detailed information about what went wrong
4. **Graceful Degradation**: Handle errors gracefully instead of crashing
5. **Resource Management**: Ensure cleanup of resources even when errors occur

### Exception Lifecycle

```
1. Error Condition Occurs
        ↓
2. Python Creates Exception Object
        ↓
3. Exception is Raised
        ↓
4. Handler Search Begins
        ↓
5a. Handler Found → Exception Handled
        ↓
5b. No Handler → Program Terminates
```


### Real-World Examples

```python
# Example 1: File not found
try:
    with open("data.txt", "r") as f:
        content = f.read()
except FileNotFoundError:
    print("File not found! Creating a new one...")
    with open("data.txt", "w") as f:
        f.write("New file created")

# Example 2: Invalid user input
try:
    age = int(input("Enter your age: "))
    if age < 0:
        raise ValueError("Age cannot be negative")
except ValueError as e:
    print(f"Invalid input: {e}")
    age = 0

# Example 3: Network timeout
import requests
try:
    response = requests.get("https://api.example.com/data", timeout=5)
    data = response.json()
except requests.Timeout:
    print("Request timed out. Using cached data.")
    data = get_cached_data()
```


---

## 2. Exception Architecture

### Runtime Errors

Runtime errors occur when Python executes code and encounters an unexpected condition:

```python
# Common runtime errors
# 1. Division by zero
result = 10 / 0  # ZeroDivisionError

# 2. Invalid type conversion
number = int("hello")  # ValueError

# 3. Accessing non-existent key
data = {"name": "Alice"}
print(data["age"])  # KeyError

# 4. Accessing non-existent index
items = 
print(items)  # IndexError
```


### Exception Propagation

Exceptions propagate up the call stack until they're caught or the program terminates:

```python
def function_a():
    function_b()

def function_b():
    function_c()

def function_c():
    # Exception occurs here
    result = 10 / 0

# Exception propagates: function_c → function_b → function_a → main
try:
    function_a()
except ZeroDivisionError:
    print("Caught exception after propagation!")
```

**Output:**

```
Caught exception after propagation!
```


### Handler Search Process

```
1. Exception raised in current function
        ↓
2. Search for except clause in current try block
        ↓
3. If not found, function returns (exception propagates)
        ↓
4. Search in caller's try block
        ↓
5. Continue up call stack
        ↓
6. If no handler found → program terminates
```

```python
def inner():
    x = 1 / 0  # Raises ZeroDivisionError

def middle():
    inner()  # No handler here

def outer():
    try:
        middle()  # Exception propagates here
    except ZeroDivisionError:
        print("Handler found in outer()")

outer()
```

**Output:**

```
Handler found in outer()
```


### Program Termination

If no handler catches an exception, Python terminates the program:

```python
# No exception handler - program terminates
result = 10 / 0
print("This line won't execute")
```

**Output:**

```
Traceback (most recent call last):
```

File "<stdin>", line 1, in <module>

```
ZeroDivisionError: division by zero
```


---

## 3. Built-in Exception Hierarchy

Python's exception hierarchy is rooted at `BaseException`. Here's the complete structure:

```
BaseException
├── Exception
│   ├── ArithmeticError
│   │   ├── ZeroDivisionError
│   │   ├── OverflowError
│   │   └── FloatingPointError
│   ├── LookupError
│   │   ├── KeyError
│   │   └── IndexError
│   ├── AttributeError
│   ├── EOFError
│   ├── ImportError
│   │   └── ModuleNotFoundError
│   ├── MemoryError
│   ├── NameError
│   ├── OSError
│   │   ├── FileNotFoundError
│   │   ├── PermissionError
│   │   ├── ConnectionError
│   │   └── TimeoutError
│   ├── RuntimeError
│   │   ├── RecursionError
│   └── TypeError
│   ├── ValueError
│   ├── AssertionError
│   ├── SystemError
│   └── Warning
│       ├── UserWarning
│       ├── DeprecationWarning
│       ├── PendingDeprecationWarning
│       ├── SyntaxWarning
│       ├── RuntimeWarning
│       └── FutureWarning
├── KeyboardInterrupt
└── SystemExit
```


### BaseException

The base class for all exceptions. Most code should inherit from `Exception` instead.

```python
print(BaseException.__doc__)
```

**Output:**

```
One instance argument (optional).
```


### Exception

The base class for all exceptions that can be raised by user code. Always inherit from this for custom exceptions.

```python
class MyCustomError(Exception):
    """My custom exception"""
    pass

try:
    raise MyCustomError("Something went wrong")
except MyCustomError as e:
    print(f"Caught: {e}")
```

**Output:**

```
Caught: Something went wrong
```


### ArithmeticError

Base class for arithmetic errors:

```python
# ZeroDivisionError
try:
    result = 10 / 0
except ZeroDivisionError:
    print("Cannot divide by zero")

# OverflowError (with floats)
import sys
try:
    result = float('inf') * float('inf')
    # This doesn't actually overflow in Python
except OverflowError:
    print("Number too large")
```

**Output:**

```
Cannot divide by zero
```


### LookupError

Base class for lookup errors:

```python
# KeyError
try:
    data = {"name": "Alice"}
    print(data["age"])
except KeyError:
    print("Key not found in dictionary")

# IndexError
try:
    items = 
    print(items)
except IndexError:
    print("Index out of range")
```

**Output:**

```
Key not found in dictionary
Index out of range
```


### OSError

Base class for operating system errors:

```python
# FileNotFoundError
try:
    with open("nonexistent.txt", "r") as f:
        content = f.read()
except FileNotFoundError:
    print("File does not exist")

# PermissionError
try:
    with open("/root/protected.txt", "r") as f:
        content = f.read()
except PermissionError:
    print("Permission denied")
```

**Output:**

```
File does not exist
```


### RuntimeError

Errors that occur when the interpreter detects an error condition:

```python
# RecursionError
def infinite_recursion():
    return infinite_recursion()

try:
    infinite_recursion()
except RecursionError:
    print("Maximum recursion depth exceeded")
```

**Output:**

```
Maximum recursion depth exceeded
```


### TypeError

Wrong operation for an object type:

```python
try:
    result = "5" + 3
except TypeError:
    print("Cannot add string and integer")
```

**Output:**

```
Cannot add string and integer
```


### ValueError

Invalid value for an operation:

```python
try:
    number = int("not a number")
except ValueError:
    print("Invalid integer value")
```

**Output:**

```
Invalid integer value
```


### NameError

Variable name not found:

```python
try:
    print(undefined_variable)
except NameError:
    print("Variable not defined")
```

**Output:**

```
Variable not defined
```


### AttributeError

Attribute reference or assignment fails:

```python
class Person:
    pass

p = Person()
try:
    print(p.age)
except AttributeError:
    print("Attribute 'age' does not exist")
```

**Output:**

```
Attribute 'age' does not exist
```


### ImportError

Import statement fails:

```python
try:
    import nonexistent_module
except ImportError:
    print("Module cannot be imported")
```

**Output:**

```
Module cannot be imported
```


### MemoryError

Not enough memory for operation:

```python
# Note: This is hard to trigger in practice
try:
    # Attempt to create a very large object
    large_list =  * (10 ** 10)
except MemoryError:
    print("Out of memory")
```


### AssertionError

Assert statement fails:

```python
def set_age(age):
    assert age > 0, "Age must be positive"
    return age

try:
    set_age(-5)
except AssertionError as e:
    print(f"Assertion failed: {e}")
```

**Output:**

```
Assertion failed: Age must be positive
```


### Warning Hierarchy

Warnings don't stop execution but indicate potential issues:

```python
import warnings

# UserWarning
warnings.warn("This is a user warning", UserWarning)

# DeprecationWarning
warnings.warn("This function is deprecated", DeprecationWarning)
```

**Output:**

```
UserWarning: This is a user warning
DeprecationWarning: This function is deprecated
```


---

## 4. BaseException vs Exception

### Differences

| Aspect | BaseException | Exception |
| :-- | :-- | :-- |
| **Purpose** | Base for ALL exceptions | Base for user exceptions |
| **Catches** | Includes system exceptions | Only user-level exceptions |
| ** inherit from** | INTERNAL use only | CUSTOM exceptions |
| **catches with** | `except BaseException` | `except Exception` |

```python
# BAD: Catching BaseException
try:
    raise KeyboardInterrupt()
except BaseException:
    print("Caught KeyboardInterrupt - program won't stop!")

# GOOD: Catching Exception
try:
    raise KeyboardInterrupt()
except Exception:
    print("This won't catch KeyboardInterrupt")
```

**Output:**

```
Caught KeyboardInterrupt - program won't stop!
```


### KeyboardInterrupt

Raised when user interrupts program (Ctrl+C):

```python
# This exception is raised when you press Ctrl+C
try:
    while True:
        print("Running...")
        # Press Ctrl+C to trigger KeyboardInterrupt
except KeyboardInterrupt:
    print("Program interrupted by user")
```


### SystemExit

Raised by `sys.exit()`:

```python
import sys

try:
    print("About to exit")
    sys.exit(0)
except SystemExit:
    print("SystemExit caught - program didn't exit!")
```

**Output:**

```
About to exit
SystemExit caught - program didn't exit!
```


### Best Practices

1. **Always inherit from `Exception`** for custom exceptions
2. **Never catch `BaseException`** unless you need to catch KeyboardInterrupt/SystemExit
3. **Use `except Exception`** for general exception handling
```python
# CORRECT custom exception
class MyError(Exception):
    pass

# INCORRECT custom exception
class MyError(BaseException):  # Don't do this!
    pass
```


---

## 5. Common Built-in Exceptions

### TypeError

| Aspect | Description |
| :-- | :-- |
| **Meaning** | Operation or function applied to object of wrong type |
| **Cause** | Mixing incompatible types |

```python
# Example
try:
    result = "5" + 3
except TypeError as e:
    print(f"TypeError: {e}")

# Prevention
def add_numbers(a, b):
    if not isinstance(a, (int, float)) or not isinstance(b, (int, float)):
        raise TypeError("Both arguments must be numbers")
    return a + b

print(add_numbers(5, 3))  # Works
# add_numbers("5", 3)  # Raises TypeError
```

**Output:**

```
TypeError: can only concatenate str (not "int") to str
5
```


### ValueError

| Aspect | Description |
| :-- | :-- |
| **Meaning** | Function receives invalid argument |
| **Cause** | Value outside acceptable range |

```python
# Example
try:
    number = int("hello")
except ValueError as e:
    print(f"ValueError: {e}")

# Prevention
def set_age(age):
    if not isinstance(age, int):
        raise ValueError("Age must be an integer")
    if age < 0 or age > 150:
        raise ValueError("Age must be between 0 and 150")
    return age

print(set_age(25))  # Works
```

**Output:**

```
ValueError: invalid literal for int() with base 10: 'hello'
25
```


### KeyError

| Aspect | Description |
| :-- | :-- |
| **Meaning** | Dictionary key not found |
| **Cause** | Accessing non-existent key |

```python
# Example
try:
    data = {"name": "Alice"}
    print(data["age"])
except KeyError as e:
    print(f"KeyError: {e}")

# Prevention
data = {"name": "Alice"}
age = data.get("age", 0)  # Returns 0 if key not found
print(f"Age: {age}")
```

**Output:**

```
KeyError: 'age'
Age: 0
```


### IndexError

| Aspect | Description |
| :-- | :-- |
| **Meaning** | Sequence index out of range |
| **Cause** | Accessing non-existent index |

```python
# Example
try:
    items = 
    print(items)
except IndexError as e:
    print(f"IndexError: {e}")

# Prevention
items = 
if 0 <= 5 < len(items):
    print(items)
else:
    print("Index out of range")
```

**Output:**

```
IndexError: list index out of range
Index out of range
```


### NameError

| Aspect | Description |
| :-- | :-- |
| **Meaning** | Local or global name not found |
| **Cause** | Using undefined variable |

```python
# Example
try:
    print(undefined_variable)
except NameError as e:
    print(f"NameError: {e}")

# Prevention
if "defined_variable" in globals():
    print(defined_variable)
else:
    print("Variable not defined")
```

**Output:**

```
NameError: name 'undefined_variable' is not defined
Variable not defined
```


### AttributeError

| Aspect | Description |
| :-- | :-- |
| **Meaning** | Attribute reference or assignment fails |
| **Cause** | Accessing non-existent attribute |

```python
# Example
class Person:
    def __init__(self):
        self.name = "Alice"

p = Person()
try:
    print(p.age)
except AttributeError as e:
    print(f"AttributeError: {e}")

# Prevention
if hasattr(p, "age"):
    print(p.age)
else:
    print("Attribute 'age' does not exist")
```

**Output:**

```
AttributeError: 'Person' object has no attribute 'age'
Attribute 'age' does not exist
```


### ImportError

| Aspect | Description |
| :-- | :-- |
| **Meaning** | Import statement fails |
| **Cause** | Module not found or import error |

```python
# Example
try:
    import nonexistent_module
except ImportError as e:
    print(f"ImportError: {e}")
```

**Output:**

```
ImportError: No module named 'nonexistent_module'
```


### ModuleNotFoundError

| Aspect | Description |
| :-- | :-- |
| **Meaning** | Module specifically not found (subclass of ImportError) |
| **Cause** | Module name doesn't exist |

```python
# Example
try:
    import nonexistent_module
except ModuleNotFoundError as e:
    print(f"ModuleNotFoundError: {e}")

# Prevention
import importlib
module_name = "os"
if importlib.find_module(module_name):
    import os
else:
    print(f"Module {module_name} not found")
```

**Output:**

```
ModuleNotFoundError: No module named 'nonexistent_module'
```


### FileNotFoundError

| Aspect | Description |
| :-- | :-- |
| **Meaning** | File or directory not found |
| **Cause** | Opening non-existent file |

```python
# Example
try:
    with open("nonexistent.txt", "r") as f:
        content = f.read()
except FileNotFoundError as e:
    print(f"FileNotFoundError: {e}")

# Prevention
import os
filename = "data.txt"
if os.path.exists(filename):
    with open(filename, "r") as f:
        content = f.read()
else:
    print(f"File {filename} does not exist")
```

**Output:**

```
FileNotFoundError: [Errno 2] No such file or directory: 'nonexistent.txt'
File data.txt does not exist
```


### PermissionError

| Aspect | Description |
| :-- | :-- |
| **Meaning** | Permission denied for operation |
| **Cause** | Accessing protected file |

```python
# Example
try:
    with open("/root/protected.txt", "w") as f:
        f.write("data")
except PermissionError as e:
    print(f"PermissionError: {e}")
```

**Output:**

```
PermissionError: [Errno 13] Permission denied: '/root/protected.txt'
```


### RuntimeError

| Aspect | Description |
| :-- | :-- |
| **Meaning** | Error not covered by other categories |
| **Cause** | General runtime error |

```python
# Example
def bad_function():
    raise RuntimeError("Something went wrong")

try:
    bad_function()
except RuntimeError as e:
    print(f"RuntimeError: {e}")
```

**Output:**

```
RuntimeError: Something went wrong
```


### AssertionError

| Aspect | Description |
| :-- | :-- |
| **Meaning** | Assert statement fails |
| **Cause** | Condition is False |

```python
# Example
def validate_age(age):
    assert age > 0, "Age must be positive"
    return age

try:
    validate_age(-5)
except AssertionError as e:
    print(f"AssertionError: {e}")
```

**Output:**

```
AssertionError: Age must be positive
```


### MemoryError

| Aspect | Description |
| :-- | :-- |
| **Meaning** | Not enough memory for operation |
| **Cause** | Memory exhaustion |

```python
# Example (hard to trigger)
try:
    # This might not actually raise MemoryError in Python
    large_list =  * (10 ** 10)
except MemoryError as e:
    print(f"MemoryError: {e}")
```


### ZeroDivisionError

| Aspect | Description |
| :-- | :-- |
| **Meaning** | Division by zero |
| **Cause** | Dividing by zero |

```python
# Example
try:
    result = 10 / 0
except ZeroDivisionError as e:
    print(f"ZeroDivisionError: {e}")

# Prevention
def safe_divide(a, b):
    if b == 0:
        raise ValueError("Cannot divide by zero")
    return a / b

print(safe_divide(10, 2))  # Works
```

**Output:**

```
ZeroDivisionError: division by zero
5.0
```


---

## 6. try Statement

The `try` statement allows you to handle exceptions gracefully.

### Structure

```python
try:
    # Code that might raise exception
except ExceptionType:
    # Handle specific exception
else:
    # Execute if no exception
finally:
    # Always execute (cleanup)
```


### Components

| Clause | Purpose | Executes When |
| :-- | :-- | :-- |
| `try` | Risky code | Always first |
| `except` | Handle exception | Exception raised |
| `else` | Success code | No exception |
| `finally` | Cleanup code | Always (even with return) |

### Execution Flow

```
┌─────────────────┐
│     try:        │
│   risky code    │
└────────┬────────┘
         │
    ┌────┴────┐
    │ Exception?│
    └────┬────┘
         │
    ┌────┴────────────┐
    │    Yes          │    No
    └────┬────────────┘    │
         │                 │
    ┌────┴────┐      ┌────┴────┐
    │ except: │      │  else:  │
    │ handler │      │success  │
    └────┬────┘      └────┬────┘
         │                 │
         └───────┬─────────┘
                 │
           ┌────┴────┐
           │ finally:│
           │ cleanup │
           └─────────┘
```

```python
# Complete example
try:
    print("1. Starting try block")
    result = 10 / 2
    print(f"2. Result: {result}")
except ZeroDivisionError:
    print("3. Caught division error")
else:
    print("4. No exception - else block")
finally:
    print("5. Finally block always runs")

print("6. Program continues")
```

**Output:**

```
1. Starting try block
2. Result: 5.0
4. No exception - else block
5. Finally block always runs
6. Program continues
```

```python
# With exception
try:
    print("1. Starting try block")
    result = 10 / 0
    print(f"2. Result: {result}")
except ZeroDivisionError:
    print("3. Caught division error")
else:
    print("4. No exception - else block")
finally:
    print("5. Finally block always runs")

print("6. Program continues")
```

**Output:**

```
1. Starting try block
3. Caught division error
5. Finally block always runs
6. Program continues
```


---

## 7. except Clauses

### Single Exception

```python
try:
    result = 10 / 0
except ZeroDivisionError:
    print("Cannot divide by zero")
```

**Output:**

```
Cannot divide by zero
```


### Multiple Exceptions (Separate except)

```python
try:
    value = int("hello")
except ValueError:
    print("Value error: invalid integer")
except TypeError:
    print("Type error: wrong type")
```

**Output:**

```
Value error: invalid integer
```


### Multiple Exceptions (Tuple)

```python
try:
    result = 10 / 0
except (ZeroDivisionError, ValueError, TypeError):
    print("Caught any of: ZeroDivisionError, ValueError, or TypeError")
```

**Output:**

```
Caught any of: ZeroDivisionError, ValueError, or TypeError
```


### Generic Exception

```python
# Catch any exception
try:
    result = 10 / 0
    value = int("hello")
except Exception:
    print("Caught any exception")
```

**Output:**

```
Caught any exception
```


### Exception with Variable

```python
try:
    result = 10 / 0
except ZeroDivisionError as e:
    print(f"Exception type: {type(e).__name__}")
    print(f"Exception message: {e}")
```

**Output:**

```
Exception type: ZeroDivisionError
Exception message: division by zero
```


### Multiple except with Different Handlers

```python
def process_data(value):
    try:
        if value == "zero":
            result = 10 / 0
        elif value == "invalid":
            result = int("hello")
        elif value == "missing":
            data = {}
            result = data["key"]
        else:
            result = value * 2
    except ZeroDivisionError:
        return "Error: Division by zero"
    except ValueError:
        return "Error: Invalid value"
    except KeyError:
        return "Error: Key not found"
    else:
        return f"Success: {result}"

print(process_data(5))      # Success
print(process_data("zero")) # Division error
print(process_data("invalid")) # Value error
print(process_data("missing")) # Key error
```

**Output:**

```
Success: 10
Error: Division by zero
Error: Invalid value
Error: Key not found
```


---

## 8. else Clause

### Purpose

The `else` clause executes when **no exception** occurs in the `try` block.

### Execution Rules

1. `else` runs only if `try` completes without exception
2. `else` runs before `finally`
3. `else` is skipped if exception occurs
4. `else` is useful for code that should run only on success
```python
# Without exception
try:
    print("1. try block - no exception")
except ValueError:
    print("2. except block - skipped")
else:
    print("3. else block - runs because no exception")
finally:
    print("4. finally block - always runs")
```

**Output:**

```
1. try block - no exception
3. else block - runs because no exception
4. finally block - always runs
```

```python
# With exception
try:
    print("1. try block - exception occurs")
    result = 10 / 0
except ZeroDivisionError:
    print("2. except block - handles exception")
else:
    print("3. else block - SKIPPED because exception")
finally:
    print("4. finally block - always runs")
```

**Output:**

```
1. try block - exception occurs
2. except block - handles exception
4. finally block - always runs
```


### Practical Examples

```python
# File reading example
filename = "data.txt"

try:
    file = open(filename, "r")
    content = file.read()
except FileNotFoundError:
    print(f"File {filename} not found")
    content = ""
else:
    print(f"File read successfully ({len(content)} characters)")
    file.close()

print(f"Content length: {len(content)}")
```

**Output:**

```
File data.txt not found
Content length: 0
```

```python
# Database connection example
def connect_and_query(db_url):
    try:
        # Simulate connection
        connection = f"Connected to {db_url}"
        print(connection)
    except Exception as e:
        print(f"Connection failed: {e}")
        return None
    else:
        # Only query if connection succeeded
        print("Executing query...")
        return {"result": "data"}
    finally:
        print("Cleanup complete")

result = connect_and_query("mysql://localhost")
print(f"Query result: {result}")
```

**Output:**

```
Connected to mysql://localhost
Executing query...
Cleanup complete
Query result: {'result': 'data'}
```


---

## 9. finally Clause

### Cleanup Guarantees

The `finally` clause **always executes**, even when:

- Exception occurs
- `return` statement in `try` or `except`
- Program interrupted

```python
# finally with return in try
def test_return_in_try():
    try:
        print("1. try block")
        return "returned from try"
    except:
        print("2. except block")
    finally:
        print("3. finally block - runs even with return!")
    
result = test_return_in_try()
print(f"Result: {result}")
```

**Output:**

```
1. try block
3. finally block - runs even with return!
Result: returned from try
```

```python
# finally with return in except
def test_return_in_except():
    try:
        print("1. try block")
        result = 10 / 0
    except ZeroDivisionError:
        print("2. except block")
        return "returned from except"
    finally:
        print("3. finally block - runs even with return!")
    
result = test_return_in_except()
print(f"Result: {result}")
```

**Output:**

```
1. try block
2. except block
3. finally block - runs even with return!
Result: returned from except
```


### Resource Management

```python
# File handling with finally
def read_file(filename):
    file = None
    try:
        print(f"Opening {filename}")
        file = open(filename, "r")
        content = file.read()
        print(f"Read {len(content)} characters")
    except FileNotFoundError:
        print(f"File {filename} not found")
        return ""
    finally:
        if file:
            print("Closing file")
            file.close()
    
    return content

# This will fail gracefully
content = read_file("nonexistent.txt")
print(f"Content: '{content}'")
```

**Output:**

```
Opening nonexistent.txt
File nonexistent.txt not found
Closing file
Content: ''
```


### Return Behavior Edge Cases

```python
# finally can override return (BAD PRACTICE!)
def bad_return_override():
    try:
        return "from try"
    finally:
        return "from finally"  # This overrides!

print(bad_return_override())
```

**Output:**

```
from finally
```

```python
# Exception in finally overrides return
def exception_in_finally():
    try:
        return "from try"
    finally:
        raise ValueError("Exception in finally")

try:
    print(exception_in_finally())
except ValueError as e:
    print(f"Caught: {e}")
```

**Output:**

```
Caught: Exception in finally
```


### Best Practice: Use try/finally for cleanup

```python
# Recommended pattern
def process_with_cleanup():
    resource = None
    try:
        resource = acquire_resource()
        return process_resource(resource)
    finally:
        if resource:
            cleanup_resource(resource)
```


---

## 10. Raising Exceptions

### raise Statement

```python
# Basic raise
raise ValueError("Invalid value")
```

**Output:**

```
ValueError: Invalid value
```

```python
# Raise with condition
def set_age(age):
    if age < 0:
        raise ValueError("Age cannot be negative")
    if age > 150:
        raise ValueError("Age cannot exceed 150")
    return age

try:
    set_age(-5)
except ValueError as e:
    print(f"Error: {e}")
```

**Output:**

```
Error: Age cannot be negative
```


### Re-raising

```python
# Re-raise the same exception
def wrapper():
    try:
        raise ValueError("Original error")
    except ValueError:
        print("Handling...")
        raise  # Re-raise same exception

try:
    wrapper()
except ValueError as e:
    print(f"Re-raised: {e}")
```

**Output:**

```
Handling...
Re-raised: Original error
```


### Raising Built-ins

```python
# Raise various built-in exceptions
try:
    raise TypeError("Wrong type")
    raise KeyError("Missing key")
    raise RuntimeError("General error")
except (TypeError, KeyError, RuntimeError) as e:
    print(f"Caught: {type(e).__name__}: {e}")
```

**Output:**

```
Caught: TypeError: Wrong type
```

```python
# Raise with specific exception type
exception_types = [
    ValueError("Invalid value"),
    TypeError("Wrong type"),
    KeyError("Missing key"),
    IndexError("Out of range"),
]

for exc in exception_types:
    try:
        raise exc
    except Exception as e:
        print(f"{type(e).__name__}: {e}")
```

**Output:**

```
ValueError: Invalid value
TypeError: Wrong type
KeyError: Missing key
IndexError: Out of range
```


### Raising Custom Exceptions

```python
# Define custom exception
class InvalidEmailError(Exception):
    """Raised when email format is invalid"""
    pass

def validate_email(email):
    if not "@" in email:
        raise InvalidEmailError(f"Email must contain '@': {email}")
    if not "." in email:
        raise InvalidEmailError(f"Email must contain '.': {email}")
    return True

try:
    validate_email("invalid@email")
except InvalidEmailError as e:
    print(f"Error: {e}")
```

**Output:**

```
Error: Email must contain '.': invalid@email
```

```python
# Custom exception with extra data
class APIError(Exception):
    def __init__(self, message, status_code, endpoint):
        super().__init__(message)
        self.message = message
        self.status_code = status_code
        self.endpoint = endpoint
    
    def __str__(self):
        return f"{self.status_code} at {self.endpoint}: {self.message}"

def call_api(endpoint):
    if endpoint == "/broken":
        raise APIError("Service unavailable", 500, endpoint)
    return {"success": True}

try:
    call_api("/broken")
except APIError as e:
    print(f"API Error: {e}")
    print(f"Status: {e.status_code}")
    print(f"Endpoint: {e.endpoint}")
```

**Output:**

```
API Error: 500 at /broken: Service unavailable
Status: 500
Endpoint: /broken
```


---

## 11. Exception Chaining

Exception chaining occurs when one exception leads to another.

### raise from

```python
# Explicit chaining with raise from
def process_file(filename):
    try:
        with open(filename, "r") as f:
            return f.read()
    except FileNotFoundError as e:
        raise RuntimeError(f"Failed to process {filename}") from e

try:
    process_file("missing.txt")
except RuntimeError as e:
    print(f"RuntimeError: {e}")
    print(f"Cause: {e.__cause__}")
```

**Output:**

```
RuntimeError: Failed to process missing.txt
Cause: [Errno 2] No such file or directory: 'missing.txt'
```


### __cause__

The `__cause__` attribute holds the original exception:

```python
try:
    try:
        raise ValueError("Original error")
    except ValueError as original:
        raise RuntimeError("New error") from original
except RuntimeError as e:
    print(f"New error: {e}")
    print(f"Original cause: {e.__cause__}")
    print(f"Cause type: {type(e.__cause__).__name__}")
```

**Output:**

```
New error: New error
Original cause: Original error
Cause type: ValueError
```


### __context__

The `__context__` attribute holds the exception that was active when the new one was raised (implicit chaining):

```python
# Implicit chaining (no 'from')
try:
    try:
        raise ValueError("Original error")
    except ValueError:
        raise RuntimeError("New error")  # No 'from'
except RuntimeError as e:
    print(f"New error: {e}")
    print(f"Context: {e.__context__}")
    print(f"Context type: {type(e.__context__).__name__}")
```

**Output:**

```
New error: New error
Context: Original error
Context type: ValueError
```


### Chained Tracebacks

```python
def level_3():
    raise ValueError("Error from level 3")

def level_2():
    try:
        level_3()
    except ValueError as e:
        raise RuntimeError("Error from level 2") from e

def level_1():
    try:
        level_2()
    except RuntimeError as e:
        raise Exception("Error from level 1") from e

try:
    level_1()
except Exception:
    import traceback
    traceback.print_exc()
```

**Output:**

```
Traceback (most recent call last):
```

File "<stdin>", line 22, in <module>

```
File "<stdin>", line 18, in level_1
Exception: Error from level 1

The above exception was the direct cause of the following exception:

Traceback (most recent call last):
File "<stdin>", line 14, in level_2
File "<stdin>", line 6, in level_3
ValueError: Error from level 3
```


---

## 12. Custom Exceptions

### Creating Custom Exceptions

```python
# Basic custom exception
class CustomError(Exception):
    """My custom exception"""
    pass

try:
    raise CustomError("Something went wrong")
except CustomError as e:
    print(f"Caught: {e}")
```

**Output:**

```
Caught: Something went wrong
```


### Inheritance

```python
# Multi-level inheritance
class AppError(Exception):
    """Base exception for application"""
    pass

class ValidationError(AppError):
    """Validation-related errors"""
    pass

class InvalidEmailError(ValidationError):
    """Email validation error"""
    pass

class InvalidAgeError(ValidationError):
    """Age validation error"""
    pass

def validate_user(email, age):
    if "@" not in email:
        raise InvalidEmailError("Email must contain '@'")
    if age < 0 or age > 150:
        raise InvalidAgeError("Age must be 0-150")
    return True

# Catch specific exception
try:
    validate_user("invalid", 200)
except InvalidEmailError:
    print("Email invalid")
except InvalidAgeError:
    print("Age invalid")

# Catch parent exception
try:
    validate_user("invalid", 200)
except ValidationError:
    print("General validation error (catches both)")
```

**Output:**

```
Age invalid
General validation error (catches both)
```


### Domain-Specific Exceptions

```python
# Banking domain exceptions
class BankingError(Exception):
    """Base banking exception"""
    pass

class InsufficientFundsError(BankingError):
    def __init__(self, balance, amount):
        super().__init__(f"Insufficient funds: balance=${balance}, amount=${amount}")
        self.balance = balance
        self.amount = amount

class Invalid_transactionError(BankingError):
    def __init__(self, transaction_type, reason):
        super().__init__(f"Invalid {transaction_type}: {reason}")
        self.transaction_type = transaction_type
        self.reason = reason

class BankAccount:
    def __init__(self, balance):
        self.balance = balance
    
    def withdraw(self, amount):
        if amount > self.balance:
            raise InsufficientFundsError(self.balance, amount)
        self.balance -= amount
        return self.balance

account = BankAccount(100)
try:
    account.withdraw(150)
except InsufficientFundsError as e:
    print(f"Error: {e}")
    print(f"Balance: ${e.balance}, Requested: ${e.amount}")
```

**Output:**

```
Error: Insufficient funds: balance=$100, amount=$150
Balance: $100, Requested: $150
```


### Real-World Examples

```python
# Configuration validation
class ConfigError(Exception):
    """Configuration error"""
    pass

class MissingConfigKeyError(ConfigError):
    """Missing configuration key"""
    pass

class InvalidConfigValueError(ConfigError):
    """Invalid configuration value"""
    pass

class Config:
    def __init__(self, data):
        self.data = data
    
    def get_required(self, key):
        if key not in self.data:
            raise MissingConfigKeyKeyError(f"Required key '{key}' missing")
        return self.data[key]
    
    def get_int(self, key):
        value = self.get_required(key)
        try:
            return int(value)
        except (ValueError, TypeError):
            raise InvalidConfigValueError(f"{key} must be an integer")

config = Config({"host": "localhost"})
try:
    port = config.get_int("port")
except MissingConfigKeyError as e:
    print(f"Missing key: {e}")
except InvalidConfigValueError as e:
    print(f"Invalid value: {e}")
```

**Output:**

```
Missing key: Required key 'port' missing
```


---

## 13. Exception Objects

### args Attribute

All exception objects have an `args` attribute containing the arguments passed to the exception:

```python
exc = ValueError("Invalid value", "additional info")
print(f"args: {exc.args}")
print(f"len(args): {len(exc.args)}")
```

**Output:**

```
args: ('Invalid value', 'additional info')
len(args): 2
```

```python
# Single argument
exc = TypeError("Wrong type")
print(f"args: {exc.args}")
print(f"str(exc): {str(exc)}")
```

**Output:**

```
args: ('Wrong type',)
str(exc): Wrong type
```


### Attributes

Custom exceptions can have additional attributes:

```python
class DetailedError(Exception):
    def __init__(self, message, error_code, details):
        super().__init__(message)
        self.error_code = error_code
        self.details = details

exc = DetailedError("Operation failed", 500, {"key": "value"})
print(f"Message: {exc}")
print(f"Error code: {exc.error_code}")
print(f"Details: {exc.details}")
print(f"args: {exc.args}")
```

**Output:**

```
Message: Operation failed
Error code: 500
Details: {'key': 'value'}
args: ('Operation failed',)
```


### Custom State

```python
class StatefulError(Exception):
    def __init__(self, message, state):
        super().__init__(message)
        self.state = state
        self.timestamp = __import__('datetime').datetime.now()
    
    def __str__(self):
        return f"{self.timestamp}: {self.args} (state={self.state})"

exc = StatefulError("Error occurred", "pending")
print(exc)
print(f"State: {exc.state}")
```

**Output:**

```
2026-06-21 18:09:00: Error occurred (state=pending)
State: pending
```


### Serialization

```python
import json

class SerializableError(Exception):
    def __init__(self, message, error_code):
        super().__init__(message)
        self.error_code = error_code
    
    def to_dict(self):
        return {
            "type": type(self).__name__,
            "message": self.args,
            "error_code": self.error_code
        }

exc = SerializableError("Request failed", 400)
error_dict = exc.to_dict()
print(json.dumps(error_dict, indent=2))
```

**Output:**

```
{
  "type": "SerializableError",
  "message": "Request failed",
  "error_code": 400
}
```


---

## 14. Assertions

### assert Statement

```python
# Basic assert
age = 25
assert age > 0, "Age must be positive"
print(f"Age is valid: {age}")
```

**Output:**

```
Age is valid: 25
```

```python
# Assert fails
def set_age(age):
    assert age > 0, "Age must be positive"
    assert age < 150, "Age must be less than 150"
    return age

try:
    set_age(-5)
except AssertionError as e:
    print(f"Assertion failed: {e}")
```

**Output:**

```
Assertion failed: Age must be positive
```


### PYTHONOPTIMIZE

When Python runs with `-O` flag, assertions are removed:

```python
# This file demonstrates assertion behavior
# Run with: python -O script.py (assertions removed)
# Run with: python script.py (assertions active)

assert False, "This assertion will be removed with -O flag"
print("Script continues")
```

**Normal output:**

```
AssertionError: This assertion will be removed with -O flag
```

**With -O flag:**

```
Script continues
```


### Production Considerations

| Aspect | Consideration |
| :-- | :-- |
| **Debugging** | Use assertions for internal checks |
| **Production** | Don't rely on assertions for validation |
| **Performance** | Assertions removed with `-O` flag |
| **Security** | Use proper validation, not assertions |

```python
# BAD: Using assert for production validation
def process_user(age):
    assert age > 0  # Removed in production!
    return age

# GOOD: Use proper validation
def process_user(age):
    if age <= 0:
        raise ValueError("Age must be positive")
    return age

# Assertions for internal checks (good)
def calculate_average(numbers):
    assert len(numbers) > 0, "List should not be empty"  # Internal check
    return sum(numbers) / len(numbers)
```


---

## 15. Tracebacks

### Reading Tracebacks

```python
# Example traceback
def function_a():
    function_b()

def function_b():
    function_c()

def function_c():
    result = 10 / 0

function_a()
```

**Output:**

```
Traceback (most recent call last):
```

File "<stdin>", line 12, in <module>

```
File "<stdin>", line 3, in function_a
File "<stdin>", line 6, in function_b
File "<stdin>", line 9, in function_c
ZeroDivisionError: division by zero
```

**Reading the traceback:**

1. `Traceback (most recent call last):` - Indicates error location
2. Lines show call stack (bottom = where error occurred)
3. Last line: Exception type and message

### Debugging

```python
# Use traceback for debugging
import traceback

def buggy_function():
    data = {"key": "value"}
    return data["missing_key"]

try:
    buggy_function()
except KeyError:
    print("Full traceback:")
    traceback.print_exc()
    print()
    print("Formatted traceback as string:")
    print(traceback.format_exc())
```

**Output:**

```
Full traceback:
Traceback (most recent call last):
```

File "<stdin>", line 13, in <module>

```
File "<stdin>", line 5, in buggy_function
KeyError: 'missing_key'

Formatted traceback as string:
Traceback (most recent call last):
```

File "<stdin>", line 13, in <module>

```
File "<stdin>", line 5, in buggy_function
KeyError: 'missing_key'
```


### traceback Module

```python
import traceback
import sys

# Get traceback info
try:
    result = 10 / 0
except:
    # Get full traceback
    tb = traceback.format_exc()
    print(tb)
    
    # Get exception info
    exc_type, exc_value, exc_tb = sys.exc_info()
    print(f"Exception type: {exc_type}")
    print(f"Exception: {exc_value}")
    
    # Print stack summary
    print("Stack summary:")
    traceback.print_stack()
```

**Output:**

```
Traceback (most recent call last):
```

File "<stdin>", line 5, in <module>

```
ZeroDivisionError: division by zero

Exception type: <class 'ZeroDivisionError'>
Exception: division by zero
Stack summary:
```

<stdin>, line 17, in <module>

```
```

```python
# Extract specific traceback info
import traceback

try:
    raise ValueError("Test error")
except:
    tb_list = traceback.extract_tb(sys.exc_info())
    for frame in tb_list:
        print(f"File: {frame.filename}, Line: {frame.lineno}, Func: {frame.name}")
```

**Output:**

```
```

File: <stdin>, Line: 5, Func: <module>

```
```


---

## 16. Logging Exceptions

### logging.exception()

```python
import logging

logging.basicConfig(level=logging.ERROR)

def risky_operation():
    try:
        result = 10 / 0
    except ZeroDivisionError:
        logging.exception("Division failed")  # Includes traceback

risky_operation()
```

**Output:**

```
ERROR:__main__:Division failed
Traceback (most recent call last):
  File "<stdin>", line 7, in risky_operation
ZeroDivisionError: division by zero
```


### exc_info

```python
import logging

logging.basicConfig(level=logging.ERROR)

def log_with_exc_info():
    try:
        result = int("not a number")
    except ValueError:
        logging.error("Value conversion failed", exc_info=True)

log_with_exc_info()
```

**Output:**

```
ERROR:__main__:Value conversion failed
Traceback (most recent call last):
  File "<stdin>", line 7, in log_with_exc_info
ValueError: invalid literal for int() with base 10: 'not a number'
```


### Structured Logging

```python
import logging
import json

# Custom formatter for structured logging
class StructuredFormatter(logging.Formatter):
    def format(self, record):
        log_data = {
            "level": record.levelname,
            "message": record.getMessage(),
            "module": record.module,
            "function": record.funcName,
            "line": record.lineno
        }
        if record.exc_info:
            log_data["exception"] = {
                "type": record.exc_info.__name__,
                "message": str(record.exc_info)
            }
        return json.dumps(log_data)

logger = logging.getLogger("structured")
logger.setLevel(logging.ERROR)
handler = logging.StreamHandler()
handler.setFormatter(StructuredFormatter())
logger.addHandler(handler)

def structured_log_example():
    try:
        result = 10 / 0
    except ZeroDivisionError:
        logger.exception("Division error in structured log")

structured_log_example()
```

**Output:**

```
{"level": "ERROR", "message": "Division error in structured log", "module": "<stdin>", "function": "structured_log_example", "line": 10, "exception": {"type": "ZeroDivisionError", "message": "division by zero"}}
```


---

## 17. Context Managers and Exceptions

### with Statement

```python
# File handling with context manager
with open("data.txt", "w") as f:
    f.write("Hello, World!")
# File automatically closed, even if exception occurs
```


### __enter__ and __exit__

```python
class Resource:
    def __enter__(self):
        print("Entering resource")
        self.data = "loaded"
        return self
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        print("Exiting resource")
        if exc_type:
            print(f"Exception caught: {exc_type.__name__}: {exc_val}")
        return False  # Don't suppress exception

with Resource() as res:
    print(f"Resource data: {res.data}")
    # raise ValueError("Test exception")  # Uncomment to see exception handling

print("After with block")
```

**Output:**

```
Entering resource
Resource data: loaded
Exiting resource
After with block
```

```python
# With exception
with Resource() as res:
    print(f"Resource data: {res.data}")
    raise ValueError("Test exception")
```

**Output:**

```
Entering resource
Resource data: loaded
Exiting resource
Exception caught: ValueError: Test exception
Traceback (most recent call last):
```

File "<stdin>", line 3, in <module>

```
ValueError: Test exception
```


### Exception Suppression

```python
# Context manager that suppresses exceptions
class SuppressErrors:
    def __enter__(self):
        return self
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        if exc_type:
            print(f"Suppressing: {exc_type.__name__}: {exc_val}")
            return True  # Suppress exception
        return False

with SuppressErrors():
    print("Before exception")
    raise ValueError("This will be suppressed")
print("After with block - still executes!")
```

**Output:**

```
Before exception
Suppressing: ValueError: This will be suppressed
After with block - still executes!
```


### Practical Context Manager

```python
import time

class TimedOperation:
    def __enter__(self):
        self.start = time.time()
        print(f"Starting operation at {self.start}")
        return self
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        end = time.time()
        duration = end - self.start
        print(f"Operation completed in {duration:.2f}s")
        if exc_type:
            print(f"Exception: {exc_type.__name__}: {exc_val}")
        return False

with TimedOperation():
    print("Doing work...")
    time.sleep(0.1)
    print("Work done")
```

**Output:**

```
Starting operation at 1719018540.123456
Doing work...
Work done
Operation completed in 0.10s
```


---

## 18. Exception Handling in Functions and Classes

### Error Propagation

```python
# Function that propagates exceptions
def parse_int(value):
    return int(value)  # May raise ValueError

def process_value(value):
    try:
        number = parse_int(value)
        return number * 2
    except ValueError:
        return None

print(process_value("10"))   # 20
print(process_value("hello")) # None
```

**Output:**

```
20
None
```


### API Design

```python
# API that raises specific exceptions
class API:
    def __init__(self):
        self.base_url = "https://api.example.com"
    
    def get(self, endpoint):
        if endpoint == "/broken":
            raise ConnectionError("Cannot connect to API")
        if endpoint == "/forbidden":
            raise PermissionError("Access denied")
        return {"status": "success"}
    
    def post(self, endpoint, data):
        if not data:
            raise ValueError("Data cannot be empty")
        return {"status": "created"}

api = API()

# Handle specific exceptions
try:
    result = api.get("/broken")
except ConnectionError as e:
    print(f"Connection error: {e}")
    result = {"status": "cached"}
except PermissionError as e:
    print(f"Permission error: {e}")
    result = {"status": "denied"}

print(result)
```

**Output:**

```
Connection error: Cannot connect to API
{'status': 'cached'}
```


### OOP Exception Patterns

```python
# Pattern: Exception per class
class InvalidStateError(Exception):
    pass

class Machine:
    def __init__(self):
        self.state = "off"
    
    def start(self):
        if self.state != "off":
            raise InvalidStateError(f"Cannot start machine in {self.state} state")
        self.state = "on"
        print("Machine started")
    
    def stop(self):
        if self.state != "on":
            raise InvalidStateError(f"Cannot stop machine in {self.state} state")
        self.state = "off"
        print("Machine stopped")

machine = Machine()
machine.start()
machine.stop()

try:
    machine.stop()  # Already off
except InvalidStateError as e:
    print(f"Error: {e}")
```

**Output:**

```
Machine started
Machine stopped
Error: Cannot stop machine in off state
```

```python
# Pattern: Retry with exception
import time

class RetryError(Exception):
    pass

def retry_operation(func, max_retries=3):
    for attempt in range(max_retries):
        try:
            return func()
        except Exception as e:
            if attempt == max_retries - 1:
                raise RetryError(f"Failed after {max_retries} attempts") from e
            print(f"Attempt {attempt + 1} failed: {e}, retrying...")
            time.sleep(0.1)

def unstable_function():
    import random
    if random.random() < 0.7:  # 70% chance of failure
        raise ConnectionError("Temporary failure")
    return "success"

try:
    result = retry_operation(unstable_function)
    print(f"Result: {result}")
except RetryError as e:
    print(f"Retry failed: {e}")
```

**Output:**

```
Attempt 1 failed: Temporary failure, retrying...
Attempt 2 failed: Temporary failure, retrying...
Attempt 3 failed: Temporary failure, retrying...
Retry failed: Failed after 3 attempts
```


---

## 19. Best Practices

### 1. Catch Specific Exceptions First

```python
# GOOD
try:
    result = int("10")
except ValueError:
    print("Value error")
except TypeError:
    print("Type error")

# BAD - too broad
try:
    result = int("10")
except Exception:
    print("Any error")
```


### 2. Don't Catch Exceptions You Don't Handle

```python
# BAD
try:
    result = 10 / 0
except ZeroDivisionError:
    pass  # Silent failure

# GOOD
try:
    result = 10 / 0
except ZeroDivisionError:
    result = 0  # Handle it
    print("Handled division by zero")
```


### 3. Use else for Success Code

```python
# GOOD
try:
    file = open("data.txt", "r")
    content = file.read()
except FileNotFoundError:
    content = ""
else:
    file.close()  # Only close if file opened successfully

# BAD
try:
    file = open("data.txt", "r")
    content = file.read()
except FileNotFoundError:
    content = ""
file.close()  # May fail if file not found
```


### 4. Use finally for Cleanup

```python
# GOOD
def process_file(filename):
    file = None
    try:
        file = open(filename, "r")
        return file.read()
    finally:
        if file:
            file.close()
```


### 5. Raise Early, Catch Late

```python
# GOOD - Validate early
def set_age(age):
    if age < 0:
        raise ValueError("Age must be positive")
    return age

# BAD - Catch and continue
def set_age(age):
    try:
        if age < 0:
            raise ValueError()
    except ValueError:
        age = 0
    return age
```


### 6. Document Exceptions

```python
def divide(a, b):
    """
    Divide a by b.
    
    Args:
        a: Number to divide
        b: Number to divide by
    
    Returns:
        Result of division
    
    Raises:
        ZeroDivisionError: If b is zero
        TypeError: If a or b is not a number
    """
    if not isinstance(a, (int, float)) or not isinstance(b, (int, float)):
        raise TypeError("Arguments must be numbers")
    if b == 0:
        raise ZeroDivisionError("Cannot divide by zero")
    return a / b
```


### 7. Use Exception Chaining

```python
# GOOD
def read_config(filename):
    try:
        with open(filename, "r") as f:
            return json.load(f)
    except FileNotFoundError as e:
        raise ConfigError(f"Config file not found: {filename}") from e
```


### 8. Don't Suppress Exceptions Silently

```python
# BAD
try:
    risky_operation()
except:
    pass

# GOOD
try:
    risky_operation()
except Exception as e:
    logging.error(f"Operation failed: {e}")
    # Handle or re-raise
```


---

## 20. Anti-Patterns

### 1. Catching All Exceptions

```python
# ANTI-PATTERN
try:
    do_something()
except Exception:
    pass

# FIX
try:
    do_something()
except SpecificError:
    handle_error()
```


### 2. Silent Exception Suppression

```python
# ANTI-PATTERN
try:
    result = risky_operation()
except:
    result = None  # Silent failure

# FIX
try:
    result = risky_operation()
except SpecificError as e:
    logging.warning(f"Operation failed: {e}")
    result = None
```


### 3. Using Exception for Control Flow

```python
# ANTI-PATTERN
try:
    value = dictionary[key]
except KeyError:
    value = default

# FIX (use get)
value = dictionary.get(key, default)
```


### 4. Catching BaseException

```python
# ANTI-PATTERN
try:
    do_something()
except BaseException:
    pass  # Catches KeyboardInterrupt, SystemExit!

# FIX
try:
    do_something()
except Exception:
    pass
```


### 5. Returning from finally

```python
# ANTI-PATTERN
def bad_function():
    try:
        return "from try"
    finally:
        return "from finally"  # Overrides!

# FIX
def good_function():
    try:
        return "from try"
    finally:
        cleanup()  # Don't return!
```


### 6. Raising Without Message

```python
# ANTI-PATTERN
try:
    risky_operation()
except ValueError:
    raise  # Re-raises but loses context in some cases

# FIX
try:
    risky_operation()
except ValueError as e:
    raise ValueError("More context") from e
```


### 7. Empty except Blocks

```python
# ANTI-PATTERN
try:
    operation()
except ValueError:
    pass

# FIX
try:
    operation()
except ValueError as e:
    logging.debug(f"Expected error: {e}")
```


### 8. Overly Broad Exception Handling

```python
# ANTI-PATTERN
def process(data):
    try:
        return int(data)
    except Exception:
        return 0  # Catches everything!

# FIX
def process(data):
    try:
        return int(data)
    except (ValueError, TypeError):
        return 0
```


---

## 21. Interview Questions (Minimum 25)

### Basic Questions

1. **What is an exception in Python?**
    - An event that disrupts normal program flow during execution
2. **How do you handle exceptions in Python?**
    - Using `try`, `except`, `else`, and `finally` blocks
3. **What is the difference between errors and exceptions?**
    - Errors are syntax problems (can't be handled); exceptions are runtime conditions (can be handled)
4. **What happens when an exception is not caught?**
    - Program terminates with traceback
5. **What is the purpose of the `else` clause in try-except?**
    - Executes only when no exception occurs in try block

### Intermediate Questions

6. **Explain exception propagation**
    - Exceptions travel up the call stack until caught or program terminates
7. **What is the difference between `except Exception` and `except BaseException`?**
    - `BaseException` catches system exceptions (KeyboardInterrupt, SystemExit); `Exception` doesn't
8. **When does the `finally` block execute?**
    - Always, even with return statements or exceptions
9. **What is exception chaining?**
    - When one exception leads to another, using `raise ... from`
10. **How do you create a custom exception?**
    - Inherit from `Exception` class
11. **What is the difference between `__cause__` and `__context__`?**
    - `__cause__` is explicit chaining (`from`); `__context__` is implicit
12. **What does `assert` do and when should you use it?**
    - Checks condition; use for debugging/internal checks, not production validation
13. **How do you log exceptions with traceback?**
    - Use `logging.exception()` or `logging.error(..., exc_info=True)`

### Advanced Questions

14. **Explain the exception hierarchy in Python**
    - `BaseException` → `Exception` → specific exceptions (ValueError, TypeError, etc.)
15. **What is the difference between `KeyError` and `IndexError`?**
    - `KeyError`: dictionary key not found; `IndexError`: sequence index out of range
16. **How do context managers handle exceptions?**
    - `__exit__` receives exception info; can suppress by returning True
17. **What is the purpose of `raise from`?**
    - Explicit exception chaining to preserve original exception context
18. **When would you use `except Exception` instead of specific exceptions?**
    - When you want to catch any user-level exception (e.g., top-level handler)
19. **How do you serialize custom exceptions?**
    - Add `to_dict()` method converting attributes to dictionary
20. **What happens if you return from a `finally` block?**
    - Return value overrides any previous return
21. **Explain `MemoryError` and when it occurs**
    - Not enough memory for operation; hard to trigger in Python

### Scenario Questions

22. **How would you handle file operations safely?**
    - Use `try/except FileNotFoundError` with `finally` for cleanup or `with` statement

23

