# Python Function Fundamentals Master Guide

> **A Complete Reference from Beginner to Advanced — With Real-World Projects, Interview Questions, and Practice Exercises**

---

## Table of Contents

1. [Introduction to Functions](#1-introduction-to-functions)
2. [Function Definition — The `def` Statement](#2-function-definition--the-def-statement)
3. [Function Calls](#3-function-calls)
4. [Parameters vs Arguments](#4-parameters-vs-arguments)
5. [Positional Arguments](#5-positional-arguments)
6. [Keyword Arguments](#6-keyword-arguments)
7. [Default Parameters](#7-default-parameters)
8. [Return Values](#8-return-values)
9. [Multiple Return Values](#9-multiple-return-values)
10. [Variable Scope — Local and Global](#10-variable-scope--local-and-global)
11. [The LEGB Rule](#11-the-legb-rule)
12. [Function Documentation — Docstrings](#12-function-documentation--docstrings)
13. [Type Hints](#13-type-hints)
14. [Comparison Tables](#14-comparison-tables)
15. [Real-World Projects](#15-real-world-projects)
16. [Quick-Reference Cheat Sheets](#16-quick-reference-cheat-sheets)
17. [Interview Master Section — 30+ Questions](#17-interview-master-section--30-questions)
18. [Practice Exercises](#18-practice-exercises)
19. [Summary, Best Practices, and Learning Roadmap](#19-summary-best-practices-and-learning-roadmap)

---

## 1. Introduction to Functions

### 1.1 What Is a Function?

A **function** is a named, reusable block of code that performs a specific task. Instead of writing the same logic repeatedly, you define it once and call it wherever needed.

Think of a function like a vending machine:
- You press a button (call the function)
- Optionally insert coins (pass arguments)
- The machine does its job internally
- You receive a snack (return value)

### 1.2 Why Use Functions?

| Benefit | Description |
|---|---|
| **Reusability** | Write once, use many times |
| **Readability** | Named blocks explain intent |
| **Maintainability** | Fix one place, not everywhere |
| **Testability** | Isolate and test individual pieces |
| **Abstraction** | Hide complexity behind a name |

### 1.3 Functions in the Python Ecosystem

Python has three categories of functions:

- **Built-in functions** — `print()`, `len()`, `range()`, `type()`
- **User-defined functions** — functions you write with `def`
- **Lambda functions** — anonymous one-liner functions (advanced topic)

---

## 2. Function Definition — The `def` Statement

### 2.1 Introduction

The `def` keyword tells Python: *"I am about to define a function."* Everything indented under `def` is the function body.

### 2.2 Conceptual Understanding

When Python encounters a `def` statement, it:
1. Creates a function object in memory
2. Binds that object to the function name
3. Does **not** execute the body yet — only when called

### 2.3 Syntax

```python
def function_name(parameter1, parameter2, ...):
    """Optional docstring."""
    # function body
    return value  # optional
```

**Anatomy of a function signature:**

```
def   greet   (   name,   age   )   :
 ^      ^          ^       ^
keyword name    param1  param2
```

### 2.4 Beginner Examples

```python
# Simplest possible function — no parameters, no return
def say_hello():
    print("Hello, World!")

say_hello()
# Output: Hello, World!
```

```python
# Function with one parameter
def greet(name):
    print(f"Hello, {name}!")

greet("Suresh")
# Output: Hello, Suresh!
```

### 2.5 Intermediate Examples

```python
# Function that accepts and processes data
def square(number):
    result = number * number
    return result

print(square(5))   # Output: 25
print(square(12))  # Output: 144
```

```python
# Function with multiple parameters and logic
def describe_person(name, age, city):
    if age < 18:
        category = "minor"
    elif age < 60:
        category = "adult"
    else:
        category = "senior"
    print(f"{name} is a {category} from {city}.")

describe_person("Riya", 22, "Hyderabad")
# Output: Riya is an adult from Hyderabad.
```

### 2.6 Advanced Examples

```python
# Function that returns another function (higher-order function)
def make_multiplier(factor):
    def multiply(number):
        return number * factor
    return multiply

triple = make_multiplier(3)
print(triple(10))  # Output: 30
print(triple(7))   # Output: 21
```

### 2.7 Common Mistakes

```python
# ❌ Mistake 1: Forgetting the colon
def greet(name)          # SyntaxError: expected ':'
    print(name)

# ✅ Fix
def greet(name):
    print(name)

# ❌ Mistake 2: Wrong indentation
def greet(name):
print(name)              # IndentationError

# ✅ Fix
def greet(name):
    print(name)

# ❌ Mistake 3: Calling before defining
greet("Suresh")          # NameError: name 'greet' is not defined
def greet(name):
    print(name)

# ✅ Fix — always define before calling
def greet(name):
    print(name)
greet("Suresh")
```

### 2.8 Best Practices

- Use **lowercase_with_underscores** for function names (PEP 8)
- Keep functions short — ideally **one responsibility** per function
- Name functions with **verbs** that describe the action: `calculate_tax()`, `validate_email()`, `get_user()`
- Avoid generic names like `func1()`, `do_stuff()`

### 2.9 Performance Considerations

- Function calls have a small overhead due to frame creation
- For extremely tight loops, inline simple logic rather than calling a function millions of times
- In practice, this rarely matters — readability first

### 2.10 Interview Questions

**Q: What happens when Python encounters a `def` statement?**
A: Python creates a function object and binds it to the name in the current namespace. The body is not executed until the function is called.

**Q: Is `def` a statement or an expression in Python?**
A: `def` is a **statement**. It does not produce a value by itself (unlike a lambda expression).

### 2.11 Practice Exercise

Write a function `describe_triangle(a, b, c)` that takes three side lengths and prints whether the triangle is equilateral, isosceles, or scalene.

---

## 3. Function Calls

### 3.1 Introduction

A **function call** is the act of executing a function by using its name followed by parentheses. Without parentheses, you reference the function object — you don't run it.

### 3.2 Conceptual Understanding

```
greet("Suresh")
  ^       ^
name  arguments
```

When a function is called:
1. Python looks up the function name
2. Creates a new local scope (stack frame)
3. Binds arguments to parameters
4. Executes the body
5. Returns a value (or `None` if no `return`)
6. Destroys the local scope

### 3.3 Syntax

```python
# Basic call
function_name()

# Call with arguments
function_name(arg1, arg2)

# Call and capture return value
result = function_name(arg1, arg2)

# Call and ignore return value
function_name(arg1, arg2)  # return value discarded
```

### 3.4 Beginner Examples

```python
def add(a, b):
    return a + b

# Different ways to use the call
result = add(3, 4)
print(result)          # Output: 7

print(add(10, 20))     # Output: 30

total = add(add(1, 2), add(3, 4))  # nested calls
print(total)           # Output: 10
```

### 3.5 Intermediate Examples

```python
def power(base, exponent):
    return base ** exponent

# Using the return value in expressions
area = power(5, 2) * 3.14
print(f"Area: {area}")      # Output: Area: 78.5

# Passing return value as argument
print(power(power(2, 3), 2))  # 2^3=8, 8^2=64
# Output: 64
```

### 3.6 Advanced Examples

```python
# Functions stored in a list and called dynamically
def add(a, b):      return a + b
def subtract(a, b): return a - b
def multiply(a, b): return a * b

operations = [add, subtract, multiply]
for op in operations:
    print(f"{op.__name__}(10, 3) = {op(10, 3)}")

# Output:
# add(10, 3) = 13
# subtract(10, 3) = 7
# multiply(10, 3) = 30
```

### 3.7 Common Mistakes

```python
# ❌ Referencing instead of calling
def greet():
    print("Hello")

result = greet   # This stores the function object, NOT the result
print(result)    # Output: <function greet at 0x...>

# ✅ Fix
result = greet()  # Call it!
```

```python
# ❌ Wrong number of arguments
def add(a, b):
    return a + b

add(1)     # TypeError: add() missing 1 required positional argument: 'b'
add(1,2,3) # TypeError: add() takes 2 positional arguments but 3 were given
```

### 3.8 Best Practices

- Always capture the return value if you intend to use it
- Use descriptive variable names for captured results: `user_age = get_age(user_id)`
- Prefer flat function calls over deeply nested ones for readability

### 3.9 Interview Questions

**Q: What is the difference between `greet` and `greet()` in Python?**
A: `greet` is a reference to the function object. `greet()` calls the function and returns its result (or `None`).

---

## 4. Parameters vs Arguments

### 4.1 Introduction

These two terms are often used interchangeably but mean different things:
- **Parameter** — the variable listed in the function **definition**
- **Argument** — the actual value passed when **calling** the function

### 4.2 Conceptual Understanding

```python
def greet(name):        # 'name' is the PARAMETER (placeholder)
    print(f"Hi {name}")

greet("Suresh")         # "Suresh" is the ARGUMENT (actual value)
```

The parameter `name` only exists inside the function. The argument `"Suresh"` is what you provide at call time.

### 4.3 Comparison Table

| Feature | Parameter | Argument |
|---|---|---|
| **Where it appears** | Function definition | Function call |
| **What it is** | Variable (placeholder) | Actual value |
| **When it exists** | At definition time | At call time |
| **Example** | `def add(a, b)` | `add(3, 5)` |
| **Scope** | Local to the function | In caller's scope |
| **Mutability concern** | Defines type expectation | The real data passed |

### 4.4 Beginner Examples

```python
# Parameter: 'price' and 'tax_rate'
# Arguments: 1000 and 0.18
def calculate_tax(price, tax_rate):
    return price * tax_rate

tax = calculate_tax(1000, 0.18)  # 1000 and 0.18 are arguments
print(tax)  # Output: 180.0
```

### 4.5 Intermediate Examples

```python
def register_student(name, roll_no, branch):
    print(f"Registered: {name} | Roll: {roll_no} | Branch: {branch}")

# Arguments can be variables too
student_name = "Rahul"
roll = "21CS101"
dept = "CSE"

register_student(student_name, roll, dept)
# Output: Registered: Rahul | Roll: 21CS101 | Branch: CSE
```

### 4.6 Common Mistakes

```python
# ❌ Confusing the count
def full_name(first, last):
    return f"{first} {last}"

# Too many arguments
full_name("Ravi", "Kumar", "Singh")
# TypeError: full_name() takes 2 positional arguments but 3 were given
```

### 4.7 Practice Exercise

Define a function `bmi(weight_kg, height_m)` with two parameters. Call it with arguments of your choice and print the BMI value rounded to 2 decimal places.

---

## 5. Positional Arguments

### 5.1 Introduction

**Positional arguments** are matched to parameters purely by their **position** (order). The first argument goes to the first parameter, the second to the second, and so on.

### 5.2 Conceptual Understanding

```python
def describe(name, age, city):
    print(f"{name}, {age}, {city}")

describe("Priya", 21, "Bengaluru")
#          ^       ^       ^
#         name   age    city  ← matched by position
```

### 5.3 Syntax

```python
function_name(value1, value2, value3)
#              pos1    pos2    pos3
```

### 5.4 Beginner Examples

```python
def subtract(a, b):
    return a - b

print(subtract(10, 3))  # Output: 7  (10-3)
print(subtract(3, 10))  # Output: -7 (3-10) — ORDER MATTERS!
```

```python
def introduce(first_name, last_name, age):
    print(f"I am {first_name} {last_name}, {age} years old.")

introduce("Arjun", "Sharma", 20)
# Output: I am Arjun Sharma, 20 years old.
```

### 5.5 Intermediate Examples

```python
def format_date(day, month, year):
    return f"{day:02d}/{month:02d}/{year}"

print(format_date(5, 8, 2024))   # Output: 05/08/2024
print(format_date(12, 1, 2025))  # Output: 12/01/2025
```

### 5.6 Advanced Examples

```python
# Unpacking a list as positional arguments using *
def volume(length, width, height):
    return length * width * height

dimensions = [5, 3, 4]
print(volume(*dimensions))  # Output: 60
# Equivalent to: volume(5, 3, 4)
```

```python
# Collecting unlimited positional args with *args
def total(*numbers):
    return sum(numbers)

print(total(1, 2, 3))         # Output: 6
print(total(10, 20, 30, 40))  # Output: 100
print(total(5))               # Output: 5
```

### 5.7 Common Mistakes

```python
# ❌ Order matters — wrong order gives wrong result
def divide(numerator, denominator):
    return numerator / denominator

print(divide(10, 2))   # Output: 5.0  ✅
print(divide(2, 10))   # Output: 0.2  ❌ (if you meant 10/2)
```

### 5.8 Best Practices

- When a function has 3+ parameters, consider using keyword arguments for clarity
- Use `*args` when the number of arguments varies
- Keep positional parameter order **intuitive**: from most to least important

### 5.9 Performance Considerations

Positional argument lookup is slightly faster than keyword argument lookup because Python doesn't need to match names. The difference is negligible in real-world code.

### 5.10 Interview Questions

**Q: What does `*args` do in a function definition?**
A: It collects all extra positional arguments into a tuple. The function can then accept any number of positional arguments.

**Q: Can you use `*` to unpack a list as positional arguments?**
A: Yes. `func(*my_list)` unpacks the list and passes each element as a separate positional argument.

### 5.11 Practice Exercise

Write a function `average(*scores)` that accepts any number of scores and returns the average. Test it with 3 values and with 5 values.

---

## 6. Keyword Arguments

### 6.1 Introduction

**Keyword arguments** (also called named arguments) are passed by explicitly naming the parameter. This makes the order irrelevant — you're telling Python exactly which parameter gets which value.

### 6.2 Conceptual Understanding

```python
def register(name, age, city):
    print(f"{name}, {age}, {city}")

# Positional (order matters)
register("Anil", 25, "Chennai")

# Keyword (order doesn't matter)
register(city="Chennai", name="Anil", age=25)  # Same result!
```

### 6.3 Syntax

```python
function_name(param1=value1, param2=value2)
```

### 6.4 Beginner Examples

```python
def greet(name, language):
    if language == "Telugu":
        print(f"Namaskaram, {name}!")
    else:
        print(f"Hello, {name}!")

# Using keyword arguments — very readable
greet(name="Suresh", language="Telugu")
# Output: Namaskaram, Suresh!

greet(language="English", name="Alice")
# Output: Hello, Alice!
```

### 6.5 Intermediate Examples

```python
def create_profile(username, email, role="user", active=True):
    print(f"User: {username} | Email: {email} | Role: {role} | Active: {active}")

# Mix of positional and keyword
create_profile("suresh_k", "suresh@example.com", role="admin")
# Output: User: suresh_k | Email: suresh@example.com | Role: admin | Active: True

create_profile("alice", "alice@example.com", active=False)
# Output: User: alice | Email: alice@example.com | Role: user | Active: False
```

### 6.6 Advanced Examples

```python
# **kwargs — collect unlimited keyword arguments
def display_info(**details):
    for key, value in details.items():
        print(f"  {key}: {value}")

display_info(name="Ravi", age=22, city="Hyderabad", branch="CSE")
# Output:
#   name: Ravi
#   age: 22
#   city: Hyderabad
#   branch: CSE
```

```python
# Unpacking a dict as keyword arguments
def connect(host, port, timeout):
    print(f"Connecting to {host}:{port} (timeout={timeout}s)")

config = {"host": "localhost", "port": 5432, "timeout": 30}
connect(**config)
# Output: Connecting to localhost:5432 (timeout=30s)
```

### 6.7 Comparison Table

| Feature | Positional Arguments | Keyword Arguments |
|---|---|---|
| **Matched by** | Position (order) | Name |
| **Order matters?** | Yes | No |
| **Readability** | Lower for many params | Higher |
| **Error-prone?** | More (easy to swap) | Less |
| **Syntax** | `func(1, 2)` | `func(a=1, b=2)` |
| **Mixed usage** | Positional must come first | After positionals |
| **Collect extras** | `*args` → tuple | `**kwargs` → dict |

### 6.8 Common Mistakes

```python
# ❌ Positional argument after keyword argument
def add(a, b, c):
    return a + b + c

add(a=1, 2, 3)  # SyntaxError: positional argument follows keyword argument

# ✅ Fix — keyword args must come AFTER positional args
add(1, 2, c=3)  # Valid: Output: 6
```

```python
# ❌ Providing same argument twice
def greet(name):
    print(f"Hello, {name}")

greet("Suresh", name="Suresh")
# TypeError: greet() got multiple values for argument 'name'
```

### 6.9 Best Practices

- Use keyword arguments when calling functions with **3 or more parameters**
- Always put keyword arguments **after** positional arguments in the call
- Use `**kwargs` when you need to forward options to another function

### 6.10 Interview Questions

**Q: What is `**kwargs` and what type does it produce inside the function?**
A: `**kwargs` collects all extra keyword arguments into a **dictionary**. Inside the function, `kwargs` is a regular `dict`.

**Q: Can you mix `*args` and `**kwargs`?**
A: Yes. The signature order must be: `def func(pos_params, *args, keyword_params, **kwargs)`.

### 6.11 Practice Exercise

Write a function `build_query(**filters)` that builds a SQL-like query string from keyword arguments, e.g., `build_query(name="Ravi", age=22)` → `"WHERE name='Ravi' AND age='22'"`.

---

## 7. Default Parameters

### 7.1 Introduction

A **default parameter** has a pre-assigned value that is used when the caller does not supply that argument. This makes arguments optional.

### 7.2 Conceptual Understanding

```python
def greet(name, language="English"):  # language defaults to "English"
    ...

greet("Suresh")              # uses default: language = "English"
greet("Suresh", "Telugu")   # overrides default: language = "Telugu"
```

### 7.3 Syntax

```python
def function_name(required_param, optional_param=default_value):
    ...
```

**Rule:** Parameters with defaults must come **after** parameters without defaults.

### 7.4 Beginner Examples

```python
def power(base, exponent=2):
    return base ** exponent

print(power(5))     # Output: 25  (5^2, exponent defaults to 2)
print(power(5, 3))  # Output: 125 (5^3, exponent overridden)
print(power(2, 10)) # Output: 1024
```

```python
def create_border(text, char="*", width=40):
    border = char * width
    return f"{border}\n  {text}\n{border}"

print(create_border("Welcome"))
# Output:
# ****************************************
#   Welcome
# ****************************************

print(create_border("Alert", char="!", width=20))
# Output:
# !!!!!!!!!!!!!!!!!!!!
#   Alert
# !!!!!!!!!!!!!!!!!!!!
```

### 7.5 Intermediate Examples

```python
def connect_db(host="localhost", port=5432, db="mydb", timeout=30):
    print(f"Connecting to {db} at {host}:{port} (timeout={timeout}s)")

connect_db()
# Output: Connecting to mydb at localhost:5432 (timeout=30s)

connect_db(host="production.server.com", db="live_db")
# Output: Connecting to live_db at production.server.com:5432 (timeout=30s)
```

### 7.6 Advanced Examples — The Mutable Default Trap

```python
# ❌ DANGEROUS: Mutable default argument
def add_item(item, cart=[]):    # The same list [] is reused across calls!
    cart.append(item)
    return cart

print(add_item("Apple"))   # Output: ['Apple']
print(add_item("Banana"))  # Output: ['Apple', 'Banana']  ← Bug!
print(add_item("Cherry"))  # Output: ['Apple', 'Banana', 'Cherry']  ← Bug!

# ✅ CORRECT: Use None as default, create inside function
def add_item(item, cart=None):
    if cart is None:
        cart = []
    cart.append(item)
    return cart

print(add_item("Apple"))        # Output: ['Apple']
print(add_item("Banana"))       # Output: ['Banana']  ← Correct!
print(add_item("Cherry", ["X"])) # Output: ['X', 'Cherry']  ← Correct!
```

**Why does this happen?** Default values are evaluated **once** when the function is defined, not each time it is called. A mutable object (list, dict) created at definition time is shared across all calls.

### 7.7 Common Mistakes

```python
# ❌ Non-default before default
def connect(host="localhost", port):  # SyntaxError!
    ...

# ✅ Fix — required parameters first
def connect(port, host="localhost"):
    ...
```

### 7.8 Best Practices

- Never use mutable objects (`[]`, `{}`, `set()`) as default values
- Use `None` as default and initialize inside the function
- Defaults should represent the **most common use case**

### 7.9 Interview Questions

**Q: Why should you not use a list as a default parameter value?**
A: Default values are evaluated once at function definition. A mutable default like `[]` is shared across all calls. Each call that modifies it will see changes from previous calls — a common, hard-to-spot bug.

**Q: What is the idiomatic Python fix for a mutable default argument?**
A: Use `None` as default and create the mutable object inside the function body:
```python
def func(data=None):
    if data is None:
        data = []
```

### 7.10 Practice Exercises

Write a function `generate_greeting(name, prefix="Mr.", punctuation="!")` with sensible defaults, then call it in three different ways.

---

## 8. Return Values

### 8.1 Introduction

The `return` statement sends a value back to the caller and exits the function. Without `return`, Python implicitly returns `None`.

### 8.2 Conceptual Understanding

```
Caller        →   [Function]   →   Return value back to caller
add(3, 4)     →   3 + 4 = 7   →   7
```

The `return` statement:
1. Evaluates the expression
2. Sends the result to the caller
3. Immediately exits the function

### 8.3 Syntax

```python
def function_name():
    return expression      # return a value

def function_name():
    return                 # return None explicitly

def function_name():
    pass                   # implicitly returns None
```

### 8.4 Beginner Examples

```python
def add(a, b):
    return a + b

result = add(5, 3)
print(result)  # Output: 8
```

```python
def is_even(number):
    return number % 2 == 0   # Returns True or False

print(is_even(4))   # Output: True
print(is_even(7))   # Output: False
```

### 8.5 Intermediate Examples

```python
# Early return for guard clauses
def safe_divide(a, b):
    if b == 0:
        return None          # Early return
    return a / b

print(safe_divide(10, 2))  # Output: 5.0
print(safe_divide(10, 0))  # Output: None
```

```python
# Returning different types conditionally
def grade(score):
    if score >= 90: return "A"
    if score >= 80: return "B"
    if score >= 70: return "C"
    if score >= 60: return "D"
    return "F"

print(grade(95))  # Output: A
print(grade(73))  # Output: C
print(grade(45))  # Output: F
```

### 8.6 Advanced Examples

```python
# Using returned values in complex pipelines
def fetch_data():
    return [85, 92, 78, 61, 95, 88]

def filter_passing(scores, threshold=70):
    return [s for s in scores if s >= threshold]

def compute_average(scores):
    if not scores:
        return 0
    return sum(scores) / len(scores)

# Chain function calls
scores = fetch_data()
passing = filter_passing(scores)
avg = compute_average(passing)
print(f"Average of passing scores: {avg:.2f}")
# Output: Average of passing scores: 87.60
```

### 8.7 Common Mistakes

```python
# ❌ Code after return is unreachable
def greet(name):
    return f"Hello, {name}"
    print("This never runs!")  # Dead code — never executed

# ❌ Forgetting to use the return value
def double(x):
    return x * 2

double(5)             # Value 10 is discarded!
result = double(5)    # ✅ Capture it
```

```python
# ❌ Implicit None return confusion
def add(a, b):
    a + b              # No return! Returns None

result = add(3, 4)
print(result)          # Output: None  ← surprised?
```

### 8.8 Best Practices

- Every function should have a **clear, documented return type**
- Use early returns for guard clauses — avoid deeply nested `if` chains
- Be consistent: if a function sometimes returns a value, it should **always** return something (or always `None`)

### 8.9 Interview Questions

**Q: What does a Python function return if there is no `return` statement?**
A: It returns `None`.

**Q: Can a function have multiple `return` statements?**
A: Yes. Only the first one executed is used; the rest are never reached.

**Q: What is the difference between `return` and `print()`?**
A: `print()` displays output to the console and returns `None`. `return` sends a value back to the caller silently — the caller can use it in expressions.

---

## 9. Multiple Return Values

### 9.1 Introduction

Python functions can return **multiple values** by returning a tuple. The caller can unpack these values directly.

### 9.2 Syntax

```python
def function():
    return value1, value2, value3   # returns a tuple

# Unpack at call site
a, b, c = function()

# Or receive as tuple
result = function()
print(result[0])
```

### 9.3 Beginner Examples

```python
def min_max(numbers):
    return min(numbers), max(numbers)

lo, hi = min_max([3, 1, 7, 5, 2])
print(f"Min: {lo}, Max: {hi}")  # Output: Min: 1, Max: 7
```

### 9.4 Intermediate Examples

```python
def divide_with_remainder(a, b):
    quotient = a // b
    remainder = a % b
    return quotient, remainder

q, r = divide_with_remainder(17, 5)
print(f"17 ÷ 5 = {q} remainder {r}")
# Output: 17 ÷ 5 = 3 remainder 2
```

### 9.5 Advanced Examples

```python
def parse_name(full_name):
    parts = full_name.strip().split()
    first = parts[0]
    last = parts[-1] if len(parts) > 1 else ""
    middle = " ".join(parts[1:-1]) if len(parts) > 2 else ""
    return first, middle, last

first, middle, last = parse_name("Sai Charan Kumar")
print(f"First: {first}, Middle: {middle}, Last: {last}")
# Output: First: Sai, Middle: Charan, Last: Kumar
```

### 9.6 Best Practices

- Return a **named tuple** or **dataclass** instead of plain tuples for complex return values — avoids confusion about what each index means
- Use multiple returns sparingly — more than 3 values is a signal to use a data structure

```python
from collections import namedtuple

Stats = namedtuple("Stats", ["mean", "median", "mode"])

def compute_stats(data):
    data.sort()
    mean = sum(data) / len(data)
    median = data[len(data) // 2]
    mode = max(set(data), key=data.count)
    return Stats(mean, median, mode)

s = compute_stats([3, 1, 4, 1, 5, 9, 2, 6, 5, 5])
print(s.mean)    # Output: 4.1
print(s.mode)    # Output: 5
```

---

## 10. Variable Scope — Local and Global

### 10.1 Introduction

**Scope** determines where a variable can be seen and accessed. Python has a strict hierarchy of scopes, and a variable defined in one scope is not automatically visible in another.

### 10.2 Local Scope

A variable defined **inside a function** is **local** — it only exists within that function and is destroyed when the function exits.

```python
def calculate():
    result = 42      # local variable
    print(result)    # ✅ accessible here

calculate()          # Output: 42
print(result)        # ❌ NameError: name 'result' is not defined
```

### 10.3 Global Scope

A variable defined **at the module level** (outside any function) is **global** — accessible anywhere in the module.

```python
company = "TechCorp"    # global variable

def show_company():
    print(company)      # ✅ can READ global variable

show_company()          # Output: TechCorp
print(company)          # Output: TechCorp
```

### 10.4 Comparison Table

| Feature | Local Variable | Global Variable |
|---|---|---|
| **Where defined** | Inside a function | Outside all functions |
| **Scope** | Only within the function | Entire module |
| **Lifetime** | Function call → function return | Program start → program end |
| **Accessible from outside?** | No | Yes |
| **Memory efficiency** | Created/destroyed per call | Persists in memory |
| **Thread safety** | Safe (each call has own copy) | Risky (shared state) |
| **Best use** | Intermediate computations | Configuration, constants |

### 10.5 The `global` Keyword

To **modify** a global variable from inside a function, use the `global` keyword.

```python
counter = 0      # global

def increment():
    global counter
    counter += 1  # ✅ modifying global

increment()
increment()
increment()
print(counter)   # Output: 3
```

Without `global`:
```python
counter = 0

def increment():
    counter += 1   # ❌ UnboundLocalError!
    # Python sees 'counter' on the left side and assumes it's local
    # But it hasn't been assigned yet in local scope
```

### 10.6 Intermediate Examples

```python
# Global constant (acceptable use)
TAX_RATE = 0.18   # all-caps signals a constant

def compute_price(base_price):
    return base_price * (1 + TAX_RATE)

print(compute_price(1000))   # Output: 1180.0
print(compute_price(500))    # Output: 590.0
```

```python
# Tracking state with global (valid but use sparingly)
login_count = 0

def login(username):
    global login_count
    login_count += 1
    print(f"{username} logged in (total logins: {login_count})")

login("Suresh")  # Output: Suresh logged in (total logins: 1)
login("Priya")   # Output: Priya logged in (total logins: 2)
```

### 10.7 Common Mistakes

```python
# ❌ Shadowing a global variable unintentionally
name = "Global Suresh"

def greet():
    name = "Local Suresh"    # Creates a NEW local variable, doesn't change global
    print(name)

greet()          # Output: Local Suresh
print(name)      # Output: Global Suresh  (global unchanged)
```

### 10.8 Best Practices

- Avoid `global` whenever possible — it creates hidden dependencies
- Pass values as parameters and return results instead
- If global state is truly needed, encapsulate it in a class

```python
# ❌ Using global for shared state
count = 0
def increment(): global count; count += 1

# ✅ Better: pass and return
def increment(count):
    return count + 1

count = 0
count = increment(count)
count = increment(count)
print(count)  # Output: 2
```

### 10.9 Interview Questions

**Q: What is the difference between a local and global variable?**
A: A local variable is defined inside a function and is only accessible there. A global variable is defined at module level and accessible throughout the module. Local variables are created when a function starts and destroyed when it returns.

**Q: What happens if you try to modify a global variable inside a function without the `global` keyword?**
A: Python raises an `UnboundLocalError`. Python sees the assignment and assumes the variable is local, but then finds it's referenced before being assigned.

**Q: Is using `global` considered good practice?**
A: Generally no. `global` creates tight coupling and makes functions depend on external state, which harms testability and readability. Better alternatives: pass the variable as a parameter and return the modified value.

---

## 11. The LEGB Rule

### 11.1 Introduction

Python resolves variable names by searching four scopes in a fixed order. This is called the **LEGB Rule**.

### 11.2 The Four Scopes

```
L → Local       (inside the current function)
E → Enclosing   (enclosing function's scope — for nested functions)
G → Global      (module-level scope)
B → Built-in    (Python's built-in names: print, len, range, ...)
```

Python searches L → E → G → B. The first match wins.

### 11.3 Visual Diagram

```
┌─────────────────────────────────────────┐
│  Built-in Scope                         │
│  (print, len, range, int, str, ...)     │
│  ┌──────────────────────────────────┐   │
│  │  Global Scope                    │   │
│  │  (module-level variables)        │   │
│  │  ┌───────────────────────────┐   │   │
│  │  │  Enclosing Scope          │   │   │
│  │  │  (outer function, if any) │   │   │
│  │  │  ┌────────────────────┐   │   │   │
│  │  │  │  Local Scope       │   │   │   │
│  │  │  │  (this function)   │   │   │   │
│  │  │  └────────────────────┘   │   │   │
│  │  └───────────────────────────┘   │   │
│  └──────────────────────────────────┘   │
└─────────────────────────────────────────┘
```

### 11.4 Examples for Each Level

```python
# Built-in scope example
print(len("hello"))   # 'len' is found in Built-in scope
```

```python
# Global scope example
x = "I am global"     # Global

def show():
    print(x)          # Not local, not enclosing → found in Global

show()                # Output: I am global
```

```python
# Local scope example
def outer():
    x = "I am enclosing"  # Enclosing for inner()

    def inner():
        print(x)           # Not local → found in Enclosing

    inner()

outer()                    # Output: I am enclosing
```

```python
# Full LEGB in one example
x = "global"

def outer():
    x = "enclosing"

    def inner():
        x = "local"
        print(x)    # Output: local   (found in L)

    inner()
    print(x)        # Output: enclosing (found in E after inner's L)

outer()
print(x)            # Output: global  (module scope)
```

### 11.5 The `nonlocal` Keyword (Enclosing Scope)

To modify an **enclosing** scope variable from an inner function:

```python
def counter():
    count = 0              # Enclosing scope

    def increment():
        nonlocal count     # Access enclosing scope
        count += 1
        return count

    return increment

c = counter()
print(c())   # Output: 1
print(c())   # Output: 2
print(c())   # Output: 3
```

### 11.6 Interview Questions

**Q: Explain the LEGB rule.**
A: Python looks up variable names in this order: Local (current function), Enclosing (any outer function), Global (module level), Built-in (Python builtins). The first scope where the name is found wins.

**Q: What is `nonlocal` used for?**
A: `nonlocal` allows a nested (inner) function to modify a variable from its enclosing (outer) function's scope — without making it global.

---

## 12. Function Documentation — Docstrings

### 12.1 Introduction

A **docstring** is a string literal placed immediately after the `def` line. It documents what the function does, its parameters, and its return value. Python stores it in the `__doc__` attribute.

### 12.2 Syntax

```python
def function_name(params):
    """One-line summary of what the function does."""
    ...

def function_name(params):
    """
    Multi-line docstring.

    Longer description of the function's behavior.

    Args:
        param1: Description of param1.
        param2: Description of param2.

    Returns:
        Description of the return value.

    Raises:
        ValueError: When the input is invalid.
    """
    ...
```

### 12.3 Examples

```python
def calculate_cgpa(marks_list):
    """
    Calculate CGPA from a list of marks.

    Args:
        marks_list (list): List of marks (0–100 each).

    Returns:
        float: CGPA on a 10-point scale.

    Raises:
        ValueError: If marks_list is empty.

    Example:
        >>> calculate_cgpa([85, 90, 78, 92, 88])
        8.66
    """
    if not marks_list:
        raise ValueError("Marks list cannot be empty.")
    return round(sum(marks_list) / len(marks_list) / 10, 2)

print(calculate_cgpa([85, 90, 78, 92, 88]))
# Output: 8.66

# Access docstring
print(calculate_cgpa.__doc__)
help(calculate_cgpa)
```

### 12.4 Best Practices

- Write docstrings for **every** public function
- Use **Google style** or **NumPy style** consistently (Google style shown above)
- Include `Args`, `Returns`, and `Raises` sections when relevant
- Include a short `Example` to show usage

---

## 13. Type Hints

### 13.1 Introduction

**Type hints** (PEP 484) let you annotate parameters and return values with their expected types. Python does **not enforce** them at runtime — they are for documentation and static analysis tools like `mypy`.

### 13.2 Syntax

```python
def function_name(param1: type1, param2: type2) -> return_type:
    ...
```

### 13.3 Examples

```python
# Basic type hints
def add(a: int, b: int) -> int:
    return a + b

def greet(name: str) -> str:
    return f"Hello, {name}!"

def is_adult(age: int) -> bool:
    return age >= 18
```

```python
# Optional parameters
from typing import Optional, List

def find_user(user_id: int, default: Optional[str] = None) -> Optional[str]:
    users = {1: "Suresh", 2: "Priya"}
    return users.get(user_id, default)

print(find_user(1))         # Output: Suresh
print(find_user(99))        # Output: None
print(find_user(99, "N/A")) # Output: N/A
```

```python
# List and complex types
def top_students(scores: List[float], n: int = 3) -> List[float]:
    return sorted(scores, reverse=True)[:n]

print(top_students([78.5, 92.0, 85.5, 61.0, 88.0]))
# Output: [92.0, 88.0, 85.5]
```

```python
# Python 3.9+ simplified syntax (no need to import from typing)
def average(numbers: list[float]) -> float:
    return sum(numbers) / len(numbers)
```

### 13.4 Best Practices

- Add type hints to all public functions
- Use `Optional[T]` for parameters that could be `None`
- Use `-> None` explicitly when a function doesn't return a value
- Run `mypy` as part of your development workflow

---

## 14. Comparison Tables

### 14.1 Parameter vs Argument

| Feature | Parameter | Argument |
|---|---|---|
| **Definition** | Variable in function signature | Actual value at call site |
| **Location** | `def func(parameter):` | `func(argument)` |
| **When created** | At function definition | At function call |
| **Lifetime** | During function execution | In caller's scope |
| **Example** | `def greet(name):` | `greet("Suresh")` |

### 14.2 Local vs Global

| Feature | Local | Global |
|---|---|---|
| **Defined** | Inside a function | Outside all functions |
| **Visible in** | Only that function | Entire module |
| **Lifetime** | Per function call | Program lifetime |
| **To modify from function** | Direct assignment | `global` keyword needed |
| **Thread safe?** | Yes | No (shared state) |
| **Recommended for** | Computation, temp data | Constants, configuration |

### 14.3 Positional vs Keyword Arguments

| Feature | Positional | Keyword |
|---|---|---|
| **Matched by** | Order | Name |
| **Syntax** | `func(1, 2)` | `func(a=1, b=2)` |
| **Order matters?** | Yes | No |
| **Readability** | Lower | Higher |
| **Collect extras** | `*args` → tuple | `**kwargs` → dict |
| **Error-prone?** | Yes (easy to swap) | Less |
| **Must appear** | Before keyword args | After positional args |

### 14.4 Default vs Non-Default Parameters

| Feature | Default Parameter | Non-Default Parameter |
|---|---|---|
| **Required at call?** | No (optional) | Yes (required) |
| **Has predefined value?** | Yes | No |
| **Position in signature** | After non-defaults | Before defaults |
| **Override possible?** | Yes | N/A |
| **Mutable default risk?** | Yes — use `None` | No |

---

## 15. Real-World Projects

### Project 1: Calculator

```python
"""
Simple Calculator with all four operations.
Demonstrates: functions, parameters, return values, default args.
"""

def add(a: float, b: float) -> float:
    """Return the sum of a and b."""
    return a + b

def subtract(a: float, b: float) -> float:
    """Return a minus b."""
    return a - b

def multiply(a: float, b: float) -> float:
    """Return the product of a and b."""
    return a * b

def divide(a: float, b: float) -> float | None:
    """
    Return a divided by b.
    Returns None if b is zero (division undefined).
    """
    if b == 0:
        print("Error: Division by zero is undefined.")
        return None
    return a / b

def power(base: float, exponent: float = 2) -> float:
    """Return base raised to exponent. Default exponent is 2 (square)."""
    return base ** exponent

def calculate(a: float, operator: str, b: float) -> float | None:
    """
    Perform a calculation based on operator string.

    Args:
        a: First number.
        operator: One of '+', '-', '*', '/'.
        b: Second number.

    Returns:
        Result of the calculation or None.
    """
    operations = {
        "+": add,
        "-": subtract,
        "*": multiply,
        "/": divide,
    }
    if operator not in operations:
        print(f"Unknown operator: {operator}")
        return None
    return operations[operator](a, b)

# --- Demo ---
print("=== Calculator Demo ===")
print(calculate(10, "+", 5))    # Output: 15
print(calculate(10, "-", 3))    # Output: 7
print(calculate(10, "*", 4))    # Output: 40
print(calculate(10, "/", 0))    # Output: Error + None
print(calculate(10, "/", 4))    # Output: 2.5
print(power(5))                 # Output: 25 (default exponent=2)
print(power(2, 10))             # Output: 1024
```

---

### Project 2: Login Validator

```python
"""
Login Validator System.
Demonstrates: parameters, return values, scope, type hints, docstrings.
"""

# Global user database (in real apps, use a database)
USER_DATABASE = {
    "suresh_k": "pass@2024",
    "alice":    "secure#789",
    "admin":    "Admin!123",
}

MAX_ATTEMPTS = 3

def is_valid_username(username: str) -> bool:
    """
    Check if username meets requirements.
    Must be 3–20 chars, letters/numbers/underscores only.
    """
    if not (3 <= len(username) <= 20):
        return False
    return all(c.isalnum() or c == "_" for c in username)

def is_valid_password(password: str) -> bool:
    """
    Check if password meets requirements.
    Must be 8+ chars, with at least one digit and one special character.
    """
    if len(password) < 8:
        return False
    has_digit = any(c.isdigit() for c in password)
    has_special = any(c in "!@#$%^&*()" for c in password)
    return has_digit and has_special

def authenticate(username: str, password: str) -> bool:
    """
    Authenticate a user against the database.

    Returns:
        True if credentials match, False otherwise.
    """
    stored = USER_DATABASE.get(username)
    if stored is None:
        return False
    return stored == password

def login(username: str, password: str) -> dict:
    """
    Full login flow with validation and authentication.

    Returns:
        dict with keys 'success' (bool) and 'message' (str).
    """
    if not is_valid_username(username):
        return {"success": False, "message": "Invalid username format."}
    if not is_valid_password(password):
        return {"success": False, "message": "Password does not meet requirements."}
    if not authenticate(username, password):
        return {"success": False, "message": "Invalid username or password."}
    return {"success": True, "message": f"Welcome back, {username}!"}

# --- Demo ---
print("=== Login Validator Demo ===")

test_cases = [
    ("suresh_k", "pass@2024"),       # Valid
    ("alice",    "wrongpassword"),   # Wrong password (no digit/special)
    ("ab",       "pass@2024"),       # Username too short
    ("admin",    "Admin!123"),       # Valid admin
    ("ghost",    "pass@2024"),       # User not found
]

for user, pwd in test_cases:
    result = login(user, pwd)
    status = "✅" if result["success"] else "❌"
    print(f"{status} [{user}]: {result['message']}")

# Output:
# ✅ [suresh_k]: Welcome back, suresh_k!
# ❌ [alice]: Password does not meet requirements.
# ❌ [ab]: Invalid username format.
# ✅ [admin]: Welcome back, admin!
# ❌ [ghost]: Invalid username or password.
```

---

### Project 3: Student Grading System

```python
"""
Student Grading System.
Demonstrates: multiple return values, default params, scope, type hints.
"""
from typing import Optional

def letter_grade(score: float) -> str:
    """Convert numeric score to letter grade."""
    if score >= 90: return "A+"
    if score >= 80: return "A"
    if score >= 70: return "B"
    if score >= 60: return "C"
    if score >= 50: return "D"
    return "F"

def grade_point(score: float) -> float:
    """Convert numeric score to grade point (0–10 scale)."""
    if score >= 90: return 10.0
    if score >= 80: return 9.0
    if score >= 70: return 8.0
    if score >= 60: return 7.0
    if score >= 50: return 6.0
    return 0.0

def compute_stats(scores: list[float]) -> tuple[float, float, float, float]:
    """
    Compute statistics for a list of scores.

    Returns:
        Tuple of (average, highest, lowest, cgpa).
    """
    if not scores:
        return 0.0, 0.0, 0.0, 0.0
    avg = sum(scores) / len(scores)
    return avg, max(scores), min(scores), round(avg / 10, 2)

def evaluate_student(name: str, scores: list[float],
                     passing_score: float = 50.0) -> dict:
    """
    Full student evaluation report.

    Args:
        name: Student's name.
        scores: List of subject scores.
        passing_score: Minimum score to pass each subject.

    Returns:
        Dictionary with full evaluation.
    """
    avg, highest, lowest, cgpa = compute_stats(scores)
    grades = [letter_grade(s) for s in scores]
    failed_subjects = sum(1 for s in scores if s < passing_score)
    passed = failed_subjects == 0

    return {
        "name":            name,
        "scores":          scores,
        "grades":          grades,
        "average":         round(avg, 2),
        "highest":         highest,
        "lowest":          lowest,
        "cgpa":            cgpa,
        "overall_grade":   letter_grade(avg),
        "passed":          passed,
        "failed_subjects": failed_subjects,
        "status":          "PASS" if passed else "FAIL",
    }

def print_report(report: dict) -> None:
    """Print a formatted student report card."""
    print(f"\n{'='*45}")
    print(f"  STUDENT REPORT CARD")
    print(f"{'='*45}")
    print(f"  Name     : {report['name']}")
    print(f"  Scores   : {report['scores']}")
    print(f"  Grades   : {report['grades']}")
    print(f"  Average  : {report['average']}")
    print(f"  CGPA     : {report['cgpa']} / 10.0")
    print(f"  Highest  : {report['highest']}")
    print(f"  Lowest   : {report['lowest']}")
    print(f"  Overall  : {report['overall_grade']}")
    print(f"  Status   : {report['status']}")
    if report['failed_subjects'] > 0:
        print(f"  ⚠ Failed : {report['failed_subjects']} subject(s)")
    print(f"{'='*45}")

# --- Demo ---
students = [
    ("Suresh Kumar", [85, 92, 78, 88, 91]),
    ("Priya Reddy",  [72, 68, 55, 80, 74]),
    ("Rahul Singh",  [45, 52, 38, 61, 49]),
]

for name, scores in students:
    report = evaluate_student(name, scores)
    print_report(report)
```

**Output:**
```
=============================================
  STUDENT REPORT CARD
=============================================
  Name     : Suresh Kumar
  Scores   : [85, 92, 78, 88, 91]
  Grades   : ['A', 'A+', 'B', 'A', 'A+']
  Average  : 86.8
  CGPA     : 8.68 / 10.0
  Highest  : 92
  Lowest   : 78
  Overall  : A
  Status   : PASS
=============================================
```

---

### Project 4: Banking Application

```python
"""
Banking Application — Account Management.
Demonstrates: global state (via dict), functions, return values, scope, defaults.
"""

# In-memory "database"
_accounts: dict = {}
_next_id: int = 1000

def _generate_id() -> str:
    """Generate the next account ID."""
    global _next_id
    account_id = f"ACC{_next_id:04d}"
    _next_id += 1
    return account_id

def create_account(name: str, initial_deposit: float = 0.0) -> dict:
    """
    Create a new bank account.

    Args:
        name: Account holder name.
        initial_deposit: Opening balance (default 0.0).

    Returns:
        Account info dict with 'account_id', 'name', 'balance'.
    """
    if initial_deposit < 0:
        return {"error": "Initial deposit cannot be negative."}

    account_id = _generate_id()
    _accounts[account_id] = {
        "account_id": account_id,
        "name":       name,
        "balance":    initial_deposit,
        "transactions": [],
    }
    return _accounts[account_id]

def deposit(account_id: str, amount: float) -> dict:
    """
    Deposit money into an account.

    Returns:
        Updated account info or error dict.
    """
    if account_id not in _accounts:
        return {"error": f"Account {account_id} not found."}
    if amount <= 0:
        return {"error": "Deposit amount must be positive."}

    _accounts[account_id]["balance"] += amount
    _accounts[account_id]["transactions"].append(("DEPOSIT", amount))
    return {"success": True, "balance": _accounts[account_id]["balance"]}

def withdraw(account_id: str, amount: float) -> dict:
    """
    Withdraw money from an account.

    Returns:
        Updated account info or error dict.
    """
    if account_id not in _accounts:
        return {"error": f"Account {account_id} not found."}
    if amount <= 0:
        return {"error": "Withdrawal amount must be positive."}
    if _accounts[account_id]["balance"] < amount:
        return {"error": "Insufficient funds."}

    _accounts[account_id]["balance"] -= amount
    _accounts[account_id]["transactions"].append(("WITHDRAW", amount))
    return {"success": True, "balance": _accounts[account_id]["balance"]}

def get_balance(account_id: str) -> float | None:
    """Return current balance or None if account not found."""
    account = _accounts.get(account_id)
    return account["balance"] if account else None

def transfer(from_id: str, to_id: str, amount: float) -> dict:
    """Transfer amount from one account to another."""
    result = withdraw(from_id, amount)
    if "error" in result:
        return result
    deposit(to_id, amount)
    return {"success": True, "message": f"₹{amount} transferred."}

def print_statement(account_id: str) -> None:
    """Print account statement."""
    if account_id not in _accounts:
        print("Account not found.")
        return
    acc = _accounts[account_id]
    print(f"\n--- Statement: {acc['name']} ({account_id}) ---")
    for t_type, amount in acc["transactions"]:
        sign = "+" if t_type == "DEPOSIT" else "-"
        print(f"  {t_type:<10} {sign}₹{amount:,.2f}")
    print(f"  {'BALANCE':<10}  ₹{acc['balance']:,.2f}")
    print("-" * 45)

# --- Demo ---
print("=== Banking Application Demo ===")

acc1 = create_account("Suresh Kumar", initial_deposit=5000.0)
acc2 = create_account("Priya Reddy", initial_deposit=2000.0)

print(f"Created: {acc1['account_id']} for {acc1['name']}, Balance: ₹{acc1['balance']}")
print(f"Created: {acc2['account_id']} for {acc2['name']}, Balance: ₹{acc2['balance']}")

deposit(acc1["account_id"], 3000.0)
withdraw(acc1["account_id"], 1500.0)
transfer(acc1["account_id"], acc2["account_id"], 1000.0)

print_statement(acc1["account_id"])
print_statement(acc2["account_id"])

# Negative tests
print(withdraw(acc2["account_id"], 99999.0))
# Output: {'error': 'Insufficient funds.'}
```

---

## 16. Quick-Reference Cheat Sheets

### Cheat Sheet 1: Function Syntax

```python
# ─── Minimal function ───────────────────────────────────────────
def func():
    pass

# ─── With parameters ────────────────────────────────────────────
def func(a, b):
    return a + b

# ─── With defaults ──────────────────────────────────────────────
def func(a, b=10):
    return a + b

# ─── With type hints ────────────────────────────────────────────
def func(a: int, b: int = 10) -> int:
    return a + b

# ─── Positional-only (Python 3.8+) ─────────────────────────────
def func(a, b, /, c):   # a, b must be positional; c can be keyword
    pass

# ─── Keyword-only ───────────────────────────────────────────────
def func(*, name, age):  # name and age MUST be keyword args
    pass

# ─── *args ──────────────────────────────────────────────────────
def func(*args):
    print(args)          # tuple of all positional arguments

# ─── **kwargs ───────────────────────────────────────────────────
def func(**kwargs):
    print(kwargs)        # dict of all keyword arguments

# ─── Full signature ─────────────────────────────────────────────
def func(pos1, pos2, /, normal, *args, kw_only, **kwargs):
    pass
```

### Cheat Sheet 2: Scope Quick Reference

```python
x = "global"

def outer():
    x = "enclosing"

    def inner():
        x = "local"         # L: Local
        print(x)            # → "local"
    inner()

    print(x)                # → "enclosing"

outer()
print(x)                    # → "global"

# ── global keyword ──────────────────────────────────────────────
count = 0
def increment():
    global count
    count += 1

# ── nonlocal keyword ────────────────────────────────────────────
def outer():
    n = 0
    def inner():
        nonlocal n
        n += 1
    inner()
    return n               # → 1
```

### Cheat Sheet 3: Argument Passing Patterns

```python
def func(a, b, c): ...

# ── Positional ──────────────────────────────────────────────────
func(1, 2, 3)

# ── Keyword ─────────────────────────────────────────────────────
func(c=3, a=1, b=2)

# ── Mixed (positional must come first) ──────────────────────────
func(1, c=3, b=2)

# ── Unpack list as positional ───────────────────────────────────
args = [1, 2, 3]
func(*args)

# ── Unpack dict as keyword ──────────────────────────────────────
kwargs = {"a": 1, "b": 2, "c": 3}
func(**kwargs)
```

### Cheat Sheet 4: Return Values

```python
def no_return():   pass               # returns None
def explicit():    return             # returns None
def single():      return 42          # returns int
def multiple():    return 1, "a", True # returns tuple (1, 'a', True)

# Unpack multiple returns
x, y, z = multiple()

# Named tuple for clarity
from collections import namedtuple
Result = namedtuple("Result", ["value", "label", "flag"])
def typed(): return Result(1, "a", True)
r = typed()
print(r.value, r.label, r.flag)
```

---

## 17. Interview Master Section — 30+ Questions

### Level 1: Beginner (1–10)

**Q1. What is a function in Python?**
A function is a named, reusable block of code that performs a specific task. Defined with `def`, it can accept inputs (parameters), execute logic, and return a value.

**Q2. What is the difference between a parameter and an argument?**
A parameter is the variable in the function definition. An argument is the actual value passed when calling the function. Example: `def greet(name):` — `name` is the parameter. `greet("Suresh")` — `"Suresh"` is the argument.

**Q3. What does a function return if there is no `return` statement?**
It returns `None`.

**Q4. Can you call a function before it is defined?**
No. Python executes scripts top-to-bottom. Calling a function before its `def` statement raises `NameError`. (Exception: in some execution contexts where `def` runs before the call due to program structure.)

**Q5. What is a default parameter?**
A parameter with a pre-assigned value used when the caller doesn't supply that argument. Example: `def greet(name, lang="English"):` — `lang` is optional.

**Q6. What is the output of this code?**
```python
def func():
    return 1
    return 2

print(func())
```
Output: `1`. The first `return` exits the function immediately. The second is never reached.

**Q7. What is the difference between `print()` and `return`?**
`print()` sends output to the console and returns `None`. `return` sends a value back to the caller silently — that value can be used in further expressions.

**Q8. How many values can a Python function return?**
A function can return any number of values — they are packed into a tuple automatically. `return 1, 2, 3` returns the tuple `(1, 2, 3)`.

**Q9. What is a local variable?**
A variable defined inside a function. It only exists during that function's execution and is not accessible outside.

**Q10. What is a global variable?**
A variable defined at module level, outside all functions. It is accessible from anywhere in the module.

---

### Level 2: Intermediate (11–20)

**Q11. What is the LEGB rule?**
Python's name resolution order: **L**ocal → **E**nclosing → **G**lobal → **B**uilt-in. Python searches these scopes in order and uses the first match.

**Q12. What happens if you try to modify a global variable inside a function without `global`?**
Python raises `UnboundLocalError` because it interprets any assignment inside a function as creating a new local variable, but then finds it's referenced before assignment.

**Q13. Why should you avoid mutable default arguments?**
Default values are evaluated once when the function is defined. A mutable default like `[]` is shared across all calls — mutations in one call affect subsequent calls, leading to hard-to-find bugs.

**Q14. What is `*args`? What type does it produce?**
`*args` collects extra positional arguments into a **tuple**. It allows a function to accept any number of positional arguments.

**Q15. What is `**kwargs`? What type does it produce?**
`**kwargs` collects extra keyword arguments into a **dictionary**. It allows a function to accept any number of named arguments.

**Q16. Can you have both `*args` and `**kwargs` in the same function?**
Yes. The signature order must be: regular params, `*args`, keyword-only params, `**kwargs`:
```python
def func(a, b, *args, keyword_only=True, **kwargs):
    pass
```

**Q17. What is `nonlocal`?**
`nonlocal` allows a nested function to modify a variable from its enclosing (outer) function's scope, without making it global.

**Q18. What is a docstring?**
A string literal immediately after the `def` line that documents the function. Stored in `__doc__`. Accessible via `help()` or `function.__doc__`.

**Q19. Are type hints enforced at runtime?**
No. Type hints are for documentation and static analysis tools (like `mypy`). Python itself does not validate them during execution.

**Q20. What is the difference between positional-only and keyword-only parameters?**
Positional-only parameters (before `/`) cannot be passed by name. Keyword-only parameters (after `*` or `*args`) must be passed by name.
```python
def func(pos_only, /, normal, *, kw_only):
    pass
```

---

### Level 3: Advanced (21–30+)

**Q21. Explain the concept of a first-class function.**
In Python, functions are first-class objects. They can be assigned to variables, passed as arguments, returned from functions, and stored in data structures.

**Q22. What is a higher-order function?**
A function that takes another function as an argument or returns a function. Examples: `map()`, `filter()`, `sorted()` with `key=`, or any function that accepts a callable.

**Q23. What is a closure?**
A closure is a nested function that remembers variables from its enclosing scope even after the outer function has finished executing.
```python
def make_counter():
    count = 0
    def increment():
        nonlocal count
        count += 1
        return count
    return increment   # closure

c = make_counter()
print(c())  # 1
print(c())  # 2
```

**Q24. What is the difference between `*args` in definition vs `*` in a call?**
In a definition, `*args` collects extra arguments into a tuple. In a call, `*iterable` unpacks the iterable into positional arguments.

**Q25. How do you enforce keyword-only arguments?**
Place them after `*` or `*args` in the signature:
```python
def connect(*, host, port):  # both must be keyword
    pass

connect(host="localhost", port=5432)  # ✅
connect("localhost", 5432)            # ❌ TypeError
```

**Q26. What is the `__name__` attribute of a function?**
A string attribute holding the function's defined name. Useful in decorators and debugging.
```python
def add(a, b): return a + b
print(add.__name__)   # Output: add
```

**Q27. What is `functools.wraps` and why is it used?**
When writing decorators, wrapping a function with `@functools.wraps(original)` preserves the original function's `__name__`, `__doc__`, and other metadata on the wrapper function.

**Q28. Can a Python function modify a mutable argument passed to it?**
Yes. Mutable objects (lists, dicts, sets) are passed by reference. Mutations inside the function affect the original object. Immutable objects (int, str, tuple) cannot be changed.

**Q29. What is the difference between `return None` and not having a `return`?**
Both produce `None` as the return value. Explicit `return None` or `return` is often used for clarity in guard clauses. There is no behavioral difference.

**Q30. Explain function annotations and their purpose.**
Function annotations (`def func(a: int) -> str:`) attach metadata to parameters and return values. They do not affect runtime behavior. They support documentation, IDE tooling, and static type checkers like `mypy`.

**Q31. What is tail recursion and does Python optimize it?**
Tail recursion is when a recursive call is the last operation in a function. Python does **not** optimize tail recursion — deep recursion raises `RecursionError`. The default recursion limit is 1000 (`sys.getrecursionlimit()`). Use iteration for deep recursive problems.

**Q32. How is argument passing in Python — by value or by reference?**
Python uses **pass-by-object-reference** (sometimes called "pass-by-assignment"). The function receives a reference to the same object. For immutable objects (int, str), you cannot change the original. For mutable objects (list, dict), you can mutate them in place.

---

## 18. Practice Exercises

### Beginner Exercises (15)

**Exercise B1.** Write a function `greet(name)` that prints "Hello, {name}! Welcome to Python."

**Exercise B2.** Write a function `add(a, b)` that returns the sum of two numbers.

**Exercise B3.** Write a function `area_rectangle(length, width)` that returns the area.

**Exercise B4.** Write a function `is_even(n)` that returns `True` if `n` is even.

**Exercise B5.** Write a function `celsius_to_fahrenheit(c)` that converts Celsius to Fahrenheit.

**Exercise B6.** Write a function `max_of_two(a, b)` that returns the larger number without using `max()`.

**Exercise B7.** Write a function `count_vowels(text)` that returns the number of vowels.

**Exercise B8.** Write a function `reverse_string(s)` that returns the reversed string.

**Exercise B9.** Write a function `is_palindrome(s)` that returns `True` if the string is a palindrome.

**Exercise B10.** Write a function `multiply_all(*numbers)` that returns the product of all arguments.

**Exercise B11.** Write a function `greet_with_title(name, title="Mr.")` using a default parameter.

**Exercise B12.** Write a function `full_name(first, last)` that returns the full name as a string.

**Exercise B13.** Write a function `circle_area(radius)` that returns the area. Use `math.pi`.

**Exercise B14.** Write a function `print_table(n)` that prints the multiplication table for `n` (1–10).

**Exercise B15.** Write a function `swap(a, b)` that returns both values swapped (use multiple return values).

---

**Solutions to Selected Beginner Exercises:**

```python
# B1
def greet(name: str) -> None:
    print(f"Hello, {name}! Welcome to Python.")

greet("Suresh")
# Output: Hello, Suresh! Welcome to Python.

# B5
def celsius_to_fahrenheit(c: float) -> float:
    return c * 9 / 5 + 32

print(celsius_to_fahrenheit(0))    # Output: 32.0
print(celsius_to_fahrenheit(100))  # Output: 212.0

# B9
def is_palindrome(s: str) -> bool:
    clean = s.lower().replace(" ", "")
    return clean == clean[::-1]

print(is_palindrome("racecar"))   # Output: True
print(is_palindrome("hello"))     # Output: False
print(is_palindrome("A man a plan a canal Panama"))  # Output: True

# B10
def multiply_all(*numbers: float) -> float:
    result = 1
    for n in numbers:
        result *= n
    return result

print(multiply_all(2, 3, 4))       # Output: 24
print(multiply_all(1, 5, 10, 2))   # Output: 100

# B15
def swap(a, b):
    return b, a

x, y = swap(10, 20)
print(x, y)  # Output: 20 10
```

---

### Intermediate Exercises (15)

**Exercise I1.** Write `factorial(n)` using recursion. Handle `n=0` as a base case.

**Exercise I2.** Write `fibonacci(n)` that returns the nth Fibonacci number.

**Exercise I3.** Write `flatten(nested_list)` that converts `[[1,2],[3,[4,5]]]` to `[1,2,3,4,5]`.

**Exercise I4.** Write `word_count(text)` that returns a dict mapping each word to its count.

**Exercise I5.** Write `safe_divide(a, b, default=0)` that returns `default` on division by zero.

**Exercise I6.** Write `validate_email(email)` that checks for `@` and `.` in the right positions.

**Exercise I7.** Write `apply_discount(price, discount_pct=10)` that returns the discounted price.

**Exercise I8.** Write a function `build_sentence(**words)` that assembles words with keys as roles.

**Exercise I9.** Write `clamp(value, minimum=0, maximum=100)` that clamps a value to a range.

**Exercise I10.** Write `profile(**data)` that accepts any keyword args and prints them as a profile card.

**Exercise I11.** Write `top_n(items, n=5)` that returns the top n items from a list.

**Exercise I12.** Write a function that uses `global` to maintain a call counter across calls.

**Exercise I13.** Write `format_currency(amount, symbol="₹", decimals=2)` for currency formatting.

**Exercise I14.** Write `find_duplicates(lst)` that returns a list of duplicate values.

**Exercise I15.** Write `matrix_multiply(a, b)` for 2x2 matrix multiplication.

---

**Solutions to Selected Intermediate Exercises:**

```python
# I1
def factorial(n: int) -> int:
    """Return n! (n factorial). n must be >= 0."""
    if n < 0:
        raise ValueError("Factorial undefined for negative numbers.")
    if n == 0:
        return 1
    return n * factorial(n - 1)

print(factorial(0))   # Output: 1
print(factorial(5))   # Output: 120
print(factorial(10))  # Output: 3628800

# I5
def safe_divide(a: float, b: float, default: float = 0) -> float:
    """Divide a by b, return default if b is zero."""
    if b == 0:
        return default
    return a / b

print(safe_divide(10, 2))          # Output: 5.0
print(safe_divide(10, 0))          # Output: 0 (default)
print(safe_divide(10, 0, default=-1))  # Output: -1

# I9
def clamp(value: float, minimum: float = 0, maximum: float = 100) -> float:
    """Clamp value to [minimum, maximum] range."""
    return max(minimum, min(maximum, value))

print(clamp(150))      # Output: 100
print(clamp(-5))       # Output: 0
print(clamp(75))       # Output: 75
print(clamp(55, 60, 90))  # Output: 60

# I13
def format_currency(amount: float, symbol: str = "₹", decimals: int = 2) -> str:
    """Format a float as a currency string."""
    return f"{symbol}{amount:,.{decimals}f}"

print(format_currency(1234567.89))          # Output: ₹1,234,567.89
print(format_currency(999.5, symbol="$"))   # Output: $999.50
print(format_currency(42, decimals=0))      # Output: ₹42
```

---

### Advanced Exercises (15)

**Exercise A1.** Write a `make_counter(start=0, step=1)` factory function using closures.

**Exercise A2.** Write a decorator `@timer` that prints how long a function takes to run.

**Exercise A3.** Write a decorator `@validate_positive` that raises `ValueError` if any argument is ≤ 0.

**Exercise A4.** Write `memoize(func)` that caches a function's results.

**Exercise A5.** Write `compose(*funcs)` that composes multiple functions: `compose(f, g)(x)` → `f(g(x))`.

**Exercise A6.** Write `partial_sum(lst, start=0)` using recursion with a default accumulator.

**Exercise A7.** Write a function that returns a function to check if a number is divisible by `n`.

**Exercise A8.** Write `flatten_dict(d, prefix="")` that flattens a nested dict with dot notation.

**Exercise A9.** Write a class-based counter equivalent of the closure counter from A1.

**Exercise A10.** Write `retry(func, times=3, exceptions=(Exception,))` that retries a failing function.

**Exercise A11.** Write a generator function `fibonacci_gen()` using `yield`.

**Exercise A12.** Implement `map()` and `filter()` from scratch using closures.

**Exercise A13.** Write `curry(func)` that curries a 2-argument function.

**Exercise A14.** Write a function `rate_limiter(func, max_calls, period)` using closures.

**Exercise A15.** Write a `pipeline(*funcs)` that applies a series of functions to data left-to-right.

---

**Solutions to Selected Advanced Exercises:**

```python
# A1 — Closure counter factory
def make_counter(start: int = 0, step: int = 1):
    """
    Return a counter function.
    Each call increments by step and returns the new count.
    """
    count = start

    def counter():
        nonlocal count
        count += step
        return count

    return counter

c1 = make_counter()          # starts at 0, step 1
c2 = make_counter(10, 5)     # starts at 10, step 5

print(c1(), c1(), c1())      # Output: 1 2 3
print(c2(), c2(), c2())      # Output: 15 20 25
```

```python
# A4 — Memoization
def memoize(func):
    """Cache function results to avoid recomputation."""
    cache = {}

    def wrapper(*args):
        if args not in cache:
            cache[args] = func(*args)
        return cache[args]

    wrapper.__name__ = func.__name__
    return wrapper

@memoize
def fibonacci(n: int) -> int:
    if n <= 1:
        return n
    return fibonacci(n - 1) + fibonacci(n - 2)

print(fibonacci(10))   # Output: 55
print(fibonacci(40))   # Output: 102334155 (fast with cache)
```

```python
# A5 — Function composition
def compose(*funcs):
    """
    Compose multiple functions right-to-left.
    compose(f, g, h)(x) == f(g(h(x)))
    """
    def composed(value):
        for func in reversed(funcs):
            value = func(value)
        return value
    return composed

double   = lambda x: x * 2
add_ten  = lambda x: x + 10
square   = lambda x: x ** 2

transform = compose(double, add_ten, square)
print(transform(3))   # square(3)=9, add_ten(9)=19, double(19)=38
# Output: 38
```

```python
# A13 — Currying
def curry(func):
    """
    Curry a 2-argument function into a chain of single-argument functions.
    curry(add)(3)(4) == add(3, 4)
    """
    def curried(a):
        def inner(b):
            return func(a, b)
        return inner
    return curried

@curry
def add(a, b):
    return a + b

add_five = add(5)        # returns a function waiting for b
print(add_five(3))       # Output: 8
print(add_five(10))      # Output: 15
print(add(2)(3))         # Output: 5
```

```python
# A15 — Pipeline
def pipeline(*funcs):
    """
    Apply a series of functions left-to-right.
    pipeline(f, g, h)(x) == h(g(f(x)))
    """
    def process(data):
        for func in funcs:
            data = func(data)
        return data
    return process

# Example: data cleaning pipeline
strip_spaces    = lambda s: s.strip()
to_lowercase    = lambda s: s.lower()
remove_dots     = lambda s: s.replace(".", "")

clean = pipeline(strip_spaces, to_lowercase, remove_dots)
print(clean("  Hello.World.  "))   # Output: helloworld
print(clean("  Python.3.12.  "))   # Output: python312
```

---

## 19. Summary, Best Practices, and Learning Roadmap

### Summary

| Concept | Key Takeaway |
|---|---|
| **Function definition** | `def name(params): body` |
| **Parameters** | Placeholders in definition |
| **Arguments** | Actual values at call |
| **Positional args** | Matched by order |
| **Keyword args** | Matched by name |
| **Default params** | Optional at call; avoid mutables |
| **Return values** | Send data back; `None` if absent |
| **Multiple returns** | Returns a tuple |
| **Local scope** | Variable lives inside function |
| **Global scope** | Variable lives at module level |
| **`global` keyword** | Needed to modify a global |
| **`nonlocal` keyword** | Needed to modify an enclosing var |
| **LEGB rule** | L → E → G → B search order |
| **Docstrings** | Document intent, args, returns |
| **Type hints** | Annotate types (not enforced) |

---

### Best Practices

1. **One function, one responsibility** — keep functions small and focused
2. **Use descriptive names** — `calculate_cgpa()` not `calc()`
3. **Never use mutable defaults** — use `None` and initialize inside
4. **Prefer parameters over globals** — pass values in, return them out
5. **Document every public function** — write docstrings
6. **Add type hints** — makes code self-documenting and enables static analysis
7. **Use early returns** — for guard clauses rather than deep nesting
8. **Consistent return types** — don't return `int` sometimes and `None` other times
9. **Limit function arguments** — more than 5 arguments is a design smell; consider a dict or dataclass
10. **Keep functions pure where possible** — same input → same output, no side effects

---

### Common Pitfalls

| Pitfall | Example | Fix |
|---|---|---|
| Mutable default argument | `def f(lst=[]):` | `def f(lst=None): if lst is None: lst = []` |
| Forgetting `return` | `def add(a,b): a+b` | `return a + b` |
| Modifying global without keyword | `count += 1` inside function | `global count` first |
| Positional after keyword | `f(a=1, 2)` | `f(1, b=2)` |
| Non-default before default | `def f(a=1, b):` | `def f(b, a=1):` |
| Code after `return` | Unreachable dead code | Remove or restructure |
| Calling without `()` | `result = greet` | `result = greet()` |
| Over-using `global` | Tight coupling, hard to test | Pass as parameter |

---

### Learning Roadmap

```
Phase 1 — Foundation (Week 1–2)
────────────────────────────────────────────
✅ def, parameters, arguments, return
✅ Positional vs keyword arguments
✅ Default parameters
✅ Local vs global scope
✅ Multiple return values

Phase 2 — Intermediate (Week 3–4)
────────────────────────────────────────────
✅ *args and **kwargs
✅ LEGB rule in depth
✅ global and nonlocal keywords
✅ Docstrings and type hints
✅ First-class functions

Phase 3 — Advanced (Week 5–6)
────────────────────────────────────────────
⬜ Closures and factory functions
⬜ Decorators (@decorator syntax)
⬜ functools module (partial, wraps, lru_cache)
⬜ Higher-order functions (map, filter, sorted)
⬜ Lambda functions
⬜ Generator functions (yield)

Phase 4 — Professional (Week 7–8)
────────────────────────────────────────────
⬜ Type hints with mypy
⬜ Function overloading patterns
⬜ Protocol and callable types
⬜ Async functions (async def, await)
⬜ Doctest and pytest for functions
⬜ Design patterns using functions
```

---

> **Tip for Interview Prep:** Practice coding the Real-World Projects from memory, then time yourself. Be able to explain every line of code, especially around scope and argument passing — these are the most common areas for interview questions.
>
> **Next Topics to Study:** After mastering functions, move to **Object-Oriented Programming (OOP)** — where functions become **methods** inside classes, and scope becomes **instance scope** and **class scope**.

---

*Guide Version 1.0 | Python 3.10+ | Covers PEP 8, PEP 484 (Type Hints), PEP 3107 (Annotations)*
