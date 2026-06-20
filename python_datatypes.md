# Python Data Types Master Guide
### The Most Complete Reference for int, float, complex, Decimal, Fraction, bool, and NoneType

> **Author:** Senior Python Language Architect | CPython Internals Expert | 25+ Years Experience  
> **Level:** Beginner → Intermediate → Advanced → CPython Internals → Interview → Production  
> **Target Length:** 20,000+ words  

---

# Table of Contents

- [Chapter 1: Introduction to Python Data Types](#chapter-1-introduction-to-python-data-types)
- [Chapter 2: Python Object Model](#chapter-2-python-object-model)
- [Chapter 3: Numeric Types Overview](#chapter-3-numeric-types-overview)
- [Chapter 4: Integer (int)](#chapter-4-integer-int)
- [Chapter 5: Floating Point Numbers (float)](#chapter-5-floating-point-numbers-float)
- [Chapter 6: Complex Numbers (complex)](#chapter-6-complex-numbers-complex)
- [Chapter 7: Decimal](#chapter-7-decimal)
- [Chapter 8: Fraction](#chapter-8-fraction)
- [Chapter 9: Boolean Type (bool)](#chapter-9-boolean-type-bool)
- [Chapter 10: NoneType](#chapter-10-nonetype)
- [Chapter 11: Type Conversion](#chapter-11-type-conversion)
- [Chapter 12: Type Checking](#chapter-12-type-checking)
- [Chapter 13: Comparisons Between Types](#chapter-13-comparisons-between-types)
- [Chapter 14: Memory Management](#chapter-14-memory-management)
- [Chapter 15: CPython Internals](#chapter-15-cpython-internals)
- [Chapter 16: Common Bugs and Pitfalls](#chapter-16-common-bugs-and-pitfalls)
- [Chapter 17: Performance Analysis](#chapter-17-performance-analysis)
- [Chapter 18: Interview Questions](#chapter-18-interview-questions)
- [Chapter 19: Coding Exercises](#chapter-19-coding-exercises)
- [Chapter 20: Cheat Sheet](#chapter-20-cheat-sheet)
- [Chapter 21: Best Practices](#chapter-21-best-practices)
- [Chapter 22: Summary](#chapter-22-summary)

---

# Chapter 1: Introduction to Python Data Types

## 1.1 What is a Data Type?

A **data type** is a classification that tells the interpreter or compiler how the programmer intends to use data. It defines:

1. **The set of values** the data can hold (domain)
2. **The set of operations** that can be performed on the data
3. **The memory representation** of the data
4. **The behavior** under various operations

In Python, every piece of data — every number, every string, every object — belongs to a data type. The type determines what you can do with the value, how it is stored in memory, and how it interacts with other values.

```python
x = 42          # int  — whole number
y = 3.14        # float — decimal number
z = 2 + 3j      # complex — has real and imaginary parts
flag = True     # bool  — True or False
nothing = None  # NoneType — absence of value
```

## 1.2 Why Data Types Exist

Data types exist for several critical reasons:

### 1.2.1 Memory Efficiency
Different types of data require different amounts of memory. A boolean needs only 1 bit logically (though Python uses more), while a 64-bit float requires 8 bytes. By categorizing data, Python can allocate memory appropriately.

### 1.2.2 Operation Safety
Data types prevent nonsensical operations. You cannot multiply `None` by `3` or compute the sine of a string. Type systems enforce logical correctness.

### 1.2.3 Performance Optimization
Knowing the type ahead of time (or at runtime) allows the interpreter to choose optimal execution paths. Integer addition in CPython is much faster than arbitrary-precision Decimal addition.

### 1.2.4 Semantic Clarity
Types communicate intent. `Decimal('19.99')` clearly signals financial precision, while `19.99` as a float signals that approximate representation is acceptable.

### 1.2.5 API Contract Enforcement
Functions and methods define expected types. A function that accepts `float` behaves differently from one that accepts `Decimal`. Types form contracts between code components.

## 1.3 Dynamic Typing

Python is a **dynamically typed** language. This means:

- Variables do **not** have fixed types
- Types are associated with **values** (objects), not variables
- Type checking happens at **runtime**, not compile time
- The same variable can reference objects of different types at different times

```python
# Dynamic typing in action
x = 10          # x references an int object
print(type(x))  # <class 'int'>

x = 3.14        # x now references a float object
print(type(x))  # <class 'float'>

x = "hello"     # x now references a str object
print(type(x))  # <class 'str'>

x = [1, 2, 3]   # x now references a list object
print(type(x))  # <class 'list'>
```

**What's happening internally:**
- `x` is just a name (label) in a namespace (a dictionary)
- When you write `x = 10`, Python creates an `int` object with value `10` in memory and binds the name `x` to it
- When you write `x = 3.14`, Python creates a new `float` object and rebinds `x` to it
- The old `int` object may be garbage collected if no other references exist

### Dynamic Typing vs Static Typing

| Feature | Dynamic (Python) | Static (Java, C++) |
|---------|-----------------|-------------------|
| Type declaration | Not required | Required |
| Type checking | Runtime | Compile time |
| Flexibility | High | Lower |
| Error detection | Later (runtime) | Earlier (compile time) |
| Performance | Slower (runtime checks) | Faster (known at compile) |
| Verbosity | Less code | More code |

## 1.4 Strong Typing

Python is both dynamically typed **and** strongly typed. **Strong typing** means:

- Python does **not** automatically coerce (convert) incompatible types
- Implicit type conversions are minimal and strict
- Operations between incompatible types raise `TypeError`

```python
# Strong typing — Python does NOT silently convert
result = "5" + 3    # TypeError: can only concatenate str (not "int") to str
result = "5" * 3    # OK! — str repetition: "555"
result = 5 + 3.0    # OK! — numeric promotion: 8.0 (float)
result = True + 1   # OK! — bool is subclass of int: 2
```

**Contrast with JavaScript (weakly typed):**
```javascript
// JavaScript silently coerces
"5" + 3     // "53"  (number converted to string)
"5" - 3     // 2     (string converted to number)
true + "1"  // "true1"
```

Python's strong typing catches bugs early and makes code behavior predictable.

## 1.5 Duck Typing

**Duck typing** is a concept in Python's type system: "If it walks like a duck and quacks like a duck, it's a duck." In practice:

- Python doesn't check the type of an object
- Python checks whether the object has the required **attributes and methods**
- Any object that supports the needed interface can be used

```python
def double(x):
    return x + x    # Works for int, float, str, list — anything supporting +

print(double(5))        # 10      (int)
print(double(3.14))     # 6.28   (float)
print(double("hello"))  # "hellohello" (str)
print(double([1, 2]))   # [1, 2, 1, 2] (list)
```

Duck typing enables Python's polymorphism without inheritance hierarchies.

## 1.6 Runtime Type Checking

In Python, type information is always available at runtime via built-in functions:

```python
x = 42

# Check the type
print(type(x))          # <class 'int'>
print(type(x).__name__) # 'int'

# Check if it's a specific type
print(isinstance(x, int))    # True
print(isinstance(x, float))  # False
print(isinstance(x, (int, float)))  # True — checks multiple types

# Check type hierarchy
print(issubclass(bool, int))  # True
print(issubclass(int, object))  # True
```

## 1.7 Type Safety

Python achieves type safety through a combination of:

1. **Strong typing** — no silent coercions
2. **Runtime type checks** — operations validate their operands
3. **Type hints (Python 3.5+)** — optional annotations for tools
4. **Type checkers** — mypy, pyright, pytype validate at development time

```python
# Type hints (annotations) — not enforced at runtime, but used by tools
def add_numbers(x: int, y: int) -> int:
    return x + y

# mypy would flag this as an error:
result = add_numbers("hello", 5)  # Type error in static analysis
```

## 1.8 Memory Representation

Every Python object in memory has a common header structure (CPython):

```
+------------------+
| ob_refcnt        |  8 bytes — reference count for GC
+------------------+
| ob_type          |  8 bytes — pointer to type object
+------------------+
| [type-specific]  |  variable — the actual value
+------------------+
```

This base structure (`PyObject`) is shared by ALL Python objects regardless of type. Specific types extend this with additional fields.

```python
import sys

# Memory sizes of basic types
print(sys.getsizeof(0))        # 24 bytes (small int)
print(sys.getsizeof(3.14))     # 24 bytes (float)
print(sys.getsizeof(True))     # 28 bytes (bool)
print(sys.getsizeof(None))     # 16 bytes (NoneType)
print(sys.getsizeof(10**100))  # 72 bytes (big int)
```

---

# Chapter 2: Python Object Model

## 2.1 Everything is an Object

This is Python's most fundamental principle: **everything is an object**. This includes:

- Numbers: `42`, `3.14`, `True`, `None`
- Strings: `"hello"`
- Functions: `def foo(): ...`
- Classes: `class MyClass: ...`
- Modules: `import os`
- Types themselves: `int`, `float`, `type`

```python
# Even the type 'int' is an object of type 'type'
print(type(42))       # <class 'int'>
print(type(int))      # <class 'type'>
print(type(type))     # <class 'type'> — type is its own type!

# Functions are objects
def greet():
    pass

print(type(greet))    # <class 'function'>
greet.custom = "yes"  # You can add attributes to functions!
print(greet.custom)   # "yes"
```

Every object has three fundamental properties:

| Property | Function | Description |
|----------|----------|-------------|
| Identity | `id(obj)` | Unique integer — memory address in CPython |
| Type | `type(obj)` | The class/type of the object |
| Value | varies | The data the object holds |

## 2.2 Variables as References

In Python, **variables are not boxes containing values**. They are **labels (references) pointing to objects** in memory.

```
Assignment: x = 42

 Variable    Memory
+---------+    +--------+
|    x    |--->| ob_ref |  int object
|         |    | ob_type|  value: 42
+---------+    |   42   |
               +--------+
```

When you do `y = x`, you're not copying the value — you're creating another reference to the same object:

```python
x = [1, 2, 3]
y = x           # y references the SAME list object

y.append(4)
print(x)        # [1, 2, 3, 4] — x sees the change!
print(y is x)   # True — same object
```

For immutable types (int, float, bool, None, str, tuple), this doesn't cause issues because the objects cannot be changed in place.

## 2.3 Object Creation

When you write `x = 42`, Python:

1. Evaluates the literal `42`
2. Checks the small integer cache (for integers -5 to 256)
3. Either returns the cached object or creates a new `PyLongObject`
4. Binds the name `x` to this object in the current namespace

```python
# Object creation flow
import dis

def create_int():
    x = 42
    return x

dis.dis(create_int)
# LOAD_CONST 42      — push 42 onto the evaluation stack
# STORE_FAST x       — pop and bind to local name 'x'
# LOAD_FAST x        — push x's value onto stack
# RETURN_VALUE       — return top of stack
```

## 2.4 Object Identity — `id()`

`id(obj)` returns a unique integer identifying the object. In CPython, this is the memory address.

```python
x = 42
y = 42
z = x

print(id(x))        # e.g., 140234567890
print(id(y))        # SAME as id(x) — small int cache!
print(id(z))        # SAME — z references same object

a = 1000
b = 1000
print(id(a) == id(b))  # False (usually) — large ints not cached
```

**The `is` operator checks identity:**
```python
x = None
y = None
print(x is y)       # True — None is a singleton
print(x is None)    # True — idiomatic None check

x = [1, 2]
y = [1, 2]
print(x == y)       # True — same value
print(x is y)       # False — different objects
```

## 2.5 Object Type — `type()`

`type(obj)` returns the type (class) of an object. It can also be used to create new types dynamically.

```python
# Basic type checking
print(type(42))         # <class 'int'>
print(type(3.14))       # <class 'float'>
print(type(True))       # <class 'bool'>
print(type(None))       # <class 'NoneType'>
print(type("hello"))    # <class 'str'>

# type() as a metaclass call — creates a new class
MyClass = type('MyClass', (object,), {'x': 10})
obj = MyClass()
print(obj.x)            # 10

# Type checking with type() — exact match only
x = True
print(type(x) is bool)  # True
print(type(x) is int)   # False! (even though bool IS a subclass of int)
```

## 2.6 Object Value

The **value** of an object is the data it holds. For numeric types:

```python
x = 42
# The value is the integer 42
# Accessed simply by using x in expressions
print(x + 1)    # 43
print(x * 2)    # 84

# For composite types, value includes all contained data
lst = [1, 2, 3]
# Value is the sequence [1, 2, 3]
```

## 2.7 `isinstance()` — The Preferred Type Check

`isinstance(obj, classinfo)` checks if `obj` is an instance of `classinfo` or any of its subclasses. This is the **preferred** way to check types in production code.

```python
# isinstance vs type()
x = True

# With isinstance — respects inheritance
print(isinstance(x, bool))   # True
print(isinstance(x, int))    # True!  (bool subclasses int)
print(isinstance(x, object)) # True   (everything is an object)

# With type() — exact match only
print(type(x) is bool)       # True
print(type(x) is int)        # False

# isinstance with tuple — check multiple types at once
def process(x):
    if isinstance(x, (int, float)):
        return x * 2
    raise TypeError(f"Expected number, got {type(x).__name__}")

print(process(5))     # 10
print(process(3.14))  # 6.28
```

## 2.8 Memory Diagram — Object Model

```
Python Object in Memory (CPython):

     Stack Frame              Heap
   +------------+         +------------------+
   |  name: x   |-------> | ob_refcnt: 1     |  <- reference count
   +------------+         | ob_type: *int    |  <- pointer to int type
                          | ob_digit[0]: 42  |  <- the value
                          +------------------+

   +------------+         +------------------+
   |  name: y   |-------> | ob_refcnt: 1     |  (different object)
   +------------+         | ob_type: *int    |
                          | ob_digit[0]: 42  |
                          +------------------+

   (If x = y = 42 and 42 is cached:)
   +------------+
   |  name: x   |---+     +------------------+
   +------------+   +---> | ob_refcnt: 2     |  <- both point here
   |  name: y   |---+     | ob_type: *int    |
   +------------+         | ob_digit[0]: 42  |
                          +------------------+
```

---

# Chapter 3: Numeric Types Overview

## 3.1 Numeric Hierarchy

Python's numeric types form a well-defined hierarchy, both mathematically and in terms of class inheritance:

```
Mathematical Hierarchy:
    Complex Numbers (ℂ)
         |
    Real Numbers (ℝ)
         |
    Rational Numbers (ℚ)
         |
    Integers (ℤ)
         |
    Natural Numbers (ℕ)

Python Type Mapping:
    complex
       |
    float (approximates ℝ)
       |
    Fraction (exact ℚ)
       |
    int (exact ℤ)
       |
    bool (subtype of int: 0 and 1)
```

## 3.2 The `numbers` Module — Abstract Numeric Tower

Python's `numbers` module defines Abstract Base Classes (ABCs) forming the numeric tower:

```python
from numbers import Number, Complex, Real, Rational, Integral

# The abstract numeric tower
# Number
#   └── Complex
#         └── Real
#               └── Rational
#                     └── Integral

# Checking membership in the tower
print(isinstance(42, Integral))    # True
print(isinstance(42, Rational))    # True
print(isinstance(42, Real))        # True
print(isinstance(42, Complex))     # True
print(isinstance(42, Number))      # True

print(isinstance(3.14, Integral))  # False
print(isinstance(3.14, Real))      # True
print(isinstance(3.14, Complex))   # True

print(isinstance(2+3j, Integral))  # False
print(isinstance(2+3j, Real))      # False
print(isinstance(2+3j, Complex))   # True

from fractions import Fraction
f = Fraction(1, 3)
print(isinstance(f, Rational))     # True
print(isinstance(f, Integral))     # False
```

## 3.3 Type Summary Table

| Type | Module | Precision | Immutable | Hashable | Use Case |
|------|--------|-----------|-----------|----------|----------|
| `int` | builtin | Unlimited | Yes | Yes | Counting, indexing, exact math |
| `float` | builtin | ~15-17 sig digits | Yes | Yes | Science, speed, approx math |
| `complex` | builtin | 2× float precision | Yes | Yes | Engineering, signal processing |
| `Decimal` | `decimal` | Configurable | Yes | Yes | Finance, exact decimal math |
| `Fraction` | `fractions` | Exact rational | Yes | Yes | Exact arithmetic, math proofs |
| `bool` | builtin | Binary (0/1) | Yes | Yes | Conditions, flags |

## 3.4 Numeric Coercion and Promotion

When mixing numeric types in operations, Python follows automatic promotion rules:

```
Promotion order (lowest to highest):
bool → int → float → complex

Rules:
- bool op int     → int
- int op float    → float
- float op complex → complex
- Any op Decimal  → Decimal (explicit conversion required)
- Any op Fraction → Fraction (explicit conversion required)
```

```python
# Automatic promotion examples
print(True + 1)       # 2        (bool → int)
print(1 + 1.0)        # 2.0      (int → float)
print(1.0 + (1+0j))   # (2+0j)   (float → complex)
print(type(True + 1)) # <class 'int'>
print(type(1 + 1.0))  # <class 'float'>

# Decimal requires explicit conversion
from decimal import Decimal
# Decimal('1.5') + 1.5  # TypeError!
print(Decimal('1.5') + Decimal('1.5'))  # Decimal('3.0')
```

---

# Chapter 4: Integer (int)

## 4.1 Definition

An **integer** (`int`) in Python represents a whole number — positive, negative, or zero — with no fractional component. Python integers have **unlimited precision**, meaning they can be arbitrarily large, bounded only by available memory. This distinguishes Python from most other languages where integers are fixed-width (e.g., 32-bit or 64-bit).

```python
# All valid Python integers
x = 0
x = -100
x = 999_999_999_999_999_999_999   # Underscore separators for readability
x = 2 ** 1000                      # 302 digits! — perfectly valid
```

## 4.2 Characteristics

### 4.2.1 Unlimited Precision (Arbitrary Precision Arithmetic)

```python
# Python handles arbitrarily large integers
factorial_100 = 1
for i in range(1, 101):
    factorial_100 *= i

print(factorial_100)
# 93326215443944152681699238856266700490715968264381621468592963895217599993229915608941463976156518286253697920827223758251185210916864000000000000000000000000
print(len(str(factorial_100)))  # 158 digits!

# Exact arithmetic — no overflow
print(2 ** 64)        # 18446744073709551616 (exact, unlike C's uint64)
print(2 ** 1000)      # 302-digit number
```

### 4.2.2 Signed Integers

Python integers are always signed — they can represent positive, negative, and zero values without separate unsigned types:

```python
positive = 42
negative = -42
zero = 0

print(-(-42))   # 42
print(abs(-42)) # 42
print(+42)      # 42
```

### 4.2.3 Immutability

Integers in Python are **immutable** — once created, the value cannot be changed:

```python
x = 42
print(id(x))   # e.g., 140234567

x += 1         # Creates a NEW int object (43), rebinds x
print(id(x))   # Different id! x now points to 43

# This means int objects are safe to share between references
a = b = 42
a += 1
print(a)  # 43
print(b)  # 42  — b still points to original 42
```

## 4.3 Integer Creation

### 4.3.1 Literal Forms

```python
# Decimal (base 10) — default
x = 42
x = -100
x = 1_000_000   # Python 3.6+ underscore separator

# Binary (base 2) — prefix 0b or 0B
b = 0b1010      # 10
b = 0B1111      # 15

# Octal (base 8) — prefix 0o or 0O
o = 0o17        # 15
o = 0O755       # 493

# Hexadecimal (base 16) — prefix 0x or 0X
h = 0xFF        # 255
h = 0xDEADBEEF  # 3735928559
h = 0x1A_2B     # underscore allowed here too

print(bin(255))  # '0b11111111'
print(oct(255))  # '0o377'
print(hex(255))  # '0xff'
```

### 4.3.2 The `int()` Constructor

```python
# From integer literal
x = int(42)        # 42

# From float — truncates toward zero
x = int(3.9)       # 3   (NOT round — truncate!)
x = int(-3.9)      # -3  (toward zero, not floor)

# From string — decimal by default
x = int("42")      # 42
x = int("-100")    # -100
x = int("  42  ")  # 42 — strips whitespace

# From string with base
x = int("ff", 16)   # 255
x = int("1010", 2)  # 10
x = int("17", 8)    # 15
x = int("42", 10)   # 42
x = int("z", 36)    # 35 — base 36 uses a-z

# From boolean
x = int(True)   # 1
x = int(False)  # 0

# From bytes
x = int.from_bytes(b'\x00\x10', byteorder='big')  # 16

# Errors
try:
    int("hello")      # ValueError
    int("3.14")       # ValueError — use float() first
    int(None)         # TypeError
except (ValueError, TypeError) as e:
    print(e)
```

## 4.4 Integer Operations

### 4.4.1 Arithmetic Operations

```python
a = 17
b = 5

# Addition
print(a + b)    # 22

# Subtraction
print(a - b)    # 12

# Multiplication
print(a * b)    # 85

# Division (always returns float!)
print(a / b)    # 3.4  — true division

# Floor Division (returns int when both operands are int)
print(a // b)   # 3    — floor division rounds toward -infinity
print(-17 // 5) # -4   (not -3!)  floor toward -inf
print(17 // -5) # -4   (not -3!)

# Modulus
print(a % b)    # 2    — remainder
print(-17 % 5)  # 3    — Python modulo always has sign of divisor

# Exponentiation
print(a ** b)   # 1419857
print(2 ** 10)  # 1024

# Absolute value
print(abs(-42))  # 42

# Divmod — returns (quotient, remainder) tuple
print(divmod(17, 5))   # (3, 2)
print(divmod(-17, 5))  # (-4, 3)
```

### 4.4.2 Comparison Operations

```python
x = 10
print(x == 10)  # True
print(x != 5)   # True
print(x > 5)    # True
print(x < 20)   # True
print(x >= 10)  # True
print(x <= 10)  # True

# Chained comparisons
print(1 < x < 20)    # True — Python supports chaining!
print(5 == 5 == 5)   # True
print(1 < 2 < 1)     # False
```

### 4.4.3 Floor Division vs Truncation

A critical distinction:

```python
# Floor division rounds toward -infinity
print(-7 // 2)   # -4  (floor of -3.5 is -4)

# int() truncates toward zero
print(int(-7/2)) # -3  (truncates -3.5 to -3)

# Python's // is TRUE floor division
# C's integer division is truncation
# Important for negative numbers!
```

## 4.5 Bitwise Operations

Bitwise operations work on the binary representation of integers:

### 4.5.1 AND (`&`)

Sets bits that are 1 in **both** operands:
```python
a = 0b1100   # 12
b = 0b1010   # 10

result = a & b
print(bin(result))  # 0b1000 = 8

# Visualization:
#   1100
# & 1010
# ------
#   1000
```

### 4.5.2 OR (`|`)

Sets bits that are 1 in **either** operand:
```python
a = 0b1100   # 12
b = 0b1010   # 10

result = a | b
print(bin(result))  # 0b1110 = 14

# Visualization:
#   1100
# | 1010
# ------
#   1110
```

### 4.5.3 XOR (`^`)

Sets bits that are 1 in **exactly one** operand:
```python
a = 0b1100   # 12
b = 0b1010   # 10

result = a ^ b
print(bin(result))  # 0b0110 = 6

# Visualization:
#   1100
# ^ 1010
# ------
#   0110
```

### 4.5.4 NOT (`~`)

Inverts all bits (bitwise complement). In Python: `~x = -(x+1)`
```python
a = 5        # 0b00000101
print(~a)    # -6  (-(5+1))

# Why -6? Two's complement:
# 5 in binary:     00000101
# flip all bits:   11111010
# in two's compl:  -6
```

### 4.5.5 Left Shift (`<<`)

Shifts bits left, filling right with zeros. Equivalent to multiplication by 2ⁿ:
```python
a = 1
print(a << 0)   # 1    (1 * 2^0)
print(a << 1)   # 2    (1 * 2^1)
print(a << 4)   # 16   (1 * 2^4)
print(a << 10)  # 1024 (1 * 2^10)

# Visualization (a = 3):
# 3 = 011
# 3 << 2 = 01100 = 12
```

### 4.5.6 Right Shift (`>>`)

Shifts bits right. Equivalent to integer division by 2ⁿ:
```python
a = 32   # 0b100000
print(a >> 1)   # 16
print(a >> 2)   # 8
print(a >> 5)   # 1

# Arithmetic right shift — sign bit is preserved for negative numbers
n = -32
print(n >> 1)   # -16 (arithmetic shift, not logical)
```

### 4.5.7 Bitwise Operation Use Cases

```python
# Check if number is even/odd
def is_even(n):
    return (n & 1) == 0

# Fast multiplication/division by powers of 2
n = 7
print(n << 3)   # 56 (7 * 8)
print(56 >> 3)  # 7  (56 / 8)

# Swap two variables without temp
a, b = 5, 10
a ^= b
b ^= a
a ^= b
print(a, b)   # 10, 5

# Set bit n
def set_bit(value, n):
    return value | (1 << n)

# Clear bit n
def clear_bit(value, n):
    return value & ~(1 << n)

# Toggle bit n
def toggle_bit(value, n):
    return value ^ (1 << n)

# Check bit n
def check_bit(value, n):
    return bool(value & (1 << n))
```

## 4.6 Integer Methods

### 4.6.1 `bit_length()`

Returns the number of bits required to represent the integer (excluding sign and leading zeros):

```python
print((0).bit_length())   # 0
print((1).bit_length())   # 1
print((255).bit_length())  # 8   (0b11111111)
print((256).bit_length())  # 9   (0b100000000)
print((-37).bit_length())  # 6   (ignores sign)

# Useful for:
# - Calculating minimum bits needed to store a value
# - Fast log2 calculation: bit_length() - 1 ≈ floor(log2(n))
def floor_log2(n):
    if n <= 0:
        raise ValueError("n must be positive")
    return n.bit_length() - 1

print(floor_log2(8))    # 3
print(floor_log2(100))  # 6
```

### 4.6.2 `bit_count()` (Python 3.10+)

Returns the number of ones (set bits) in the binary representation (population count / Hamming weight):

```python
print((0).bit_count())     # 0
print((255).bit_count())   # 8   (all 8 bits set)
print((256).bit_count())   # 1   (only bit 8 set)
print((0b10101).bit_count()) # 3  (three 1-bits)

# Use cases: error detection, cryptography, counting differences
def hamming_distance(a, b):
    """Number of positions where bits differ"""
    return (a ^ b).bit_count()

print(hamming_distance(0b1010, 0b1100))  # 2
```

### 4.6.3 `to_bytes(length, byteorder, *, signed=False)`

Converts integer to bytes:

```python
n = 1024  # 0x400

# Big-endian (most significant byte first)
b = n.to_bytes(2, byteorder='big')
print(b)          # b'\x04\x00'
print(list(b))    # [4, 0]

# Little-endian (least significant byte first)
b = n.to_bytes(2, byteorder='little')
print(b)          # b'\x00\x04'

# Signed integers
n = -1
b = n.to_bytes(1, byteorder='big', signed=True)
print(b)          # b'\xff'

# Must specify enough bytes
try:
    (1000).to_bytes(1, byteorder='big')  # OverflowError!
except OverflowError as e:
    print(e)

# Use case: network protocols, binary file formats
ip = 192 * 2**24 + 168 * 2**16 + 1 * 2**8 + 1
print(ip.to_bytes(4, byteorder='big'))  # b'\xc0\xa8\x01\x01'
```

### 4.6.4 `int.from_bytes(bytes, byteorder, *, signed=False)`

Class method that converts bytes to integer:

```python
# Big-endian
n = int.from_bytes(b'\x04\x00', byteorder='big')
print(n)    # 1024

# Little-endian
n = int.from_bytes(b'\x00\x04', byteorder='little')
print(n)    # 1024

# Signed
n = int.from_bytes(b'\xff', byteorder='big', signed=True)
print(n)    # -1

n = int.from_bytes(b'\xff', byteorder='big', signed=False)
print(n)    # 255

# Reading a 4-byte integer from binary data
data = b'\x00\x00\x01\x00'
value = int.from_bytes(data, byteorder='big')
print(value)  # 256
```

### 4.6.5 `as_integer_ratio()`

Returns a pair `(numerator, denominator)` such that `numerator / denominator == self` and `denominator > 0`:

```python
print((10).as_integer_ratio())   # (10, 1)
print((-3).as_integer_ratio())   # (-3, 1)
print((0).as_integer_ratio())    # (0, 1)

# Useful for Fraction conversion
from fractions import Fraction
n = 42
frac = Fraction(*n.as_integer_ratio())
print(frac)   # 42
```

## 4.7 Internal CPython Implementation

### 4.7.1 PyLongObject Structure

In CPython (the reference implementation), integers are stored as `PyLongObject`:

```c
/* Simplified from CPython Include/longobject.h and Objects/longobject.c */

typedef struct _PyLongValue {
    uintptr_t lv_tag;        /* tags and size information */
    digit ob_digit[1];       /* array of digits (base 2^30) */
} _PyLongValue;

typedef struct _longobject {
    PyObject_HEAD             /* ob_refcnt + ob_type */
    _PyLongValue long_value;
} PyLongObject;
```

### 4.7.2 Digit Storage

Python stores large integers in **base 2³⁰ (1,073,741,824)** using an array of 30-bit "digits". Each element of `ob_digit[]` stores 30 bits of the integer:

```
Integer 1,073,741,824 (= 2^30):

ob_digit[0] = 0         (low 30 bits: 0)
ob_digit[1] = 1         (next 30 bits: 1)
Actual value = ob_digit[0] + ob_digit[1] * (2^30)
             = 0 + 1 * 1073741824
             = 1073741824
```

For the number `1,000,000,000,000`:
```python
import sys
n = 1_000_000_000_000

# Check size in memory
print(sys.getsizeof(n))  # 32 bytes (2 digits needed)

# Compare
print(sys.getsizeof(1))         # 28 bytes (1 digit, fits in compact form)
print(sys.getsizeof(2**30))     # 28 bytes (still fits in 1 "digit")
print(sys.getsizeof(2**30 + 1)) # 32 bytes (needs 2 digits)
```

### 4.7.3 Memory Layout

```
Small integer (e.g., 5):  — Python 3.12+ uses compact representation
+-------------------+
| ob_refcnt: N      |  8 bytes
| ob_type: *int     |  8 bytes
| lv_tag: ...       |  8 bytes (encodes size and sign)
| ob_digit[0]: 5    |  4 bytes
+-------------------+
Total: ~28 bytes

Large integer (e.g., 2^64):
+-------------------+
| ob_refcnt: N      |  8 bytes
| ob_type: *int     |  8 bytes
| lv_tag: ...       |  8 bytes
| ob_digit[0]:      |  4 bytes (low 30 bits)
| ob_digit[1]:      |  4 bytes (middle 30 bits)
| ob_digit[2]:      |  4 bytes (high bits)
+-------------------+
Total: ~36 bytes
```

## 4.8 Small Integer Cache (Interning)

CPython pre-allocates integer objects for values in the range **-5 to 256** (inclusive). These are cached and reused, saving memory and time:

```python
# Small integers are cached — same object
a = 100
b = 100
print(a is b)    # True  — same object from cache
print(id(a) == id(b))  # True

# Large integers are NOT cached — different objects
x = 1000
y = 1000
print(x is y)   # False (usually) — different objects
print(x == y)   # True  — same value

# Why -5 to 256?
# -5 to 0: common negative indices and error codes
# 0 to 255: covers all ASCII byte values and common counts
# 256: frequently used as 2^8

# IMPORTANT: Never use 'is' to compare integer values!
# Always use '==' for value comparison
```

**Why this matters:**
```python
# This can cause subtle bugs:
n = 1000
m = 1000
if n is m:   # WRONG — may be True or False depending on context!
    print("equal")
if n == m:   # CORRECT — always checks value
    print("equal")
```

### String Interning for Integers
```python
# In interactive mode vs module context — behavior may differ
# In a module, Python may intern more constants at compile time
a = 257
b = 257
# In same code block, compiler may optimize: a is b → True
# As separate statements at REPL: a is b → False
# This inconsistency is why you MUST use == for int comparison
```

## 4.9 Time Complexity

| Operation | Time Complexity | Notes |
|-----------|----------------|-------|
| Addition `a + b` | O(max(len(a), len(b))) | len = number of digits |
| Subtraction `a - b` | O(max(len(a), len(b))) | |
| Multiplication `a * b` | O(n^1.585) | Karatsuba algorithm for large n |
| Division `a // b` | O(n²) | Long division |
| Exponentiation `a**b` | O(log(b) × M(len(a))) | Fast exponentiation |
| Modulo `a % b` | O(n²) | Similar to division |
| Comparison | O(min(len(a), len(b))) | |
| `bit_length()` | O(1) | Stored in tag |

Where `n = max(len(a), len(b))` and `M(n)` is the cost of multiplication.

## 4.10 Real-World Applications

```python
# 1. Cryptography — RSA key generation needs large primes
def is_prime_naive(n):
    if n < 2:
        return False
    for i in range(2, int(n**0.5) + 1):
        if n % i == 0:
            return False
    return True

# Python handles: pow(base, exp, mod) efficiently
# This is modular exponentiation — used in RSA, Diffie-Hellman
result = pow(2, 10000, 1000000007)  # Fast, no memory explosion

# 2. Combinatorics
import math
print(math.factorial(20))   # 2432902008176640000

# 3. Checksums
def checksum(data: bytes) -> int:
    return sum(data) % 256

# 4. Bit manipulation in flags
READ  = 0b001   # 1
WRITE = 0b010   # 2
EXEC  = 0b100   # 4

permissions = READ | WRITE    # 3
print(bool(permissions & READ))   # True
print(bool(permissions & EXEC))   # False

# 5. Hash values
print(hash(42))    # 42 (ints hash to themselves for small values)
```

## 4.11 Common Mistakes

```python
# Mistake 1: Integer division returns float in Python 3
result = 7 / 2       # 3.5 (float!) NOT 3
result = 7 // 2      # 3   (floor division)

# Mistake 2: Using 'is' instead of '==' for large integers
x = 1000
y = 1000
if x is y:           # WRONG! May fail unpredictably
    pass
if x == y:           # CORRECT
    pass

# Mistake 3: Floor division with negatives
print(-7 // 2)       # -4, not -3!

# Mistake 4: Forgetting int() truncates, doesn't round
print(int(2.9))      # 2, not 3!
print(round(2.9))    # 3

# Mistake 5: Assuming Python int behaves like C int (overflow)
# In C: INT_MAX + 1 overflows to negative
# In Python: no overflow, arbitrary precision
x = 2**63 - 1        # 9223372036854775807
print(x + 1)         # 9223372036854775808 — no overflow!
```

## 4.12 Interview Questions — Integers

**Beginner:**
1. What is the difference between `//` and `/` in Python?
2. Can Python integers overflow? Why or why not?
3. What does `~5` evaluate to and why?

**Intermediate:**
4. What is the small integer cache? What range does it cover?
5. Why should you avoid using `is` to compare integers?
6. What does `divmod(17, 5)` return?

**Advanced:**
7. How does CPython store large integers internally?
8. What is the time complexity of multiplying two n-digit Python integers?
9. Explain the difference between floor division and truncation for negative numbers.

**FAANG-Level:**
10. Implement `pow(base, exp, mod)` using fast exponentiation.
11. Explain how Python's arbitrary-precision integers affect memory usage when processing large datasets.

---

# Chapter 5: Floating Point Numbers (float)

## 5.1 Definition

A **float** in Python represents a real number using floating-point notation. Python's `float` type is implemented as a C `double`, which conforms to the IEEE 754 double-precision standard. This gives approximately 15-17 significant decimal digits of precision, with a range from approximately ±5 × 10⁻³²⁴ to ±1.8 × 10³⁰⁸.

```python
x = 3.14
x = -0.001
x = 1.23e10     # Scientific notation: 1.23 × 10^10
x = 1.23E-5     # 0.0000123
x = .5          # 0.5 (leading zero optional)
x = 5.          # 5.0 (trailing decimal point valid)
```

## 5.2 IEEE 754 Standard

Python's `float` follows IEEE 754 double-precision binary floating-point format, using 64 bits total:

```
 63  62      52  51                             0
+----+----------+-------------------------------+
|Sign| Exponent |           Mantissa            |
| 1b |   11b    |             52b               |
+----+----------+-------------------------------+

Sign bit:     0 = positive, 1 = negative
Exponent:     Biased exponent (bias = 1023)
Mantissa:     Fractional part (1.mantissa implied)
```

### 5.2.1 Sign Bit (1 bit)

- `0` = positive number
- `1` = negative number

This is why floats have both `+0.0` and `-0.0`:
```python
pos_zero = 0.0
neg_zero = -0.0
print(pos_zero == neg_zero)  # True
print(1/pos_zero)   # inf
print(1/neg_zero)   # -inf
```

### 5.2.2 Exponent (11 bits)

Stored as a biased exponent: actual exponent = stored value − 1023

| Stored Value | Actual Exponent | Meaning |
|-------------|----------------|---------|
| 0 | −1023 | Zero or subnormal |
| 1 | −1022 | Minimum normal |
| 1023 | 0 | (1.mantissa × 2⁰) |
| 2046 | 1023 | Maximum normal |
| 2047 | Special | NaN or ±Infinity |

### 5.2.3 Mantissa/Significand (52 bits)

The fractional part. The leading `1` is implicit (normalized form):

```
Actual value = (-1)^sign × 2^(exponent-1023) × 1.mantissa
```

### 5.2.4 Example: How 0.1 Is Stored

```python
import struct

# Get the IEEE 754 bytes for 0.1
b = struct.pack('d', 0.1)
print(b.hex())  # 9a9999999999b93f (little-endian)

# In binary:
# 0.1 in binary is: 0.000110011001100110011... (repeating!)
# IEEE 754 representation:
# Sign:     0 (positive)
# Exponent: 01111111011 = 1019 → actual = 1019-1023 = -4
# Mantissa: 1001100110011001100110011001100110011001100110011010
# Value:    1.1001100110011... × 2^(-4) ≈ 0.1 (but not exactly!)
```

## 5.3 The Fundamental Precision Problem

This is the most important thing to understand about floating-point:

```python
# The famous 0.1 + 0.2 problem
print(0.1 + 0.2)           # 0.30000000000000004 !!!
print(0.1 + 0.2 == 0.3)    # False !!!

# Why? 0.1 cannot be represented exactly in binary:
# 0.1 in binary = 0.00011001100110011... (infinite repeating)
# The 52-bit mantissa stores a rounded approximation

# What Python actually stores:
import decimal
print(decimal.Decimal(0.1))
# 0.1000000000000000055511151231257827021181583404541015625
# That's what 0.1 really is in IEEE 754!

print(decimal.Decimal(0.2))
# 0.200000000000000011102230246251565404236316680908203125

print(decimal.Decimal(0.3))
# 0.299999999999999988897769753748434595763683319091796875

# Their sum:
# 0.1000...5551... + 0.2000...1110... ≈ 0.3000...0004...
# which is slightly more than 0.3
```

### 5.3.1 Correct Ways to Compare Floats

```python
import math

a = 0.1 + 0.2
b = 0.3

# Option 1: math.isclose() — recommended!
print(math.isclose(a, b))              # True
print(math.isclose(a, b, rel_tol=1e-9)) # True (relative tolerance)
print(math.isclose(a, b, abs_tol=1e-9)) # True (absolute tolerance)

# Option 2: round() before comparing
print(round(a, 10) == round(b, 10))    # True

# Option 3: abs difference threshold
EPSILON = 1e-10
print(abs(a - b) < EPSILON)            # True

# NEVER use == for float comparison:
print(a == b)   # False — DON'T DO THIS
```

## 5.4 Float Creation

```python
# Literals
x = 3.14
x = -2.718
x = 1e10      # 10000000000.0
x = 1.5e-3    # 0.0015

# Constructor
x = float(42)       # 42.0    (from int)
x = float("3.14")   # 3.14    (from string)
x = float("1e10")   # 1e10    (scientific notation)
x = float("inf")    # inf
x = float("-inf")   # -inf
x = float("nan")    # nan

# From bool
x = float(True)    # 1.0
x = float(False)   # 0.0

# Errors
try:
    float("hello")  # ValueError
    float(None)     # TypeError
except (ValueError, TypeError) as e:
    print(e)
```

## 5.5 Float Methods

### 5.5.1 `is_integer()`

Returns `True` if the float has no fractional part:

```python
print((3.0).is_integer())    # True
print((3.5).is_integer())    # False
print((0.0).is_integer())    # True
print((-2.0).is_integer())   # True
print((1e100).is_integer())  # True — very large, but technically whole

# Use case: validate input
def to_int_if_whole(x):
    if isinstance(x, float) and x.is_integer():
        return int(x)
    return x

print(to_int_if_whole(3.0))   # 3
print(to_int_if_whole(3.5))   # 3.5
```

### 5.5.2 `hex()` and `fromhex()`

Convert float to/from hexadecimal representation. Hex representation is **exact** — no rounding errors:

```python
# hex() — exact representation
print((0.1).hex())    # '0x1.999999999999ap-4'
print((0.5).hex())    # '0x1.0000000000000p-1'
print((1.0).hex())    # '0x1.0000000000000p+0'
print((-3.14).hex())  # '-0x1.47ae147ae147bp+1'

# fromhex() — exact reconstruction
x = float.fromhex('0x1.999999999999ap-4')
print(x)              # 0.1  (exact reconstruction!)

# This is how float literals are specified exactly
# Useful in cross-language data exchange where exact floats matter

# Format: 0x[integer].[fraction]p[exponent]
# Value = integer.fraction in hex × 2^exponent
```

### 5.5.3 `as_integer_ratio()`

Returns `(numerator, denominator)` such that their ratio exactly equals the float:

```python
print((0.5).as_integer_ratio())    # (1, 2)
print((0.1).as_integer_ratio())    # (3602879701896397, 36028797018963968)
print((3.14).as_integer_ratio())   # (7070651414971679, 2251799813685248)
print((1.0).as_integer_ratio())    # (1, 1)
print((-2.5).as_integer_ratio())   # (-5, 2)

# This shows you the EXACT value stored:
n, d = (0.1).as_integer_ratio()
print(f"0.1 is stored as {n}/{d}")
# 0.1 is stored as 3602879701896397/36028797018963968
# = 0.100000000000000005551115...

# Errors
try:
    float('inf').as_integer_ratio()  # OverflowError
    float('nan').as_integer_ratio()  # ValueError
except (OverflowError, ValueError) as e:
    print(e)
```

## 5.6 Rounding

### 5.6.1 The `round()` Function

```python
# Basic rounding
print(round(3.14159))     # 3       (nearest int)
print(round(3.14159, 2))  # 3.14    (2 decimal places)
print(round(3.14159, 4))  # 3.1416  (4 decimal places)

# Negative ndigits — round to tens, hundreds, etc.
print(round(1234, -1))    # 1230
print(round(1234, -2))    # 1200
print(round(1234, -3))    # 1000
```

### 5.6.2 Banker's Rounding (Round Half to Even)

Python uses **banker's rounding** (round half to even), not the "round half up" you learned in school:

```python
# Banker's rounding — rounds 0.5 to the nearest EVEN number
print(round(0.5))   # 0  — rounds to 0 (even)
print(round(1.5))   # 2  — rounds to 2 (even)
print(round(2.5))   # 2  — rounds to 2 (even)
print(round(3.5))   # 4  — rounds to 4 (even)
print(round(4.5))   # 4  — rounds to 4 (even)

# Why? Reduces cumulative rounding bias in statistical computations
# Used in financial and scientific computing
# IEEE 754 standard default rounding mode

# Traditional "round half up":
import math
def round_half_up(n, decimals=0):
    multiplier = 10 ** decimals
    return math.floor(n * multiplier + 0.5) / multiplier

print(round_half_up(2.5))   # 3.0
print(round_half_up(3.5))   # 4.0
```

### 5.6.3 Rounding Issues

```python
# Floating-point representation can cause surprises
print(round(2.675, 2))   # 2.67, not 2.68!

# Why? 2.675 is stored as:
import decimal
print(decimal.Decimal(2.675))
# 2.67499999999999982236431605997495353221893310546875
# So it rounds DOWN to 2.67

# For financial rounding, use Decimal:
from decimal import Decimal, ROUND_HALF_UP
result = Decimal('2.675').quantize(Decimal('0.01'), rounding=ROUND_HALF_UP)
print(result)  # 2.68 — correct!
```

## 5.7 Infinity

```python
# Creating infinity
pos_inf = float('inf')
neg_inf = float('-inf')

import math
print(math.inf)     # inf
print(-math.inf)    # -inf

# Arithmetic with infinity
print(pos_inf + 1000)    # inf
print(pos_inf * 2)       # inf
print(pos_inf + neg_inf)  # nan (indeterminate form ∞ - ∞)
print(1 / pos_inf)       # 0.0
print(pos_inf * 0)       # nan (indeterminate form ∞ × 0)

# Comparisons
print(pos_inf > 10**1000)   # True — infinity is larger than any finite number
print(neg_inf < -10**1000)  # True

# Checking for infinity
print(math.isinf(pos_inf))   # True
print(math.isinf(1.0))       # False

# Use case: initialize minimum/maximum search
def find_max(numbers):
    max_val = float('-inf')
    for n in numbers:
        if n > max_val:
            max_val = n
    return max_val

print(find_max([3, 1, 4, 1, 5, 9]))  # 9
```

## 5.8 NaN (Not a Number)

```python
# Creating NaN
nan = float('nan')
import math
print(math.nan)    # nan

# NaN is contagious — any operation with NaN returns NaN
print(nan + 1)     # nan
print(nan * 0)     # nan
print(nan == nan)  # False!!! NaN is not equal to itself
print(nan != nan)  # True!!!

# This is a gotcha — cannot use == to check for NaN
def check_nan_wrong(x):
    return x == float('nan')    # ALWAYS False!

def check_nan_correct(x):
    return math.isnan(x)        # Correct

print(check_nan_wrong(nan))   # False (wrong!)
print(check_nan_correct(nan)) # True (correct!)

# NaN comparison behavior
print(nan > 0)    # False
print(nan < 0)    # False
print(nan == 0)   # False
# All comparisons with NaN return False (except !=)

# Use cases: sentinel values in numeric computations
data = [1.0, float('nan'), 3.0, float('nan'), 5.0]
clean = [x for x in data if not math.isnan(x)]
print(clean)  # [1.0, 3.0, 5.0]
```

## 5.9 Float Internal Representation (CPython)

```c
/* CPython Objects/floatobject.c */
typedef struct {
    PyObject_HEAD      /* ob_refcnt + ob_type */
    double ob_fval;    /* the IEEE 754 double value */
} PyFloatObject;
```

Size: 24 bytes total (on 64-bit system):
- 8 bytes: `ob_refcnt` (reference count)
- 8 bytes: `ob_type` (pointer to float type)
- 8 bytes: `ob_fval` (the actual double)

```python
import sys
print(sys.getsizeof(3.14))   # 24 bytes — always, regardless of value
print(sys.getsizeof(0.0))    # 24 bytes
print(sys.getsizeof(1e308))  # 24 bytes — same size for any float!
```

## 5.10 Float Limits

```python
import sys

print(sys.float_info)
# sys.float_info(
#   max=1.7976931348623157e+308,   # Largest float
#   max_exp=1024,                   # Maximum binary exponent
#   max_10_exp=308,                 # Maximum decimal exponent
#   min=2.2250738585072014e-308,    # Smallest POSITIVE NORMAL float
#   min_exp=-1021,
#   min_10_exp=-307,
#   dig=15,                         # Significant decimal digits
#   mant_dig=53,                    # Bits in mantissa (including implicit 1)
#   epsilon=2.220446049250313e-16,  # Machine epsilon
#   radix=2,
#   rounds=1
# )

# Machine epsilon — smallest x such that 1.0 + x != 1.0
epsilon = sys.float_info.epsilon
print(1.0 + epsilon == 1.0)          # False
print(1.0 + epsilon/2 == 1.0)        # True  (too small to make a difference)

# Overflow
try:
    result = 1.8e308 * 2    # OverflowError
except OverflowError as e:
    print(e)
# But:
print(float('inf'))  # No error — use infinity directly

# Underflow — very small numbers become 0.0
print(5e-324)         # 5e-324 (smallest positive subnormal float)
print(5e-325)         # 0.0 — underflows to zero
```

## 5.11 Performance

```python
import timeit

# Float operations are fast — hardware-accelerated
float_add = timeit.timeit("3.14 + 2.71", number=10_000_000)
int_add = timeit.timeit("314 + 271", number=10_000_000)

print(f"Float add: {float_add:.3f}s")
print(f"Int add:   {int_add:.3f}s")
# Both are similar — small ints and floats use hardware operations

# But large int arithmetic is slow:
big_int_add = timeit.timeit("(10**100) + (10**100)", number=100_000)
print(f"Big int add: {big_int_add:.3f}s")  # Much slower!
```

## 5.12 Best Practices

```python
# 1. Use math.isclose() for float comparison
import math
assert math.isclose(0.1 + 0.2, 0.3)

# 2. Use Decimal for financial calculations
from decimal import Decimal
price = Decimal('19.99')
tax = Decimal('0.08')
total = price * (1 + tax)
print(total)  # 21.5892 — exact!

# 3. Avoid accumulating float errors
# BAD:
total = 0.0
for i in range(1000):
    total += 0.1
print(total)   # 99.9999... (not exactly 100.0)

# BETTER:
from decimal import Decimal
total = Decimal('0')
for i in range(1000):
    total += Decimal('0.1')
print(total)   # 100.0 — exact!

# 4. Use math.fsum() for accurate float summation
print(math.fsum([0.1] * 10))   # 1.0 — uses compensated summation

# 5. Be aware of NaN propagation
def safe_divide(a, b):
    if b == 0:
        return float('nan')
    return a / b
```

## 5.13 Interview Questions — Float

**Beginner:**
1. Why does `0.1 + 0.2 != 0.3` in Python?
2. What is `float('inf')`? What operations produce it?
3. What is `NaN`? How do you check for it correctly?

**Intermediate:**
4. Explain the IEEE 754 double-precision format.
5. What is banker's rounding? How does Python's `round()` implement it?
6. What is `sys.float_info.epsilon` and why is it important?

**Advanced:**
7. When would you use `Decimal` instead of `float`?
8. Explain why `round(2.675, 2)` gives `2.67` instead of `2.68`.
9. What is the difference between `+0.0` and `-0.0`?

---
