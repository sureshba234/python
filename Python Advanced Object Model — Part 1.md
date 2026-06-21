<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" style="height:64px;margin-right:32px"/>

# Python Advanced Object Model — Part 1

> A complete internal guide to descriptors and Method Resolution Order (MRO) in Python

***

## Table of Contents

1. [Descriptors](#descriptors)
    - [What are Descriptors?](#what-are-descriptors)
    - [Descriptor Protocol](#descriptor-protocol)
    - [Data vs Non-Data Descriptors](#data-vs-non-data-descriptors)
    - [Attribute Lookup Order](#attribute-lookup-order)
    - [Functions as Descriptors](#functions-as-descriptors)
    - [Built-in Descriptor Types](#built-in-descriptor-types)
    - [Custom Descriptors](#custom-descriptors)
    - [Validation Descriptors](#validation-descriptors)
    - [Lazy Loading Descriptors](#lazy-loading-descriptors)
    - [ORM and Framework Usage](#orm-and-framework-usage)
    - [Internal Execution Flow](#internal-execution-flow)
    - [Best Practices](#best-practices)
    - [Common Mistakes](#common-mistakes)
2. [Method Resolution Order (MRO)](#method-resolution-order-mro)
    - [What is MRO?](#what-is-mro)
    - [Single Inheritance](#single-inheritance)
    - [Multiple Inheritance](#multiple-inheritance)
    - [Diamond Problem](#diamond-problem)
    - [C3 Linearization](#c3-linearization)
    - [MRO Algorithm](#mro-algorithm)
    - [MRO Inspection Tools](#mro-inspection-tools)
    - [super() Internals](#super-internals)
    - [Step-by-Step Traces](#step-by-step-traces)
    - [Enterprise Examples](#enterprise-examples)
    - [Best Practices](#best-practices-mro)
    - [Common Mistakes](#common-mistakes-mro)
3. [Summary](#summary)
4. [Cheat Sheet](#cheat-sheet)
5. [Interview Questions](#interview-questions)
6. [Exercises](#exercises)

***

## Descriptors

### What are Descriptors?

Descriptors are **objects that customize attribute access** for other objects. They implement special methods (`__get__`, `__set__`, `__delete__`) that Python calls during attribute lookup, assignment, or deletion.

```python
# Conceptual definition
class Descriptor:
    def __get__(self, instance, owner):
        """Called when attribute is accessed"""
    
    def __set__(self, instance, value):
        """Called when attribute is assigned"""
    
    def __delete__(self, instance):
        """Called when attribute is deleted"""
```

**Key insight**: Descriptors are the mechanism behind `property`, `classmethod`, `staticmethod`, and many framework features.

### Descriptor Protocol

The descriptor protocol consists of three methods:


| Method | Signature | Called When |
| :-- | :-- | :-- |
| `__get__` | `__get__(self, instance, owner)` | Attribute access (`obj.attr`) |
| `__set__` | `__set__(self, instance, value)` | Attribute assignment (`obj.attr = value`) |
| `__delete__` | `__delete__(self, instance)` | Attribute deletion (`del obj.attr`) |

```python
class Descriptor:
    def __get__(self, instance, owner):
        print(f"Getting {self} from {instance}")
        return "value"
    
    def __set__(self, instance, value):
        print(f"Setting {self} to {value} on {instance}")
    
    def __delete__(self, instance):
        print(f"Deleting {self} from {instance}")

class MyClass:
    desc = Descriptor()

obj = MyClass()
obj.desc           # Triggers __get__
obj.desc = 100     # Triggers __set__
del obj.desc       # Triggers __delete__
```


### `__get__`

The `__get__` method receives three parameters:

- `self`: The descriptor instance itself
- `instance`: The object instance (or `None` if accessed from class)
- `owner`: The class containing the descriptor

```python
class GetDescriptor:
    def __get__(self, instance, owner):
        if instance is None:
            # Accessed from class
            return f"Class access: {owner.__name__}"
        else:
            # Accessed from instance
            return f"Instance access: {instance}"

class MyClass:
    desc = GetDescriptor()

print(MyClass.desc)   # "Class access: MyClass"
print(MyClass().desc) # "Instance access: MyClass()"
```


### `__set__`

The `__set__` method receives:

- `self`: The descriptor instance
- `instance`: The object instance
- `value`: The value being assigned

```python
class SetDescriptor:
    def __init__(self):
        self._value = 0
    
    def __set__(self, instance, value):
        print(f"Setting value to {value}")
        self._value = value
    
    def __get__(self, instance, owner):
        return self._value

class MyClass:
    desc = SetDescriptor()

obj = MyClass()
obj.desc = 50      # Triggers __set__
print(obj.desc)    # Triggers __get__, returns 50
```


### `__delete__`

The `__delete__` method receives:

- `self`: The descriptor instance
- `instance`: The object instance

```python
class DeleteDescriptor:
    def __init__(self):
        self._value = "exists"
    
    def __get__(self, instance, owner):
        return self._value
    
    def __delete__(self, instance):
        print("Deleting attribute")
        del self._value

class MyClass:
    desc = DeleteDescriptor()

obj = MyClass()
print(obj.desc)    # "exists"
del obj.desc       # Triggers __delete__
```


### Data vs Non-Data Descriptors

#### Data Descriptors

Data descriptors implement **at least `__set__` or `__delete__`** (in addition to `__get__`).

```python
class DataDescriptor:
    def __get__(self, instance, owner):
        return "data"
    
    def __set__(self, instance, value):
        pass  # Implements __set__ → Data descriptor
```

**Priority**: Data descriptors have **higher priority** than instance attributes.

#### Non-Data Descriptors

Non-data descriptors implement **only `__get__`** (no `__set__` or `__delete__`).

```python
class NonDataDescriptor:
    def __get__(self, instance, owner):
        return "non-data"
    # No __set__ or __delete__ → Non-data descriptor
```

**Priority**: Non-data descriptors have **lower priority** than instance attributes.

### Attribute Lookup Order

Python's attribute lookup follows this exact order:

```
┌─────────────────────────────────────────┐
│  1. Data Descriptors (in class dict)    │ ← Highest priority
│  2. Instance attributes (__dict__)      │
│  3. Non-Data Descriptors (in class dict)│
│  4. Class attributes                    │
│  5. Parent classes (via MRO)            │ ← Lowest priority
└─────────────────────────────────────────┘
```


#### Diagram: Attribute Lookup Flow

```
obj.attr
   │
   ├─► Is 'attr' in obj.__class__.__dict__?
   │    │
   │    ├─► YES: Is it a data descriptor?
   │    │    │
   │    │    ├─► YES: Call descriptor.__get__(obj, cls)
   │    │    │         └─── RETURN
   │    │    │
   │    │    ├─► NO: Is it a non-data descriptor?
   │    │    │    │
   │    │    │    ├─► YES: Check obj.__dict__ first
   │    │    │    │    │
   │    │    │    │    ├─► In __dict__? RETURN instance attr
   │    │    │    │    │
   │    │    │    │    ├─► NOT in __dict__? Call descriptor.__get__()
   │    │    │    │    │         └─── RETURN
   │    │    │    │
   │    │    │    └─► NO: Is it a regular attribute?
   │    │    │         │
   │    │    │         ├─► Return class attribute
   │    │    │
   │    │
   │    └─► NO: Check obj.__dict__
   │         │
   │         ├─► Found? RETURN instance attribute
   │         │
   │         └─► NOT found: Continue MRO search
   │
   └─► Continue through parent classes (MRO)
```

```python
class DataDesc:
    def __get__(self, instance, owner):
        return "data descriptor"
    
    def __set__(self, instance, value):
        pass

class NonDataDesc:
    def __get__(self, instance, owner):
        return "non-data descriptor"

class MyClass:
    data = DataDesc()
    non_data = NonDataDesc()

obj = MyClass()
obj.__dict__['data'] = "instance data"      # Won't override data descriptor
obj.__dict__['non_data'] = "instance nd"    # Will override non-data descriptor

print(obj.data)      # "data descriptor" (data descriptor wins)
print(obj.non_data)  # "instance nd" (instance attr wins)
```


### Functions as Descriptors

**Functions are non-data descriptors**. When you access a function through an instance, Python calls `function.__get__(instance, class)` to create a **bound method**.

```python
class MyClass:
    def method(self):
        return "bound method"

obj = MyClass()

# Internal flow:
# obj.method → MyClass.method.__get__(obj, MyClass)
print(obj.method())  # "bound method"

# Accessing from class:
print(MyClass.method)  # Unbound function
```

```python
# Verify functions are descriptors
class MyClass:
    def method(self):
        pass

print(hasattr(MyClass.method, '__get__'))  # True
print(isinstance(MyClass.method, object))  # True
```


### Built-in Descriptor Types

#### `property`

`property` is the most common data descriptor for creating managed attributes.

```python
class Person:
    def __init__(self, name):
        self._name = name
    
    @property
    def name(self):
        """Getter"""
        return self._name
    
    @name.setter
    def name(self, value):
        """Setter with validation"""
        if not value:
            raise ValueError("Name cannot be empty")
        self._name = value
    
    @name.deleter
    def name(self):
        """Deleter"""
        del self._name

person = Person("Alice")
print(person.name)    # "Alice" (__get__)
person.name = "Bob"   # __set__
del person.name       # __delete__
```

**Internal implementation** (simplified):

```python
class Property:
    def __init__(self, fget=None, fset=None, fdel=None, doc=None):
        self.fget = fget
        self.fset = fset
        self.fdel = fdel
        self.__doc__ = doc
    
    def __get__(self, instance, owner):
        if instance is None:
            return self
        if self.fget is None:
            raise AttributeError("Cannot read attribute")
        return self.fget(instance)
    
    def __set__(self, instance, value):
        if self.fset is None:
            raise AttributeError("Cannot set attribute")
        self.fset(instance, value)
    
    def __delete__(self, instance):
        if self.fdel is None:
            raise AttributeError("Cannot delete attribute")
        self.fdel(instance)
```


#### `classmethod`

`classmethod` is a non-data descriptor that binds a method to the class instead of the instance.

```python
class MyClass:
    count = 0
    
    @classmethod
    def increment(cls):
        cls.count += 1
        return cls.count

print(MyClass.increment())  # "1" (cls = MyClass)
print(MyClass().increment()) # "2" (cls = MyClass)
```

**Internal behavior**:

```python
# classmethod.__get__(instance, owner) returns:
#   - If instance is None: the original function
#   - If instance exists: a function that passes owner as first arg
```


#### `staticmethod`

`staticmethod` is a non-data descriptor that returns the original function without binding.

```python
class MyClass:
    @staticmethod
    def static_method(x, y):
        return x + y

print(MyClass.static_method(2, 3))   # 5
print(MyClass().static_method(2, 3)) # 5 (no cls or self)
```


### Custom Descriptors

Create your own descriptors for reusable attribute behavior.

```python
class Typed:
    """Descriptor that enforces type"""
    def __init__(self, expected_type):
        self.expected_type = expected_type
        self._value = None
    
    def __get__(self, instance, owner):
        if instance is None:
            return self
        return self._value
    
    def __set__(self, instance, value):
        if not isinstance(value, self.expected_type):
            raise TypeError(f"Expected {self.expected_type}, got {type(value)}")
        self._value = value

class Employee:
    name = Typed(str)
    age = Typed(int)
    salary = Typed(float)

emp = Employee()
emp.name = "John"     # OK
emp.age = 30          # OK
emp.salary = 50000.0  # OK
emp.age = "thirty"    # TypeError!
```


### Validation Descriptors

Descriptors for enforcing business rules and validation.

```python
class Validated:
    """Descriptor with validation callback"""
    def __init__(self, validator):
        self.validator = validator
        self._values = {}
    
    def __get__(self, instance, owner):
        if instance is None:
            return self
        return self._values.get(instance)
    
    def __set__(self, instance, value):
        if not self.validator(value):
            raise ValueError(f"Invalid value: {value}")
        self._values[instance] = value

def positive(n):
    return n > 0

def email(addr):
    return '@' in addr and '.' in addr

class Product:
    price = Validated(positive)
    stock = Validated(positive)
    email = Validated(email)

p = Product()
p.price = 99.99    # OK
p.stock = 100      # OK
p.email = "test@a.com"  # OK
p.price = -5       # ValueError!
```


### Lazy Loading Descriptors

Descriptors that load values on first access.

```python
class Lazy:
    """Descriptor that loads value on first access"""
    def __init__(self, factory):
        self.factory = factory
        self._values = {}
    
    def __get__(self, instance, owner):
        if instance is None:
            return self
        if instance not in self._values:
            print(f"Lazy loading for {instance}")
            self._values[instance] = self.factory()
        return self._values[instance]
    
    def __set__(self, instance, value):
        self._values[instance] = value

def create_database():
    print("Creating database connection...")
    return "DatabaseConnection()"

class AppConfig:
    db = Lazy(create_database)

config = AppConfig()
print(config.db)  # Lazy loads on first access
print(config.db)  # Returns cached value
```


### ORM and Framework Usage

Descriptors are fundamental to ORM frameworks like SQLAlchemy, Django ORM, and marshmallow.

#### SQLAlchemy Example (simplified)

```python
class Column:
    """SQLAlchemy-like column descriptor"""
    def __init__(self, type_, name=None):
        self.type_ = type_
        self.name = name
        self._values = {}
    
    def __get__(self, instance, owner):
        if instance is None:
            return self
        return self._values.get(instance)
    
    def __set__(self, instance, value):
        if not isinstance(value, self.type_):
            raise TypeError(f"Expected {self.type_}")
        self._values[instance] = value

class User:
    id = Column(int, name="id")
    name = Column(str, name="name")
    age = Column(int, name="age")

user = User()
user.id = 1
user.name = "Alice"
user.age = 30

# These values are stored in Column._values, not user.__dict__
print(user.name)  # "Alice"
```


#### Django Model Fields (conceptual)

```python
class CharField:
    """Django-like CharField descriptor"""
    def __init__(self, max_length=None):
        self.max_length = max_length
        self._values = {}
    
    def __get__(self, instance, owner):
        if instance is None:
            return self
        return self._values.get(instance)
    
    def __set__(self, instance, value):
        if self.max_length and len(value) > self.max_length:
            raise ValueError(f"Max length is {self.max_length}")
        self._values[instance] = value

class Model:
    pass

class Article(Model):
    title = CharField(max_length=100)
    content = CharField()

article = Article()
article.title = "Hello"  # OK
article.title = "x" * 150  # ValueError!
```


### Internal Execution Flow

#### Step-by-Step: Attribute Access

```python
class Descriptor:
    def __get__(self, instance, owner):
        return "descriptor value"

class MyClass:
    attr = Descriptor()

obj = MyClass()
result = obj.attr  # What happens internally?
```

**Execution flow**:

1. Python evaluates `obj.attr`
2. Looks up `"attr"` in `MyClass.__dict__`
3. Finds `Descriptor` instance
4. Checks if it's a data descriptor (has `__set__` or `__delete__`)
5. **Yes**: Calls `Descriptor.__get__(obj, MyClass)`
6. Returns `"descriptor value"`
7. **Never** checks `obj.__dict__`
```python
# CPython internal (simplified):
def PyObject_GetAttr(obj, name):
    # 1. Check for data descriptors in class dict
    for cls in obj.__class__.__mro__:
        if name in cls.__dict__:
            attr = cls.__dict__[name]
            if hasattr(attr, '__set__') or hasattr(attr, '__delete__'):
                return attr.__get__(obj, obj.__class__)
    
    # 2. Check instance __dict__
    if name in obj.__dict__:
        return obj.__dict__[name]
    
    # 3. Check for non-data descriptors
    for cls in obj.__class__.__mro__:
        if name in cls.__dict__:
            attr = cls.__dict__[name]
            if hasattr(attr, '__get__'):
                return attr.__get__(obj, obj.__class__)
    
    # 4. Raise AttributeError
    raise AttributeError(name)
```


### Best Practices

1. **Use `@property` for simple attribute access** — it's built-in and well-documented
2. **Store descriptor values in a separate dict** (not `instance.__dict__`) to avoid conflicts
3. **Always handle `instance is None`** in `__get__` for class-level access
4. **Document descriptor behavior** clearly for other developers
5. **Use type hints** for descriptor parameters and return values
6. **Cache computed values** in lazy descriptors to avoid repeated computation
7. **Raise appropriate exceptions** (`TypeError`, `ValueError`, `AttributeError`)
```python
class GoodDescriptor:
    """Well-documented descriptor with proper error handling"""
    def __init__(self, default=None):
        self.default = default
        self._values = {}
    
    def __get__(self, instance, owner):
        if instance is None:
            return self
        return self._values.get(instance, self.default)
    
    def __set__(self, instance, value):
        if value is None:
            raise ValueError("Value cannot be None")
        self._values[instance] = value
```


### Common Mistakes

#### Mistake 1: Storing in `instance.__dict__`

```python
# ❌ WRONG: Conflicts with attribute lookup
class BadDescriptor:
    def __set__(self, instance, value):
        instance.__dict__[self.name] = value  # Creates circular reference
    
    def __get__(self, instance, owner):
        return instance.__dict__[self.name]

# ✅ CORRECT: Use separate dict
class GoodDescriptor:
    def __init__(self):
        self._values = {}
    
    def __set__(self, instance, value):
        self._values[instance] = value
    
    def __get__(self, instance, owner):
        return self._values.get(instance)
```


#### Mistake 2: Not handling `instance is None`

```python
# ❌ WRONG: Crashes on class access
class BadDescriptor:
    def __get__(self, instance, owner):
        return instance.value  # instance is None!

# ✅ CORRECT
class GoodDescriptor:
    def __get__(self, instance, owner):
        if instance is None:
            return self
        return instance.value
```


#### Mistake 3: Confusing data vs non-data

```python
# ❌ WRONG: Unexpected behavior due to non-data
class NonData:
    def __get__(self, instance, owner):
        return "non-data"

class MyClass:
    attr = NonData()

obj = MyClass()
obj.__dict__['attr'] = "instance"
print(obj.attr)  # "instance" (non-data allows override)

# ✅ Use data descriptor if you want to prevent override
class Data:
    def __get__(self, instance, owner):
        return "data"
    
    def __set__(self, instance, value):
        pass  # Makes it a data descriptor
```


#### Mistake 4: Missing docstrings

```python
# ❌ WRONG: No documentation
class MysteryDescriptor:
    def __get__(self, instance, owner):
        ...

# ✅ CORRECT: Document behavior
class MysteryDescriptor:
    """
    Descriptor that returns cached computation.
    
    Behavior:
    - Returns cached value on first access
    - Raises AttributeError if not set
    """
    def __get__(self, instance, owner):
        ...
```


***

## Method Resolution Order (MRO)

### What is MRO?

**Method Resolution Order (MRO)** is the order in which Python searches for methods in a class hierarchy during inheritance. It determines which method is called when multiple classes in the hierarchy define the same method.

```python
class A:
    def method(self):
        return "A"

class B:
    def method(self):
        return "B"

class C(A, B):
    pass

obj = C()
print(obj.method())  # "A" (MRO: C → A → B)
```


### Single Inheritance

Single inheritance has a straightforward MRO: child → parent → grandparent.

```python
class Grandparent:
    def method(self):
        return "Grandparent"

class Parent(Grandparent):
    def method(self):
        return "Parent"

class Child(Parent):
    pass

print(Child.mro())
# [Child, Parent, Grandparent, object]

obj = Child()
print(obj.method())  # "Parent"
```

**MRO trace**:

1. Look in `Child` → not found
2. Look in `Parent` → found! Return `"Parent"`

### Multiple Inheritance

Multiple inheritance creates complex MRO chains.

```python
class A:
    def method(self):
        return "A"

class B:
    def method(self):
        return "B"

class C(A, B):
    pass

print(C.mro())
# [C, A, B, object]

obj = C()
print(obj.method())  # "A" (A comes before B in MRO)
```


### Diamond Problem

The diamond problem occurs when a class inherits from two classes that both inherit from a common base.

```
       Base
      /    \
     A      B
      \    /
       Child
```

```python
class Base:
    def method(self):
        return "Base"

class A(Base):
    def method(self):
        return "A"

class B(Base):
    def method(self):
        return "B"

class Child(A, B):
    pass

print(Child.mro())
# [Child, A, B, Base, object]

obj = Child()
print(obj.method())  # "A" (A comes before B)
```

**Python's solution**: C3 linearization ensures Base is visited only once, and order follows class declaration.

### C3 Linearization

**C3 linearization** is the algorithm Python uses to compute MRO. It merges inheritance lists while preserving:

1. Local order (order in class definition)
2. Consistency (no class appears before its descendants)

#### Algorithm Steps

Given classes with inheritance:

```python
class D: pass
class E: pass
class F: pass

class A(D, E): pass
class B(E, F): pass
class C(B, A): pass
```

**C3 merge process**:

1. Start with each class's MRO list:
    - `D`: [D, object]
    - `E`: [E, object]
    - `F`: [F, object]
    - `A`: [A, D, E, object]
    - `B`: [B, E, F, object]
    - `C`: [C, B, A, object]
2. Merge lists using C3 algorithm:
    - Pick first element from first list if it doesn't appear in any other list's tail
    - Remove picked element from all lists
    - Repeat until all lists empty
3. Result: `[C, B, A, D, E, F, object]`
```python
print(C.mro())
# [C, B, A, D, E, F, object]
```


### MRO Algorithm

Python's MRO algorithm (simplified):

```python
def compute_mro(class_):
    """Compute MRO using C3 linearization"""
    if class_ in class_.__dict__.get('__mro__', []):
        return class_.__mro__
    
    # Get direct bases
    bases = class_.__bases__
    
    if not bases:
        return [class_, object]
    
    # Recursively compute MRO for bases
    base_mros = [compute_mro(base) for base in bases]
    
    # Merge using C3
    mro = c3_merge([base_mros] + [list(bases) + [object]])
    
    class_.__mro__ = mro
    return mro
```


### MRO Inspection Tools

#### `mro()` method

```python
class A: pass
class B: pass
class C(A, B): pass

print(C.mro())
# [C, A, B, object]

for cls in C.mro():
    print(cls.__name__)
# C
# A
# B
# object
```


#### `__mro__` attribute

```python
class A: pass
class B: pass
class C(A, B): pass

print(C.__mro__)
# (C, A, B, object)

# __mro__ is a tuple (immutable)
# mro() returns a list (mutable copy)
```


#### `inspect.getmro()`

```python
import inspect

class A: pass
class B: pass
class C(A, B): pass

print(inspect.getmro(C))
# (C, A, B, object)
```


### `super()` Internals

`super()` uses MRO to find the **next class** after the current one.

```python
class A:
    def method(self):
        return "A"

class B:
    def method(self):
        return "B"

class C(A, B):
    def method(self):
        return "C -> " + super().method()

print(C().method())  # "C -> A"
```

**What `super()` does internally**:

```python
# super() equivalent:
class Super:
    def __init__(self, current_class, instance):
        self.current_class = current_class
        self.instance = instance
        self.mro = current_class.__mro__
        # Find next class after current_class in MRO
        self.next_index = self.mro.index(current_class) + 1
    
    def __getattr__(self, name):
        # Search in classes after current_class
        for cls in self.mro[self.next_index:]:
            if name in cls.__dict__:
                return cls.__dict__[name]
        raise AttributeError(name)

# Usage:
# super() → Super(C, instance)
# super().method() → finds method in A (next after C in MRO)
```


#### `super()` with arguments

```python
class A:
    def method(self):
        return "A"

class B:
    def method(self):
        return "B"

class C(A, B):
    def method(self):
        # Explicit super: skip A, go to B
        return "C -> " + super(C, self).method()

print(C().method())  # "C -> B"
```


### Step-by-Step Traces

#### Trace 1: Simple Diamond

```python
class Base:
    def method(self):
        print("Base")

class A(Base):
    def method(self):
        print("A")
        super().method()

class B(Base):
    def method(self):
        print("B")
        super().method()

class C(A, B):
    def method(self):
        print("C")
        super().method()

C().method()
```

**Execution trace**:

```
1. C().method() → prints "C"
2. super().method() in C → next in MRO after C is A
3. A.method() → prints "A"
4. super().method() in A → next after A is B
5. B.method() → prints "B"
6. super().method() in B → next after B is Base
7. Base.method() → prints "Base"
8. super().method() in Base → next after Base is object (no method)

Output:
C
A
B
Base
```

MRO: `[C, A, B, Base, object]`

#### Trace 2: Complex Inheritance

```python
class X:
    def method(self):
        print("X")

class Y:
    def method(self):
        print("Y")

class Z(X, Y):
    def method(self):
        print("Z")
        super().method()

class A(Z, Y):
    def method(self):
        print("A")
        super().method()

class B(Z, X):
    def method(self):
        print("B")
        super().method()

class C(A, B):
    def method(self):
        print("C")
        super().method()

C().method()
```

**MRO**: `[C, A, B, Z, X, Y, object]`

**Execution**:

1. C → "C"
2. super() → A
3. A → "A"
4. super() → B
5. B → "B"
6. super() → Z
7. Z → "Z"
8. super() → X
9. X → "X"
10. super() → Y
11. Y → "Y"
12. super() → object (stop)

Output:

```
C
A
B
Z
X
Y
```


### Enterprise Examples

#### Example 1: Plugin System

```python
class PluginBase:
    def process(self, data):
        return data
    
    def validate(self, data):
        return True

class LoggingPlugin(PluginBase):
    def process(self, data):
        print(f"[LOG] Processing: {data}")
        return super().process(data)

class ValidationPlugin(PluginBase):
    def process(self, data):
        if not self.validate(data):
            raise ValueError("Invalid data")
        return super().process(data)

class TransformPlugin(LoggingPlugin, ValidationPlugin):
    def process(self, data):
        transformed = data.upper()
        return super().process(transformed)

plugin = TransformPlugin()
print(plugin.process("hello"))
# [LOG] Processing: HELLO
# HELLO
```

**MRO**: `[TransformPlugin, LoggingPlugin, ValidationPlugin, PluginBase, object]`

#### Example 2: Middleware Chain

```python
class Middleware:
    def handle(self, request):
        return request

class AuthMiddleware(Middleware):
    def handle(self, request):
        if not request.get('auth'):
            raise PermissionError("No auth")
        print("[Auth] Checked")
        return super().handle(request)

class LogMiddleware(Middleware):
    def handle(self, request):
        print(f"[Log] Request: {request}")
        return super().handle(request)

class CacheMiddleware(LogMiddleware, AuthMiddleware):
    def handle(self, request):
        if request.get('cached'):
            print("[Cache] Returned")
            return request
        print("[Cache] Miss")
        return super().handle(request)

middleware = CacheMiddleware()
result = middleware.handle({'auth': True, 'cached': False})
# [Cache] Miss
# [Log] Request: {'auth': True, 'cached': False}
# [Auth] Checked
```

**MRO**: `[CacheMiddleware, LogMiddleware, AuthMiddleware, Middleware, object]`

### Best Practices (MRO)

1. **Use cooperative inheritance** — always call `super()` in multiple inheritance
2. **Keep inheritance trees shallow** — deep trees make MRO hard to predict
3. **Document MRO** explicitly in complex hierarchies
4. **Use single inheritance** when possible for clarity
5. **Check MRO early** with `class.mro()` during development
6. **Avoid mixing mixins with stateful classes** unless carefully designed
7. **Name mixins clearly** (e.g., `LoggingMixin`, `ValidationMixin`)
```python
# ✅ Good: Cooperative inheritance with super()
class Base:
    def process(self):
        return "base"

class MixinA(Base):
    def process(self):
        return "a" + super().process()

class MixinB(Base):
    def process(self):
        return "b" + super().process()

class Combined(MixinA, MixinB):
    def process(self):
        return "combined" + super().process()

print(Combined().process())  # "combinedabbase"
```


### Common Mistakes (MRO)

#### Mistake 1: Not calling `super()`

```python
# ❌ WRONG: Breaks cooperative inheritance
class A:
    def method(self):
        return "A"

class B:
    def method(self):
        return "B"

class C(A, B):
    def method(self):
        return "C"  # Never calls super()!

print(C().method())  # "C" only (B is skipped)

# ✅ CORRECT
class C(A, B):
    def method(self):
        return "C" + super().method()  # "CA"
```


#### Mistake 2: Ignoring MRO

```python
# ❌ WRONG: Unexpected behavior
class A:
    def method(self):
        return "A"

class B:
    def method(self):
        return "B"

class C(B, A):  # Note: B before A
    pass

print(C().method())  # "B" (not A!)

# ✅ CORRECT: Check MRO explicitly
print(C.mro())  # [C, B, A, object]
```


#### Mistake 3: Circular inheritance

```python
# ❌ WRONG: Circular reference
class A(B): pass
class B(A): pass  # RecursionError!

# ✅ CORRECT: Avoid circular dependencies
class Base: pass
class A(Base): pass
class B(Base): pass
```


#### Mistake 4: Mixing stateful classes with mixins

```python
# ❌ WRONG: State conflicts
class Database:
    def __init__(self):
        self.conn = "connection"

class Logging:
    def __init__(self):
        self.log = []  # Conflicts with Database.__init__
    
    def log_msg(self, msg):
        self.log.append(msg)

class DBWithLogging(Database, Logging):
    def __init__(self):
        Database.__init__(self)
        # Logging.__init__(self) never called!

# ✅ CORRECT: Use super() in all __init__
class Database:
    def __init__(self):
        self.conn = "connection"
        super().__init__()

class Logging:
    def __init__(self):
        self.log = []
        super().__init__()

class DBWithLogging(Database, Logging):
    def __init__(self):
        super().__init__()
```


***

## Summary

### Descriptors

| Concept | Key Point |
| :-- | :-- |
| **Descriptor** | Object with `__get__`, `__set__`, `__delete__` |
| **Data descriptor** | Has `__set__` or `__delete__` (higher priority) |
| **Non-data descriptor** | Only `__get__` (lower priority) |
| **Lookup order** | Data desc → instance attr → non-data desc → class attr → MRO |
| **Functions** | Non-data descriptors (create bound methods) |
| **`property`** | Data descriptor for managed attributes |
| **`classmethod`** | Non-data descriptor binding to class |
| **`staticmethod`** | Non-data descriptor returning unbound function |

### MRO

| Concept | Key Point |
| :-- | :-- |
| **MRO** | Order Python searches for methods in inheritance |
| **Single inheritance** | Child → Parent → Grandparent → object |
| **Multiple inheritance** | Follows declaration order + C3 linearization |
| **Diamond problem** | Solved by C3 ( Base visited once) |
| **C3 linearization** | Merges MRO lists preserving order and consistency |
| **`super()`** | Finds next class in MRO after current class |
| **`class.mro()`** | Returns MRO as list |
| **`class.__mro__`** | Returns MRO as tuple |


***

## Cheat Sheet

### Descriptor Protocol

```python
class Descriptor:
    def __get__(self, instance, owner):
        if instance is None:
            return self  # Class access
        return value     # Instance access
    
    def __set__(self, instance, value):
        pass  # Data descriptor
    
    def __delete__(self, instance):
        pass  # Data descriptor
```


### Attribute Lookup Priority

```
1. Data descriptors (class dict)
2. Instance attributes (__dict__)
3. Non-data descriptors (class dict)
4. Class attributes
5. Parent classes (MRO)
```


### MRO Inspection

```python
# Get MRO as list
class.mro()

# Get MRO as tuple
class.__mro__

# Using inspect
inspect.getmro(class)
```


### Common Descriptor Patterns

```python
# Type validation
class Typed:
    def __init__(self, type_):
        self.type_ = type_
        self._values = {}
    
    def __get__(self, instance, owner):
        return self._values.get(instance)
    
    def __set__(self, instance, value):
        if not isinstance(value, self.type_):
            raise TypeError()
        self._values[instance] = value

# Lazy loading
class Lazy:
    def __init__(self, factory):
        self.factory = factory
        self._values = {}
    
    def __get__(self, instance, owner):
        if instance not in self._values:
            self._values[instance] = self.factory()
        return self._values[instance]
```


### MRO Best Practices

```python
# ✅ Always use super() in multiple inheritance
class Base:
    def method(self):
        return "base"

class A(Base):
    def method(self):
        return "A" + super().method()

class B(Base):
    def method(self):
        return "B" + super().method()

class C(A, B):
    def method(self):
        return "C" + super().method()
```


***

## Interview Questions

### Descriptors

1. **What is a descriptor?**
An object that implements `__get__`, `__set__`, or `__delete__` to customize attribute access.
2. **Difference between data and non-data descriptors?**
Data descriptors have `__set__` or `__delete__`; non-data only have `__get__`. Data descriptors have higher priority than instance attributes.
3. **What is the attribute lookup order?**
Data descriptors → instance attributes → non-data descriptors → class attributes → parent classes (MRO).
4. **How does `property` work internally?**
`property` is a data descriptor that calls getter/setter/deleter functions during attribute access.
5. **Why are functions descriptors?**
Functions implement `__get__`, which creates bound methods when accessed from instances.
6. **Write a custom descriptor for type validation.**
7. **What happens when you access a descriptor from a class vs instance?**
From class: `instance` is `None`. From instance: `instance` is the object.

### MRO

1. **What is MRO?**
Method Resolution Order — the order Python searches for methods in inheritance hierarchy.
2. **How does Python solve the diamond problem?**
Using C3 linearization, which ensures base classes are visited once in a consistent order.
3. **What is C3 linearization?**
Algorithm that merges MRO lists while preserving local order and consistency.
4. **How does `super()` work?**
`super()` finds the next class in MRO after the current class and calls its method.
5. **What's the difference between `mro()` and `__mro__`?**
`mro()` returns a list; `__mro__` returns a tuple.
6. **Given classes A, B, C where C(A, B), what is C's MRO?**
`[C, A, B, object]`
7. **Why is cooperative inheritance important?**
Ensures all classes in hierarchy get a chance to execute their methods via `super()`.

***

## Exercises

### Exercise 1: Type-Checking Descriptor

Create a `Typed` descriptor that enforces type checking:

```python
class Typed:
    # Implement __init__, __get__, __set__
    pass

class Person:
    name = Typed(str)
    age = Typed(int)

p = Person()
p.name = "Alice"  # OK
p.age = 30        # OK
p.age = "thirty"  # TypeError!
```


### Exercise 2: Lazy Descriptor

Create a `Lazy` descriptor that loads values on first access:

```python
class Lazy:
    # Store computed values per instance
    pass

class Config:
    db = Lazy(lambda: "DatabaseConnection()")

c = Config()
print(c.db)  # Loads on first access
print(c.db)  # Returns cached
```


### Exercise 3: MRO Trace

Given this hierarchy:

```python
class A:
    def method(self):
        print("A")
        super().method()

class B:
    def method(self):
        print("B")
        super().method()

class C(A, B):
    def method(self):
        print("C")
        super().method()
```

1. What is C's MRO?
2. What is the output of `C().method()`?
3. Trace each `super()` call.

### Exercise 4: Validation Descriptor

Create a `Validated` descriptor with a validator function:

```python
def positive(n):
    return n > 0

class Product:
    price = Validated(positive)

p = Product()
p.price = 100   # OK
p.price = -50   # ValueError!
```


### Exercise 5: Mixin with super()

Create cooperative mixins:

```python
class LoggingMixin:
    def process(self):
        print("[LOG]")
        return super().process()

class ValidationMixin:
    def process(self):
        print("[VALID]")
        return super().process()

class Base:
    def process(self):
        return "base"

class Combined(LoggingMixin, ValidationMixin, Base):
    def process(self):
        return "combined" + super().process()
```

What is the output of `Combined().process()`?

### Exercise 6: Debug MRO

```python
class X: pass
class Y: pass
class Z(X, Y): pass
class A(Z, Y): pass
class B(Z, X): pass
class C(A, B): pass

print(C.mro())
# What is the output? Why?
```


***

> **Next**: Part 2 will cover metaclasses, `__init_subclass__`, and advanced class creation patterns.

