<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" style="height:64px;margin-right:32px"/>

# \# ROLE

You are a Python Language Expert, CPython Enthusiast, Software Architect, Technical Author, and Senior Python Instructor.

# TASK

Create a COMPLETE Markdown (.md) guide on:

1. Magic Methods (Dunder Methods)

# OUTPUT RULES

Generate ONLY valid Markdown.

Start with:

# Advanced Python Object Customization – Part 1

Include a detailed Table of Contents.

Explain concepts from beginner to advanced level.

Use diagrams, examples, tables, and practical use cases.

For every method include:

- Definition
- Purpose
- Syntax
- Parameters
- Return Value
- Internal Python Behavior
- Execution Flow
- Real-world Usage
- Best Practices
- Common Mistakes


# TOPICS

## Introduction to Magic Methods

Cover:

- What are Dunder Methods
- Why Python uses them
- Object Model Overview
- How Python Calls Magic Methods
- Difference between normal methods and magic methods

---

# Object Lifecycle Methods

### __new__

Cover:

- Object creation process
- Relationship with __init__
- Immutable object customization
- Singleton implementation

Examples:

- Custom object creation
- Singleton pattern
- Immutable classes

---

### __init__

Cover:

- Object initialization
- Constructor patterns
- Validation techniques

Examples:

- Configuration objects
- User models

---

### __del__

Cover:

- Object destruction
- Garbage collection interaction
- Resource cleanup

Examples:

- File handlers
- Network resources

---

# Representation Methods

### __str__

### __repr__

### __format__

Cover:

- Purpose
- Differences
- Debugging use cases
- Logging use cases
- String formatting internals

Examples:

- Product class
- Employee class
- Financial objects

Comparison table:

- str() vs repr() vs format()

---

# Comparison Methods

### __eq__

### __ne__

### __lt__

### __gt__

### __le__

### __ge__

Cover:

- Rich comparison protocol
- Sorting behavior
- Equality semantics

Examples:

- Student ranking
- Product comparison
- Version comparison

Flow diagrams showing comparison execution.

---

# Arithmetic Methods

### __add__

### __sub__

### __mul__

### __truediv__

### __floordiv__

### __mod__

Cover:

- Operator overloading
- Mathematical object design
- Vector arithmetic
- Money calculations

Examples:

- Vector class
- Matrix class
- Currency class

Include operator-to-method mapping table.

---

# Include

- Internal execution diagrams
- Call sequence diagrams
- Advanced operator overloading examples
- Real-world projects
- Best practices
- Common mistakes
- Exercises
- Interview questions

End with:

## Summary

## Magic Methods Cheat Sheet (Part 1)

```markdown
# Advanced Python Object Customization – Part 1

## Table of Contents

1. [Introduction to Magic Methods](#introduction-to-magic-methods)
   - [What are Dunder Methods](#what-are-dunder-methods)
   - [Why Python Uses Them](#why-python-uses-them)
   - [Object Model Overview](#object-model-overview)
   - [How Python Calls Magic Methods](#how-python-calls-magic-methods)
   - [Difference Between Normal Methods and Magic Methods](#difference-between-normal-methods-and-magic-methods)

2. [Object Lifecycle Methods](#object-lifecycle-methods)
   - [`__new__`](#__new__)
   - [`__init__`](#__init__)
   - [`__del__`](#__del__)

3. [Representation Methods](#representation-methods)
   - [`__str__`](#__str__)
   - [`__repr____`](#__repr__)
   - [`__format__`](#__format__)
   - [Comparison: `str()` vs `repr()` vs `format()`](#comparison-str-vs-repr-vs-format)

4. [Comparison Methods](#comparison-methods)
   - [`__eq__`](#__eq__)
   - [`__ne__`](#__ne__)
   - [`__lt__`](#__lt__)
   - [`__gt__`](#__gt__)
   - [`__le__`](#__le__)
   - [`__ge__`](#__ge__)
   - [Rich Comparison Protocol](#rich-comparison-protocol)

5. [Arithmetic Methods](#arithmetic-methods)
   - [`__add__`](#__add__)
   - [`__sub__`](#__sub__)
   - [`__mul__`](#__mul__)
   - [`__truediv__`](#__truediv__)
   - [`__floordiv__`](#__floordiv__)
   - [`__mod__`](#__mod__)
   - [Operator-to-Method Mapping](#operator-to-method-mapping)

6. [Advanced Topics](#advanced-topics)
   - [Internal Execution Diagrams](#internal-execution-diagrams)
   - [Call Sequence Diagrams](#call-sequence-diagrams)
   - [Advanced Operator Overloading Examples](#advanced-operator-overloading-examples)
   - [Real-World Projects](#real-world-projects)

7. [Best Practices](#best-practices)

8. [Common Mistakes](#common-mistakes)

9. [Exercises](#exercises)

10. [Interview Questions](#interview-questions)

11. [Summary](#summary)

12. [Magic Methods Cheat Sheet (Part 1)](#magic-methods-cheat-sheet-part-1)

---

## Introduction to Magic Methods

### What are Dunder Methods

**Magic Methods** (also called **Dunder Methods**) are special methods in Python that have double underscores before and after their name (e.g., `__init__`, `__str__`). The term "dunder" comes from "double underscore."

```python
# Example of dunder method naming
__init__      # double underscore + init + double underscore
__str__       # double underscore + str + double underscore
__len__       # double underscore + len + double underscore
```

These methods are **not called directly** by developers in normal code. Instead, Python **automatically invokes them** when specific operations are performed on objects.

### Why Python Uses Them

Python uses magic methods to implement **operator overloading** and **behavior customization**:


| Purpose | Example |
| :-- | :-- |
| Operator overloading | `+` calls `__add__()` [web:1] |
| Type conversion | `str(obj)` calls `__str__()` [web:1] |
| Attribute access | `obj.attr` triggers `__getattribute__()` [web:1] |
| Object lifecycle | `class()` calls `__new__()` then `__init__()` [web:1] |
| Container behavior | `len(obj)` calls `__len__()` [web:1] |

**Key benefits:**

- Make custom objects behave like built-in types
- Enable intuitive syntax for complex operations
- Implement Python's "natural" programming style
- Create domain-specific languages within Python


### Object Model Overview

Python's object model is based on **classes** and **instances**:

```
┌─────────────────────────────────────┐
│           CLASS (Type)              │
│  ┌───────────────────────────┐     │
│  │  __new__    (creator)     │     │
│  │  __init__   (initializer) │     │
│  │  __str__    (string rep)  │     │
│  │  __eq__     (equality)    │     │
│  └───────────────────────────┘     │
└─────────────────────────────────────┘
         ↓ instance created
┌─────────────────────────────────────┐
│          INSTANCE (Object)          │
│  - Has attributes                   │
│  - Has methods                      │
│  - Behaves according to class       │
│    magic methods                    │
└─────────────────────────────────────┘
```

**Key concepts:**

- **Type**: The class that defines the object's behavior
- **Instance**: A concrete object created from the class
- **Attribute**: Data or methods attached to an object
- **Method**: Function bound to an object


### How Python Calls Magic Methods

Python's **interpreter** automatically calls magic methods during specific operations:

```python
class MyClass:
    def __add__(self, other):
        return "Added!"

obj1 = MyClass()
obj2 = MyClass()

# Python automatically calls __add__()
result = obj1 + obj2  # Internally: obj1.__add__(obj2)
print(result)         # Output: "Added!"
```

**Execution flow:**

```
User Code: obj1 + obj2
    ↓
Python Interpreter detects '+' operator
    ↓
Looks up __add__ method in obj1's class
    ↓
Calls obj1.__add__(obj2)
    ↓
Returns result to user code
```


### Difference Between Normal Methods and Magic Methods

| Aspect | Normal Methods | Magic Methods |
| :-- | :-- | :-- |
| **Naming** | Any valid name (e.g., `calculate()`) | Double underscores (`__method__`) [web:1] |
| **Invocation** | Called explicitly: `obj.method()` | Called automatically by Python [web:1] |
| **Purpose** | Custom business logic | Language behavior customization [web:1] |
| **Overridability** | Optional | Defines built-in behavior [web:1] |
| **Examples** | `calculate_total()`, `send_email()` | `__init__`, `__str__`, `__add__` [web:1] |

```python
# Normal method - called explicitly
class Person:
    def greet(self):
        return "Hello!"

p = Person()
print(p.greet())  # Must call .greet() explicitly

# Magic method - called automatically
class Person:
    def __str__(self):
        return "Person Object"

p = Person()
print(p)  # Python automatically calls __str__()
# Output: "Person Object"
```


---

## Object Lifecycle Methods

Object lifecycle methods control how objects are **created**, **initialized**, and **destroyed**.

### `__new__`

#### Definition

`__new__` is the **first method** called when creating an object. It's a **static method** that creates and returns a new instance.

#### Purpose

- Control object **creation** (not just initialization)
- Implement **singleton patterns**
- Create **immutable objects**
- Customize creation of **built-in types**


#### Syntax

```python
def __new__(cls, *args, **kwargs):
    # Create and return instance
    return super().__new__(cls)
```


#### Parameters

| Parameter | Type | Description |
| :-- | :-- | :-- |
| `cls` | class | The class being instantiated |
| `*args` | tuple | Positional arguments passed to constructor |
| `**kwargs` | dict | Keyword arguments passed to constructor |

#### Return Value

Must return a **new instance** of the class (or subclass).

#### Internal Python Behavior

1. Python calls `__new__()` **before** `__init__()`
2. `__new__()` creates the raw object
3. If `__new__()` returns an instance of `cls`, Python calls `__init__()` on it
4. If `__new__()` returns something else, `__init__()` is **not called**

#### Execution Flow

```
┌─────────────────────────────────────────────┐
│  User: obj = MyClass(args)                  │
└─────────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────────┐
│  1. Python calls MyClass.__new__(cls, args) │
└─────────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────────┐
│  2. __new__ creates instance:               │
│      instance = super().__new__(cls)        │
└─────────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────────┐
│  3. __new__ returns instance                │
└─────────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────────┐
│  4. If instance is of cls, call:            │
│      instance.__init__(args)                │
└─────────────────────────────────────────────┘
```


#### Real-world Usage

**Example 1: Custom Object Creation**

```python
class ConfigObject:
    def __new__(cls, config_dict):
        # Validate config before creation
        if not isinstance(config_dict, dict):
            raise TypeError("Config must be a dictionary")
        
        # Create instance
        instance = super().__new__(cls)
        return instance
    
    def __init__(self, config_dict):
        self.config = config_dict
        self.validate()
    
    def validate(self):
        required_keys = ['name', 'version']
        for key in required_keys:
            if key not in self.config:
                raise ValueError(f"Missing required key: {key}")

# Usage
config = ConfigObject({'name': 'MyApp', 'version': '1.0'})
print(config.config)  # {'name': 'MyApp', 'version': '1.0'}
```

**Example 2: Singleton Pattern**

```python
class Singleton:
    _instance = None
    
    def __new__(cls, *args, **kwargs):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
            cls._instance.initialized = False
        return cls._instance
    
    def __init__(self, value):
        if not self.initialized:
            self.value = value
            self.initialized = True

# Usage
obj1 = Singleton("first")
obj2 = Singleton("second")

print(obj1 is obj2)  # True - same instance
print(obj1.value)    # "first" - second init was skipped
```

**Example 3: Immutable Classes**

```python
class ImmutablePoint:
    def __new__(cls, x, y):
        instance = super().__new__(cls)
        instance._x = x
        instance._y = y
        return instance
    
    def __init__(self, x, y):
        # Prevent any modification in __init__
        pass
    
    @property
    def x(self):
        return self._x
    
    @property
    def y(self):
        return self._y
    
    def __setattr__(self, name, value):
        raise AttributeError("Cannot modify immutable object")

# Usage
point = ImmutablePoint(10, 20)
print(point.x)  # 10
point.x = 30    # Raises AttributeError
```


#### Best Practices

- Always call `super().__new__(cls)` to create the instance
- Return the instance from `__new__()`
- Use `__new__()` when you need to control **creation**, not just initialization
- For most cases, use `__init__()` instead (simpler)


#### Common Mistakes

```python
# ❌ Mistake 1: Not returning instance
def __new__(cls):
    super().__new__(cls)  # Missing return!

# ❌ Mistake 2: Calling __init__ manually
def __new__(cls):
    instance = super().__new__(cls)
    instance.__init__()  # Wrong - Python calls __init__ automatically

# ❌ Mistake 3: Not passing args to super
def __new__(cls, x, y):
    instance = super().__new__(cls)  # Missing x, y!
    return instance
```


---

### `__init__`

#### Definition

`__init__` is the **initializer method** called after `__new__()` to initialize the object's attributes.

#### Purpose

- Set up object **attributes**
- Perform **validation**
- Initialize **state**
- Configure object for use


#### Syntax

```python
def __init__(self, *args, **kwargs):
    # Initialize attributes
    self.attribute = value
```


#### Parameters

| Parameter | Type | Description |
| :-- | :-- | :-- |
| `self` | instance | The newly created instance |
| `*args` | tuple | Positional arguments from constructor |
| `**kwargs` | dict | Keyword arguments from constructor |

#### Return Value

Must return **None** (Python enforces this).

#### Internal Python Behavior

1. Called automatically after `__new__()` returns an instance
2. Receives the instance created by `__new__()`
3. Initializes object attributes
4. Cannot change the instance's type

#### Execution Flow

```
┌─────────────────────────────────────────┐
│  obj = MyClass(args)                    │
└─────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────┐
│  __new__ creates instance               │
└─────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────┐
│  __init__(instance, args) called        │
│  - Initializes attributes               │
│  - Performs validation                  │
└─────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────┐
│  obj is ready for use                   │
└─────────────────────────────────────────┘
```


#### Real-world Usage

**Example 1: Configuration Objects**

```python
class Config:
    def __init__(self, app_name, version, debug=False, max_connections=100):
        self.app_name = app_name
        self.version = version
        self.debug = debug
        self.max_connections = max_connections
        
        # Validation
        if not isinstance(app_name, str) or len(app_name) == 0:
            raise ValueError("app_name must be a non-empty string")
        
        if not isinstance(version, str):
            raise ValueError("version must be a string")
        
        if max_connections < 1:
            raise ValueError("max_connections must be at least 1")
    
    def __repr__(self):
        return f"Config(app_name='{self.app_name}', version='{self.version}')"

# Usage
config = Config("MyApp", "2.0", debug=True, max_connections=500)
print(config)  # Config(app_name='MyApp', version='2.0')
```

**Example 2: User Models**

```python
from datetime import datetime
import re

class User:
    def __init__(self, username, email, age):
        self.username = self._validate_username(username)
        self.email = self._validate_email(email)
        self.age = self._validate_age(age)
        self.created_at = datetime.now()
        self.is_active = True
    
    def _validate_username(self, username):
        if not isinstance(username, str):
            raise TypeError("Username must be a string")
        if len(username) < 3 or len(username) > 50:
            raise ValueError("Username must be 3-50 characters")
        if not re.match(r'^[a-zA-Z0-9_]+$', username):
            raise ValueError("Username can only contain letters, numbers, and underscores")
        return username
    
    def _validate_email(self, email):
        if not isinstance(email, str):
            raise TypeError("Email must be a string")
        email_pattern = r'^[a-zA-Z0-9._+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
        if not re.match(email_pattern, email):
            raise ValueError("Invalid email format")
        return email.lower()
    
    def _validate_age(self, age):
        if not isinstance(age, int):
            raise TypeError("Age must be an integer")
        if age < 0 or age > 150:
            raise ValueError("Age must be between 0 and 150")
        return age
    
    def activate(self):
        self.is_active = True
    
    def deactivate(self):
        self.is_active = False

# Usage
user = User("john_doe", "john@example.com", 25)
print(user.username)  # "john_doe"
print(user.email)     # "john@example.com" (lowercased)
```


#### Constructor Patterns

**Pattern 1: Default Arguments**

```python
class Product:
    def __init__(self, name, price, quantity=0, category="General"):
        self.name = name
        self.price = price
        self.quantity = quantity
        self.category = category
```

**Pattern 2: Keyword-only Arguments**

```python
class APIClient:
    def __init__(self, base_url, *, timeout=30, retry_count=3, debug=False):
        self.base_url = base_url
        self.timeout = timeout
        self.retry_count = retry_count
        self.debug = debug
```

**Pattern 3: Type Conversion**

```python
class Money:
    def __init__(self, amount, currency="USD"):
        self.amount = float(amount)  # Convert to float
        self.currency = currency
```


#### Validation Techniques

| Technique | Example |
| :-- | :-- |
| **Type checking** | `if not isinstance(x, int): raise TypeError()` |
| **Range validation** | `if x < 0 or x > 100: raise ValueError()` |
| **Pattern matching** | `if not re.match(pattern, x): raise ValueError()` |
| **Required fields** | `if field is None: raise ValueError()` |
| **Business logic** | `if end_date < start_date: raise ValueError()` |

#### Best Practices

- Initialize all attributes in `__init__()`
- Perform validation early (fail fast)
- Use default arguments for common cases
- Keep `__init__()` focused on initialization (not heavy processing)
- Raise appropriate exceptions (`TypeError`, `ValueError`)


#### Common Mistakes

```python
# ❌ Mistake 1: Returning a value
def __init__(self):
    return self  # TypeError!

# ❌ Mistake 2: Not initializing attributes
def __init__(self, name):
    pass  # name is not stored!

# ❌ Mistake 3: Heavy computation in __init__
def __init__(self, data):
    self.result = self._process_large_file(data)  # Slow!
    # Better: use a separate method
```


---

### `__del__`

#### Definition

`__del__` is the **destructor method** called when an object is about to be destroyed.

#### Purpose

- Clean up **resources**
- Close **file handlers**
- Release **network connections**
- Perform **cleanup** before garbage collection


#### Syntax

```python
def __del__(self):
    # Cleanup code
    pass
```


#### Parameters

| Parameter | Type | Description |
| :-- | :-- | :-- |
| `self` | instance | The instance being destroyed |

#### Return Value

Must return **None**.

#### Internal Python Behavior

1. Called when object's **reference count** reaches zero
2. Called during **garbage collection** for cyclic references
3. **Not guaranteed** to be called immediately
4. May not be called if program exits abruptly

#### Execution Flow

```
┌─────────────────────────────────────────┐
│  obj = SomeResource()                   │
└─────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────┐
│  obj = None  (or last reference removed)│
└─────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────┐
│  Reference count = 0                    │
└─────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────┐
│  Python schedules __del__ call          │
└─────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────┐
│  __del__(obj) called (timing uncertain) │
│  - Cleans up resources                  │
└─────────────────────────────────────────┘
```


#### Real-world Usage

**Example 1: File Handlers**

```python
class FileHandler:
    def __init__(self, filename, mode='r'):
        self.filename = filename
        self.mode = mode
        self.file = open(filename, mode)
        print(f"Opened {filename}")
    
    def read(self):
        return self.file.read()
    
    def write(self, content):
        if 'w' in self.mode:
            self.file.write(content)
    
    def __del__(self):
        if self.file and not self.file.closed:
            self.file.close()
            print(f"Closed {self.filename}")
    
    def __enter__(self):
        return self
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        self.__del__()

# Usage
with FileHandler('test.txt', 'w') as f:
    f.write("Hello")
# Automatically closes when exiting 'with' block
```

**Example 2: Network Resources**

```python
import socket

class NetworkConnection:
    def __init__(self, host, port):
        self.host = host
        self.port = port
        self.socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        self.socket.connect((host, port))
        print(f"Connected to {host}:{port}")
    
    def send(self, data):
        self.socket.send(data.encode())
    
    def receive(self, bufsize=1024):
        return self.socket.recv(bufsize).decode()
    
    def __del__(self):
        if self.socket:
            self.socket.close()
            print(f"Disconnected from {self.host}:{self.port}")
    
    def __enter__(self):
        return self
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        self.__del__()

# Usage
conn = NetworkConnection('example.com', 80)
conn.send("GET /")
response = conn.receive()
# Connection automatically closed when conn is garbage collected
```

**Example 3: Database Connection Pool**

```python
class DatabaseConnection:
    def __init__(self, db_url):
        self.db_url = db_url
        self.connection = self._connect()
        self.is_connected = True
    
    def _connect(self):
        # Simulate database connection
        return f"Connection({self.db_url})"
    
    def query(self, sql):
        if self.is_connected:
            return f"Result of: {sql}"
        raise ConnectionError("Not connected")
    
    def __del__(self):
        if self.is_connected and self.connection:
            self.connection = None
            self.is_connected = False
            print(f"Database connection closed: {self.db_url}")

# Usage
db = DatabaseConnection("postgresql://localhost/mydb")
result = db.query("SELECT * FROM users")
print(result)
# Connection closed when db is garbage collected
```


#### Best Practices

- **Don't rely on `__del__`** for critical cleanup (use `try/finally` or context managers)
- Handle exceptions gracefully in `__del__` (don't raise exceptions)
- Check if resources are already closed
- Use context managers (`__enter__`, `__exit__`) for explicit cleanup
- Consider using `weakref` to avoid reference cycles


#### Common Mistakes

```python
# ❌ Mistake 1: Raising exceptions in __del__
def __del__(self):
    raise RuntimeError("Cleanup failed!")  # Ignored by Python!

# ❌ Mistake 2: Not checking if resource exists
def __del__(self):
    self.file.close()  # AttributeError if file wasn't created!

# ❌ Mistake 3: Creating reference cycles
class Node:
    def __init__(self):
        self.parent = None
        self.children = []
    
    def add_child(self, child):
        self.children.append(child)
        child.parent = self  # Creates cycle!
    
    def __del__(self):
        print("Deleting node")  # May not be called!

# ✅ Better: Use context managers
class FileHandler:
    def __enter__(self):
        self.file = open(self.filename, self.mode)
        return self
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        if self.file and not self.file.closed:
            self.file.close()
```


---

## Representation Methods

Representation methods control how objects are **converted to strings** for display, logging, and debugging.

### `__str__`

#### Definition

`__str__` returns the **informal string representation** of an object, intended for **end users**.

#### Purpose

- Provide **human-readable** output
- Display object information to users
- Format objects for **presentation**


#### Syntax

```python
def __str__(self):
    return "string representation"
```


#### Parameters

| Parameter | Type | Description |
| :-- | :-- | :-- |
| `self` | instance | The instance to convert to string |

#### Return Value

Must return a **string** (`str`).

#### Internal Python Behavior

1. Called by `str(obj)`
2. Called by `print(obj)`
3. Called when using `{}` in `format()` (without `!r`)
4. If not defined, Python uses `__repr__()`

#### Real-world Usage

**Example: Product Class**

```python
class Product:
    def __init__(self, name, price, description):
        self.name = name
        self.price = price
        self.description = description
    
    def __str__(self):
        return f"{self.name} - $${self.price:.2f} - {self.description}"

# Usage
product = Product("Laptop", 999.99, "High-performance gaming laptop")
print(product)  # Laptop - $999.99 - High-performance gaming laptop
print(str(product))  # Same output
```


### `__repr__`

#### Definition

`__repr__` returns the **formal string representation** of an object, intended for **developers** (debugging, logging).

#### Purpose

- Provide **unambiguous** representation
- Enable **debugging**
- Support **reconstruction** of object
- Show object's **internal state**


#### Syntax

```python
def __repr__(self):
    return "representational string"
```


#### Parameters

| Parameter | Type | Description |
| :-- | :-- | :-- |
| `self` | instance | The instance to represent |

#### Return Value

Must return a **string** (`str`). Ideally, a string that could be used to recreate the object.

#### Internal Python Behavior

1. Called by `repr(obj)`
2. Called in **interactive interpreter** when printing objects
3. Called when using `{!r}` in `format()`
4. Called in **lists, dicts** when printing
5. If not defined, Python uses default object representation

#### Real-world Usage

**Example: Employee Class**

```python
class Employee:
    def __init__(self, id, name, department, salary):
        self.id = id
        self.name = name
        self.department = department
        self.salary = salary
    
    def __repr__(self):
        return f"Employee(id={self.id}, name='{self.name}', department='{self.department}', salary={self.salary})"

# Usage
emp = Employee(101, "John Doe", "Engineering", 75000)
print(repr(emp))  
# Employee(id=101, name='John Doe', department='Engineering', salary=75000)

# In a list
employees = [emp]
print(employees)  
# [Employee(id=101, name='John Doe', department='Engineering', salary=75000)]
```


### `__format__`

#### Definition

`__format__` controls how an object is formatted within **string formatting operations**.

#### Purpose

- Customize **format specifiers**
- Support domain-specific formatting
- Control output in `format()` and `{}`


#### Syntax

```python
def __format__(self, format_spec):
    return "formatted string"
```


#### Parameters

| Parameter | Type | Description |
| :-- | :-- | :-- |
| `self` | instance | The instance to format |
| `format_spec` | str | The format specification (e.g., `".2f"`, `">10"`) |

#### Return Value

Must return a **string** (`str`).

#### Internal Python Behavior

1. Called by `format(obj, format_spec)`
2. Called in `{：format_spec}` within f-strings
3. Called in `{：format_spec}` within `format()` string
4. If not defined, Python uses `__str__()`

#### Real-world Usage

**Example: Financial Objects**

```python
class Money:
    def __init__(self, amount, currency="USD"):
        self.amount = float(amount)
        self.currency = currency
    
    def __str__(self):
        return f"${self.amount:.2f} {self.currency}"
    
    def __repr__(self):
        return f"Money(amount={self.amount}, currency='{self.currency}')"
    
    def __format__(self, format_spec):
        # Custom formatting based on format_spec
        if format_spec == 'euro':
            return f"€{self.amount * 0.85:.2f} EUR"
        elif format_spec == 'compact':
            return f"${self.amount/1000:.1f}k {self.currency}"
        elif format_spec == 'precise':
            return f"${self.amount:.4f} {self.currency}"
        else:
            # Default formatting
            return f"${self.amount:.2f} {self.currency}"

# Usage
money = Money(1234.567, "USD")

print(money)                    # $1234.57 USD (uses __str__)
print(repr(money))              # Money(amount=1234.567, currency='USD') (uses __repr__)
print(format(money, 'euro'))    # €1049.43 EUR
print(format(money, 'compact')) # $1.2k USD
print(format(money, 'precise')) # $1234.5670 USD
print(f"{money:euro}")          # €1049.43 EUR
print(f"{money:compact}")       # $1.2k USD
```


### Comparison: `str()` vs `repr()` vs `format()`

| Aspect | `str()` | `repr()` | `format()` |
| :-- | :-- | :-- | :-- |
| **Method** | `__str__()` | `__repr__()` | `__format__()` |
| **Purpose** | User display | Developer/debug | Custom formatting |
| **Called by** | `print()`, `str()` | `repr()`, interactive | `format()`, `{}` |
| **Expected output** | Human-readable | Unambiguous | Format-specific |
| **Fallback** | Uses `__repr__()` | Default object repr | Uses `__str__()` |
| **Example** | `"Laptop - $999.99"` | `"Product(name='Laptop', price=999.99)"` | `"€849.43"` (euro format) |

```python
class Product:
    def __init__(self, name, price):
        self.name = name
        self.price = price
    
    def __str__(self):
        return f"{self.name} - $${self.price:.2f}"
    
    def __repr__(self):
        return f"Product(name='{self.name}', price={self.price})"

p = Product("Laptop", 999.99)

print(str(p))      # Laptop - $999.99  (__str__)
print(repr(p))     # Product(name='Laptop', price=999.99)  (__repr__)
print(p)           # Laptop - $999.99  (__str__ via print)
```


---

## Comparison Methods

Comparison methods enable **rich comparison** between objects, supporting operators like `==`, `!=`, `<`, `>`, `\<=`, `\>=`.

### `__eq__`

#### Definition

`__eq__` defines **equality** behavior for the `==` operator.

#### Purpose

- Determine if two objects are **equal**
- Define **semantic equality** (not just identity)
- Enable object comparison in collections


#### Syntax

```python
def __eq__(self, other):
    return True  # or False
```


#### Parameters

| Parameter | Type | Description |
| :-- | :-- | :-- |
| `self` | instance | The left-hand object |
| `other` | any | The right-hand object to compare |

#### Return Value

Returns `True` if equal, `False` otherwise. Can return `NotImplemented` for unsupported types.

#### Real-world Usage

**Example: Student Ranking**

```python
class Student:
    def __init__(self, name, grade):
        self.name = name
        self.grade = grade
    
    def __eq__(self, other):
        if not isinstance(other, Student):
            return NotImplemented
        return self.grade == other.grade

# Usage
s1 = Student("Alice", 95)
s2 = Student("Bob", 95)
s3 = Student("Charlie", 88)

print(s1 == s2)  # True (same grade)
print(s1 == s3)  # False (different grade)
```


### `__ne__`

#### Definition

`__ne__` defines **inequality** behavior for the `!=` operator.

#### Purpose

- Determine if two objects are **not equal**
- Complement `__eq__()`


#### Syntax

```python
def __ne__(self, other):
    return True  # or False
```


#### Parameters

| Parameter | Type | Description |
| :-- | :-- | :-- |
| `self` | instance | The left-hand object |
| `other` | any | The right-hand object |

#### Return Value

Returns `True` if not equal, `False` otherwise.

#### Note

Python automatically negates `__eq__()` if `__ne__()` is not defined. Usually, you don't need to implement `__ne__()` explicitly.

#### Real-world Usage

```python
class Version:
    def __init__(self, version_str):
        self.version = version_str
    
    def __eq__(self, other):
        if not isinstance(other, Version):
            return NotImplemented
        return self.version == other.version
    
    def __ne__(self, other):
        if not isinstance(other, Version):
            return NotImplemented
        return self.version != other.version

# Usage
v1 = Version("1.0.0")
v2 = Version("1.0.0")
v3 = Version("2.0.0")

print(v1 != v2)  # False
print(v1 != v3)  # True
```


### `__lt__`

#### Definition

`__lt__` defines **less than** behavior for the `<` operator.

#### Purpose

- Enable **sorting**
- Define **ordering semantics**
- Support comparison operations


#### Syntax

```python
def __lt__(self, other):
    return True  # or False
```


#### Parameters

| Parameter | Type | Description |
| :-- | :-- | :-- |
| `self` | instance | The left-hand object |
| `other` | any | The right-hand object |

#### Return Value

Returns `True` if `self < other`, `False` otherwise.

#### Real-world Usage

**Example: Product Comparison (by price)**

```python
class Product:
    def __init__(self, name, price):
        self.name = name
        self.price = price
    
    def __lt__(self, other):
        if not isinstance(other, Product):
            return NotImplemented
        return self.price < other.price

# Usage
products = [
    Product("Laptop", 999),
    Product("Phone", 599),
    Product("Headphones", 199)
]

# Sort by price (uses __lt__)
sorted_products = sorted(products)
for p in sorted_products:
    print(f"{p.name}: $${p.price}")

# Output:
# Headphones: $199
# Phone: $599
# Laptop: $999
```


### `__gt__`

#### Definition

`__gt__` defines **greater than** behavior for the `>` operator.

#### Purpose

- Enable **reverse sorting**
- Define **ordering semantics**
- Support comparison operations


#### Syntax

```python
def __gt__(self, other):
    return True  # or False
```


#### Real-world Usage

**Example: Version Comparison**

```python
class Version:
    def __init__(self, version_str):
        self.version = tuple(map(int, version_str.split('.')))
    
    def __gt__(self, other):
        if not isinstance(other, Version):
            return NotImplemented
        return self.version > other.version
    
    def __repr__(self):
        return f"Version('{self.version}')"

# Usage
v1 = Version("2.0.0")
v2 = Version("1.5.0")
v3 = Version("2.0.1")

print(v1 > v2)  # True
print(v1 > v3)  # False
```


### `__le__`

#### Definition

`__le__` defines **less than or equal** behavior for the `\<=` operator.

#### Syntax

```python
def __le__(self, other):
    return True  # or False
```


#### Real-world Usage

```python
class Student:
    def __init__(self, name, score):
        self.name = name
        self.score = score
    
    def __le__(self, other):
        if not isinstance(other, Student):
            return NotImplemented
        return self.score <= other.score

# Usage
s1 = Student("Alice", 95)
s2 = Student("Bob", 95)
s3 = Student("Charlie", 88)

print(s1 <= s2)  # True (equal scores)
print(s1 <= s3)  # False
```


### `__ge__`

#### Definition

`__ge__` defines **greater than or equal** behavior for the `\>=` operator.

#### Syntax

```python
def __ge__(self, other):
    return True  # or False
```


#### Real-world Usage

```python
class Temperature:
    def __init__(self, value, unit="C"):
        self.value = value
        self.unit = unit
    
    def __ge__(self, other):
        if not isinstance(other, Temperature):
            return NotImplemented
        return self.value >= other.value

# Usage
t1 = Temperature(30, "C")
t2 = Temperature(25, "C")
t3 = Temperature(30, "C")

print(t1 >= t2)  # True
print(t1 >= t3)  # True (equal)
```


### Rich Comparison Protocol

Python's **rich comparison** protocol includes all six comparison methods:


| Operator | Method | Description |
| :-- | :-- | :-- |
| `==` | `__eq__` | Equal |
| `!=` | `__ne__` | Not equal |
| `<` | `__lt__` | Less than |
| `>` | `__gt__` | Greater than |
| `\<=` | `__le__` | Less than or equal |
| `\>=` | `__ge__` | Greater than or equal |

#### Sorting Behavior

When you call `sorted()` or `.sort()`, Python uses:

1. `__lt__()` by default (for `<` comparison)
2. Custom key functions can override this
```python
class Product:
    def __init__(self, name, price, rating):
        self.name = name
        self.price = price
        self.rating = rating
    
    def __lt__(self, other):
        return self.price < other.price
    
    def __repr__(self):
        return f"{self.name} ($${self.price}, {self.rating}★)"

products = [
    Product("Laptop", 999, 4.5),
    Product("Phone", 599, 4.8),
    Product("Headphones", 199, 4.2)
]

# Sort by price (uses __lt__)
print("By price:", sorted(products))

# Sort by rating (uses key function)
print("By rating:", sorted(products, key=lambda p: p.rating, reverse=True))
```


#### Equality Semantics

**Identity vs Equality:**

```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y
    
    def __eq__(self, other):
        if not isinstance(other, Point):
            return NotImplemented
        return self.x == other.x and self.y == other.y

p1 = Point(1, 2)
p2 = Point(1, 2)
p3 = p1

print(p1 == p2)  # True (same values - equality)
print(p1 is p2)  # False (different objects - identity)
print(p1 is p3)  # True (same object - identity)
```


#### Comparison Execution Flow

```
┌─────────────────────────────────────────┐
│  obj1 < obj2                            │
└─────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────┐
│  1. Try obj1.__lt__(obj2)               │
└─────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────┐
│  2. If returns NotImplemented:          │
│      Try obj2.__gt__(obj1)              │
└─────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────┐
│  3. If both fail: raise TypeError       │
└─────────────────────────────────────────┘
```


---

## Arithmetic Methods

Arithmetic methods enable **operator overloading** for mathematical operations, allowing custom objects to behave like numbers.

### `__add__`

#### Definition

`__add__` defines **addition** behavior for the `+` operator.

#### Purpose

- Enable `obj1 + obj2` syntax
- Implement **domain-specific addition**
- Support **vector/matrix** arithmetic


#### Syntax

```python
def __add__(self, other):
    return result
```


#### Real-world Usage

**Example: Vector Class**

```python
class Vector:
    def __init__(self, x, y):
        self.x = x
        self.y = y
    
    def __add__(self, other):
        if not isinstance(other, Vector):
            return NotImplemented
        return Vector(self.x + other.x, self.y + other.y)
    
    def __repr__(self):
        return f"Vector({self.x}, {self.y})"

# Usage
v1 = Vector(1, 2)
v2 = Vector(3, 4)
v3 = v1 + v2

print(v3)  # Vector(4, 6)
```


### `__sub__`

#### Definition

`__sub__` defines **subtraction** behavior for the `-` operator.

#### Real-world Usage

**Example: Currency Class**

```python
class Money:
    def __init__(self, amount, currency="USD"):
        self.amount = float(amount)
        self.currency = currency
    
    def __sub__(self, other):
        if not isinstance(other, Money):
            return NotImplemented
        if self.currency != other.currency:
            raise ValueError("Cannot subtract different currencies")
        return Money(self.amount - other.amount, self.currency)
    
    def __repr__(self):
        return f"${self.amount:.2f} {self.currency}"

# Usage
m1 = Money(100, "USD")
m2 = Money(30, "USD")
m3 = m1 - m2

print(m3)  # $70.00 USD
```


### `__mul__`

#### Definition

`__mul__` defines **multiplication** behavior for the `*` operator.

#### Real-world Usage

**Example: Matrix Class**

```python
class Matrix:
    def __init__(self, rows):
        self.rows = rows
    
    def __mul__(self, other):
        if isinstance(other, Matrix):
            # Matrix multiplication
            if len(self.rows) != len(other.rows):
                raise ValueError("Matrix dimensions incompatible")
            
            result = []
            for i in range(len(self.rows)):
                row = []
                for j in range(len(other.rows)):
                    val = sum(self.rows[i][k] * other.rows[k][j] for k in range(len(other.rows)))
                    row.append(val)
                result.append(row)
            return Matrix(result)
        elif isinstance(other, (int, float)):
            # Scalar multiplication
            return Matrix([[val * other for val in row] for row in self.rows])
        return NotImplemented
    
    def __repr__(self):
        return f"Matrix({self.rows})"

# Usage
m1 = Matrix([, ])
m2 = Matrix([, ])
m3 = m1 * m2

print(m3)  # Matrix([, ])

# Scalar multiplication
m4 = m1 * 2
print(m4)  # Matrix([, ])
```


### `__truediv__`

#### Definition

`__truediv__` defines **true division** behavior for the `/` operator.

#### Real-world Usage

```python
class Percentage:
    def __init__(self, value):
        self.value = float(value)
    
    def __truediv__(self, other):
        if isinstance(other, (int, float)):
            return Percentage(self.value / other)
        return NotImplemented
    
    def __repr__(self):
        return f"{self.value:.2f}%"

# Usage
p1 = Percentage(50)
p2 = p1 / 2

print(p2)  # 25.00%
```


### `__floordiv__`

#### Definition

`__floordiv__` defines **floor division** behavior for the `//` operator.

#### Real-world Usage

```python
class Samples:
    def __init__(self, count):
        self.count = int(count)
    
    def __floordiv__(self, other):
        if isinstance(other, int):
            return Samples(self.count // other)
        return NotImplemented
    
    def __repr__(self):
        return f"{self.count} samples"

# Usage
s1 = Samples(100)
s2 = s1 // 3

print(s2)  # 33 samples
```


### `__mod__`

#### Definition

`__mod__` defines **modulo** behavior for the `%` operator.

#### Real-world Usage

```python
class Cycle:
    def __init__(self, position, length):
        self.position = position % length
        self.length = length
    
    def __mod__(self, other):
        if isinstance(other, int):
            return Cycle(self.position % other, self.length)
        return NotImplemented
    
    def __repr__(self):
        return f"Cycle({self.position}/{self.length})"

# Usage
c1 = Cycle(7, 10)
c2 = c1 % 5

print(c2)  # Cycle(2/10)
```


### Operator-to-Method Mapping

| Operator | Method | Example |
| :-- | :-- | :-- |
| `+` | `__add__` | `a + b` → `a.__add__(b)` |
| `-` | `__sub__` | `a - b` → `a.__sub__(b)` |
| `*` | `__mul__` | `a * b` → `a.__mul__(b)` |
| `/` | `__truediv__` | `a / b` → `a.__truediv__(b)` |
| `//` | `__floordiv__` | `a // b` → `a.__floordiv__(b)` |
| `%` | `__mod__` | `a % b` → `a.__mod__(b)` |
| `**` | `__pow__` | `a ** b` → `a.__pow__(b)` |
| `&` | `__and__` | `a & b` → `a.__and__(b)` |
| `|` | `__or__` | `a | b` → `a.__or__(b)` |
| `^` | `__xor__` | `a ^ b` → `a.__xor__(b)` |
| `<<` | `__lshift__` | `a << b` → `a.__lshift__(b)` |
| `>>` | `__rshift__` | `a >> b` → `a.__rshift__(b)` |


---

## Advanced Topics

### Internal Execution Diagrams

#### Object Creation Flow

```
┌─────────────────────────────────────────────────────┐
│  obj = MyClass(arg1, arg2)                          │
└─────────────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────────────┐
│  1. MyClass.__new__(MyClass, arg1, arg2)            │
│     - Creates raw instance                            │
│     - Returns instance                                │
└─────────────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────────────┐
│  2. instance.__init__(arg1, arg2)                    │
│     - Initializes attributes                          │
│     - Performs validation                             │
└─────────────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────────────┐
│  3. obj is ready for use                             │
└─────────────────────────────────────────────────────┘
```


### Call Sequence Diagrams

#### Expression Evaluation

```
Expression: a + b * c

┌─────────────────────────────────────────┐
│  1. Evaluate b * c                      │
│     → b.__mul__(c)                      │
│     → result1                           │
└─────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────┐
│  2. Evaluate a + result1                │
│     → a.__add__(result1)                │
│     → final_result                      │
└─────────────────────────────────────────┘
```


### Advanced Operator Overloading Examples

**Example: Money Class with All Arithmetic**

```python
class Money:
    def __init__(self, amount, currency="USD"):
        self.amount = float(amount)
        self.currency = currency
    
    def __add__(self, other):
        if isinstance(other, Money):
            if self.currency != other.currency:
                raise ValueError("Cannot add different currencies")
            return Money(self.amount + other.amount, self.currency)
        elif isinstance(other, (int, float)):
            return Money(self.amount + other, self.currency)
        return NotImplemented
    
    def __sub__(self, other):
        if isinstance(other, Money):
            if self.currency != other.currency:
                raise ValueError("Cannot subtract different currencies")
            return Money(self.amount - other.amount, self.currency)
        elif isinstance(other, (int, float)):
            return Money(self.amount - other, self.currency)
        return NotImplemented
    
    def __mul__(self, other):
        if isinstance(other, (int, float)):
            return Money(self.amount * other, self.currency)
        return NotImplemented
    
    def __truediv__(self, other):
        if isinstance(other, (int, float)):
            if other == 0:
                raise ZeroDivisionError("Cannot divide by zero")
            return Money(self.amount / other, self.currency)
        return NotImplemented
    
    def __abs__(self):
        return Money(abs(self.amount), self.currency)
    
    def __neg__(self):
        return Money(-self.amount, self.currency)
    
    def __pos__(self):
        return Money(+self.amount, self.currency)
    
    def __repr__(self):
        return f"${self.amount:.2f} {self.currency}"

# Usage
m1 = Money(100, "USD")
m2 = Money(50, "USD")

print(m1 + m2)  # $150.00 USD
print(m1 - m2)  # $50.00 USD
print(m1 * 2)   # $200.00 USD
print(m1 / 2)   # $50.00 USD
print(abs(Money(-100)))  # $100.00 USD
```


### Real-World Projects

**Project 1: DateTime Range**

```python
from datetime import datetime, timedelta

class DateRange:
    def __init__(self, start, end):
        self.start = start
        self.end = end
    
    def __contains__(self, date):
        return self.start <= date <= self.end
    
    def __len__(self):
        return (self.end - self.start).days + 1
    
    def __add__(self, other):
        if isinstance(other, timedelta):
            return DateRange(self.start + other, self.end + other)
        return NotImplemented
    
    def __sub__(self, other):
        if isinstance(other, timedelta):
            return DateRange(self.start - other, self.end - other)
        return NotImplemented
    
    def __repr__(self):
        return f"DateRange({self.start}, {self.end})"

# Usage
start = datetime(2024, 1, 1)
end = datetime(2024, 1, 10)
range = DateRange(start, end)

print(datetime(2024, 1, 5) in range)  # True
print(len(range))  # 10
print(range + timedelta(days=1))  # DateRange(2024-01-02, 2024-01-11)
```


---

## Best Practices

1. **Return `NotImplemented`** for unsupported types (not `TypeError`)
2. **Implement symmetric operations** (e.g., `__add__` and `__radd__`)
3. **Keep magic methods focused** - one purpose per method
4. **Use `__repr__` for debugging**, `__str__` for users
5. **Don't rely on `__del__`** - use context managers instead
6. **Validate early** in `__init__()` (fail fast)
7. **Document behavior** clearly for custom operators
8. **Test edge cases** (zero, negative, None, different types)

---

## Common Mistakes

1. ❌ **Returning values from `__init__()`** - must return `None`
2. ❌ **Not checking types** in comparison/arithmetic methods
3. ❌ **Raising exceptions in `__del__()`** - ignored by Python
4. ❌ **Not returning `NotImplemented`** for unsupported types
5. ❌ **Implementing `__ne__()`** when `__eq__()` exists (auto-negated)
6. ❌ **Creating reference cycles** with `__del__()`
7. ❌ **Heavy computation in `__init__()`** - slow instantiation

---

## Exercises

1. **Create a `Fraction` class** with `__add__`, `__sub__`, `__mul__`, `__truediv__`, `__eq__`, and `__repr__`.
2. **Implement a `SmartList` class** that supports:
    - `len(list)` using `__len__`
    - `list[index]` using `__getitem__`
    - `list[index] = value` using `__setitem__`
    - `value in list` using `__contains__`
3. **Create a `BoundingBox` class** with `__add__` (union), `__and__` (intersection), and `__contains__` (point inside).
4. **Implement a `DecimalMoney` class** that uses `decimal.Decimal` for precise money calculations.

---

## Interview Questions

1. **What's the difference between `__new__` and `__init__`?**
2. **When does Python call `__str__` vs `__repr__`?**
3. **How do you implement custom comparison for a class?**
4. **What happens if `__add__` returns `NotImplemented`?**
5. **Why is `__del__` not guaranteed to be called?**
6. **How would you implement a singleton using magic methods?**
7. **What magic methods would you need for a sortable, comparable collection?**

---

## Summary

This guide covered **Object Lifecycle Methods** and **Representation Methods**:

- **`__new__`**: Controls object creation (before `__init__`)
- **`__init__`**: Initializes object attributes (most common)
- **`__del__`**: Cleanup before destruction (use context managers instead)
- **`__str__`**: User-facing string representation
- **`__repr__`**: Developer/debug representation
- **`__format__`**: Custom formatting support
- **Comparison methods**: `__eq__`, `__ne__`, `__lt__`, `__gt__`, `__le__`, `__ge__`
- **Arithmetic methods**: `__add__`, `__sub__`, `__mul__`, `__truediv__`, `__floordiv__`, `__mod__`

Magic methods enable **operator overloading**, **type customization**, and **Python-native behavior** for custom objects.

---

## Magic Methods Cheat Sheet (Part 1)

| Category | Method | Operator/Call | Purpose |
| :-- | :-- | :-- | :-- |
| **Lifecycle** | `__new__` | `Class()` | Object creation |
|  | `__init__` | `Class()` | Object initialization |
|  | `__del__` | `del obj` | Object destruction |
| **Representation** | `__str__` | `str()`, `print()` | User string |
|  | `__repr__` | `repr()`, interactive | Developer string |
|  | `__format__` | `format()`, `{}` | Custom formatting |
| **Comparison** | `__eq__` | `==` | Equal |
|  | `__ne__` | `!=` | Not equal |
|  | `__lt__` | `<` | Less than |
|  | `__gt__` | `>` | Greater than |
|  | `__le__` | `\<=` | Less or equal |
|  | `__ge__` | `\>=` | Greater or equal |
| **Arithmetic** | `__add__` | `+` | Addition |
|  | `__sub__` | `-` | Subtraction |
|  | `__mul__` | `*` | Multiplication |
|  | `__truediv__` | `/` | True division |
|  | `__floordiv__` | `//` | Floor division |
|  | `__mod__` | `%` | Modulo |

**Key Rules:**

- Return `NotImplemented` for unsupported types
- `__init__` must return `None`
- `__new__` must return instance
- Use context managers instead of `__del__`
- `__repr__` should be unambiguous
- `__str__` should be human-readable

```
```

