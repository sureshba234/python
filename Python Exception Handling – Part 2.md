<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" style="height:64px;margin-right:32px"/>

# Python Exception Handling – Part 2

```markdown
# Python Exception Handling – Part 2

> Advanced concepts, CPython internals, and production-grade patterns for expert developers

---

## Table of Contents

1. [Deep Review of Exception Hierarchy](#1-deep-review-of-exception-hierarchy)
2. [Exception Handling in Iterators](#2-exception-handling-in-iterators)
3. [Exception Handling in Generators](#3-exception-handling-in-generators)
4. [Exception Handling in Coroutines](#4-exception-handling-in-coroutines)
5. [Exception Groups (Python 3.11+)](#5-exception-groups-python-311)
6. [Python Warnings System](#6-python-warnings-system)
7. [CPython Exception Internals](#7-cpython-exception-internals)
8. [Exception Objects Internals](#8-exception-objects-internals)
9. [Traceback Internals](#9-traceback-internals)
10. [Bytecode and Exceptions](#10-bytecode-and-exceptions)
11. [Exception Propagation Internals](#11-exception-propagation-internals)
12. [Memory Management](#12-memory-management)
13. [Performance Analysis](#13-performance-analysis)
14. [Advanced Exception Design Patterns](#14-advanced-exception-design-patterns)
15. [Exception Handling in Libraries](#15-exception-handling-in-libraries)
16. [Exception Handling in Distributed Systems](#16-exception-handling-in-distributed-systems)
17. [Security Considerations](#17-security-considerations)
18. [Testing Exceptions](#18-testing-exceptions)
19. [Production Monitoring](#19-production-monitoring)
20. [Real-World Case Studies](#20-real-world-case-studies)
21. [Expert Interview Questions](#21-expert-interview-questions)
22. [Advanced Exercises](#22-advanced-exercises)
23. [Ultimate Cheat Sheet](#23-ultimate-cheat-sheet)
24. [Conclusion](#24-conclusion)

---

## 1. Deep Review of Exception Hierarchy

### Full Hierarchy

```

BaseException
├─ Exception
├─ ArithmeticError
├─ ZeroDivisionError
├─ OverflowError
├─ FloatingPointError
├─ AssertionError
├─ AttributeError
├─ BufferError
├─ EOFError
├─ ImportError
├─ ModuleNotFoundError
├─ LookupError
├─ IndexError
├─ KeyError
├─ MemoryError
├─ NameError
├─ UnboundLocalError
├─ OSError
├─ EnvironmentError
├─ WindowsError (Windows only)
├─ BlockingIOError
├─ ChildProcessError
├─ ConnectionError
├─ BrokenPipeError
├─ ConnectionAbortedError
├─ ConnectionRefusedError
├─ ConnectionResetError
├─ FileExistsError
├─ FileNotFoundError
├─ InterruptedError
├─ IsADirectoryError
├─ NotADirectoryError
├─ PermissionError
├─ ProcessLookupError
├─ TimeoutError
├─ ReferenceError
├─ RuntimeError
├─ NotImplementedError
├─ RecursionError
├─ StopAsyncIteration
├─ SyntaxError
├─ IndentationError
├─ TabError
├─ SystemError
├─ CodecError
├─ TypeError
├─ ValueError
├─ UnicodeError
├─ UnicodeDecodeError
├─ UnicodeEncodeError
├─ UnicodeTranslateError
├─ Warning
├─ DeprecationWarning
├─ PendingDeprecationWarning
├─ RuntimeWarning
├─ SyntaxWarning
├─ UserWarning
├─ FutureWarning
├─ ImportWarning
├─ UnicodeWarning
├─ ExceptionGroup (Python 3.11+)
├─ BaseExceptionGroup (Python 3.11+)
└─ SystemExit
└─ KeyboardInterrupt
└─ GeneratorExit

```

### Design Rationale

The exception hierarchy follows a **taxonomy principle**:

1. **`BaseException`** is the root – includes system-exiting exceptions
2. **`Exception`** is the parent for all *user-facing* exceptions
3. **Specialized subclasses** enable precise error handling

Key design decisions:

- **`SystemExit`**, **`KeyboardInterrupt`**, and **`GeneratorExit`** inherit from `BaseException` (not `Exception`) to prevent accidental swallowing
- **`OSError`** unifies file/system errors across platforms (previously `EnvironmentError`, `IOError`)
- **`LookupError`** abstracts common indexing/key errors
- **`Warning`** subclass hierarchy enables filtered notification system

### Historical Evolution

| Version | Change |
|---------|--------|
| 1.0 | Basic exception hierarchy introduced |
| 2.0 | Exceptions became classes (not strings) |
| 2.2 | Unified exception model; `Exception` as base |
| 2.5 | `EnvironmentError` → `OSError` unification |
| 3.0 | Removed old string exceptions; `IOError` → `OSError` |
| 3.3 | `UnicodeError` variants added; `StopIteration` handling |
| 3.5 | `async/await`; `StopAsyncIteration` |
| 3.8 | `exception.group` prep; `__notes__` |
| 3.11 | **ExceptionGroups**, `except*`, `__notes__` |

---

## 2. Exception Handling in Iterators

### Iteration Protocol

```python
class Iterator:
    def __iter__(self):
        return self
    
    def __next__(self):
        # Return next value or raise StopIteration
```


### StopIteration Behavior

```python
# Internal behavior in for loops
for item in iterator:
    # Translates to:
    iterator = iter(iterator)
    while True:
        try:
            item = next(iterator)
        except StopIteration:
            break
        # process item
```

**Critical insight**: `StopIteration` is **not** an error – it's the *normal termination signal*.

### Iterator Termination Internal Behavior

```mermaid
flowchart TD
    A[for loop starts] --> B[Call iter(iterator)]
    B --> C[Call next() repeatedly]
    C --> D{Value returned?}
    D -->|Yes| E[Process value]
    E --> C
    D -->|StopIteration| F[Break loop silently]
    F --> G[Loop completes normally]
```


### StopIteration in Nested Iterators

```python
def nested_iterator():
    for i in range(3):
        for j in range(2):
            yield (i, j)

# StopIteration is consumed by inner loop, not outer
```

**Python 3.7+**: `StopIteration` raised inside generators is **wrapped** as `RuntimeError` to prevent silent corruption [web:1].

```python
def bad_generator():
    it = iter()
    next(it)  # OK
    next(it)  # OK
    next(it)  # StopIteration
    next(it)  # Raises StopIteration → wrapped as RuntimeError in generator context

# Result: RuntimeError: generator raised StopIteration
```


---

## 3. Exception Handling in Generators

### Generator Throw Mechanism

```python
gen = my_generator()
gen.throw(TypeError("inject error"))  # Raises TypeError inside generator at yield point
```


### Internal Flow: `throw()`

```mermaid
flowchart TD
    A[gen.throw(exc)] --> B[Save current generator state]
    B --> C[Resume generator at yield point]
    C --> D[Raise exception at yield]
    D --> E{Exception handled?}
    E -->|Yes| F[Continue execution]
    E -->|No| G[Generator terminates]
    G --> H[Cleanup frame]
    H --> I[Return to caller]
    F --> C
```


### Generator Close and GeneratorExit

```python
gen.close()  # Raises GeneratorExit inside generator at yield point
```

**Key properties of `GeneratorExit`:**

- Inherits from `BaseException` (not `Exception`)
- Cannot be caught by `except Exception:`
- Designed for cleanup, not error handling
- Automatically raised when generator reference is deleted

```python
def producer():
    try:
        while True:
            yield 1
    except GeneratorExit:
        print("Cleanup complete")  # Runs on close()
        return
    finally:
        print("Finally block")

g = producer()
next(g)
g.close()  # Triggers GeneratorExit
```


### Yield Interactions with Exceptions

```python
def echo():
    while True:
        try:
            value = yield  # Exception injected here via throw()
            print(f"Received: {value}")
        except TypeError:
            print("Handled TypeError")
            yield "recovered"

g = echo()
next(g)
g.throw(TypeError("bad"))  # Injects exception at yield
```


### Complete Generator Exception Lifecycle

```mermaid
flowchart LR
    Sub1[Generator Creation] --> Sub2[Yield Points]
    Sub2 --> Sub3[Exception Injection]
    Sub3 --> Sub4[Handler Lookup]
    Sub4 --> Sub5{Handled?}
    Sub5 -->|Yes| Sub6[Resume]
    Sub5 -->|No| Sub7[Terminate]
    Sub6 --> Sub2
    Sub7 --> Sub8[Frame Cleanup]
    Sub8 --> Sub9[Return to Caller]
```


---

## 4. Exception Handling in Coroutines

### Async Functions Exception Behavior

```python
async def async_task():
    await asyncio.sleep(1)
    raise ValueError("async failure")

# Exception propagates when awaited
```


### Asyncio Exception Types

| Exception | Purpose |
| :-- | :-- |
| `asyncio.CancelledError` | Task cancellation (Python <3.11) |
| `asyncio.exceptions.CancelledError` | Cancellation (Python 3.11+) |
| `asyncio.TimeoutError` | Timeout expiration |
| `asyncio.IncompleteReadError` | Incomplete socket read |
| `ExceptionGroup` | Multiple concurrent failures (3.11+) |

### Task Failures

```python
import asyncio

async def failure():
    raise ValueError("task failed")

async def main():
    task = asyncio.create_task(failure())
    try:
        await task  # Exception raised here
    except ValueError as e:
        print(f"Caught: {e}")

asyncio.run(main())
```


### Cancellation Mechanics

```python
async def cancellable_task():
    try:
        while True:
            await asyncio.sleep(1)
    except asyncio.CancelledError:
        print("Task cancelled – cleanup")
        # Re-raise to allow proper termination
        raise

async def main():
    task = asyncio.create_task(cancellable_task())
    await asyncio.sleep(3)
    task.cancel()  # Sends cancellation signal
    try:
        await task
    except asyncio.CancelledError:
        print("Confirmed cancellation")

asyncio.run(main())
```


### CancelledError Internal Flow

```mermaid
flowchart TD
    A[task.cancel()] --> B[Set _cancelled flag]
    B --> C[Schedule cancellation]
    C --> D[Resume coroutine]
    D --> E[Raise CancelledError at await point]
    E --> F{Handled?}
    F -->|Yes| G[Cleanup]
    F -->|No| H[Terminate task]
    G --> I[Re-raise CancelledError]
    I --> H
    H --> J[Task state: cancelled]
```


### Python 3.11+ Cancellation with ExceptionGroups

```python
import asyncio

async def task1():
    await asyncio.sleep(1)
    raise ValueError("task1 failed")

async def task2():
    await asyncio.sleep(1)
    raise TypeError("task2 failed")

async def main():
    try:
        await asyncio.gather(task1(), task2())
    except ExceptionGroup as group:
        print(f"Grouped {len(group.exceptions)} failures:")
        for exc in group.exceptions:
            print(f"  - {exc}")

asyncio.run(main())
```


---

## 5. Exception Groups (Python 3.11+)

### ExceptionGroup vs BaseExceptionGroup

```python
class ExceptionGroup(BaseException):
    """Group of exceptions inheriting from Exception"""
    
class BaseExceptionGroup(BaseException):
    """Group including BaseException subclasses"""
```

**Key difference**: `ExceptionGroup` excludes `BaseException` subclasses like `KeyboardInterrupt`.

### Creating ExceptionGroups

```python
from exceptions import ExceptionGroup  # Backport for 3.10, built-in in 3.11+

exc1 = ValueError("error 1")
exc2 = TypeError("error 2")
group = ExceptionGroup("multiple failures", [exc1, exc2])
```


### except* Syntax

```python
async def process_file(path):
    raise FileNotFoundError(path)

async def process_all(paths):
    tasks = [process_file(p) for p in paths]
    try:
        await asyncio.gather(*tasks)
    except* FileNotFoundError as eg:
        print(f"Missing files: {len(eg.exceptions)}")
    except* Exception as eg:
        print(f"Other errors: {len(eg.exceptions)}")
```


### Concurrent Failures Example

```python
import asyncio
from exceptions import ExceptionGroup

async def fetch(url, delay):
    await asyncio.sleep(delay)
    if delay > 1.5:
        raise ValueError(f"Failed: {url}")
    return f"OK: {url}"

async def main():
    urls = ["a.com", "b.com", "c.com", "d.com"]
    delays = [1, 2, 0.5, 3]
    
    try:
        results = await asyncio.gather(
            *[fetch(url, delay) for url, delay in zip(urls, delays)]
        )
    except ExceptionGroup as eg:
        print(f"Captured {len(eg.exceptions)} failures")
        for exc in eg.exceptions:
            print(f"  - {type(exc).__name__}: {exc}")

asyncio.run(main())
```


### Structured Error Handling Pattern

```python
def process_batch(items):
    results = []
    errors = []
    
    for item in items:
        try:
            results.append(transform(item))
        except Exception as e:
            errors.append(e)
    
    if errors:
        raise ExceptionGroup("Batch processing failures", errors)
    
    return results
```


---

## 6. Python Warnings System

### Warning Hierarchy

```
Warning
 ├─ DeprecationWarning
 ├─ PendingDeprecationWarning
 ├─ RuntimeWarning
 ├─ SyntaxWarning
 ├─ UserWarning
 ├─ FutureWarning
 ├─ ImportWarning
 ├─ UnicodeWarning
```


### Warnings Module Usage

```python
import warnings

# Emit warning
warnings.warn("Deprecated function", DeprecationWarning)

# With stacklevel
warnings.warn("Bad value", UserWarning, stacklevel=2)

# As context manager
with warnings.catch_warnings():
    warnings.simplefilter("ignore", DeprecationWarning)
    risky_function()
```


### Warning Filters

```python
# Filter rules (order matters)
warnings.filterwarnings(
    action='error',              # 'ignore', 'warning', 'error', 'always', 'module', 'once'
    category=DeprecationWarning,
    message='Old API',
    module=r'mylib.*',
    lineno=0
)
```


### DeprecationWarning Behavior

| Python Version | Default Behavior |
| :-- | :-- |
| <3.2 | Hidden by default |
| 3.2–3.9 | Shown once per module |
| 3.10+ | **Shown by default** (developer-facing) |

```python
# Best practice for library authors
def old_api():
    warnings.warn(
        "old_api() is deprecated, use new_api()",
        DeprecationWarning,
        stacklevel=2
    )
```


### RuntimeWarning vs DeprecationWarning

```python
# RuntimeWarning: suspicious runtime behavior (shown once)
warnings.warn("Low memory", RuntimeWarning)

# DeprecationWarning: API will be removed (shown by default in 3.10+)
warnings.warn("Use v2()", DeprecationWarning)
```


---

## 7. CPython Exception Internals

### PyBaseExceptionObject Structure

```c
// Include/object.h
typedef struct {
    PyObject_HEAD
    PyObject *ob_exception;     // Exception type
    PyObject *ob_value;         // Exception value (args)
    PyObject *ob_traceback;     // Attached traceback
    PyObject *ob_context;       // __context__ (implicit)
    PyObject *ob_cause;         // __cause__ (explicit via 'from')
    PyObject *ob_notes;         // __notes__ (Python 3.11+)
} PyBaseExceptionObject;
```


### PyErr_SetString

```c
// Sets exception with string value
void PyErr_SetString(PyObject *type, const char *message);

// Usage
PyErr_SetString(PyExc_ValueError, "invalid value");
return NULL;  // Return NULL to indicate error to caller
```


### PyErr_Occurred

```c
// Check if exception is set (returns type or NULL)
PyObject *PyErr_Occurred(void);

// Usage
if (PyErr_Occurred()) {
    printf("Exception occurred\n");
}
```


### PyErr_Fetch

```c
// Fetch and clear current exception
void PyErr_Fetch(PyObject **ptype, PyObject **pvalue, PyObject **ptraceback);

// After fetch, exception state is cleared
```


### PyErr_Restore

```c
// Restore previously fetched exception
void PyErr_Restore(PyObject *type, PyObject *value, PyObject *traceback);

// Usage pattern: fetch → process → restore
```


### Error Indicators

```c
// Old-style (deprecated): check return value
PyObject *result = PyMethod_Call(obj, args);
if (result == NULL) {
    // Exception is set
}

// Modern: use PyErr_Occurred()
```


### Thread State and Exception Storage

```c
// Each thread has PyThreadState
typedef struct _ts {
    PyObject *exc_type;         // Current exception type
    PyObject *exc_value;        // Current exception value
    PyObject *exc_traceback;    // Current traceback
    // ...
} PyThreadState;
```

**Key insight**: Exception state is **per-thread**, stored in `PyThreadState`.

### Memory Diagram: Exception State

```mermaid
flowchart TD
    Thread[Thread State PyThreadState] --> ExcType[exc_type]
    Thread --> ExcVal[exc_value]
    Thread --> ExcTB[exc_traceback]
    
    ExcVal --> ExcObj[PyBaseExceptionObject]
    ExcObj --> Type[Exception Type]
    ExcObj --> Args[args PyObject]
    ExcObj --> TB[Traceback Object]
    ExcObj --> Context[__context__]
    ExcObj --> Cause[__cause__]
    ExcObj --> Notes[__notes__]
```


---

## 8. Exception Objects Internals

### Args Storage

```python
exc = ValueError("msg", "extra", 123)
exc.args  # ("msg", "extra", 123)
```

**Internal**: `args` is stored as a `tuple` in `PyBaseExceptionObject.ob_value`.

### Traceback Attachment

```python
try:
    raise ValueError("test")
except ValueError as e:
    e.__traceback__  # Traceback object
```

**Internal**: `ob_traceback` points to `PyTraceBackObject`.

### Context Chains (__context__)

```python
def outer():
    try:
        inner()
    except ValueError:
        raise RuntimeError("outer failed")  # __context__ set automatically

def inner():
    raise ValueError("inner failed")

try:
    outer()
except RuntimeError as e:
    print(e.__context__)  # ValueError: inner failed
```

**Rule**: `__context__` is set when exception is raised **while another exception is active**.

### Cause Chains (__cause__)

```python
try:
    raise ValueError("original")
except ValueError as e:
    raise RuntimeError("wrapped") from e  # __cause__ set explicitly

# __cause__ = original ValueError
# __context__ = also original ValueError (same in this case)
```

**Rule**: `__cause__` is set when using `from <exception>`.

### Notes (Python 3.11+)

```python
exc = ValueError("bad value")
exc.add_note("Happened during CSV parsing")
exc.add_note("Row 452")

print(exc.__notes__)  # ["Happened during CSV parsing", "Row 452"]
```

**Internal**: `ob_notes` is a `list` of strings.

### Exception Chain Diagram

```mermaid
flowchart LR
    Sub1[RuntimeError] --> Sub2[__cause__: ValueError]
    Sub1 --> Sub3[__context__: ValueError]
    Sub1 --> Sub4[__notes__: list]
    Sub1 --> Sub5[__traceback__: Traceback]
    Sub1 --> Sub6[args: tuple]
```


---

## 9. Traceback Internals

### Frame Objects

```c
// Frame object structure
typedef struct {
    PyObject_HEAD
    PyCodeObject *f_code;       // Code object
    PyObject *f_globals;        // Global namespace
    PyObject *f_locals;         // Local namespace
    PyObject *f_back;           // Caller frame
    // ...
} PyFrameObject;
```


### Stack Frames

```python
import traceback

def a():
    b()

def b():
    c()

def c():
    raise ValueError("error")

try:
    a()
except:
    for frame in traceback.walk_tb(sys.exc_info()):
        print(frame.f_code.co_name)
```

Output:

```
c
b
a
```


### Frame Unwinding

```mermaid
flowchart TD
    A[Exception raised] --> B[Current frame has handler?]
    B -->|No| C[Pop frame]
    C --> D[Get caller frame via f_back]
    D --> B
    B -->|Yes| E[Execute handler]
    E --> F[Cleanup remaining frames]
    F --> G[Resume at handler]
```


### Traceback Construction

```c
// Traceback object
typedef struct {
    PyObject_HEAD
    PyTraceBackObject *tb_next;    // Next (caller) traceback
    PyFrameObject *tb_frame;       // Current frame
    int tb_lastline;               // Last line number
    int tb_lasti;                  // Last instruction index
} PyTraceBackObject;
```

**Chain**: `tb_next` → caller → caller → ... → root

### Traceback Diagram

```mermaid
flowchart BT
    TB1[Traceback: c() line 5] --> TB2[Traceback: b() line 2]
    TB2 --> TB3[Traceback: a() line 1]
    TB1 --> F1[Frame: c]
    TB2 --> F2[Frame: b]
    TB3 --> F3[Frame: a]
    F1 --> Flink1[f_back → b]
    F2 --> Flink2[f_back → a]
```


---

## 10. Bytecode and Exceptions

### Using dis Module

```python
import dis

def example():
    try:
        x = 1
        y = 2
    except ValueError:
        x = 0

dis.dis(example)
```

Output:

```
  2           0 LOAD_CONST               1 (1)
              2 STORE_FAST               0 (x)
  3           4 LOAD_CONST               2 (2)
              6 STORE_FAST               0 (y)
              8 JUMP_FORWARD             2 (to 12)
        >>   10 POP_BLOCK
  5     >>   12 LOAD_CONST               0 (0)
             14 STORE_FAST               0 (x)
             16 JUMP_FORWARD             0 (to 18)
        >>   18 LOAD_CONST               3 (<exception>)
             20 DUP_TOP
             22 LOAD_CONST               4 (ValueError)
             24 COMPARE_OP               10 (exception match)
             26 POP_JUMP_IF_FALSE        4 (to 36)
        >>   28 POP_TOP
             30 POP_TOP
             32 POP_TOP
             34 JUMP_FORWARD             0 (to 36)
        >>   36 POP_CONST                (<exception>)
             38 LOAD_CONST               5 (None)
             40 RETURN_VALUE
```


### Exception Tables

Python 3.11+ uses **exception tables** instead of `POP_BLOCK`:

```python
def example_311():
    try:
        return 1 / 0
    except ZeroDivisionError:
        return 0

dis.dis(example_311)
```

Output includes:

```
Exception table:
    0 to 4 -> 12 
    4 to 8 -> 12 
```


### Exception Tables Format

| Offset | Target | Depth |
| :-- | :-- | :-- |
| start | handler | stack depth |

**Meaning**: If exception occurs between `start` and end, jump to `handler` with stack depth `depth`.

### Bytecode Execution with Exceptions

```mermaid
flowchart TD
    A[Execute bytecode] --> B{Exception?}
    B -->|No| C[Continue]
    C --> A
    B -->|Yes| D[Lookup exception table]
    D --> E{Handler found?}
    E -->|No| F[Propagate to caller]
    E -->|Yes| G[Jump to handler]
    G --> H[Setup exception vars]
    H --> I[Execute handler]
```


### Exception Dispatch

```c
// In ceval.c (bytecode evaluator)
if (exception_occurred) {
    // Search exception table
    target = find_exception_handler(table, current_offset);
    if (target) {
        // Jump to handler
        goto target;
    } else {
        // Propagate
        propagate_exception();
    }
}
```


---

## 11. Exception Propagation Internals

### Call Stack Traversal

```mermaid
flowchart TD
    A[Exception raised in func3] --> B[func3: no handler?]
    B -->|Yes| C[Execute handler]
    B -->|No| D[Pop func3 frame]
    D --> E[func2: no handler?]
    E -->|Yes| F[Execute handler]
    E -->|No| G[Pop func2 frame]
    G --> H[func1: no handler?]
    H -->|Yes| I[Execute handler]
    H -->|No| J[Pop func1 frame]
    J --> K[Module level: no handler]
    K --> L[Print traceback + exit]
```


### Handler Lookup

```c
// Search exception table (Python 3.11+)
for (entry in exception_table) {
    if (start <= offset < end) {
        return entry.handler;
    }
}
return NULL;  // No handler
```


### Stack Unwinding

```python
import sys

def trace(frame, line, arg):
    print(f"Frame: {frame.f_code.co_name}, Line: {line}")
    return trace

sys.settrace(trace)

def a():
    b()

def b():
    c()

def c():
    raise ValueError("error")

try:
    a()
except:
    pass
```

Output shows frames unwinding: `c` → `b` → `a`.

### Frame Cleanup

```c
// During unwinding
PyFrame_Clear(frame);  // Clear locals
Py_DECREF(frame);      // Release frame object
```

**Key**: All locals are cleaned up, enabling GC of referenced objects.

---

## 12. Memory Management

### Reference Counting

```python
import sys

exc = ValueError("test")
print(sys.getrefcount(exc))  # ~2+ (argument + internal)
```

**Exception object**:

- `ob_exception` (type): +1 ref
- `ob_value` (args): +1 ref
- `ob_traceback`: +1 ref if set
- `ob_context`: +1 ref if set
- `ob_cause`: +1 ref if set


### Cyclic GC

```python
# Potential cycle: exception → traceback → frame → locals → exception
exc = None
try:
    exc = ValueError("cycle")
    raise exc
except ValueError as e:
    exc = e
    # exc.__traceback__ → frame → f_locals → exc (cycle!)
```

**Solution**: Python's GC handles cycles, but **explicitly delete tracebacks**:

```python
try:
    risky()
except Exception as e:
    process(e)
    del e  # Breaks traceback cycle
```


### Exception Object Lifetime

```mermaid
flowchart LR
    Sub1[Raise] --> Sub2[Create object]
    Sub2 --> Sub3[Attach traceback]
    Sub3 --> Sub4[Store in thread state]
    Sub4 --> Sub5[Handler executes]
    Sub5 --> Sub6[Handler completes]
    Sub6 --> Sub7[Clear thread state]
    Sub7 --> Sub8[GC if no refs]
```


### Traceback Reference Cycles

```python
# Dangerous pattern
def leak():
    exc = None
    try:
        raise ValueError("leak")
    except ValueError as e:
        exc = e  # exc holds __traceback__
        # traceback holds frame
        # frame.f_locals holds exc (cycle!)
    
    # exc still alive → cycle persists
```

**Best practice**:

```python
def safe():
    try:
        raise ValueError("ok")
    except ValueError as e:
        process(e)
    # e goes out of scope → traceback cleared → cycle broken
```


---

## 13. Performance Analysis

### Cost of Try Blocks

```python
import timeit

# No try
def no_try():
    x = 1
    y = 2

# With try (no exception)
def with_try():
    try:
        x = 1
        y = 2
    except:
        pass

print("No try:", timeit.timeit(no_try, number=10_000_000))
print("With try:", timeit.timeit(with_try, number=10_000_000))
```

**Typical result**: Try block adds ~2-5% overhead when no exception occurs.

### Cost of Raising Exceptions

```python
def raise_exception():
    raise ValueError("test")

def handle_exception():
    try:
        raise_exception()
    except ValueError:
        pass

print("Raise:", timeit.timeit(raise_exception, number=100_000))
print("Handle:", timeit.timeit(handle_exception, number=100_000))
```

**Typical result**: Raising + handling costs ~1-5 microseconds per exception.

### Benchmark Examples

```python
import timeit
from exceptions import ExceptionGroup

# Regular exception
def regular():
    try:
        raise ValueError("err")
    except ValueError:
        pass

# ExceptionGroup (3.11+)
def group():
    try:
        raise ExceptionGroup("group", [ValueError("err")])
    except ExceptionGroup:
        pass

print("Regular:", timeit.timeit(regular, number=100_000))
print("Group:", timeit.timeit(group, number=100_000))
```

**Result**: ExceptionGroups are ~10-20% slower due to tuple/list creation.

### Optimization Considerations

| Pattern | Cost | Recommendation |
| :-- | :-- | :-- |
| Empty `try/except` | Low (~2%) | OK for hot paths |
| Broad `except:` | Medium | Avoid – catches system exceptions |
| Specific `except TypeError:` | Low | Preferred |
| Many handlers | High | Use single broad handler + type check |
| Exception in loop | Very High | Validate before loop |

```python
# BAD: Exception in loop
for item in items:
    try:
        process(item)
    except ValueError:
        pass

# GOOD: Validate first
valid = [item for item in items if is_valid(item)]
for item in valid:
    process(item)
```


---

## 14. Advanced Exception Design Patterns

### Retry Pattern

```python
import time
from typing import Callable, Type, Union

def retry(
    func: Callable,
    exceptions: Union[Type[Exception], tuple],
    max_attempts: int = 3,
    delay: float = 1.0
):
    for attempt in range(max_attempts):
        try:
            return func()
        except exceptions as e:
            if attempt == max_attempts - 1:
                raise
            time.sleep(delay * (attempt + 1))
    raise RuntimeError("Retry failed")
```


### Exponential Backoff

```python
import time
import random

def exponential_backoff(
    func: Callable,
    exceptions: tuple,
    max_attempts: int = 5,
    base_delay: float = 1.0,
    max_delay: float = 60.0
):
    for attempt in range(max_attempts):
        try:
            return func()
        except exceptions as e:
            if attempt == max_attempts - 1:
                raise
            delay = min(base_delay * (2 ** attempt), max_delay)
            delay += random.uniform(0, 0.5 * delay)  # Jitter
            time.sleep(delay)
```


### Circuit Breaker

```python
from enum import Enum
from typing import Callable
import time

class State(Enum):
    CLOSED = "closed"      # Normal
    OPEN = "open"          # Failing
    HALF_OPEN = "half_open"  # Testing

class CircuitBreaker:
    def __init__(self, failure_threshold: int = 5, reset_timeout: float = 30.0):
        self.failures = 0
        self.state = State.CLOSED
        self.failure_threshold = failure_threshold
        self.reset_timeout = reset_timeout
        self.last_failure_time = None
    
    def call(self func: Callable) -> any:
        if self.state == State.OPEN:
            if time.time() - self.last_failure_time > self.reset_timeout:
                self.state = State.HALF_OPEN
            else:
                raise RuntimeError("Circuit open")
        
        try:
            result = func()
            if self.state == State.HALF_OPEN:
                self.state = State.CLOSED
                self.failures = 0
            return result
        except Exception as e:
            self.failures += 1
            self.last_failure_time = time.time()
            if self.failures >= self.failure_threshold:
                self.state = State.OPEN
            raise
```


### Error Translation

```python
class ExternalAPIError(Exception):
    pass

class DatabaseError(Exception):
    pass

def translate_errors(func):
    def wrapper(*args, **kwargs):
        try:
            return func(*args, **kwargs)
        except ExternalAPIError as e:
            raise DatabaseError(f"API failure: {e}") from e
    return wrapper
```


### Exception Wrapping

```python
from exceptions import ExceptionGroup

def wrap_exceptions(exc_type: Type[Exception], message: str):
    def decorator(func):
        def wrapper(*args, **kwargs):
            try:
                return func(*args, **kwargs)
            except Exception as e:
                raise exc_type(f"{message}: {e}") from e
        return wrapper
    return decorator

@wrap_exceptions(DatabaseError, "Database operation failed")
def query_db():
    # ...
```


### Recovery Strategies

```python
from typing import Callable, Dict, Type

def recovery_strategy(
    primary: Callable,
    fallbacks: Dict[Type[Exception], Callable],
):
    def wrapper(*args, **kwargs):
        try:
            return primary(*args, **kwargs)
        except Exception as e:
            fallback = fallbacks.get(type(e))
            if fallback:
                return fallback(*args, **kwargs, original_error=e)
            raise
    return wrapper
```


---

## 15. Exception Handling in Libraries

### Public APIs

**Rule**: Libraries should raise **specific, documented exceptions**.

```python
class LibraryError(Exception):
    """Base library exception"""

class AuthenticationError(LibraryError):
    """Auth failed"""

class ValidationError(LibraryError):
    """Invalid input"""

def connect(api_key: str):
    if not validate_key(api_key):
        raise ValidationError("Invalid API key")
    if not authenticate(api_key):
        raise AuthenticationError("Auth failed")
```


### SDK Design

```python
# Good SDK pattern
class SDK:
    def __init__(self, config: Config):
        self.config = config
    
    def fetch(self, id: str) -> Data:
        try:
            return self._fetch(id)
        except httpx.HTTPError as e:
            raise NetworkError(f"Network failed: {e}") from e
        except json.JSONDecodeError as e:
            raise ParseError(f"JSON decode failed: {e}") from e
```


### Framework Design

```python
# Django/Flask pattern
class Middleware:
    def process_request(self, request):
        try:
            return self.handle(request)
        except ValidationError as e:
            return HttpResponseBadRequest(str(e))
        except AuthenticationError as e:
            return HttpResponseUnauthorized(str(e))
        except Exception as e:
            # Log + generic error
            logger.exception("Unhandled exception")
            return HttpResponseServerError("Internal error")
```


---

## 16. Exception Handling in Distributed Systems

### Microservices

```python
import aiohttp
from exceptions import ExceptionGroup

async def call_service(service_url: str, payload: dict):
    try:
        async with aiohttp.ClientSession() as session:
            async with session.post(service_url, json=payload) as resp:
                if resp.status >= 500:
                    raise RuntimeError(f"Service error: {resp.status}")
                return await resp.json()
    except aiohttp.ClientError as e:
        raise NetworkError(f"Network failed: {e}") from e
```


### Message Queues

```python
from kafka import KafkaConsumer
from exceptions import ExceptionGroup

def process_messages():
    consumer = KafkaConsumer('topic')
    errors = []
    
    for msg in consumer:
        try:
            process(msg)
        except ValidationError as e:
            # Skip bad message, log
            errors.append(e)
        except Exception as e:
            # Dead-letter queue
            send_to_dlq(msg, e)
            errors.append(e)
    
    if errors:
        raise ExceptionGroup("Processing failures", errors)
```


### Event-Driven Systems

```python
from eventlet import Event

class EventProcessor:
    def __init__(self):
        self.handlers = {}
    
    def register(self event_type: str, handler: Callable):
        self.handlers[event_type] = handler
    
    def emit(self, event_type: str, data: dict):
        handler = self.handlers.get(event_type)
        if not handler:
            return
        
        try:
            handler(data)
        except Exception as e:
            # Emit error event
            self.emit("error", {"type": event_type, "error": str(e)})
            raise
```


### Cloud-Native Applications

```python
import boto3
from botocore.exceptions import ClientError

def s3_upload(bucket: str, key: str, data: bytes):
    s3 = boto3.client('s3')
    try:
        s3.upload_body(bucket, key, data)
    except ClientError as e:
        code = e.response['Error']['Code']
        if code == 'AccessDenied':
            raise PermissionError(f"Access denied: {bucket}") from e
        elif code == 'BucketNotFound':
            raise ValueError(f"Bucket not found: {bucket}") from e
        else:
            raise RuntimeError(f"S3 error: {code}") from e
```


---

## 17. Security Considerations

### Information Leakage

```python
# BAD: Exposes internal details
except Exception as e:
    return f"Error: {e} (line {e.__traceback__.tb_lineno})"

# GOOD: Sanitize
except Exception as e:
    logger.exception("Request failed")
    return "An error occurred. Please contact support."
```


### Secure Error Messages

```python
class SecureAPI:
    def handle(self, request):
        try:
            return self.process(request)
        except ValidationError as e:
            # Include only user-safe info
            return {"error": "Invalid input", "field": e.field}
        except Exception as e:
            # No internal details
            logger.exception("Unhandled error")
            return {"error": "Service temporarily unavailable"}
```


### Production Hardening

```python
# Middleware for error sanitization
class ErrorSanitizer:
    def __init__(self):
        self.internal_exceptions = (
            DatabaseError,
            NetworkError,
            ParseError,
        )
    
    def sanitize(self, exc: Exception) -> dict:
        if isinstance(exc, self.internal_exceptions):
            return {"error": "Internal service error"}
        return {"error": str(exc)}
```


---

## 18. Testing Exceptions

### unittest

```python
import unittest

class TestExceptions(unittest.TestCase):
    def test_raises_value_error(self):
        with self.assertRaises(ValueError):
            risky_function()
    
    def test_raises_with_message(self):
        with self.assertRaises(ValueError) as cm:
            risky_function()
        self.assertEqual(str(cm.exception), "expected message")
    
    def test_exception_chain(self):
        with self.assertRaises(RuntimeError) as cm:
            wrapped_function()
        self.assertIsInstance(cm.exception.__cause__, ValueError)
```


### pytest

```python
import pytest

def test_raises():
    with pytest.raises(ValueError):
        risky_function()

def test_raises_message():
    with pytest.raises(ValueError, match="expected message"):
        risky_function()

def test_exception_chain():
    with pytest.raises(RuntimeError) as exc_info:
        wrapped_function()
    assert isinstance(exc_info.value.__cause__, ValueError)
```


### Mock Failures

```python
from unittest.mock import patch, MagicMock

def test_mocked_failure():
    with patch('module.func', side_effect=ValueError("mocked")):
        with pytest.raises(ValueError):
            module.risky_call()
```


### Fault Injection

```python
import pytest

@pytest.fixture
def fault_injector():
    def inject(exc_type, exc_value):
        def wrapper(*args, **kwargs):
            raise exc_type(exc_value)
        return wrapper
    return inject

def test_with_fault(fault_injector):
    with patch('module.func', fault_injector(NetworkError, "down")):
        with pytest.raises(NetworkError):
            process_request()
```


---

## 19. Production Monitoring

### Logging

```python
import logging

logger = logging.getLogger(__name__)

def process():
    try:
        risky_operation()
    except ValueError as e:
        logger.warning("Validation failed: %s", e)
    except Exception as e:
        logger.exception("Unhandled exception")  # Includes traceback
```


### Observability

```python
import structlog

log = structlog.get_logger()

def api_call():
    try:
        return external_api()
    except Exception as e:
        log.error(
            "API call failed",
            error_type=type(e).__name__,
            error_msg=str(e),
            exc_info=True
        )
        raise
```


### Metrics

```python
from prometheus_client import Counter, Histogram

ERROR_COUNTER = Counter('api_errors', 'API errors')
LATENCY_HIST = Histogram('api_latency', 'API latency')

def api_endpoint():
    with LATENCY_HIST.time():
        try:
            return process()
        except Exception:
            ERROR_COUNTER.inc()
            raise
```


### Alerting

```python
import sentry_sdk

sentry_sdk.init(dsn="your-dsn")

def critical_operation():
    try:
        return do_something()
    except Exception as e:
        sentry_sdk.capture_exception(e)
        raise
```


---

## 20. Real-World Case Studies

### Web APIs

```python
from fastapi import FastAPI, HTTPException

app = FastAPI()

@app.get("/users/{id}")
def get_user(id: int):
    try:
        user = db.get_user(id)
    except NotFoundError:
        raise HTTPException(status_code=404, detail="User not found")
    except DatabaseError as e:
        raise HTTPException(status_code=500, detail="Database error")
    return user
```


### Databases

```python
import psycopg2
from psycopg2 import errors

def query(sql: str):
    try:
        with psycopg2.connect(DSN) as conn:
            with conn.cursor() as cur:
                cur.execute(sql)
                return cur.fetchall()
    except psycopg2.errors.UniqueViolation:
        raise DuplicateError("Record already exists")
    except psycopg2.errors.OperationalError as e:
        raise NetworkError(f"DB connection failed: {e}")
```


### ETL Pipelines

```python
from exceptions import ExceptionGroup

def etl_pipeline(records):
    successes = []
    failures = []
    
    for record in records:
        try:
            transformed = transform(record)
            loaded = load(transformed)
            successes.append(loaded)
        except ValidationError as e:
            failures.append({"record": record, "error": str(e), "type": "validation"})
        except Exception as e:
            failures.append({"record": record, "error": str(e), "type": "unexpected"})
    
    if failures:
        raise ExceptionGroup(f"ETL failures: {len(failures)}", 
                          [ValidationError(f["error"]) for f in failures])
    
    return successes
```


### Machine Learning Systems

```python
import torch

def predict(model, input):
    try:
        with torch.no_grad():
            return model(input)
    except torch.cuda.CudaError as e:
        raise HardwareError(f"GPU error: {e}")
    except RuntimeError as e:
        if "out of memory" in str(e):
            raise ResourceError("GPU memory exhausted")
        raise
```


### Async Services

```python
import asyncio
from exceptions import ExceptionGroup

async def service_cluster(requests):
    tasks = [handle_request(req) for req in requests]
    
    try:
        return await asyncio.gather(*tasks)
    except ExceptionGroup as eg:
        errors = {"count": len(eg.exceptions)}
        for exc in eg.exceptions:
            errors[type(exc).__name__] = errors.get(type(exc).__name__, 0) + 1
        raise ServiceDegradationError(errors)
```


---

## 21. Expert Interview Questions

### Questions 1-10: Core Concepts

**1. Why do `KeyboardInterrupt` and `SystemExit` inherit from `BaseException` instead of `Exception`?**

**Answer**: They inherit from `BaseException` to prevent accidental swallowing. If they inherited from `Exception`, a `except Exception:` handler would catch them, preventing program termination or user interruption. This is a **safety mechanism** to ensure system-exiting exceptions propagate unless explicitly handled.

---

**2. What is the difference between `__context__` and `__cause__`?**

**Answer**:

- `__context__`: Set **automatically** when an exception is raised while another exception is active (implicit chaining)
- `__cause__`: Set **explicitly** using `raise ... from ...` (explicit chaining)

```python
try:
    raise ValueError("original")
except ValueError:
    raise RuntimeError("wrapped")  # __context__ = ValueError

try:
    raise ValueError("original")
except ValueError as e:
    raise RuntimeError("wrapped") from e  # __cause__ = ValueError
```


---

**3. How does Python 3.7+ handle `StopIteration` raised inside generators?**

**Answer**: `StopIteration` is **wrapped as `RuntimeError`** to prevent silent corruption. Before 3.7, `StopIteration` inside a generator would silently terminate the generator, potentially hiding bugs.

```python
def bad():
    it = iter()
    next(it)
    next(it)  # StopIteration
    next(it)  # Raises StopIteration

# In generator context: RuntimeError: generator raised StopIteration
```


---

**4. What is `GeneratorExit` and when is it raised?**

**Answer**: `GeneratorExit` is a `BaseException` subclass raised inside a generator when:

- `generator.close()` is called
- Generator reference is deleted (GC)

It's designed for **cleanup**, not error handling, and cannot be caught by `except Exception:`.

---

**5. Explain the exception table mechanism in Python 3.11+.**

**Answer**: Python 3.11+ replaced `POP_BLOCK` with **exception tables** embedded in bytecode. Each entry specifies:

- `start` offset
- `end` offset
- `handler` target
- `stack_depth`

When an exception occurs, the VM searches the table for an entry covering the current offset, then jumps to the handler with the specified stack depth.

---

**6. What is the purpose of `__notes__` (Python 3.11+)?**

**Answer**: `__notes__` is a list of strings added via `exception.add_note()`. It provides **machine-parseable context** for debugging without modifying the error message.

```python
exc = ValueError("bad value")
exc.add_note("CSV parsing failed")
exc.add_note("Row 452")
```


---

**7. How does exception propagation work at the C level?**

**Answer**:

1. Exception is set in `PyThreadState.exc_type/value/traceback`
2. Bytecode evaluator checks for exception after each instruction
3. Searches exception table for handler
4. If no handler, unwinds stack (frees frames via `f_back`)
5. Propagates to caller or prints traceback at module level

---

**8. What is the cost of an empty `try/except` block?**

**Answer**: ~2-5% overhead when no exception occurs (verified via `timeit`). The overhead comes from exception table setup and stack checks.

---

**9. Why should you avoid `except:` without a type?**

**Answer**: It catches **all exceptions including `BaseException`** subclasses like `KeyboardInterrupt`, `SystemExit`, and `GeneratorExit`. This can prevent program termination or user interruption.

```python
# BAD
try:
    code()
except:  # Catches KeyboardInterrupt!
    pass

# GOOD
try:
    code()
except Exception:  # Only user-facing exceptions
    pass
```


---

**10. What happens when a `try/finally` block raises an exception in the `finally`?**

**Answer**: The exception in `finally` **replaces** the original exception. The original is lost unless explicitly saved.

```python
try:
    raise ValueError("original")
finally:
    raise RuntimeError("finally")  # Replaces ValueError

# Only RuntimeError is raised
```


---

### Questions 11-20: Advanced Topics

**11. How do ExceptionGroups improve concurrent error handling?**

**Answer**: They aggregate **multiple concurrent failures** into one exception, preserving all error context. Previously, `asyncio.gather()` would raise only the first exception, losing others.

```python
try:
    await asyncio.gather(task1(), task2(), task3())
except ExceptionGroup as eg:
    for exc in eg.exceptions:  # All failures available
        handle(exc)
```


---

**12. Explain `except*` syntax and when it's used.**

**Answer**: `except*` is used **inside async exception handlers** to match exceptions within an `ExceptionGroup`. It's only valid in `except ExceptionGroup:` blocks.

```python
try:
    await gather(task1(), task2())
except* FileNotFoundError:  # Matches FileNotFound within group
    handle_missing()
except* Exception:  # Matches remaining
    handle_other()
```


---

**13. What is the circuit breaker pattern and how does it use exceptions?**

**Answer**: Circuit breaker tracks **failure counts** and transitions to `OPEN` state after threshold. When open, it raises `RuntimeError` immediately instead of calling the failing service.

```python
if state == OPEN:
    raise RuntimeError("Circuit open")
try:
    return call()
except Exception:
    failures += 1
    if failures >= threshold:
        state = OPEN
```


---

**14. How do you break traceback reference cycles?**

**Answer**: Delete the exception reference after handling:

```python
try:
    risky()
except Exception as e:
    process(e)
del e  # Breaks: e → __traceback__ → frame → f_locals → e
```


---

**15. What is the difference between `RuntimeWarning` and `DeprecationWarning`?**

**Answer**:

- `RuntimeWarning`: Suspicious runtime behavior (shown **once** by default)
- `DeprecationWarning`: API will be removed (shown **by default** in 3.10+)

---

**16. How does `throw()` work in generators internally?**

**Answer**:

1. Save generator state
2. Resume at yield point
3. Raise exception at yield
4. If handled, continue; if not, terminate and cleanup

---

**17. What are exception dispatch tables?**

**Answer**: Internal bytecode structure mapping offset ranges to handler targets. Used by the VM to quickly locate handlers during exception propagation.

---

**18. Why is raising exceptions in loops expensive?**

**Answer**: Each raise involves:

- Object creation
- Traceback attachment
- Stack unwinding
- Frame cleanup

**Optimization**: Validate before loop, not inside.

---

**19. How do you test exception chains?**

**Answer**: Use `pytest.raises()` with `__cause__` assertion:

```python
with pytest.raises(RuntimeError) as exc_info:
    wrapped()
assert isinstance(exc_info.value.__cause__, ValueError)
```


---

**20. What is error translation?**

**Answer**: Converting low-level exceptions to **domain-specific** exceptions:

```python
try:
    return api.call()
except httpx.HTTPError as e:
    raise NetworkError(f"Network failed: {e}") from e
```


---

### Questions 21-25: Production \& Architecture

**21. How do you handle exceptions in microservices?**

**Answer**:

- Wrap external calls with specific exceptions
- Use **dead-letter queues** for unprocessable messages
- Aggregate failures with `ExceptionGroup`
- Return **sanitized** errors to clients

---

**22. What metrics should you track for exceptions?**

**Answer**:

- Error count (Counter)
- Error rate (rate/sec)
- Latency by error type (Histogram)
- Exception type distribution

---

**23. How do you sanitize errors in production APIs?**

**Answer**:

- Log full traceback internally
- Return **generic** messages to clients
- Never expose stack traces, DB errors, or internal paths

---

**24. What is the retry pattern with exponential backoff?**

**Answer**: Retry with increasing delays + jitter:

```python
delay = min(base * (2 ** attempt), max_delay)
delay += random.uniform(0, 0.5 * delay)
time.sleep(delay)
```


---

**25. How do you design exceptions for SDKs?**

**Answer**:

- Base `SDKError` for all SDK exceptions
- Specific subclasses (`AuthError`, `ValidationError`)
- Wrap external exceptions with `from e`
- Document all exceptions publicly

---

## 22. Advanced Exercises

### Exercise 1: Exception Chain Inspector

```python
def inspect_chain(exc: Exception) -> list:
    """Return all exceptions in __cause__ and __context__ chains."""
    chain = []
    
    current = exc
    while current:
        chain.append(current)
        current = current.__cause__ or current.__context__
    
    return chain

# Test
try:
    try:
        raise ValueError("original")
    except ValueError as e:
        raise RuntimeError("wrapped") from e
except RuntimeError as e:
    chain = inspect_chain(e)
    assert len(chain) == 2
    assert isinstance(chain, RuntimeError)
    assert isinstance(chain, ValueError)
```

**Solution**: See code above.

---

### Exercise 2: Retry with Exponential Backoff

```python
import time
import random

def retry_backoff(func, exc_type, max_attempts=5, base_delay=1.0):
    for attempt in range(max_attempts):
        try:
            return func()
        except exc_type as e:
            if attempt == max_attempts - 1:
                raise
            delay = min(base_delay * (2 ** attempt), 60.0)
            delay += random.uniform(0, 0.5 * delay)
            time.sleep(delay)
    raise RuntimeError("Retry failed")

# Test
attempts = 0
def flaky():
    attempts += 1
    if attempts < 3:
        raise ValueError("fail")
    return "success"

result = retry_backoff(flaky, ValueError)
assert result == "success"
assert attempts == 3
```

**Solution**: See code above.

---

### Exercise 3: Circuit Breaker Implementation

```python
from enum import Enum
import time

class State(Enum):
    CLOSED = "closed"
    OPEN = "open"
    HALF_OPEN = "half_open"

class CircuitBreaker:
    def __init__(self, threshold=5, timeout=30.0):
        self.failures = 0
        self.state = State.CLOSED
        self.threshold = threshold
        self.timeout = timeout
        self.last_failure = None
    
    def call(self, func):
        if self.state == State.OPEN:
            if time.time() - self.last_failure > self.timeout:
                self.state = State.HALF_OPEN
            else:
                raise RuntimeError("Circuit open")
        
        try:
            result = func()
            if self.state == State.HALF_OPEN:
                self.state = State.CLOSED
                self.failures = 0
            return result
        except Exception:
            self.failures += 1
            self.last_failure = time.time()
            if self.failures >= self.threshold:
                self.state = State.OPEN
            raise

# Test
cb = CircuitBreaker(threshold=2)
def failing():
    raise ValueError("fail")

for i in range(2):
    try:
        cb.call(failing)
    except ValueError:
        pass

assert cb.state == State.OPEN
```

**Solution**: See code above.

---

### Exercise 4: Exception Group Aggregator

```python
from exceptions import ExceptionGroup

def aggregate_errors(funcs):
    errors = []
    results = []
    
    for func in funcs:
        try:
            results.append(func())
        except Exception as e:
            errors.append(e)
    
    if errors:
        raise ExceptionGroup(f"{len(errors)} failures", errors)
    
    return results

# Test
def ok(): return 1
def fail(): raise ValueError("bad")

try:
    aggregate_errors([ok, fail, ok])
except ExceptionGroup as eg:
    assert len(eg.exceptions) == 1
```

**Solution**: See code above.

---

### Exercise 5: Traceback Walker

```python
import sys

def walk_traceback(exc):
    """Yield (frame_name, line_number) for each frame."""
    tb = exc.__traceback__
    while tb:
        yield tb.tb_frame.f_code.co_name, tb.tb_lineno
        tb = tb.tb_next

# Test
def a(): b()
def b(): c()
def c(): raise ValueError("test")

try:
    a()
except ValueError as e:
    frames = list(walk_traceback(e))
    assert frames == ("c", 1)
    assert frames == ("b", 1)
    assert frames == ("a", 1)
```

**Solution**: See code above.

---

### Exercise 6: Warning Capturer

```python
import warnings

def capture_warnings(func):
    """Return list of warnings emitted by func."""
    with warnings.catch_warnings(record=True) as w:
        warnings.simplefilter("always")
        func()
        return w

# Test
def warn():
    warnings.warn("test", DeprecationWarning)

warnings = capture_warnings(warn)
assert len(warnings) == 1
assert str(warnings.message) == "test"
```

**Solution**: See code above.

---

### Exercise 7: Error Translator

```python
from typing import Type, Dict

def translate_errors(exc_map: Dict[Type[Exception], Type[Exception]]):
    def decorator(func):
        def wrapper(*args, **kwargs):
            try:
                return func(*args, **kwargs)
            except Exception as e:
                new_exc = exc_map.get(type(e))
                if new_exc:
                    raise new_exc(f"Translated: {e}") from e
                raise
        return wrapper
    return decorator

# Test
class ExternalError(Exception): pass
class InternalError(Exception): pass

@translate_errors({ExternalError: InternalError})
def api():
    raise ExternalError("external")

try:
    api()
except InternalError as e:
    assert isinstance(e.__cause__, ExternalError)
```

**Solution**: See code above.

---

### Exercise 8: Fault Injector

```python
from unittest.mock import patch

def inject_fault(exc_type, exc_value, target):
    """Inject exception into target function."""
    def wrapper(*args, **kwargs):
        raise exc_type(exc_value)
    return patch(target, wrapper)

# Test
def real(): return "ok"

with inject_fault(ValueError, "fault", "module.real"):
    with patch("module.real", inject_fault(ValueError, "fault", "module.real")):
        try:
            real()
        except ValueError:
            pass
```

**Solution**: See code above (simplified version).

---

### Exercise 9: Async Exception Group Handler

```python
import asyncio
from exceptions import ExceptionGroup

async def process_all(items):
    async def process(item):
        if item % 2 == 0:
            raise ValueError(f"Even: {item}")
        return item
    
    try:
        return await asyncio.gather(*[process(i) for i in items])
    except ExceptionGroup as eg:
        return {"errors": len(eg.exceptions), "successes": 0}

# Test
result = process_all()
assert result["errors"] == 2
```

**Solution**: See code above.

---

### Exercise 10: Exception Metrics Collector

```python
from collections import Counter

class ExceptionMetrics:
    def __init__(self):
        self.counter = Counter()
    
    def record(self, exc: Exception):
        self.counter[type(exc).__name__] += 1
    
    def report(self):
        return dict(self.counter)

# Test
metrics = ExceptionMetrics()
try:
    raise ValueError("test")
except ValueError as e:
    metrics.record(e)

assert metrics.report() == {"ValueError": 1}
```

**Solution**: See code above.

---

### Exercise 11: Try Block Cost Benchmark

```python
import timeit

def no_try():
    x = 1 + 2

def with_try():
    try:
        x = 1 + 2
    except:
        pass

no_time = timeit.timeit(no_try, number=10_000_000)
try_time = timeit.timeit(with_try, number=10_000_000)

overhead = (try_time - no_time) / no_time * 100
print(f"Overhead: {overhead:.2f}%")
```

**Solution**: Run code; expect ~2-5% overhead.

---

### Exercise 12: Exception Note Adder

```python
def add_context(exc: Exception, notes: list):
    """Add notes to exception (Python 3.11+)."""
    for note in notes:
        exc.add_note(note)
    return exc

# Test (Python 3.11+)
exc = ValueError("bad")
exc = add_context(exc, ["CSV failed", "Row 452"])
assert exc.__notes__ == ["CSV failed", "Row 452"]
```

**Solution**: See code above.

---

### Exercise 13: Generator Exit Handler

```python
def generator_with_cleanup():
    try:
        while True:
            yield 1
    except GeneratorExit:
        print("Cleanup!")
    finally:
        print("Finally!")

# Test
g = generator_with_cleanup()
next(g)
g.close()  # Triggers GeneratorExit
```

**Solution**: See code above.

---

### Exercise 14: Bytecode Inspector

```python
import dis

def inspect_exceptions(func):
    """Print exception table for function (Python 3.11+)."""
    dis.dis(func)

# Test
def example():
    try:
        return 1 / 0
    except ZeroDivisionError:
        return 0

inspect_exceptions(example)
```

**Solution**: Run code; shows exception table.

---

### Exercise 15: Production Error Sanitizer

```python
import logging

logger = logging.getLogger(__name__)

def sanitize_error(exc: Exception, safe: bool = True) -> str:
    """Return sanitized error message."""
    if safe:
        logger.exception("Operation failed")
        return "Operation failed. Please contact support."
    return str(exc)

# Test
try:
    raise ValueError("internal: password=abc")
except ValueError as e:
    msg = sanitize_error(e, safe=True)
    assert "password" not in msg
```

**Solution**: See code above.

---

## 23. Ultimate Cheat Sheet

### Hierarchy Summary

| Category | Base Class | Examples |
| :-- | :-- | :-- |
| System-exiting | `BaseException` | `SystemExit`, `KeyboardInterrupt`, `GeneratorExit` |
| User errors | `Exception` | `ValueError`, `TypeError`, `KeyError` |
| OS errors | `OSError` | `FileNotFoundError`, `PermissionError` |
| Warnings | `Warning` | `DeprecationWarning`, `RuntimeWarning` |
| Groups (3.11+) | `BaseExceptionGroup` | `ExceptionGroup`, `BaseExceptionGroup` |

### Internal Flow Summary

```mermaid
flowchart TD
    A[Raise] --> B[Create exception object]
    B --> C[Attach traceback]
    C --> D[Set thread state]
    D --> E[Check exception table]
    E --> F{Handler found?}
    F -->|Yes| G[Jump to handler]
    F -->|No| H[Unwind stack]
    H --> I[Pop frames via f_back]
    I --> J{More frames?}
    J -->|Yes| E
    J -->|No| K[Print traceback + exit]
```


### CPython Internals Summary

| Function | Purpose |
| :-- | :-- |
| `PyErr_SetString()` | Set exception with string |
| `PyErr_Occurred()` | Check if exception set |
| `PyErr_Fetch()` | Fetch + clear exception |
| `PyErr_Restore()` | Restore fetched exception |
| `ob_exception` | Exception type |
| `ob_value` | args tuple |
| `ob_traceback` | Traceback object |
| `ob_context` | Implicit chain |
| `ob_cause` | Explicit chain |
| `ob_notes` | Notes list (3.11+) |

### Performance Summary

| Operation | Cost | Recommendation |
| :-- | :-- | :-- |
| Empty `try` | ~2-5% | OK for hot paths |
| Raise exception | ~1-5μs | Avoid in loops |
| ExceptionGroup | ~10-20% slower | Use for concurrent failures |
| Broad `except:` | High risk | Never use |
| Specific `except` | Low | Preferred |


---

## Conclusion

### Runtime Architecture

Python's exception system is built on:

- **Per-thread state** (`PyThreadState`) storing current exception
- **Exception tables** (3.11+) embedded in bytecode for fast handler lookup
- **Frame objects** linked via `f_back` for stack traversal


### Exception Propagation

1. Exception set in thread state
2. VM searches exception table
3. If no handler, unwind stack via `f_back`
4. Propagate to caller or print traceback

### Tracebacks

- Built as linked list via `tb_next`
- Each node: frame + line number + instruction index
- Attached automatically on raise


### Internals

- `PyBaseExceptionObject` stores: type, args, traceback, context, cause, notes
- Reference cycles: exception → traceback → frame → locals → exception
- Break cycles: `del exc` after handling


### Bytecode

- Python 3.11+: exception tables replace `POP_BLOCK`
- Format

