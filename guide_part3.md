# Chapter 11: Type Conversion

## 11.1 Implicit Conversion (Coercion)

Python performs automatic type promotion in limited cases:

```python
# int → float (when mixing in arithmetic)
print(1 + 2.0)          # 3.0  (int promoted to float)
print(type(1 + 2.0))    # float

# bool → int
print(True + 5)          # 6
print(False * 100)       # 0

# int/float → complex
print(1 + 2j + 3)        # (4+2j)  (int → complex)
print(1.5 + 2j)          # (1.5+2j) (float → complex)

# NOTE: No implicit conversion to Decimal or Fraction
from decimal import Decimal
try:
    Decimal('1.5') + 1.5   # TypeError — no auto-convert!
except TypeError as e:
    print(e)
```

## 11.2 Explicit Conversion Functions

### `int()` Conversion Matrix

```python
# From various types to int:
print(int(3.9))          # 3      — truncates (NOT rounds)
print(int(-3.9))         # -3     — toward zero
print(int(True))         # 1
print(int(False))        # 0
print(int("42"))         # 42
print(int("0xFF", 16))   # 255    — base 16
print(int("0b1010", 2))  # 10     — base 2
print(int("0o17", 8))    # 15     — base 8
print(int(Decimal('3.7')))  # 3   — truncates

# What int() CANNOT convert:
# int(None)         → TypeError
# int([1, 2])       → TypeError
# int("3.14")       → ValueError (needs float() first)
# int("hello")      → ValueError
```

### `float()` Conversion Matrix

```python
print(float(42))          # 42.0
print(float(True))        # 1.0
print(float(False))       # 0.0
print(float("3.14"))      # 3.14
print(float("1e10"))      # 10000000000.0
print(float("inf"))       # inf
print(float("nan"))       # nan
from decimal import Decimal
print(float(Decimal('3.14')))  # 3.14

# Cannot convert:
# float(None)     → TypeError
# float("hello")  → ValueError
# float(2+3j)     → TypeError (use abs() for magnitude)
```

### `complex()` Conversion Matrix

```python
print(complex(3))          # (3+0j)
print(complex(3.14))       # (3.14+0j)
print(complex(True))       # (1+0j)
print(complex(3, 4))       # (3+4j)
print(complex("3+4j"))     # (3+4j)   — no spaces allowed!
print(complex("j"))        # 1j

# Complex to real/imag
z = 3 + 4j
print(z.real, z.imag)     # 3.0  4.0
print(abs(z))             # 5.0 (magnitude)
```

### `bool()` Conversion Matrix

```python
# Falsy values:
print(bool(0))       # False
print(bool(0.0))     # False
print(bool(0j))      # False
print(bool(""))      # False
print(bool([]))      # False
print(bool({}))      # False
print(bool(None))    # False

# Truthy values:
print(bool(1))       # True
print(bool(-1))      # True
print(bool(3.14))    # True
print(bool("0"))     # True  (non-empty string!)
print(bool([0]))     # True  (non-empty list!)
print(bool(object())) # True (default)
```

### `Decimal()` Conversion

```python
from decimal import Decimal

# From string — PREFERRED (exact)
print(Decimal('3.14'))         # Decimal('3.14')

# From int — exact
print(Decimal(42))             # Decimal('42')

# From float — IMPRECISE (inherits float error)
print(Decimal(0.1))            # Decimal('0.1000000000000000055511...')

# From tuple
print(Decimal((0, (3, 1, 4), -2)))  # Decimal('3.14')

# From Fraction
from fractions import Fraction
print(Decimal(Fraction(1, 3)))  # Decimal('0.333...') with current precision
```

### `Fraction()` Conversion

```python
from fractions import Fraction

# From integers
print(Fraction(3, 4))          # 3/4

# From float
print(Fraction(0.1))           # 3602879701896397/36028797018963968 (float's value)
print(Fraction(1.5))           # 3/2  (1.5 is exact in binary)

# From string — PREFERRED
print(Fraction('0.1'))         # 1/10  (exact!)
print(Fraction('3/4'))         # 3/4

# From Decimal
from decimal import Decimal
print(Fraction(Decimal('0.1'))) # 1/10  (exact!)

# From float with limit_denominator
print(Fraction(math.pi).limit_denominator(100))  # 22/7
```

## 11.3 Type Conversion Compatibility Matrix

| From ↓ To → | `int` | `float` | `complex` | `bool` | `Decimal` | `Fraction` |
|-------------|-------|---------|-----------|--------|-----------|------------|
| `int` | — | ✅ | ✅ | ✅ | ✅ | ✅ |
| `float` | ✅ truncates | — | ✅ | ✅ | ⚠️ imprecise | ⚠️ exact but long |
| `complex` | ❌ TypeError | ❌ TypeError | — | ✅ | ❌ | ❌ |
| `bool` | ✅ (0 or 1) | ✅ | ✅ | — | ✅ | ✅ |
| `Decimal` | ✅ truncates | ✅ (approximate) | ❌ | ✅ | — | ✅ |
| `Fraction` | ✅ truncates | ✅ (approximate) | ❌ | ✅ | ✅ | — |
| `str` | ✅ if valid | ✅ if valid | ✅ if valid | any str | ✅ if valid | ✅ if valid |
| `None` | ❌ | ❌ | ❌ | ✅ (False) | ❌ | ❌ |

Legend: ✅ = works, ❌ = TypeError/ValueError, ⚠️ = works but with caveats

## 11.4 Numeric Promotion Rules

```python
# Python follows these promotion rules in mixed arithmetic:
# bool + int     → int
# int + float    → float
# float + complex → complex

# Promotion chain: bool → int → float → complex
examples = [
    (True, 1),         # bool + int   = int
    (1, 1.0),          # int + float  = float
    (1.0, 1+0j),       # float + complex = complex
    (True, 1.0),       # bool + float = float
    (True, 1+0j),      # bool + complex = complex
]

for a, b in examples:
    result = a + b
    print(f"{type(a).__name__} + {type(b).__name__} = {type(result).__name__}: {result}")
```

---

# Chapter 12: Type Checking

## 12.1 `type()` — Exact Type Check

`type(obj)` returns the exact type of an object. Use for strict type matching:

```python
x = True

# type() — exact match
print(type(x))           # <class 'bool'>
print(type(x) is bool)   # True
print(type(x) is int)    # False! bool is NOT int via type()

# type() for type creation (metaclass usage)
# type(name, bases, dict) creates a new class
MyInt = type('MyInt', (int,), {'double': lambda self: self * 2})
n = MyInt(5)
print(n.double())  # 10
print(type(n))     # <class '__main__.MyInt'>
```

## 12.2 `isinstance()` — Inheritance-Aware Type Check

`isinstance(obj, classinfo)` is the **recommended** type check — it respects inheritance:

```python
x = True

# isinstance — respects inheritance hierarchy
print(isinstance(x, bool))     # True
print(isinstance(x, int))      # True!  (bool IS-A int)
print(isinstance(x, object))   # True   (everything IS-A object)

# Checking multiple types simultaneously
def process(x):
    if isinstance(x, (int, float)):
        return f"Number: {x}"
    elif isinstance(x, str):
        return f"String: {x}"
    else:
        return f"Other: {type(x).__name__}"

print(process(42))      # Number: 42
print(process(3.14))    # Number: 3.14
print(process("hi"))    # String: hi
print(process([1, 2]))  # Other: list

# isinstance with numbers ABC
from numbers import Integral, Rational, Real, Complex

print(isinstance(42, Integral))         # True
print(isinstance(3.14, Rational))       # False
print(isinstance(3.14, Real))           # True
from fractions import Fraction
print(isinstance(Fraction(1,3), Rational))  # True
```

## 12.3 `issubclass()` — Class Hierarchy Check

```python
# issubclass(subclass, superclass)
print(issubclass(bool, int))       # True
print(issubclass(bool, object))    # True
print(issubclass(int, float))      # False
print(issubclass(float, complex))  # False (structurally, but not in Python's class hierarchy)

# Check against multiple superclasses
print(issubclass(bool, (int, str)))   # True

# Every class is a subclass of itself
print(issubclass(int, int))    # True

# Real numeric tower via numbers ABC
from numbers import Integral, Rational, Real, Complex, Number
print(issubclass(int, Integral))    # True
print(issubclass(int, Rational))    # True
print(issubclass(int, Real))        # True
print(issubclass(int, Number))      # True
print(issubclass(float, Integral))  # False
print(issubclass(float, Real))      # True
```

## 12.4 Type Checking Best Practices

```python
# 1. Use isinstance() in most cases (respects inheritance)
def area(shape):
    if isinstance(shape, Circle):    # Works for subclasses too!
        return math.pi * shape.radius ** 2

# 2. Use type() only when you need EXACT type match
def is_bool_exactly(x):
    return type(x) is bool

print(is_bool_exactly(True))   # True
print(is_bool_exactly(1))      # False (1 is int, not bool)

# 3. Use Duck Typing when possible (most Pythonic)
def make_iterable(x):
    try:
        iter(x)
        return x
    except TypeError:
        return [x]

# 4. Use Abstract Base Classes for interface checking
from collections.abc import Sequence, Mapping, Callable
print(isinstance([1, 2], Sequence))     # True
print(isinstance({"a": 1}, Mapping))   # True

# 5. Type hints for documentation (not runtime enforcement)
from typing import Union, Optional
def process(x: Union[int, float]) -> float:
    return float(x)
```

---

# Chapter 13: Comparisons Between Types

## 13.1 Feature Comparison Table

| Feature | `int` | `float` | `complex` | `Decimal` | `Fraction` | `bool` | `NoneType` |
|---------|-------|---------|-----------|-----------|------------|--------|------------|
| **Memory (bytes)** | 28+ | 24 | 32 | ~100 | ~200 | 28 | 16 |
| **Precision** | Unlimited | ~15-17 digits | ~15-17 per part | Configurable | Exact rational | Binary | N/A |
| **Mutable** | No | No | No | No | No | No | No |
| **Hashable** | Yes | Yes | Yes | Yes | Yes | Yes | Yes |
| **Ordered** | Yes | Yes | No | Yes | Yes | Yes | No |
| **Arithmetic** | All | All | No // % | All | All | Inherited | None |
| **Infinite values** | No | Yes | No | Yes | No | No | No |
| **NaN** | No | Yes | No | Yes | No | No | No |

## 13.2 Memory Comparison

```python
import sys
from decimal import Decimal
from fractions import Fraction

print(sys.getsizeof(None))              # 16 bytes
print(sys.getsizeof(True))             # 28 bytes
print(sys.getsizeof(0))               # 24 bytes
print(sys.getsizeof(1))               # 28 bytes
print(sys.getsizeof(2**30))           # 28 bytes
print(sys.getsizeof(2**31))           # 32 bytes
print(sys.getsizeof(3.14))            # 24 bytes
print(sys.getsizeof(2+3j))            # 32 bytes
print(sys.getsizeof(Decimal('3.14'))) # ~100 bytes
print(sys.getsizeof(Fraction(1, 3)))  # ~60+ bytes

# Memory usage grows with integer size:
for exp in [1, 10, 100, 1000]:
    n = 10**exp
    print(f"10^{exp}: {sys.getsizeof(n)} bytes")
# 10^1:    28 bytes
# 10^10:   32 bytes
# 10^100:  72 bytes
# 10^1000: 472 bytes
```

## 13.3 Performance Comparison

```python
import timeit

setups = {
    'int': 'a, b = 1000000, 999999',
    'float': 'a, b = 1000000.0, 999999.0',
    'decimal': 'from decimal import Decimal; a, b = Decimal("1000000"), Decimal("999999")',
    'fraction': 'from fractions import Fraction; a, b = Fraction(1000000), Fraction(999999)',
}

for name, setup in setups.items():
    t = timeit.timeit('a + b', setup=setup, number=1_000_000)
    print(f"{name:10s}: {t:.4f}s")

# Typical results:
# int       : ~0.05s  (fastest for moderate size)
# float     : ~0.05s  (similar to int)
# decimal   : ~1.50s  (30x slower)
# fraction  : ~2.00s  (40x slower)
```

## 13.4 Precision Comparison

```python
# Comparing precision when computing 1/3
from decimal import Decimal, getcontext
from fractions import Fraction

# float — approximately 15-17 significant digits
f = 1/3
print(f)  # 0.3333333333333333 (16 significant digits)

# Decimal — configurable precision
getcontext().prec = 50
d = Decimal(1) / Decimal(3)
print(d)  # 0.33333333333333333333333333333333333333333333333333

# Fraction — exact rational (infinite precision)
fr = Fraction(1, 3)
print(fr)  # 1/3 (exact! stored as numerator/denominator)

# The crucial difference — accumulation
n = 1000
float_sum = sum(1/3 for _ in range(n))
decimal_sum = sum(Decimal(1)/Decimal(3) for _ in range(n))
fraction_sum = sum(Fraction(1, 3) for _ in range(n))

print(float_sum)     # 333.3333333333326 (accumulated error!)
print(decimal_sum)   # 333.333... (limited by precision)
print(fraction_sum)  # 1000/3 (exact! = 333.333...)
```

## 13.5 Use Case Comparison

| Scenario | Recommended Type | Reason |
|----------|----------------|--------|
| Counting things | `int` | Exact, unlimited |
| Array indices | `int` | Required |
| Loop counters | `int` | Natural |
| Scientific computation | `float` | Fast, hardware-native |
| Physics simulations | `float` | Speed over precision |
| Currency/money | `Decimal` | Exact decimal arithmetic |
| Tax calculation | `Decimal` | Legal precision requirements |
| Probability | `Fraction` | Exact rational results |
| Signal processing | `complex` | Mathematical requirement |
| Feature flags | `bool` | Semantic clarity |
| Optional values | `None` | Standard Python convention |

---

# Chapter 14: Memory Management

## 14.1 Reference Counting

Python's primary memory management mechanism is **reference counting**. Every object has a counter (`ob_refcnt`) tracking how many references point to it:

```python
import sys

# Check reference count
x = [1, 2, 3]
print(sys.getrefcount(x))   # 2 (x + argument to getrefcount)

y = x
print(sys.getrefcount(x))   # 3 (x, y, getrefcount arg)

del y
print(sys.getrefcount(x))   # 2 (back to x + arg)

# When refcount hits 0, object is immediately deallocated
z = [1, 2, 3]
del z    # Object destroyed immediately (reference count → 0)
```

## 14.2 Reference Counting for Numeric Types

```python
import sys

# Integers in the small cache have high reference counts
print(sys.getrefcount(0))     # Very high! — 0 is referenced everywhere
print(sys.getrefcount(True))  # High — True is a singleton used often
print(sys.getrefcount(None))  # Very high — None is a singleton

# Large integers have low reference counts
big = 10**100
print(sys.getrefcount(big))   # 2 (big + getrefcount arg)
```

## 14.3 Garbage Collection — Cycle Detection

Reference counting alone cannot handle **reference cycles**:

```python
import gc

# Reference cycle — neither object can be freed by refcounting alone
a = [1, 2, 3]
b = [4, 5, 6]
a.append(b)   # a references b
b.append(a)   # b references a — CYCLE!

del a
del b
# a and b still exist in memory — cycle keeps refcount > 0!
# CPython's cyclic GC handles this

# Force garbage collection
gc.collect()   # Finds and frees cycles

# Numeric types cannot form cycles (they're immutable)
# int, float, bool, None, Decimal, Fraction — no GC needed
```

## 14.4 Small Integer Cache Details

```python
# Python pre-creates integers from -5 to 256 at startup
# These objects are NEVER freed

# Verify the cache
a = -5
b = -5
print(a is b)   # True — cached

a = -6
b = -6
print(a is b)   # False — not cached

for i in range(-5, 257):
    assert i is int(i)   # All cached ints are singletons

# The cache saves memory for common integers
# But large integers create new objects each time
big1 = 1000
big2 = 1000
print(big1 is big2)   # False — separate objects
```

## 14.5 Float Memory Optimization

CPython maintains a **float free list** — a pool of up to 256 recently freed float objects that are reused:

```python
# When a float is deleted, its memory goes to the free list
x = 3.14
id_x = id(x)
del x

y = 2.71
print(id(y) == id_x)   # Often True! — reuses freed memory

# This optimization makes float creation fast in loops
# New float objects don't require malloc/free each time
```

## 14.6 None and bool — Eternal Singletons

```python
import sys

# None and True/False are NEVER garbage collected
# They exist for the lifetime of the Python interpreter

# None is always the same object
print(id(None))  # Always the same value per Python session

# True and False are always the same objects
print(id(True))  # Always same
print(id(False)) # Always same

# Reference count shows these are heavily referenced:
print(sys.getrefcount(None))   # Hundreds or thousands
```

## 14.7 Memory Layout Summary

```
Object Memory Layouts in CPython (64-bit):

NoneType:
+------------------+
| ob_refcnt (8b)   |  Reference count
| ob_type   (8b)   |  Pointer to NoneType
+------------------+
Total: 16 bytes

bool (inherits from int):
+------------------+
| ob_refcnt (8b)   |
| ob_type   (8b)   |  Pointer to bool
| lv_tag    (8b)   |  Size/sign info
| ob_digit  (4b)   |  Value: 0 or 1
+------------------+
Total: 28 bytes

float:
+------------------+
| ob_refcnt (8b)   |
| ob_type   (8b)   |  Pointer to float
| ob_fval   (8b)   |  IEEE 754 double
+------------------+
Total: 24 bytes

complex:
+------------------+
| ob_refcnt (8b)   |
| ob_type   (8b)   |  Pointer to complex
| real      (8b)   |  IEEE 754 double (real part)
| imag      (8b)   |  IEEE 754 double (imag part)
+------------------+
Total: 32 bytes

int (small, e.g., 42):
+------------------+
| ob_refcnt (8b)   |
| ob_type   (8b)   |  Pointer to int
| lv_tag    (8b)   |  Encodes size/sign
| ob_digit  (4b)   |  42
+------------------+
Total: 28 bytes

int (large, needs N digits):
+------------------+
| ob_refcnt (8b)   |
| ob_type   (8b)   |
| lv_tag    (8b)   |
| ob_digit[0](4b)  |
| ob_digit[1](4b)  |
...
| ob_digit[N](4b)  |
+------------------+
Total: 28 + 4N bytes
```

---

# Chapter 15: CPython Internals

## 15.1 PyObject — The Universal Base

Every Python object starts with `PyObject`:

```c
/* Include/object.h */
typedef struct _object {
    Py_ssize_t ob_refcnt;    /* Reference count */
    PyTypeObject *ob_type;   /* Pointer to type */
} PyObject;
```

Variable-size objects also have:
```c
typedef struct {
    PyObject ob_base;
    Py_ssize_t ob_size;  /* Number of items */
} PyVarObject;
```

## 15.2 PyLongObject — Integer Implementation

```c
/* Include/longobject.h */
typedef uint32_t digit;  /* 30 bits used per digit */
#define PyLong_SHIFT 30
#define PyLong_BASE  ((digit)1 << PyLong_SHIFT)

/* Python 3.12+ uses compact representation */
typedef struct _PyLongValue {
    uintptr_t lv_tag;    /* ndigits and sign encoded */
    digit ob_digit[1];   /* Variable length digit array */
} _PyLongValue;

typedef struct _longobject {
    PyObject_HEAD         /* ob_refcnt + ob_type */
    _PyLongValue long_value;
} PyLongObject;
```

### How Integers Are Computed from Digits

```
For integer X with digits [d0, d1, d2, ...]:
X = d0 * (2^30)^0 + d1 * (2^30)^1 + d2 * (2^30)^2 + ...
X = d0 + d1 * 1073741824 + d2 * 1152921504606846976 + ...
```

### CPython Integer Operations (simplified)

```c
/* Integer addition — add two PyLongObjects */
PyObject *
long_add(PyObject *a, PyObject *b) {
    /* Get sizes */
    Py_ssize_t size_a = Py_ABS(Py_SIZE(a));
    Py_ssize_t size_b = Py_ABS(Py_SIZE(b));

    /* Result needs at most max(size_a, size_b) + 1 digits */
    Py_ssize_t size_z = (size_a > size_b ? size_a : size_b) + 1;

    PyLongObject *z = _PyLong_New(size_z);
    /* ... digit-by-digit addition with carry ... */
    return (PyObject *)z;
}
```

## 15.3 PyFloatObject — Float Implementation

```c
/* Objects/floatobject.c */
typedef struct {
    PyObject_HEAD     /* ob_refcnt + ob_type */
    double ob_fval;   /* the C double value */
} PyFloatObject;

/* Float free list for recycling */
#define PyFloat_MAXFREELIST 256
static int numfree = 0;
static PyFloatObject *free_list = NULL;

/* Creating a float — checks free list first */
PyObject *
PyFloat_FromDouble(double fval) {
    PyFloatObject *op;
    if (free_list == NULL) {
        op = (PyFloatObject *) PyObject_Malloc(sizeof(PyFloatObject));
    } else {
        op = free_list;
        free_list = (PyFloatObject *) Py_TYPE(op);
        numfree--;
    }
    op->ob_fval = fval;
    return (PyObject *) op;
}
```

## 15.4 PyComplexObject — Complex Implementation

```c
/* Objects/complexobject.c */
typedef struct {
    double real;
    double imag;
} Py_complex;

typedef struct {
    PyObject_HEAD
    Py_complex cval;
} PyComplexObject;

/* Complex multiplication — uses the formula (ac-bd) + (ad+bc)j */
static Py_complex
c_prod(Py_complex a, Py_complex b) {
    Py_complex r;
    r.real = a.real*b.real - a.imag*b.imag;
    r.imag = a.real*b.imag + a.imag*b.real;
    return r;
}
```

## 15.5 How Python Dispatches Numeric Operations

When you write `a + b`, Python:

1. Calls `type(a).__add__(a, b)` (tries left operand's type first)
2. If that returns `NotImplemented`, calls `type(b).__radd__(b, a)` (right operand)
3. If both return `NotImplemented`, raises `TypeError`

```python
# You can see this with custom classes:
class MyNum:
    def __init__(self, val):
        self.val = val

    def __add__(self, other):
        print(f"MyNum.__add__ called with {other}")
        if isinstance(other, MyNum):
            return MyNum(self.val + other.val)
        return NotImplemented

    def __radd__(self, other):
        print(f"MyNum.__radd__ called with {other}")
        return MyNum(other + self.val)

n = MyNum(5)
print(n + MyNum(3))  # __add__ called
print(10 + n)        # __radd__ called (10 doesn't know MyNum)
```

## 15.6 The Type Object (`PyTypeObject`)

Each type is itself an object with method slots:

```c
/* Simplified PyTypeObject */
typedef struct _typeobject {
    PyObject_VAR_HEAD
    const char *tp_name;           /* Type name: "int", "float" */
    Py_ssize_t tp_basicsize;       /* Size of instances */
    ...
    /* Number protocol slots */
    PyNumberMethods *tp_as_number;
    /* Sequence protocol slots */
    PySequenceMethods *tp_as_sequence;
    ...
    reprfunc tp_repr;              /* repr() */
    hashfunc tp_hash;              /* hash() */
    ternaryfunc tp_call;           /* callable? */
    ...
} PyTypeObject;
```

The `PyNumberMethods` structure contains all arithmetic operations:

```c
typedef struct {
    binaryfunc nb_add;         /* + */
    binaryfunc nb_subtract;    /* - */
    binaryfunc nb_multiply;    /* * */
    binaryfunc nb_remainder;   /* % */
    binaryfunc nb_divmod;      /* divmod() */
    ternaryfunc nb_power;      /* ** */
    unaryfunc  nb_negative;    /* unary - */
    unaryfunc  nb_positive;    /* unary + */
    unaryfunc  nb_absolute;    /* abs() */
    inquiry    nb_bool;        /* bool() */
    ...
    binaryfunc nb_true_divide; /* / */
    binaryfunc nb_floor_divide; /* // */
    ...
} PyNumberMethods;
```

---

# Chapter 16: Common Bugs and Pitfalls

## Bug #1: Float Comparison with ==

```python
# Problem
0.1 + 0.2 == 0.3   # False!

# Why: Binary floating-point cannot represent 0.1 exactly
# Solution:
import math
math.isclose(0.1 + 0.2, 0.3)   # True
```

## Bug #2: Using `is` for Integer Comparison

```python
# Problem
x = 1000
y = 1000
if x is y:   # Unpredictable! May be True or False
    pass

# Solution: Always use == for value comparison
if x == y:   # Correct!
    pass
```

## Bug #3: `int()` Truncates, Doesn't Round

```python
# Problem
int(2.9)    # 2 — NOT 3!
int(-2.9)   # -2 — NOT -3!

# Solution: Use round() if rounding is needed
round(2.9)   # 3
round(-2.9)  # -3
```

## Bug #4: Floor Division with Negative Numbers

```python
# Problem
-7 // 2     # -4, not -3!

# Why: Floor division rounds toward -infinity
# -3.5 → floor → -4

# Solution: Understand the difference
import math
math.floor(-7/2)    # -4  (floor)
math.trunc(-7/2)    # -3  (truncate toward zero, same as C behavior)
int(-7/2)           # -3  (truncation)
```

## Bug #5: Mutable Default Argument

```python
# Problem
def append_item(item, lst=[]):  # BAD!
    lst.append(item)
    return lst

print(append_item(1))  # [1]
print(append_item(2))  # [1, 2] — list persists!

# Solution: Use None as default
def append_item(item, lst=None):  # GOOD
    if lst is None:
        lst = []
    lst.append(item)
    return lst
```

## Bug #6: `==` vs `is` for None Check

```python
# Problem
x = get_value()
if x == None:   # Anti-pattern! May call __eq__ on x

# Solution: Always use 'is' for None
if x is None:
    pass
if x is not None:
    pass
```

## Bug #7: `bool("False")` is `True`

```python
# Problem — common mistake from beginners
user_input = "False"
if bool(user_input):   # True! Non-empty string is truthy
    print("User said True")

# Solution: Parse boolean from string properly
def parse_bool(s):
    return s.lower() in ('true', '1', 'yes', 'on')

print(parse_bool("False"))  # False
print(parse_bool("true"))   # True
```

## Bug #8: Division Always Returns Float in Python 3

```python
# Problem (Python 2 habit)
result = 7 / 2    # In Python 2: 3; In Python 3: 3.5!

# Solution: Use explicit floor division for integer result
result = 7 // 2   # 3 (always)

# Or convert explicitly:
result = int(7 / 2)  # 3 (but goes through float first)
```

## Bug #9: Large Integer Multiplication Slowness

```python
# Problem: Not aware that very large int ops are slow
# Multiplying RSA-key-sized numbers is slow

a = 2**2048
b = 2**2048
result = a * b   # Takes measurable time

# Solution: Use appropriate types
# For modular arithmetic: use pow(base, exp, mod)
result = pow(2, 2048, 2**2048 + 1)   # Fast modular exponentiation!
```

## Bug #10: `Decimal` Initialized from Float

```python
# Problem
from decimal import Decimal
d = Decimal(0.1)   # Inherits float's imprecision!
print(d)  # 0.1000000000000000055511151231...

# Solution: Always use string
d = Decimal('0.1')  # Exact!
print(d)  # 0.1
```

## Bug #11: NaN Comparison

```python
# Problem
nan = float('nan')
if nan == nan:   # ALWAYS False! NaN ≠ NaN
    pass

# Solution
import math
math.isnan(nan)   # True — correct check
```

## Bug #12: Mixing Decimal and Float

```python
# Problem
from decimal import Decimal
result = Decimal('1.5') + 1.5   # TypeError!

# Solution: Convert explicitly
result = Decimal('1.5') + Decimal('1.5')
# OR
result = Decimal('1.5') + Decimal(str(1.5))
```

## Bug #13: Complex Numbers Have No Ordering

```python
# Problem
1+2j > 2+1j   # TypeError: '>' not supported for complex!

# Solution: Compare magnitudes if needed
abs(1+2j) > abs(2+1j)   # Compare magnitudes

# Or compare real parts separately:
a = 1+2j
b = 2+1j
if a.real > b.real:
    pass
```

## Bug #14: `True + True` Arithmetic Surprise

```python
# Problem — unexpected arithmetic
sum([True, True, False, True])   # 3 — some beginners surprised
True / True   # 1.0 — returns float!
True ** True  # 1

# Solution: If counting booleans, this is actually CORRECT behavior
# and very useful! Just understand that bool IS int
flags = [True, False, True, True]
count_true = sum(flags)   # 3 — idiomatic!
```

## Bug #15: Bitwise vs Logical Operators

```python
# Problem: & vs and, | vs or
x = 5
if x > 0 & x < 10:   # Bug! & has higher precedence than >
    pass
# Parsed as: x > (0 & x) < 10
# (0 & 5) = 0, then 5 > 0 < 10 → True (chain comparison)

# Solution:
if x > 0 and x < 10:    # Use 'and' for logical
    pass
if (x > 0) & (x < 10):  # Or use parentheses with &
    pass
```

## Bug #16: `round()` Banker's Rounding Surprise

```python
# Problem
round(0.5)   # 0  (not 1!)
round(1.5)   # 2
round(2.5)   # 2  (not 3!)
round(3.5)   # 4

# Python uses banker's rounding (round half to even)
# This surprises people expecting "round half up"

# Solution: Use explicit rounding mode if needed
from decimal import Decimal, ROUND_HALF_UP
def round_half_up(x, decimals=0):
    d = Decimal(str(x))
    factor = Decimal('0.' + '0' * decimals + '1') if decimals > 0 else Decimal('1')
    return float(d.quantize(factor, rounding=ROUND_HALF_UP))

print(round_half_up(0.5))   # 1.0
print(round_half_up(2.5))   # 3.0
```

## Bug #17: Infinity in Arithmetic

```python
# Problem: Unexpected infinity results
inf = float('inf')
print(inf + 1)     # inf   — OK
print(inf - inf)   # nan   — Surprise! Indeterminate form
print(inf * 0)     # nan   — Surprise! Indeterminate form
print(inf / inf)   # nan   — Surprise!
print(1 / inf)     # 0.0   — OK

# Solution: Check for infinity before arithmetic
import math
if not math.isinf(x) and not math.isinf(y):
    result = x - y
```

## Bug #18: Float to int Conversion at Range Boundaries

```python
# Problem: Float loses precision for large integers
x = 2**53 + 1    # int
y = float(x)      # float(2^53 + 1)
print(int(y) == x)  # False! float can't represent this exactly

# Why: float has 53 bits of mantissa, exactly 2^53 fits
# but 2^53 + 1 rounds to 2^53

print(int(float(2**53)))       # 9007199254740992  — correct
print(int(float(2**53 + 1)))   # 9007199254740992  — same! (lost the +1)

# Solution: Keep as int, avoid round-tripping through float
```

## Bug #19: `is` for bool vs `==`

```python
# Problem
x = 1
if x is True:    # False! x is int 1, not bool True
    print("True")

if x is False:   # False
    print("False")

# Solution: Use == or just truthiness
if x == True:    # True (1 == True)
if x:            # True (idiomatic)
```

## Bug #20: Operator Precedence with Unary Minus and Exponent

```python
# Problem
print(-2**2)    # -4, not 4!
# Parsed as: -(2**2) = -(4) = -4
# Exponentiation has HIGHER precedence than unary minus

# Solution:
print((-2)**2)   # 4 (correct if you mean to square -2)
print(-(2**2))   # -4 (correct if you mean to negate the square)
```

## Bug #21: Fraction Created from Float Surprise

```python
# Problem
from fractions import Fraction
f = Fraction(0.1)   # NOT 1/10!
print(f)    # 3602879701896397/36028797018963968

# Because float 0.1 is not exactly 0.1 in binary

# Solution: Use string
f = Fraction('0.1')  # 1/10 — exact!
# Or from Decimal:
from decimal import Decimal
f = Fraction(Decimal('0.1'))  # 1/10 — exact!
```

## Bug #22: Integer Division Result Type in Python 3

```python
# Problem (Python 2 to 3 migration)
def average(numbers):
    return sum(numbers) / len(numbers)  # Python 3: returns float

# In Python 2, sum([1,2,3]) / 3 == 2 (integer division)
# In Python 3, sum([1,2,3]) / 3 == 2.0 (float division)

# Python 2 behavior if needed:
def average_floor(numbers):
    return sum(numbers) // len(numbers)   # Integer result
```

## Bug #23: Comparing Different Numeric Types

```python
# This actually works and is well-defined in Python!
print(1 == 1.0)        # True
print(1 == True)       # True
print(0 == False)      # True
print(1 == Fraction(1))  # True
print(1 == Decimal('1')) # True

# But NOT all types compare with all:
print(1 == "1")         # False (no TypeError, just False)
print(1 == [1])         # False
```

## Bug #24: Using `sum()` With Wrong Starting Type

```python
# Problem
from decimal import Decimal
numbers = [Decimal('1.5'), Decimal('2.5')]
total = sum(numbers)    # sum() starts from 0 (int)
print(type(total))      # Decimal — actually works here!

# But this could be a problem:
total = sum(numbers, 0)    # Explicitly starting from int
print(type(total))          # Decimal — still works (Decimal + int)

# For complex numbers with sum():
complexes = [1+2j, 3+4j]
print(sum(complexes))         # (4+6j) — works!
print(sum(complexes, 0j))     # (4+6j) — explicit start
```

## Bug #25: Relying on repr() for Exact Float Recovery

```python
# Python 3 repr() is designed to roundtrip floats exactly
x = 3.141592653589793
print(repr(x))    # '3.141592653589793'
print(float(repr(x)) == x)   # True — repr roundtrips!

# But pre-Python 3.1, repr gave more digits unnecessarily
# Python 3.1+ uses shortest representation that roundtrips

# Important: str() also gives shortest representation in Python 3.1+
print(str(0.1))   # '0.1' (not the full imprecise value)
print(repr(0.1))  # '0.1' (same)
```

## Bug #26: Assuming Decimal Calculations Are Always Correct

```python
# Problem: Decimal can still be imprecise if precision is insufficient
from decimal import Decimal, getcontext

getcontext().prec = 5   # Low precision!
result = Decimal(1) / Decimal(3)
print(result)   # 0.33333  — only 5 digits!

# Multiple operations accumulate error:
result = result * 3
print(result)   # 0.99999  — not 1.0!

# Solution: Set adequate precision
getcontext().prec = 50
```

## Bug #27: None in Arithmetic

```python
# Problem
x = None
result = x + 1   # TypeError: unsupported operand type(s)

# Solution: Check before arithmetic
result = (x + 1) if x is not None else 0
# OR
result = (x or 0) + 1   # Using 'or' for default (careful with 0!)
```

## Bug #28: Assuming Complex Numbers Are Always Needed

```python
# Problem: Not knowing when complex math auto-appears
import math
# math.sqrt(-1) raises ValueError in Python!
# Must use cmath for complex math

# Solution:
import cmath
print(cmath.sqrt(-1))   # 1j
print((-1)**0.5)        # (6.123233995736766e-17+1j) — approximately 1j
```

## Bug #29: Integer Overflow When Working With Bytes/C Extensions

```python
# Python ints don't overflow, but C types do
# When interfacing with C libraries (ctypes, numpy):
import ctypes

# ctypes.c_int overflows like C:
overflow = ctypes.c_int(2**31)    # OverflowError
overflow = ctypes.c_int(2**31 - 1)  # OK: max int32

# numpy integers have fixed width
import numpy as np
x = np.int32(2**31 - 1)
print(x + 1)   # -2147483648 — overflows!

# Solution: Understand the type you're working with
# Python int: no overflow
# numpy/ctypes integers: overflow like C
```

## Bug #30: Confusing `//` and `%` Relationship

```python
# The relationship: a = (a // b) * b + (a % b)
# Python's // and % always satisfy this identity

a, b = -7, 3
q = a // b   # -3 (floor division)
r = a % b    # 2  (positive remainder — sign of b!)

print(q * b + r)   # -7 — verifies: -3*3 + 2 = -9+2 = -7 ✓

# C's % has sign of DIVIDEND:
# C: -7 % 3 = -1
# Python: -7 % 3 = 2

# Get C-style remainder in Python:
import math
c_remainder = math.remainder(-7, 3)   # -1.0 — C-style
c_remainder = -7 - int(-7/3)*3       # -1 — manual C-style
```

---
