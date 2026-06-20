# Chapter 6: Complex Numbers (complex)

## 6.1 Definition

A **complex number** in Python represents a number with both a real and an imaginary part, written as `a + bj` where `a` is the real part and `b` is the imaginary part. Python uses `j` (not `i` as in mathematics) for the imaginary unit, following engineering notation.

```python
z1 = 2 + 3j        # real=2, imag=3
z2 = -1.5 + 0.5j   # real=-1.5, imag=0.5
z3 = 4j             # pure imaginary: 0 + 4j
z4 = complex(3, 4)  # 3 + 4j via constructor
z5 = 5 + 0j         # real number as complex: 5 + 0j
```

## 6.2 Mathematical Background

The imaginary unit `j` is defined as `j² = -1`. This extends the real number line to a 2D plane (the complex plane or Argand plane):

```
    Imaginary axis (Im)
         |
    3j   |    *  z = 2 + 3j
         |   /
    2j   |  /
         | /
    1j   |/
         +-------------- Real axis (Re)
         0    1    2    3
```

Key mathematical properties:
- **Magnitude** (modulus): |z| = √(a² + b²)
- **Argument** (angle): arg(z) = atan2(b, a)
- **Conjugate**: z̄ = a - bj
- **Polar form**: z = r(cos θ + j sin θ) = re^(jθ)

## 6.3 Internal Representation

```c
/* CPython Objects/complexobject.c */
typedef struct {
    double real;
    double imag;
} Py_complex;

typedef struct {
    PyObject_HEAD
    Py_complex cval;   /* two IEEE 754 doubles */
} PyComplexObject;
```

Memory: 40 bytes (8 + 8 header + 8 + 8 for real and imag + 8 padding)

```python
import sys
z = 2 + 3j
print(sys.getsizeof(z))  # 32 bytes
```

## 6.4 Creating Complex Numbers

```python
# Method 1: j-suffix literal
z = 3 + 4j
z = 1.5 - 2.5j
z = 0 + 1j    # Pure imaginary

# Method 2: complex() constructor
z = complex(3, 4)      # 3 + 4j
z = complex(3)         # 3 + 0j
z = complex(0, 1)      # 0 + 1j
z = complex("3+4j")    # From string (no spaces allowed in string!)
z = complex("j")       # 0 + 1j

# Method 3: Arithmetic
z = 3 + 4*1j   # Same as 3+4j

# Errors
try:
    complex("3 + 4j")   # ValueError — no spaces allowed!
    complex("3+4i")     # ValueError — 'i' not valid, must be 'j'
except ValueError as e:
    print(e)
```

## 6.5 Properties

```python
z = 3 + 4j

# Access real and imaginary parts
print(z.real)   # 3.0  (always float, even if set as int)
print(z.imag)   # 4.0  (always float)

# Conjugate
print(z.conjugate())  # (3-4j)

# Magnitude (absolute value)
import cmath
print(abs(z))         # 5.0  (sqrt(3² + 4²) = sqrt(25) = 5)

# Phase/argument (angle in radians)
print(cmath.phase(z)) # 0.9272952180016122 (atan2(4, 3))

# Polar form
print(cmath.polar(z)) # (5.0, 0.9272952180016122) — (magnitude, phase)

# From polar
magnitude, phase = 5.0, cmath.pi/4
z = cmath.rect(magnitude, phase)   # converts back to a+bj form
print(z)  # (3.5355339059327378+3.5355339059327373j)
```

## 6.6 Complex Arithmetic

```python
a = 2 + 3j
b = 1 - 1j

# Addition
print(a + b)    # (3+2j)
# (2+1) + (3-1)j = 3 + 2j

# Subtraction
print(a - b)    # (1+4j)
# (2-1) + (3-(-1))j = 1 + 4j

# Multiplication
print(a * b)    # (5+1j)
# (2+3j)(1-j) = 2 - 2j + 3j - 3j²
# = 2 + j - 3(-1) = 2 + j + 3 = 5 + j

# Division
print(a / b)    # (-0.5+2.5j)
# (2+3j)/(1-j) = (2+3j)(1+j)/((1-j)(1+j))
# = (2+2j+3j+3j²)/(1+1)
# = (2+5j-3)/2 = (-1+5j)/2 = -0.5+2.5j

# Exponentiation
print(a ** 2)   # (-5+12j)
# (2+3j)² = 4 + 12j + 9j² = 4 + 12j - 9 = -5 + 12j

# Note: no >, <, >=, <= for complex (no ordering defined!)
try:
    print(a > b)    # TypeError!
except TypeError as e:
    print(e)

# But == and != work:
print(2 + 3j == 2 + 3j)   # True
print(2 + 3j == 2 + 4j)   # False
```

## 6.7 The `cmath` Module

The `cmath` module provides complex-number versions of mathematical functions:

```python
import cmath
import math

z = 1 + 1j

# Square root (cmath handles negative numbers correctly)
print(cmath.sqrt(-1))    # 1j
print(cmath.sqrt(z))     # (1.0986841134678098+0.45508986056222733j)

# Exponential
print(cmath.exp(z))      # (1.4686939399158851+2.2873552871788423j)
# Euler's formula: e^(a+bj) = e^a (cos b + j sin b)

# Euler's formula — e^(jπ) + 1 = 0 (most beautiful equation)
result = cmath.exp(1j * cmath.pi)
print(result)   # (-1+1.2246467991473532e-16j) ≈ -1 + 0j

# Logarithm
print(cmath.log(z))      # (0.34657359027997264+0.7853981633974483j)

# Trigonometric functions for complex arguments
print(cmath.sin(z))      # (1.2984575814159773+0.6349639147847361j)
print(cmath.cos(z))      # (0.8337300251311491-0.9888977057628651j)
```

## 6.8 Applications

### 6.8.1 Signal Processing

```python
import cmath
import math

# Discrete Fourier Transform (DFT) using complex arithmetic
def dft(signal):
    N = len(signal)
    result = []
    for k in range(N):
        X_k = sum(signal[n] * cmath.exp(-2j * cmath.pi * k * n / N)
                  for n in range(N))
        result.append(X_k)
    return result

# Simple signal: pure 440 Hz sine wave (A4 note)
N = 8
signal = [math.sin(2 * math.pi * k / N) for k in range(N)]
spectrum = dft(signal)
print([abs(x) for x in spectrum])
```

### 6.8.2 Electrical Engineering — Impedance

```python
import cmath

# AC circuit analysis using complex impedance
frequency = 60   # Hz
omega = 2 * cmath.pi * frequency

# Resistor: purely real impedance
R = 100        # 100 ohms
Z_R = complex(R, 0)

# Inductor: purely imaginary impedance
L = 0.1        # 0.1 Henry
Z_L = complex(0, omega * L)   # jωL

# Capacitor: purely imaginary impedance
C = 0.00001    # 10 μF
Z_C = complex(0, -1 / (omega * C))   # 1/(jωC)

# Total series impedance
Z_total = Z_R + Z_L + Z_C
print(f"Impedance: {Z_total:.2f}")
print(f"Magnitude: {abs(Z_total):.2f} Ω")
print(f"Phase: {cmath.phase(Z_total) * 180 / cmath.pi:.2f}°")
```

### 6.8.3 2D Rotation Using Complex Multiplication

```python
import cmath
import math

# Complex multiplication rotates a point
# Point (x, y) as complex number x + yj
point = 1 + 0j    # Point at (1, 0)

# Rotate 90 degrees = multiply by j
rotated_90 = point * 1j
print(rotated_90)   # 1j — now at (0, 1)

# Rotate by arbitrary angle θ
def rotate(point, theta_degrees):
    theta = math.radians(theta_degrees)
    rotation = cmath.exp(1j * theta)
    return point * rotation

p = 1 + 0j
print(rotate(p, 45))   # (0.707+0.707j) — 45° rotation
print(rotate(p, 90))   # (6.12e-17+1j) ≈ (0+1j) — 90° rotation
print(rotate(p, 180))  # (-1+1.22e-16j) ≈ (-1+0j) — 180° rotation
```

---

# Chapter 7: Decimal

## 7.1 Why Decimal Exists

The `decimal` module exists because `float` cannot exactly represent most decimal fractions. When you type `0.1`, you get an approximation. For financial calculations, exact decimal arithmetic is essential:

```python
# The problem with float in finance
price = 19.99
quantity = 3
total = price * quantity
print(total)         # 59.96999999999999 — WRONG for invoicing!

# The solution: Decimal
from decimal import Decimal
price = Decimal('19.99')
quantity = 3
total = price * quantity
print(total)         # 59.97 — CORRECT!
```

## 7.2 The `decimal` Module

```python
from decimal import (
    Decimal,
    getcontext,
    setcontext,
    Context,
    ROUND_UP,
    ROUND_DOWN,
    ROUND_CEILING,
    ROUND_FLOOR,
    ROUND_HALF_UP,
    ROUND_HALF_DOWN,
    ROUND_HALF_EVEN,
    ROUND_05UP,
    InvalidOperation,
    DivisionByZero,
    Overflow,
    Underflow,
    Inexact
)
```

## 7.3 Creating Decimal Objects

```python
from decimal import Decimal

# From string — PREFERRED (exact representation)
d = Decimal('3.14')
d = Decimal('0.1')     # Exact 0.1, not float approximation!
d = Decimal('-19.99')
d = Decimal('1E+2')    # Scientific notation: 100
d = Decimal('Infinity')
d = Decimal('NaN')

# From integer — exact
d = Decimal(42)        # Decimal('42')
d = Decimal(-100)      # Decimal('-100')

# From float — CAUTION! Inherits float imprecision
d = Decimal(0.1)       # Decimal('0.1000000000000000055511151231...')
# This is usually NOT what you want!

# Correct way to convert float to Decimal
d = Decimal(str(0.1))  # Decimal('0.1') — uses string repr
d = Decimal('0.1')     # Even better — just use string literal

# From tuple (sign, digits, exponent)
d = Decimal((0, (1, 2, 3), -2))  # Decimal('1.23')
# sign=0 (positive), digits=(1,2,3), exponent=-2 → 1.23
```

## 7.4 Context — Precision and Rounding Control

The `decimal` module uses a **context** object that controls:
- **Precision**: Number of significant digits
- **Rounding mode**: How to round when result exceeds precision
- **Signals**: Error handling behavior

```python
from decimal import getcontext, setcontext, Context, Decimal

# View current context
print(getcontext())
# Context(prec=28, rounding=ROUND_HALF_EVEN, Emin=-999999, Emax=999999, ...)

# Change precision globally
getcontext().prec = 50

from decimal import Decimal
print(Decimal(1) / Decimal(3))
# 0.33333333333333333333333333333333333333333333333333 (50 digits)

# Use local context (doesn't affect global)
with localcontext() as ctx:
    ctx.prec = 10
    result = Decimal(1) / Decimal(3)
    print(result)  # 0.3333333333 (10 digits)

# Back to global precision
print(Decimal(1) / Decimal(3))  # 50 digits again

# Create a specific context
my_ctx = Context(prec=4, rounding=ROUND_HALF_UP)
setcontext(my_ctx)
```

## 7.5 Precision Control

```python
from decimal import Decimal, getcontext, localcontext

# Default precision is 28 significant digits
getcontext().prec = 28
print(Decimal(1) / Decimal(7))
# 0.1428571428571428571428571429

# High precision computation
getcontext().prec = 100
from decimal import Decimal
pi_approx = sum(
    Decimal((-1)**k) / Decimal(2*k + 1)
    for k in range(10000)
) * 4
print(pi_approx)   # Very accurate approximation of π
```

## 7.6 Rounding Modes

Python's `decimal` module supports 8 rounding modes from the IBM General Decimal Arithmetic standard:

| Mode | Constant | Description | 1.5 → | 2.5 → | -1.5 → |
|------|----------|-------------|-------|-------|--------|
| Round up | `ROUND_UP` | Away from zero | 2 | 3 | -2 |
| Round down | `ROUND_DOWN` | Toward zero (truncate) | 1 | 2 | -1 |
| Round ceiling | `ROUND_CEILING` | Toward +infinity | 2 | 3 | -1 |
| Round floor | `ROUND_FLOOR` | Toward -infinity | 1 | 2 | -2 |
| Round half up | `ROUND_HALF_UP` | Traditional rounding | 2 | 3 | -2 |
| Round half down | `ROUND_HALF_DOWN` | Half rounds down | 1 | 2 | -1 |
| Round half even | `ROUND_HALF_EVEN` | Banker's rounding (default) | 2 | 2 | -2 |
| Round 05 up | `ROUND_05UP` | Special IBM mode | 2 | 3 | -2 |

```python
from decimal import Decimal, ROUND_HALF_UP, ROUND_HALF_EVEN, ROUND_CEILING, ROUND_FLOOR

d = Decimal('2.675')

# quantize() — round to specific number of decimal places
print(d.quantize(Decimal('0.01'), rounding=ROUND_HALF_UP))   # 2.68
print(d.quantize(Decimal('0.01'), rounding=ROUND_HALF_EVEN)) # 2.68
print(d.quantize(Decimal('0.01'), rounding=ROUND_CEILING))   # 2.68
print(d.quantize(Decimal('0.01'), rounding=ROUND_FLOOR))     # 2.67

# Financial rounding — always use ROUND_HALF_UP
price = Decimal('19.995')
rounded = price.quantize(Decimal('0.01'), rounding=ROUND_HALF_UP)
print(rounded)  # 19.99 or 20.00 depending on context
```

## 7.7 Decimal Operations

```python
from decimal import Decimal

a = Decimal('10')
b = Decimal('3')

print(a + b)    # 13
print(a - b)    # 7
print(a * b)    # 30
print(a / b)    # 3.333333333333333333333333333 (28 digits)
print(a // b)   # 3   (floor division)
print(a % b)    # 1   (modulo)
print(a ** b)   # 1000

# Decimal-specific methods
d = Decimal('3.14159')
print(d.sqrt())          # 1.772004514666935040420547...
print(d.ln())            # Natural log
print(d.log10())         # Base-10 log
print(d.exp())           # e^d

# Comparison
print(Decimal('1.0') == Decimal('1.00'))   # True (same value)
print(Decimal('1.0').compare(Decimal('1.00')))  # Decimal('0')

# Normalize — remove trailing zeros
print(Decimal('1.000').normalize())   # Decimal('1')
print(Decimal('100').normalize())     # Decimal('1E+2')

# Sign operations
print(Decimal('-3.14').copy_abs())    # Decimal('3.14')
print(Decimal('3.14').copy_negate())  # Decimal('-3.14')
```

## 7.8 Decimal vs Float Performance

```python
import timeit

float_time = timeit.timeit(
    '3.14 + 2.71',
    number=1_000_000
)
decimal_time = timeit.timeit(
    'Decimal("3.14") + Decimal("2.71")',
    setup='from decimal import Decimal',
    number=1_000_000
)

print(f"Float:   {float_time:.4f}s")
print(f"Decimal: {decimal_time:.4f}s")
# Decimal is 50-100x SLOWER than float
# Only use Decimal when you need exact decimal arithmetic

# Memory:
import sys
print(sys.getsizeof(3.14))             # 24 bytes
print(sys.getsizeof(Decimal('3.14'))) # ~100 bytes (much larger)
```

## 7.9 Financial Applications

```python
from decimal import Decimal, ROUND_HALF_UP

def calculate_invoice(items):
    """
    items: list of (quantity, unit_price_str) tuples
    Returns: Decimal total with tax
    """
    subtotal = Decimal('0')
    for qty, price in items:
        line_total = Decimal(str(qty)) * Decimal(price)
        subtotal += line_total

    TAX_RATE = Decimal('0.18')  # 18% GST
    tax = (subtotal * TAX_RATE).quantize(
        Decimal('0.01'),
        rounding=ROUND_HALF_UP
    )
    total = subtotal + tax

    return {
        'subtotal': subtotal,
        'tax': tax,
        'total': total
    }

items = [
    (3, '19.99'),
    (1, '5.49'),
    (10, '2.99')
]
invoice = calculate_invoice(items)
print(f"Subtotal: ₹{invoice['subtotal']}")
print(f"GST:      ₹{invoice['tax']}")
print(f"Total:    ₹{invoice['total']}")
```

## 7.10 Best Practices

```python
# 1. Always initialize Decimal from STRING, not float
good = Decimal('0.1')          # Exact!
bad  = Decimal(0.1)            # Inherits float imprecision!

# 2. Set precision before computation
from decimal import getcontext
getcontext().prec = 50

# 3. Use localcontext() for temporary precision changes
from decimal import localcontext
with localcontext() as ctx:
    ctx.prec = 10
    result = Decimal('1') / Decimal('3')

# 4. Use quantize() for final monetary rounding
amount = Decimal('99.995')
final = amount.quantize(Decimal('0.01'), rounding=ROUND_HALF_UP)

# 5. Don't mix float and Decimal
try:
    Decimal('1.5') + 1.5   # TypeError!
except TypeError:
    pass
# Convert explicitly:
result = Decimal('1.5') + Decimal(str(1.5))
```

---

# Chapter 8: Fraction

## 8.1 Definition and Mathematical Background

A **Fraction** represents a rational number exactly as a ratio `numerator/denominator`. While `float` approximates `1/3` as `0.3333...` (losing precision), `Fraction(1, 3)` stores the exact rational value.

Rational numbers form the set ℚ: all numbers expressible as p/q where p and q are integers and q ≠ 0.

```python
from fractions import Fraction

# Exact representation
one_third = Fraction(1, 3)
print(one_third)           # 1/3
print(float(one_third))    # 0.3333333333333333 (approximation)

# But arithmetic stays exact!
print(one_third * 3)       # 1  — exactly 1!
print(one_third + Fraction(1, 6))  # 1/2  — exactly!
```

## 8.2 Creating Fractions

```python
from fractions import Fraction

# From two integers (numerator, denominator)
f = Fraction(3, 4)     # 3/4
f = Fraction(1, 3)     # 1/3
f = Fraction(7, 1)     # 7/1 = 7
f = Fraction(0, 5)     # 0
f = Fraction(-1, 2)    # -1/2

# Automatically reduces to lowest terms!
f = Fraction(4, 8)     # → 1/2 (GCD reduction)
f = Fraction(6, 4)     # → 3/2
f = Fraction(-4, -8)   # → 1/2 (normalizes sign)
f = Fraction(-4, 8)    # → -1/2 (sign on numerator)

# From integer
f = Fraction(5)        # 5/1

# From float — may be surprising!
f = Fraction(1.5)      # 3/2 (exact for 1.5)
f = Fraction(0.1)      # 3602879701896397/36028797018963968 (float's actual value!)
# 0.1 is not exactly representable in binary!

# From string — PREFERRED for non-terminating decimals
f = Fraction('0.1')    # 1/10 (exact!)
f = Fraction('3/4')    # 3/4
f = Fraction('1.5')    # 3/2
f = Fraction('-7/8')   # -7/8
f = Fraction('1e-3')   # 1/1000

# From Decimal
from decimal import Decimal
f = Fraction(Decimal('0.1'))   # 1/10 (exact!)
```

## 8.3 Internal Representation

```python
from fractions import Fraction

f = Fraction(3, 4)
print(f.numerator)    # 3
print(f.denominator)  # 4

# The denominator is always positive
f = Fraction(-3, -4)
print(f.numerator)    # 3  — negative signs cancel
print(f.denominator)  # 4

f = Fraction(3, -4)
print(f.numerator)    # -3  — sign on numerator
print(f.denominator)  # 4
```

## 8.4 GCD Reduction

Fractions are always stored in their **lowest terms** using the GCD (Greatest Common Divisor):

```python
import math
from fractions import Fraction

# Python automatically reduces using GCD
f = Fraction(12, 8)
print(f)    # 3/2 (GCD of 12 and 8 is 4: 12/4=3, 8/4=2)

# Manual GCD
gcd = math.gcd(12, 8)
print(gcd)  # 4

# How Fraction reduces internally:
def make_fraction(num, den):
    g = math.gcd(abs(num), abs(den))
    return num // g, den // g

print(make_fraction(12, 8))   # (3, 2)
print(make_fraction(-9, 6))   # (-3, 2)
```

## 8.5 Arithmetic Operations

```python
from fractions import Fraction

a = Fraction(1, 2)   # 1/2
b = Fraction(1, 3)   # 1/3

# Addition: a/b + c/d = (ad + bc) / bd
print(a + b)    # 5/6  (3/6 + 2/6 = 5/6)

# Subtraction
print(a - b)    # 1/6  (3/6 - 2/6 = 1/6)

# Multiplication: a/b × c/d = ac/bd
print(a * b)    # 1/6

# Division: a/b ÷ c/d = ad/bc
print(a / b)    # 3/2

# Exponentiation (integer exponent only for exact result)
print(a ** 2)   # 1/4
print(a ** -1)  # 2

# Mixed operations with int
print(a + 1)    # 3/2
print(a * 4)    # 2

# Mixed operations with float — result is float
print(a + 0.5)  # 1.0  (float)

# All operations maintain exact rational arithmetic
result = Fraction(1, 7) * 7
print(result)   # 1  — EXACTLY 1, no rounding!
print(float(Fraction(1, 7) * 7))  # 1.0

# Compare with float:
print(1/7 * 7)  # 0.9999999999999999  (floating point error!)
```

## 8.6 Comparison and Conversion

```python
from fractions import Fraction

f = Fraction(1, 2)

# Comparisons
print(f == 0.5)                    # True
print(f == Fraction(2, 4))         # True
print(f < Fraction(3, 4))          # True
print(f > 0)                       # True

# Convert to float
print(float(f))    # 0.5

# Convert to int (truncates)
print(int(Fraction(7, 2)))   # 3  (truncates, not rounds)

# limit_denominator() — best rational approximation
import math
pi_approx = Fraction(math.pi).limit_denominator(1000)
print(pi_approx)    # 355/113 (famous approximation of π!)
print(float(pi_approx))  # 3.1415929203539825

# Very useful for converting floats to "nice" fractions
print(Fraction(0.333333).limit_denominator(100))  # 1/3
print(Fraction(0.1).limit_denominator(10))        # 1/10
print(Fraction(3.14159).limit_denominator(100))   # 22/7
```

## 8.7 Applications

```python
from fractions import Fraction

# 1. Exact probability calculations
def probability_exactly(n, k, p):
    """P(X=k) for Binomial(n, p)"""
    from math import comb
    p = Fraction(p) if not isinstance(p, Fraction) else p
    return Fraction(comb(n, k)) * p**k * (1 - p)**(n - k)

# Fair coin: P(exactly 3 heads in 5 flips)
p = probability_exactly(5, 3, Fraction(1, 2))
print(p)         # 5/16
print(float(p))  # 0.3125

# 2. Music theory — exact frequency ratios
unison    = Fraction(1, 1)
octave    = Fraction(2, 1)
fifth     = Fraction(3, 2)
fourth    = Fraction(4, 3)
major_3rd = Fraction(5, 4)
print(f"Perfect 5th: {float(fifth):.4f}")   # 1.5000

# 3. Chemistry — stoichiometry
# 2 H₂ + O₂ → 2 H₂O
hydrogen = Fraction(4)   # 4 atoms
oxygen   = Fraction(2)   # 2 atoms
water    = Fraction(2)   # 2 molecules
ratio = hydrogen / oxygen
print(f"H:O ratio = {ratio}")  # 2 (2 hydrogen per oxygen)

# 4. Cooking — exact recipe scaling
class Recipe:
    def __init__(self, name, ingredients):
        self.name = name
        self.ingredients = {
            k: Fraction(v) for k, v in ingredients.items()
        }

    def scale(self, factor):
        return {k: v * Fraction(factor) for k, v in self.ingredients.items()}

recipe = Recipe("Cake", {"flour": "2", "sugar": "1", "butter": "1/2"})
scaled = recipe.scale("1/3")
for ingredient, amount in scaled.items():
    print(f"{ingredient}: {amount}")
```

---

# Chapter 9: Boolean Type (bool)

## 9.1 Definition

A **bool** in Python represents a logical truth value. There are exactly two boolean objects: `True` and `False`. The `bool` type is a **subclass of `int`**, which means booleans are integers with `True == 1` and `False == 0`.

```python
x = True
y = False

print(type(True))   # <class 'bool'>
print(type(False))  # <class 'bool'>
```

## 9.2 History

The `bool` type was added to Python in **version 2.3** (2003). Before that, Python used integers 1 and 0 for boolean values. The implementation decision to make `bool` a subclass of `int` was made for backward compatibility.

Key design decisions:
- `True` and `False` are keywords (not just names)
- `bool` inherits all integer operations
- Boolean literals are capitalized (unlike other languages: `true`/`false`)
- `True` and `False` are **singletons** — there's only one instance of each

## 9.3 Internal Relationship With int

```python
# bool IS a subclass of int
print(issubclass(bool, int))    # True
print(isinstance(True, int))    # True
print(isinstance(True, bool))   # True

# True == 1 and False == 0
print(True == 1)    # True
print(False == 0)   # True
print(True == 2)    # False
print(True is 1)    # False! — different objects (True is bool, not int)

# Arithmetic with booleans
print(True + True)   # 2
print(True + 1)      # 2
print(False + 1)     # 1
print(True * 5)      # 5
print(sum([True, True, False, True]))  # 3

# Counting True values in a list
flags = [True, False, True, True, False]
count = sum(flags)   # Elegant idiom!
print(count)         # 3
```

## 9.4 Internal Implementation (CPython)

```c
/* CPython Objects/boolobject.c */
/* True and False are singletons defined at startup */
PyObject _Py_FalseStruct = {
    _PyObject_EXTRA_INIT
    1, &PyBool_Type
};

PyObject _Py_TrueStruct = {
    _PyObject_EXTRA_INIT
    1, &PyBool_Type
};

/* PyBool_Type inherits from PyLong_Type */
PyTypeObject PyBool_Type = {
    PyVarObject_HEAD_INIT(&PyType_Type, 0)
    "bool",
    sizeof(struct _longobject),
    ...
    (reprfunc)bool_repr,
    ...
    &PyLong_Type,   /* tp_base: bool inherits from int */
};
```

Memory layout:
```python
import sys
print(sys.getsizeof(True))   # 28 bytes (same as small int)
print(sys.getsizeof(False))  # 28 bytes
```

## 9.5 Boolean Operations

### 9.5.1 Logical `and`

Returns the first falsy value, or the last value if all are truthy:

```python
# Logical and — NOT necessarily bool result!
print(True and True)     # True
print(True and False)    # False
print(False and True)    # False (short-circuits!)

# With non-bool operands:
print(1 and 2)           # 2    (both truthy, returns last)
print(0 and 2)           # 0    (0 is falsy, returns it)
print([] and "hello")    # []   (empty list is falsy)
print("hi" and "bye")    # 'bye' (both truthy, returns last)
```

### 9.5.2 Logical `or`

Returns the first truthy value, or the last value if all are falsy:

```python
print(True or False)    # True  (returns first truthy)
print(False or True)    # True
print(False or False)   # False

# With non-bool operands:
print(0 or 42)          # 42   (0 is falsy, returns 42)
print([] or [1, 2])     # [1, 2]
print("" or "default")  # 'default' (common pattern!)
print("hello" or "bye") # 'hello' (first truthy)

# Common pattern: provide default value
name = "" or "Anonymous"   # "Anonymous"
config = {} or {'debug': False}
```

### 9.5.3 Logical `not`

Always returns `True` or `False`:

```python
print(not True)     # False
print(not False)    # True
print(not 0)        # True
print(not 1)        # False
print(not "")       # True
print(not "hello")  # False
print(not [])       # True
print(not [1, 2])   # False
print(not None)     # True

# Double negation to convert to bool
print(bool(42))     # True
print(not not 42)   # True  — same effect
```

## 9.6 Truthy and Falsy Values

Every Python object has a boolean interpretation. The `bool()` function reveals it:

### Complete Falsy Values Table

| Value | Type | Why Falsy |
|-------|------|-----------|
| `False` | bool | Definition |
| `0` | int | Zero |
| `0.0` | float | Zero |
| `0j` | complex | Zero |
| `0.0` + `0j` | complex | Zero |
| `Decimal('0')` | Decimal | Zero |
| `Fraction(0, 1)` | Fraction | Zero |
| `""` | str | Empty |
| `b""` | bytes | Empty |
| `[]` | list | Empty |
| `()` | tuple | Empty |
| `{}` | dict | Empty |
| `set()` | set | Empty |
| `frozenset()` | frozenset | Empty |
| `range(0)` | range | Empty |
| `None` | NoneType | Null |

**Everything else is truthy**, including:
- Non-zero numbers: `1`, `-1`, `3.14`, `2+3j`
- Non-empty containers: `[0]`, `{"": ""}`, `(False,)` — yes, a tuple with only False is truthy!
- Non-empty strings: `"False"`, `"0"`, `" "` — a space is truthy!
- User-defined objects (unless they define `__bool__` or `__len__`)

```python
# Surprising truthy values
print(bool([False]))      # True!  — non-empty list
print(bool([0]))          # True!  — non-empty list
print(bool({"": ""}))    # True!  — non-empty dict
print(bool((False,)))    # True!  — non-empty tuple
print(bool("False"))     # True!  — non-empty string
print(bool("0"))         # True!  — non-empty string
print(bool(" "))         # True!  — space is non-empty!

# Surprising falsy values
print(bool([]))          # False — empty list
print(bool({}))          # False — empty dict
print(bool(""))          # False — empty string
print(bool(0.0))         # False — zero float
```

### Custom `__bool__` and `__len__`

```python
class Account:
    def __init__(self, balance):
        self.balance = balance

    def __bool__(self):
        return self.balance > 0   # Truthy if has balance

    # OR if __bool__ not defined, Python checks __len__:
    # def __len__(self):
    #     return self.balance

acc1 = Account(100)
acc2 = Account(0)
print(bool(acc1))   # True
print(bool(acc2))   # False

if acc1:
    print("Account has funds")   # Prints!
if not acc2:
    print("Account is empty")    # Prints!
```

## 9.7 Short-Circuit Evaluation

Python's `and` and `or` use **short-circuit evaluation** — they don't evaluate the second operand if the first is sufficient:

```python
# AND short-circuits on first False
def expensive_check():
    print("Computing...")
    return True

result = False and expensive_check()  # expensive_check() NOT called!
print(result)   # False

# OR short-circuits on first True
result = True or expensive_check()    # expensive_check() NOT called!
print(result)   # True

# Practical use cases:
# 1. Null check before attribute access
user = None
name = user and user.name    # Avoids AttributeError!

# 2. Default values
config = get_config() or {}   # Default to empty dict

# 3. Conditional computation
def divide(a, b):
    return b != 0 and a / b   # Safe division without if statement

# 4. Chained validation
def valid_age(age):
    return (isinstance(age, int) and
            age >= 0 and
            age <= 150)

# 5. Guard against exception
import os
path = "/some/path"
content = os.path.exists(path) and open(path).read()
```

## 9.8 Boolean Conversion

```python
# bool() constructor
print(bool(0))          # False
print(bool(1))          # True
print(bool(-1))         # True  (any non-zero)
print(bool(""))         # False
print(bool("hello"))    # True
print(bool([]))         # False
print(bool([0]))        # True  (non-empty list!)
print(bool(None))       # False

# Comparison operators always return bool
print(type(1 < 2))      # <class 'bool'>
print(type(1 == 1))     # <class 'bool'>
```

## 9.9 Common Mistakes

```python
# Mistake 1: Comparing to True/False explicitly (anti-pattern)
flag = True
if flag == True:     # WRONG — verbose and fragile
    pass
if flag:             # CORRECT — Pythonic
    pass

# Mistake 2: Assuming bool values in arithmetic are unexpected
print(True + True + True)   # 3 — may surprise beginners
print(True > False)          # True (1 > 0)

# Mistake 3: not checking truthiness correctly
items = []
if items == False:   # WRONG — [] != False (they're different objects)
    pass
if not items:        # CORRECT — checks falsy
    pass

# Mistake 4: Confusing 'is' for bool check
x = 1
if x is True:       # False! x is int 1, not bool True
    pass
if x:               # CORRECT
    pass

# Mistake 5: Returning int from __bool__
class MyClass:
    def __bool__(self):
        return 1    # Should return bool, not int!
        # Will work due to int/bool relationship but bad practice

# Correct:
class MyClass:
    def __bool__(self):
        return True  # or: return bool(self.condition)
```

## 9.10 Interview Questions — Boolean

1. What is the output of `True + True + True`? Why?
2. Is `bool` a subclass of `int` in Python?
3. What's the difference between `is` and `==` when checking boolean values?
4. What does `and` return when both operands are non-bool objects?
5. Name five falsy values in Python other than `False`.
6. Why is `bool("False")` equal to `True`?
7. What is short-circuit evaluation? Give a practical example.
8. What method must a custom class define to control its boolean value?

---

# Chapter 10: NoneType

## 10.1 Definition

`None` is Python's null value — it represents the **absence of a value** or a **null reference**. `None` is the sole instance of the `NoneType` class. Unlike `null` in many other languages, `None` in Python is a first-class object with a type.

```python
x = None
print(x)            # None
print(type(x))      # <class 'NoneType'>
print(type(None))   # <class 'NoneType'>
```

## 10.2 Why None Exists

`None` serves several critical purposes:

1. **Default return value** — functions that don't explicitly return a value return `None`
2. **Uninitialized state** — represents "not yet set" or "missing"
3. **Optional parameters** — signals "no argument provided"
4. **Null object pattern** — explicit representation of absence

```python
# Functions without explicit return
def greet(name):
    print(f"Hello, {name}!")

result = greet("Suresh")
print(result)    # None — implicit return

# Void-like behavior
def configure():
    # Does work but returns nothing meaningful
    pass

config = configure()
print(config)    # None
```

## 10.3 Internal Representation — Singleton Pattern

`None` is implemented as a **singleton** in CPython — there is exactly ONE `None` object in memory, and all references to `None` point to this same object:

```c
/* CPython Objects/object.c */
PyObject _Py_NoneStruct = {
    _PyObject_EXTRA_INIT
    1, &_PyNone_Type
};

/* The None object itself */
#define Py_None (&_Py_NoneStruct)
```

```python
# All None references point to same object
a = None
b = None
c = None

print(id(a) == id(b) == id(c))  # True — same object!
print(a is b is c)               # True — identity check

# This is why 'is None' is the correct check
print(a is None)     # True — correct
print(a == None)     # True — but fragile (see below)

import sys
print(sys.getsizeof(None))  # 16 bytes — smallest object!
```

## 10.4 Checking for None

**Always use `is` (not `==`) to check for `None`:**

```python
x = None

# CORRECT — identity check
if x is None:
    print("x is None")

if x is not None:
    print("x has a value")

# WRONG — value check (may trigger __eq__ on custom objects)
if x == None:     # Anti-pattern!
    print("x is None")

# Why is 'is' preferred?
class Tricky:
    def __eq__(self, other):
        return True   # Always says "equal"!

t = Tricky()
print(t == None)    # True! (but t is NOT None!)
print(t is None)    # False (correct!)
```

### PEP 8 Recommendation

> "Comparisons to singletons like `None` should always be done with `is` or `is not`, never the equality operators."

```python
# PEP 8 compliant:
if value is None:
    handle_none()

if value is not None:
    process(value)

# The 'if not x' vs 'if x is not None' distinction:
x = 0
if not x:         # True for 0, "", [], None, False...
    pass
if x is not None: # True for 0, "", [], False — NOT None!
    pass          # Important distinction!
```

## 10.5 Common Use Cases

### 10.5.1 Default Parameter Values

```python
# WRONG — mutable default (famous Python gotcha)
def append_to(element, to=[]):   # BAD!
    to.append(element)
    return to

print(append_to(1))  # [1]
print(append_to(2))  # [1, 2] — reuses same list!

# CORRECT — use None as sentinel
def append_to(element, to=None):   # GOOD
    if to is None:
        to = []
    to.append(element)
    return to

print(append_to(1))   # [1]
print(append_to(2))   # [2] — fresh list each time
```

### 10.5.2 Representing Missing Values

```python
class Student:
    def __init__(self, name, grade=None):
        self.name = name
        self.grade = grade   # None means grade not yet assigned

    def has_grade(self):
        return self.grade is not None

    def display(self):
        grade_str = str(self.grade) if self.grade is not None else "Not Graded"
        print(f"{self.name}: {grade_str}")

s1 = Student("Suresh", 85)
s2 = Student("Priya")

s1.display()  # Suresh: 85
s2.display()  # Priya: Not Graded
```

### 10.5.3 Function Return Values

```python
# Function returns None to signal "not found"
def find_user(user_id, database):
    for user in database:
        if user['id'] == user_id:
            return user
    return None   # Explicit: user not found

db = [{'id': 1, 'name': 'Suresh'}, {'id': 2, 'name': 'Priya'}]

user = find_user(1, db)
if user is not None:
    print(f"Found: {user['name']}")
else:
    print("User not found")

# Pattern: None check before use
result = find_user(99, db)
name = result['name'] if result is not None else "Unknown"
print(name)   # Unknown
```

### 10.5.4 Optional Type Hints

```python
from typing import Optional

def process(value: Optional[int] = None) -> Optional[str]:
    """
    Optional[int] means the argument can be int or None
    Same as: Union[int, None]
    """
    if value is None:
        return None
    return str(value * 2)

print(process(5))     # "10"
print(process())      # None
print(process(None))  # None
```

## 10.6 None in Containers

```python
# None is a valid value in any container
lst = [1, None, 3, None, 5]
print(None in lst)         # True
print(lst.count(None))     # 2

dct = {'a': 1, 'b': None, 'c': 3}
print(dct['b'])            # None
print(dct.get('x'))        # None — default return for missing keys

# Filtering None values
clean = [x for x in lst if x is not None]
print(clean)   # [1, 3, 5]

# Or using filter()
clean = list(filter(None, lst))   # Removes ALL falsy, not just None!
# [1, 3, 5] — works here since 0 not in list, but caution!
```

## 10.7 Anti-patterns

```python
# Anti-pattern 1: Returning None explicitly when unnecessary
def get_value(x):
    if x > 0:
        return x
    return None   # Redundant — functions return None implicitly

# Better:
def get_value(x):
    if x > 0:
        return x
    # Implicit None return

# Anti-pattern 2: Using None as a general-purpose sentinel
# when other options exist
class Cache:
    _MISSING = object()   # Better sentinel than None

    def get(self, key, default=_MISSING):
        value = self._data.get(key, self._MISSING)
        if value is self._MISSING:
            if default is self._MISSING:
                raise KeyError(key)
            return default
        return value   # Can return None as valid cached value!

# Anti-pattern 3: Chaining on potentially-None values
user = get_user(id)
# WRONG — AttributeError if user is None
print(user.name.upper())

# CORRECT — check first
if user is not None:
    print(user.name.upper())

# Or use walrus operator (Python 3.8+):
if (user := get_user(id)) is not None:
    print(user.name.upper())
```

## 10.8 Interview Questions — NoneType

1. Why is `None is None` always `True`?
2. What's the difference between `if x is None` and `if not x`?
3. What does a function return if it has no `return` statement?
4. Why is it an anti-pattern to use mutable defaults like `def f(lst=[]):`?
5. Can `None` be used as a dictionary key? What would that mean?
6. What is `Optional[int]` equivalent to in Python type hints?

```python
# Answer to Q5:
d = {None: "null_value", 1: "one"}
print(d[None])    # "null_value" — None IS hashable and can be a key!
print(hash(None)) # Some integer
```

---
