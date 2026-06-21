<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" style="height:64px;margin-right:32px"/>

# Python OOP Fundamentals – Part 2

## Table of Contents

1. [Introduction](#introduction)
2. [Encapsulation](#encapsulation)
    - [What is Encapsulation?](#what-is-encapsulation)
    - [Why Encapsulation Exists](#why-encapsulation-exists)
    - [Real-World Analogy](#real-world-analogy-encapsulation)
    - [How Python Implements Encapsulation](#how-python-implements-encapsulation)
    - [Public Members](#public-members)
    - [Protected Members](#protected-members)
    - [Private Members](#private-members)
    - [Name Mangling](#name-mangling)
    - [Getters and Setters](#getters-and-setters)
    - [Property Decorator](#property-decorator)
    - [Data Hiding](#data-hiding)
    - [Encapsulation Patterns](#encapsulation-patterns)
    - [Read-Only Properties](#read-only-properties)
    - [Computed Properties](#computed-properties)
    - [Validation Patterns](#validation-patterns)
    - [Edge Cases](#edge-cases-encapsulation)
    - [Common Mistakes](#common-mistakes-encapsulation)
    - [Best Practices](#best-practices-encapsulation)
    - [Performance Considerations](#performance-considerations-encapsulation)
3. [Abstraction](#abstraction)
    - [What is Abstraction?](#what-is-abstraction)
    - [Why Abstraction Matters](#why-abstraction-matters)
    - [Real-World Analogy](#real-world-analogy-abstraction)
    - [Abstract Base Classes (ABCs)](#abstract-base-classes-abcs)
    - [abc Module](#abc-module)
    - [abstractmethod](#abstractmethod)
    - [ABCMeta](#abcmeta)
    - [Interface-like Design](#interface-like-design)
    - [Contract-Based Programming](#contract-based-programming)
    - [Multiple Inheritance with ABCs](#multiple-inheritance-with-abcs)
    - [Polymorphic Contracts](#polymorphic-contracts)
    - [Plugin Architecture](#plugin-architecture)
    - [Framework Design Concepts](#framework-design-concepts)
    - [Dependency Inversion Basics](#dependency-inversion-basics)
    - [Edge Cases](#edge-cases-abstraction)
    - [Common Mistakes](#common-mistakes-abstraction)
    - [Best Practices](#best-practices-abstraction)
    - [Performance Considerations](#performance-considerations-abstraction)
4. [Encapsulation vs Abstraction](#encapsulation-vs-abstraction)
5. [Common OOP Mistakes](#common-oop-mistakes)
6. [Best Practices](#best-practices-overall)
7. [Performance Considerations](#performance-considerations-overall)
8. [Real-world OOP Design Examples](#real-world-oop-design-examples)
9. [Coding Exercises](#coding-exercises)
10. [Interview Questions](#interview-questions)
11. [Summary](#summary)
12. [Cheat Sheet](#cheat-sheet)
13. [Practice Problems](#practice-problems)

***

## Introduction

Python's object-oriented programming (OOP) paradigm revolves around four core principles: **Encapsulation**, **Abstraction**, **Inheritance**, and **Polymorphism**. This guide dives deep into **Encapsulation** and **Abstraction** — two foundational concepts that enable robust, maintainable, and scalable software design.

Whether you're a beginner learning class structures or an advanced developer designing frameworks, mastering these concepts will elevate your Python expertise.

***

## Encapsulation

### What is Encapsulation?

**Encapsulation** is the mechanism of bundling data (attributes) and methods (functions) that operate on that data within a single unit (class), while restricting direct access to some of the object's components.

In Python, encapsulation is implemented through **conventions** rather than strict enforcement:

- `public` members: accessible from anywhere
- `_protected` members: intended for internal/use within subclass
- `__private` members: intended for internal use only


### Why Encapsulation Exists

| Purpose | Explanation |
| :-- | :-- |
| **Data Protection** | Prevents unintended modification of internal state |
| **Code Organization** | Groups related data and behavior together |
| **Maintainability** | Isolates internal implementation details |
| **Controlled Access** | Provides validated interfaces for data modification |
| **Encourages Good Design** | Promotes separation of concerns |

### Real-World Analogy (Encapsulation)

Think of a **car**:

- The **engine** (internal data) is hidden under the hood
- You interact via **controls** (steering wheel, pedals, ignition) — these are the public methods
- You can't directly access engine components (private data)
- The car validates your actions (e.g., you can't shift to reverse while accelerating)

```
┌─────────────────────────────┐
│         CAR (Class)         │
│  ┌─────────────────────┐   │
│  │   ENGINE (Private)  │   │
│  │  - fuelLevel        │   │
│  │  - temperature      │   │
│  └─────────────────────┘   │
│                             │
│  Public Methods:            │
│  - start()                  │
│  - accelerate()             │
│  - brake()                  │
│  - turnSteering(angle)      │
└─────────────────────────────┘
```


### How Python Implements Encapsulation

Python uses **name conventions** rather than access modifiers:


| Convention | Syntax | Meaning |
| :-- | :-- | :-- |
| Public | `attribute` | Accessible from anywhere |
| Protected | `_attribute` | Internal use, "don't touch" hint |
| Private | `__attribute` | Name-mangled, harder to access |

**Key Point**: Python doesn't enforce true privacy. It's a **community convention** that trusts developers to respect the design.

```python
class Example:
    public_var = "I'm public"      # Anyone can access
    _protected_var = "Protected"   # Subclasses/internal use
    __private_var = "Private"      # Name-mangled
```


### Public Members

**Definition**: Attributes and methods accessible from anywhere.

**Syntax**: No prefix

```python
class Person:
    def __init__(self, name):
        self.name = name  # Public attribute
    
    def greet(self):  # Public method
        return f"Hello, {self.name}"

# Usage
p = Person("Alice")
print(p.name)        # ✅ Accessible: "Alice"
print(p.greet())     # ✅ Accessible: "Hello, Alice"
```

**Memory Behavior**: Stored directly in the instance's `__dict__`

```python
print(p.__dict__)  # {'name': 'Alice'}
```


### Protected Members

**Definition**: Attributes/methods intended for internal use or subclass access. Python treats this as a **convention** ("don't access directly").

**Syntax**: Single underscore prefix `_`

```python
class BankAccount:
    def __init__(self, balance):
        self._balance = balance  # Protected attribute
    
    def _validate_amount(self, amount):  # Protected method
        return amount > 0
    
    def deposit(self, amount):
        if self._validate_amount(amount):
            self._balance += amount
            return True
        return False

# Usage
account = BankAccount(100)
print(account._balance)  # ⚠️ Accessible but discouraged
```

**Why Single Underscore?**

- Signals "internal use" to other developers
- Won't be imported by `from module import *`
- Subclasses are expected to access these

**Memory Behavior**: Same as public — stored in `__dict__`

```python
print(account.__dict__)  # {'_balance': 100}
```


### Private Members

**Definition**: Attributes/methods intended for internal class use only. Python implements this via **name mangling**.

**Syntax**: Double underscore prefix `__`

```python
class SecureData:
    def __init__(self):
        self.__secret = "Top Secret"  # Private attribute
    
    def __internal_method(self):  # Private method
        return "Internal"
    
    def access_secret(self):
        return self.__secret

# Usage
data = SecureData()
print(data.__secret)  # ❌ AttributeError!
print(data.access_secret())  # ✅ "Top Secret"
```

**Name Mangling Internals**:
Python transforms `__secret` to `_SecureData__secret`

```python
print(data._SecureData__secret)  # ✅ Accessible (but discouraged)
```

**Memory Behavior**: Name-mangled in `__dict__`

```python
print(data.__dict__)  # {'_SecureData__secret': 'Top Secret'}
```


### Name Mangling

**Definition**: Python's mechanism to make "private" attributes harder to access by prefixing with `_ClassName`.

**Internal Working**:

```python
class Parent:
    def __init__(self):
        self.__value = 42

class Child(Parent):
    def show(self):
        # Can't access __value directly
        return self.__value  # ❌ AttributeError

c = Child()
print(c.show())
```

**Why Name Mangling Exists**:

- Prevents accidental override in subclasses
- Makes private attributes "class-specific"
- Not true security — still accessible via mangled name

**Demonstration**:

```python
class Example:
    def __init__(self):
        self.__private = "hidden"

e = Example()
print(e.__dict__)  # {'_Example__private': 'hidden'}
print(e._Example__private)  # ✅ "hidden"
```


### Getters and Setters

**Traditional Approach** (before `@property`):

```python
class Person:
    def __init__(self, name):
        self._name = name
    
    def get_name(self):  # Getter
        return self._name
    
    def set_name(self, name):  # Setter
        if name and len(name) > 0:
            self._name = name
        else:
            raise ValueError("Name must be non-empty")

# Usage
p = Person("Alice")
print(p.get_name())  # ✅ "Alice"
p.set_name("Bob")    # ✅ Sets name
```

**Problems with this approach**:

- Verbose syntax (`get_name()` vs `name`)
- Breaks attribute-like access pattern
- Less Pythonic


### Property Decorator

**Definition**: Python's `@property` decorator provides a **Pythonic** way to implement getters, setters, and deleters while maintaining attribute-like syntax.

**Syntax**:

```python
@property
def attribute(self):
    # Getter

@attribute.setter
def attribute(self, value):
    # Setter

@attribute.deleter
def attribute(self):
    # Deleter
```

**Complete Example**:

```python
class Person:
    def __init__(self, name, age):
        self._name = name
        self._age = age
    
    @property
    def name(self):  # Getter
        return self._name
    
    @name.setter
    def name(self, value):  # Setter
        if not value or len(value) < 2:
            raise ValueError("Name must be at least 2 characters")
        self._name = value
    
    @property
    def age(self):
        return self._age
    
    @age.setter
    def age(self, value):
        if value < 0 or value > 150:
            raise ValueError("Age must be between 0 and 150")
        self._age = value
    
    @property
    def birth_year(self):  # Read-only computed property
        from datetime import datetime
        return datetime.now().year - self._age

# Usage
p = Person("Alice", 30)
print(p.name)      # ✅ Getter: "Alice" (no .name())
p.name = "Bob"     # ✅ Setter (no .set_name())
print(p.age)       # ✅ 30
p.age = 31         # ✅ Validation occurs
print(p.birth_year)  # ✅ Computed: read-only
```


### Data Hiding

**Definition**: Restricting direct access to internal data to prevent unintended modification.

**Implementation Strategies**:


| Strategy | Example | Protection Level |
| :-- | :-- | :-- |
| Protected | `_data` | Conventional |
| Private | `__data` | Name-mangled |
| Property | `@property` | Validation + controlled access |
| Read-only | No setter | Full protection |

**Example with Data Hiding**:

```python
class Temperature:
    def __init__(self, celsius):
        self.__celsius = celsius  # Hidden data
    
    @property
    def celsius(self):
        return self.__celsius
    
    @celsius.setter
    def celsius(self, value):
        if value < -273.15:
            raise ValueError("Cannot be below absolute zero")
        self.__celsius = value
    
    @property
    def fahrenheit(self):
        return (self.__celsius * 9/5) + 32

# Usage
t = Temperature(25)
print(t.celsius)     # ✅ 25
t.celsius = 30       # ✅ Validated
print(t.fahrenheit)  # ✅ 86.0
t.celsius = -300     # ❌ ValueError
```


### Encapsulation Patterns

#### Pattern 1: Validation in Setter

```python
class Email:
    def __init__(self, address):
        self._validate(address)
        self.__address = address
    
    @property
    def address(self):
        return self.__address
    
    def _validate(self, address):
        if not address or '@' not in address:
            raise ValueError("Invalid email format")

# Usage
e = Email("user@example.com")  # ✅
e = Email("invalid")           # ❌ ValueError
```


#### Pattern 2: Immutable Properties

```python
class Point:
    def __init__(self, x, y):
        self.__x = x
        self.__y = y
    
    @property
    def x(self):
        return self.__x
    
    @property
    def y(self):
        return self.__y

# Usage
p = Point(10, 20)
print(p.x)  # ✅ 10
p.x = 15    # ❌ AttributeError (no setter)
```


#### Pattern 3: Lazy Computation

```python
class DataLoader:
    def __init__(self, filepath):
        self.__filepath = filepath
        self.__data = None
    
    @property
    def data(self):
        if self.__data is None:
            self.__data = self._load()
        return self.__data
    
    def _load(self):
        print(f"Loading from {self.__filepath}")
        return {"loaded": True}

# Usage
loader = DataLoader("file.txt")
print(loader.data)  # ✅ Loads on first access
print(loader.data)  # ✅ Reuses cached data
```


### Read-Only Properties

**Definition**: Properties with only a getter (no setter), making them immutable after initialization.

```python
class Circle:
    def __init__(self, radius):
        self.__radius = radius
    
    @property
    def radius(self):
        return self.__radius
    
    @property
    def area(self):
        return 3.14159 * self.__radius ** 2
    
    @property
    def circumference(self):
        return 2 * 3.14159 * self.__radius

# Usage
c = Circle(5)
print(c.radius)     # ✅ 5
print(c.area)       # ✅ 78.53975
c.radius = 10       # ❌ AttributeError (no setter)
```


### Computed Properties

**Definition**: Properties that calculate and return values based on other attributes.

```python
class Rectangle:
    def __init__(self, width, height):
        self.__width = width
        self.__height = height
    
    @property
    def width(self):
        return self.__width
    
    @width.setter
    def width(self, value):
        if value <= 0:
            raise ValueError("Width must be positive")
        self.__width = value
    
    @property
    def height(self):
        return self.__height
    
    @height.setter
    def height(self, value):
        if value <= 0:
            raise ValueError("Height must be positive")
        self.__height = value
    
    @property
    def area(self):
        return self.__width * self.__height
    
    @property
    def perimeter(self):
        return 2 * (self.__width + self.__height)
    
    @property
    def is_square(self):
        return self.__width == self.__height

# Usage
r = Rectangle(4, 5)
print(r.area)        # ✅ 20
print(r.perimeter)   # ✅ 18
print(r.is_square)   # ✅ False
r.width = 5          # ✅ Updates computed properties
print(r.is_square)   # ✅ True
```


### Validation Patterns

#### Pattern 1: Type Validation

```python
class TypedContainer:
    def __init__(self, value, expected_type):
        self._validate_type(value, expected_type)
        self.__value = value
        self.__type = expected_type
    
    @property
    def value(self):
        return self.__value
    
    @value.setter
    def value(self, new_value):
        self._validate_type(new_value, self.__type)
        self.__value = new_value
    
    def _validate_type(self, value, expected_type):
        if not isinstance(value, expected_type):
            raise TypeError(f"Expected {expected_type}, got {type(value)}")

# Usage
c = TypedContainer(42, int)
print(c.value)      # ✅ 42
c.value = "hello"   # ❌ TypeError
c.value = 100       # ✅ 100
```


#### Pattern 2: Range Validation

```python
class Percentage:
    def __init__(self, value):
        self.value = value  # Uses setter for validation
    
    @property
    def value(self):
        return self.__value
    
    @value.setter
    def value(self, v):
        if not 0 <= v <= 100:
            raise ValueError("Percentage must be 0-100")
        self.__value = v

# Usage
p = Percentage(50)
print(p.value)      # ✅ 50
p.value = 75        # ✅ 75
p.value = -10       # ❌ ValueError
p.value = 150       # ❌ ValueError
```


#### Pattern 3: Format Validation

```python
class PhoneNumber:
    def __init__(self, number):
        self.number = number
    
    @property
    def number(self):
        return self.__number
    
    @number.setter
    def number(self, value):
        import re
        if not re.match(r'^\+?\d{10,15}$', value):
            raise ValueError("Invalid phone number format")
        self.__number = value

# Usage
pn = PhoneNumber("+1234567890")  # ✅
pn = PhoneNumber("123")          # ❌ ValueError
```


### Edge Cases (Encapsulation)

| Edge Case | Example | Behavior |
| :-- | :-- | :-- |
| Accessing private via mangled name | `obj._Class__attr` | ✅ Works (but discouraged) |
| Subclass overriding private | `class Child: __attr` | ✅ Creates new `_Child__attr` |
| Property with no getter | `@property` missing function | ❌ AttributeError |
| Setter without getter | `@attr.setter` only | ⚠️ Works but unusual |
| Deleting non-deletable | `del obj.attr` without deleter | ❌ AttributeError |
| Multiple properties same name | Conflicting `@property` | ❌ Overrides previous |

```python
# Subclass name mangling example
class Parent:
    def __init__(self):
        self.__value = "parent"

class Child(Parent):
    def __init__(self):
        super().__init__()
        self.__value = "child"  # Different mangled name
    
    def show(self):
        return self.__value

c = Child()
print(c.show())  # ✅ "child" (not "parent")
```


### Common Mistakes (Encapsulation)

| Mistake | Bad Code | Correct Code |
| :-- | :-- | :-- |
| **True privacy expectation** | `self.__attr` expects complete blocking | Understand it's name-mangled, not true privacy |
| **Over-using private** | Everything is `__private` | Use `_protected` for subclass access |
| **Missing validation** | Setter without checks | Always validate in setters |
| **Breaking attribute syntax** | `get_name()` instead of `@property` | Use `@property` for Pythonic access |
| **Public mutable attributes** | `self.data = []` exposed | Use `_data` or property with validation |
| **Ignoring computed properties** | Manual calculation everywhere | Use `@property` for computed values |

```python
# Bad: No validation
class BadPerson:
    def __init__(self, age):
        self.age = age  # Anyone can set negative age

# Good: Validation
class GoodPerson:
    def __init__(self, age):
        self.age = age
    
    @property
    def age(self):
        return self.__age
    
    @age.setter
    def age(self, value):
        if value < 0:
            raise ValueError("Age must be positive")
        self.__age = value
```


### Best Practices (Encapsulation)

1. **Use `_protected` for internal attributes** — signals intent without over-restricting
2. **Use `@property` for validated access** — maintains attribute syntax with control
3. **Make computed attributes read-only** — no setter for calculated properties
4. **Validate in setters, not just constructors** — maintain integrity throughout lifetime
5. **Don't over-engineer privacy** — Python trusts developers; use conventions
6. **Document protected members** — clarify what's internal vs. subclass API
7. **Prefer properties over get/set methods** — more Pythonic
8. **Use private for class-internal-only data** — when subclasses shouldn't access
```python
# Best practice example
class UserProfile:
    def __init__(self, username, email):
        self._username = username  # Protected for subclasses
        self.__email = email       # Private, class-internal
    
    @property
    def username(self):
        return self._username
    
    @username.setter
    def username(self, value):
        if len(value) < 3:
            raise ValueError("Username too short")
        self._username = value
    
    @property
    def email(self):
        return self.__email
    
    @email.setter
    def email(self, value):
        if '@' not in value:
            raise ValueError("Invalid email")
        self.__email = value
    
    @property
    def display_name(self):  # Read-only computed
        return f"{self._username} ({self.__email})"
```


### Performance Considerations (Encapsulation)

| Aspect | Impact | Optimization |
| :-- | :-- | :-- |
| **Property access** | Slightly slower than direct access | Use for validation; direct for performance-critical |
| **Name mangling** | No performance cost | Purely syntactic transformation |
| **Protected vs public** | No difference | Both stored in `__dict__` |
| **Lazy computation** | Reduces initial cost | Cache computed values |
| **Validation overhead** | Adds runtime cost | Validate once at construction if immutable |

```python
# Performance comparison
import timeit

class DirectAccess:
    def __init__(self):
        self.value = 42

class PropertyAccess:
    def __init__(self):
        self._value = 42
    
    @property
    def value(self):
        return self._value

direct = DirectAccess()
prop = PropertyAccess()

print(timeit.timeit(lambda: direct.value, number=1000000))  # Faster
print(timeit.timeit(lambda: prop.value, number=1000000))    # ~10-20% slower
```

**When to Avoid Properties**:

- High-frequency access in tight loops
- Simple data containers without validation
- Performance-critical paths

***

## Abstraction

### What is Abstraction?

**Abstraction** is the mechanism of hiding complex implementation details while exposing only essential features. It allows you to focus on **what** an object does rather than **how** it does it.

In Python, abstraction is implemented through **Abstract Base Classes (ABCs)** using the `abc` module.

### Why Abstraction Matters

| Benefit | Explanation |
| :-- | :-- |
| **Simplifies Complexity** | Hide implementation details, expose interface |
| **Enforces Contracts** | Ensure classes implement required methods |
| **Facilitates Polymorphism** | Different objects respond to same interface |
| **Enables Plugin Architecture** | Define interfaces for extensible systems |
| **Supports Framework Design** | Create base classes for extension |
| **Improves Testability** | Mock abstract interfaces for testing |

### Real-World Analogy (Abstraction)

Think of a **remote control**:

- You press buttons (**interface**: `power()`, `volume_up()`)
- You don't know how it sends signals to the TV (**implementation hidden**)
- Any TV works with the remote (**polymorphism**: different implementations, same interface)

```
┌─────────────────────────────┐
│    REMOTE CONTROL (ABC)     │
│                             │
│  Abstract Methods:          │
│  - power()                  │
│  - volume_up()              │
│  - volume_down()            │
│  - set_channel(n)           │
└─────────────────────────────┘
        ▲           ▲           ▲
        │           │           │
┌───────┴───────┐ ┌─┴─────────┐ ┌─┴──────────┐
│ SamsungRemote │ │LGRemote   │ │SonyRemote  │
│ (Concrete)    │ │(Concrete) │ │(Concrete)  │
│ - implements  │ │- implements│ │- implements│
│   all methods │ │  all methods││  all methods│
└───────────────┘ └───────────┘ └────────────┘
```


### Abstract Base Classes (ABCs)

**Definition**: Classes that cannot be instantiated directly and define methods that subclasses must implement.

**Key Characteristics**:

- Contains at least one `@abstractmethod`
- Cannot be instantiated without implementing all abstract methods
- Provides a **contract** for subclasses

```python
from abc import ABC, abstractmethod

class Shape(ABC):  # Abstract base class
    @abstractmethod
    def area(self):
        pass
    
    @abstractmethod
    def perimeter(self):
        pass
    
    def describe(self):  # Regular method (not abstract)
        return f"Area: {self.area()}, Perimeter: {self.perimeter()}"

# Cannot instantiate
# shape = Shape()  # ❌ TypeError

class Rectangle(Shape):  # Concrete subclass
    def __init__(self, width, height):
        self.width = width
        self.height = height
    
    def area(self):  # Must implement
        return self.width * self.height
    
    def perimeter(self):  # Must implement
        return 2 * (self.width + self.height)

# Can instantiate
rect = Rectangle(5, 10)
print(rect.area())       # ✅ 50
print(rect.describe())   # ✅ "Area: 50, Perimeter: 30"
```


### abc Module

**Definition**: Python's `abc` (Abstract Base Classes) module provides infrastructure for defining abstract base classes.

**Key Components**:


| Component | Purpose |
| :-- | :-- |
| `ABC` | Base class for abstract classes |
| `abstractmethod` | Decorator for abstract methods |
| `abstractproperty` | Deprecated (use `@property` + `@abstractmethod`) |
| `ABCMeta` | Metaclass for ABCs |

```python
from abc import ABC, abstractmethod, abstractproperty

# Modern approach
class Database(ABC):
    @abstractmethod
    def connect(self):
        pass
    
    @abstractmethod
    def disconnect(self):
        pass
    
    @property
    @abstractmethod
    def is_connected(self):
        pass
```


### abstractmethod

**Definition**: Decorator that marks a method as abstract — subclasses must implement it.

**Syntax**:

```python
@abstractmethod
def method_name(self, params):
    pass
```

**Behavior**:

- Raises `TypeError` if called on abstract class
- Subclass must override all abstract methods
- Can have implementation in abstract class (but still must be overridden)

```python
from abc import ABC, abstractmethod

class Animal(ABC):
    @abstractmethod
    def speak(self):
        """All animals must speak"""
        pass
    
    @abstractmethod
    def move(self):
        pass

class Dog(Animal):
    def speak(self):
        return "Woof!"
    
    def move(self):
        return "Walking on four legs"

class Bird(Animal):
    def speak(self):
        return "Chirp!"
    
    def move(self):
        return "Flying"

# Usage
dog = Dog()
bird = Bird()
print(dog.speak())  # ✅ "Woof!"
print(bird.move())  # ✅ "Flying"
```


### ABCMeta

**Definition**: The metaclass used by ABCs to enforce abstract method implementation.

**How it Works**:

- `ABC` uses `ABCMeta` as its metaclass
- `ABCMeta` checks for `@abstractmethod` during class creation
- Prevents instantiation if abstract methods aren't implemented

```python
from abc import ABC, abstractmethod, ABCMeta

# Explicit ABCMeta usage (rare)
class MyAbstract(ABCMeta):
    pass

class Shape(ABC):  # Uses ABCMeta internally
    @abstractmethod
    def area(self):
        pass

class IncompleteShape(Shape):
    # Missing area() implementation
    pass

# Cannot instantiate
# incomplete = IncompleteShape()  # ❌ TypeError: unimplemented abstract methods
```


### Interface-like Design

**Definition**: Using ABCs to define interfaces (contracts) without implementation — similar to interfaces in Java/C\#.

```python
from abc import ABC, abstractmethod

class PaymentProcessor(ABC):
    """Interface for payment processors"""
    
    @abstractmethod
    def process_payment(self, amount):
        pass
    
    @abstractmethod
    def refund(self, transaction_id, amount):
        pass
    
    @abstractmethod
    def get_balance(self):
        pass

class CreditCardProcessor(PaymentProcessor):
    def process_payment(self, amount):
        return f"Processing ${amount} via Credit Card"
    
    def refund(self, transaction_id, amount):
        return f"Refunding ${amount} to transaction {transaction_id}"
    
    def get_balance(self):
        return "$10,000 available"

class PayPalProcessor(PaymentProcessor):
    def process_payment(self, amount):
        return f"Processing ${amount} via PayPal"
    
    def refund(self, transaction_id, amount):
        return f"PayPal refund: ${amount} to {transaction_id}"
    
    def get_balance(self):
        return "$50,000 available"

# Usage (Polymorphism)
def checkout(processor: PaymentProcessor, amount):
    print(processor.process_payment(amount))
    print(f"Balance: {processor.get_balance()}")

checkout(CreditCardProcessor(), 100)  # ✅
checkout(PayPalProcessor(), 200)      # ✅
```


### Contract-Based Programming

**Definition**: Programming style where classes adhere to defined contracts (interfaces) ensuring predictable behavior.

```python
from abc import ABC, abstractmethod

class Serializer(ABC):
    """Contract for data serialization"""
    
    @abstractmethod
    def serialize(self, data):
        """Convert data to string format"""
        pass
    
    @abstractmethod
    def deserialize(self, data_string):
        """Convert string back to data"""
        pass
    
    @property
    @abstractmethod
    def format(self):
        """Return serialization format name"""
        pass

class JSONSerializer(Serializer):
    import json
    
    def serialize(self, data):
        return json.dumps(data)
    
    def deserialize(self, data_string):
        return json.loads(data_string)
    
    @property
    def format(self):
        return "JSON"

class XMLSerializer(Serializer):
    def serialize(self, data):
        # Simplified XML
        return f"<data>{data}</data>"
    
    def deserialize(self, data_string):
        # Simplified parsing
        return data_string.strip("<data>").strip("</data>")
    
    @property
    def format(self):
        return "XML"

# Usage
def save_data(serializer: Serializer, data):
    serialized = serializer.serialize(data)
    print(f"Saving as {serializer.format}: {serialized}")

save_data(JSONSerializer(), {"name": "Alice"})  # ✅
save_data(XMLSerializer(), "hello")             # ✅
```


### Multiple Inheritance with ABCs

**Definition**: Combining multiple abstract base classes to create more specific contracts.

```python
from abc import ABC, abstractmethod

class Drawable(ABC):
    @abstractmethod
    def draw(self):
        pass
    
    @abstractmethod
    def get_bounds(self):
        pass

class Clickable(ABC):
    @abstractmethod
    def on_click(self):
        pass
    
    @abstractmethod
    def is_clickable(self):
        pass

class Button(Drawable, Clickable):  # Multiple ABC inheritance
    def __init__(self, x, y, width, height, label):
        self.x = x
        self.y = y
        self.width = width
        self.height = height
        self.label = label
        self.clickable = True
    
    def draw(self):
        return f"Drawing button: {self.label}"
    
    def get_bounds(self):
        return (self.x, self.y, self.width, self.height)
    
    def on_click(self):
        return f"Button clicked: {self.label}"
    
    def is_clickable(self):
        return self.clickable

# Usage
btn = Button(10, 20, 100, 50, "Submit")
print(btn.draw())           # ✅
print(btn.on_click())       # ✅
print(btn.get_bounds())     # ✅ (10, 20, 100, 50)
```


### Polymorphic Contracts

**Definition**: Different objects implementing the same abstract interface, allowing interchangeable usage.

```python
from abc import ABC, abstractmethod

class NotificationService(ABC):
    @abstractmethod
    def send(self, message, recipient):
        pass

class EmailService(NotificationService):
    def send(self, message, recipient):
        return f"Email to {recipient}: {message}"

class SMSService(NotificationService):
    def send(self, message, recipient):
        return f"SMS to {recipient}: {message}"

class PushService(NotificationService):
    def send(self, message, recipient):
        return f"Push to {recipient}: {message}"

# Polymorphic usage
def notify_all(service: NotificationService, message, recipients):
    for recipient in recipients:
        print(service.send(message, recipient))

services = [EmailService(), SMSService(), PushService()]
notify_all(services[0], "Hello!", ["alice@email.com"])  # ✅
notify_all(services[1], "Hello!", ["+1234567890"])      # ✅
notify_all(services[2], "Hello!", ["user123"])          # ✅
```


### Plugin Architecture

**Definition**: Using ABCs to define plugin interfaces that external modules can implement.

```python
from abc import ABC, abstractmethod

class Plugin(ABC):
    """Base interface for all plugins"""
    
    @property
    @abstractmethod
    def name(self):
        pass
    
    @property
    @abstractmethod
    def version(self):
        pass
    
    @abstractmethod
    def initialize(self):
        pass
    
    @abstractmethod
    def execute(self, context):
        pass
    
    @abstractmethod
    def cleanup(self):
        pass

class DataExportPlugin(Plugin):
    name = "Data Export"
    version = "1.0.0"
    
    def initialize(self):
        print("DataExportPlugin: Initializing")
    
    def execute(self, context):
        print(f"DataExportPlugin: Exporting {context.get('data')}")
    
    def cleanup(self):
        print("DataExportPlugin: Cleaning up")

class LoggingPlugin(Plugin):
    name = "Logging"
    version = "2.1.0"
    
    def initialize(self):
        print("LoggingPlugin: Setting up logger")
    
    def execute(self, context):
        print(f"LoggingPlugin: Log entry - {context}")
    
    def cleanup(self):
        print("LoggingPlugin: Closing logger")

# Plugin Manager
class PluginManager:
    def __init__(self):
        self.plugins = []
    
    def register(self, plugin: Plugin):
        if not isinstance(plugin, Plugin):
            raise TypeError("Plugin must implement Plugin interface")
        self.plugins.append(plugin)
    
    def run(self, context):
        for plugin in self.plugins:
            plugin.initialize()
            plugin.execute(context)
            plugin.cleanup()

# Usage
manager = PluginManager()
manager.register(DataExportPlugin())
manager.register(LoggingPlugin())
manager.run({"data": "user_records"})
```


### Framework Design Concepts

**Definition**: Using ABCs to create extensible framework base classes that developers extend.

```python
from abc import ABC, abstractmethod

class Middleware(ABC):
    """Framework middleware interface"""
    
    @abstractmethod
    def process_request(self, request):
        """Modify request before reaching handler"""
        pass
    
    @abstractmethod
    def process_response(self, response):
        """Modify response before sending to client"""
        pass

class AuthenticationMiddleware(Middleware):
    def process_request(self, request):
        if not request.get('token'):
            request['error'] = "Missing token"
        return request
    
    def process_response(self, response):
        response['authenticated'] = True
        return response

class LoggingMiddleware(Middleware):
    def process_request(self, request):
        print(f"Request: {request}")
        return request
    
    def process_response(self, response):
        print(f"Response: {response}")
        return response

class Framework:
    def __init__(self):
        self.middleware = []
    
    def add_middleware(self, mw: Middleware):
        self.middleware.append(mw)
    
    def handle(self, request):
        # Process through middleware
        for mw in self.middleware:
            request = mw.process_request(request)
        
        # Handle request
        response = {"status": "ok", "data": "response"}
        
        # Process response through middleware
        for mw in self.middleware:
            response = mw.process_response(response)
        
        return response

# Usage
app = Framework()
app.add_middleware(AuthenticationMiddleware())
app.add_middleware(LoggingMiddleware())
result = app.handle({"token": "abc123", "user": "alice"})
print(result)  # ✅
```


### Dependency Inversion Basics

**Definition**: Using ABCs to invert dependencies — high-level modules depend on abstract interfaces, not concrete implementations.

```python
from abc import ABC, abstractmethod

class DatabaseConnection(ABC):
    """Abstract database interface"""
    
    @abstractmethod
    def connect(self):
        pass
    
    @abstractmethod
    def query(self, sql):
        pass
    
    @abstractmethod
    def disconnect(self):
        pass

class MySQLConnection(DatabaseConnection):
    def connect(self):
        print("Connecting to MySQL")
    
    def query(self, sql):
        return f"MySQL result: {sql}"
    
    def disconnect(self):
        print("Disconnecting from MySQL")

class PostgreSQLConnection(DatabaseConnection):
    def connect(self):
        print("Connecting to PostgreSQL")
    
    def query(self, sql):
        return f"PostgreSQL result: {sql}"
    
    def disconnect(self):
        print("Disconnecting from PostgreSQL")

class UserService:
    """High-level module - depends on abstract interface"""
    
    def __init__(self, db: DatabaseConnection):
        self.db = db  # Dependency injected
    
    def get_user(self, user_id):
        self.db.connect()
        result = self.db.query(f"SELECT * FROM users WHERE id={user_id}")
        self.db.disconnect()
        return result

# Usage (Dependency Injection)
mysql_db = MySQLConnection()
postgres_db = PostgreSQLConnection()

user_service_mysql = UserService(mysql_db)
user_service_postgres = UserService(postgres_db)

print(user_service_mysql.get_user(1))   # ✅
print(user_service_postgres.get_user(1)) # ✅
```


### Edge Cases (Abstraction)

| Edge Case | Example | Behavior |
| :-- | :-- | :-- |
| Unimplemented abstract method | Subclass missing `@abstractmethod` | ❌ Cannot instantiate |
| Abstract method with implementation | ABC has body for abstract method | ✅ Must still override in subclass |
| Multiple abstract methods | Class with 5 `@abstractmethod` | ✅ All must be implemented |
| Abstract property | `@property` + `@abstractmethod` | ✅ Works correctly |
| Abstract class inheriting ABC | `class Child(Abstract)` | ✅ if all methods implemented |
| Calling abstract method | Direct call on ABC | ❌ Raises `TypeError` |

```python
from abc import ABC, abstractmethod

class BadSubclass(ABC):
    @abstractmethod
    def method(self):
        pass

class Incomplete(BadSubclass):
    # Missing method() implementation
    pass

# class Complete(BadSubclass):
#     def method(self):
#         return "done"

# incomplete = Incomplete()  # ❌ TypeError
# complete = Complete()       # ✅ Works
```


### Common Mistakes (Abstraction)

| Mistake | Bad Code | Correct Code |
| :-- | :-- | :-- |
| **Instantiating ABC** | `obj = AbstractClass()` | ABCs are for subclassing only |
| **Missing @abstractmethod** | `def method(self): pass` without decorator | Add `@abstractmethod` |
| **Not implementing all methods** | Subclass skips one abstract method | Implement all required methods |
| **Over-using ABCs** | Making everything abstract | Use ABCs only for true interfaces |
| **Abstract with full implementation** | ABC provides complete logic | ABC should define contract, not implementation |
| **Ignoring abstract properties** | Using regular `@property` for interface | Use `@property` + `@abstractmethod` |

```python
# Bad: Can instantiate ABC
class Wrong(ABC):
    def method(self):  # Missing @abstractmethod
        return "ok"

wrong = Wrong()  # ❌ Should not be allowed

# Good: Proper ABC
class Right(ABC):
    @abstractmethod
    def method(self):
        pass

class Correct(Right):
    def method(self):
        return "ok"

correct = Correct()  # ✅
```


### Best Practices (Abstraction)

1. **Use ABCs for true interfaces** — when you need to enforce a contract
2. **Keep abstract methods minimal** — define only essential operations
3. **Provide default implementations** — for methods that can have common behavior
4. **Document abstract contracts** — clarify what subclasses must implement
5. **Use abstract properties for state** — when interface requires attributes
6. **Avoid deep ABC hierarchies** — keep abstraction layers shallow
7. **Combine with dependency injection** — for testable, flexible code
8. **Use for plugin systems** — define clear extension points
```python
# Best practice ABC
class Repository(ABC):
    """Abstract repository interface for data access"""
    
    @abstractmethod
    def get(self, id):
        """Retrieve item by ID"""
        pass
    
    @abstractmethod
    def save(self, item):
        """Save item to storage"""
        pass
    
    @abstractmethod
    def delete(self, id):
        """Delete item by ID"""
        pass
    
    def get_all(self):
        """Default implementation - subclasses can override"""
        raise NotImplementedError("Subclasses must implement get_all()")
```


### Performance Considerations (Abstraction)

| Aspect | Impact | Optimization |
| :-- | :-- | :-- |
| **ABC instantiation check** | Small overhead at class creation | Negligible in production |
| **Abstract method calls** | Same as regular methods | No performance difference |
| **Multiple ABC inheritance** | Slightly more complex MRO | Use for clarity, not performance |
| **Interface enforcement** | Runtime type checking | Validate at injection, not call |

```python
import timeit
from abc import ABC, abstractmethod

class AbstractService(ABC):
    @abstractmethod
    def process(self):
        pass

class ConcreteService(AbstractService):
    def process(self):
        return "done"

class DirectService:
    def process(self):
        return "done"

abstract_obj = ConcreteService()
direct_obj = DirectService()

print(timeit.timeit(lambda: abstract_obj.process(), number=1000000))  # Same speed
print(timeit.timeit(lambda: direct_obj.process(), number=1000000))    # No difference
```

**Key Insight**: Abstraction has **no runtime performance cost** — it's purely a design mechanism.

***

## Encapsulation vs Abstraction

### Detailed Comparison Table

| Aspect | Encapsulation | Abstraction |
| :-- | :-- | :-- |
| **Primary Goal** | Protect data, control access | Hide complexity, expose interface |
| **Focus** | **How** data is stored/modified | **What** operations are available |
| **Implementation** | Conventions (`_`, `__`), `@property` | ABCs, `@abstractmethod`, `abc` module |
| **Access Control** | Public, protected, private | Abstract vs concrete methods |
| **Enforcement** | Conventional (Python trusts developers) | Runtime enforcement (can't instantiate ABC) |
| **Level** | Class/internal level | System/architecture level |
| **Key Concept** | Data hiding, controlled access | Interface definition, contract |
| **Example** | `__private_attr`, `@property` | `class Shape(ABC)`, `@abstractmethod` |

### Differences

| Dimension | Encapsulation | Abstraction |
| :-- | :-- | :-- |
| **Purpose** | Secrete implementation details of data | Hide implementation details of behavior |
| **Mechanism** | Access modifiers (conventions) | Abstract base classes |
| **Scope** | Individual class attributes/methods | Whole class/interface design |
| **When to Use** | Protect internal state | Define extensible interfaces |
| **Python Feature** | `_`, `__`, `@property` | `ABC`, `@abstractmethod`, `abc` |

### Similarities

| Similarity | Explanation |
| :-- | :-- |
| **Hide Details** | Both hide implementation from users |
| **Simplify Usage** | Both provide cleaner interfaces |
| **Improve Maintainability** | Both isolate changes |
| **OOP Principles** | Both are core OOP concepts |
| **Design Goals** | Both support good software design |

### When to Use Each

| Scenario | Use Encapsulation | Use Abstraction |
| :-- | :-- | :-- |
| **Protecting data** | ✅ Yes | ❌ No |
| **Validating input** | ✅ Yes (via `@property`) | ❌ No |
| **Defining plugin interface** | ❌ No | ✅ Yes |
| **Creating framework base** | ❌ No | ✅ Yes |
| **Hiding internal helpers** | ✅ Yes (`_method`) | ❌ No |
| **Enforcing contracts** | ❌ No | ✅ Yes |
| **Making read-only properties** | ✅ Yes | ❌ No |
| **Multiple implementations** | ❌ No | ✅ Yes |

```python
# When to use Encapsulation
class BankAccount:
    def __init__(self, balance):
        self.__balance = balance  # Encapsulation: protect data
    
    @property
    def balance(self):
        return self.__balance  # Encapsulation: controlled access
    
    def deposit(self, amount):
        if amount > 0:  # Encapsulation: validation
            self.__balance += amount

# When to use Abstraction
class PaymentProcessor(ABC):
    @abstractmethod
    def process(self, amount):  # Abstraction: define contract
        pass

class CreditCardProcessor(PaymentProcessor):
    def process(self, amount):  # Abstraction: implement contract
        return f"Processed ${amount}"
```


***

## Common OOP Mistakes

| Mistake | Description | Fix |
| :-- | :-- | :-- |
| **Leaking internal state** | Exposing `__dict__` or mutable attributes | Use `@property` with validation |
| **Over-encapsulation** | Making everything `__private` | Use `_protected` for subclass access |
| **Under-encapsulation** | Everything is public | Protect sensitive data |
| **ABCs for everything** | Making simple classes abstract | Use ABCs only for true interfaces |
| **Ignoring abstract methods** | Subclass doesn't implement all methods | Implement all required methods |
| **Breaking encapsulation** | Accessing `__private` via mangled name | Respect conventions, don't bypass |
| **Too deep inheritance** | 5+ levels of class hierarchy | Use composition over inheritance |
| **Mixin confusion** | Mixing unrelated functionality | Keep mixins focused and single-purpose |
| **Ignoring `__init__`** | Not initializing attributes | Always define `__init__` |
| **Mutable default args** | `def __init__(self, items=[])` | Use `items=None` and create list |

```python
# Bad: Mutable default argument
class Bad:
    def __init__(self, items=[]):  # ❌ Shared list!
        self.items = items

# Good: Immutable default
class Good:
    def __init__(self, items=None):
        self.items = items if items is not None else []  # ✅
```


***

## Best Practices (Overall)

### Encapsulation Best Practices

1. Use `_protected` for internal attributes
2. Use `@property` for validated access
3. Make computed properties read-only
4. Validate in setters, not just constructors
5. Document what's internal vs. public API
6. Prefer properties over get/set methods
7. Use `__private` for class-internal-only data

### Abstraction Best Practices

1. Use ABCs for true interfaces
2. Keep abstract methods minimal
3. Provide default implementations where possible
4. Document abstract contracts clearly
5. Use abstract properties for required attributes
6. Avoid deep ABC hierarchies
7. Combine with dependency injection
8. Use for plugin systems and frameworks

### General OOP Best Practices

1. **Follow SOLID principles** — Single Responsibility, Open/Closed, etc.
2. **Prefer composition over inheritance** — more flexible
3. **Keep classes small and focused** — single responsibility
4. **Document public APIs** — clarify intended usage
5. **Test with mocks** — use abstractions for testing
6. **Avoid God objects** — classes doing too much
7. **Use type hints** — improve code clarity

***

## Performance Considerations (Overall)

| Concern | Impact | Mitigation |
| :-- | :-- | :-- |
| **Property access** | ~10-20% slower than direct | Use for validation; direct for hot paths |
| **ABC checks** | Negligible (class creation only) | No production impact |
| **Name mangling** | No cost | Purely syntactic |
| **Multiple inheritance** | Slight MRO complexity | Use for clarity, not speed |
| **Method resolution** | O(1) for direct, O(log n) for MRO | Generally negligible |
| **Validation overhead** | Adds per-call cost | Validate once at construction if immutable |

**When to Optimize**:

- Tight loops with frequent property access → use direct access
- High-frequency data access → minimize validation
- Performance-critical paths → avoid abstraction overhead

***

## Real-world OOP Design Examples

### Example 1: E-Commerce System

```python
from abc import ABC, abstractmethod

# Abstraction: Payment interface
class PaymentProvider(ABC):
    @abstractmethod
    def process_payment(self, amount, currency):
        pass
    
    @abstractmethod
    def refund(self, transaction_id, amount):
        pass

class StripePayment(PaymentProvider):
    def process_payment(self, amount, currency):
        return f"Stripe: ${amount} {currency} charged"
    
    def refund(self, transaction_id, amount):
        return f"Stripe: Refunded ${amount} to {transaction_id}"

class PayPalPayment(PaymentProvider):
    def process_payment(self, amount, currency):
        return f"PayPal: ${amount} {currency} charged"
    
    def refund(self, transaction_id, amount):
        return f"PayPal: Refunded ${amount} to {transaction_id}"

# Encapsulation: Order class
class Order:
    def __init__(self, order_id, items):
        self.__order_id = order_id
        self.__items = items
        self.__total = self._calculate_total()
        self.__payment_status = "pending"
        self.__transaction_id = None
    
    @property
    def order_id(self):
        return self.__order_id
    
    @property
    def items(self):
        return self.__items  # Read-only
    
    @property
    def total(self):
        return self.__total  # Computed, read-only
    
    @property
    def payment_status(self):
        return self.__payment_status
    
    def pay(self, provider: PaymentProvider, currency="USD"):
        if self.__payment_status != "pending":
            raise ValueError("Order already paid")
        
        result = provider.process_payment(self.__total, currency)
        self.__payment_status = "paid"
        self.__transaction_id = f"TXN-{self.__order_id}"
        return result
    
    def refund(self, provider: PaymentProvider, amount=None):
        if self.__payment_status != "paid":
            raise ValueError("Order not paid")
        
        amount = amount or self.__total
        result = provider.refund(self.__transaction_id, amount)
        self.__payment_status = "refunded"
        return result
    
    def _calculate_total(self):
        return sum(item['price'] * item['quantity'] for item in self.__items)

# Usage
order = Order("ORD-001", [
    {"name": "Laptop", "price": 999, "quantity": 1},
    {"name": "Mouse", "price": 25, "quantity": 2}
])

print(order.total)  # ✅ 1049 (computed, read-only)
print(order.pay(StripePayment()))  # ✅ Stripe payment
print(order.payment_status)  # ✅ "paid"
```


### Example 2: Logging Framework

```python
from abc import ABC, abstractmethod
from datetime import datetime

# Abstraction: Logger interface
class Logger(ABC):
    @abstractmethod
    def log(self, level, message):
        pass
    
    @abstractmethod
    def info(self, message):
        pass
    
    @abstractmethod
    def error(self, message):
        pass

# Encapsulation: FileLogger with internal state
class FileLogger(Logger):
    def __init__(self, filepath, min_level="INFO"):
        self.__filepath = filepath
        self.__min_level = min_level
        self.__levels = {"DEBUG": 0, "INFO": 1, "WARNING": 2, "ERROR": 3}
        self.__file_handle = None
    
    @property
    def filepath(self):
        return self.__filepath
    
    @property
    def min_level(self):
        return self.__min_level
    
    def log(self, level, message):
        if self.__levels[level] < self.__levels[self.__min_level]:
            return  # Skip below min level
        
        timestamp = datetime.now().isoformat()
        log_entry = f"[{timestamp}] [{level}] {message}\n"
        
        with open(self.__filepath, 'a') as f:
            f.write(log_entry)
    
    def info(self, message):
        self.log("INFO", message)
    
    def error(self, message):
        self.log("ERROR", message)

# Usage
logger = FileLogger("app.log", min_level="INFO")
logger.info("Application started")  # ✅
logger.error("Connection failed")   # ✅
logger.log("DEBUG", "Debug info")   # ❌ Skipped (below min_level)
```


***

## Coding Exercises

### Exercise 1: Encapsulation - Bank Account

Create a `BankAccount` class with:

- Private `__balance` attribute
- Read-only `balance` property
- `deposit(amount)` with validation (positive only)
- `withdraw(amount)` with validation (positive, sufficient funds)
- `transfer(amount, other_account)` method

```python
# Your implementation here
```


### Exercise 2: Abstraction - Shape Calculator

Create an abstract `Shape` class with:

- Abstract `area()` method
- Abstract `perimeter()` method
- Concrete `describe()` method

Implement `Circle` and `Rectangle` subclasses.

```python
# Your implementation here
```


### Exercise 3: Both - User Profile

Create a `UserProfile` class with:

- Encapsulation: private `__email`, protected `_username`
- Property validation for email (must contain `@`)
- Property validation for username (min 3 chars)
- Computed read-only `display_name` property

Then create an abstract `Notifier` ABC with:

- Abstract `send(message, user)` method
- Implement `EmailNotifier` and `SMSNotifier`

```python
# Your implementation here
```


### Exercise 4: Advanced - Plugin System

Build a plugin system with:

- `Plugin` ABC with `name`, `version`, `initialize()`, `execute()`, `cleanup()`
- At least 3 plugin implementations
- `PluginManager` that registers and runs plugins

```python
# Your implementation here
```


***

## Interview Questions

### Beginner Questions

1. **What is encapsulation in Python?**
    - Answer: Bundling data and methods, restricting direct access via conventions (`_`, `__`)
2. **What does single underscore (`_`) mean in Python?**
    - Answer: Protected member — internal use hint, not enforced
3. **What does double underscore (`__`) do?**
    - Answer: Private member — triggers name mangling (`_ClassName__attr`)
4. **What is the `@property` decorator?**
    - Answer: Enables getter methods with attribute-like syntax
5. **What is abstraction?**
    - Answer: Hiding implementation details, exposing only essential interface
6. **How do you create an abstract class in Python?**
    - Answer: `class MyClass(ABC):` with `@abstractmethod` decorators

### Intermediate Questions

1. **Explain name mangling with an example**
    - Show `__attr` becomes `_ClassName__attr`
2. **How do you create a read-only property?**
    - Answer: `@property` without `@setter`
3. **What happens if a subclass doesn't implement all abstract methods?**
    - Answer: Cannot instantiate — raises `TypeError`
4. **Difference between `_protected` and `__private`?**
    - Answer: `_` is convention, `__` triggers name mangling
5. **How do you create an abstract property?**
    - Answer: `@property` + `@abstractmethod`
6. **Can an abstract class have non-abstract methods?**
    - Answer: Yes, regular methods can exist in ABCs

### Advanced Questions

1. **Design a payment system using abstraction**
    - Create `PaymentProvider` ABC, implement `Stripe`, `PayPal`
2. **Implement validation using encapsulation**
    - Use `@property` with validation in setters
3. **Explain dependency inversion with ABCs**
    - High-level modules depend on abstract interfaces
4. **Create a plugin architecture**
    - Define `Plugin` ABC, implement multiple plugins
5. **When would you use `__private` vs `_protected`?**
    - Answer: `__` for class-internal, `_` for subclass API
6. **How does ABCMeta enforce contracts?**
    - Explain metaclass checking at class creation
7. **Design a repository pattern with abstraction**
    - `Repository` ABC with `get()`, `save()`, `delete()`
8. **Implement computed properties with encapsulation**
    - Read-only properties that calculate values

***

## Summary

### Encapsulation Key Points

- **Purpose**: Protect data, control access, hide implementation
- **Mechanisms**: `_protected`, `__private`, `@property`
- **Python Approach**: Conventions over enforcement (trust developers)
- **Name Mangling**: `__attr` → `_ClassName__attr`
- **Properties**: `@property`, `@setter`, `@deleter` for validated access
- **Best Practice**: Use `_protected` for internal, `@property` for validation


### Abstraction Key Points

- **Purpose**: Hide complexity, define contracts, enable polymorphism
- **Mechanisms**: `ABC`, `@abstractmethod`, `abc` module
- **Enforcement**: Runtime — can't instantiate unimplemented ABCs
- **Use Cases**: Interfaces, plugins, frameworks, dependency injection
- **Best Practice**: Minimal abstract methods, document contracts clearly


### When to Use

| Need | Use |
| :-- | :-- |
| Protect internal data | Encapsulation |
| Validate attribute access | Encapsulation (`@property`) |
| Define interface for subclasses | Abstraction (ABC) |
| Create plugin system | Abstraction (ABC) |
| Build framework base | Abstraction (ABC) |
| Hide implementation details | Both (different levels) |


***

## Cheat Sheet

### Encapsulation Quick Reference

```python
# Public
self.public = "accessible anywhere"

# Protected (convention)
self._protected = "internal/subclass use"

# Private (name-mangled)
self.__private = "class-internal only"

# Name mangling demonstration
class Example:
    def __init__(self):
        self.__secret = "hidden"

e = Example()
print(e.__dict__)  # {'_Example__secret': 'hidden'}
print(e._Example__secret)  # ✅ "hidden"

# Property decorator
class Person:
    def __init__(self, name):
        self._name = name
    
    @property
    def name(self):  # Getter
        return self._name
    
    @name.setter
    def name(self, value):  # Setter
        if len(value) < 2:
            raise ValueError("Too short")
        self._name = value
    
    @property
    def read_only(self):  # No setter = read-only
        return "computed"

# Usage
p = Person("Alice")
print(p.name)    # Getter
p.name = "Bob"   # Setter (with validation)
del p.name       # ❌ No deleter
```


### Abstraction Quick Reference

```python
from abc import ABC, abstractmethod

# Abstract base class
class Shape(ABC):
    @abstractmethod
    def area(self):
        pass
    
    @abstractmethod
    def perimeter(self):
        pass
    
    def describe(self):  # Regular method
        return f"Area: {self.area()}"

# Concrete subclass
class Rectangle(Shape):
    def __init__(self, w, h):
        self.w, self.h = w, h
    
    def area(self):  # Must implement
        return self.w * self.h
    
    def perimeter(self):  # Must implement
        return 2 * (self.w + self.h)

# Abstract property
class Database(ABC):
    @property
    @abstractmethod
    def is_connected(self):
        pass

# Cannot instantiate ABC
# shape = Shape()  # ❌ TypeError

# Can instantiate concrete
rect = Rectangle(5, 10)  # ✅
```


### Common Patterns

```python
# Read-only property
class Immutable:
    def __init__(self, value):
        self.__value = value
    
    @property
    def value(self):
        return self.__value  # No setter

# Validation pattern
class Validated:
    def __init__(self, value):
        self.value = value  # Triggers setter
    
    @property
    def value(self):
        return self.__value
    
    @value.setter
    def value(self, v):
        if v < 0:
            raise ValueError("Must be positive")
        self.__value = v

# Interface pattern
class Processor(ABC):
    @abstractmethod
    def process(self, data):
        pass

# Plugin pattern
class Plugin(ABC):
    @property
    @abstractmethod
    def name(self):
        pass
    
    @abstractmethod
    def execute(self, context):
        pass
```


***

## Practice Problems

### Problem 1: Encapsulation - Temperature Converter

Create a `Temperature` class with:

- Private `__celsius` attribute
- Read-only `celsius` property
- `celsius` setter with validation (≥ -273.15)
- Computed read-only properties: `fahrenheit`, `kelvin`

```python
# Your solution
```


### Problem 2: Abstraction - Vehicle System

Create abstract `Vehicle` class with:

- Abstract `start_engine()`, `stop_engine()`, `get_fuel_type()`
- Concrete `Car` and `Truck` implementations

```python
# Your solution
```


### Problem 3: Both - Inventory Manager

Create:

- `InventoryItem` with encapsulation (private `__price`, validated setter)
- Abstract `InventoryProvider` ABC with `add()`, `remove()`, `get()`
- Concrete `DatabaseInventory` and `FileInventory` implementations

```python
# Your solution
```


### Problem 4: Advanced - Notification System

Build a notification system with:

- Encapsulation: `Notification` class with private `__recipients`, validated `add_recipient()`
- Abstraction: `NotificationChannel` ABC with `send(message)`
- Implement `EmailChannel`, `SMSChannel`, `PushChannel`
- `NotificationManager` that routes to multiple channels

```python
# Your solution
```


### Problem 5: Real-world - API Client

Create an API client with:

- Encapsulation: private `__api_key`, `__base_url`, validated properties
- Abstraction: `APIClient` ABC with `get()`, `post()`, `delete()`
- Implement `RESTClient` and `GraphQLClient`
- Add authentication middleware using ABC pattern

```python
# Your solution
```


***

**Congratulations!** You've completed this comprehensive guide on Python Encapsulation and Abstraction. Practice the exercises, review the cheat sheet, and apply these concepts in your projects to write better, more maintainable code.

